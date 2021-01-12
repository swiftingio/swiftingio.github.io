---
layout: post
author: maciej
title: \#7 Do I love or crash something? – shortly on capture lists
excerpt: 
---
#### I love pizza... or pasta... or it just crashes...
Some time ago I have attended an iOS meetup in Poznań, on which the attendants were given a short quiz during a break. We had to answer what would be printed in the console after code execution. One of the code snippets looked similarly to this one:

```Swift
var myLove = "pizza"
let iLove = { [myLove] in
    print(" I love \(myLove)")
}
myLove = "pasta"
iLove()
```

I have answered that the code would crash, since I have never ever seen  before a closure's capture list without any specifier, i.e. 'unowned' or 'weak'. Two people who solved correctly the whole 3-questions long quiz were supposed to get a fancy reward. I was sure the code would crash after execution of the snippet. In fact it cannot crash. The code is correct. Want to know what would be printed out to the console? Read further ;) !

#### This thing in a closure that starts and ends with a square bracket \[ \]... 
What a capture list is?  A closure can capture values from their surrounding context, i.e. an enclosing function, class, struct or any scope defined by curly braces \{ \}. Value is captured implicitly by using a variable defined in this surrounding context or explicitly by declaring a captured value in square brackets ```[myLove, myLover] in```. This declaration is called closure's *capture list*. Swift's capture lists are used to break strong reference cycles between a captured object and a closure.

#### Capturing a value: strong, weak and unowned reference
Value can be captured strongly by a closure, i.e. a strong reference to an object is created, like here:

```Swift
var capturedObject = MyObject()
let closure = {
	capturedObject.foo()
}	
```

If a closure was defined as a property of a class and if we used inside that closure another property and did not use a capture list,  a strong reference cycle would be created (you should rather avoid this).  This code leads to a memory leak:

```Swift
class Capturer {
	var capturedObject = MyObject()
	let closure = {
		capturedObject.foo()
	}	
}
```
	

To avoid a leak, value should by captured as ```weak``` or ```unowned``` in closure's capture list. A ```weak``` value is used inside a closure as an optional, whereas an ```unowned``` as an implicitly unwrapped optional (code crashes if value is nil). Beware!

```Swift
class Capturer {
	var capturedObject = MyObject()
	let closure = { [unowned capturedObject]
		capturedObject.foo()
	}	
	let closure = { [weak capturedObject]
		guard let capturedObject = capturedObject else { return }
		capturedObject.foo()
	}	
}
```

#### When a closure does not escape
Sometimes closure passed as a function's argument does not escape and a reference cycle between objects won't be created. When does it happen? For sure when the closure that is a function's  argument is tagged with the ```@noescape``` attribute. It assures that the closure won't be used anywhere else apart function's scope, so you can omit capture list.

Another interesting [information](https://developer.apple.com/library/ios/documentation/Swift/Conceptual/Swift_Programming_Language/Closures.html?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post#//apple_ref/doc/uid/TP40014097-CH11-ID94) I've found is that global functions do not capture any values. Thanks to that we can omit  for example ```[unowned self]``` in GCD dispatch calls (i.e. in ```dispatch_async()``` and ```dispatch_sync()``` and others).
		
When passing animation blocks to ```UIView.animateWithDuration()``` you can also omit capture list. Good point from [krakendev](http://krakendev.io/blog/weak-and-unowned-references-in-swift?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) that since ```animateWithDuration``` is a static method on ```UIView``` the closure passed that uses ```self``` does not have to be captured using a capture list.

#### Missing "in" keyword
Last week [Chris Eidhof](https://twitter.com/chriseidhof?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) mentioned in a [tweet](https://twitter.com/chriseidhof/status/700746170436419585?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) a missing 'in' keyword that caused compiler to treat ```[x]``` as an unused array literal. Remember about the ```in``` in closures (when needed of course) :)!

#### Ok, but what about this pizza thing?
I didn't win this fancy price. This really strange for me syntax was correct. Value captured in a closure without any specifier causes closure to store a copy of the value at the time of closure definition. Want to know what ```iLove()``` did? Check [this link](http://swiftlang.ng.bluemix.net/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post#/repl/44fbe4743c5d4ae591c4a81c7b693018781f84e2ce06783ab0cb07bd783d95c6) for the answer. BTW, What do you think about the [Bluemix](http://swiftlang.ng.bluemix.net?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)?

If you want to dig more on closures and retain cycles you can check the [documentation](https://developer.apple.com/library/ios/documentation/Swift/Conceptual/Swift_Programming_Language/Closures.html?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post#//apple_ref/doc/uid/TP40014097-CH11-ID94), [krakendev](http://krakendev.io/blog/weak-and-unowned-references-in-swift?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) and [digital leaves](http://digitalleaves.com/blog/2015/05/demystifying-retain-cycles-in-arc/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) blog posts.

And in fact I love both: pasta and pizza. And the code won't crash ;)!
