---
layout: post
author: kacper bartek
title: \#56 First steps with Swift Concurrency.
excerpt: If you haven't tried the modern approach of handling swift concurrency in your application yet, here we are with a few examples. #AsyncAwait #SwiftConcurrency #AsyncStream #TaskGroup #Actors #SectionedFetchRequest

---

# Intro

If you haven't tried the modern approach of handling swift concurrency in your application yet, here we are with the example app which does the most popular things that your application for sure does and which may be helpful for you to see how the syntax looks like and how it works in action.

The application is based on the NASA API which allows  displaying the APOD (Astronomic picture of the Day) using SwiftUI components. You can also add the pictures to locally stored Favorites.

<p float="left">
<img width="300" alt="Zrzut ekranu 2021-12-17 o 09 53 06" src="https://raw.githubusercontent.com/swiftingio/WWDC21/main/Images/list.png">
<img width="300" alt="Zrzut ekranu 2021-12-17 o 09 53 06" src="https://raw.githubusercontent.com/swiftingio/WWDC21/main/Images/details.png">
</p>

Please find the github repo under this [link](https://github.com/swiftingio/WWDC21).

In the below sections we will give you also some more context for our code. So the examples below will rely on our app. But no worries, those examples does not require you to know the whole codebase :)


# Fetch data from the network using async/await.

The new async/await syntax is really straightforward. All you have to do is use a new API call on `URLSession` which is `let (data, response) = try await URLSession.shared.data(from: url)`. The await keyword is how you notice that a block of code doesn’t get executed as one transaction. The function may suspend, and other things may happen while it’s suspended between the lines of the function. More than that, the function may resume onto an entirely different thread.

The full function which fetches the APOD data in the Networking layer in our case looks like this:

```swift
public func fetchApody(startDate: Date, endDate: Date) async throws -> [ApodModel] {
// 1. Prepare url.
	let endpoint = ApodEndpoint.apody
	let parameters = [
       ApodParameter.startDate(startDate),
       ApodParameter.endDate(endDate),
       ApodParameter.apiKey,
    ]

	let url = try urlBuilder.build(endpoint: endpoint, parameters: parameters)
// 2. Fetch data.
	let (data, response) = try await URLSession.shared.data(from: url)
	guard let httpResponse = response as? HTTPURLResponse, 
		httpResponse.statusCode == 200 else {
			throw ApodNetworkingError.invalidServerResponse
	}
// 3. Parse received data.
	let parsedData = try decoder.decode([ApodModel].self, from: data)

	return parsedData
}

```

As we used the `try await URLSession.shared.data(from: url)` we had to mark also the function as `async` which would inform the compiler that this method is asynchronous. 

Since our networking do also the parsing part (`try decoder.decode([ApodModel].self, from: data)`), we decided to put this function as a part of an `actor`, which is ok since from our observation now everything done within the function will be executed on a background thread, not main.

So as the API overview of our networking layer looks like this:

```swift
public protocol ApodNetworking {
    func fetchApody(startDate: Date, endDate: Date) async throws -> [ApodModel]
	...
}

public actor DefaultApodNetworking: ApodNetworking {
	...
	public func fetchApody(startDate: Date, endDate: Date) async throws -> [ApodModel] {
	...
	}
}
```

And the usage of it is like below:

```swift
@MainActor class ApodViewModel: ObservableObject {
	...
    private let networking: ApodNetworking

    public func refreshData() async throws {
        ...
        let objects = try await networking.fetchApody(
            startDate: endDate,
            endDate: currentDate
        )
        try await persistence.save(apody: objects)
    }
	...
}
```

*Now saving the data to the persistence after fetching them from the network is very clear for the reader without any nested closures. You read it top-down as it executes.*

# Actors

### What is an actor ?

- Actors are a synchronization mechanism for shared mutable state. It has its own state and that state is isolated from the rest of the program to ensure synchronized access to that data. In short: it gives us protection from concurrent access.

- Actors are a new kind of (reference) type in Swift, which can have properties, methods, initializers, subscripts, and so on.... but they do not support subclassing.

Is very importnat to distinguish two types of actors:

- MainActor (everything what is main actor runs on the main queue)
- Actor (methods and properties are **not** accessed on the main queue)

### Actor

What could be the an actor? Good candidates which can become actors are classes which possess logic not connected to the UI. So it can be for example part of code responsible for:

- Networking
- Persistence
- Caching
- Etc.

Let's look on the example from our application:

```swift
public actor DefaultApodNetworking: ApodNetworking {
    let decoder: JSONDecoder
    let urlBuilder: URLBuilder
  
     public func fetchApody(count: Int) async throws -> [ApodModel] {}
     public func fetchApody(startDate: Date, endDate: Date) async throws -> [ApodModel] {}
     public func fetchImage(url: String) async throws -> UIImage {}
}
```

