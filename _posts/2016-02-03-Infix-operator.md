---
layout: post
author: bartek
title: \#3 Infix operator
excerpt: 
---
Have you ever had a chance to see code similar to the one below? 

```
infix operator +++ {associativity left precedence 100 }
func +++ (letf: X, right: X) {
	return ...
}
```
and you haven't really known what all these words mean? 

Perfect☺  
Let’s decode all keywords and make some simple examples to explain everything!

#### But before we do that ...

Above code is used to define new custom operators in Swift. These operators can have no existing meaning defined in Swift, like in example **+++**. New operators are declared at the global level, so they are visible from everywhere in our code. After this short introduction we can start. so! what does __infix__ mean?

#### Infix

Infixed operators are for example: plus, minus, multiplication, etc... They just take a left and right hand argument: 

* These are infixed operators:  2 __+__ 2,  2 __*__ 2, 2 __/__ 2
* These aren't: 2 __--__ , __++__ 2 

Easy :) 

#### Operator

Keyword __operator__ has to always appear before definition of our custom set of characters.

#### +++, ---, |||, ????, ...., and much more other options

The best explanation about which characters we can use to create our custom operator you can find [here](https://developer.apple.com/library/ios/documentation/Swift/Conceptual/Swift_Programming_Language/AdvancedOperators.html?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

And finally two magic parameters:

#### Associativity 

This parameter says how operators of the same precedence are grouped in the absence of parentheses: from left or from right. If operator is left associative, then we go from the left to the right:

(a o b) o c 

And if it is right associative, then:

a o (b o c)

 **Example with left assocativate operator**

```
var x = 12 / 6 / 2 
```
Becesuse __/__ is left associativate it is evaluated as following: : 

```
var x  = (12  / 6 ) / 2     // 1
``` 

If it were right associative then it would be evaluated as:

```
var x = 12  / ( 6  / 2 )     // 4 
```

 **Example with right assocativate operator**
 
```
var x = 5
x += 2 
```
It means that at the beginning 5 will be increased by two, and then value will be assigned to x, so operation will be evaluated from right to left:

```
x = (5 + 2) 
```

#### Precedence

Operator precedence gives some operators higher priority than others and these operators are applied first. For example operator __*__ is applied before __+__ , because it has higher precedence.

That’s why 2 + 2 * 2 = 6 not 8 ☺ So simple! :)

#### Default values

| parameter | possible Values | default value  |
| :--------- |:---------| :-----|
| associativity | left, right, and none | none |
| precedence  |  0-255 | 100 |

To feel better these parameters, below are some examples of existing operators in Swift with their precedence and associativity values:

| Operator  |  precedence | associativity |
| :--------|:-------| :------|
| + | 140| left |
| *  |  150 | left |
| / | 150| left |
| += |  90 | right |
| == |  90 | none |

Precedence and associativity can be only defined for infix operators. There is no reason to use their for prefix and postfix operators, so we can simply write: 

```
prefix operator +++ {}
```

#### Summarizing

In this post I presented only the basic clue of custom operators in swift but I hope that now you feel stronger in infix operators subject and you are ready to create your own operators:) If you want to expand your operators knowledge, then definitely visit the following websites: 

* all posibble operators available in Swift you can find on [swift library](https://developer.apple.com/library/ios/documentation/Swift/Reference/Swift_StandardLibrary_Operators/index.html?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post#//apple_ref/doc/uid/TP40016054)
* of course you can't miss [advanced operators](https://developer.apple.com/library/ios/documentation/Swift/Conceptual/Swift_Programming_Language/AdvancedOperators.html?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) topic
* great post about operators in Swift you can find on [NSHipster](http://nshipster.com/swift-operators/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)
* and finally irreplaceable wikipedia with definitin of [operator associativity](https://en.wikipedia.org/wiki/Operator_associativity?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

Have a nice day!:)
