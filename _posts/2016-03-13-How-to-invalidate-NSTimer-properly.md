---
layout: post
author: michal
title: \#9 How to invalidate NSTimer properly?
excerpt: 
---
`NSTimer` is a useful little one. It waits for some time and then fires, can fire periodically and runs in the run loop. There is also one unexpected feature. `NSTimer` retains its target. It's not a big deal when you use a non-repeating timers as they get invalidated automatically after firing. However, when you use a repeating timer and forget about retaining, this means trouble. 

This post includes 2 ways applied by my team to deal with that issue. There is also a very simple trick to detect whether you have the retaining timer problem or not.

#### Our case
In our project we have a Facebook style wall with posts and status updates. As this is a very interactive part of an app it should be refreshed quite often. You can probably recall quite easily one of your cases. 

***NOTE:*** Problem is universal for any architecture of iOS app. You may find some articles for MVC and others, where `UIViewControllers` or other Controllers are responsible for `NSTimer` lifecycle. That is why we have decided to present examples with [VIPER](https://swifting.io/blog/2016/03/07/8-viper-to-be-or-not-to-be/?utm_source=swifting.io) architecture. In this scenario an Interactor is responsible for creating and invalidating `NSTimer`.


#### How NOT to do it: Invalidate in deinit
Problem in our code was detected during a code review of one of pull requests in repository. If you maintain `NSTimer` this way, your target, in our case an Interactor will never get deinitialised:

```
class WallInteractor: WallInteractorInputProtocol {
    private var timer: NSTimer?
    deinit {
        timer?.invalidate()
        timer = nil
    }
    func fetchWallPosts() {
		//Fetch posts and pass them to display
    }     
    func enableAutoSync() {
        guard timer == nil else { return }
        timer = NSTimer.scheduledTimerWithTimeInterval(15, target: self, selector: "fetchWallPosts", userInfo: nil, repeats: true)
    }
}
```

All trouble starts at the moment of passing ```self``` as a target. `NSTimer` holds a strong reference to ```WallInteractor``` instance. Even that we exit the wall screen, so that all other references are removed this one remains. 

As stated in [Apple's documentation](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/Foundation/Classes/NSTimer_Class/#//apple_ref/occ/instm/NSTimer/invalidate?utm_source=swifting.io) ```invalidate()``` method is the only way to remove `NSTimer` from a run loop and remove strong references to ```target``` and ```userInfo``` objects.



#### Do I have the same problem?

A quick way to verify whether problem is existing in your setup is to use a simple [print debugging](https://swifting.io/blog/2016/01/26/1-meaningful-print-debugging/?utm_source=swifting.io) method:

```
deinit{
	timer?.invalidate()
	timer = nil
	print("WallInteractor deinited")     
}
```
As we had assigned self as a target of `NSTimer` and had been trying to invalidate timer in ```deinit``` we have never seen that statement printed in a console.

#### Solution #1: Invalidate on view disappear events

One way to tackle this issue is to use ```viewWillDisappear()``` or ```viewDidDisappear()``` events to invalidate `NSTimer`. If you use ```UIViewController``` to handle `NSTimer` then it is pretty straightforward

```
override func viewWillDisappear(animated: Bool){
	super.viewWillDisappear(animated)
	timer?.invalidate()
}
```

In case of using VIPER architecture and handling `NSTimer` in Interactor we had to propagate that event from View through Presenter to Interactor:


```
//View
class WallView:WallViewProtocol{
override func viewWillDisappear(animated: Bool){
	super.viewWillDisappear(animated)
	presenter?.viewWillDisappear()
}
...
}
//Presenter
class WallPresenter:WallPresenterProtocol{
	func viewWillDisappear(){
		interactor?.viewWillDisappear()
	}
...
}
//Interactor
class WallInteractor:WallInteractorProtocol{
	private var timer: NSTimer?
  	func viewWillDisappear(){
  		timer?.invalidate()
  	}
 	deinit{
		print("WallInteractor deinited")  // get's printed
  	}
...
}
```

As you can see, passing such a simple information through half of the VIPER structure may be one of the reasons to start looking for an alternative. You could use notifications but that would not be the cleanest solution. 

#### Solution #2: Wrap self in an intermediate object

For some reasons you cannot or may not want to use view disappear events. Solution #1 solved the problem by moving ```invalidate()``` call to a different place. There is also possibility to deal with the problem by not giving self as `NSTimer`'s target. How? By using a wrapper class. For example:

```
class WallInteractor: WallInteractorInputProtocol {
    private class TimerTargetWrapper {
        weak var interactor: WallInteractor?
        init(interactor: WallInteractor) {
            self.interactor = interactor
        }
        @objc func timerFunction(timer: NSTimer?) {
            interactor?.fetchWallPosts()
        }
    }
    private var timer: NSTimer?
    deinit {
        timer?.invalidate() // no strong reference to Interactor. It gets called!
    }
    func fetchWallPosts() {
    	...   
    }
    func enableAutoSync() {
        guard timer == nil else { return }
        timer = NSTimer.scheduledTimerWithTimeInterval(15, target: TimerTargetWrapper(interactor: self), selector: "timerFunction:", userInfo: nil, repeats: true)
    }
}
```

What happens in the code above? 

Firstly, when scheduling `NSTimer` we pass an intermediate object ```TimerTargetWrapper``` as a target, that has a weak reference to our interactor. 

Secondly, when entire VIPER module is destroyed and it comes to deiniting Interactor it has just a weak reference from ```TimerTargetWrapper``` left. In this way ```deinit``` gets called and `NSTimer` gets invalidated. 

As mentioned above `NSTimer` after invalidation releases reference to its target, it this case the ```TimerTargetWrapper```. Everything gets cleaned up.


#### Summary
Hopefully you will find this article a useful reference for problems with invalidating `NSTimer`. Clearly it does not cover all edge cases and possible solutions to the problem. Additionally, we wanted to share some more insights from our work with VIPER architecture.

You can find some useful references below:

* [NSTimer - Apple Reference](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/Foundation/Classes/NSTimer_Class/?utm_source=swifting.io)
* [zpasternack.org post](http://zpasternack.org/dont-invalidate-your-nstimer-in-dealloc/?utm_source=swifting.io)
* StackOverflow [here](http://stackoverflow.com/questions/1876180/problem-invalidating-nstimer-in-dealloc?utm_source=swifting.io), [here](http://stackoverflow.com/questions/14490178/how-to-know-when-to-invalidate-an-nstimer?utm_source=swifting.io), [here](http://stackoverflow.com/questions/3478361/best-time-to-invalidate-nstimer-inside-uiviewcontroller-to-avoid-retain-cycle?utm_source=swifting.io) and [here](http://stackoverflow.com/questions/16821736/weak-reference-to-nstimer-target-to-prevent-retain-cycle?utm_source=swifting.io)
