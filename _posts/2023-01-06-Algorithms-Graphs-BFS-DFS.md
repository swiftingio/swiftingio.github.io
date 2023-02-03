# Graph traversal algorithms: BFS & DFS

### Intro

Graphs for beginners seems really hard, but the true is that they are not so difficult when you learn the basics and you really understand the main mechanisms.

### What is a graph?

- Consits of nodes which are connected by edges

- Each edge can have direction (directed graph) or not (undirected graph) 

- Additionally graph can be weighted or not. Wighted means that edge has some value. It can for example represent distance between two nodes. 

- Example of the graph you can see here (Undircted Cyclic Graph):

<p align="center">
<img src="https://raw.githubusercontent.com/swiftingio/blog/%2357-Algorithms-DFS-BFS/Graph.svg " width="643">
</p>
  

### Graph represensnation

Typically we represent graph by adjanceny list (is the most popular option). Every node has a list poiting to another nodes to which has connection. For implementation point of view can use for that a dicionary.

So for example to reprsent such a graph (which I showed on the image above) we can construct a adjancy list like:

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

that show that element node 1 (columns 1) is connected with 2 and 3 (so that why row 2 and row 4 has ones in columns 1).

then node 2 (column 2) is connected with 1, 4, 5 (thats why in row 1, 4 and 5 have ones in column 2), etc.

To be honest I have never used this kind of representation in any graph related problem, but for sure is good to know that exists this kind of representation.

### Very popular graph terms

From my experience I really often saw in diffrent articles the following terms. So is very I importnat I thik to get familar with those:

- Cycle - if within a graph we  travel and we meet the visited node once again then it mean that we have a cycle in the graph. Finding a cycle will be a subject of my next post. Is quite intersing topic, becuase first of all we can ask ourselve what to choose DFS or BFS. Then to make things more complicated how to find a pth of the cycle...

<p align="center">
<img src="https://raw.githubusercontent.com/swiftingio/blog/%2357-Algorithms-DFS-BFS/Cycle.svg" width="643">
</p>

- Connected component (cc) - we can imagine connected component as an lonly island which is not connect to other lands. There exists algorithms for finding connected components in the graph. I hope in the future as well writes more abiut it in the sepearated post.

<p align="center">
<img src="https://raw.githubusercontent.com/swiftingio/blog/%2357-Algorithms-DFS-BFS/ConnectedComponents.svg" width="643">
</p>

### The most popular graph traversal algorithm

Let's focus on the main subject of this post: Graph traversal algorithms. We can distinguish two:

- DFS (Depth First Search)
- BFS (Breath First Search)

Is really important to understand how they works and when they are the most useful (in which use case). Below I tried to highligt the main features for both ones. I prestented it in table for better undesratnding the diffrence between them.

|                                                      | DFS                                                          | BFS                                                          |
| :--------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **Main usability**                                   | Is commonly used for visiting (traversing) all nodes in the graph (or to be more concrete: in connected component ) | Is used when we want to find the shortest path in the unweighted graph |
| **Implementation**                                   | Using recursion (iterative as well but definitly less popular and used) | Iterative                                                    |
| **Data structure used for iterative iplemnentation** | Stack                                                        | Queue                                                        |
| **Visiting nodes order**                             | Branch by branch (so it goes by depth as much as possible then it swithces to next branch) | It visiting the nodes level by level. First it starts with the nearest neighboorhood of staring node and then it goes to the next level etc. |
| **Cyclce detection**                                 | Commonly used for cycle detection                            | Not very efficientg for finding a cycle in the graph.        |
| **Complexity**                                       | O(V+ E)                                                      | O(V+ E)                                                      |
| **Practice**                                         | - https://leetcode.com/problems/shortest-path-in-a-grid-with-obstacles-elimination/description/ | https://leetcode.com/problems/minimum-genetic-mutation/solution/    https://leetcode.com/problems/nearest-exit-from-entrance-in-maze/ |

### Visualization

#### DFS

As we said the DFS is going branch by branch. Is going depp as much as possible than it turns back and start to investigate a new branch. Having this in mind let's look what could be the order of visiting nodes if we would start at node with id equal to 3:

<p align="center">
<img src="
https://raw.githubusercontent.com/swiftingio/blog/%2357-Algorithms-DFS-BFS/DFS.svg" width="643">
</p>

Algorithm start with selection some first branch and goes to the end (end mens that encounters node with no other children, or all the childrens aere already visited):

Node 5 has no aother children so we come back to the place when it can check another branch (it this case it will be a node 3)

And Then algorithm starts to viist the new unexplored branch, so it goes to node one and the it finishes the job.

#### BFS

The visialization shows that we go level by level and I think this is the best way how we can image how the algorithms works.

<p align="center">
<img src="
https://raw.githubusercontent.com/swiftingio/blog/%2357-Algorithms-DFS-BFS/BFS.svg" width="643">
</p>

