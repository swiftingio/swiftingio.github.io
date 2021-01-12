---
layout: post
author: michal
title: \#20 SiriKit – Can you outsmart provided Intents?
excerpt: 
---
In my first year of blogging in iOS space it became obvious to me that a period of time after WWDC is a great time for bloggers. There are plenty of things to explore and write about. Today, I would like to write about my first steps and attempts to outsmart limited SiriKit capability given to us, developers in iOS 10. Was I successful? Hop in and check out yourself. Seven getting started tips will get you on board faster.

#### Who is it for?! 
These were probably your first words when you have seen an initial batch of Intents made available for developers. You are not alone. There are plenty of similar comments around. Just for a reference. Here is a modest list of Intents:

- Audio or video calling
- Messaging
- Payments
- Searching photos
- Workouts
- Ride booking

You can see Uber, Endomondo and WhatsApp dudes cheering but it is easy to notice that if you are not a big shot, there is nothing useful for you here. 

Or is there?
 

#### Sample scenario
For my little experiment I have used an imaginary app **SupportMe**. SupportMe is a help desk app. It helps customers to search for answers in FAQ, Knowledge Base Articles and Community threads. 

You may notice that Siri integration would be nice here - you could ask her a question and she would return a list of matching results. All that possible without launching an app and typing a question yourself. 

What about new Speech Recognition API? Yes, that could work. A great idea for the next post 😆.

#### Getting started - Tips & Tricks

Getting started with SiriKit would not be that hard if it was not for Xcode 8 beta. James' Quave tutorial and Apple's Siri Integration Guide come in handy. You can find links to both in references. Xcode 8 beta is sometimes not stable enough and its new code signing features may be initially in your way.

##### Tip 1: How to debug Intent
First question that may come to your mind is how to debug Intent? 

***NOTE:*** You should debug on a device. 

Step 1: Build and run your base app from Xcode as usual

Step 2: Then select Intent scheme, build and run it.

##### Tip 2: Cannot debug - 'Cannot attach...' error

Following Tip 1 may not be enough. My debugger could not attach to Intent and I still have no clear answer how to deal with it. A combination of restarting Xcode, iPhone, removing an app and using Xcode 8 code signing switches fixed the issue. My best bet is code signing issues. Try to redownload profiles or reinstall developer profile. Unchecking and rechecking automatic signing feature in Xcode 8 may help too.

