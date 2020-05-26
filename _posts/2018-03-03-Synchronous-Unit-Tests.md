---
layout: post
author: maciej
title: \#50 Synchronous Unit Tests
excerpt: Mobile applications are usually multi-threaded. We perform UI operations on the main thread and dispatch heavy tasks (e.g. network requests, JSON parsing, writing to a file on a disk) on background threads. The iOS allows us to use backgrounds threads for example by using Grand Central Dispatch API (GCD), i.e. by performing operations on `DispatchQueue` objects. Work dispatched to a background `DispatchQueue` is usually done asynchronously with `queue.async{}` call.
---
#### Asynchronous Expectations

Mobile applications are usually multi-threaded. We perform UI operations on the main thread and dispatch heavy tasks (e.g. network requests, JSON parsing, writing to a file on a disk) on background threads. The iOS allows us to use backgrounds threads for example by using Grand Central Dispatch API (GCD), i.e. by performing operations on `DispatchQueue` objects. Work dispatched to a background `DispatchQueue` is usually done asynchronously with `queue.async{}` call.

If we wanted to test an object that uses a queue to perform work in background, we would use an `XCTestExpectation` and wait for an async operation to finish. We can fulfil the expectation in a callback.


```Swift
func testCompletionGetsAtLeast1Message() {

	//Arrange
	let laoder = MessageLoader()
	let expectation = XCTestExpectation(description: "should call completion handler with at least 1 message")

	//Act
	laoder.load { messages in
		//Assert
		XCTAssertFalse(messages.isEmpty)
		expectation.fulfill()
	}
	
	wait(for: [expectation], timeout: 5)
}
```

In autumn I asked on Twitter about the best way to wait for an `XCTestExpectation`. The interface of `XCTestCase` class declares a few methods:

