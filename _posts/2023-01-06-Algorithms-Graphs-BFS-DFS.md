---
layout: post
author: bartek
title: \#57 Graph traversal algorithms BFS and DFS
excerpt: If you would like to start learn graph algotithms I hope this post can help you with finding some basic information about it. With shared playthrounds and examples you can check them it action by yourself.
---
### Intro

Graphs for beginners seem really hard, but the truth is that they are not so difficult when you learn the basics and you really understand the main mechanisms. Let's start with some basic knowledge.

### What is a graph?

- Consists of nodes which are connected by edges

- Each edge can have a direction (directed graph) or not (undirected graph) 

- Additionally, graph can be weighted or not. Weighted means that edge has some value. It can for example represent the distance between two nodes. 

- Example of the graph you can see here (Undirected Cyclic Graph):

<p align="center">
<img src="https://raw.githubusercontent.com/swiftingio/blog/%2357-Algorithms-DFS-BFS/Graph.svg " width="643">
</p>
  
### Graph representation

Typically we represent a graph by adjacency list (it is the most popular option). Every node has a list pointing to other nodes to which it has a connection. For an implementation point of view we can use for that a dictionary.

For example, we can construct an adjacency list like this:

```swift
var adjList: [Int: [Int]] = [:]

adjList[1] = [2, 3]
adjList[2] = [1, 4]
adjList[3] = [1, 4]
adjList[2] = [5]

```

The second option to represent a graph is a matrix:

| 0    | 1    | 1    | 0    | 0    |
| ---- | ---- | ---- | ---- | ---- |
| 1    | 0    | 0    | 0    | 1    |
| 1    | 0    | 0    | 1    | 0    |
| 0    | 1    | 1    | 1    | 0    |
| 0    | 1    | 0    | 0    | 0    |

