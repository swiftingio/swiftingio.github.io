---
layout: post
author: kacper bartek
title: \#56 Swift Concurrency by examples.
excerpt: If you haven't tried the modern approach of handling swift concurrency in your application yet, here we are with a few examples. #AsyncAwait #SwiftConcurrency #AsyncStream #TaskGroup #Actors #SectionedFetchRequest

---

# Intro

If you haven't tried the modern approach of handling swift concurrency in your application yet, here we are with the example app which does the most popular things that your application for sure does and which may be helpful for you to see how the syntax looks like and how it works in action.

The application is based on the NASA API which allows displaying the APOD (Astronomic picture of the Day) using SwiftUI components. You can also add the pictures to locally stored Favorites.

<p float="left">
<img width="300" alt="Zrzut ekranu 2021-12-17 o 09 53 06" src="https://raw.githubusercontent.com/swiftingio/WWDC21/main/Images/list.png">
<img width="300" alt="Zrzut ekranu 2021-12-17 o 09 53 06" src="https://raw.githubusercontent.com/swiftingio/WWDC21/main/Images/details.png">
</p>

Please find the Github repo under this [link](https://github.com/swiftingio/WWDC21).

In the below sections we will give you also some more context for our code. So the examples below will rely on our app. But no worries, those examples do not require you to know the whole codebase :)


# Fetch data from the network using async/await.

The new async/await syntax is really straightforward. All you have to do is use a new API call on `URLSession` which is `let (data, response) = try await URLSession.shared.data(from: url)`. The await keyword is how you notice that a block of code doesn’t get executed as one transaction. The function may suspend, and other things may happen while it’s suspended between the lines of the function. More than that, the function may resume onto an entirely different thread.

The full function which fetches the APOD data in the Networking layer in our case looks like this:

```swift
public func fetchApods(startDate: Date, endDate: Date) async throws -> [ApodModel] {
// 1. Prepare url.
	let endpoint = ApodEndpoint.apods
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

Since our networking does also the parsing part (`try decoder.decode([ApodModel].self, from: data)`), we decided to put this function as a part of an `actor`, which is ok since from our observation now everything done within the function will be executed on a background thread, not main.

So the API overview of our networking layer looks like this:

```swift
public protocol ApodNetworking {
    func fetchApods(startDate: Date, endDate: Date) async throws -> [ApodModel]
	...
}

public actor DefaultApodNetworking: ApodNetworking {
	...
	public func fetchApods(startDate: Date, endDate: Date) async throws -> [ApodModel] {
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
        let objects = try await networking.fetchApods(
            startDate: endDate,
            endDate: currentDate
        )
        try await persistence.save(apods: objects)
    }
	...
}
```

*Now saving the data to the persistence after fetching them from the network is very clear for the reader without any nested closures. You read it top-down as it executes.*

# Actors

### What is an actor?

- Actors are a synchronization mechanism for the shared mutable state. It has its own state and that state is isolated from the rest of the program to ensure synchronized access to that data. In short: it gives us protection from concurrent access.

- Actors are a new kind of (reference) type in Swift, which can have properties, methods, initializers, subscripts, and so on.... but they do not support subclassing.

Is very important to distinguish two types of actors:

- MainActor (everything that is the main actor runs on the main queue)
- Actor (methods and properties are **not** accessed on the main queue)

### Actor

What could be an actor? Good candidates which can become actors are classes that possess logic not connected to the UI. So it can be for example part of the code responsible for:

- Networking
- Persistence
- Caching
- Etc.

Let's look on the example from our application:

```swift
public actor DefaultApodNetworking: ApodNetworking {
    let decoder: JSONDecoder
    let urlBuilder: URLBuilder
  
     public func fetchApods(count: Int) async throws -> [ApodModel] {}
     public func fetchApods(startDate: Date, endDate: Date) async throws -> [ApodModel] {}
     public func fetchImage(url: String) async throws -> UIImage {}
}
```

As above, if we define a new actor, that way we can make sure that all methods of that actor will be  **not** run on the main thread, the access to the properties either. Actor helps us to be thread-safe (which we cannot guarantee in the classes without using some special mechanism like GCD, semaphores, locks, etc.)

### @MainActor  

- The main actor is an actor that represents the **main** thread.
- The good practice would be to add `@MainActor` attribute to all observable object classes to be absolutely sure that all UI updates happen on the main thread

In our app we marked **ApodViewModel** as a MainActor: 

```swift
@MainActor class ApodViewModel: ObservableObject {
    @Published var thumbnails: [String: UIImage] = [:]

