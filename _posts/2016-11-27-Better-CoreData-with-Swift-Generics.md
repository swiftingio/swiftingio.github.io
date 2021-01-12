---
layout: post
author: michal
title: \#28 Better CoreData with Swift Generics
excerpt: 
---

In [Issue #25](https://swifting.io/blog/2016/09/25/25-core-data-in-ios10-nspersistentcontainer?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) I have been talking about building a modern CoreData service using ```NSPersistentStoreContainer```. This resulted in a lot of boilerplate code removed and a simpler API. Now, if you are using architectures like VIPER, VIP (see more in [Issue #24](https://swifting.io/blog/2016/09/07/architecture-wars-a-new-hope/)) or simply ensuring immutability of your models, you are probably wrapping your CoreData's ```NSManagedObject``` subclasses into structs of some sort and use them in upper layers of your app. 

I will not try convince you here of superiority of having additional layer of immutable struct based model. You can find proper articles in the references section. I will just say what convinced me - thread safety, data consistency and no shared state.


This post will introduce two approaches that tap into the power of Swift generics and protocols. First was the initially designed for use in our app while the second is an evolution of the first one. Second one just suits our current needs better. 

#### How does it work?
In the past we have been trying various approaches. There were worker classes that did mapping for the entire model, there were extensions or methods added to ```NSManagedObject``` subclasses and so on. These were good solutions but sometimes we have had a feeling that we are repeating ourselves too often. With Swift generics we have decided to redo our stack.

Key components:

1. ```CoreDataWorker``` - has methods to get, remove, upsert, etc. It is generic and works with any pair of model struct and ManagedObject.
2. ```ManagedObjectProtocol``` with ```toEntity()``` method. It ensures mapping from CoreData's ```NSManagedObject``` to struct model (called Entity)
3. ```ManagedObjectConvertible``` with ```toManagedObject()``` method. Provides reverse operation of converting struct model to ```NSManagedObject```

#### Approach #1: CoreDataWorker per Entity - NSManagedObject pair

Enough talking. Time to see some code! 

**NOTE: CoreDataWorker uses CoreDataService from [previous post]((https://swifting.io/blog/2016/09/25/25-core-data-in-ios10-nspersistentcontainer?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)) as a dependency** 

There you go:

```
//#1
protocol CoreDataWorkerProtocol {
    associatedtype EntityType

    func get(with predicate: Predicate?, sortDescriptors: [SortDescriptor]?,
    completion: @escaping (Result<[EntityType]>) -> Void)
    
    //...
}

//#2
class CoreDataWorker<ManagedEntity, Entity>: CoreDataWorkerProtocol where
    ManagedEntity: NSManagedObject,
    ManagedEntity: ManagedObjectProtocol,
	Entity: ManagedObjectConvertible  {

//#3
    let coreData: CoreDataServiceProtocol
    init(coreData: CoreDataServiceProtocol = CoreDataService.shared) {
        self.coreData = coreData
    }
    
    func get(with predicate: Predicate? = nil, sortDescriptors: [SortDescriptor]? = nil, fetchLimit: Int? = nil,
             completion: @escaping (Result<[Entity]>) -> Void) {
             
//#4
        coreData.performForegroundTask { (context) in
            do {
                let fetchRequest = ManagedEntity.fetchRequest()
                fetchRequest.predicate = predicate
                fetchRequest.sortDescriptors = sortDescriptors
                if let fetchLimit = fetchLimit {
                    fetchRequest.fetchLimit = fetchLimit
                }
                let results = try context.fetch(fetchRequest) as? [ManagedEntity]
//#5
                let items: [Entity] = results?.flatMap { $0.toEntity() as? Entity } ?? []
//#6
                completion(.success(items))
            } catch {
//#7
                let fetchError = CoreDataWorkerError.cannotFetch("Cannot fetch error: \(error))")
                completion(.failure(fetchError))
            }
        }
    }
   // other methods: upsert, remove, ...
   
}
```

For simplicity we have stared with get method. Other actions are very similar. 

What's up there? Let's go step by step through it:
##### #1 
This worker is a crucial part of our app. We want it and components that depend on it tested. That is why we have a protocol for it. The `associatedtype` allows us to use a generic types as parameters. ```Result<T>``` is a simple enum that can be either ```.successs(T)``` or ```.failure(Error)```.

##### #2
In ```CoreDataWorker``` we are defining generic types for Entities and ```NSManagedObject```s with conditions that they have to meet. In this way we ensure that ```ManagedObject``` is indeed a CoreData object and that two way conversion will be possible.

##### #3
Here we inject ```CoreDataService``` created in [previous post](https://swifting.io/blog/2016/09/25/25-core-data-in-ios10-nspersistentcontainer?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post). It is responsible for running foreground or background tasks and provides CoreData contexts.

##### #4
This starts CoreData context. All further actions are embedded into closure to ensure proper thread use. As we are returning simple structs here we will be safe to use them later on any thread.

##### #5
That's where all the magic kicks in. With Swift's ```flatMap``` we are converting each ```NSManagedObject``` to its corresponding model struct. 

##### #6
Here, after successful operation we return converted model structs ready to use in further parts of app.

##### #7
Something may go wrong during fetch, so we can complete with failure and inform user what went wrong.


#### Approach #2: CoreDataWorker per any Entity - NSManagedObject pair   

Previous approach seems pretty neat. You have generic worker that can be used for any entity. But usually we operate on more than one entity in our code. So, this initial version usually forced us to define multiple ```CoreDataWorkers``` in one scope. 
Thanks to [higheror](https://twitter.com/higheror) from our team we have switched to more elegant version of CoreDataWorker. Now, just the methods of ```CoreDataWorker``` are generic, not the whole worker. In this way we can have one instance of worker reused for multiple entities.
Let's have a look:

```
protocol CoreDataWorkerProtocol {
    func get<Entity: ManagedObjectConvertible>
        (with predicate: NSPredicate?,
         sortDescriptors: [NSSortDescriptor]?,
         completion: @escaping (Result<[Entity]>) -> Void)
}

class CoreDataWorker: CoreDataWorkerProtocol {
    let coreData: CoreDataServiceProtocol

    init(coreData: CoreDataServiceProtocol = CoreDataService.shared) {
        self.coreData = coreData
    }

    func get<Entity: ManagedObjectConvertible>
        (with predicate: NSPredicate? = nil,
         sortDescriptors: [NSSortDescriptor]? = nil,
         fetchLimit: Int? = nil,
         completion: @escaping (Result<[Entity]>) -> Void) {
        coreData.performForegroundTask { context in
            do {
                let fetchRequest = Entity.ManagedObject.fetchRequest()
                fetchRequest.predicate = predicate
                fetchRequest.sortDescriptors = sortDescriptors
                if let fetchLimit = fetchLimit {
                    fetchRequest.fetchLimit = fetchLimit
                }
                let results = try context.fetch(fetchRequest) as? [Entity.ManagedObject]
                let items: [Entity] = results?.flatMap { $0.toEntity() as? Entity } ?? []
                completion(.success(items))
            } catch {
                let fetchError = CoreDataWorkerError.cannotFetch("Cannot fetch error: \(error))")
                completion(.failure(fetchError))
            }
        }
    }
}
```
As you can see most of the code is pretty much the same. It is just a place where generics are declared that makes so much difference in the end result.

At this point, I owe you one explanation. What the heck is ```Entity.ManagedObject``` ? Read on 😁.

#### Converting entities with ```ManagedObjectConvertible``` and ```ManagedObjectProtocol``` protocols

These are two simple protocols that force Entity and ```NSManagedObject``` pair to implement conversion methods.

```
protocol ManagedObjectProtocol {
	associatedtype Entity
	func toEntity() -> Entity?
}
```
```ManagedObjectProtocol``` is implemented by ```NSManagedObject``` to convert it to simple model struct.

```
protocol ManagedObjectConvertible {
	associatedtype ManagedObject
	func toManagedObject(in context: NSManagedObjectContext) -> ManagedObject?
}
```

```ManagedObjectConvertible``` is implemented by model struct to convert it to ```NSManagedObject``` in a given context.
Note the `associatedtype` here and type of ```Entity``` generic parameter in get method of second approach. This ```ManagedObject``` associatedtype is accessible from Entity in form of ```Entity.ManagedObject``` and gives the type to fetch on.

#### Conversion protocols in practice
How this would look like in practice? 
We will define simple user struct and corresponding ```NSManagedObject```. User struct could look as follows:

```
struct User {
    let id: String
    let username: String?
    var name: String?
    var birthday: Date?
}

extension User: ManagedObjectConvertible {
    func toManagedObject(in context: NSManagedObjectContext) -> UserMO? {
        guard let user = UserMO.getOrCreateSingle(with: id, from: context)
        else { return nil }

        user.identifier = id
        user.username = username
        user.name = name
        user.birthday = birthday as NSDate?
        return user
	}
```

```NSManagedObject```s are generated automatically, but we still have to extend them with ```ManagedObjectProtocol``` to enable two-way conversion:

```
// Managed object properties & conversion
extension UserMO {
	// CoreData autogenerated
    @nonobjc public class func fetchRequest() -> NSFetchRequest<UserMO> {
        return NSFetchRequest<UserMO>(entityName: "UserMO")
    }
    @NSManaged public var identifier: String
    @NSManaged public var username: String?
    @NSManaged public var name: String?
    @NSManaged public var birthday: NSDate?
}

extension UserMO: ManagedObjectProtocol {
	func toEntity() -> User? {
		return User(id: identifier,
                    username: username,
                    name: name,
                    birthday: birthday as? Date)
    }
}
``` 

#### Ready..., set..., fetch!
Finally we would like to see our ```CoreDataWorker``` in action, right?

For the first approach you initialize your worker as follows:

```
let worker: CoreDataWorker<UserMO, User> = CoreDataWorker<UserMO, User>()

```


...and use it like that:

```
func fetchUser(completion: @escaping (User?) -> Void) {
        worker.get{ [weak self](result: Result<[User]>) in
            switch result {
            case .success(let users):
                self?.currentUser = users.first
                completion(users.first)
            case .failure(let error):
                print("\(error)")
                completion(nil)
            }
        }
    }
```

To fetch data for another entity, say ```AccountMO``` you would need another worker typed ```CoreDataWorker<AccountMO, Account>```.

With the second approach you just need one worker:

```
let worker: CoreDataWorkerProtocol = CoreDataWorker()
```

The ```fetchUser``` call is exactly the same, but thanks to neat trick with type inference in this new worker, it knows by itself that for ```User``` it should fetch ```UserMO``` from CoreData.

Your eagle eye, has probably cought the protocol in second worker declaration. This is additional bonus that facilitates mocking in unit testing. You cannot have it in first example as we have there an abstract protocol, which cannot be used as variable type.

If you try to declare first worker not as ```CoreDataWorker<UserMO,User>``` but ```CoreDataWorkerProtocol```, this is what happens:
>Protocol 'CoreDataWorkerProtocol' can only be used as a generic constraint because it has Self or associated type requirements.

Hungry for more?
Take a look at our [github repository](https://github.com/swiftingio/BetterSwifterCoreData?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) and see both approaches in action.


#### Summary 
There are as many supporters as opponents of using Core Data's ```NSManagedObject```'s throughout all layers of project. The approach with structs also has it's strengths and weaknesses. 

We get immutability which is great (especially in big projects) and more importantly we have thread safety ensured. We have tried the approach with struct models also in or VIPER project. 

On the con side there is slightly more work to manage additional layer and less flexibility in retrieving relations especially in complex graphs.

For us this extra work is worth it, especially that we run our projects with ```-com.apple.CoreData.ConcurrencyDebug 1``` flag on and haven't seen any CoreData crashes. I hope one day CoreData will become more Swifty 😉. 


We are still eager to improve and fine-tune our approach, tell us what you think 🤓.


#### References
 
- [Swifting.io sample app with CoreDataWorkers](https://github.com/swiftingio/BetterSwifterCoreData?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)
- [Realm - Managing Consistency of Immutable Models](https://realm.io/news/slug-peter-livesey-managing-consistency-immutable-models/)
- [Pinterest - immutable models and data consistency
](https://engineering.pinterest.com/blog/immutable-models-and-data-consistency-our-ios-app?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

There are also some out of the box solutions like [CoreValue](https://github.com/terhechte/CoreValue?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) or [CoreStore](https://github.com/JohnEstropia/CoreStore?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) - promising but haven't tried them yet. They may offer some good solutions.