As above, if we define a new actor, that way we can make sure that all methods of that actor will be  **not** run on the main thread, the access to the properties either. Actor helps us to to be thread safe (which we cannot guarantee in the classes without using some special mechanism like: GCD, semaphores, locks, etc.)

### @MainActor  

- The main actor is an actor that represents the **main** thread.
- The good practice would be to add `@MainActor` attribute to all observable object classes to be absolutely sure that all UI updates happen on the main thread

In our app we marked **ApodViewModel** as a MainActor: 

```swift
@MainActor class ApodViewModel: ObservableObject {
    @Published var thumbnails: [String: UIImage] = [:]

		...

```

From the arhitecture point of view this is a good candidate because is an `ObservableObject` and accessing properties of this class should be as fast as possible because `ApodViewModel` is the source for our UI. In your application for sure you have very similar patterns that you have view model which contains collecions which needs to be reloded on the main thread when synchronization (from other actors) ends. For sure it can be potentailly good candicate for MainActor.

Probably in your application you don't want to mark whole class as a main actor, then you have two options:

- You can mark only the function with @MainActor, to be sure that the logic inside the fuctions will be executed always on the main thread
- or you can explicitly say that you want to run some part of function on the main thread, then you can use this pattern:

```swift
    await MainActor.run { 
        ....
    }
```

# Fetch thumbnails concurrently using AsyncStream and Task Group

Each APOD item on the list has its own image with a specific url. Our goal was to trigger the fetch of all possible thumbnails at once in the background.

*NOTE: Please note that if had 1000 elements on the list, all of the images would be fetched at once so it is not the desired goal for the application. The mechanism for fetching should be triggered in a lazy way (eg. fetching nearest 20 elements depending on the scroll position). But as an experiment for the Task Group let's assume that it is completely fine :)*

In addition, when there is a new thumbnail fetched, we want to inform UI about it to refresh the view and present the loaded images on screen.

The best candidates to fulfill those requirements are:

- `TaskGroup` which allows us to create mutliple concurrent tasks for fetching the thumbnails.
- `AsyncStream` which allows us to receive the data when they are delivered one after another.

So first of all - we start with creating the new data type which will perform all of this. Once again `actor` is a great choice since all of those things are not user-facing, so it should be done in the background. Let's call it `ThumbnailDataSource`.

```swift
public actor ThumbnailDataSource { }
```

Great, now we define the function which will start fetching the thumbnails.

```swift
public actor ThumbnailDataSource { 
	public typealias UrlAsString = String
	public typealias ThumbnailsStream = AsyncStream<(UrlAsString, UIImage)>

	public nonisolated func getThumbnails(models: [ApodModel]) -> ThumbnailsStream {
		return ThumbnailsStream { continuation in
			Task {
				try await fetchThumbnails(from: models, continuation: continuation)
			}
		}
	}
}
```

As a parameter we take an array of ApodModels which contains the urls for our images. Our function will return the `AsyncStream`, which will be delivering the tuple `(String, UIImage)` when the particular image is fetched (`String` will contain the image URL).

