---
layout: post
author: kacper
title: \#56 Async await - try it on your own!
excerpt: WWDC21 new stuff template. 

---

### Intro

If you haven't tried the modern approach of handling concurrency in your application yet, here we are with the example application which does the most popular things that your application for sure will also do and which may be helpful for you to see how the syntax looks like and how it works in action.

The application is based on the NASA API which allows for displaying the APOD (Astronomic picture of the Day) in a SwiftUI List, which you can also add to locally stored Favorites.

Please find the github repo under the link below:


In the below sections we will give you also some more context for our code. So the examples below will strongly relay on the whole codebase. But no worries, examples does not require you to know the whole codebase :)


### 1. Fetch data from the network using async/await.

The new async/await syntax is really straightforward. All you have to do is use a new API call on `URLSession` which is `let (data, response) = try await URLSession.shared.data(from: url)`. The await keyword is how you notice that a block of code doesn’t execute as one transaction. The function may suspend, and other things may happen while it’s suspended between the lines of the function. More than that, the function may resume onto an entirely different thread.

The full function which fetches the APOD data in the Networking layer in our case looks like this:

```
public func fetchApods(startDate: Date, endDate: Date) async throws -> [ApodModel] {
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

        guard let httpResponse = response as? HTTPURLResponse, httpResponse.statusCode == 200 else {
            throw ApodNetworkingError.invalidServerResponse
        }
    // 3. Parse received data.
        let parsedData = try decoder.decode([ApodModel].self, from: data)

        return parsedData
}

```

As we used the `try await URLSession.shared.data(from: url)` we had to mark also the function as `async` which would inform the compiler that this method is asynchronous. 

Since our networking do also the parsing part (`try decoder.decode([ApodModel].self, from: data)`), we decided to put this function as a part of an `actor`, which is ok because now we are sure that everything done within the function will be done in the background Thread, not main.

So as an overview our networking layer looks like this:

```
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

```
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

*Curious about the `ApodViewModel` implementation? Read the next section :)*

### 2. Fetch thumbnails concurrently using AsyncSequence and Task Group.

Each APOD item on the list has its own image with a specific url. Our goal was to trigger the fetch all possible thumbnails at once in the background (I know, it is not the best usecase, but we still wanted to use the Task Group somewhere :)). 

In addition, when there is a new thumbnail fetched, we want to inform UI about it to refresh the view and present the loaded images on screen.

The best candidates to fulfill those requirements are:

- `Task Group` which allows us to create mutliple concurrent tasks for fetching the thumbnails.
- `AsyncSequence` which allows us to receive the data when they are delivered one after another.

So first of all - we start with creating the new data type which will perform all of this. Once again `actor` is a great choice since all of those thing are not user-facing, so it should be done in background. Let's call it `ThumbnailDataSource`.

```
public actor ThumbnailDataSource { }
```

Great, now we define the function which will start fetching the thumbnails.

```
public typealias ThumbnailsStream = AsyncStream<(String, UIImage)>

public nonisolated func getThumbnails(models: [ApodModel]) -> ThumbnailsStream {
        return ThumbnailsStream { continuation in
            Task {
                try await fetchThumbnails(from: models, continuation: continuation)
            }
        }
}
```

As a parameter we take the ApodModels which contains the url for our images. Our function will return the `AsyncStream`, which will be delivering the tuple `(String, UIImage)` when the particular image is fetched (`String` will contain the image URL).

Now we create the `AsyncStream` using its intializer, which is returning the `continuation` as a closure parameter, which we will use later on (please find more about continuations in here (link)). In short continuation is needed to inform the stream about the received elements or when the stream is finishing its work.

As the work within the stream will be done concurrently we can use the `Task` syntax to wrap the async work since our desired `fetchThumbnails` function is `async`.

Please note that `func getThumbnails()` is not an `async` function, because we would like to only bind the data and receive them later on.
`nonisolated` prefix gave a possibility for the function to synchronously create the AsyncStream in the code. Since we do not modify anything in terms of the actors properties in the function, it is safe to expose the method as `nonisolated`.

And then the usage of Task Group looks like below:

```
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

In the func `fetchThumbnails` the continuation property yields the result every time the group task finishes its fetch. Once all of the data are fetched, we call `conitnuation.finish()` to inform that the stream has finished publishing data.

The binding in the parent would look like this:

```
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
```

Our ViewModel (`ApodViewModel`) is responsible for fetching thumbnails and updating the UI when new thumbnails appears (via `@Published var thumbnails` property and `ObservableObject` conformance). It is marked as a `@MainActor`, which means that every update of the properties or actions within the functions will be called on the main thread. So in our case when the thumbnails are fetched - the UI will be updated from the main thread.