		...

```

From the architecture point of view, this is a good candidate because is an `ObservableObject`, and accessing properties of this class should be as fast as possible because `ApodViewModel` is the source for our UI. In your application for sure, you have very similar patterns that you have a view model which contains collections that need to be reloaded on the main thread when synchronization (from other actors) ends. For sure it can be a potentially good candidate for MainActor.

Probably in your application, you don't want to mark the whole class as the main actor, then you have two options:

- You can mark only the function with @MainActor, to be sure that the logic inside the functions will be executed always on the main thread
- or you can explicitly say that you want to run some part of the  function on the main thread, then you can use this pattern:

```swift
    await MainActor.run { 
        ....
    }
```

# Fetch thumbnails concurrently using AsyncStream and Task Group

Each APOD item on the list has its own image with a specific URL. Our goal was to trigger the fetch of all possible thumbnails at once in the background.

*NOTE: Please note that if had 1000 elements on the list, all of the images would be fetched at once so it is not the desired goal for the application. The mechanism for fetching should be triggered lazily (eg. fetching the nearest 20 elements depending on the scroll position). But as an experiment for the Task Group let's assume that it is completely fine :)*

In addition, when there is a new thumbnail fetched, we want to inform UI about it to refresh the view and present the loaded images on the screen.

The best candidates to fulfill those requirements are:

- `TaskGroup` allows us to create multiple concurrent tasks for fetching the thumbnails.
- `AsyncStream` allows us to receive the data when they are delivered one after another.

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

As a parameter, we take an array of ApodModels which contains the URLs for our images. Our function will return the `AsyncStream`, which will be delivering the tuple `(String, UIImage)` when the particular image is fetched (`String` will contain the image URL).

Now we create an `AsyncStream` using its initializer, which is given a `continuation` as a closure parameter, which we will use later on (more about [AsyncStream](https://developer.apple.com/documentation/swift/asyncstream)). In short, the continuation is needed to inform the stream about the received elements or when the stream is finishing its work.

Please note that `func getThumbnails()` is not an `async` function, because we would like to only bind the data and receive them later on. That's why all async code was wrapped as a `Task`. The `nonisolated` prefix gave a possibility for the function to synchronously create the `AsyncStream` in the code. Since we do not modify anything in terms of the actor's properties in the function, it is safe to expose the method as `nonisolated`.

And then the usage of the Task Group looks like below:

```swift
private func fetchThumbnails(
        from apods: [ApodModel],
        continuation: ThumbnailsStream.Continuation
    ) async throws {
        try await withThrowingTaskGroup(of: (String, UIImage).self) { group in
            for apod in apods.filter({ $0.media_type == .image }) {
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

As you may notice our view model (`ApodViewModel`) is responsible for fetching thumbnails and updating the UI when new thumbnails appear (via `@Published var thumbnails` property and `ObservableObject` conformance). It is marked as a `@MainActor`, which means that every update of the properties or actions within the functions will be called on the main thread. So in our case when the thumbnails are fetched - the UI will be updated from the main thread.

# SectionedFetchRequest

Now let's focus on an improvement and think how we could in a better way present the data by sectioning and improve searching by section sorting. WWDC 21 introduced a new API (SectionedFetchRequest) which can help us with dealing with this task. 

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
    public var apods: SectionedFetchResults<String, Apod>
    
```

As we see, **SectionedFetchRequest** takes two arguments:

- sectionIdentifier: which is a key path to the entity property 
- sortDescriptors: that takes an array of sort descriptors

After defining SectionedFetchRequest  is quite easy now to use [`SectionedFetchResults`](https://developer.apple.com/documentation/swiftui/sectionedfetchresults) property for defining a List view with sections as follows:

```swift
   List {
        ForEach(apods) { section in
             Section(header: Text(section.id)) {
                  ForEach(section) { apod in
                         ApodView(...
```

here in the example, each section has items for a specific day. But what if we want to do sectioning by week, month, year, etc. ? Before we will do this, we have to answer the follow-up question: if we can section items by property not defined in the entity? Fortunately, the answer is: yes.

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

What is really cool is that we can change the sort descriptor and sectionIdentifier dynamically. It gives the possibility to change sectioning in the application based on the user's choice. The only thing is to create the picker with a list of possible options (SortSelectionView) and then when the user chooses some option we reload the table with new sectioning by changing **sortDescriptors** and **sectionIdentifier** of the sectioned request.

```swift
SortSelectionView(
    selectedSortItem: $selectedSort,
    sorts: ApodSort.sorts
)
.onChange(of: selectedSort) { _ in
		let request = apods
    request.sortDescriptors = selectedSort.descriptors
    request.sectionIdentifier = selectedSort.section
}
```

Using SwiftUI `Menu` and `Picker` components we display the available sort options when user taps on a top-right toolbar button:

<p float="left">
<img width="300" alt="Zrzut ekranu 2021-12-17 o 09 53 06" src="https://raw.githubusercontent.com/swiftingio/WWDC21/main/Images/sorting.png">
</p>

# Summary

During creating the demo app we saw that we ended with no class (only actors and structs). What gives us ensurance that we are thread-safe. Of course, using actors doesn't get us an ensurance that we will not have performance problems, so still, you have to really pay attention to what you have in the main actor. We also encountered some performance issues, but then we realized that some part of the code needs to be moved from the main actor to other actors which solves the problem.

###Authors:

Bartłomiej Woronin,	contact: [email](mailto:bartlomiej.woronin@gmail.com), [Twitter](https://twitter.com/BWoronin).

Kacper Mazurkiewicz, contact: [email](mailto:kacper.mazurk@gmail.com), [Twitter](https://twitter.com/juniortjt1).

