---
layout: post
author: maciej
title: \#2 Un-crash swiftly from CBCentralManager callbacks to a deallocated object
excerpt: 
---
#### CBCentralManager 
```CBCentralManager``` is a class from CoreBluetooth framework for Bluetooth connectivity. It servers as a configurator of communication with external devices, represented as ```CBPeripheral``` objects by the framework. Configuration of Bluetooth interface and communication with peripherals is done asynchronuousley, so one can set self as delegate in order to catch Bluetooth-related events.

![Credits: Apple Inc., Core Bluetooth Programming Guide](https://developer.apple.com/library/ios/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/Art/CBObjects_CentralSide_2x.png?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)


#### CBCentralManagerDelegate and the problem
To receive callbacks from ```CBCentralManager``` instance its delegate has to implement ```CBCentralManagerDelegate``` protocol methods. The only required method is ```centralManagerDidUpdateState:```, which tells us about Bluetooth support of the device the app is run on.

#### Problem    
Aforementioned method can be called eventhough delegate object became deallocated. Let's see an example of that. 

Suppose we had a class ```Object``` that is initialised with an instant of ```CBCentralManager``` class. During initialisation the Object instance sets itself as ```CBCentralManager```'s delegate.

```
class Object {
    let centralManager: CBCentralManager
    init(centralManager: CBCentralManager) {
        self.centralManager = centralManager
        self.centralManager.delegate = self
    }
}
```

We implement required ```CBCentralManagerDelegate``` protocol methods in order to become a ```CBCentralManager```'s delegate:

```
extension Object: CBCentralManagerDelegate {
    @objc func centralManagerDidUpdateState(central: CBCentralManager){}
}
```

One remark - ```CBCentralManagerDelegate``` protocol requires conforming to ```NSObjectProtocol``` as well. The simplest way to do that is to inherit from ```NSObject``` class:

```
class Object: NSObject {
    //...
    init(centralManager: CBCentralManager) {
        self.centralManager = centralManager
        super.init()
        self.centralManager.delegate = self
    }
}
```

Imagine we had a function in which an instance of our ```Object``` was created but wasn't  stored anywhere else.

```
func foo() {
    let centralManager = CBCentralManager(delegate: nil, queue: nil)            
    let _ = Object(centralManager: centralManager)
}
```

After function call app would crash with exception of type ***Unrecognised selector sent to instance***, since ```CBCentralManager``` would try to send messages to its delegate, despite it would be deallocated immediately after returning from the function.

#### Fix and un-crash from doom!

There is however a quick fix for such a problem. We could use deinit method of our ```Object``` class to nullify centralManager's delegate. After deallocation of  ```Object``` instance messages from centralManager won't be sent. Hurray :)!

```
deinit  {
    centralManager.delegate = nil
}
```

Grab [source code](https://github.com/swiftingio/blog/tree/%232-Un-crash-swiftly-from-CBCentralManager-callbacks-to-a-deallocated-object?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) from our repository to see it in action :)!

To find out more about Bluetooth Low Energy on iOS check out [Core Bluetooth Programming Guide](https://developer.apple.com/library/ios/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/AboutCoreBluetooth/Introduction.html?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post#//apple_ref/doc/uid/TP40013257-CH1-SW1?utm_source=swifting.io).
