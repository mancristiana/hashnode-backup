## Advent of Code 2021: Day 1

The [Advent of Code](https://adventofcode.com/2021) is an Advent calendar with programming challenges. It is a wonderful way to keep your skills sharp, learn a new programming language, prepare for interviews, or have something to look forward to each day leading up to Christmas.

This blog series will walk you through my thought process and show possible solutions to the challenges. I used JavaScript to solve these, but you can use any programming language.

All solutions are in this [GitHub repository](https://github.com/mancristiana/advent-of-code-2021).

## Project Setup
To use JavaScript, we need to set a Node.js project. I opted for Yarn as a package manager due to improved performance when installing dependencies and personal preference, but `npm` works well too.

Steps:
1. Initialize the project by running `yarn init`. This command creates the `package.json` file containing project name, description, licencing and dependencies.
2. Add `"type": "module"` to the `package.json` file to enable [ESModules](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules) since Node.js uses CommonJS modules by default.
3. Install  [Prettier](https://prettier.io) to format code automatically. Run `yarn add prettier` and add the following configuration in `.prettierrc`: 
```
trailingComma: "es5"
tabWidth: 2
semi: false
singleQuote: true
endOfLine: "lf"
```
4. Add `.gitignore` file for the Node.js project. [This template from GitHub](https://github.com/github/gitignore/blob/master/Node.gitignore) works well.

The solution and input files are grouped by day in the `src` folder (e.g. `src/day-01-sonar-sweep`).

```
advent-of-code-2021
│   .gitignore
│   .prettierrc
│   package.json
│   yarn.lock
│
├───node_modules
└───src
    └───day-01-sonar-sweep
            input.txt
            one.js
            two.js
```

## Input Reader Utility
Reading input data from a file is a common need when solving any of the challenges. Thus, I created a utility function that accepts a file path as a parameter and returns a `Promise` with the data as a string.
```js
// src/util/readInput.js
import fs from 'fs'

export const readInput = (inputPath) => {
  return new Promise((resolve, reject) =>
    fs.readFile(inputPath, 'utf8', (err, data) => {
      if (err) {
        console.error(err)
        reject(err)
      }
      resolve(data)
    })
  )
}
```

We can then import and use the `readInput` utility in each solution file (e.g. `one.js`):
```js
import { readInput } from '../utils/readInput.js'

const data = await readInput(inputPath)
```

The input path is relative to the solution file. We can use the `url` module from Node.js to build the path needed for the file reader.
```js
import { URL, fileURLToPath } from 'url'

const inputPath = fileURLToPath(new URL('./input.txt', import.meta.url))
```

## Solving Part One
You can read the problem on [Advent of Code Day 1](https://adventofcode.com/2021/day/1).
Given an array of depths, the objective is to compare every two adjacent depths and count when the depth increases.

To solve this problem, we first need to parse the contents of the input file. Since each number is given on a new line we can use the `split` function with the new line character `\n` as a separator to get an array of strings representing the contents of each line. Then we convert each line to a number.
```js
const depths = data.split('\n').map((line) => Number(line))
```
Since the first depth does not increase or decrease, we can remove it from the array and store it with the purpose of comparing it to the next depth.
```js
let previousDepth = depths.shift()
```
The depth increases when `depth - previousDepth > 0` is true.
We can iterate the array and count how many times the depth increases using `forEach` and a counter variable.

```js
let increaseCount = 0

depths.forEach((depth) => {
  if (depth - previousDepth > 0) {
    increaseCount += 1
  }
  previousDepth = depth
})
```
Put together, the solution is:

```js
import { URL, fileURLToPath } from 'url'
import { readInput } from '../utils/readInput.js'

const getIncreaseCount = (data) => {
  const depths = data.split('\n').map((i) => Number(i))
  let previousDepth = depths.shift()
  let increaseCount = 0

  depths.forEach((depth) => {
    if (depth - previousDepth > 0) {
      increaseCount += 1
    }
    previousDepth = depth
  })
  return increaseCount
}

const inputPath = fileURLToPath(new URL('./input.txt', import.meta.url))

const data = await readInput(inputPath)

console.log(getIncreaseCount(data))

```

## Solving Part Two
The second part has the objective of comparing sums of three depths to the next set of three depths. For example, given the depths a, b, c, d, e. We would have to compare:
- a + b + c and b + c + d
- b + c + d and c + d + e

Since we subtract the sums to find whether the depth increases or decreases, the problem can be reduced to compare
- a and d
- b and e

In other words, we need to compare every two depths positioned three indexes apart.
```js
let increaseCount = 0

for(let i = 0; i < depths.length - 2; i++) {
  if (depths[i + 3] - depths[i] > 0) {
    increaseCount += 1
  }
}
```
You can see the final solution on my [GitHub repository](https://github.com/mancristiana/advent-of-code-2021/blob/main/src/day-01-sonar-sweep/two.js).

## Summary
Day 1 consisted of setting up a Node.js project with Yarn and Prettier, creating an input reader utility, parsing numbers from an input file, iterating through a number array and comparing every two numbers positioned either one or three indexes apart.