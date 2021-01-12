---
layout: post
author: bartek
title: \#26 SwiftLint, SonarQube and CheckMarx… what else?
excerpt: 
---
Today we would like to talk about news from SwiftLint and also look at different static analyzers frameworks like:

- SonarQube
- Checkmarx

Concentrating on some basics, best practices, tips and just personal feelings about each one.

**Important:** if  you would like to learn more about SwiftLint I recommend visiting our previos post about [SwiftLint](https://swifting.io/blog/2016/03/29/11-swiftlint/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post).

Ready?

![](https://raw.githubusercontent.com/swiftingio/blog/%2326-Static-Code-Analyzers/Images/WhatElse.jpg?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

Let's start!

#### SwiftLint

A tool to enforce Swift style and conventions, loosely based on [GitHub's Swift Style Guide](https://github.com/github/swift-style-guide?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post).

##### Basics

* Current version: [0.12](https://github.com/realm/SwiftLint/releases?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)
* 38 rules available [here](https://github.com/realm/SwiftLint/tree/master/Source/SwiftLintFramework/Rules)
* It's free 
* Developed and supported by [Realm](https://realm.io/)
* 76 Contributors

##### What's new in 0.12 version?

New version has a few new rules, options and fixes. For example:

- MARK Rule - MARK comment should be in valid format (to enforce *//MARK:* syntax):

![](https://raw.githubusercontent.com/swiftingio/blog/%2326-Static-Code-Analyzers/Images/mark.jpg?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

- Private Outlet Rule - IBOutlets should be private to avoid leaking UIKit to higher layers:

![](https://raw.githubusercontent.com/swiftingio/blog/%2326-Static-Code-Analyzers/Images/privateOutlet.jpg?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

- Vertical Whitespace Rule: Limit vertical whitespace to a single empty line:

![](https://raw.githubusercontent.com/swiftingio/blog/%2326-Static-Code-Analyzers/Images/VerticalWhiteSpace.jpg?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

- Private Unit Test Rule: Unit tests marked private are silently skipped:

![](https://raw.githubusercontent.com/swiftingio/blog/%2326-Static-Code-Analyzers/Images/privateUnitTest.jpg?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

- Legacy NSGeometry Functions Rule - Struct extension properties and methods are preferred over legacy functions (available for SDK: macOS 10.10+)


- New ***junit*** report option:

```
swiftlint lint --report junit
```
In result we can get a report like the one below:

```
...
<testcase classname='Formatting Test' name='.../SwiftLintTest/SwiftLintTestTests/SwiftLintTestTests.swift'>
<failure message='Line should be 100 characters or less: currently 109 characters'>warning:
Line:16 </failure>	</testcase>
	<testcase classname='Formatting Test' name='.../SwiftLintTestTests/SwiftLintTestTests.swift'>
<failure message='Line should be 100 characters or less: currently 111 characters'>warning:
Line:20 </failure>	
</testcase>
...
```

It can be very useful for continuous integration system like Bamboo where you can have online dashboard of SwiftLint results:

![](https://raw.githubusercontent.com/swiftingio/blog/%2326-Static-Code-Analyzers/Images/Bamboo.jpg?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

##### Lessons learned

After almost one year of using it, personally I think it's a very great tool! Below are main thougts about my experience:

- Use it from the start of your project. Every single build will correct your tiny mistakes and will inform you about violations.
- Meet with your iOS team and establish your own rules (according to your company guidelines).
- Experiment with values for different rules and change it during development of your project, because sometimes very strict rules can change your coding into a fight with a tool:/.
- It's very useful especially when you work in a big iOS team to have consistent codebase.
- Very easily configurated in just one file: *.swiftlint.yml*.
- Can be integrated with continuous integration systems (like Bamboo or Jenkins).

##### Tips

- You can also integrate SwiftLint with your Xcode by using [SwiftLintXcode plugin](https://github.com/ypresto/SwiftLintXcode?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) (unfortunately it doesn't have seamless installation process for Xcode 8):

![](https://cloud.githubusercontent.com/assets/400558/14304460/d2a133dc-fbed-11e5-9573-2c21cce699e0.png?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

- Move *.swiftlint.yml* configuration file to Xcode project file structure:

![](https://raw.githubusercontent.com/swiftingio/blog/%2326-Static-Code-Analyzers/Images/swiftlintyml.jpg?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

In this way you can configure it very easily.

#### SonarQube

[SonarQube](http://www.sonarqube.org/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) is an open source platform for continuous inspection of code quality. Supports more han 20 languages: Java, C/C++, Objective-C, C#, PHP, Flex, Groovy, JavaScript, Python, PL/SQL, COBOL, Swift, etc. Unfortunately plugins for some of the languages are commercial, as is the plugin for Swift. Additionally it has available more than 50 plugins.

##### Basics

Sonar at this moment has [92 rules](http://dist.sonarsource.com/reports/coverage/rules_in_swift.html) for Swift, which are divided into three groups:

![](https://raw.githubusercontent.com/swiftingio/blog/%2326-Static-Code-Analyzers/Images/SonarBugsVulBad.jpg?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

They can be marked in aspect of severity as:
Blocker, Critical, Major, Minor, Info:

![](https://raw.githubusercontent.com/swiftingio/blog/%2326-Static-Code-Analyzers/Images/SonarSeverity.jpg?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

Some examples below:

- Bugs:
	 - URIs should not be hardcoded (Critical)
	 - Optionals should not be force-unwrapped (Major)
	 - Functions and closures should not be empty (Major)
- Vulnerabilities:
	- Credentials should not be hard-coded (Critical)
	- IP addresses should not be hardcoded (Major)
	- Access control should be specified for top-level definitions (Major)
- Code smells:
	- Files should not have too many lines (Major)
	- Source files should not have any duplicated blocks (Major)
	- "self" should only be used when required (Minor)

##### Metrics

Sonar uses following [metrics](http://docs.sonarqube.org/display/SONAR/Metric+Definitions#MetricDefinitions-Security?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post):


- Security: Number of vulnerabilities.

![](https://raw.githubusercontent.com/swiftingio/blog/%2326-Static-Code-Analyzers/Images/Security.jpg?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

- Maintability: Number of code smells.

![](https://raw.githubusercontent.com/swiftingio/blog/%2326-Static-Code-Analyzers/Images/Maintability.jpg?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

- Reliability: Number of bugs.

![](https://raw.githubusercontent.com/swiftingio/blog/%2326-Static-Code-Analyzers/Images/Reliability.jpg?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

- Duplications: Number of duplicated blocks of lines.

![](https://raw.githubusercontent.com/swiftingio/blog/%2326-Static-Code-Analyzers/Images/Duplications.jpg?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

- Complexity: It is the complexity calculated based on the number of paths through the code. 

![](https://raw.githubusercontent.com/swiftingio/blog/%2326-Static-Code-Analyzers/Images/Complexity.jpg?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

- Documentation: Number of lines containing either comment or commented-out code.

![](https://raw.githubusercontent.com/swiftingio/blog/%2326-Static-Code-Analyzers/Images/Documentation.jpg?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

What we can also see is:

- Technical debt:

Wikipedia: *"Technical debt is a concept in programming that reflects the extra development work that arises when code that is easy to implement in the short run is used instead of applying the best overall solution. Technical debt is commonly associated with extreme programming, especially in the context of refactoring"* 

In Sonar technical debt is calculated based on algorithm below:

```
Debt(in man days) = cost_to_fix_duplications + cost_to_fix_violations +
 	cost_to_comment_public_API + cost_to_fix_uncovered_complexity +
 	cost_to_bring_complexity_below_threshold
```

![](https://raw.githubusercontent.com/swiftingio/blog/%2326-Static-Code-Analyzers/Images/TechnicalDebt.jpg?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

##### Lessons learned 

+ SonarQube has a Web dashboard which can be easily shared and custiomized to your needs in development team
+ Has a lot of rules
+ Establish rules and their severity with your team
+ Can be really helpful in a situation when you work on a legacy code to check its quality
+ Can be integrated with continuous integration systems
+ A lot of plugins (SwiftLint)
+ But unfortunately it is quite expensive: 5,000 euro per year

If you would like to test SonarQube by yourself before buying it, you can visit [this demo website](https://sonarqube.com/overview?id=promisekit_swift?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

#### Checkmarx 

Checkmarx is a **security** platform providing several tools for introducing advanced static security analysis into applications written in C#, Java, jscript, native C/C++ or APEX. 

##### Basics

Checkmarx scanner interprets Swift to Objective-C in the backend before scanning the code. As a result, Checkmarx scans Swift code for over 60 quality and security issues, including twelve of the most severe and most common issues that cannot be left unfixed.

On Checkmarx website you can find a lot of interesting articles dedicated to mobile application, for example:

- [Mobile Health App Developers: FTC Best Practices](https://www.ftc.gov/tips-advice/business-center/guidance/mobile-health-app-developers-ftc-best-practices?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

- [40 Tips You Must Know About Secure iOS App Development](https://www.checkmarx.com/2015/11/10/40-tips-you-must-know-about-secure-ios-app-development/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

Checkmarx offers a Web dashboard on which you can find summary for all vulnerabilities found in your projects:

![](https://raw.githubusercontent.com/swiftingio/blog/%2326-Static-Code-Analyzers/Images/CheckMarx.jpg?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

##### Lessons learned

Checkmarx misses rules typical for Swift. I think that it is because Swift is a new and rapidly changing language. For sure there are still a lot of possibilities for defining new rules like they did for the others languages.

#### Other frameworks

At this moment there are not many different libraries for linting Swift. I have only found this one:

- [Tailor](https://github.com/sleekbyte/tailor?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

I hope that in the future we will see much more alternatives.

#### Summary

In my opinion static code analyzers can make your codebase more consistent, and for sure can really help during your daily coding and speed up the process. Of course we should remember about golden mean in using such libraries. It would be good not to turn our job into a nightmare by using very strict rules:)!
