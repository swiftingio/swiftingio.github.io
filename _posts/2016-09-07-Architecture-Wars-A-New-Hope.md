---
layout: post
author: maciej
title: \#24 Architecture Wars – A New Hope
excerpt: 
---
A long time ago in a galaxy far, far away… Or maybe not that far away... 

Have you ever had a feeling, as I have had many times, that the design of your app seemed so brilliant at first, but suddenly adding more and more features to it made your source code more complicated and eventually, unmaintainable. Presumably you hadn’t written unit tests because there hadn’t been time for that.  You wish you hadn’t chosen standard MVC approach, but MVVM or VIPER instead, that you had heard so many times about. After all they’re so brilliant, so shiny and bright and give **A New Hope** to all developers. They're so cool and they're so many other things… Or are they?

#### What is this all about?
 
For a few years of my professional career I have been creating apps in different architectures (MVC, MVVM, VIP)  and have been searching for the best one. Recently at work, I have been having a roaring discussion about architecture used in the current project. My team and I had decided to try out VIP (a.k.a [Clean-Swift](http://clean-swift.com?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)) architecture and after two months of development we have argued about its pros and cons.

This blog post is an initial comparison of all architectures that the swifting.io team has used in the past in their applications. In this issue, I will explain basic concepts of MVC (does it even need to be explained  😉 ?), MVVM, VIPER and VIP (a.k.a. Clean-Swift) and point out their advantages and drawbacks.

I hope you will find these thoughts useful and helpful, wether you're about to fight your own battles in **Architecture Wars** at your workplace or you just want to select right solution for your own app 😎!

<br/>

<div style="text-align:center">
<img src ="https://raw.githubusercontent.com/swiftingio/blog/%2324-Architecture-Wars/Images/Meme.jpg?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post"/>
</div>

<br/>
 
#### MVC

Probably many developers started their adventure with iOS applications with the [CS193P](https://itunes.apple.com/us/course/developing-ios-9-apps-swift/id1104579961?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) Stanford course on iTunes U, as I did. In the course, one can get to know basic **Cocoa Touch** concepts, as long as the **Model - View - Controller** architecture hailed by [Apple](https://developer.apple.com/library/ios/documentation/General/Conceptual/DevPedia-CocoaCore/MVC.html?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post).

The basic concept of it is that an object called **Controller** responds to events triggered by a **View**. It can update a **Model** upon an action, and update the **View** when the **Model** changes. In the worst-case scenario, the **Controller**, or rather the **View Controller** in Cocoa Touch, deals with animations, data persistence and network calls, growing into a so called **Massive View Controller**. If we extracted some of the logic into separate objects, such as Core Data (Persistence) and Network services, the architecture would look as follows:

<br/>

![](https://raw.githubusercontent.com/swiftingio/blog/%2324-Architecture-Wars/Images/MVC.png?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

<br/>

The MVC architecture is used all the time by Apple and the majority of developers follow the lead. When for Apple it's the easiest way to explain their API concepts, for developers it's the fastest way to develop an application. I think that Apple knows that Massive View Controllers are a problem, so to improve testability of MVC components Apple mentioned dependency injection as a good practice on WWDC 2016's Improving Existing Apps with Modern Best Practices [session](https://developer.apple.com/videos/play/wwdc2016/213/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post). I interpret this as if they said "Hey, it's not good to put all of the code to a View Controller, inject some dependency objects that do certain things and test interactions with them" 😉.

#### MVVM

MVC is not the only valid app architecture. There are also other, and amongst them there is a **Model - View - View Model**. It assumes existence of a **View Model**, which is an additional layer that  a **View Controller** communicates with. It contains business logic and to tackle with it, it can contain other objects, such as e.g. Network and Core Data services. It exposes a **Model** as formatted data in its properties, so that the **View Controller** can bind the data to a **View**. Usually the **View Controller** observes changes in those properties and updates its **View** on a property change.

<br/>

![](https://raw.githubusercontent.com/swiftingio/blog/%2324-Architecture-Wars/Images/MVVM.png?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

<br/>

As every approach to application design, **MVVM** has its drawbacks. In our project, proverbial 1000 lines of View Controller became 1000 lines of View Model. Of course, View Controller got rid of other responsibilities than responding to View events and animations, but enormously huge business logic made our View Model to swell a lot.

With MVVM you can also use some reactive extensions, that allow to observe changes in View Model's properties and act upon those changes. You can either bind a View Model's property to a View directly or put a block of code that executes when property value changes and bind its output to a View. I have used [Swift Bond](https://github.com/SwiftBond/Bond?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) for that, which contains extensions for UIKit objects that simplify binding View Model properties to views. It is no longer maintained, but you can tap into [ReactiveKit](https://github.com/ReactiveKit/ReactiveKit?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post), which is a successor of Swift Bond.

There is also a more known framework called [RxSwift](https://github.com/ReactiveX/RxSwift?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post), which is a part of ReactiveX family. It contains RxCocoa, which are reactive extensions for Cocoa and CocoaTouch. What's great about RxSwift is that if you have created an Rx app in the past in other language, the rules, operators, functions and keywords in RxSwift are the same. You can check [RxMarbles](http://rxmarbles.com?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)  website to get familiar with Rx operators and functions.
  
#### VIPER

We've already written the whole [issue](https://swifting.io/blog/2016/03/07/8-viper-to-be-or-not-to-be/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) about **View - Interactor - Presenter - Entity - Router**. It is another way of setting up your app's architecture. VIPER divides View Controller's responsibility in standard MVC to different objects. A **View Controller** sets data from its **Presenter** to a **View**. **Presenter** formats data from **Interactor** and asks **Router** to perform navigation to a different module. **Interactor** contains business logic on data manipulation, performs network requests and saves **Entities** (model) to a persistent store via proxy objects.  View, Interactor, Presenter, Entity and Router components of a View Controller, along with it, are called a **module**.

<br/>

![](https://raw.githubusercontent.com/swiftingio/blog/%2324-Architecture-Wars/Images/VIPER.png?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

<br/>

This architecture clearly enforces to write code in a structured way and divides View Controller's responsibilities onto smaller objects. When compared to MVVM, business logic and data presentation was split from View Model to Interactor and Presenter. There is also an additional component called Router (a.k.a Wireframe), which deals with setting up other view controllers, wiring up their dependencies and setting delegations which allow communication with other scenes. After all, a View Controller shouldn't have knowledge on what the next view controller is and how to set it up.

Using VIPER for code of your app is hard at first. The architecture is hard to understand, there are too little materials on the topic for iOS world and there is a large entry barrier for new developers in the team. Architecture adds code overhead, components of a module rather cannot be reused. Communication is based on protocols which hardens navigation between many files in Xcode. But c'mon, protocols facilitate testing, which is the thing you should probably use in your apps 😉.
 
#### VIP (Clean Swift)

This architecture was proposed by [Raymond Law](https://twitter.com/rayvinly) on his [Clean Swift blog](http://clean-swift.com/clean-swift-ios-architecture/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post). In my opinion, VIP (a.k.a. Clean Swift) can be considered as a variation of VIPER. It gets its name from **View (Controller) - Interactor - Presenter** uni-directional cycle. VIP diverges from VIPER but yet it nicely divides responsibilities and propagates data among components. **View Controller** interacts directly with an **Interactor** by sending Requests to it. The **Interactor** responds to those requests by sending a Response with data model to a **Presenter**. The **Presenter** formats data to be displayed, creates a View Model and notifies the **View Controller** that it should update its **View** based on the View Model. **View Controller** decides when the navigation to a different scene should happen by calling a method on a **Router**. The **Router** performs setup of the next View Controller and deals with wiring, passing data and delegation setup. VIP components belonging to a single View Controller form a **scene**.

<br/>

![](https://raw.githubusercontent.com/swiftingio/blog/%2324-Architecture-Wars/Images/VIP.png?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

<br/>

Clean Swift has a very good documentation and reasoning behind its concept explained on the [Clean Swift blog](http://clean-swift.com?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post). Clean Swift concepts are rigid, especially namespaces created for every scene, data passing in **Request**, **Response** and **ViewModel** objects, which are rather non-reusable within scene, because they correspond to a specific Request-Response chain. Sometimes Interactor methods doesn't have to take any arguments, yet the architecture enforces usage of empty Request params. Communication between objects is again based on protocols, but it has some overhead in protocol declaration. 

Clean Swift assumes, that View Controller has an **output** that implements methods from **ViewControllerOutput** protocol. The **Interactor** is **ViewController**'s output, but methods that View Controller calls on Interactor are its **input**, hence it implements methods from **InteractorInput**. In code it would look as following:

```Swift
protocol ViewControllerOutput {
    func doSomehtnig(_ request: Request)
}
protocol InteractorInput: ViewControllerOutput {}
```

Although Clean Swift imposes strict rules, it appropriately divides responsibilities and creates uni-directional cycle. When compared to VIPER, another difference is that now the View Controller itself contacts Router for navigation, which is in my opinion a nice improvement (no routing of navigation request through VIPER's Presenter). Currently, we use a slightly modified VIP architecture for a project and we are quite fond of it.

#### Comparison

It's time for a comparison of MVC, MVVM, VIPER and VIP. For some of the metrics, MVC is a baseline. We take into consideration the following aspects:

-  **Responsibilities** - describes if there is an object that plays all roles or if there are many objects with different liabilities
- **View Controller**- describes the role of a View Controller
- **Data flow**  - describes how data flows in the scene
- **Testability** - evaluates how easy it is to test components
- **Entry** - rates how easy it is for a person to dive into a project or to start developing an app in a certain architecture
- **Collaboration** - rates how easy it is to collaborate on a scene written in a certain architecture
- **Reusability** - evaluates if scene components can be reused in a different scene
- **Refactoring** - rates how many components are affected during refactoring process
- **Number of files** - evaluates number of files in the project
- **Lines of code in a single file** - evaluates number of lines in the single file
 
<br/>

| | MVC | MVVM | VIPER | VIP | |
| --- | :---: | :---:| :---: | :---: | --- |
| **Responsibilities** | entangled | dispersed | dispersed | dispersed | |
| **View Controller** | does everything | passes actions to and binds data from View Model |  passes actions and binds data from Presenter | passes actions and displays data from Presenter, decides when to navigate to the next scene | |
| **Data flow** | multi-directional | multi-directional |  multi-directional | uni-directional | |
| **Testability** | hard (too many responsibilities) | better |  best | best | |
| **Entry** | easy | hard (when starting with reactive extensions) | hardest | hard, but good documentation exists | |
| **Collaboration** | hard | better | best | best | |
| **Reusability** | rather none | rather small | ok | ok | |
| **Refactoring** | normal | normal (affects View Controller - View Model boundary) | worse (can affect many boundaries due to multi-directional flow) | normal (affects one boundary due to uni-directional flow) | |
| **Number of files** | normal |  additional View Model for every View Controller | many | many | |
| **Lines of code in a single file** | too many | many | most-satisfactory | most-satisfactory | |

<br/>

#### Summary

It's not easy to select the best architecture for an app. I hope that this issue will enlighten differences between a few architectures and help you choosing the one that suits you best. As a curiosity, recently I came across [Flux](https://realm.io/news/benji-encz-unidirectional-data-flow-swift/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) architecture for iOS, which develops Facebook's Flux concept for web apps. I haven't tried it yet, but the main idea is that application state is passed through a uni-directional flow that allows views to update accordingly and store history of app states. Let us know if you have already tapped into it!

Sometimes we have to fight fiercely for the chosen architecture. Not all developers in the team will like it, not everything will be easy at first, but it is worth discussing and worth using a nice architecture. In the discussion, you can get to the point where you realise that certain aspects of the architecture do not suit or are redundant and you can compromise. After all, it's better to have a good atmosphere in the team, where developers are friends rather than enemies that feel resentment to each other because of this new and stupid architecture you write your application in. Remember that, if you happen to battle in your Architecture Wars, eventually source code will look as usual... 

<br/>

![](https://raw.githubusercontent.com/swiftingio/blog/%2324-Architecture-Wars/Images/clean%20slate.jpg?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

<br/>

This issue is just an introduction to architectures realm. We will continue the topic with a sample app written in MVC, MVVM, VIPER and VIP. Stay tuned!

#### References

- [Apple's MVC](https://developer.apple.com/library/ios/documentation/General/Conceptual/DevPedia-CocoaCore/MVC.html?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post).
- [Artsy's MVVM in Swift post](http://artsy.github.io/blog/2015/09/24/mvvm-in-swift/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)
- [objc.io's MVVM article](https://www.objc.io/issues/13-architecture/mvvm/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)
- [Alt Conf - Scott Gardner - Reactive programming with RxSwift](https://realm.io/news/altconf-scott-gardner-reactive-programming-with-rxswift?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)
- [swifting.io's VIPER issue](https://swifting.io/blog/2016/03/07/8-viper-to-be-or-not-to-be/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) 
- [objc.io's VIPER article](https://www.objc.io/issues/13-architecture/viper/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)
- [Clean Swift blog](http://clean-swift.com/clean-swift-ios-architecture/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)
- [Real-world Flux iOS](http://blog.benjamin-encz.de/post/real-world-flux-ios/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)
