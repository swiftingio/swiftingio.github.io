---
layout: post
author: maciej
title: \#32 Errorify String
excerpt: 
---
#### Swift.Error
Swift introduces pattern of throwing errors, to propagate error conditions in a program. Errors are represented by types that conform to empty `Error` protocol, usually `enum` types.

```
enum AppError: Error {
	case type1
	case type2
	//..
	case typeN
}
```

If you want to propagate an error condition in your app, you can use your `AppError` enum for that:

```
func bar() throws {
	if shouldThrowError() {
		throw AppError.type1
	}
}
```

The `throws` keyword says that a function can propagate an error, so it should be used with special care. All calls to the method have to be marked with `try` keyword and embedded in `do{} catch{}` block:

```
do {
	try bar()
} catch {  
	print("\(error)")
}
```

There is one dirty trick to get rid of `do{} catch{}` block. You can fore `try!` the method that throws:

```
	try! bar()
```

❗️Remember: force `try!` causes your app to crash in the case your method actually propagates an error. In order to call the method seamlessly you can use `try?` to tell the compiler you're not interested in the error.

```
	try? bar()
```

In Swift a method can be marked with `rethrows` keyword. It means that it throws an error only if one of its arguments throws an error.

```
func bar(callback: () throws -> Void) rethrows {
    try callback()
}
```

#### Named just Error
You may want to name your `Error`-conforming type simply `Error`. In order to do that, you have to point out that your `Error` conforms to protocol defined in standard Swift library: 

```
enum Error: Swift.Error {
	case type1
	case type2
	//..
	case typeN
}
```

#### Errorify String
Some time ago I was watching [episode 11](https://talk.objc.io/episodes/S01E11-evaluating-expressions?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) of [Swift Talk](https://talk.objc.io?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) and I noticed that [Florian](https://twitter.com/floriankugler?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) and [Chris](https://twitter.com/chriseidhof?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) are using `String` to throw an error.

```
throw "Something went wrong!"
```

I thought "Cool, I didn't know you can do such things in Swift!". So I typed it in my code, and I was disappointed ...

![](https://raw.githubusercontent.com/swiftingio/blog/%2332-Errorify-String/StringError.png?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

In order for the mechanism to work, I had to **Errorify** `String` 😉:

```
extension String: Error {}
```

#### TL;DR;

```
extension String: Error {}
func bar() throws {
	throw "Something went wrong!"
}
```