If we start with node id = 3 (first layer), then we visit 1 and 4 (second layer) then 2 (third layeer) and last will be 5 (fourth layer)

### Impmenenation

Having this knowledge let's implement DFS and BFS with Swift. Additionally let's take a look on some additional topics which are as well very interesting:

- Finding the shortes path with unweighted graph
- Finding cycle in undirected graph
- Finding cycle in indirected graph

#### DFS

```swift
var visited: Set<Int> = []

func dfs(_ root: TreeNode?)  {

	guard let root = root else {
		return
	}

	visited.insert(root.val)

	for child in root.children {
		if !visited.contains(child.val) {
			dfs(child)
		}
	}
}
```

#### BFS

```swift
func bfs(_ start: Node) {

	var queue: [Node] = []
	var visited: Set<Node> = []

	queue.append(start)

	while !queue.isEmpty { // maybe then this while is not needed anymore (worth to check)
  	for item in queue {
    	let first = queue.removeFirst()
   		for child in first.children {
				if !visited.contains(child) {
      				visited.insert(child)
						queue.append(child)
					}
				}
			}
		}
	}	
}
```

#### BFS + shortest path

implementations with additinal finding distance from the start and a path, and important note here is that here we are discussing graph without weights. If we want to find the shortest distance for a graph with weights we need to consider another algorithm which is Dijktstra.


```swift
func bfs(_ start: Node) {
	var queue: [Node] = []
	var visited: Set<Node> = []
	var distance: [Node: Int] = [:]
	var parent: [Node: Node] = [:]

	queue.append(start)

	while !queue.isEmpty {
		for item in queue {
			let first = queue.removeFirst()
			for child in first.children {
				if !visited.contains(child) {
    	    		dist[child] = dist[item] + 1
    	    		parent[child] = item 
  					visited.insert(child)
					queue.append(child)
				}
			}
		}
	}
}

func getPath(_ target: Node, _ start: Node) {
  var x = target
  var path: [Int] = []
  
  path.append(target)
  
  while x != start {
    x = parent[x]
    path.append(x)
  }
  
  path.reverse()
}
```


#### Detecting cycle in undirected graph

```swift

var visited: Set<Node> = []

func dfs(_ root: TreeNode?) {	

  guard let root = root else {	
		return
	}

	visited.insert(root)
	let children = root.children

	for child in children {
		if !visited.contains(child) {
				dfs(child)
		} else if child != previous {
    		// here we can be sure that we don't have a cycle consists of two elements
		}
	}
}
```

#### Detecting cycle in directed graph

Undirected are much more easier to handle because we don't care about direction of each edge. In this section I would like to show you how to detect the cycle in directed graph and as well how to find a path for such a graph. 

There is two options to find a cycle:

1. DFS with finding cycle with coloring

```swift
enum NodeState {
	case unvisited
	case visiting
  case visited
}

class TreeNode: Hashable {
        
   static func == (lhs: FindCycleWithColor2.TreeNode, rhs: FindCycleWithColor2.TreeNode) -> Bool {
       return lhs.value  == rhs.value
   }
        
    func hash(into hasher: inout Hasher) {
       hasher.combine(value)
      }
        
      let value: Int
       var children: [TreeNode]
        
       init(value: Int, children: [TreeNode]) {
           self.value = value
           self.children = children
        }
    }

var parent: [TreeNode: TreeNode?] = [:]
var visited: [TreeNode: NodeState] = [:]

func dfs(_ root: TreeNode?) {	

  guard let root = root else {	
		return
	}

	visited[root] = visiting
	let children = root.children

	for child in children {
		if visited[child] == unvisited {
				dfs(child)
				parent[child] = root
		} else if visited[child] == visiting {
			findCyclePath(child, root)
		}
	}

	visited[root] = .visited	
}

private func findCyclePath(_start: TreeNode?, end: TreeNode?) {
  	var path: [Int] = []
		 var parentElement = start
  	while parentElement != end {
    	  path.append(parentElement.id)
      	parentElement = parent[parentElement]
    }
}
```

2. Alternate way for finding cycle with DFS with the usage of two arrays:

- isVisited: [Bool]
- inStack: [Bool] // track the item on which we go deep and then if we visited all branch then we need to clear it, or it syas taht we found a cycle

```swift
var visited: Set<Node> = []
var inStack: Set<Node> = []

func dfs(_ root: TreeNode?, stack: [Node]) {	

  guard let root = root else {	
		return
	}
	
	visited.insert(root)
	let children = root.children

	for child in children {
		if !visited.contains(child) {
				dfs(child, stack + child)
		} else if inStack.contains(child) {
	      print(stack)
		}
}
 
inStack.remove(root)
  
}
  
```

#### Summary

Thank you for reading it! I hope it somehow was helpful to you. In terms of any question please email me ;) 

