---
layout: post
author: maciej
title: \#14 Code Review – Tips & Swifts
excerpt: 
---
Last week, in [issue \#13](https://swifting.io/blog/2016/04/18/13-code-review-are-we-too-busy-to-improve/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post), we introduced a definition of **Code Review**. Today we'll get deeper into the topic, present some tips & swifts & tricks when performing a review and focus on some mistakes found when reviewing a 🐦**Swift** code.

#### General Tips 🔍

It is said that a review goes best when conducted on less than 400 lines of code at a time. You should do a **proper** and slow review, however, don't spend on it more than 90 minutes 🕑 - you definitely will get tired after that time. It's tiring to write and understand your own code and it's even more tiring to understand someone's. Other tips from the Internet are:

After some years of experience in software development you know what common programming mistakes are. You also do realise what solutions to common problems are most efficient. It's a good practice to write it down and to check code being reviewed against a **checklist** ✅ - it leads improved results. Checklists are especially important for reviewers because, if the author forgets a task, the reviewer is likely to miss it too.

In the git flow mentioned in [issue \#13](https://swifting.io/blog/2016/04/18/13-code-review-are-we-too-busy-to-improve/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post), code can get merged after receiving agreed number of **approvals** for a pull request. The number should depend on your team size, but it's good to get at least 1️⃣ !😀

When you recommend fixes to the code, suggest their **importance** 💬. Maybe some of them could be postponed and extracted as separate tasks. Before approving a pull request, **verify** that defects are **fixed** 🔧🔨.

Foster in your company a **Good Code Review Culture** in which *finding* defects is viewed *positive*ly 😀. The point of software code review is to *eliminate* as many *defects* as possible, regardless of who "caused" the error. Few things feel better than getting **praise** from a *peer*. *Reward* developers for growth and effort. Offer as many *positive* comments as possible. I always try to put a line saying that a solution is good and clever (if it really is 😉).

You can also benefit from **The Ego Effect** 💎. The Ego Effect drives developers to *write better code* because they know that *others will be looking* at their code and their metrics.  No one wants to be known as the guy who makes all those *junior-level mistakes*. The Ego Effect drives developers to *review* their *own work* carefully before passing it on to others.

And **don't** bother with code formatting style ...

... there are much better things on which to spend your time, than arguing ☔️ over the placement of white spaces. 

Looking for *bugs* and *performance issues* is the primary goal and is where the majority of your time should be spent.

#### So, what to review?

Much better things to look for should be a part of your checklist. When I scrutinise a piece of code I look especially:

- at ```if``` conditions, ```for``` and ```while``` loops for *"off-by-one"* errors
- for interchanged ```<``` versus ```<=``` and ```>``` versus ```>=```
- <del>for accidental *interchange* of *&&* with *||* or bitwise operators like *&* and *|*</del> **UPDATED 11.06.2016** Swift compiler ensures that expression evaluated in a condition returns a ```Bool``` value, hence there's no possibility not to catch compilation error if one interchanges operators
- at *operators* importance and *execution* **UPDATED 11.06.2016** If you happen a "million" of operators in one expression you're probably doing something wrong by overcomplicating the solution... ;)
- if *dependancy injection* is possible (for the sake of unit testing).
- if variable *names* as *verbose* as needed
- if *method* names *express* what they do
- if a file contains *commented out* code
- if a file contains a single *class* (this one is arguable, depends on agreement with your team and company's guidelines)
- for code *duplication*, *function length* and *class length* < x lines - [SwiftLint](https://github.com/realm/SwiftLint?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) or [codebeat](https://codebeat.co?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) mentioned in  [issue \#11](https://swifting.io/blog/2016/03/29/11-swiftlint/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) and [issue \#10](https://swifting.io/blog/2016/03/21/10-is-christmas-earlier-this-year-code-quality-analyser-abc/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) is an option to point out such a code without your participation

When performing a review, one can also get extra points 💎 for pointing out a more *efficient implementation* of used algorithm 😉.

You should also check for **readability**. Is the code easy to understand? Do you have to pause frequently during the review to decipher it?


#### Some 🐦swifts

Boom! Time for some loosely coupled Swift code examples that I look for when reviewing Swift code❗️Wait a moment,  this exclamation mark looks strangely familiar ... ❗️Look out for code that uses it ...
```Swift
var foo: Object!
print("\(foo.someString)")
```
Make sure that your peers (and you as well) use the ```force unwrapping``` operator wisely in their code.

If you have ever ended up with code like the one below, then probably something went wrong ... 💣💥

```Swift
override init(_ objectId: String?, 
viewModel1: ViewModel1, 
viewModel2: ViewModel2, 
viewModel3: ViewModel3, 
viewModel4: ViewModel4, 
viewModel5: ViewModel5, 
viewModel6: ViewModel6, 
viewModel7: ViewModel7, 
viewModel8: ViewModel8, 
viewModel9: ViewModel9, 
viewModel10: ViewModel10,
isSomething: Bool = false) { ... }
```

This is an excerpt from from one of my projects, object types and names are changed . It's an initialiser of a 'container' view model that encapsulates 'child' view models used by a container view controller. If you happen to achieve this and there's no way to change it, probably usage of a configurator *facade*, *default argument* values or *convenience initialisers* are the best solution to live up with your legacy.

```Swift
class Configurator {
    class func mainViewController() -> MainViewController {
        let dependency1 = Configurator.dependency1()
        let dependency2 = Configurator.dependency2()
        let dependency3 = Configurator.dependency3()
        let dependency4 = Configurator.dependency4()

        return MainViewController(dependency1: dependency1,
            dependency2: dependency2,
            dependency3: dependency3,
            dependency4: dependency4)
    }
}
```

The *off-by-one* errors and interchanged *greater than* & *less than* were my favourite mistakes at some point of time. I've caught myself a few times last year with using ```if i > array.count``` instead of ``` if i < array.count``` ☔️.

Remember local ```autoreleasepool```? For those who answered 'what!?' [here](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/MemoryMgmt/Articles/mmAutoreleasePools.html?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post#//apple_ref/doc/uid/20000047-SW2) is some explanation. I pay attention when reviewing body of a loop to check if local *autoreleasepool* could be used to reduce peak memory footprint.

```Swift
for i in 1...100
autoreleasepool {
	NSString *superImportantString = "Super \(i)"
	//strange operations take place
	//superImportantString can be released earlier thanks to local autoreleasepool
}
```

More than a year ago, when Swift was still in its early days, I hated when my team members overused ```var```. It's ok when needed, but not all the time! Thanks god that now Swift compiler warns about not mutated ```var```. 

Another matter is your object's properties. If it depends on some values, never ever use ```var```. Use ```let```. Always value more *immutable* state over ```var```! I wonder how many people would argue with me about that☔️😉. And if your property really have to be mutable, expose it 'to the public' as immutable like that:

```Swift
private(set) var foo: AnyObject
let bar: AnyObject
```

Swift's *protocols* are a great deal - do you prefer *composition* over inheritance? Is such an inheritance chain ok for you? One superclass would be probably ok, but this becomes an exaggeration:

```Swift
class ChildViewModel: TableViewModel {}
class TableViewModel: CRUDViewModel {}
class CRUDViewModel: BaseViewModel {}
class BaseViewModel {}
```

#### Final thoughts
Kofi Annan, Ghanaian ex-Secretary-General of the United Nations, when in primary school, was attending a weekly lesson in 'spoken English'. He and his peers were asked by the teacher what did they see at the picture:

![](https://raw.githubusercontent.com/swiftingio/blog/%2313-Code-Review--are-we-too-busy-to-improve/black-dot.jpeg?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

All shouted together, that they saw a black dot! Professor  stepped back and said 'So, not a single one saw the white sheet of paper. (...). This is the awful thing about human nature. People never see the goodness of things, and the broader picture. (...)'.

It's also true for code review. We perceive better *differences* than loads of new code when using a merge tool. We get a cognitive *overload* if flooded with more content. Have that in mind and use the first rule - review for less than 90 minutes 🕑, take breaks. Remember that:

- new code is more difficult to *understand*
- *not all bugs* can be caught during a review
- *hot fix*es 🔩 without review can happen ... 😉

And **always** - Review your own code first! Before committing a code *look through all changes* you have made!

![](https://raw.githubusercontent.com/swiftingio/blog/%2313-Code-Review--are-we-too-busy-to-improve/noNeedToLook.png?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

Let us know in comments or on [Twitter](https://twitter.com/swiftingio?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) what is on your checklists for code review!

#### The only *true measure* of code quality

![](https://raw.githubusercontent.com/swiftingio/blog/%2313-Code-Review--are-we-too-busy-to-improve/wtfsPerMin.jpg?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

#### Additional links

- [11 proven practices for peer review by IBM](http://www.ibm.com/developerworks/rational/library/11-proven-practices-for-peer-review/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)
- [Code Review Best Practices by Kevin London](http://kevinlondon.com/2015/05/05/code-review-best-practices.html?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)
- [Best Practices: Code Reviews by MSDN](https://msdn.microsoft.com/en-us/library/bb871031.aspx?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)
