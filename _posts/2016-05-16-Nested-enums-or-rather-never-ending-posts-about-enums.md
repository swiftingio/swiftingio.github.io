---
layout: post
author: bartek
title: \#15 Nested enums or rather never ending posts about enums:)
excerpt: 
---
Today I would like to discuss how to present *UITableViewCell* contents by using enums. A simple switch statement can help us with displaying data on our tableView. What scenario would I like to focus on?


![PlaceInfoView](https://raw.githubusercontent.com/swiftingio/blog/%2315-Nested-Enums/Images/InfoView.jpg)

To be more precise, I have created a simple schema of this screen:

![PlaceInfoScheme](https://raw.githubusercontent.com/swiftingio/blog/%2315-Nested-Enums/Images/PlaceInfoScheme.jpg)

By the way, have you had a chance to use [Sketch](https://www.sketchapp.com/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) for prototyping? Quite a nice tool!!!

The question is: what is the best proposal of enum configurations and functions, which can help us with displaying data when using ***tableView:cellForRowAtIndexPath:*** method for an example above?

My first proposal was:


```
enum PlaceInformation: Int { // Used on UISegmentControl
    case General
    case More
}

enum GeneralInfo: Int {//Cells for General Info Segment (Tab)
    case Description
    case HowToGoHere
    case Hotel
}

enum MoreInfo: Int {//Cells for More Info Segment (Tab)
    case CurrentWheather
    case InterestingPlacesNearby
}

class PlaceInformationViewController: UIViewController, UITableViewDataSource {
    
    var tableView: UITableView?
    
    override func viewDidLoad() {
        super.viewDidLoad()
        tableView = UITableView()
        ...
    }

    let selectedTabIndex = 0 //selected segment on UISegmentedControl
    
    func tableView(tableView: UITableView, cellForRowAtIndexPath indexPath: NSIndexPath) -> UITableViewCell {

        let cell = UITableViewCell()
        let row = indexPath.row
        let selectedInformationTab: PlaceInformation = PlaceInformation(rawValue: selectedTabIndex)!
        
        switch selectedInformationTab {
        case .General:
            guard let informationCellType = GeneralInfo(rawValue: row) else { break }
            switch informationCellType {
            case .Description:
                print("Setup Description Cell")
            case .HowToGoHere:
                print("Setup HowToGoHere Cell")
            case .Hotel:
                print("Setup Hotel Cell")
            }
        case .More:
            guard let informationCellType = MoreInfo(rawValue: row) else {  break }
            
            switch informationCellType {
            case .CurrentWheather:
                print("Setup Wheather Cell")
            case .InterestingPlacesNearby:
                print("Setup InterestingPlacesNearby Cell")
            }
        }
        return cell
    }

    func tableView(tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return 1
    }
}
```

Some statistics of the above code:

* Lines of code of ***tableView:cellForRowAtIndexPath:*** function: 28
* What did [SwiftLint](https://swifting.io/blog/2016/03/29/11-swiftlint/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) say?

![SwiftLint](https://raw.githubusercontent.com/swiftingio/blog/%2315-Nested-Enums/Images/SwiftLintStart.jpg)
	
* [Codebeat](https://swifting.io/blog/2016/03/21/10-is-christmas-earlier-this-year-code-quality-analyser-abc/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) rated the code to 3.97 [GPA](https://help.codebeat.co/docs/gpa-explained/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) (Function too long)

My intuition prompted that it can be done better:). So I have started thinking of how to refactor the code.

### First refactoring

So here we go with first refactoring. Below is my to do list to make this enum better:

* Remove white spaces according to SwiftLint
* Reduce cases in enum to one line
* Use nested enums 
* Necessarily I wanted to check if [enum associated values](https://developer.apple.com/library/ios/documentation/Swift/Conceptual/Swift_Programming_Language/Enumerations.html?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) can improve code metrics. I don't know why, but I hoped that it will improve my situation:) 

```
enum PlaceInformationCellModel {
    // returned cell depends on selectedTab and tableview section
    case Header(selectedTab: PlaceInformation, section: Int)

    enum PlaceInformation: Int { // Used on UISegmentControl
        case General
        case More
    }
    
    enum PlaceHeader {
        case Description, HowToGoHere, Hotel, CurrentWheather, InterestingPlacesNearby
        static let allGeneralHeaders = [Description, HowToGoHere, Hotel]
        static let allMoreHeaders = [CurrentWheather, InterestingPlacesNearby]
    }

    var value: PlaceHeader {
        switch self {
        case .Header(let tab, let sectionIndex):
            switch tab {
            case .General:
                return PlaceHeader.allGeneralHeaders[sectionIndex]
            case .More:
                return PlaceHeader.allMoreHeaders[sectionIndex]
            }
        }
    }
}

class PlaceInformationViewController: UIViewController, UITableViewDataSource {

    var tableView: UITableView?

    override func viewDidLoad() {
        super.viewDidLoad()
        tableView = UITableView()
        ...
    }
    
    let selectedTabIndex = 1 //selected segment on UISegmentedControl

    func tableView(tableView: UITableView, cellForRowAtIndexPath indexPath: NSIndexPath) -> UITableViewCell {
    
        let cellModel: PlaceInformationCellModel = PlaceInformationCellModel.Header(selectedTab: PlaceInformationCellModel.PlaceInformation(rawValue: selectedTabIndex)!, section: indexPath.row)
        let header = cellModel.value
        let cell = UITableViewCell()

            switch header {
            case PlaceInformationCellModel.PlaceHeader.Description:
                print("Setup Description Cell")
            case PlaceInformationCellModel.PlaceHeader.HowToGoHere:
                print("Setup HowToGoHere Cell")
            case PlaceInformationCellModel.PlaceHeader.Hotel:
                print("Setup Hotel Cell")
            case PlaceInformationCellModel.PlaceHeader.CurrentWheather:
                print("Setup CurrentWheather Cell")
            case PlaceInformationCellModel.PlaceHeader.InterestingPlacesNearby:
                print("Setup InterestingPlacesNearby Cell")
            }
        return cell
    }
    func tableView(tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return 1
    }
}
```

Is it better? I hoped so, so I have asked my friends about feedback:

* Lines of code of ***tableView:cellForRowAtIndexPath:*** function: 20

* What did SwiftLint say?

![SwiftLint](https://raw.githubusercontent.com/swiftingio/blog/%2315-Nested-Enums/Images/SwiftLintRefactorOne.jpg)

* What did codebeat say?: 4.00 GPA

#### Second refactoring

I still had a feeling that the code was too complicated. I thought that getting rid of associated values would make it simpler.

```
enum PlaceInformation {

    case General, More

    enum PlaceHeader: Int {
        case Description, HowToGoHere, Hotel, CurrentWheather, InterestingPlacesNearby
        static let allGeneralHeaders = [Description, HowToGoHere, Hotel]
        static let allMoreHeraders = [CurrentWheather, InterestingPlacesNearby]
    }

    var placeValues: [PlaceHeader] {
        switch self {
        case .General:
            return PlaceHeader.allGeneralHeaders
        case .More:
            return PlaceHeader.allMoreHeraders
        }
    }
}

class PlaceInformationViewController: UIViewController, UITableViewDataSource {

    var tableView: UITableView?

    override func viewDidLoad() {
        super.viewDidLoad()
        tableView = UITableView()
    }
    
   	let selectedInformationTab: PlaceInformation = .General

    func tableView(tableView: UITableView, cellForRowAtIndexPath indexPath: NSIndexPath) -> UITableViewCell {

        let header = selectedInformationTab.placeValues[indexPath.row]
        let cell = UITableViewCell()

        switch header {
        case .Description:
             print("Setup Description Cell")
        case .HowToGoHere:
            print("Setup HowToGoHere Cell")
        case .Hotel:
            print("Setup Hotel Cell")
        case .CurrentWheather:
            print("Setup CurrentWheather Cell")
        case .InterestingPlacesNearby:
            print("Setup InterestingPlacesNearby Cell")
        }
        return cell
    }
    func tableView(tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return 1
    }
}
```

Feedback:

* Lines of code of ***tableView:cellForRowAtIndexPath:*** function: 18
* SwiftLint:

![SwiftLint](https://raw.githubusercontent.com/swiftingio/blog/%2315-Nested-Enums/Images/SwiftLintrefactorTwo.jpg)

* Codebeat gave me 4.00 GPA for this code. [![codebeat badge](https://codebeat.co/badges/f6524e16-8679-444a-b14f-c12e47c0096f)](https://codebeat.co/projects/github-com-woroninb-enumtest?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

It seems that the easiest solution is the best:

![GitHub Logo](https://raw.githubusercontent.com/swiftingio/blog/%2315-Nested-Enums/Images/enum_gif.gif)

#### Third refactoring

I would like to leave the next iteration of refactoring for you. What would you change to improve the solution?

#### Resources

* [Advanced practical enum examples](https://appventure.me/2015/10/17/advanced-practical-enum-examples/??utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

* [Swift Enums. Apple documentation.](https://developer.apple.com/library/ios/documentation/Swift/Conceptual/Swift_Programming_Language/Enumerations.html??utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)
