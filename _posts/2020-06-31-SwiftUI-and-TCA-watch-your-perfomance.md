---
layout: post
author: kacper
title: \#55 Swift UI & The Composable Architecture - watch your performance (5 rules worth applying).
excerpt: I would like to present some helpful tips that might help you keep your app performance at the high level and ultimately improve the user experience when working with SwiftUI and TCA. 

---

### Intro

SwiftUI allows us to create simple app views in a well-structured way. But If you create more complex apps you may notice that vanilla state management - `@Environment`, `@EnvironmentObject`, `@ObservableObject` etc. is becoming quite complex to follow. Especially if we embed more and more views downstream the views hierarchy. The more views - the more states and possible changes that may occur in the different situations. The more states - the more state-change-triggers and more state-observers which are hard to manage. This could lead to long sessions of debugging of unexpected situations and many unhandled problems. [TCA (The Composable Architecture)](https://github.com/pointfreeco/swift-composable-architecture) is a great marriage for SwiftUI. It introduces unidirectional data flow and also simplifies the composition of the states, so following the state changes is not a problem anymore.

However, from the hands-on experience I noticed that there are a couple of places which we have to be careful when working with SwiftUI and TCA in terms of the app performance. Especially for a large number of views that react to the state changes. As a result your app may be laggy, freeze or even crash. I would like to present some helpful tips that might help you keep your app performance at the high level and ultimately improve the user experience.

### 1. Dispatch heavy operations to the background thread

Yes, I know. It is quite obvious that devs have to off-load the main thread as much as possible in order to keep the good performance. However I have a feeling that in the SwiftUI and TCA world it is sometimes not obvious how and where we can improve it. Some of the places are even overlooked because we got used to some practices that we saw in tutorials/articles etc. I would like to emphasize two following fields that are worth keeping in mind when developing or even doing code review for others.

#### a) Avoid calculations in the SwfitUI’s View

You don’t have to use TCA to experience this problem. A SwiftUI’s View represents part of your app’s user interface. Initialization of the view and all of the changes that happen in the `body` runs on the main thread. Keep in mind that any calculations performed in the initializer of the view or calling internal functions and calculated properties within the `var body { }` will be performed on the main thread. 

Below I am presenting a **bad example** of the SwiftUI View that suffers because of the described problem (especially if this view will be reloaded very often):

```
struct MyView: View {
   ....
    var calculatedProperty: CGFloat {
        var result: CGFloat = 0
        // Imagine some heavy operations here
        // ...
        return result
    }
        
    init(
        data: ViewData,
    ) {
        self.viewData = data
        // #1 Transformations done in the view initializer.
        self.transformedData = data.map { viewData in TransformedData(viewData) }
        // #2 Calling heavy functions in the initializer.
        self.calculatedData = performSomeCalculations()
    }
 
    var body: some View {
        ChildView(transformedData: transformedData,
                  calculatedData: calculatedData,
                  // #3 Everytime body reloads, this calculation is performed.
                  property: calculateSomethingWhenBodyReloads(),
                  // #4 The same goes for dynamically calculated properties.
                  property2: calculatedProperty)
    }
    
    func performSomeCalculations() -> Int {
        return viewData
            .enumerated()
            .map { data in
                // Some heavy calculations while iterating through the collection.
                // ...
            }
    }
    
    func calculateSomethingWhenBodyReloads() -> CGFloat {
        var result = 0
        // Another heavy operations may appear here
        // ...
        return result
    }
}

```

You might think - “I did it multiple times and it works”. Trust me, if you would have more complex views with several child subviews nested and created in the way presented above - sooner or later you will notice performance drops. 

Better option would be to move all of the calculations somewhere else and dispatch to the background thread. The question is - where to move the code from the functions and calculated properties? The answer is **TCA**.

We can declare TCA elements in the following way:

```
struct MyViewData {
    public struct State: Equatable {
        var viewData: [ViewData]
        var transformedData: [TransformedData]
        // ...
        // Other needed data
    }
    
    public enum Action: Equatable {
        case updateState(with: [ViewData])
    }
    
    public static let reducer = Reducer<State, Action, Void> {
        state, action, _ in
        switch action {
        case let .updateState(viewData):
            state.transformedData = viewData.map { viewData in TransformedData(viewData) }
            	// ...
            	// other heavy calculations moved from MyView.
            return .none
        }
    }
}

```

Our View looks much more cleaner with the TCA store injected and all things moved out:

```
struct MyView: View {
    let store: Store<MyViewData.State, MyViewData.Action>
        
    init(store: Store<MyViewData.State, MyViewData.Action>) {
        self.store = store
    }
 
    var body: some View {
        WithViewStore(store) { viewStore in
            ChildView(transformedData: viewStore.transformedData,
                      calculatedData: viewStore.calculatedData,
                      property: viewStore.property,
                      property2: viewStore.property2)
        }
    }
}

```

Ok we’ve made our code cleaner, however the performance problem still exists. Here we came to the second part of the problem...

#### b) Do not put heavy operations directly into the reducer

In the documentation excerpt of the The Composable Architecture we can read that: "*All interactions with an instance of Store (including all of its scopes and derived ViewStores) must be done on the same thread. If the store is powering a SwiftUI or UIKit view then, all interactions must be done on the main thread.*"

So unfortunately **our reducer is not declared** properly since everything defined below has to be performed on the main thread.

```
...
  public static let reducer = Reducer<State, Action, Void> {
        state, action, _ in
        switch action {
        case let .updateState(viewData):
            state.transformedData = viewData.map { viewData in TransformedData(viewData) }
            //... 
			// and other heavy calculations moved from the example 1 
			// will be performed on the main thread..
            return .none
        }
    }
...

```
 **Solution**

Let’s create the Environment within the `MyViewData` struct, where we can define our function that performs those heavy operations. As a result of the function we can run an `Effect`* so in the later usage we could easily dispatch the execution to the proper queue. 

**`Effect` type encapsulates a unit of work that can be run in the outside world, and can feed data back to the `Store`. It is the perfect place to do side effects, such as network requests, saving/loading from disk, creating timers, interacting with dependencies, and more.* [(source)](https://pointfreeco.github.io/swift-composable-architecture/Effect/)


```
 public struct Environment {
      	  // It is good practice to define the main and background queue as follows. 
      	  // It is easier to write unit tests in the future.
        var mainQueue: AnySchedulerOf<DispatchQueue>
        var backgroundQueue: AnySchedulerOf<DispatchQueue>

        func performCalculations(with viewData: [ViewData]) -> Effect<StateUpdate, Never> {
            Effect.run { effect in
            // Everything that you would like to do on the background queue, put here.
           	 let transformedData = viewData.map { viewData in TransformedData(viewData: viewData) }
           	 // some other heavy calculations
           	 let update = StateUpdate(
               	 transformedData: transformedData,
              	   ...
           		)
               effect.send(update)
               effect.send(completion: .finished)
			}
        }
    }

```

As you can see we’ve also created a separate struct (`StateUpdate`) just to keep our calculated results somewhere. The rest of the code is quite simple. Let’s add another action which will assign the results to the state.

```
public enum Action: Equatable {
	case updateState(with: [ViewData])
	case finishUpdate(with: StateUpdate)
}

```

Then let’s modify the reducer like this:

```
 public static let reducer = Reducer<State, Action, Environment> {
 	state, action, environment in
        switch action {
        case let .updateState(viewData):
            return environment.performCalculations(with: viewData)
                .subscribe(on: environment.backgroundQueue)
                .receive(on: environment.mainQueue)
                .eraseToEffect()
                .map { update in Action.finishUpdate(with: update) }
        		   // …
        case let .finishUpdate(with: stateUpdate):
            state.calculatedData = stateUpdate.calculatedData
            state.transformedData = stateUpdate.transformedData
		    // …
			return .none
        }
}
```

Thanks to the `.subscribe(on: environment.backgroundQueue)` calculations in the `performCalculations(with: viewData)` will be executed on the background queue. After receiving the result we dispatch back to the mainQueue to update the state. 

Following above TCA and SwiftUI pattern allows you to:

* keep `View` struct much cleaner and with less code,
* have state of the view under control (along with the queues),
* have centralized place of handling actions.

### 2. Take care of the equality checks of the State.

Rephrasing the statement quoted in 1b) - "*If the store is powering a SwiftUI or UIKit view then, all interactions must be done on the main thread.*". This creates another very important problem. TCA’s `State` has to conform to the `Equatable` protocol (you may notice that in the previous examples). It means that every time we perform state changes - the equality checks are performed underneath to recalculate it to see whether to reload its `WithViewStore` body or not. Unfortunately, those equality checks are also performed on the main thread. For the lightweight `State` it won’t be a noticeable problem. However at some point you will notice that the `State` could keep more complex structures. For example an array of thousands of elements which contains multiple properties for the collection view. In this case the equality checks would be a huge pain when it comes to the state updates on the main thread.

What can we do to improve this? We can prepare a custom implementation for the equality check to make it more efficient and faster. 

Depending on the case it may turn out that comparing only elements count or specific property is sufficient:

```
public struct State: Equatable {
        var transformedData: [TransformedData]
        var calculatedData: CalculatedData
        //  ...
        // Other needed data
        
	 	// By defining following equality function we can reduce its calculations complexity,
	 	// especially if eg. TransformedData contains more complex properties underneath.
        public static func == (lhs: Self, rhs: Self) -> Bool {
            lhs.transformedData.count == rhs.transformedData.count
        }
}
```

**In this case the TCA’s `WithViewStore` body reloads when the count of the transformed data changes.**

When there is no option to rely on one of the properties to determine the equality we can introduce additional property to the `State`. For example, the timestamp which would indicate the last update done in the struct.

```
public struct State: Equatable {
        var transformedData: [TransformedData]
         // Let’s add timestamp property
        var timestamp: Date
        //  ...
        
          // Then let’s simplify equality checks by verifying only if the timestamp has changed 
          // instead of comparing every single embedded property which is the default behavior.
        public static func == (lhs: Self, rhs: Self) -> Bool {
            lhs.timestamp == rhs.timestamp
        }
}

```

Then everytime we modify the state, let’s ensure that the timestamp is also updated. Taking our example we can do it in the reducer's action `finishUpdate`:

```
...
case let .finishUpdate(with: stateUpdate):
            state.transformedData = stateUpdate.transformedData
			// ...
            state.timestamp = Date()
...

```

This little change boosts the equality performance a lot if your state struct is very complex. However, keep in mind that this also makes your related-view reload every time - even though all properties are the same. Due to the timestamp - you force your state and view to reload. The choice is yours.

### 3. Scope your store state when possible

If you ever used a `pullback` feature from the TCA, for sure you have in your code something like the "main state" which also contains multiple sub-states which are needed for the child views.

Let me provide an example. Let’s assume that we have a separate state and reducer for the scrolling position. 

```
struct Scrolling {
    public struct State: Equatable {
        var position: CGFloat = 0
    }
    
    public enum Action: Equatable {
        case positionDidChange(value: CGFloat)
    }
    
    
    public static let reducer = Reducer<State, Action, Void> {
        state, action, _ in
        switch action {
        case let .positionDidChange(value):
            state.position = value
        }
    }
}

```

We also have a main state that keeps parent-related data like `transformedData`, but also sub-state information about the scrolling position in order to react to those changes and perform some operations. In addition, let's assume that this main state also keeps another sub-state called `UserData`. 

```
struct Main {
    public struct State: Equatable {
        var transformedData: [TransformedData]
         // our main state needs a scrolling state.
        var scrollingState = Scrolling.State()
         // here we could also have more states defined like UserData.State etc.
         var userDataState = UserData.State()
         // ...
    }
    
    public enum Action: Equatable {
        case updateState(with: [ViewData])
        // Let’s also add a new action which will handle scrolling position actions.
        case scrollingAction(Scrolling.Action)
        // ...
    }
    
    public static let reducer = Reducer<Main.State, Main.Action, Void> {
        state, action, _ in
        switch action {
        case let .updateState(viewData):
            // ...
            return .none
        case let .scrollingAction(action):
            switch action {
            case let  .positionDidChange(value):
                // do something with this value in the Main State.
                return .none
            default:
                return .none
            }
        }
    }
}
```

To make the reducers cooperate, we can create a new global one and use a pullback mechanism.

```
let mainReducer = Reducer<Main.State, Main.Action, Void>
    .combine(
        Main.reducer,
        Scrolling.reducer.pullback(
            state: \.scrollingState,
            action: /Main.Action.scrollingAction,
            environment: { /* No environment */ }
        ),
       UserData.reducer.pullback(
            ...
        ),
       // Here we could add in the future more co-reducers.
    )
```

We would like to reload the ChildView content every time the scrolling state changes. (the `Store` has been initialized with the `mainReducer`.) 
**The bad practice** of making use of it would look like this:

```
struct ParentView: View {
    let store: Store<Main.State, Main.Action>
        
    init(store: Store<Main.State, Main.Action>) {
        self.store = store
    }
 
    var body: some View {
        WithViewStore(store) { viewStore in
            ChildView()
                .position(x: viewStore.scrollingState.position)
        }
        // ...
    }
}
```

The problem is with the `WithViewStore` declaration because it binds the `main` store. It means that the child view reloads not only when the scroll position changes, but also if all other properties of the `Main.State` are updated (so `UserDataState`, `transformedData` etc.). So the `ChildView` reloading (along with the state equality checks) would happen more frequently which could quite significantly make the app performance worse.

Solution would be to scope the sub-state, so that the body will be reloaded only when scrolling state changes. In the context of the whole app this might be a great boost if scoping is done properly by using only necessary states. This is crucial especially for multiple nested views that have to react to different states. So the state composition has to be well defined at the very beginning. 

**Good approach:**

```
var body: some View {
        WithViewStore(store.scope(
                        state: \.scrollingState,
                        action: Main.Action.scrollingAction))
        { viewStore in
            ChildView()
                .position(x: viewStore.position)
        }
    }
```

### 4. Check how often your state is updated 

If you feel that you did everything described in the previous points but your app still feels not performant enough, It is worth checking - how often your state updates are triggered in the reducer. The two simple ways that I recommend:


a) The most effective - use instruments (SwiftUI analysis) and check the amount of the WithViewStore calls in the `View Properties` section. 