Now we create an `AsyncStream` using its intializer, which is given a `continuation` as a closure parameter, which we will use later on (more about [AsyncStream](https://developer.apple.com/documentation/swift/asyncstream)). In short the continuation is needed to inform the stream about the received elements or when the stream is finishing its work.

Please note that `func getThumbnails()` is not an `async` function, because we would like to only bind the data and receive them later on. That's why all async code was wrapped as a `Task`. The `nonisolated` prefix gave a possibility for the function to synchronously create the `AsyncStream` in the code. Since we do not modify anything in terms of the actor's properties in the function, it is safe to expose the method as `nonisolated`.

And then the usage of the Task Group looks like below:

```swift
private func fetchThumbnails(
        from apody: [ApodModel],
        continuation: ThumbnailsStream.Continuation
    ) async throws {
        try await withThrowingTaskGroup(of: (String, UIImage).self) { group in
            for apod in apody.filter({ $0.media_type == .image }) {
                group.addTask(priority: .background) {
                    (apod.url, try await self.fetchImage(url: apod.url))
                }
            }
            for try await data in group {
                continuation.yield(data)
            }
            continuation.finish()
        }
}
```

Once added a child task to a group (by calling `group.addTask()`), child tasks are being executed immediately and in any order.

The `continuation` property yields the result every time the group task finishes its fetch. Once all of the data are fetched, we call `conitnuation.finish()` to inform that the stream has finished publishing data.

The binding in the parent would look like this:

```swift
@MainActor class ApodViewModel: ObservableObject {
 
	@Published var thumbnails: [String: UIImage] = [:]

	private let dataSource: ThumbnailDataSource
	...
	public func fetchThumbnails(for models: [ApodModel]) async {
        for await (url, image) in dataSource.getThumbnails(models: models) {
            thumbnails[url] = image
        }
    }
	...
}
```

Great, now every time we call `fetchThumbnails(for models: [ApodModel])`, all of the thumbnails for the passed ApodModels will be fetched in the background, by delivering fetched images one after another and populate the thumbnails dictionary.

As you may notice our view model (`ApodViewModel`) is responsible for fetching thumbnails and updating the UI when new thumbnails appears (via `@Published var thumbnails` property and `ObservableObject` conformance). It is marked as a `@MainActor`, which means that every update of the properties or actions within the functions will be called on the main thread. So in our case when the thumbnails are fetched - the UI will be updated from the main thread.

# SectionedFetchRequest

Now let's focus on an improvement and  think how we could in a better way present the data by sectioning and improve searching by section sorting. WWDC 21 introduced a new API (SectionedFetchRequest) which can really help us with dealing this task. 

### About 

```swift
@propertyWrapper struct SectionedFetchRequest<SectionIdentifier, Result> where SectionIdentifier : Hashable, Result : NSFetchRequestResult
```

-  is a property wrapper type that retrieves entities, grouped into sections, from a Core Data persistent store

- use a `SectionedFetchRequest` property wrapper to declare a [`SectionedFetchResults`](https://developer.apple.com/documentation/swiftui/sectionedfetchresults) property that provides a grouped collection of Core Data managed objects to a SwiftUI view

- by using SectionedFetchRequest SwiftUI gives automatic support for collapsing sections

### SecionedFetchRequest in action in our demo app

SectionedFetchRequest is used in our ListView for fetching Apod's from our Persistence layer (Core Data):

```swift
    @SectionedFetchRequest(
        sectionIdentifier: \Apod.date,
        sortDescriptors: [SortDescriptor(\Apod.date, order: .reverse)]
    )
    public var apody: SectionedFetchResults<String, Apod>
    
```

As we see, **SectionedFetchRequest** takes two arguments:

- sectionIdentifier: which is a key path to the entity property 
- sortDescriptors: that takes an array of sort descriptors

After defining SectionedFetchRequest  is quite easy now to use [`SectionedFetchResults`](https://developer.apple.com/documentation/swiftui/sectionedfetchresults) property for defining a List view with sections as follow:

```swift
   List {
        ForEach(apody) { section in
             Section(header: Text(section.id)) {
                  ForEach(section) { apod in
                         ApodView(...
```

here in the example, each section has items for specific day. But what if we want to do a sectioning by week, month, year, etc. ? Before we will do this, we have to answer for the follow up question: if we can section items by property not defined in the entity? Fortunatelly the answer is: yes.

### Sectioning by dynamic property 

Let's say that we want to have section per week or month (which are not enity properties), then we can add dynamic properties to the **Apod** entity, like for example:

```swift
    @objc var month: String {
        return date.formatted(.dateTime.month(.wide))
    }

    @objc var week: String {
        return date.formatted(.dateTime.month(.wide).week())
    }
```

Then we can use those properties as a section identifiers:

```swift
    @SectionedFetchRequest(
        sectionIdentifier: \Apod.week,
        sortDescriptors: [SortDescriptor(\Apod.date, order: .reverse)]
    )
```

What is really cool is that we can change the sort descriptor and sectionIdentifier in a dynamic way. It gives possibility to change sectioning in the application based on the user choice. The only thing is to create the picker with a list of possible options (SortSelectionView) and then when user choose some option we reload table with a new sectioning by changing **sortDescriptors** and **sectionIdentifier** of the sectioned request.

```swift
SortSelectionView(
    selectedSortItem: $selectedSort,
    sorts: ApodSort.sorts
)
.onChange(of: selectedSort) { _ in
		let request = apody
    request.sortDescriptors = selectedSort.descriptors
    request.sectionIdentifier = selectedSort.section
}
```

Using SwiftUI `Menu` and `Picker` components we display the available sort options when user taps on a top-right toolbar button:

<p float="left">
<img width="300" alt="Zrzut ekranu 2021-12-17 o 09 53 06" src="https://raw.githubusercontent.com/swiftingio/WWDC21/main/Images/sorting.png">
</p>

# Summary

During creating demo app we saw that we ened with no class (only actors and structs). What gives us ensurance that we are thread safe. Of course, using actors dosen't get us an ensurance that we will not have performance problems, so still you have to really pay attention on what you have in the main actor. We  also encountered some performance isues, but then we realized that some part of the code needs to be moved from main actor to other actors what solves the problem.