![Could not attach](https://raw.githubusercontent.com/swiftingio/blog/%2320-SiriKit-Challenge/%2320%20SiriKit%20Challenge/Images/attach.png)

##### Tip 3: Enable an app in Siri Settings
Your app should be enabled in Siri Settings. You can achieve it in two ways:

1. Go to Siri in Settings app and enable your app. 
2. Request it from the app, similarly to requesting other capabilites like push notifications. For Siri use ```requestSiriAuthorization``` class method of ```INPreferences```:

```
INPreferences.requestSiriAuthorization { (status) in
	// process status
	}

```

![Enable Siri](https://raw.githubusercontent.com/swiftingio/blog/%2320-SiriKit-Challenge/%2320%20SiriKit%20Challenge/Images/enable.png)

##### Tip 4: Remove print statements
If you want to use print debugging in IntentHandling methods make sure to launch them on the main queue. At the time of writing (Xcode 8 beta 2) print function seems to be broken, so you may experience some throws or even crashes. Hopefully, this will get better in beta 3. For now it is better to experiment without it.


##### Tip 5: Do not leave empty protocol methods
If you leave IntentHandling methods empty, you can get all sorts of weird behaviors from Siri.  *'Sorry, YourApp is incompatible'* or *'Sorry, You will have to continue in the app'* are examples. Luckily, usually all methods apart from ```handle``` are optional so you can quickly get started and then add new methods as you progress.

![Incompatibile App message](https://raw.githubusercontent.com/swiftingio/blog/%2320-SiriKit-Challenge/%2320%20SiriKit%20Challenge/Images/incompatibile.png)
![Continue in the App message](https://raw.githubusercontent.com/swiftingio/blog/%2320-SiriKit-Challenge/%2320%20SiriKit%20Challenge/Images/continue.png)

##### Tip 6: Siri Entitlement
This is new to Xcode 8 beta 2. To be able to play with SiriKit you should enable Siri in Capabilities tab of your app's target:

![Siri Entitlement](https://raw.githubusercontent.com/swiftingio/blog/%2320-SiriKit-Challenge/%2320%20SiriKit%20Challenge/Images/capability.png)

##### Tip 7: Fill plist with supported Intents
Many features in iOS require this. It is similar for SiriKit. Your SiriKit Intent Extension contains separate ```Info.plist``` file. Under NSExtension -> NSExtensionAttributes there is IntentsSupported array that should be filled with supported Intents.

![Intent](https://raw.githubusercontent.com/swiftingio/blog/%2320-SiriKit-Challenge/%2320%20SiriKit%20Challenge/Images/intent.png)

#### Mission (Im)possible
Let's get back to our challenge. Is it possible to use SiriKit Intents when our app has little to do with them? Can you also get a nice experience for your users?

There is no generic Intent that would fit into our app's context. So we will pretend to be a messaging app as there is Intent for that. To let Siri know we have to add ```INSearchForMessagesIntent``` in Siri Extension Info.plist as mentioned in Tip 7.

***WARNING:*** This mission has more talking to Siri than coding. 

##### Some code
Let's take a look at a very simple implementation of Intent Handling:

```
struct SupportMe{
    static let systems = [
    INPerson(personHandle: INPersonHandle(value: "MyNotes",
     type: INPersonHandleType.unknown),
     nameComponents: nil,
     displayName: "MyNotes",
     image: nil,
     contactIdentifier: "MyNotes",
     customIdentifier: "MyNotes")]
    
    static let articles = [
    INMessage(identifier: "MyNotesPassword",
	  content: "Retrieving password in MyNotes app. To retrieve
	   password use 'forgot password' button that is located below
	   sign in button. Then type email address that your account has
	   been assigned to and press reset password",
      dateSent: Date(),
      sender: SupportMe.systems[0],
      recipients: [SupportMe.systems[0]])]
}

extension IntentHandler: INSearchForMessagesIntentHandling{
    func handle(searchForMessages intent: INSearchForMessagesIntent,
     completion: (INSearchForMessagesIntentResponse) -> Void){
        let userActivity = NSUserActivity(activityType: String(INSearchForMessagesIntent.self))
        let response = INSearchForMessagesIntentResponse(code: .success,
         userActivity: userActivity)
        response.messages = [SupportMe.articles[0]]
        completion(response)
    }
}
```

You can find entire sample project on our GitHub [repository](https://github.com/swiftingio/blog/blob/%2320-SiriKit-Challenge/%2320%20SiriKit%20Challenge/Code/SupportMe?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post).

What is up there? 

```SupportMe``` struct works as data store of our app. From this store we will provide content for Siri to handle.

```IntentHandler``` is a class generated automatically when you create Intent Extension. As good Swift citizens we add conformance to ```INSearchForMessagesIntentHandling``` in an extension.

You may have noticed that we do not process intent at all. It just returns a hardcoded answer. That is because we want to check if we get our handler launched and response returned when we speak to Siri without the context of specified Intent.

##### Doing it like Siri expects

SupportMe app has ```INSearchForMessagesIntent``` defined. Siri will expect messaging app behaviours from it. For our first try, let's do it by the book:

- 'Search SupportMe messages' 👍

![Search messages](https://raw.githubusercontent.com/swiftingio/blog/%2320-SiriKit-Challenge/%2320%20SiriKit%20Challenge/Images/searchmessage.png)

- 'Search SupportMe messages for retrieving a password' 👍

![Search messages detail](https://raw.githubusercontent.com/swiftingio/blog/%2320-SiriKit-Challenge/%2320%20SiriKit%20Challenge/Images/searchdetail.png)

- 'Look in SupportMe messages' 👎

![Look in](https://raw.githubusercontent.com/swiftingio/blog/%2320-SiriKit-Challenge/%2320%20SiriKit%20Challenge/Images/lookin.png)

- 'Browse in SupportMe messages' 👎

![Browse in](https://raw.githubusercontent.com/swiftingio/blog/%2320-SiriKit-Challenge/%2320%20SiriKit%20Challenge/Images/browse.png)

We test ```INSearchForMessagesIntent``` heavily here. As it comes out the 'Search' word in this Intent definition is obligatory. We do not see text of article on screens marked with 👍 because Siri reads it 👏. When trying to use synonyms like *'Look in'* or *'Browse in'* we end up with web searches rather than being attached to our app.

##### What about 'message' in commands?
To be honest there is nothing wrong with the word search in Siri commands. We would also search for answers in SupportMe app. 'messages' is more disturbing. Knowledge Base Articles or Community threads have little to do with messages. 

Let's see what we can do:

- 'Search in SupportMe for retrieving a password' 👎

![Search in](https://raw.githubusercontent.com/swiftingio/blog/%2320-SiriKit-Challenge/%2320%20SiriKit%20Challenge/Images/searchin.png)

- 'Search SupportMe articles' 👎

![Articles](https://raw.githubusercontent.com/swiftingio/blog/%2320-SiriKit-Challenge/%2320%20SiriKit%20Challenge/Images/articles.png)

- 'Search SupportMe threads' 👎

![Threads](https://raw.githubusercontent.com/swiftingio/blog/%2320-SiriKit-Challenge/%2320%20SiriKit%20Challenge/Images/threads.png)

- 'Search SupportMe entries' 👎

![Entries](https://raw.githubusercontent.com/swiftingio/blog/%2320-SiriKit-Challenge/%2320%20SiriKit%20Challenge/Images/entries.png)

Sorry, no luck here. Apparently we are forced to use very strict syntax that ```INSearchForMessagesIntent``` understands. Without *'search'* and *'message'* words Siri will not pass intent to our app extension.

##### Adjusting vocabulary 

There is one more thing to check. We never give up so easily. ```INVocabulary``` class is documented as follows:

>The INVocabulary object ***lets you augment your app’s fixed vocabulary with terms that are both unique to your app and to the current user of your app***. Registering custom terms provides Siri with the hints it needs to apply those terms appropriately to the corresponding intent objects. You may register ***only specific types of custom terms, such as the name of a contact, the name of a user’s workout, a custom tag applied to a photo, or a user-specific payment type***.


It would be nice to have a custom name for the 'message' word, right? Sadly ```INVocabularyStringType``` enum has a following implementation:

```
public enum INVocabularyStringType : Int {
    case contactName
    case contactGroupName
    case photoTag
    case photoAlbumName
    case workoutActivityName
    case carProfileName
}

```

#### Summary and references
Experiment that we have conducted proves that SiriKit in a current version is quite limited and not that customisable. For now if you want to use SiriKit you will need to conform to strict Intents vocabulary. It may feel akward for users, so I would think twice. 

I would still praise the fact of opening the API and expect more Intents and customisation points in future releases. Maybe already in next betas or release candidate. It is common for Apple to release features gradually, giving everyone time to learn on a subset of functionallity. This is a valid point especially when it comes to Virtual Assistant, where it is quite easy to spoil user experience. Quoting a classic 😉:
> With great power comes great responsibility. 
 
Ready to hack SiriKit on your own? Grab a useful pack of references below and share your findings in comments or on [Twitter](https://twitter.com/swiftingio)!

- [Mission (Im)possible: code](https://github.com/swiftingio/blog/blob/%2320-SiriKit-Challenge/%2320%20SiriKit%20Challenge/Code/SupportMe?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)
- [Siri Integration Guide](https://developer.apple.com/library/prerelease/content/documentation/Intents/Conceptual/SiriIntegrationGuide/index.html?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)
- [WWDC 2016: Introducing SiriKit](https://developer.apple.com/videos/play/wwdc2016/217/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)
- [WWDC 2016: Extending your Apps with SiriKit](https://developer.apple.com/videos/play/wwdc2016/225/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)
- [WWDC 2016: Sample Code](https://developer.apple.com/library/prerelease/content/samplecode/UnicornChat/Introduction/Intro.html#//apple_ref/doc/uid/TP40017332-Intro-DontLinkElementID_2?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)
- [James Quave: SiriKit tutorial](http://jamesonquave.com/blog/adding-siri-to-ios-10-apps-in-swift-tutorial/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)
- [Possible Mobile: Code Signing Xcode8](https://possiblemobile.com/2016/06/code-signing-xcode-8?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)
- [SiriKit: First impressions](http://mjtsai.com/blog/2016/06/16/sirikit/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)
