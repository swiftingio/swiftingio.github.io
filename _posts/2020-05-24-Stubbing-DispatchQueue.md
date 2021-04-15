---
layout: post
author: maciej
title: \#54 Stubbing DispatchQueue in unit tests to run tests synchronously
excerpt: In issue \#50 Synchronous Unit Tests I described an approach on how to get rid of waiting for `XCTestExpectation` to be **fulfill**ed. The approach assumes hiding an asynchronous dispatch of a block/closure of code to a **GCD** queue behind an abstraction layer called the `Dispatching` protocol.

---

#### The `Dispatching` protocol

In issue [\#50 Synchronous Unit Tests](https://swifting.io/blog/2018/03/03/50-synchronous-unit-tests/) I described an approach on how to get rid of waiting for `XCTestExpectation` to be **fulfill**ed. The approach assumes hiding an asynchronous dispatch of a block/closure of code to a **GCD** queue behind an abstraction layer called the `Dispatching` protocol.

```
protocol Dispatching {
    func dispatch(_ work: @escaping ()->Void)
}
```

In the production code you would implement a `Dispatcher` type like this:

```
class Dispatcher: Dispatching {

    let queue: DispatchQueue
    
    init(queue: DispatchQueue) {
        self.queue = queue
    }
    
    func dispatch(_ work: @escaping ()->Void) {
        queue.async(execute: work) // 👈 This runs your code asynchronously on the queue
    }
}
```


If you had e.g. a view model which needed to perform some asynchronous computations instead of calling `DispatchQueue.global().async { /* your code */ }` you inject an instance of the `Dispatcher` and use its `dispatch(:)` method:

```
class MessagesViewModel {
    let dispatcher: Dispatching
    
    init(dispatcher: Dispatching) {
        self.dispatcher = dispatcher
    }
    
    func load(_ callback: @escaping([Message]) -> Void) {
        dispatcher.dispatch { //NEW!
            var messages: [Message] = []
            //TODO: fetch messages in background
            completion(messages)
        }
    }
}
```

#### Running unit tests synchronously - a simpler way

In the previous post I suggested using a synchronous dispatch to a queue during a unit test run. Actually, there is a simpler approach. Thanks to the abstraction layer of `Dispatching` protocol you can create a **stub** which invokes your block of code instantly. It doesn't have to use `DispatchQueue` type at all, as it was suggested in the previous post ([\#50 Synchronous Unit Tests](https://swifting.io/blog/2018/03/03/50-synchronous-unit-tests/)).


```
class DispatcherStub: Dispatching {
    func dispatch(_ work: @escaping ()->Void) {
        work()
    }
}
```

In a unit test case you create an instance of the stub and the code runs synchronously :)!

```
func testAtLeast1MessageOnLoad() {
 
    //Arrange
    var messages: [Message] = []
    let dispatcher = DispatcherStub()
    let viewModel = MessagesViewModel(dispatcher: dispatcher)
               
    //Act
    viewModel.load { fetched in
        messages = fetched
    }
    
    //Assert
    XCTAssertFalse(messages.isEmpty)
}
```