This matrix shows that element node 1 (columns 1) is connected with node 2 and 3 (so that's why row 2 and row 4 has ones in column 1).

Then node 2 (column 2) is connected with node 1, 4, 5 (that's why in row 1, 4 and 5 there is "1" in column 2), etc.

To be honest I have never used matrix representation in any graph related problem, but for sure it is good to know that this kind of representation exists.

### Very popular graph terms

- **Cycle**: if within a graph we travel and meet the visited node once again: it means that we have a cycle in the graph. Finding a path of a cycle can be complicated and we can use an algorithm to do that (BFS or DFS - more about them in further parts of the article).

<p align="center">
<img src="https://raw.githubusercontent.com/swiftingio/blog/%2357-Algorithms-DFS-BFS/Cycle.svg" width="643">
</p>

- **Connected component (cc)**: we can imagine a connected component as a lonely island which is not connected to other lands. There exist algorithms for finding connected components in the graph. I hope in the future I will write more about it in a separate post.

<p align="center">
<img src="https://raw.githubusercontent.com/swiftingio/blog/%2357-Algorithms-DFS-BFS/ConnectedComponents.svg" width="643">
</p>

### The most popular graph traversal algorithm

Let's focus on the main subject of this post: Graph traversal algorithms. We can distinguish two:

- DFS (Depth First Search)
- BFS (Breadth First Search)

It is really important to understand how they work and when they are most useful (in which use case). Below I tried to highlight the main features of both. I presented it in the table for a better understanding of the difference between them.

|                                                      | DFS                                                          | BFS                                                          |
| :--------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **Main usability**                                   | Is commonly used for visiting (traversing) all nodes in the graph (or to be more concrete: in connected component ) | Is used when we want to find the shortest path in the unweighted graph |
| **Implementation**                                   | Using recursion (iterative as well but definitely less popular and used) | Iterative                                                    |
| **Data structure used for iterative implementation** | Stack                                                        | Queue                                                        |
| **Visiting nodes order**                             | Branch by branch (so it goes by depth as much as possible, then it switches to the next branch) | It visits the nodes level by level. First, it starts with the nearest neighborhood of starting node and then it goes to the next level etc. |
| **Cycle detection**                                 | Commonly used for cycle detection                            | Not very efficient for finding a cycle in the graph.        |
| **Complexity**                                       | O(V+ E)                                                      | O(V+ E)                                                      |
| **Practice**                                         | - https://leetcode.com/problems/shortest-path-in-a-grid-with-obstacles-elimination/description/ | https://leetcode.com/problems/minimum-genetic-mutation/solution/    https://leetcode.com/problems/nearest-exit-from-entrance-in-maze/ |

### Visualization

#### DFS

As we said, the DFS is going branch by branch, is going deep as much as possible than it turns back and starts investigating a new branch. Having this in mind let's look at what could be the order of visiting nodes if we started from the node with an id equal to 3:

<p align="center">
<img src="https://raw.githubusercontent.com/swiftingio/blog/%2357-Algorithms-DFS-BFS/DFS.svg" width="643">
</p>

Algorithm starts with the selection of some first branch and goes to the end (end means that the algorithm encounters a node with no other children, or all the children are already visited):

Node 5 has no other children so we come back to the place when it can check another branch (in this case it will be  node 3)

And then algorithm starts to vist the new unexplored branch, so it goes to node one and then it finishes the job.

#### BFS

The visualization shows that we go level by level and I think this is the best way how we can imaging the algorithm .

<p align="center">
<img src="https://raw.githubusercontent.com/swiftingio/blog/%2357-Algorithms-DFS-BFS/BFS.svg" width="643">
</p>

If we start with node id = 3 (first layer), then we visit 1 and 4 (second layer) then 2 (third layer) and last will be 5 (fourth layer).

### Implementation

Having this knowledge let's implement DFS and BFS in Swift. Additionally let's take a look at some additional topics which are very interesting as well:

- Finding the shortest path with unweighted graph
- Finding a cycle in an undirected graph
- Finding a cycle in a directed graph

#### DFS

```swift

// This class will be used later in every algorithm
class Node {
    let id: Int
    var children: [Node]
    
    init(id: Int, children: [Node] = []) {
        self.id = id
        self.children = children
    }
}

var visited: Set<Int> = []

func dfs(_ root: Node?)  {

	guard let root = root else {
		return
	}

	visited.insert(root.id)

	for child in root.children {
		if !visited.contains(child.id) {
			dfs(child)
		}
	}
}
```

Download playground from [here](https://github.com/woroninb/GraphTraversalAlgorithms/blob/main/DFS.playground/Contents.swift).

#### BFS

```swift

func bfs(_ start: Node) {
    var queue: [Node] = []
    var visited: Set<Int> = []
    
    visited.insert(start.id)
    queue.append(start)
    
    while !queue.isEmpty {
        for item in queue {
            let first = queue.removeFirst()
            for child in first.children {
                if !visited.contains(child.id) {
                    visited.insert(child.id)
                    queue.append(child)
                }
            }
        }
    }
}
```

Download playground from [here](https://github.com/woroninb/GraphTraversalAlgorithms/blob/main/BFS.playground/Contents.swift).

#### BFS + shortest path

Implementation with additional finding distance from the start node and printing a path (an important note here is that here we are discussing unweighted graphs. If we want to find the shortest distance for a weighted graph, we need to consider another algorithm like [Dijktstra's algorithm](https://en.wikipedia.org/wiki/Dijkstra%27s_algorithm)).


```swift

var parent: [Int: Int] = [:]

func bfs(_ start: Node) {
    var queue: [Node] = []
    var visited: Set<Int> = []
    var distance: [Int: Int] = [:]
    
    visited.insert(start.id)
    queue.append(start)
    
    while !queue.isEmpty {
        for item in queue {
            let first = queue.removeFirst()
            for child in first.children {
                if !visited.contains(child.id) {
                    distance[child.id] = distance[item.id, default: 0] + 1
                    parent[child.id, default: 0] = item.id
                    visited.insert(child.id)
                    queue.append(child)
                }
            }
        }
        
    }
}

func getPath(_ target: Node, _ start: Node) {
    var x = target.id
    var path: [Int] = []
    
    path.append(target.id)
    
    while x != start.id {
        x = parent[x, default: 0]
        path.append(x)
    }
    
    path.reverse()
}
```

Download playground from [here](https://github.com/woroninb/GraphTraversalAlgorithms/blob/main/BFS%2BShortestPath.playground/Contents.swift).

#### Detecting cycle in an undirected graph

```swift

var visited: Set<Int> = []
var parent: [Int: Int] = [:]

func dfs(_ root: Node?, _ previous: Node?) {

    guard let root = root else {
        return
      }

    visited.insert(root.id)
    
    let children = root.children
    for child in children {
        if !visited.contains(child.id) {
            parent[child.id] = root.id
            dfs(child, root)
        } else if child.id != previous?.id {
            // here we can be sure that we don't have a cycle which consists of only two elements
            // and then we can print stack starting from child
            getPath(root, child)
            return
        }
    }
}
```

Download playground from [here](https://github.com/woroninb/GraphTraversalAlgorithms/blob/main/DetectingCycleInAnUndirectedGraph.playground/Contents.swift).

#### Detecting cycle in a directed graph

Undirected graphs are much easier to handle, because we don't care about direction of each edge. In this section I would like to show you how to detect a cycle in a directed graph and as well how to find a path for the detected cycle. 

There are two options to find a cycle:

1. DFS with node coloring

```swift
enum NodeState {
    case unvisited
    case visiting
    case visited
}

var parent: [Int: Int?] = [:]
var visited: [Int: NodeState] = [:]

func dfs(_ root: Node?) {
    
    guard let root = root else {
        return
    }
    
    visited[root.id] = .visiting
    let children = root.children
        
    for child in children {
        parent[child.id] = root.id

        if visited[child.id] == .unvisited {
            dfs(child)
        } else if visited[child.id] == .visiting {
            findCyclePath(root, child)
            return
        }
    }
    visited[root.id] = .visited
}

private func findCyclePath(_ start: Node?, _ end: Node) {
    var path: [Int?] = []
    var parentElement: Int? = start?.id
    path.append(end.id)
    while parentElement != nil && parentElement != end.id {
        path.append(parentElement)
        parentElement = parent[parentElement!, default: 0]
    }
}

```

Download playground from [here](https://github.com/woroninb/GraphTraversalAlgorithms/blob/main/DetectingCycleInDirectedGraphColoring.playground/Contents.swift).


2. Alternate way for finding cycle with DFS is the the usage of two arrays:

- `var isVisited: [Bool]`
- `var inStack: [Bool]` - used to track the items on which we go deep and then if we visited all branches then we need to clear it or it say that we found a cycle

```swift
var visited: Set<Int> = []
var inStack: Set<Int> = []

func dfs(_ root: Node?, _ stack: [Node]) {
    guard let root = root else {
        return
    }
    
    inStack.insert(root.id)
    visited.insert(root.id)
    let children = root.children
    
    for child in children {
        if !visited.contains(child.id) {
            var newStack = stack
            newStack.append(child)
            dfs(child, newStack)
        } else if inStack.contains(child.id) {
            var path = stack
            path.append(child)
            print(path.map { $0.id })
        }
    }
    inStack.remove(root.id)
}  
```

Download playground from [here](https://github.com/woroninb/GraphTraversalAlgorithms/blob/main/DetectingCycleInDirectedGraphArrays.playground/Contents.swift).

#### Summary

Thank you for reading it! I hope it somehow was helpful to you. In terms of any questions please email me: bartlomiej.woronin@gmail.com ;) 

