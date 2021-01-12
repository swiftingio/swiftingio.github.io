---
layout: post
author: bartek
title: \#23 Notifications in iOS 10
excerpt: 
---

iOS 10 gave us new ***rich notifications*** with a lot more functionalities comparing to old ones. We can view photos and videos or respond to a message right from our notifications. In this post I would like to focus on them, show you some code snippets, examples and good practices. I hope it will be helpful for development in your current and future applications.

Before we start, make sure that you have all tools needed:

* Xcode 8 beta 1 - only devices with 3d touch support expanded content (since Xcode 8 beta 2 -devices with and without 3D Touch support expanded content) 
* OS X minimum version: El Capitan 10.11.4
* import *UserNotifications.framework* in every file where you need notifications

You can download example Xcode project (Xcode 8 Beta 6, Swift 3.0) [here](https://github.com/swiftingio/blog/tree/%2323-Notifications-in-iOS-10/Example%20Project/SwiftingNotification?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post). 

#### Quick APNs overview

Apple Push Notification service transports and routes remote notifications for your apps from your provider to each user’s device. Provider sends the notification and a device token to the APNs servers. The APNs servers handle the routing of that notification to the correct user device, and the operating system handles the delivery of the notification to your client app:

![](https://raw.githubusercontent.com/swiftingio/blog/%2323-Notifications-in-iOS-10/Images/APNS.jpg?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

A few important facts:

* Max Payload 4kB
* HTTP/2 network protocol 
* TLS 1.2 is required between backend and APNs
* 2 environments (development and production)

More information about APNs you can find [here](https://developer.apple.com/library/ios/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/Chapters/ApplePushService.html?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post).

#### New notifications in general

I strongly recommend watching two WWDC sessions:

* [Introduction to Notifications](https://developer.apple.com/videos/play/wwdc2016/707/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) 
* [Advanced Notification](https://developer.apple.com/videos/play/wwdc2016/708/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) 

Main assumptions:

* Work in foreground 
* Familiar API with feature parity
* Expanded content
* Same code path for local and remote notification handling
* Simplified delegate methods
* Better notification management
* In-app presentation option
* Schedule and handle notifications in extensions
* Notification Extensions (Content and Service)!

Ok, let's start with some basics:

#### Registration of notification

Registration is needed for local and remote notifications. To turn it on just run below method in *application(_:didFinishLaunchingWithOptions:launchOptions:)* :

```
UNUserNotificationCenter.current().requestAuthorization([.alert, .sound, .badge]
 { (granted, error) in
...
}
```
As we see what we can authorize is:

* Banners
* Sound alerts
* Badging

I think that everyone knows result of registration: 

![](https://raw.githubusercontent.com/swiftingio/blog/%2323-Notifications-in-iOS-10/Images/Registration.png?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

#### Get Settings

iOS 10 gives you the ability to access user settings in your application so you can be smarter about the notifications that you want to send to the user depending upon their preferences:

```
UNUserNotificationCenter.current().getNotificationSettings { (settings) in 
... 
}
```

***Note:*** *settings* is a [*UNNotificationSettings*](https://developer.apple.com/reference/usernotifications/unusernotificationcenter/1649524-getnotificationsettings?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) object, which contains  the authorization settings for your app.

#### Token Registration 

If we want to handle remote notifications we need to send device token (which identifies the device to APNs) to our backend, to do this in *UIApplication()* use old method:

```
func application(_ application: UIApplication,
                     didRegisterForRemoteNotificationsWithDeviceToken deviceToken: Data) {
...
}
```

After calling the *requestAuthorization* method of the *UIApplication* object, the app calls this method when device registration completes successfully.

#### Triggers

We have 3 types of triggers:

* Time Interval
* Calendar
* Location

##### Time Interval trigger

![](https://raw.githubusercontent.com/swiftingio/blog/%2323-Notifications-in-iOS-10/Images/timeIcon.png?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

Using time trigger you can set how often do you want to run notification, or how big delay do you want to set.
 
```
UNTimeIntervalNotificationTrigger(timeInterval: 120,
repeats: false)
```

##### Calendar trigger 

![](https://raw.githubusercontent.com/swiftingio/blog/%2323-Notifications-in-iOS-10/Images/calendarIcon.png?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) 
 
Using calendar trigger you can set exact time for triggering.
 
```
let date = NSDateComponents()
date.hour = 8
date.minute = 30
UNCalendarNotificationTrigger(dateMatching: dateComponents,
repeats: false)
```

##### Location trigger

![](https://raw.githubusercontent.com/swiftingio/blog/%2323-Notifications-in-iOS-10/Images/locationIcon.png?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) 

This kind of trigger is used to schedule notification delivery when you enter or leave certain location, for example when you are leaving your workplace.
 
```
let center = CLLocationCoordinate2DMake(37.335400, -122.009201)
let region = CLCircularRegion.init(center: center, radius: 2000.0,
                                   identifier: "Headquarters")
region.notifyOnEntry = true
region.notifyOnExit = false
UNLocationNotificationTrigger(region: region, repeats: false)
```

#### Schedule
 
 * For local:
 
```
let content = UNMutableNotificationContent()
content.title = "Swifting.io Notifications"

let request = UNNotificationRequest(identifier: Consts.requestIdentifier, content: content, trigger: nil)
        UNUserNotificationCenter.current().add(request) { error in
            UNUserNotificationCenter.current().delegate = self
            if (error != nil){
                //handle here
            }
        }

```

A *UNNotificationRequest* object is used to schedule a local notification and manages the content for a delivered notification. A notification request object contains: *UNNotificationContent* object with the contents of the notification and *UNNotificationTrigger* object that specifies the conditions that trigger the delivery of the notification.
 
* For remote:

Backend sends notification payload directly to APNs servers:
  
```
	{
 	"aps" : {
 		"alert" : {
			...
 			},
 		},
	}
```

#### Rich notifications

![](https://raw.githubusercontent.com/swiftingio/blog/%2323-Notifications-in-iOS-10/Images/Rich.gif?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

In iOS 10 now we have much more content build comparing to old ones:

* title
* subtitle
* body
* attachment (showed in next sections)

Let's create a local notification:

```
let content = UNMutableNotificationContent()
content.title = "Swifting.io Notifications"
content.subtitle = "Swifting.io presents"
content.body = "Rich notifications"
```

And remote:

```
{
 "aps" : {
 	"alert" : {
 		"title" : "Swifting.io Notifications",
 		"subtitle" : "Swifting.io presents",
 		"body" : "Rich notifications" 
 		},
 	},
}
```

#### Media attachment

![](https://raw.githubusercontent.com/swiftingio/blog/%2323-Notifications-in-iOS-10/Images/Media.gif?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

Quick overview about media attachment in new iOS notifications:

* Can be used in both: local and remote notifications
* Support: image, audio, video
* Download in the service extension for remote notification (more in section: 'Service Extension')
* Limited processing time and size

Remote:

```
{
 aps: {
 alert: { … },
 mutable-content: 1
 }
 my-attachment: "https://example.com/photo.jpg"
}
```

Local:

```
let attachment = try? UNNotificationAttachment(identifier: Consts.imageIdentifier,
                                                       url: url,
                                                       options: [:])
if let attachment = attachment {
	content.attachments.append(attachment)
}                                              

```

***Note:*** What is very important is that *url* is a url to image saved in local space on device. You can find sample implementation in the example project.

#### Notification Content Extension

![](https://raw.githubusercontent.com/swiftingio/blog/%2323-Notifications-in-iOS-10/Images/Custom.gif?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

It displays custom user interface upon notification expanasion. Below the most important things:

* Custom views
* No interaction with views (possible only with notification actions)
* Respond to notification actions (UNNotificationAction)
* Create in Xcode as a new target *Notification Content Extension*
* Support for remote and local notifications

How to create it from scratch?

1) Add a new target:

![](https://raw.githubusercontent.com/swiftingio/blog/%2323-Notifications-in-iOS-10/Images/NotificationContentExtensionAdd.jpg?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

2) Set an identifier for *UNNotificationExtensionCategory* key in extension's *Info.plist*:

![](https://raw.githubusercontent.com/swiftingio/blog/%2323-Notifications-in-iOS-10/Images/NotificationExtensionInfoPlist.jpg?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

***Note:*** You may include multiple Notification Content extensions in your app’s bundle. You can change type of *UNNotificationExtensionCategory* key to Array and add a new category (showed below) or just add a new extension to your project and set a new value for *UNNotificationExtensionCategory* key in *Info.Plist* file.

![](https://raw.githubusercontent.com/swiftingio/blog/%2323-Notifications-in-iOS-10/Images/TwoCategoriesInfoPlist.jpg?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

3) Create custom view in *NotificationViewController.swift*

When creating new notifications we need to add new *category* key and set its value to what we typed in the Info.plist in step 2):

Remote:

```
{
 aps: {
 alert: { … },
 category: 'io.swifting.notification-category' 
 }
}
```

Local:

```
let content = UNMutableNotificationContent()
content.title = "Swifting.io Notifications"
content.subtitle = "Swifting.io presents"
content.body = "Custom notifications"
conrent.category = "io.swifting.notification-category"
```

Notifications can contain images, gifs, and videos (up to 50 megabytes in size).

What is also worth mentioning is keys of *Info.plist* in Content Extension target:

* *UNNotificationExtensionInitialContentSizeRatio*: number that represents the initial size of your view controller’s view expressed as a ratio of its height to its width.
* *UNNotificationExtensionDefaultContentHidden*:  When set to *YES*, the system displays only your custom view controller in the notification interface. When set to *NO*, the system displays the default notification content in addition to your view controller’s content.

![](https://raw.githubusercontent.com/swiftingio/blog/%2323-Notifications-in-iOS-10/Images/NotificationExtensionInfoPlist.jpg?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

Do you want to know more about Notification content extenions? Visit [API Reference](https://developer.apple.com/reference/usernotificationsui/unnotificationcontentextension?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post).

#### Debugging Content Extension

Unfortunately, at this moment debugging is not possible (tested on Xcode 8 Beta 6). Breakpoints set in *NotificationViewController.swift* don't work. Fixing bugs, as always, can be very time consuming and of course it happend to me. I have spent a lot of time by searching for bugs in my notifications content extenion...:/ I hope Apple will fix debugging in the future.

#### Service extension

Service extension lets you customize the content of a remote notification before it is delivered to the user:

![](https://raw.githubusercontent.com/swiftingio/blog/%2323-Notifications-in-iOS-10/Images/ServiceGraph.jpg?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

Main aspects:

* Non UI iOS Extension
* Augment or replace the content of visible Remote Notifications
* Short execution time (see [serviceExtensionTimeWillExpire()](https://developer.apple.com/reference/usernotifications/unnotificationserviceextension/1648227-serviceextensiontimewillexpire?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) method)

For example, you could use the extension to decrypt an encrypted data block or to download images associated with the notification:

```Swift
// Adding an attachment to a user notification
public class NotificationService: UNNotificationServiceExtension {
 override public func didReceive(_ request: UNNotificationRequest,
 withContentHandler contentHandler: (UNNotificationContent) -> Void) {
	 let fileURL = // ...
		 let attachment = UNNotificationAttachment(identifier: "image",
		 url: fileURL,
		 options: nil)
	 let content = request.content.mutableCopy as! UNMutableNotificationContent
	 content.attachments = [ attachment ]
	 contentHandler(content)
 }
}
``` 

How to add service extension to your application?

1) Just add a new target to your project:

![](https://raw.githubusercontent.com/swiftingio/blog/%2323-Notifications-in-iOS-10/Images/ServiceExtensionAdd.jpg?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

Do you want to know more? visit [API Reference](https://developer.apple.com/reference/usernotifications/unnotificationserviceextension?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post).

#### Handling Notifications

##### Application in foreground
 
If you would like to present your notification in foreground don't forget to implement *UNUserNotificationCenterDelegate* method: 
 
```
protocol UNUserNotificationCenterDelegate : NSObjectProtocol

func userNotificationCenter(_ center: UNUserNotificationCenter,
 willPresent notification: UNNotification,
 withCompletionHandler completionHandler:
 (UNNotificationPresentationOptions) -> Void) {
      completionHandler( [.alert,.sound])
 }

```
##### For interacting purposes implement

```
    func userNotificationCenter(_ center: UNUserNotificationCenter, didReceive response: UNNotificationResponse, withCompletionHandler completionHandler: () -> Void) {
        print("Tapped in notification")   
    }
```

It will be called when a user responded to the notification by opening the application, dismissing the notification or choosing a *UNNotificationAction*.

### Notifications Actions

![](https://raw.githubusercontent.com/swiftingio/blog/%2323-Notifications-in-iOS-10/Images/Action.gif?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

Lets recall a few facts from WWDC:

* Buttons with customizable title
* Text input functionality support([UNTextInputNotificationAction](https://developer.apple.com/reference/usernotifications/untextinputnotificationaction?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post))
* Actions can be diplayed for background or foreground notifications
* Supoort for iOS and watchOS

How to add a custom action? 

1) First of all we have to set up new category and add to this category our actions:

```
let action = UNNotificationAction(identifier:”reply", title:"Reply",options:[])
let category = UNNotificationCategory(identifier: "io.swifting.notification-request", actions: [action],
minimalActions: [action], intentIdentifiers: [], options: [])
UNUserNotificationCenter.current().setNotificationCategories([category])
```

2) Then, exactly as documentation says: The UNNotificationCategory's actions will be displayed on notifications when the UNNotificationCategory's identifier matches the UNNotificationRequest's categoryIdentifier:

```
...
content.categoryIdentifier = "io.swifting.notification-category"
        
let request = UNNotificationRequest(identifier: "io.swifting.notification-request", content: content, trigger: nil)
...
```

Remote:

```
{
aps : {
	alert : “Swifting.io Notifications”,
	category: "io.swifting.notification-category"
	}
}
```

Local:

```
content.categoryIdentifier = "io.swifting.notification-category"
```

#### Summary

I hope that you enjoyed this post and that stuff will be helpful in your applications!:) Don't hesitate to share your comments! Don' forget to download our sample project from [here](https://github.com/swiftingio/blog/tree/%2323-Notifications-in-iOS-10/Example%20Project/SwiftingNotification?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post).

#### Resources

* [Swifting.io example project](https://github.com/swiftingio/blog/tree/%2323-Notifications-in-iOS-10/Example%20Project/SwiftingNotification?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

* [UNNotificationContentExtension API Reference](https://developer.apple.com/reference/usernotificationsui/unnotificationcontentextension?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

* [UNNotificationServiceExtension API Reference](https://developer.apple.com/reference/usernotifications/unnotificationserviceextension?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

* [Push Notifications Tutorial by Ray Wanderlich](https://www.raywenderlich.com/123862/push-notifications-tutorial?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

* [Example Github Project with rich notifications in iOS 10 by ChenYilong](https://github.com/ChenYilong/iOS10AdaptationTips?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

* [WWDC 2016: Rich Notifications in iOS 10](https://willowtreeapps.com/blog/wwdc-2016-rich-notifications-in-ios-10/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

* [Notifications are getting a big upgrade in iOS 10 by George Deglin](https://onesignal.com/blog/notifications-are-getting-a-big-upgrade-in-ios-10/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)
