---
layout: post
author: michal
title: \#1 Meaningful print debugging
excerpt: 
---
#### Print debugging problems
Print debugging is simple and yet a powerful method. It is usually enough to toss a ```print()``` in a couple of places and a problem is recognised. As it works well for values and simple objects, when we print a more complex instance, the result is less informing.

To illustrate this, let's take this simple ```Wheel``` class: 

``` 
class Wheel {
    var spokes:Int = 0
    var diameter:Double = 0.0
    
    init(spokes:Int = 32, diameter:Double = 26.0) {
        self.spokes = spokes
        self.diameter = diameter
    }
    
    func removeSpoke() {
        spokes = spokes > 0 ? spokes-- : spokes
    }
}

var wheel = Wheel(spokes: 36, diameter: 29)
print(wheel) // Wheel
debugPrint(wheel) //Wheel
```

#### Struct benefits 
Note that if ```Wheel``` was a struct and not a class we would already get a better print output:

```
var wheel = Wheel(spokes: 36, diameter: 29) // a struct
print(wheel) // Wheel(spokes: 36, diameter: 29)
debugPrint(wheel) // Wheel(spokes: 36, diameter: 29)
```
#### Protocols come to rescue
But we do have some classes and that's where usually protocols [CustomStringConvertible](http://swiftdoc.org/v2.1/protocol/CustomStringConvertible/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) for ```print()``` and [CustomDebugStringConvertible](http://swiftdoc.org/v2.1/protocol/CustomDebugStringConvertible/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) for ```debugPrint()``` come in handy: 

```
extension Wheel:CustomStringConvertible, CustomDebugStringConvertible {
    var description:String {
        return "Wheel has \(spokes) spokes"
    }
    var debugDescription:String {
        return "Wheel has \(spokes) spokes and \(diameter) inches"
    }
}
```

So now, if we print our wheel object after confoming to ```CustomStringConvertible``` and ```CustomDebugStringConvertible``` protocols then we get:

```
print(wheel) //Wheel has 36 spokes
debugPrint(wheel) //Wheel has 36 spokes and 29.0 inches
```
Neat!

#### UIViewControllers and NSObject inheritance
One of the least verbose print outputs belongs to ```UIViewController```:

```
class WheelsViewController:UIViewController {
    var wheels = [Wheel]()
}
var wheelsViewController = WheelsViewController()
print(wheelsViewController)//<WheelsViewController: 0x7f8e0861b590>
```

To make it simple, let's just focus on ```CustomStringConvertible``` protocol and ```print()``` function. However, now if we try to conform to ```CustomStringConvertible``` protocol:

```
extension WheelsViewController:CustomStringConvertible {
    var description:String {
        return "WheelsViewController has \(wheels)"
    }
}
```
we receive an error ***"Redundant conformance of 'WheelsViewController' to protocol 'CustomStringConvertible'"***

That is because ```UIViewController``` inherits from ```NSObject``` that in turn conforms to ```NSObjectProtocol```. We can see that all ```NSObject``` subclasses already implement ```description``` and optionally ```debugDescription``` properties:

```
public protocol NSObjectProtocol {
	...
    public var description: String { get }   
    optional public var debugDescription: String { get }
    ...
}
```
To fix that issue we will override inherited properties.

#### Overriding inherited properties
As we are not satisfied with the default implementation of ```description``` property inherited from ```UIViewController```, we can override it within the body of our ```WheelsViewController``` class or as an extension:

```
extension WheelsViewController {
    override var description:String{
        return "WheelsViewController has \(wheels.count) wheels"
    }
}

var wheelsViewController = WheelsViewController()
print(wheelsViewController) //WheelsViewController has 0 wheels

wheelsViewController.wheels = [wheel1,wheel2]
print(wheelsViewController) //WheelsViewController has 2 wheels
```

#### That's it!
This post is inspired by Udacity course [Xcode Debugging](https://www.udacity.com/course/viewer?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post#!/c-ud774). It's a great starting point in mastering your debugging skills. More advanced developers will also find it useful. Playground with the code from this post is available [here](https://github.com/swiftingio/blog/tree/%231-Meaningful-print-debugging?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post).
