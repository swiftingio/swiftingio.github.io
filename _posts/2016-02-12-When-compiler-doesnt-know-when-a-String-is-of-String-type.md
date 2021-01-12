---
layout: post
author: maciej
title: \#5 When compiler doesn’t know when a String is of String type
excerpt: 
---
#### String initialised with a class
Did you know that Swift 2.1 introduced new syntax for initialisation that works more or less like ```NSStringFromClass([Object class])``` in Objective-C? From now on, you can initialise String in the following way:

```Swift
let className = String(Object)
```

I dug the documentation and I found out that there is no initialiser that takes a class as an init argument. Because of that I'm not sure how it works under the hood but my guess would be that Swift compiler makes some magic tricks to make that work. If anyone has some thoughts - please, share them with us :)!

#### Where this 'trick' can be used?
Some time ago I've seen a nice blog entry from [NatashaTheRobot](https://www.natashatherobot.com/nsstringfromclass-in-swift/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post). It shows a clever way to create a cell reusable identifiers from class name.

```Swift
let reuseIdentifier = String(MyTableViewCell)
```

#### Usage in dictionaries
Recently I had to create a dictionary that would return a specific string based on a class name it holds. It was used in ```UIViewController``` and its subclasses for configuration purposes in ```viewDidLoad``` method. It looked similarly to this snippet:

```Swift
let dictionary = [
	String(Class1) : "Custom string for class 1",
	...
	String(ClassN) : "Custom string for class n"
]
```

I thought it should be enough, however I got a fancy message from the compiler: ```Expression was too complex to be solved in reasonable time; consider breaking up the expression into distinct sub-expressions```.

![Compiler Error](https://raw.githubusercontent.com/swiftingio/blog/%235-When-compiler-doesn't-know-when-a-String-is-of-String-type/%235%20When%20compiler%20doesn't%20know%20when%20a%20String%20is%20of%20String%20type/Compiler%20message.jpg?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) 

#### Solving issues
Resolution of a problem is pretty simple. From time to time one has to help compiler a bit by explicitly defining expression result type. By adding a 'casting' to String the problem was solved. Thanks to [@higheror](https://twitter.com/higheror?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) for helping out with this one! :)

```Swift
let dictionary = [
	String(Class1) as String : "Custom string for class 1",
	...
	String(ClassN) as String : "Custom string for class n"
]
```

I had a good few entries in aforementioned dictionary, hence manual addition of ```as String``` would be painful. Thanks god we can use (masked) regular expressions in Xcode's  find & replace text option (```CMD+ALT+F```).

PS. You can also have a look at [@krzyzanowskim](https://twitter.com/krzyzanowskim?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)'s post about type ambiguity when overloading methods. Grab it [here](http://blog.krzyzanowskim.com/2016/02/07/overload-swift-ambiguity-define-default-type/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post).