b) The second method is to use the `.debug()` modifier on the reducer and observe Xcode logs. (more in the readme of TCA).


If you noticed that this is the issue you can eg. try to play with the `.debounce()` or `.throttle()` mechanism which will prevent the state from updating too often in a short time interval and ultimately reduce amount of the view reloads (and equality checks in state) that are binded with ’WithViewStore’. An example use of the debounce mechanism based on the reducer from previous example could look like this:

```
public static let reducer = Reducer<State, Action, Environment> {
        state, action, environment in
        switch action {
        case let .updateState(viewData):
            return environment.performCalculations(with: viewData)
                .subscribe(on: environment.backgroundQueue)
                // debounce updates for 1 second and then dispatch back to the mainQueue.
                .debounce(for: .seconds(1), scheduler: environment.mainQueue)
                .eraseToEffect()
                .map { update in Action.finishUpdate(with: update) }
                // ...
            return .none
        case let .finishUpdate(with: stateUpdate):
            state.calculatedData = stateUpdate.calculatedData
            state.transformedData = stateUpdate.transformedData
            state.timestamp = Date()
            // ...
        }
}
```

### 5. drawingGroup() to the rescue.

If you still encounter the app lag/freeze even though you are sure that nothing from your code blocks the main thread, debugging tools show you that the majority of the main thread overload comes from the SwiftUI Views reload, then it seems like you reached the limits of its drawing capabilities. Of course there could be multiple other problems that are not described in this article - every single case is different. However if you feel that you optimized everything, there is one more thing that might help and it is also worth playing with - the `drawingGroup()` modifier.

