---
layout: post
author: maciej
title: \#18 Do pure Swift objects have load() method?
excerpt: 
---
I'm pretty sure this day, or actually the whole week, will be pretty exciting due to the start of WWDC 2016. Before that happens I'd like to share my finding from unit testing of one of my view controllers. If you're not familiar with unit testing, check out our last issue [Unit Test all the things!](https://swifting.io/blog/2016/06/06/17-unit-test-all-the-things/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post).

#### Unit test all the things! 

We encourage you to test all the things ;) ! But mainly **interactions** between objects. I also like to test communication of  **view controllers** with their dependencies. The example below shows how a sample test could look like:

```Swift
context("ViewController") {
    var sut: ViewController!
    var dependencyOfSut: Dependency!
    
    beforeEach {
        dependencyOfSut = MockDependency()
        sut = ViewController(dependency: dependencyOfSut)
    }
    
    describe("when view loads") {
        beforeEach {
            sut.viewDidLoad()
        }
        
        it("setups something") {
            expect(dependencyOfSut.setupSomethingCount).to(equal(1))
        }
        
    }
}
```

The snippet should be self-explanatory. If it isn't, let me elucidate it a bit.  This is a test of ViewController class, that is initialised with a Dependency object in  ```beoforeEach```'s closure argument. Test checks what happens when ```viewDidLoad``` method gets called. It is expected that ```setupSomething``` method is called once when view loads.

#### Ok, but what else can we test?

We can check if appropriate buttons are set on **navigationItem**:

```Swift
it("has right button") {

	expect(sut.navigationItem.rightBarButtonItem)
		.to(beAKindOf(UIBarButtonItem))
		
	expect(sut.navigationItem.rightBarButtonItem?.enabled)
		.to(beTrue())
		
}
```

We can also assert if view controller is UITableView's or UICollectionView's **delegate**:

```Swift
expect(sut.tableView.delegate 
	as? MyViewController).to(equal(sut))
	
expect(sut.collectionView.delegate 
	as? MyViewController).to(equal(sut))
```

Another super important  thing to check is if an appropriate action takes place after a **tap** on a **button**. Last week we migrated an app from Swift 2.1 to 2.2 and we didn't have tests for navigation bar buttons. Buttons simply stopped working. If we had tests for their actions we would have noticed the bug before releasing demo version of our app to client. Of course a tester could assert that everything works correctly, but you need to have one. Unit test all the things. Really! :)

