---
layout: post
author: michal
title: \#31 Getting started with Alexa on iOS
excerpt: 
---
Happy New Year 2017! 🍾🎉

January is a special time for swifting.io, as we will soon celebrate the first year of our blogging adventure 😁.

Today I would like to give you a brief introduction into Amazon Alexa Voice Assistant service. You might remember [our first steps with SiriKit](https://swifting.io/blog/2016/07/18/20-sirikit-can-you-outsmart-provided-intents?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post), where we were not so happy about it 😜. Apple has some work to do in VA space and while the do their job, I have decided to take a look at other contenders and have picked Alexa, one of the leaders.

#### Alexa vs. Echo

Let me briefly clarify two concepts. Alexa is a cloud based voice service. You can enhance it writing your own Skills using Alexa Skills Kit and you can run it virtually on any platform with a microphone and a speaker available. 

Echo on the other hand is a series of Alexa enabled devices by Amazon. Currently family consists of 3 devices:

- Amazon Echo - standalone mic/speaker
- Amazon Echo Dot - mini version, needs a speaker system
- Amazon Tap - portable version of Echo

For now devices are available in US, UK and Germany.

![Echo Devices Family](https://raw.githubusercontent.com/swiftingio/blog/%2331-Alexa-iOS-Basics/%2331%20Alexa%20iOS%20Basics/EchoFamily.png)

#### Your own Alexa powered device

What is really cool about Alexa Voice Service (AVS) is that it can be run on any device. You 'just' need to capture voice, build an authenticated request and be able to playback the response.

That being said, on the official Alexa GitHub page you can find code and step-by-step tutorials on how to run Alexa on RaspberryPi, Linux, Mac, Windows and RaspberryPi Dev Kit from Amazon.

Neat! 👍

I have personally went through [Mac](https://github.com/alexa/alexa-avs-sample-app/wiki/Mac?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) setup and after about 2 hours of work I was able to run it. 

##### Tips to get you trough 
Make sure to properly match all IDs generated in Alexa developer portal with config files in your code and have VLC environment paths set. 

If all that is too long, you may also want to check MacLexa project on GitHub - comes with even a ready to use binary.

#### So, can I run Alexa in iOS App?

Yes, you can! 
There is even a [small tutorial](https://developer.amazon.com/public/solutions/alexa/alexa-voice-service/docs/authorizing-your-alexa-enabled-mobile-app?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) on how to authorize Alexa-enabled Mobile Product in iOS and Android versions.

> Scenario: You want to add Alexa to your Android or iOS application. You do not have a headless device, like a smart speaker, that requires you to authorize it and associate it with a customer’s account.

I won't get into the details as portal gives you step-by-step instructions that may change over time, so it is better to have the link from the top of this section as a reference. Basically, what you will do in the tutorial above:


1. Register a Mobile App in Alexa Developer Portal
2. Get the Amazon SDK
3. Create simple Login with Amazon iOS app
4. Build a request to obtain a token to call AVS

##### Tips if you get stuck

- You will need to enable ***Keychain Sharing Capability*** for your app's target. This is ***#1*** tip for me. Took me 2 hours to figure it out.
- Make sure you have properly configured your *Info.plist* file. Check carefully API Key, URL Schemes and App Transport Security.
- Set a proper *productID* when building your request. This is a value from ID column in Amazon Developer Portal. 
- Make sure your app's *BundleID* matches BundleID set in Security Profile in Amazon Developer Portal.
- When building AVS sample app you have an option to authenticate using a companion service or a companion app (iOS or Android). iOS companion app has many common parts to app that you are building. Worth [looking](https://github.com/alexa/alexa-avs-sample-app/tree/master/samples/iOSCompanionApp?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) at it. This got me through with missing ***Keychain Sharing*** thing from Tip #1.

#### But how do I talk to it?

Well, Amazon's tutorial leaves you with AVS token ready to use. The rest is up to you. 

The good news is that community is awesome and that the job of porting Mac code to iOS was done by chintan1891. He has a fully working example [here on GitHub](https://github.com/chintan1891/iOS-Alexa?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post). Audio & response processing is done in Swift 2 and the login part is in ObjC, so you will need XCode7 to run it.

I am considering rewriting it to be Swift 3 only project. Let me know if you have seen it somewhere around or you have already started 😅.

#### TL;DR
Yes, you can have Alexa in your iOS app. 
There is even a sample ObjC/Swift2 code on [GitHub](https://github.com/chintan1891/iOS-Alexa?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) (credit for chintan1891).


#### Final thoughts
To sum up I am really pleased with Alexa voice recognition quality and overall experience. What is really appealing is the growing popularity of the service and devices. This directly translates to the growing developer community and number of Skills in Alexa Skills Store. With SkillsKit you can virtually build any capability or integration. 

IMHO Alexa is way ahead of Siri under many criteria, but in this rapidly growing area you never know what will happen in 1 year time.


There is plenty of topics yet to be covered when talking about Alexa. There are gCal integrations, home automation integrations or ... AlexaSkillsKit in Swift 😎. 

Have you tried Alexa or Echo already? Let us know in comments!

#### References

##### General
- [Alexa Skills Kit](https://developer.amazon.com/alexa?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) 
- Alexa's (Amazon official) sample app on [GitHub](https://github.com/alexa/alexa-avs-sample-app?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) 
- MacLexa project on [GitHub](https://github.com/kunal732/MacLexa?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)
- Devices: [Amazon Echo](https://www.amazon.com/Amazon-Echo-Bluetooth-Speaker-with-WiFi-Alexa/dp/B00X4WHP5E?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post), [Amazon Echo Dot](https://www.amazon.com/dp/B015TJD0Y4/ref=fs_ods_fs_aucc_bt?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post), [Amazon Tap](https://www.amazon.com/dp/B01BH83OOM/ref=fs_ods_fs_aucc_fx?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)
-  [Alexa Skills Store](https://www.alexaskillstore.com?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)
-  [AlexaSkillsKit in Swift](https://github.com/choefele/AlexaSkillsKit?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) and [@claushoefele post about it](https://medium.com/@claushoefele/building-alexa-skills-in-swift-3d596aa0ee95#.7cur49km8?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

##### iOS Alexa setup
- [Getting started with Alexa Voice Service](https://developer.amazon.com/public/solutions/alexa/alexa-voice-service/getting-started-with-the-alexa-voice-service?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)
- [Authorizing Alexa-enable mobile device](https://developer.amazon.com/public/solutions/alexa/alexa-voice-service/docs/authorizing-your-alexa-enabled-mobile-app?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)
- [Install Amazon SDK for iOS](https://developer.amazon.com/public/apis/engage/login-with-amazon/docs/create_ios_project.html?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)
- [Create Login with Amazon iOS project](https://developer.amazon.com/public/apis/engage/login-with-amazon/docs/create_ios_project.html?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) 
- [Ready to use iOS Alexa project (ObjC/Swift2)](https://github.com/chintan1891/iOS-Alexa?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) (credit for chintan1891)
