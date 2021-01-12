---
layout: post
author: maciej
title: \#27 Localize your strings swiftly
excerpt: 
---
Hey, long time no see! It's because we're working on two important issues, stay tuned! This issue will be short and it will show you how you can define and access localized version of your strings in a swift manner!

#### NSLocalizedString
In Xcode we can easily create a `Localizable.strings` file in which one can define pairs of `key` and `value` strings that can be used throughout our application. A sample file would look like this:

```
"Hello" = "Hello";
"This application is created by the swifting.io team" = "This application is created by the swifting.io team";
"Ops! It looks like this feature haven't been implemented yet :(!" = "Ops! It looks like this feature haven't been implemented yet :(!";
```

If we wanted to create e.g. an Italian version of our app, we would have to create another `Localizable.strings` and put Italian version of strings (values). Just use ⌘⇧N  (CMD + SHIFT + N) shortcut to create new file, type *strings* in filter field and select  *Strings File*. Name the file `Localizable.strings`:


![](https://raw.githubusercontent.com/swiftingio/blog/%2327-Localize-your-strings-swiftly/1.png?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)


You can edit the file with localisations:


![](https://raw.githubusercontent.com/swiftingio/blog/%2327-Localize-your-strings-swiftly/0.png?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)


Once you decide on creating a translation, select project file in *Project navigator* and in *Editor area*. Go to *Localizations* section, tap **+** button and select a language:


![](https://raw.githubusercontent.com/swiftingio/blog/%2327-Localize-your-strings-swiftly/2.png?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)


Then select files you want to localize (only `Localizable.strings` in our case):


![](https://raw.githubusercontent.com/swiftingio/blog/%2327-Localize-your-strings-swiftly/3.png?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)


And start translating your strings :)!

You can check full tutorial from Ray Wenderlich linked in **Resources** section at the bottom of the post. The tutorial is for an ancient 😉 language called **Objective-C**, but yet it's still one of my favourite languages (after **Swift** of course!).

```
"Hello" = "Ciao";
"This application is created by the swifting.io team" = "Questa applicazione è stato creato dal gruppo di swifting.io";
"Ops! It looks like this feature haven't been implemented yet :(!" = "Ops! Sembra che questa funzione non è stata ancora attuata :(!";
```

Ok, we have those files, but what's next? To use such strings we have to call `NSLocalizedString(key:comment:)` function:

```
let myString = NSLocalizedString(key:  "This application is created by the swifting.io team", comment:"")
```

Looks kinda ancient 😂. Do we really need this long` NSLocalizedString(key:comment:)` calls? And what's the `comment` param, left here empty?

You can pass a hint for a translator inside this argument, e.g. `NSLocalizedString(key:"Hello", comment: "This is the string shown on the first screen after app launch, to welcome the user")`. The comment is used by Xcode when exporting files for localization, but more on that can be found in [Apple's Internationalization and Localization Guide] (https://developer.apple.com/library/content/documentation/MacOSX/Conceptual/BPInternational/LocalizingYourApp/LocalizingYourApp.html?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post).

But still, do I really need to use such approach in my Swift code 🐦 !?

#### Swiftier way

No, a swifter way exists. Thanks to extensions, we can extend types and classes with functions and static variables. Hence, we can extend `String`. But before that, let's create a simpler version of `NSLocalizedString(key:comment:)` without `comment` param, in order not to type the empty comment hundreds of times throughout the file, if we don't intend to use `comment` hints for our translations 👹.

```
fileprivate func NSLocalizedString(_ key: String) -> String {
    return NSLocalizedString(key, comment: "")
}
```

We can mark this function as `fileprivate` so nobody would use this function unintentionally outside the file with our strings, but I can stay `internal` or `public` as well, doesn't matter. Oh, btw. you can create a `Strings+Localized.swift` file in which your Swift strings can be kept. So what's now?

Now we will extend the `String` type to include our localized versions of strings:

```
import Foundation
extension String {
    static let Hello = NSLocalizedString("Hello")
    static let ThisApplicationIsCreated = NSLocalizedString("This application is created by the swifting.io team")
    static let OpsNoFeature = NSLocalizedString("Ops! It looks like this feature haven't been implemented yet :(!")
}
```

You can use such an approach even without `Localizable.strings` file at all, just to have in mind that your app might need translation in the future. Having such an extension on `String` allows you to use localized version of strings like this:

```
let message: String = .ThisApplicationIsCreated
print(message)
let alert = UIAlertController(title: .Hello, message: .OpsNoFeature, preferredStyle: .alert)
```

##### A few more tips on extending String
If you have a large app, your `Strings+Localized.swift` file can become large and unreadable. You can divide strings into multiple extension blocks and annotate them using  `//MARK: `:

```
//MARK: Login screen
extension String {
//...
}
//MARK: Welcome screen
extension String {
//...
}
```

You could even create a file for each screen, e.g. `Strings+Login`, `Strings+Welcome.swift`, but be aware that in rapid development it can be a hinder task to do and maintaining multiple files could become even trickier.

One more thing to note is that Strings defined in the described way are static strings. Your app should be relaunched after changing language in iOS Settings app. If not, relaunch it by yourself in order to see changes. It can also have a memory overhead, since we initialize all strings at once, not at the time they're needed.

#### Resources
- [Our sample project on Github](https://github.com/swiftingio/SwiftyStrings?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)
- [Apple's Internationalization and Localization Guide](https://developer.apple.com/library/content/documentation/MacOSX/Conceptual/BPInternational/LocalizingYourApp/LocalizingYourApp.html?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)
- [Ray Wenderlich's tutorial](https://www.raywenderlich.com/64401/internationalization-tutorial-for-ios-2014?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)
