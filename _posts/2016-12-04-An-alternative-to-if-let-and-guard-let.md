---
layout: post
author: maciej
title: \#29 An alternative to if let and guard let
excerpt: 
---
It's brilliant when you can work with people smarter (or even a lot smarter) than you! You can learn so much from them. And then write about the things you've learned 🙃. 

#### if and guard let

Have you ever been tired of repeating `if let` or `guard let` statements, to perform certain operations, that should be performed only if a value is not nil? From time to time your code could become like this if you wanted to call a function with unwrapped value:

```
if let unwrapped1 = optional1 {
	doTaskThatReturnsVoid(with: unwrapped1)
}
if let unwrapped2 = optional2 {
	doTaskThatReturnsVoid(with: unwrapped2)
}
//...
if let unwrappedX = optionalX {
	doTaskThatReturnsVoid(with: unwrappedX)
}
```

or like this:

```
struct Bar {
	let name: String
	let error: Error?
}

func foo(with bars: [Bar]) {
	var errors: [Error] = []
	bars.forEach {
		guard let error = $0.error else { return }
		errors.append(error)
	}
	process(errors)
}
```

Sometimes, it's too much typing, isn't it? But there's an alternative ...

#### flatMapping from T→Void

Optionals are monads, on which you can call `flatMap` function, that unwraps an optional (of hypothetical type A, a.k.a  `Optional<A>`) if it contains a value, applies to it a transform, that converts a value from type A to type B and wraps it back into an optional of type B (a.k.a. `Optional<B>`). PS. if you want to understand functors, applicatives and monads I strongly recommend [this video](https://realm.io/news/slug-andy-bartholomew-understand-monads-one-weird-trick/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post). 

So, if we have an optional, we can call `flatMap` on it. We have to pass a closure transforming from A→B to the function. What if we wanted to transform from `T→Void`?

It would work. According to Swift compiler, transformation from `T` to nothing (a.k.a. `Void`) is a valid transformation. So let's use that fact!

```
let optional1 = getOptional1()
optional1.flatMap { self.doTaskThatReturnsVoid(with: $0) }
```

Note that we don't have to use a capture list (see more on them [here](https://swifting.io/blog/2016/02/28/7-do-i-love-or-crash-something-shortly-on-capture-lists/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)) since `flatMap` takes a non-escaping closure as an argument. In my opinion the real beauty of the solution meets the eye here:

```
func foo(with bars: [Bar]) {
	var errors: [Error] = []
	bars.forEach {
		$0.error.flatMap { errors.append($0) }
	}
	process(errors)
}
```

Neat! We can append `error` to the array, only if it contains a value. We got rid of `guard let` similarly to `if let` in the previous example.

#### Drawbacks

Seeing code with transforms from `T→Void` in `flatMap` might seem a bit odd at first glance. It was odd for me. But once I've seen it used multiple times in the code that would require a few `if lets` in a row I started appreciating the solution. Check it out, maybe you will start using it too! :)

#### Final words

Big thanks to people I work with for the things I've been learning from you ❤️!

#### References

- [Understand Monads with this One Weird Trick](https://realm.io/news/slug-andy-bartholomew-understand-monads-one-weird-trick/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post).
- [Functors, Applicatives, and Monads in Plain English](http://www.russbishop.net/monoids-monads-and-functors?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

#### Update 4.12.2016 21:30

We've received a tip and correction from [Ole Begemann](https://twitter.com/olebegemann). Thank you Ole ❤️🙃!

There's a mistake in **flatMapping from T→Void** paragraph. It actually describes `map` instead of `flatMap` function. The `map` function takes a `transform` closure from `T→U`, whereas `flatMap` from `T→U?`. You can see declarations below:

```
public func map<U>(_ transform: (Wrapped) throws -> U) rethrows -> U?
public func flatMap<U>(_ transform: (Wrapped) throws -> U?) rethrows -> U?
```

So actually `map` and `flatMap` would have the same effect in our scenario. You can check this snippet in a **Playground** to see how it works!

```
import Foundation

enum Error: Swift.Error {
    case small
    case big
}

struct Bar {
    let name: String
    let error: Error?
}

let bars = [
    Bar(name: "1", error: Error.small),
    Bar(name: "2", error: nil),
    Bar(name: "3", error: Error.big),
]

func fooFlatMap(with bars: [Bar]) {
    var errors: [Error] = []
    bars.forEach {
        $0.error.flatMap { errors.append($0) }
    }
    print("\(errors)")
}

func fooMap(with bars: [Bar]) {
    var errors: [Error] = []
    bars.forEach {
        $0.error.map { errors.append($0) }
    }
    print("\(errors)")
}

fooFlatMap(with: bars)
fooMap(with: bars)
```

#### Update 5.12.2016 9:30

[@rosskimes](https://twitter.com/rosskimes/status/805636719810899968) had a good thought - `forEach` in the example can actually be replaced by `let errors: [Error] = bars.flatMap { $0.error }`. So, let us show you a more convincing example.

In snippet below `CoreDataWorker` mentioned in [issue #28](https://swifting.io/blog/2016/11/27/28-better-coredata-with-swift-generics/) is used to remove all entities from the database. A `dispatchGroup` synchronizes delete operations. All potential `errors` are appended to an array and processed when all operations are finished.

```
func removeData() {
	let dispatchGroup = DispatchGroup()
	
	var errors: [Error] = []
	
	let completion = { (error: Error?) in
	    error.flatMap { errors.append($0) }
	    dispatchGroup.leave()
	}
	
	dispatchGroup.enter()
	coreDataWorker.removeAllEntitiesOfType(Entity1.self, completion: completion)
	
	dispatchGroup.enter()
	coreDataWorker.removeAllEntitiesOfType(Entity2.self, completion: completion)
	//...
	
	dispatchGroup.notify(queue: DispatchQueue.main) {
	    guard errors.isEmpty else { process(errors); return }
	    //success, proceed with further operations
	}
}
```
