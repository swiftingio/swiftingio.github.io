---
layout: post
author: michal
title: \#43 Bye Bye fileprivate
excerpt: 
---
Just one week left till WWDC 2017! I am expecting much more power given to developers in SiriKit and a couple of surprises. I guess you are waiting for some new APIs too 😬.

The new Apple developer season is not only about APIs, OSes but also about Swift update. Swift 4 is coming. There are no big breaking changes, backwards compatibility will be ensured. Some neat things there. [Ole Begemann](https://oleb.net?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) has created a [neat playground](https://github.com/ole/whats-new-in-swift-4?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) with all news.

#### `private` gets boost in Swift 4

Among all new features, one thing came particularly to my interest - visibility of `private` declaration in extensions in the same file (SE-0169). 

About a year ago, in [Issue #22](https://swifting.io/blog/2016/08/17/22-swift-3-access-control-beta-6/)
we have discussed, and complained a bit about the new `fileprivate`. As much as splitting implementation to extensions is considered a good practice, `fileprivate` was initially intended to be used rarely. In reality it was used a lot. We've had to update our code in around 200 places 😅. 

But this will change with Swift 4. `private` declaration will be accessible in type extensions in the same file. This change concerns only extensions; subclasses in the same files are not in scope.

#### `private` in Swift 3 vs. Swift 4

Take a look at this sample code:

```
// ListViewController.swift

class ListViewController: UIViewController {

	private let tableView = UITableView()
	private let items: [String] = ["Item 1", "Item 2", 	"Item 3"]

// ...
}

extension ListViewController: UITableViewDataSource {
	func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return items.count // Inaccessible in Swift 3
    }
    
    //...
}

```

This code is invalid in Swift 3 as `items` is not accessible in extension. In Swift 3 `items` would require `fileprivate` However in Swift 4 this will be valid.


#### So what happens with `fileprivate` now?

It will stay, but will be used as intended - rarely.
Take a look at the following example:

```
struct A {
    fileprivate var a: Int = 1
    private var b: Int = 2
}

struct B {
    private var c: A = A()
    
}

extension B {
    func foo(){
        print(c.a) //accessible
        //print(c.b) //💥inaccessible
    }
}

extension A {
    func boo(){
        print(a) //accessible
        print(b) //accessible
    }
}

B().foo()
A().boo()
```

Property `a` is marked with `fileprivate` so it will be visible even for other types within scope of this file. 
However property `b` is marked with `private` so it remains restricted to its owning type extension.

#### Swift 4 access control diagram?

Well, our previous diagram remains valid. Just a bump in language version:

![](https://raw.githubusercontent.com/swiftingio/blog/%2343-bye-bye-fileprivate/%2343%20Swift%204%20Access%20Control.png)



#### Summary & References

If SE-0169 makes it, `fileprivate` will have less prominent role. Luckily your Swift 3 code will continue to work, as `fileprivate` is now just a bit less restrictive than `private`. For me, in practice this means I will probably forget about `fileprivate` existence.

- [SE-0169](https://github.com/apple/swift-evolution/blob/master/proposals/0169-improve-interaction-between-private-declarations-and-extensions.md?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)
- [What's new in Swift 4 playground](https://github.com/ole/whats-new-in-swift-4?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)
- [Nice article about Swift 4 breaking changes by Bart den Hollander](http://blog.xebia.com/breaking-changes-swift-4?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)
