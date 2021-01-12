---
layout: post
author: maciej
title: \#44 Watch your Bluetooth!
excerpt: 
---
On WWDC 2017 the breaking news was revealed - WatchOS 4 ships with `CoreBluetooth` and allows apps to connect up to 2 peripherals! ❤️! This issue will show a simple implementation of a Bluetooth Central that can be used in apps built for iOS 11 and WatchOS 4!

![](https://raw.githubusercontent.com/swiftingio/blog/%2344-Watch-your-Bluetooth/watch.png)


#### Bluetooth Low Energy (BLE)

Bluetooth Low Energy is a standard of low-range wireless communications between devices. It was introduced in Bluetooth Core Specification 4.0 and is developed by Bluetooth Special Interest Group. The current version of the specification is 5.0 ([Bluetooth official website](https://www.bluetooth.com)). The standard assumes low energy consumption and client-server architecture. Clients are called centrals. Centrals try to access data on peripherals (servers). Peripherals expose characteristics with values. Similar characteristics are grouped into higher-level construct called services. 

The standard is supported on Apple platforms via `Core Bluetooth` framework since `macOS 10.7`, `iOS 5`, `tvOS 9` and now will be available also on `watchOS 4`!

####  Core Bluetooth

The Core Bluetooth framework reflects central and peripheral roles with `CBCentral` and  `CBPeripheral` objects. Apps that connect to BLE devices do so with `CBCentralManager` objects.

![](https://raw.githubusercontent.com/swiftingio/blog/%2344-Watch-your-Bluetooth/Watch%20your%20bluetooth/Watch%20your%20bluetooth.010.jpeg)

The manager scans for peripherals that advertise certain services identified with an UUID (Universally Unique Identifiers). UUIDs are represented by `CBUUID` objects. An identifier is a 128-bit hexadecimal number, e.g. `B7AC06DC-09FF-40ED-B03A-55D09B08EB4A`. The identifier can be created with `uuidgen` tool on macOS Terminal.

![](https://raw.githubusercontent.com/swiftingio/blog/%2344-Watch-your-Bluetooth/uuidgen.png)

Surprise, surprise. An iOS/macOS app can also become a `CBPeripheral`. It has to use a `CBPeripheralManager` to advertise available services. 

![](https://raw.githubusercontent.com/swiftingio/blog/%2344-Watch-your-Bluetooth/Watch%20your%20bluetooth/Watch%20your%20bluetooth.001.jpeg)

Services and characteristics exposed by the peripheral are seen as `CBService` and `CBCharacteristic` objects.

![](https://raw.githubusercontent.com/swiftingio/blog/%2344-Watch-your-Bluetooth/Watch%20your%20bluetooth/Watch%20your%20bluetooth.002.jpeg)

They are identified by UUIDs, as mentioned earlier. In the next section you will get to know where to store used in your app UUIDs. 

![](https://raw.githubusercontent.com/swiftingio/blog/%2344-Watch-your-Bluetooth/Watch%20your%20bluetooth/Watch%20your%20bluetooth.003.jpeg)

#### Bird Service

![](https://raw.githubusercontent.com/swiftingio/blog/%2344-Watch-your-Bluetooth/watch.png)

Our watchOS 4 app will connect to peripherals that expose a **Bird Service**. The service should expose data related to bird's properties: name, color and transparency. It can look like this:

- `CBService` - Bird
	- name `CBCharacteristic` - contains bird's name encoded as `UTF-8 String`
	- color `CBCharacteristic` -	 contains hex color encoded as `UTF-8 String`
	- alpha `CBCharacteristic` -	 contains transparency Integer value ranging from 0-100 encoded as `UTF-8 String`

##### Storing UUIDs

Usually UUIDs are repeated over an over in Core Bluetooth - related code. It's handy to store them in static variables instead of instantiating `CBUUID` objects every time we want to compare a UUID of a service or a characteristic with another UUID. We propose an empty enum as a storage for UUIDs corresponding to a service.

```Swift
enum BirdBluetoothService {
    static let uuid:  CBUUID = CBUUID(string: "B7AC06DC-09FF-40ED-B03A-55D09B08EB4A")
    static let color: CBUUID = CBUUID(string: "B7AC06DC-09FF-40ED-B03A-55D09B080001")
    static let name:  CBUUID = CBUUID(string: "B7AC06DC-09FF-40ED-B03A-55D09B080002")
    static let alpha: CBUUID = CBUUID(string: "B7AC06DC-09FF-40ED-B03A-55D09B080003")

    static let characteristics: [CBUUID] = [ color, name, alpha ]    
}
```

The enum contains the `characteristics: [CBUUID]` property which facilities recognition of UUIDs corresponding to characteristics.

##### Bird Peripheral

A peripheral that our watchOS app will connect to should advertise Bird Service, respond to READ requests of a characteristic's value and NOTIFY subscribers about value updates. Peripheral role is not allowed on a watchOS and is not a subject of this post. Our demo project on [Github](https://github.com/swiftingio/watch-your-bluetooth) contains an implementation of a macOS app that uses `CBPeripheralManager` object to satisfy Bird Peripheral requirements.
 
![](https://raw.githubusercontent.com/swiftingio/blog/%2344-Watch-your-Bluetooth/mac.png)

#### Bird Central

Our watchOS app should be able to:

- scan for the Bird Service
- discover service's characteristics
- read characteristics' values
- subscribe for notifications of value changes

So, let's write a `BirdCentral` that fulfils those requirements!

First of all, our object should conform to `CBCentralManagerDelegate` and `CBPeripheralDelegate`. It requires conformance to `NSObjectProtocol` so `BirdCentral` will inherit from `NSObject` class.

```Swift
class BirdCentral: NSObject, CBCentralManagerDelegate, CBPeripheralDelegate {  
	//...
}
```

In the designated initialiser we create a `DispatchQueue` on which our object will be notified about Bluetooth-related events. We create a `CBCentralManager` instance with the queue and set the object as central's delegate.

```Swift    
    let central: CBCentralManager
    override init() {
        let queue = DispatchQueue(label: "io.swifting.bluetooth")
        central = CBCentralManager(delegate: nil, queue: queue)
        super.init()
        central.delegate = self
    }
   
```    
    
When the central manager’s state is updated we can start scanning for Bird Service. The `scanServices()` method checks if Bluetooth is supported and powered on on a device.


```Swift        
	func centralManagerDidUpdateState(_ central: CBCentralManager) {
		scanServices()
	}

	func scanServices() {
		guard central.state == .poweredOn else { return }
		central.scanForPeripherals(withServices: [BirdService.uuid], options: nil)
	}
```

If a peripheral is found, we can stop the scan, store the peripheral, become its delegate and connect to the peripheral.
    
```Swift
    
    var peripheral: CBPeripheral?
    
    func centralManager(_ central: CBCentralManager, didDiscover peripheral: CBPeripheral, advertisementData: [String : Any], rssi RSSI: NSNumber) {
        central.stopScan()
        self.peripheral = peripheral
        peripheral.delegate = self
        central.connect(peripheral, options: nil)
    }
```    
    
When the central connects to the peripheral, we can discover its services. In our case we pass the `BirdService.uuid` in an array of services for discovery. 
    
```Swift
    weak var delegate: BirdCentralDelegate?

    func centralManager(_ central: CBCentralManager, didConnect peripheral: CBPeripheral) {
        peripheral.discoverServices([BirdService.uuid])
        DispatchQueue.main.async {
            self.delegate?.central(self, didPerformAction: Action.connectPeripheral(true))
        }
    }
```   

We use a `BirdCentralDelegate` protocol to notify a delegate about certain `Action`s that occur - peripheral (un)successful connection, disconnection or reading a value. 

```Swift
protocol BirdCentralDelegate: class {
	func central(_ central: BirdCentral, didPerformAction: BirdCentral.Action)
}

extension BirdCentral {
	enum Action {
		case connectPeripheral(Bool)
		case disconnectPeripheral
		case read(Value)
	}  
}
```   

If the central fails to connect to or disconnects from the peripheral, we nullify the `peripheral` property, start scanning for peripherals and notifies the delegate about the action.

```Swift
    
    func centralManager(_ central: CBCentralManager, didFailToConnect peripheral: CBPeripheral, error: Error?) {
        print(error.debugDescription)
        self.peripheral = nil
        scanServices()
        DispatchQueue.main.async {
            self.delegate?.central(self, didPerformAction: Action.connectPeripheral(false))
        }
    }
    
    func centralManager(_ central: CBCentralManager, didDisconnectPeripheral peripheral: CBPeripheral, error: Error?) {
        print(error.debugDescription)
        self.peripheral = nil
        scanServices()
        DispatchQueue.main.async {
            self.delegate?.central(self, didPerformAction: BirdCentral.Action.disconnectPeripheral)
        }
    }
```    


When we ask peripheral to discover services we get a callback `peripheral(:didDiscoverServices:)`. We can extract the Bird Service from an array of peripheral's services and discover its characteristics.
    
```Swift
    
    func peripheral(_ peripheral: CBPeripheral, didDiscoverServices error: Error?) {
        guard error == nil else { return }
        guard let service = (peripheral.services?.filter { $0.uuid == BirdService.uuid })?.first else { return }
        peripheral.discoverCharacteristics(BirdService.characteristics, for: service)
    }
```    

When characteristics are discovered we can read their values, subscribe for notifications about value update and store a characteristic in a property.

```Swift
    
    var name: CBCharacteristic?
    var color: CBCharacteristic?
    var alpha: CBCharacteristic?
    
    func peripheral(_ peripheral: CBPeripheral, didDiscoverCharacteristicsFor service: CBService, error: Error?) {
        guard error == nil else { return }
        
        service.characteristics?.forEach {
            peripheral.readValue(for: $0)
            peripheral.setNotifyValue(true, for: $0)
            switch $0.uuid {
            case BirdService.nameCharacteristicUUID:
                name = $0
            case BirdService.alphaCharacteristicUUID:
                alpha = $0
            case BirdService.colorCharacteristicUUID:
                color = $0
            default:
                return
            }
        }
    }
```    
    
Finally, when a value of a characteristic is read or updated the `peripheral(:didUpdateValueFor characteristic:error:)` gets called. We convert binary data stored in the `value` property of the characteristic into a string. In the case of the `name` characteristic we just wrap it into `BirdCentral.Value` enum. In other cases we have either to convert it into an alpha value that can be used as alpha of a `UIView` object (i.e. `CGFloat` from 0.0 - 1.0) or to convert a hexadecimal string into a `UIColor`. We also tell our delegate about new reading from the peripheral.

```Swift
func peripheral(_ peripheral: CBPeripheral, didUpdateValueFor characteristic: CBCharacteristic, error: Error?) {
	guard error == nil else { return }
	guard let string = characteristic.value?.string else { return }

	let response: Value
	
	switch characteristic.uuid {
		case BirdService.nameCharacteristicUUID:
			response = .name(string)
		case BirdService.alphaCharacteristicUUID:
			guard let int = Int(string) else { return }
			response = .alpha(CGFloat(int) / 100)
		case BirdService.colorCharacteristicUUID:
			guard let color = UIColor(hex: string) else { return }
			response = .color(color)
		default:
			return
		}
		
		DispatchQueue.main.async {
			self.delegate?.central(self, didPerformAction: .read(response))
    }
}
```

Neat! We have just written the `BirdCentral` that can be used in both - an iOS and a watchOS applications! A single component is portable to both platforms.

![](https://raw.githubusercontent.com/swiftingio/blog/%2344-Watch-your-Bluetooth/iphone.png)

#### Summary

You can check out our [Github](https://github.com/swiftingio/watch-your-bluetooth) project to have a full overview on how to use it. It contains three targets - a macOS app that simulates the `BirdPeripheral` and an iOS and a watchOS app that use the `BirdCentral` component. 

Starting from watchOS 4 you can benefit from Core Bluetooth and connect up to 2 peripherals from your apps. Check out the [WWDC 2017 - 712](https://developer.apple.com/videos/play/wwdc2017/712/) session **What's New in Core Bluetooth** for more!
 
![](https://raw.githubusercontent.com/swiftingio/blog/%2344-Watch-your-Bluetooth/watch.png)

#### Links
- [Watch your Bluetooth](https://github.com/swiftingio/watch-your-bluetooth) - demo source code on Github
- [Core Bluetooth](https://developer.apple.com/documentation/corebluetooth?changes=latest_minor) - documentation changes 
- [WWDC 2017 - 712](https://developer.apple.com/videos/play/wwdc2017/712/) - What's New in Core Bluetooth
- [WWDC 2017 - 205](https://developer.apple.com/videos/play/wwdc2017/205/) - What's New in watchOS
- [WWDC 2017 - 408](https://developer.apple.com/videos/play/wwdc2017/408/) - What’s New in Playgrounds
- [Apple](https://developer.apple.com/bluetooth/) - Bluetooth for Developers
- [swifting.io](https://swifting.io/blog/category/corebluetooth/) - #Core Bluetooth
- [Bluetooth](https://www.bluetooth.com) - official website
