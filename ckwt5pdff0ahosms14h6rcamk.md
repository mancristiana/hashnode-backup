## Advent of Code 2021: Day 5

This blog post is fifth in the Advent of Code 2021 series and shows a JavaScript-based solution to the problem described in [Day 5](https://adventofcode.com/2021/day/5). The challenges were about finding overlapping points given a set of lines.

All solutions are in this [GitHub repository](https://github.com/mancristiana/advent-of-code-2021).

## Solving Part One
The input contains a set of lines defined using x, y coordinates of two points in the following format `x1,y1 => x2,y2`. The goal is to find how many points consisting of integer coordinates overlap. The first part requires only checking points overlapping horizontal or vertical lines.

We can break the problem as follows:
- read lines from the input file
- find vertical and horizontal lines
- store points in a hash map
- count overlapping points

### Read lines from the input file
We can reuse the `readInput` function written on Day 1. If you haven't already, give [Advent of Code: Day 1](https://blog.cristiana.tech/advent-of-code-2021-day-1) a read!
After extracting the data as a string, we split it into lines using the newline separator. Then, we map each line to an array containing the coordinates as numbers. So the line `x1,y1 -> x2,y2` is mapped to `[x1, y1, x2, y2]`

```js
const lines = data.split('\n').map((line) =>
  line
    .replace(' -> ', ',')
    .split(',')
    .map((n) => Number(n))
)
```

### Find vertical and horizontal lines
We can then iterate through the lines and destructure the coordinates.
```js
lines.forEach((line) => {
  const [x1, y1, x2, y2] = line
  
})
```
We must only consider points from vertical or horizontal lines in the first part of the problem. Vertical lines have the exact x coordinates, whereas horizontal lines have the same y coordinates.
```
const isVertical = x1 === x2
const isHorizontal = y1 === y2
```

We can find the points on a vertical line using either x1 or x2 coordinate and all numbers between y1 and y2. Since we don't know which one is higher, we can calculate the minimum and maximum using `Math.min(y1, y2)` and `Math.max(y1, y2)` respectively. Then, using a for loop we can determine all integer values from the minimum to the maximum value. 
```js
if (isVertical) {
  const minY = Math.min(y1, y2)
  const maxY = Math.max(y1, y2)
  for (let y = minY; y <= maxY; y++) {
    addPointToHashMap(pointHashMap, x1, y)
  }
} 
```

Using the same approach, we can determine the points on horizontal lines.
```js
if (isHorizontal) {
  const minX = Math.min(x1, x2)
  const maxX = Math.max(x1, x2)
  for (let x = minX; x <= maxX; x++) {
    addPointToHashMap(pointHashMap, x, y1)
  }
}
```

### Store points in a hash map
Since the problem does not define maximum coordinate value, storing the points using a two-dimensional matrix is not the most efficient memory-wise. Instead, I opted for a hash map where keys are strings in `x,y` format and values are the number of overlapping points.

We initialize the point hash map with an empty object.
```js
const pointHashMap = {}
```
Then, we create new keys and initialize them with 1 when we find a new point, or increase the value of that point when the point already exists in the map.
```js
const addPointToHashMap = (hashMap, x, y) => {
  const key = `${x},${y}`
  if (hashMap[key]) {
    hashMap[key]++
  } else {
    hashMap[key] = 1
  }
}
```
### Count overlapping points
Finally, we need to count overlapping points, in other words, points with a value larger than `2`.

The `Object.keys(hashMap)` function gives an iterable array of the points. We can then use the `Array.reduce` function to calculate the total amount of overlapping points.

```js
const countOverlappingPoints = (hashMap) => {
  const points = Object.keys(hashMap)
  return points.reduce((total, point) => {
    if (hashMap[point] > 1) {
      return total + 1
    }
    return total
  }, 0)
}
```
We call the function with the previously defined `pointHashMap`.
```js
countOverlappingPoints(pointHashMap)
```
You can see the final solution for part one on my [GitHub repository](https://github.com/mancristiana/advent-of-code-2021/blob/main/src/day-05-hydrothermal-venture/one.js).

## Solving Part Two
In part two, we must consider all lines. Fortunately, the problem gives an input constraint where all lines are horizontal, vertical or diagonal at precisely 45 degrees.

We can reuse most of the solution from part one, with the following additions:
- find diagonal lines
- find the points on diagonal lines


### Find diagonal lines
Given the input constraint, all lines that are not horizontal neither vertical must be diagonal.
```js
const isDiagonal = !isHorizontal && !isVertical
```

### Find the points on diagonal lines
According to the problem, 

> An entry like 1,1 -> 3,3 covers points 1,1, 2,2, and 3,3.
> An entry like 9,7 -> 7,9 covers points 9,7, 8,8, and 7,9.

So if we get the ranges of numbers between each coordinate `x1 -> x2` and `y1 -> y2`, we can determine the points on the diagonals.

```js
if (isDiagonal) {
  const xRange = range(x1, x2)
  const yRange = range(y1, y2)
  xRange.forEach((x, index) => {
    const y = yRange[index]
    addPointToHashMap(pointHashMap, x, y)
  })
}
```

Given two x or y coordinates (e.g. `x1 = 9`, `x2 = 7`), we can generate an array of steps (e.g. `0, 1, 2`) corresponding to each number in the range. Then depending on whether coordinates increase or decrease, we can build the range as follows. For example, the coordinates `x1 = 9` and `x2 = 7` decrease. Thus, we decrease the number with each step (`c1 - p`).
```js
const range = (c1, c2) => {
  const pointsCount = Math.abs(c1 - c2) + 1
  return [...Array(pointsCount).keys()].map((p) => {
    if (c1 < c2) {
      return c1 + p
    }
    return c1 - p
  })
}
```

You can see the final solution for part two on my [GitHub repository](https://github.com/mancristiana/advent-of-code-2021/blob/main/src/day-05-hydrothermal-venture/two.js).

## Conclusion
Day 5 was about finding overlapping points from a set of given lines. A hash map helped store points and determine how many lines overlap.  JavaScript functions like `Math.abs`, `Math.min`, `Math.max` as well as `Object.keys` and `Array.reduce` were quite handy.