What has changed in [Swift 2.2](https://swift.org/blog/swift-2-2-new-features/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)? Among other things the syntax for selectors. 

The old way:

```Swift
navigationItem.rightBarButtonItem =
        UIBarButtonItem(barButtonSystemItem: .Add, target: self,
                        action: "doSomething")
```

Code in Swift 2.2:

```Swift
 navigationItem.rightBarButtonItem =
        UIBarButtonItem(barButtonSystemItem: .Add, target: self,
                        action: #selector(doSomething))
```

BTW, if you don't know what's your Swift version in Xcode you can check it out with the ```xcrun swift -version``` command in Terminal.

Ok, but how to test if an action is connected to a button? Look at the snippet below. Imagine we had a Helper class that could perform a tap on a button (more about it in a moment). In assert part of your test you simply need to check if a desired action happened, just like before, by using method counters.

What if you wanted to check if a new view controller was shown after action? You need to use ```toEventually``` function that is able to asynchronously assert the expectation. Your expectation is to check if presented view controller of tested object is of certain kind:

```Swift
describe("tap on the right button") {
	beforeEach {
		Helper.tap(sut.navigationItem.rightBarButtonItem)
	}
	
	it("shows my other view controller") {
		expect(sut.presentedViewController)
			.toEventually(beAKindOf(MyOtherViewController))
	}
}
```

The ```Helper``` class actually exists. It was typealiased in the snippet. I've actually called it UnitTestHelper and its contents is available in this [gist](https://gist.github.com/paciej00/01f458c9877b85159a5125b446c258ae?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post).

```Swift
class UnitTestHelper {
    
    class func tap(button: UIButton?) {}

    class func tap(button: UIBarButtonItem?) {}
    	
	//...
	
}
```

The ```tap``` function uses ```UIApplication```'s ```sendAction:to:from:forEvent``` method for sending an action to button's target.

If you want to see more samples of UIViewController unit tests, check this [gist](https://gist.github.com/paciej00/2c4d77ce457683727738?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post). It contains snippet for testing a view controller with a view model and a table view. Ok but let's go to the main question of this post.

#### Do pure Swift objects have **load** method?!

```NSObject``` instances/subclasses have  ```load()``` class function that is
 
> Invoked whenever a class or category is added to the Objective-C runtime;
 
--Apple's [documentation](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/Foundation/Classes/NSObject_Class/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post#//apple_ref/occ/clm/NSObject/load)
 
In my recent project I used MVVM architecture, where every view controller has its own view model (a component that deals with business logic and formats data to be displayed).  One of the desired interactions of view controller with its view model was that  view controller, in ```viewDidLoad```, calls ```load``` method on view model to load data to be displayed.

```Swift
class MyViewModel {
	func load() {}
}

class MyViewController: UIViewController {
	let viewModel: MyViewModel
	init(viewModel: MyViewModel = MyViewModel()) {
		self.viewModel = viewModel
	}
	override func viewDidLoad() {
		super.viewDidLoad()
		view.backgroundColor = UIColor.whiteColor()
		viewModel.load()
	}
}	
```

In terms of unit test spec the desired behaviour was asserted like this:

```Swift
describe("when view loads") {
	 beforeEach {
		sut.viewDidLoad()
     }
	 it("calls load on viewModel") {
		expect(viewModel.loadCount)
		.to(equal(1)) 
	}
}
```

I ran tests, they failed, which was ok at that point. After all I use BDD and I was in the red phase. Then I implemented the code,  ran tests again and the aforementioned assertion failed again. I started scrutinising my code, but I didn't find any flaw. What had possibly gone wrong? The assertion looked like this:

![](https://raw.githubusercontent.com/swiftingio/blog/%2318-Do-pure-Swift-objects-have-load-method/load-count-assertion.png?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

I couldn't believe my eyes. How?! Pure Swift object with ```load``` method called before I called it? Do pure Swift objects have ```load``` method?! I scrutinised my code again, put some breakpoints here and there. Then I realised that this method gets called after I called ```viewDidLoad```:
 
```Swift
func loadView()
```

> The view controller calls this method when its view property is requested but is currently nil.

--Apple's [documentation](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIViewController_Class/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post#//apple_ref/occ/instm/UIViewController/loadView)

Do you know what's called after ```loadView```? The ```viewDidLoad``` method. Hence ```load``` was called twice on ```viewModel```. And why on Earth was the ```loadView``` called? Because I touch ```view``` property for the first time during test runtime:

```Swift
override func viewDidLoad() {
		super.viewDidLoad()
		view.backgroundColor = UIColor.whiteColor() //HERE view is touched for the first time during the unit test
		viewModel.load()
}
```

What's the solution to this problem? Mine was to accept the ```loadCount``` to be ```greaterThan(0)```.  Instead of calling ```viewDidLoad``` I could just access ```sut.view``` in ```beforeEach``` in order to call ```viewDidLoad``` only once.  **UPDATE July 9th 2016:** Starting from iOS9 you can also use the ```loadViewIfNeeded``` [method](https://developer.apple.com/reference/uikit/uiviewcontroller/1621446-loadviewifneeded?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) and remember not to touch view during initialisation 😉! Thx for **Kakamil** for the tip!

Maybe you have other solution to the problem, or have experienced other unit testing story. I encourage you to share them in comments :)!

#### TL; DR

Pure Swift objects don't have **load** method.

![](https://raw.githubusercontent.com/swiftingio/blog/%2318-Do-pure-Swift-objects-have-load-method/test-duplicated-load-call.png?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)
