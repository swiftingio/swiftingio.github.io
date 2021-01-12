---
layout: post
author: maciej
title: \#13 Code Review – are we too busy to improve?
excerpt: 
---
[Issue \#10](https://swifting.io/blog/2016/03/21/10-is-christmas-earlier-this-year-code-quality-analyser-abc/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) has had a very good perception by the community. For this reason we will blog about **code quality** topics from time to time. Today we'd like to discuss **code review** process, challenges one may encounter when introducing it in their workplace and what to consider when reviewing one's source code.

#### What is a code review?

The good old [Wikipedia](https://en.wikipedia.org/wiki/Code_review?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) once wrote:

```
Code review is systematic examination (often known as peer review) of computer source code.

It is intended to find and fix mistakes overlooked in the initial development phase,  

improving both the overall quality of software and the developers' skills. 
```
Far enough. I want to do it! 💥Wait, but ...

#### How to do it?

Reviews can be done in various forms such as pair programming, informal walkthroughs, and formal inspections. The most known is probably this one - **show me your code** (aka  informal review)**!** Simply ask your peer to have a look at your code. 

![](https://raw.githubusercontent.com/swiftingio/blog/%2313-Code-Review--are-we-too-busy-to-improve/sideBySide.jpg?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

Formal code reviews can be performed by using e.g.  Atlassian's Crucible/Fisheye (I don't like this tool, bad UX, hard to navigate and even start a review) or a pull request on Stash/BitBucket/GitHub or whatever you use.  Thanks to those tools you get a nice visualisation of what changes were made to source code, you can comment on them, ask author some questions and they can explain their code in return. It's like a conversation you would have in real life, but documented - what has been agreed should be performed before merging into develop. Wait, what merging?! We were just talking about a review ...
 
#### Git flow

When developing different parts of an application at work we use Git and Git Flow to merge all changes into a parent branch. Once a feature of an app is finished we create a pull request that contains changes to be added to a predecessor branch (usually *develop*). The best picture showing the flow comes, in my opinion, from [GitHub](https://guides.github.com/introduction/flow/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post). I still don't get what the squirrel on it represents... ⁉️

![](https://raw.githubusercontent.com/swiftingio/blog/%2313-Code-Review--are-we-too-busy-to-improve/git-flow.jpg?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

The flow goes like this:

- create a branch from e.g. *develop*
- apply your changes to the source code
- create a pull request to be merged into e.g. *develop*
- discuss changes with your peers, explain your point of view & apply suggested improvements
- a squirrel⁉️  approves  your changes (or your peers, depends on the setup of your team and agreements, could be a squirrel in some cases ... 😉)
- merge your code into source branch
 
#### What to consider when doing a review?

You definitely should check code integrity - does the style match previous solutions, does it follow agreed conventions? Are features implemented correctly, does the old source code work correctly after changes? 
 
#### Why to tap into code review in your development process?

For sure because it ensures code integrity 😀, catches what others' eyes didn't see. It allows to learn and share your knowledge and expertise, strengthens communication in the team and builds good relationships due to conversations about what you have in common - code and programming skills ;)!

Unfortunately, from my experience, unit tests and code review are first things that get removed from project scope 😢. It takes too much time and there's no budget for that. Do you encounter the same problems?

Consider code review as an investment into the future. If you don't catch bugs now you will definitely have to conquer them in the future.
 
#### What if you didn't perform code reviews 

Imagine a company X. It delivers mobile app solutions for multiple clients. Their team is small and at some point they cannot meet clients' demands. So they decide to outsource some of the work to an external company. They give project requirements to this company and meet again after 3 months to receive app source code. But the app is useless - it freezes at network calls, `CoreData.ConcurrencyDebug` flag crashes all the time. The project is delayed for a few months, team has to start from a scratch. The wish they had reviewed outsourced code on a daily basis...

#### How to *start* the process at your place?
 
Code review culture wasn't common for mobile teams at my workplace. It still isn't for some of the due to rapid development and short lifecycle of some of the mobile apps. However, I wanted to improve myself by learning from others. I've started with a *prank*. I've developed  changes on a *branch* and have created a *pull request* for my colleagues. And it got the ball rolling. Now all mobile projects within my division embed code review in their development process.

#### Do you want to perform code reviews?

This is usually the question that gets **lack of enthusiasm** from the audience :(. But really, are we too busy to improve??

![](https://raw.githubusercontent.com/swiftingio/blog/%2313-Code-Review--are-we-too-busy-to-improve/toobusy.png?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

This was an introduction to code review. In the upcoming weeks we'll get deeper into the process. Stay tuned 📻😀!
