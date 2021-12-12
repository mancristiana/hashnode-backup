## Advent of Code 2021: Day 12


This blog post is twelfth in the Advent of Code 2021 series and shows a JavaScript-based solution to the problem described on [Day 12](https://adventofcode.com/2021/day/12).

All solutions are in this [GitHub repository](https://github.com/mancristiana/advent-of-code-2021).

## Solving Part One
Given a set of connections between underground caves, we must find all possible paths between the `start` and `end` caves and output the number of found paths. We can visit large caves (named using uppercase letters) multiple times, whereas we can only visit small caves (names using lowercase letters) once.

The strategy for solving part one was a recursive algorithm that returns the number of paths from a given cave to the `end` cave. At each recursion step, we visit each linked cave of the current cave and recursively return the number of paths from the linked cave to the `end` cave.

In terms of data structure, I opted for representing the cave graph using an [Adjacency List](https://en.wikipedia.org/wiki/Adjacency_list). In other words, a key-value map, where the keys are the cave names and each value is an array corresponding to linked caves. For example, the input from the problem description would produce the following key-value map:
```js
{
  start: [ 'A', 'b' ],
  A: [ 'start', 'c', 'b', 'end' ],
  b: [ 'start', 'A', 'd', 'end' ],
  c: [ 'A' ],
  d: [ 'b' ],
  end: [ 'A', 'b' ]
}
```
This data structure works well for iterating through adjacent caves.

I broke down the solution as follows:
- build the adjacency list from the input file
- get number of paths from the start cave recursively
- prevent visiting small caves twice

### Build the adjacency list from the input file
The first step to solving part one is reading each line representing a connection between two caves and adding this information to the adjacency list.
```js
const links = {}
const lines = data.split('\n')
lines.forEach((line) => {
  const [caveA, caveB] = line.split('-')
  addLink(links, caveA, caveB)
  addLink(links, caveB, caveA)
})
```
The `addLink` function adds a connected cave (e.g. `caveB`) to the corresponding list (e.g. `caveA` list) or creates a new list if the cave does not exist.
```js
const addLink = (links, caveA, caveB) => {
  if (links[caveA]) {
    links[caveA].push(caveB)
  } else {
    links[caveA] = [caveB]
  }
}
```

### Get number of paths from the start cave recursively

The first call of the recursive `getPaths` function begins from the `start` cave. It passes the following parameters:
- the adjacent list `links` used for finding connected caves
- an array of visited caves containing the `start` cave
- the cave from which to count paths to the `end` cave

```js
getPaths(links, ["start"], "start")
```

The outline for getting the number of paths recursively is the following:
```js
const getPaths = (links, visitedSmallCaves, cave) => {
  if (cave === "end") {
    return 1
  }

  let generatedPaths = 0
  links[cave].forEach(linkedCave => {
    // TODO: if small cave not visited 
    generatedPaths += getPaths(links, visitedSmallCaves, linkedCave)
  })
  return generatedPaths
}
```

The recursion stop condition is when we reach the `end` cave. Thus we can return `1` representing a path found. Alternatively, we might return `0` if we reach a dead-end (e.g. a small cave connected to already visited small caves).

Each recursion step, we iterate through the connected caves and sum the possible paths to the `end` cave. 

Keep in mind that the outlined algorithm doesn't yet consider visited small caves. Thus, if you run it as is, it will go back and forth between the same two caves until the program throws a stack overflow exception. We'll take a look at visited caves next.

### Prevent visiting small caves twice
To check whether a cave is small we need to verify whether it has lowercase characters
```js
const isSmallCave = (cave) => cave.toLowerCase() === cave
```
Then we can update the `visitedSmallCaves` like so:
```js
const newVisitedArray = isSmallCave(linkedCave) 
  ? [...visitedSmallCaves, linkedCave]
  : visitedSmallCaves
```
We are then able to check whether we have visited a small cave using the `include` function like so:
```js
visitedSmallCaves.includes(linkedCave)
```

Thus, the updated `getPath` function contains the following:
```
links[cave].forEach(linkedCave => {
  if (!visitedSmallCaves.includes(linkedCave)) {
    const newVisitedArray = isSmallCave(linkedCave) ? [...visitedSmallCaves, linkedCave] : visitedSmallCaves
    generatedPaths += getPaths(links, newVisitedArray, linkedCave)
  }
})
```

The final solution for part one is in my [GitHub repository](https://github.com/mancristiana/advent-of-code-2021/blob/main/src/day-12-passage-pathing/one.js).

## Solving Part Two
Part two changes the visiting rules by allowing one to visit a single small cave twice, except the `start` and `end` caves, which one can only visit once.

The strategy for solving part two was adding an item called `twice` to the `visitedSmallCaves` array after visiting a small cave twice. 

I updated the recursive `getPath` function in order to separate the visiting logic to two separate functions: `hasVisitedCave` and `getVisitedCaves`.
```js
  links[cave].forEach(linkedCave => {
    if (!hasVisitedCave(visitedSmallCaves, linkedCave)) {
      generatedPaths += getPaths(links, getVisitedCaves(visitedSmallCaves, linkedCave), linkedCave)
    }
  })
```

The `hasVisitedCave` function includes the new rules and allows visiting a single small cave that are not names `start` twice.
```js
const hasVisitedCave = (visited, cave) => {
  if (cave === "start") {
    return true
  }
  return visited.includes(cave) && visited.includes("twice")
}
```

The `getVisitedCaves` function skips adding large caves, adds the `twice` element to the array once a small cave has been visited twice and adds small caves that have not been visited before.
```js
const getVisitedCaves = (visited, cave) => {
  const isSmall = isSmallCave(cave)
  if (!isSmall) {
    return visited
  }
  if (visited.includes(cave)) {
    return [...visited, "twice"]
  }
  return [...visited, cave]
}
```

The final solution for part two is in my [GitHub repository](https://github.com/mancristiana/advent-of-code-2021/blob/main/src/day-12-passage-pathing/two.js).

## Conclusion
Day 12 was about finding all paths between two caves recursively. A graph implemented using an Adjacency List helped store connected caves.