```
struct LineView: View {
    public var body: some View {
        let linearGradient = LinearGradient.customLinearGradient()
        
        return ZStack {
            LinePath(path: path)
                .stroke(style: StrokeStyle(lineWidth: width, lineCap: .butt, lineJoin: .round))
                .fill(linearGradient)
                // … other views below
        }
    }
}
```

In the example above we displayed a custom View called LinePath which conforms to SwiftUI `Shape`. We added there stroke and the `LinearGradient` in order to add some colors to it. Unfortunately the LinearGradient uses quite a large amount of colors and also the Shape that we create is not trivial. Every time this view appears on the screen - the app freezes for a while. At the first glance there was nothing that we could improve in this case. It turned out that it was enough to add a `drawingGroup()` modifier. This simple line of code makes the composition of the view being created off screen as a bitmap using `Metal` rendering. When the bitmap is created - then it is presented on screen. Simple and easy.

```
 return ZStack {
            LinePath(path: path)
                .stroke(style: StrokeStyle(lineWidth: width, lineCap: .butt, lineJoin: .round))
                .fill(linearGradient)
             ....
        }
        // Add drawing group to get some performance boost :)
        .drawingGroup()
```

Of course it won’t work with every single view you create. If the `.drawingGroup()` function doesn't handle the creation of the view - instead you will see the warning placeholder on screen with some logs in the Xcode console - so you have to check on your own at what level of the View structure you can use it.

### Outro

Please keep in mind that the performance drop is a very complex topic and your problems may be not even related to the presented points. This article has been created based on the issues that we’ve encountered and learned upon during the development process. 

It is always important to rely on the Xcode debugging tools like Instruments which is providing quite often a clear picture of what’s going on under the hood. I also strongly recommend to perform stress tests for the complex views which for example are displaying multiple elements based on the amount of data that the user has. Especially if the number of the data is growing throughout the app usage. Sometimes at the first glance everything seems working fine, but when you enhance the view by adding more and more complexity, then the listed problems might grow to an unexpected level which is really hard to debug and improve.