![](https://raw.githubusercontent.com/swiftingio/blog/synchronous-unit-testing/twitter1.png?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

I didn't get a clear answer, but the reply gave food for thought...

![](https://raw.githubusercontent.com/swiftingio/blog/synchronous-unit-testing/twitter2.png?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)


#### Asynchronous drawbacks
##### Unclear AAA pattern
It's a good practice to divide test into 3 phases:
- **Arrange** - setup a subject under test, set it in the desired state 
- **Act** - perform an action on the subject
- **Assert** - test if a desired behaviour or change happened 

In an asynchronous test the phases aren't clearly visible. How would you name the last phase? A **Wait** phase?

```Swift
func testCompletionGetsAtLeast1Message() {

	//Arrange
	let laoder = MessageLoader()
	let expectation = XCTestExpectation(description: "should call completion handler with at least 1 message")

	//Act
	laoder.load { messages in
		//Assert
		XCTAssertFalse(messages.isEmpty)
		expectation.fulfill()
	}
	
	//???
	wait(for: [expectation], timeout: 5)
}
```

##### Overhead
In the code snippet above we wait 5 seconds for the expectation. It means that our unit test can take up to 5 seconds in the worst case scenario (i.e. if execution of `load(:)` method took more that 5 seconds). `XCTest` framework would fail the test after 5 seconds of waiting.

```Swift
    wait(for: [expectation], timeout: 5)
```

Waiting for 5 seconds for a unit test seems too much... Even 1 second seems! Imagine we had 1200 unit tests with asynchronous expectations fulfilment and because of some mistake in our code all of them fail. The framework needs to wait for each of them for 1s. Math is like that:

```
1s * 1200 tests = 1200s = 20min
```

We would have to wait 20 minutes for results of our unit test suite execution. Feels like eternity... Even Swift apps compile faster nowadays... 😉

```
    wait(for: [expectation], timeout: 0.1)
```

Typically the waiting time should be set to 100ms. But even 100ms is a bit too much. In the case of failing tests we would have to wait for 2 minutes...

```
100ms * 1200 tests = 120s = 2 minutes
```

##### No control
But the  main drawback is that we have **NO CONTROL** over `DispatchQueue.main` singleton used. We don't know how much time the execution of our code will take. But we can regain the control. How? By running code **synchronously**!

#### Synchronous Assertions

Let's assume we have a `MessageLoader` class that performs some operations on a background `DispatchQueue`:

```Swift
class MessageLoader {
    let queue: DispatchQueue
    
    init(queue: DispatchQueue) {
        self.queue = queue
    }
    
    func load(_ completion: @escaping ([Message])->Void) {
        queue.async {
						var messages: [Message] = []
						//NOTE: fetches messages in background
						completion(messages)
        }
    }
}
```

In this approach we cannot test in other way than using an `XCTestExpectation` that `completion` closure gets called after our function finishes doing stuff.

We can create a `Dispatching` protocol that declares a single method - `dispatch(:)`.

```Swift
protocol Dispatching {
    func dispatch(_ work: @escaping ()->Void)
}
```

Let's create a `Dispatcher` class that gets initialised with a queue. It will serve as a superclass for other dispatchers.

```Swift
class Dispatcher {
    let queue: DispatchQueue
    
    init(queue: DispatchQueue) {
        self.queue = queue
    }
}
```

We can hide async dispatch of a job to a `DispatchQueue` by creating an async dispatcher - let's call it `AsyncQueue`. The class conforms to our `Dispatching` protocol and performs action asynchronously on a queue that an instance is initialised with.

```Swift
class AsyncQueue: Dispatcher {} //inheritance gives an initialiser with a queue

extension AsyncQueue: Dispatching {
    func dispatch(_ work: @escaping ()->Void) {
        queue.async(execute: work) //IMPORTANT!
    }
}
```

We can also create a `SyncQueue` that would dispatch a job synchronously on a queue.

```Swift
class SyncQueue: Dispatcher {} //inheritance gives an initialiser with a queue

extension SyncQueue: Dispatching {
    func dispatch(_ work: @escaping ()->Void) {
        queue.sync(execute: work) //IMPORTANT!
    }
}
```

Ok, but what is this all for? We wanted to test synchronously! So we need to upgrade our `MessageLoader` to use the `Dispatching` queue to perform a job.

```Swift
class MessageLoader {
    let queue: Dispatching
    
    init(queue: Dispatching) {
        self.queue = queue
    }
    
    func load(_ callback: @escaping([Message]) -> Void) {
        queue.dispatch { //NEW!
            var messages: [Message] = []
            //TODO: fetch messages in background
            completion(messages)
        }
    }
}
```

Instead of using `sync` or `async` method on a `DispatchQueue` we call `dispatch` on our `Dispatching` type. How to test synchronously that `completion` closure gets called? We just need to initialise `MessageLoader` with a `SyncQueue`. 

Before that, let's create some queues just like `DispatchQueue` gives access to commonly used queues:

```Swift
extension SyncQueue {
    static let main: SyncQueue = SyncQueue(queue: .main)
    static let global: SyncQueue = SyncQueue(queue: .global())
    static let background: SyncQueue = SyncQueue(queue: .global(qos: .background))
}

extension AsyncQueue {
    static let main: AsyncQueue = AsyncQueue(queue: .main)
    static let global: AsyncQueue = AsyncQueue(queue: .global())
    static let background: AsyncQueue = AsyncQueue(queue: .global(qos: .background))
}
```

Let's write a unit test for our `completion` closure being called. We need to initialise `MessageLoader` with a background queue on which jobs are dispatched synchronously. Imagine we are writing a messaging app. Our product manager gave us a task to welcome a user with a "hello" message even if there are no other messages.

```Swift
let welcome = Message(author: "The App", text: "Welcome in the app!") 
```

So we need to assert that we have at least one message in the array given as an argument to the completion handler. We can create an array outside a completion closure and assign a value to it in the completion.

```Swift
func testAtLeast1MessageOnLoad() {

	//Arrange
	var messages: [Message] = []
	let background = SyncQueue.background
	let loader = MessageLoader()
	loader.queue = background
               
	//Act
	loader.load { fetched in
		messages = fetched
	}
	
	//Assert
	XCTAssertFalse(messages.isEmpty)
}
```

Thanks to the synchronous nature of this testing approach we also benefit from it by enhancing test readability. We have clearly visible **Arrange**, **Act** and **Assert** phases.

#### TL;DR;(1) - making things clear

We don't have to use the `XCTestExpectation` because our code executes synchronously with `SyncQueue`. Unit test is run on the main thread and then `load(:)` executes it's job synchronously on a background thread. The `sync` dispatch means, that the calling thread waits until execution of the job dispatched on a background thread finishes.

![](https://raw.githubusercontent.com/swiftingio/blog/synchronous-unit-testing/diagram1.png?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

#### Beware of the 🐕 deadlock!

OK, let's refactor. Or make things more difficult... But look out for a deadlock! 

Imagine we have a `loader` that dispatches work on a background queue but calls `completion` closure on the main queue. If we used `DispatchQueue` objects our code would look like this:

```Swift
func load(_ completion: @escaping ([Message])->Void) {
    
    DispatchQueue.global().async {
        //fetch messages on a background thread
        
        DispatchQueue.main.async {
            completion([ ]) //on the main thread
        }
        
    }
 }
```

Again, as in previous example, we can use the `Dispatching` protocol to hide a use of a `DispatchQueue`.

```Swift
class MessageLoader {
    let main: Dispatching
    let background: Dispatching
    
    init(main: Dispatching, background: Dispatching) {
        self.main = main
        self.background = background
    }
    
    func load(_ callback: @escaping([Message]) -> Void) {
    
        background.dispatch { //NEW!
            
            var messages = [ Message.welcome ]    
                    
            //TODO: fetch messages in background
            
            self.main.dispatch { //NEW!
                callback(messages)
            }
        }
    }
}
```


If we wanted to test that `completion` closure gets called and we didn't pay attention to what queue we dispatch jobs during a unit test we might end up with a deadlock.

What is the [deadlock](https://en.wikipedia.org/wiki/Deadlock?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)? In a multi-threaded application it occurs when a thread enters a waiting state because a requested system resource is held by another waiting thread.

We want to test our new `loader` with a simple test that calls `load(:)` method on `MessageLoader` instance. We initialise our `loader` with `SyncQueue.main` and `SyncQueue.background` objects that perform work synchronously on the main queue and on a background queue respectively.

```Swift
func testAtLeast1MessageOnLoad() {

	//Arrange
	var messages: [Message] = []
	let loader = MessageLoader(main: SyncQueue.main,
	                           background: SyncQueue.background)
               
	//Act
	loader.load { fetched in
		messages = fetched
	}
	
	//Assert
	XCTAssertFalse(messages.isEmpty)
}
```

Just to remind you - unit test is run on the main thread and then `load(:)` executes it's job synchronously on a background thread. When the job on the background thread finishes the method dispatches work synchronously to the main thread to call `completion` closure. Do you see an issue with this approach? 

A **synchronous** dispatch means that a dispatching thread waits for a work to be finished on a thread it dispatches the job to. In the `load(:)` we dispatch work synchronously from the main thread to a background thread and then we perform another synchronous dispatch to the main thread. Because the previous synchronous dispatch is not finished and we try to do another synchronous dispatch to the same queue we end up with a deadlock. 

![](https://raw.githubusercontent.com/swiftingio/blog/synchronous-unit-testing/diagram2.png?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

How can we fix that? We need to use a different queue than `main`, e.g. `SyncQueue.global` 😉.

```Swift
	let loader = MessageLoader(main: SyncQueue.global,
	                           background: SyncQueue.background)

```

And now we use threads as following and no deadlock occurs:

![](https://raw.githubusercontent.com/swiftingio/blog/synchronous-unit-testing/diagram3.png?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)


#### Summary a.k.a TL;DR;(2)

Asynchronous tests are unreadable, might give an overhead to test execution and we don't have full control over them.

Using a protocol to "hide" dispatching jobs to different threads allows running tests synchronously. It gives us **FULL CONTROL** over unit tests execution and prevents us from waiting for too long in the case of their failure. We can also clearly see the **Arrange**, **Act** and **Assert** phases of a unit test.

You can check our sample code on [GitHub](https://github.com/swiftingio/sync-unit-testing-50?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post).

Special thanks to [Paweł Dudek](https://twitter.com/eldudi) for a review!

#### Links
- [Sample code on GitHub](https://github.com/swiftingio/sync-unit-testing-50?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)
- [Unit testing asynchronous Swift code](https://www.swiftbysundell.com/posts/unit-testing-asynchronous-swift-code?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)
- [Unit testing on swifting.io](https://swifting.io/blog/category/unit-testing/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)
- [Deadlock](https://en.wikipedia.org/wiki/Deadlock?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)


