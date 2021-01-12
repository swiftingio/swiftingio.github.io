---
layout: post
author: bartek
title: \#11 SwiftLint
excerpt: 
---
Should the opening brace of a function or control flow statement be on a new line or not ?:) This and many other questions cross my mind when I think about coding style. I love the comment from Ray Wenderlich's [Swift style guide](https://github.com/raywenderlich/swift-style-guide/issues/44?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post): 

*"You like braces on new lines @elephantronic ? This will strain our friendship!"* ...hehe, so it really does matter!:)

But coming back to the main subject: This year one of the items on my to do list is a task to learn more about static code analysis tools: linters, Sonar and everything else that can fight against bad smells. In our [previous post](https://swifting.io/blog/2016/03/21/10-is-christmas-earlier-this-year-code-quality-analyser-abc/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) Maciej mentioned more about code quality and about [Codebeat](https://codebeat.co/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post). Today I would like to concentrate on [SwitLint](https://github.com/realm/SwiftLint/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post), which can help us keep cohesion in our code without straining friendships in our project team:) 

SwiftLint checks the source code for programmatic as well as stylistic errors. This is most helpful in identifying some common and uncommon mistakes that are made during coding. SwiftLint is based on guidelines from [Swift style guide](https://github.com/github/swift-style-guide?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post). Simply saying it can just helps us with:

* maintaining a higher level of code discipline, 

* increasing the reliability of the code.


#### Installation

* using brew:

```
$ brew install swiftlint
```

* or by downloading and installing a fresh [release.](https://github.com/realm/SwiftLint/releases?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

Necessary after installation check version using command ```swiftlint version``` and compare it to the latest version available on website.

### Running

We have two options:

* just type the following command in the Terminal:

```
$ swiftlint lint 
```

* or we can integrate SwiftLint with Xcode:

![RunScript](https://raw.githubusercontent.com/swiftingio/blog/%2311-SwifLint/Images/RunScript.jpg?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

Just for the test I have run SwiftLint with default options in my current project and I wish I had been using it from the beginning of the project &#128517; : 

![SwiftLintRun](https://raw.githubusercontent.com/swiftingio/blog/%2311-SwifLint/Images/SwiftLintRun.jpg?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

#### SwiftLint Rules 

All available rules which you can use are [here](https://github.com/realm/SwiftLint/tree/master/Source/SwiftLintFramework/Rules?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post).

If you want to have a possibility to control which rule is disabled/enabled and to set thresholds for warnings and errors for a given rule, just create a **.swiftlint.yml** file in your project directory: 

![swiftlintYML](https://raw.githubusercontent.com/swiftingio/blog/%2311-SwifLint/Images/swiftlintYML.jpg?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

You can download the example configuration file from [here](https://github.com/swiftingio/blog/blob/%2311-SwifLint/.swiftlint.yml?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post).
 
Now by typing in the Terminal ```swiftlint rules``` you can check your settings:

![SwiftLintRules](https://raw.githubusercontent.com/swiftingio/blog/%2311-SwifLint/Images/SwiftLintRules.jpg?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

#### Running Options

Sometimes running options can be very useful. Let’s focus on them:

* ```-- path``` The path to the file or directory. For example if you want to lint only one file: 

```
$ swiftlint autocorrect --path just_this_file.swift
```

* ```-- config``` The path to SwiftLint's configuration file *.swiftlint.yml:* 

```
$ swiftlint lint --config .swiftlint.yml 
```

* ```-- use-script-input-files```  read SCRIPT_INPUT_FILE* environment variables as files. When using this parameter can be useful? For example when we only would like to lint modified files and new files tracked in our github repository. Example of usage which I found [here](https://www.snip2code.com/Snippet/1100061/Run-SwiftLint-only-on-untracked-modified/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post):

```
if which swiftlint >/dev/null; then
	count=0
	for file_path in $(git ls-files -om --exclude-from=.gitignore | grep ".swift$"); do
		export SCRIPT_INPUT_FILE_$count=$file_path
		count=$((count + 1))
	done
	for file_path in $(git diff --cached --name-only | grep ".swift$"); do
		export SCRIPT_INPUT_FILE_$count=$file_path
		count=$((count + 1))
	done
	
	export SCRIPT_INPUT_FILE_COUNT=$count

	swiftlint lint --use-script-input-files
else
  echo "warning: SwiftLint not installed, download from https://github.com/realm/SwiftLint"
fi
```
```
- git ls-files: outputs the filenames in working directory
- o: (others: Show other (i.e. untracked) files in the output) 
- m: (modified) Show modified files in the output
- exclude-from=.gitignore: it omits ignored files 
- grep ".swift$" : search only .swift files
```

* ```-- quiet```  flag that prevents status messages like 'Linting ' & 'Done linting' from being logged.

* ```-- reporter``` generates report with selected format: JSON, Checkstyle, CSV, Xcode. For example:

```
$ swiftlint lint --reporter json
```
```
{
    "reason" : "Opening braces should be preceded by a single space and on the same line as the declaration.",
    "character" : 23,
    "file" : "../ViewController.swift",
    "rule_id" : "opening_brace",
    "line" : 50,
    "severity" : "Warning",
    "type" : "Opening Brace Spacing"
  },
  ...
```

#### Autocorrect

Very nice feature of SwiftLint is the auto-correction ```swiftlint autocorrect```, which can automatically fix violations in your code. What type of violations can we fix?

* Closing brace. Closing brace with closing parenthesis should not have any whitespaces in the middle:

![closingBrace](https://raw.githubusercontent.com/swiftingio/blog/%2311-SwifLint/Images/ClosingBraceViolation.jpg?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

* Colon. Colons should be next to the identifier when specifying a type:

![colonRule](https://raw.githubusercontent.com/swiftingio/blog/%2311-SwifLint/Images/ColonViolation.jpg?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

* Comma Spacing. There should be no space before and one after any comma:

![commaRule](https://raw.githubusercontent.com/swiftingio/blog/%2311-SwifLint/Images/CommaViolation.jpg?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

* Legacy Constant. Struct-scoped constants are preferred over legacy global constants:

![legacy_constant](https://raw.githubusercontent.com/swiftingio/blog/%2311-SwifLint/Images/LegacyConstantViolation.jpg?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

Very nice [article](https://medium.com/swift-programming/swift-cgrect-cgsize-cgpoint-5f4196da9cf8?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post#.aum1xrdz9) explaining the **why**.

* Legacy Constructor. Swift constructors are preferred over legacy convenience functions:

![legacy_constructor](https://raw.githubusercontent.com/swiftingio/blog/%2311-SwifLint/Images/LegacyContstructorViolation.jpg?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

* Opening Brace Spacing. Opening braces should be preceded by a single space and on the same line as the declaration:

![opening_brace](https://raw.githubusercontent.com/swiftingio/blog/%2311-SwifLint/Images/OpeningBrace.jpg?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

* Statement Position. Else and catch should be on the same line, one space after the previous declaration:

![statement_position](https://raw.githubusercontent.com/swiftingio/blog/%2311-SwifLint/Images/StatementViolation.jpg?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

* Trailing Newline. Files should have a single trailing newline:

![trailing_newline](https://raw.githubusercontent.com/swiftingio/blog/%2311-SwifLint/Images/TrailingNewLineViolation.jpg?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

* Trailing Semicolon. Lines should not have trailing semicolons:

![trailing_semicolon](https://raw.githubusercontent.com/swiftingio/blog/%2311-SwifLint/Images/SemicolonViolation.jpg?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

* Trailing Whitespace. Lines should not have trailing whitespace:

![trailing_whitespace](https://raw.githubusercontent.com/swiftingio/blog/%2311-SwifLint/Images/TrailingWhiteSpaceViolation.jpg?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

#### Cyclomatic Complexity

In SwiftLint you can find very nice rules, for example:

* File length rule: Files should not span too many lines.
* Function Parameter Count: Number of function parameters should be low.
* Type Body Length: Type bodies should not span too many lines.
* Type Name Rule: Type name should only contain alphanumeric characters, start with an uppercase character and span between 3 and 40 characters in length.

But when I first looked at the rules list my attention was captured by the [Cyclomatic Complexity](https://github.com/realm/SwiftLint/blob/master/Source/SwiftLintFramework/Rules/CyclomaticComplexityRule.swift?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) rule. What is this? Wiki says:

*"is a software metric (measurement), used to indicate the complexity of a program. It is a quantitative measure of the number of linearly independent paths through a program's source code."*

This parameter just says how complex our functions are. To many ifs are not recommended. If you add 3rd nested *if*, you should wait a moment and reconsider your solution. This is my weakness, because sometimes I propose really overcomplicated solutions for quite easy problems: it can always be done much simpler... great rule for using by me:) How to add this bodyguard to your config file? Just add it on the end on *.swiftlint.yml*:

```
cyclomatic_complexity:
  warning: 2 # two nested ifs are acceptable
  error: 5   # six nested ifs shows warning, 6 causes compile error
```

#### Summary

I think that the use of such a tool from the start of the project can be very useful. By setting up together configuration file with considering parameters for each rule, you can be sure that everyone will have to use the same style and for sure it will decrease the amount of bad smells.

For sure I will user this tool in my next project! Hope to hear about your experience with using SwiftLint or similar tools!:) I can't wait for your comments!

Don't forget to visit also:

* [SwitLint on GitHub](https://github.com/realm/SwiftLint?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)
* [Little bites of Cocoa: Enforcing coding style and conventions with SwiftLint](https://littlebitesofcocoa.com/149-enforcing-coding-style-and-conventions-with-swiftlint?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)
* [SwiftLint's Issue 413 on GitHub](https://github.com/realm/SwiftLint/issues/413?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)
