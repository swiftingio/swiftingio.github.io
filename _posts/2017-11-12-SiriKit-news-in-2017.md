---
layout: post
author: michal
title: \#47 SiriKit news in 2017
excerpt: 
---
The 2017 season for Apple news is almost over. 
WWDC 2017 is a far history already. We've got new frameworks, revamped Xcode 9, iPhone X and ... HomePod coming soon.

In [Issue #43](https://swifting.io/blog/2017/05/30/43-bye-bye-fileprivate/) I was wondering if Apple would join the Virtual Assistants war with a device and vast access to Domains and Intents in SiriKit. The first one is true, there is HomePod coming in December. It's expensive ($350), it's heavy (2.5kg!), it's music oriented but will it be a threat for family of Echos or Google Homes? We will see 🤔. Let's focus on a developer side of the Force right now. With Google and Amazon furiously adding features like calling and remote notifications, my appetite has really increased.
Hop in and let's see what 2017 update brought us.


If you are new to SiriKit you may want to check [Issue #20](https://swifting.io/blog/2016/07/18/20-sirikit-can-you-outsmart-provided-intents/) and my first attempts to outsmart Intents provided in 2016.


#### Any new Intents or Domains?
The first thing I wanted to check was if there were any new Intents and hopefully domains that would bring SiriKit to masses. Well... here we go.

##### Payments
Payments domain got a neat refresh. 
Send Payment Intent has received resolving currencies and payees. There are also new intents ```INSearchForAccountsIntent``` to request information about user's accounts in an app and ```INTransferMoneyIntent``` to make transfers between two accounts belonging to users.
See references for full documentation.

##### Lists and notes
Lists and notes are completely new domains. In the first release you are able to perform the following:

- Create Note ```INCreateNoteIntent```
- Append to Note ```INAppendToNoteIntent```
- Search for Notebook Items ```INSearchForNotebookItemsIntent ```
- Create Task List ```INCreateTaskListIntent```
- Add Task ```INAddTasksIntent```
- Set Task Attributes ```INSetTaskAttributeIntent```

It's great to see an entirely new domain, but I was expecting more.

##### Visual codes
Visual codes is another and last new domain added so far. There is only one intent ```INGetVisualCodeIntent```
that is used to get one of 3 types of codes: contact, request payment or send payment. 

Once this intent is correctly interpreted, the QR or a barcode from the app is displayed to share a contact or to facilitate payments.


#### Improvements
During WWDC 2017, there were two videos dedicated to SiriKit - [one](https://developer.apple.com/videos/play/wwdc2017/214?utm_source=swifting.io&utm_medium=web&utm_campaign=blog)
 with news and [another one](https://developer.apple.com/videos/play/wwdc2017/228?utm_source=swifting.io&utm_medium=web&utm_campaign=blog) with best practices. 

##### INPerson.siriMatches
One of the topics covered on WWDC was siriMatches property added in iOS 10.3 to ```INPerson``` object.
In this way Siri helps you to resolve similar contact and multiple contact numbers.

##### INMessage.conversationIdentifier

```INMessage``` and ```INSendMessageIntent``` has received a new property called ```conversationIdentifier```. The goal is to make it easier to reply to the messages that Siri just read to the user. So, whenever you are using Siri to read a message it will contain ```conversationIdentifier```. 
The same identifier will be used when a user will want to reply using ```INSendMessageIntent```. ```INSendMessageIntent```'s ```conversationIdentifier``` property will contain the very same 'id'. With this 'id' it will be faster to extract a proper recipient from a conversation.


##### Response codes
Calling intents, messaging intents and payment intents have received a new response codes. It will help to provide more precise answers to the user.

- ```INSendPaymentIntent``` has received ```failureNotEligible``` code
- Audio and Video Calling intents have received ``` failureContactNotSupportedByApp``` and ```failureNoValidNumber```
- When resolving recipients in messaging now you have an option to return an unsupported recipient in 3 flavours: ```noAccount```, ```offline```, `messagingServiceNotEnabledForRecipient`

 
 
##### Security

- `IntentsRestrictedWhileLocked` use this dictionary in Info.plist of Intents Extension Target to restrict access for given intent while the device is locked (see image below).
- For actions that would require authentication even when the device is unlocked (like [unlocking a car](https://developer.apple.com/documentation/sirikit/car_commands?utm_source=swifting.io&utm_medium=web&utm_campaign=blog)) you can use LocalAuthentication.framework with `LAContext().evaluatePolicy...` during Siri request.

![](https://raw.githubusercontent.com/swiftingio/blog/%2347-SiriKit-2017/SiriKitInfoPlistItentsRestricted.png)





##### Vocabulary

Sometimes your app may use specific language that Siri may natively not understand. Lucklily, we can help her understand an app and user specific words using [INVocabulary](https://developer.apple.com/documentation/sirikit/invocabulary):

>The INVocabulary object lets you augment your app’s global vocabulary with terms that are both unique to your app and to the current user of your app. Registering custom terms provides Siri with hints it needs to apply those terms appropriately to the corresponding intent objects.




In iOS 10 when creating an `INVocabulary`, one could only provide plain strings. That caused some troubles in matching. This is why in iOS 11 `INSeakableProtocol` was introduced. There is a new API that allows to create INVocabulary of objects conforming to `INSpeakableProtocol`:

```
func setVocabulary(_ vocabulary: NSOrderedSet, 
                of type: INVocabularyStringType)
```

With `INSpeakableProtocol` you can provide a phrase, a pronounciation hint and alternative matches.
This works for both user specific and app global values.
 
#### Developer - UITesting & Siri Intent Query
There is also a set of improvements that will make developer's life easier.

##### UI Testing
It's now possible to use SiriKit within your UI Tests:

```
let siriService = XCUIDevice.shared.siriService
siriService.activate(voiceRecognitionText: "<Your SiriKit Command>"
)
```
Basically UI Testing Siri action would be composed of the following steps:

1. Setup for testing
2. Invoking Siri
3. Waiting for Siri response
4. Checking for expected state

This also allows you to test Siri in other languages that you or your testing team are not familiar with. Same for various hardware configurations that you do not own.

##### Siri Intent Query
Simple, yet powerful feature for developers in Xcode 9 is a field called 'Siri Intent Query' in Scheme Debug Info Tab. Filling this field will free you from being prompted for Siri query everytime while  debugging.
Neat!

![](https://raw.githubusercontent.com/swiftingio/blog/%2347-SiriKit-2017/SiriIntentQuery.png)

#### SiriKit UI
When creating SiriKitUI Target you can customize SiriKit generated views in auto-generated `IntentViewController` using `INUIHostedViewControlling` protocol method `configure view`. 

So far there was a possibility to customize only certain parts of SiriKitUI views. In iOS 11 we get more control with `INParameter` and `configureView(for:of:interactiveBehavior:context:completion:)` method of `INUIHostedViewControlling` protocol. Now, our initial `IntentViewController` looks similar to this:


```
import IntentsUI

class IntentViewController: UIViewController, INUIHostedViewControlling {
   
   ....
    
    // MARK: - INUIHostedViewControlling
    
    // Prepare your view controller for the interaction to handle.
    func configureView(for parameters: Set<INParameter>, of interaction: INInteraction, interactiveBehavior: INUIInteractiveBehavior, context: INUIHostedViewContext, completion: @escaping (Bool, Set<INParameter>, CGSize) -> Void) {
        // Do configuration here, including preparing views and calculating a desired size for presentation.
        completion(true, parameters, self.desiredSize)
    }
    
    var desiredSize: CGSize {
        return self.extensionContext!.hostedViewMaximumAllowedSize
    }
    
}

```



How does it work?

Siri keeps calling the `configureView` method mentioned above for every view and provides us various parameters like values or interactivity. For each such a call we can let Siri know in completion block if we want to draw the view by ourselves or let her draw the default view.

When we decide to draw the view by ourselves we pass `true` in the callback and provide the size. We can also decide to replace multiple default views with a single custom one. 



#### Other
Well, there is more to it. 

##### Background workout app intent handling
A pretty useful new feature is starting workout in background. Previously, Siri may ask a user to unlock an app before going any further which may be annoying.
In iOS 11, it is possible to launch workout apps in background.

This is executed using the response code `handleInApp` and the new `AppDelegate` method that handles intents:

```
// SirKit Extension Code
INStartWorkoutIntentResponse(code:.handleInApp , userActivity: activity) 
```

```
// AppDelegate code 
func application(_ application: UIApplication, handleIntent intent: INIntent,
 completionHandler: @escaping (INIntentResponse) -> Void) 
```

As you can see, in this case it's the app not the extension that handles Intent and decides about resolution success or failure.

##### Alternative app names

Another useful trick that we have in iOS 11 are alternative app names. While launching the app with your touch is pretty straightforward because you see it on your screen. With voice this kind of flexibility may be useful. To add alternative names fill in `INAlternativeAppNames` dictonary in SiriKit extension `plist` file with app names and pronounciations.

#### watchOS 3.2
Just a quick reminder - with watchOS 3.2 SiriKit finally reaches Apple Watch with a subset of supported Intents. Take a look at references for a full list.

#### The HomePod
What do we know so far? 
First things first. iOS 11.2 introduces SiriKit for HomePod. HomePod will relay requests recognized via Siri to user's iOS device for processing. iOS 11.2 is already availabe as beta. 
Second thing is that you can't have it until December. 
Some of most notable features are AirPlay 2 support for multiple speakers, smart home accessories control, room sensing for best acoustics and promised multiple layers of security 🤔. Look at references for a HomePod link to see all details.

![](https://raw.githubusercontent.com/swiftingio/blog/%2347-SiriKit-2017/An_Apple_HomePod_speaker_.png)


#### Summary
In 2017 SiriKit brought us serveral new Intents and Domains, plus some updates to existing ones. Then there is a pack of updates for developers like UI Testing or Siri Intent Query. However, the biggest news is  expected HomePod speaker release in December and it's support since iOS 11.2 for Messaging, Lists and Notes Domains. Will December bring some more updates for developers in SiriKit? Or maybe a new XYZKit? We will see. 
My sentiment towards provided Domains has not changed. There is not much use of them in typical projects. How many lists or messaging apps are there on the market, huh? 😏
In the next post related to SiriKit, I will attempt to create sample integration of SiriKit with one of the most popular list services. Stay tuned!


#### References
- [HomePod](https://www.apple.com/homepod?utm_source=swifting.io&utm_medium=web&utm_campaign=blog)
- [SiriKit in HomePod News](https://developer.apple.com/news/?id=10302017a?utm_source=swifting.io&utm_medium=web&utm_campaign=blog)
- [WWDC 2017: 214 - What's new in SiriKit](https://developer.apple.com/videos/play/wwdc2017/214?utm_source=swifting.io&utm_medium=web&utm_campaign=blog)
- [WWDC 2017: 228 - Making Great SiriKit Experiences](https://developer.apple.com/videos/play/wwdc2017/228?utm_source=swifting.io&utm_medium=web&utm_campaign=blog)
- [SiriKit - supported Domains & Intents](https://developer.apple.com/sirikit?utm_source=swifting.io&utm_medium=web&utm_campaign=blog)
- [SiriKit Payments](https://developer.apple.com/documentation/sirikit/payments?utm_source=swifting.io&utm_medium=web&utm_campaign=blog)
- [SiriKit Apple Documentation & Change Log](https://developer.apple.com/documentation/sirikit?changes=latest_minor)
- [SiriKit in watchOS 3.2](https://developer.apple.com/library/content/releasenotes/General/WhatsNewInwatchOS/Articles/watchOS3_2.html?utm_source=swifting.io&utm_medium=web&utm_campaign=blog)
- [INHostedViewControlling Apple Documentation](https://developer.apple.com/documentation/sirikit/inuihostedviewcontrolling?utm_source=swifting.io&utm_medium=web&utm_campaign=blog)
- [handleInApp Workout Intent response code](https://developer.apple.com/documentation/sirikit/instartworkoutintentresponsecode/2873780-handleinapp?utm_source=swifting.io&utm_medium=web&utm_campaign=blog)
