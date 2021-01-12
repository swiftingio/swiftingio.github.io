---
layout: post
author: michal
title: \#22 Swift 3 Access Control (Xcode 8 Beta 6)
excerpt: 
---

On August 15th 2016, Xcode 8 Beta 6 was released and brought some significant changes to Swift Access Control and other parts of language.

#### Swift 2.2 Access Control

We have been used to ```public```, ```internal``` and ```private``` access levels:

- ```public``` gives access from any source file in given module or in different module that imported the defining module. In short - good fit for public framework interface.

- ```internal``` restricts access to files from defining module.

- ```private``` restricts access to defining file.


#### Swift 3 Beta Access Control

##### Stricter private and a new fileprivate

If you look into the first bullet in release notes, you will see:


>A declaration marked as private can now only be accessed within the lexical scope it is declared in (essentially the enclosing curly braces {}). A private declaration at the top level of a file can be accessed anywhere in that file, as in Swift 2. The access level formerly known as private is now called fileprivate. (SE-0025) 


**This is big** 
Usually we have some ```private``` vars or methods that are accessed in extensions in the same sourcefile. This will no longer work. You will need to use ```fileprivate``` access level to make your code compile. The new levels name looks weird but it was proposed by Chris Lattner himself.

##### Stricter public and liberal open

This one will be interesting for you if you use frameworks and subclass/override their methods. Now if a class is not declared as ```open``` you will not be able to subclass it outside defining module. For methods this means no overriding outside its defining module.

Full release note on the topic:

>Classes declared as public can no longer be subclassed outside of their defining module, and methods declared as public can no longer be overridden outside of their defining module. To allow a class to be externally subclassed or a method to be externally overridden, declare them as open, which is a new access level beyond public. Imported Objective-C classes and methods are now all imported as open rather than public. Unit tests that import a module using an @testable import will still be allowed to subclass public or internal classes as well as override public or internal methods. (SE-0117) 


#### TL;DR
![Swift 3 Access Control](https://raw.githubusercontent.com/swiftingio/blog/%2321-Swift3-Access-Control/%2321%20Swift3%20Access%20Control/Swift3AcccesControl.png)

#### Summary & References
New Beta 6 brought significant changes to Access Control. For those developing in Beta this means even a couple of hours of refactoring. There were many more surprises such as ubiquitous change from ```AnyObject``` to ```Any``` and making non-escaping a default option for closure parameters. 


- [Xcode 8 Beta 6 Release Notes (works after login)](http://adcdownload.apple.com/Developer_Tools/Xcode_8_beta_6/Release_Notes_for_Xcode_8_beta_6.pdf?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)
- [Swift 2.2 Access Control](https://developer.apple.com/library/ios/documentation/Swift/Conceptual/Swift_Programming_Language/AccessControl.html?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)
- [SE-0025 Scoped Access Level proposal](https://github.com/apple/swift-evolution/blob/master/proposals/0025-scoped-access-level.md?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)
- [SE-0117 Public access and public overridability proposal](https://github.com/apple/swift-evolution/blob/master/proposals/0117-non-public-subclassable-by-default.md?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)
