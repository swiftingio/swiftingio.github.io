---
layout: post
author: maciej
title: \#10 Is Christmas earlier this year? – code quality analyser & ABC
excerpt: 
---
#### Code quality - what to use? 🔧🔨
There are a few ways we can ensure better code quality of our projects. Starting from unit testing then going through lint-like programs, code formatters, static code analysers, code reviews and ending with all that stuff I forgot to mention ... ;) The most demanding and also worth applying, in my opinion, are unit tests and code reviews, but we can get pretty much hints on code that should be improved from static code analysers. 

This issue will briefly show some tools I use on a daily basis to assure good quality of developed code, the main focus though is put in particular on a free web-tool for Swift open-source projects and the **ABC** that drove me a bit crazy recently. If you are not interested in code quality tools and just would like to get to know what the **ABC** is and why it got me crazy go straight to the **Codebeat** section.

![](https://raw.githubusercontent.com/swiftingio/blog/%2310-ABC/CodeQuality.png?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

#### Unit Tests ❤️
Probably every developer had a problem at the beginning of applying unit tests in their workflow. What to test, how to test and many others were the questions probably everyone has asked themselves... The easiest way for me to learn and understand unit testing was to take part in a training on that topic. If you happen to be from Poland you can check out Mobile Academy's [offer](http://mobile-academy.io?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) on the matter (pss... some workshops are free!). There are also good materials regarding testing on objc.io's [issue #15](https://www.objc.io/issues/15-testing/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post). 

##### A few hints on what to use for unit tests

You can tap into the standard ```XCTest``` framework to perform testing, however I encourage you to try out the BDD-like style and test interactions between objects instead of method implementation. There's a great framework for that in Swift called [Quick](https://github.com/Quick/Quick?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) - I use ```QuickSpec``` objects to group related tests with a valid description. It's more natural for me to tap into such framework than to use standard XCTest's approach. I also use [Nimble](https://github.com/Quick/Nimble?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) matchers, which is a counterpart for Quick, for code assertions:

```
context("ViewController") {
    //...
    describe("when view loads") {
        beforeEach {
            sut.viewDidLoad()
        }
        it("does sth") {
            expect(sut.didSomething).to(beTrue())
        }
    }
}
```

If you happen to use a CI server then [xctool](https://github.com/facebook/xctool?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) might be a good selection to run tests for you because it can translate XCTest report into *jUnit* output that is understood by such tools as e.g. [Bamboo](https://www.atlassian.com/software/bamboo?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post).  Some time ago I also  had to write a tool that interprets report from *xctool*'s JSON format and converts it into human - friendly output. The *xctool* had been a great help in doing that!

#### Git Flow with code reviews 🔩
Do you tap into code reviews in your projects? You should be! Imagine outsourcing development of project features to another developer or company. Their work lasts for two months or so. Spaghetti source code gets delivered to you. You wish you had reviewed it when they had been developing it. You wish your company didn't lose money because of that. You wish you didn't have to start from a scratch with those features, cause their code is a mess ... It happened a few times to me. This is why Git Flow - development on feature branches, pull requests and code reviews became a normal process for my team. You can check out [GitHub guide](https://guides.github.com/introduction/flow/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) to understand how it works. Btw., can anyone explain to us what that squirrel represent 🔮?

#### SwiftLint 
I was searching for a good Swift code-formatting plugin for Xcode, however it seems there aren't any at the moment. Then I realised that [SwiftLint](https://github.com/realm/SwiftLint?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) has the `autocorrect` option to fix some of code-format-related issues.

I've added the SwiftLint to a run script for each Xcode build for my project, but suddenly I realised that it generated over 3000 issues. Thanks to this autocorrect option it went down to more than 1500. It was still to much and we don't have resources to fix all of them. To silent them I had to disable some of SwiftLint's rules in the ```swiftlint.yml``` configuration file. The tool is nice, however I encourage you to **use it from the very beginning** than embedding it in project after almost a year of development :)!!!
 
#### Code analysers
Have you tried any of those? Some of them we get with Xcode, for some of them we have to pay, but probably the greatest are those we get for free :)! 

- [clang](http://clang.llvm.org?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) - contains a  standard static code analyser used by Xcode. Press ⌘⇧B to check out what happens! (If nothing happened it probably means that your code doesn't have any issues. Or that something went wrong. Could be both ;) )
- [SonarQube](http://www.sonarsource.com/products/plugins/languages/swift/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) - is an open source tool, but you have to pay loads of 💰 for a language plugin (5k EUR per plugin per year), and it's difficult to setup
- [codebeat](https://codebeat.co?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) - is really Christmas 🎄🎉🎁 earlier this year? 

#### Codebeat
is at the moment my favourite code analyser. Comparing it to other analysers I feel as if Christmas were earlier this year because we can use this tool for **free** for **open source** projects. Additionaly, it's easy to setup (just login and add a link to your GitHub repository and the analysis begins), has a clean & nice UI and what's more, it of course supports Swift📱!  Check how it works [here](https://help.codebeat.co/docs/how-is-codebeat-different?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post). 

The analysis takes into consideration a few simple metrics that are explained in details in their [guide](https://help.codebeat.co/docs/software-quality-metrics?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) .  Here are short excerpts about each of them:

- **Cyclomatic complexity** - the number of linearly independent paths within a section of a code. The lower the number of paths the better.
- **Lines of code** - refers to non-commentary lines, meaning pure whitespace and lines containing only comments are not included in the metric. Again, less lines in a file means better.
- **Arity** - number of arguments that a function takes. Functions with longer lists of parameters are more difficult to use and are more difficult to test.
- **Number of return values (Go only)**  - this one is at the moment valid for Go language only. In Swift it could be applied to tuples returned from functions. Maybe in the future.
- **Maximum block nesting** - calculates how deeply nested the deepest statement in a single function is. 
- **Code duplication** - the DRY (Don't Repeat Yourself) Principle states that every piece of knowledge must have a single, unambiguous, authoritative representation within a system. Avoid copy-pastes ;)!

and ...
 
#### The freaking ABC that drove me crazy
- **Assignment Branch Condition (ABC)**  - represents the size of the source code from a structural point of view. It is computed by counting the number of assignments, branches and conditions for a given section of code.
 
In the ABC, an **Assignment** is understood as an explicit transfer of data into a variable, i.e. by using any variation of ```=``` operator, a **Branch** is understood as a function or method call, whereas a **Condition** is of course a logical (or boolean) test, e.g. ```==``` or ```switch```.

A scalar ABC metric is computed as:

```
|ABC| = sqrt((A*A)+(B*B)+(C*C))
```

The measure can be used to indicate how much work a piece of code does. Shorter procedures without many execution paths are desired because they're simpler to understand and easier to test. But let's get back for a moment to the *Assignment* part of the metric: *an explicit transfer of data into a variable*...

Imagine you had a view controller with three views that were not initialised from a Storyboard or Xib file. You would end up with a code in ```init``` or ```viewDidLoad``` similar to this one:

```
init() {
        imageView = UIImageView(frame: CGRectZero)
        imageView.translatesAutoresizingMaskIntoConstraints = false
        imageView.contentMode = .ScaleAspectFill
        imageView.clipsToBounds = true
        imageView.accessibilityIdentifier = "image"
        imageView.isAccessibilityElement = true

        nameLabel = UILabel(frame: CGRectZero)
        nameLabel.translatesAutoresizingMaskIntoConstraints = false
        nameLabel.numberOfLines = 0
        nameLabel.backgroundColor = UIColor.whiteColor()
        nameLabel.textColor = UIColor.darkGrayColor()
        nameLabel.accessibilityIdentifier = "name"
        nameLabel.isAccessibilityElement = true

        ratingLabel = UILabel(frame: CGRectZero)
        ratingLabel.translatesAutoresizingMaskIntoConstraints = false
        ratingLabel.numberOfLines = 1
        ratingLabel.backgroundColor = UIColor.whiteColor()
        ratingLabel.textColor = UIColor.darkGrayColor()
        ratingLabel.accessibilityIdentifier = "rating"
        ratingLabel.isAccessibilityElement = true

        super.init(nibName: nil, bundle: nil)
}
```

I usually initialise views in the ```init``` in such a way, because I prefer ```let``` over ```var view: UIView!``` and having focused over distributed setup of objects. The other thing is that I cannot invoke any instance method to initialise a view before calling ```super.init```. Because of that every time I set a property in a variable the ABC metric for that function increases. For one of my old projects I ended up with a metric of size ```abc = 23.02``` for ```init``` of a view controller. It was very frustrating since a yellow sad face appeared on the list of my projects. I wanted to fix it. It kept staring at me:

![](https://raw.githubusercontent.com/swiftingio/blog/%2310-ABC/SadFace.png?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)  

Firstly, I have contacted the support team in order to ask them why an assignment to a property of an object increases the ABC. Thanks to them I have realised that the ABC is about any assignment, not just about too many assignments to a particular variable. The less assignments in a method the better. Knowledge++ ;)! 
    
#### Improving the ABC metric
Then I focused on how to solve the problem. I have performed a small test to check which of the approaches have the least ABC metric. 

##### #1
First approach was to use the standard initialisation in the ```init``` method. It's put in here just for reference, to check how bad the aforementioned stand-alone code behaves. It got the ```abc=21.54```. A bit better than the one from my previous project, because over there I had a few more assignments. Grab the report [here](https://codebeat.co/projects/github-com-paciej00-testviewcontroller0?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) and check out the source code [here](https://github.com/paciej00/TestViewController0/blob/master/ABC/TestViewController0.swift?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post).

##### #2
Second test was with a usage of [Then](https://github.com/devxoul/Then?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) library. It's a syntactic sugar that allows to set object's parameters in a non-escaping closure just after object's initialisation:

```
imageView = UIImageView().then { imageView in
        imageView.translatesAutoresizingMaskIntoConstraints = false
        imageView.contentMode = .ScaleAspectFill
        imageView.clipsToBounds = true
        imageView.accessibilityIdentifier = "image"
        imageView.isAccessibilityElement = true
}
```

In fact, the [code](https://github.com/paciej00/TestViewController1/blob/master/ABC/TestViewController1.swift?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) was still present in the ```init``` method. The metric in the [report](https://codebeat.co/projects/github-com-paciej00-testviewcontroller1?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) increased to ```abc=22.83```. Not good ...

##### #3
The 3rd test went well. In fact it went very well. I cannot extract from the [report](https://codebeat.co/projects/github-com-paciej00-testviewcontroller2?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) the exact metric I have achieved, but I totally changed the approach. Now views are lazily initialised when used for the first time in the [code](https://github.com/paciej00/TestViewController2/blob/master/ABC/TestViewController2.swift?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post), when added to their superview:

```
lazy var imageView: UIImageView = {
        let imageView = UIImageView(frame: CGRectZero)
        imageView.translatesAutoresizingMaskIntoConstraints = false
        imageView.contentMode = .ScaleAspectFill
        imageView.clipsToBounds = true
        imageView.accessibilityIdentifier = "image"
        imageView.isAccessibilityElement = true
        return imageView
}()
```

##### #4
Next [solution](https://github.com/paciej00/TestViewController3/blob/master/ABC/TestViewController3.swift?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) again used ```lazy var``` but this time named functions to initialise variables:

```
func setupImageView() -> UIImageView {
        let imageView = UIImageView(frame: CGRectZero)
        imageView.translatesAutoresizingMaskIntoConstraints = false
        imageView.contentMode = .ScaleAspectFill
        imageView.clipsToBounds = true
        imageView.accessibilityIdentifier = "image"
        imageView.isAccessibilityElement = true
        return imageView
}
```

I had to use currying though. It's sad that currying probably won't be present in the next Swift release, I like it very much ❤️:

```
lazy var imageView: UIImageView = setupImageView(self)()
lazy var nameLabel: UILabel = setupNameLabel(self)() 
lazy var ratingLabel: UILabel = setupRatingLabel(self)()
```
No metric in the [report](https://codebeat.co/projects/github-com-paciej00-testviewcontroller3?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) for this approach, but no issue with ABC reported.

##### #5
The 5th attempt also didn't cause any issues in the [report](https://codebeat.co/projects/github-com-paciej00-testviewcontroller4?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post). This time I went back to initialising views in the ```init``` method, however I used a ```Configurator``` object to set up views for me (remember that you cannot invoke any instance method before initialising super). Grab the full code [here](https://github.com/paciej00/TestViewController4/blob/master/ABC/TestViewController4.swift?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post).

```
init() {
        //imageView = setupImageView()
        //nameLabel = setupNameLabel()
        //ratingLabel = setupRatingLabel()
        imageView = Configurator.setupImageView()
        nameLabel = Configurator.setupNameLabel()
        ratingLabel = Configurator.setupRatingLabel()

        super.init(nibName: nil, bundle: nil)
}
```

##### #6
The last test didn't change much, again I still have three assignments, use ```setup*View``` method but I wanted to show an approach when doing it inside the ```viewDidLoad``` method. Surprise, surprise - ABC not an issue in the [report](https://codebeat.co/projects/github-com-paciej00-testviewcontroller5?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post). Full code [here](https://github.com/paciej00/TestViewController5/blob/master/ABC/TestViewController5.swift?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post), an excerpt below:

```
override func viewDidLoad() {
        super.viewDidLoad()
        imageView = setupImageView()
        ratingLabel = setupRatingLabel()
        nameLabel = setupNameLabel()
        view.addSubview(imageView)
        view.addSubview(ratingLabel)
        view.addSubview(nameLabel)
}
```

#### Summary
It isn't difficult to invent a few attitudes in making methods simpler. I think I like the approach with the ```Configurator``` object the most, because I prefer immutable properties whenever possible. Thanks to [codebeat](https://codebeat.co/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) I got a food for thought and I could ponder about the solution that suits me best. 

What are your favourite tools for code quality? Share your thoughts in comments or on our Twitter [account](https://twitter.com/swiftingio?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)!

PS. I was also asked by the **codebeat** support team if I thought that the ABC metric should be relaxed for Swift language. After performing my tests, I think it's ok the way it is.

PS. They [change](https://www.youtube.com/watch?v=z_KmNZNT5xw&feature=youtu.be&t=38s&utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) their algorithms from time to time. 
