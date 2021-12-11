## Advent of Code 2021: Day 11

This blog post is eleventh in the Advent of Code 2021 series and shows a JavaScript-based solution to the problem described in [Day 11](https://adventofcode.com/2021/day/11). This day had many similarities to [Day 9](https://adventofcode.com/2021/day/9), and I solved the challenge using similar methods.

All solutions are in this [GitHub repository](https://github.com/mancristiana/advent-of-code-2021).

## Solving Part One
Part one aims to count the number of flashes a group of bioluminescent dumbo octopuses produce. 

We can break down the solution as follows:
- read the input into a matrix
- outline algorithm for counting flashes for 100 steps
- increment energy levels of all octopuses
- initialize a binary flash map
- recursively mark flashes and update matrix
- sum elements of the binary flash map
- reset energy levels for flashing octopuses

Initializing and traversing a matrix of numbers is done in the same way as described in my previous blog post [Advent of Code 2021: Day 9](https://blog.cristiana.tech/advent-of-code-2021-day-9). I suggest you check it out as I won't write about it again here.

### Outline algorithm for counting flashes for 100 steps
We can use a `for` loop to count flashes for 100 steps.
```js
let flashesCount = 0

for (let i = 0; i < 100; i++) {
  flashesCount += countFlashes(matrix, rows, cols)
}

return flashesCount
```

The `countFlashes` function outlines the changes in energy levels at each step and counts the corresponding flashes. As described in the problem, we must first increment the energy level of each octopus. Then, each octopus with an energy level higher than `9` flashes, which triggers increments energy levels of neighbouring octopuses. We can use a binary flash map to store and count the octopuses that flash in each step.

```js
const countFlashes = (matrix, rows, cols) => {
  // increment energy level
  // initialize binary flash map
  // recursively flash octopuses with energy level higher than 9
  // reset energy level
  // sum binary flash map
}
```

### Increment energy levels of all octopuses
The first part of our algorithm is incrementing each number in the matrix by `1`. 

```js
const countFlashes = (matrix, rows, cols) => {
  incrementEnergyLevel(matrix)
  // initialize binary flash map
  // recursively flash octopuses with energy level higher than 9
  // reset energy level
  // sum binary flash map
}
```

We can achieve this by traversing the matrix using nested `forEach` functions.
```js
const incrementEnergyLevel = (matrix) => {
  matrix.forEach((row, i) => {
    row.forEach((energyLevel, j) => {
      matrix[i][j]++
    })
  })
}
```

### Recursively mark flashes and update matrix

We can initialize a binary flash map to indicate which octopuses flash at each step. The binary flash map is a matrix of the same size as the octopuses matrix. We initialize each element with 0 and mark flashing octopuses with 1.

```js
const flashMap = initializeFlashMap(rows, cols)
```

We can then traverse the matrix of octopuses to find the ones with an energy level higher than 9. Once found, we mark that octopus and potentially neighbouring octopuses as flashing using the recursive `flash` function.

```js
const countFlashes = (matrix, rows, cols) => {
  incrementEnergyLevel(matrix)
  
  const flashMap = initializeFlashMap(rows, cols)
  matrix.forEach((row, i) => {
    row.forEach((energyLevel, j) => {
      if (energyLevel > 9) {
        flash(matrix, flashMap, rows, cols, i, j)
      }
    })
  })
  // reset energy level
  // sum binary flash map
}
```

The recursive `flash` function marks the current octopus as flashing by setting `flashMap[i][j] = 1`. Then goes through each neighbour to determine whether it also needs to flash. The stop condition for this recursion is when we reach an element that has already flashed.

```js
const flash = (matrix, flashMap, rows, cols, i, j) => {
  if (flashMap[i][j] === 1) {
    return
  }
  flashMap[i][j] = 1
  const neighbours = getNeighbours(rows, cols, i, j)
  neighbours.forEach(({ ni, nj }) => {
    matrix[ni][nj]++
    if (matrix[ni][nj] > 9) {
      flash(matrix, flashMap, rows, cols, ni, nj)
    }
  })
}
```

#### Find neighbours
To find the neighbours of an octopus, I used the following relative indices as shown in the diagram.

![MatrixNeighboursIndices.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1639231413376/ne-4izdK3J.png)

```js
const getNeighbours = (rows, cols, i, j) => {
  const neighbours = []
  for (let ni = i - 1; ni <= i + 1; ni++) {
    for (let nj = j - 1; nj <= j + 1; nj++) {
      if (ni === i && nj === j) {
        continue
      }
      neighbours.push({ ni, nj })
    }
  }
  return neighbours
}
```

The algorithm, however, does not take into account elements on the edge of the matrix. We can use `Math` functions to adjust the limits of the `for` loops to accommodate edges.
```js
const getNeighbours = (rows, cols, i, j) => {
  const neighbours = []
  for (let ni = Math.max(0, i - 1); ni <= Math.min(rows - 1, i + 1); ni++) {
    for (let nj = Math.max(0, j - 1); nj <= Math.min(cols - 1, j + 1); nj++) {
      if (ni === i && nj === j) {
        continue
      }
      neighbours.push({ ni, nj })
    }
  }
  return neighbours
}
```

### Sum elements of the binary flash map
We can traverse the binary flash matrix using `forEach` function to sum each element. Since flashing octopuses are marked with `1` and all other octopuses with `0`, this function counts the number of flashing octopuses.
```js
const sumFlashMap = (flashMap) => {
  let sum = 0
  flashMap.forEach((row) => {
    row.forEach((flash) => {
      sum += flash
    })
  })
  return sum
}
```

### Reset energy levels for flashing octopuses
To reset the energy levels for flashing octopuses we need to traverse the octopus matrix and set the octopuses with energy larger than `9` to `0`. 
```js
const resetEnergyLevel = (matrix) => {
  matrix.forEach((row, i) => {
    row.forEach((energyLevel, j) => {
      if (energyLevel > 9) {
        matrix[i][j] = 0
      }
    })
  })
}
```

The final solution for part one is in my [GitHub repository](https://github.com/mancristiana/advent-of-code-2021/blob/main/src/day-11-dumbo-octopus/one.js).

## Solving Part Two
Part two was about finding the step when all octopuses flash simultaneously. Since we already implemented a function that counts flashes on each step, we can run the program and increment the step until all 100 octopuses flash.
```js
let step = 1
while (countFlashes(matrix, rows, cols) !== 100) {
  step++
}
return step
```

The final solution for part two is in my [GitHub repository](https://github.com/mancristiana/advent-of-code-2021/blob/main/src/day-11-dumbo-octopus/two.js).

## Conclusion
Day 11 was about traversing a matrix of octopuses, increasing their energy level with each step, recursively flashing octopuses and their neighbours, and finally, determining when all octopuses flash simultaneously.