## Advent of Code 2021: Day 2

This blog post is second in the Advent of Code 2021 series and shows a JavaScript-based solution to the problem described in [Day 2: Dive!](https://adventofcode.com/2021/day/2).

All solutions are in this [GitHub repository](https://github.com/mancristiana/advent-of-code-2021).

## Solving Part One
The goal is to find the horizontal position and depth of a moving submarine based on a set of given instructions.

According to the problem the initial depth and horizontal positions are `0`.
```js
let depth = 0
let horizontalPosition = 0
```

We can read the set of instructions using the `readInput` utility created on Day 1. Then we can split the data line by line.
```js
const commands = data.split('\n')
```
Each line contains a command type and a number of steps. To use these, we can split the line by the space character, then convert the steps to a number.

```js
const [type, stepsAsString] = command.split(' ')
const steps = Number(stepsAsString)
```

Finally, we can calculate the depth and horizontal positions based on the command type and steps to move according to the rules described in the challenge. 
```js
 switch (type) {
  case 'forward':
    horizontalPosition += steps
    break
  case 'down':
    depth += steps
    break
  case 'up':
    depth -= steps
    break
  default:
    break
}
```

The final solution:
```js
import { URL, fileURLToPath } from 'url'
import { readInput } from '../utils/readInput.js'

const solve = (data) => {
  const commands = data.split('\n')
  let depth = 0
  let horizontalPosition = 0

  commands.forEach((command) => {
    const [type, stepsAsString] = command.split(' ')
    const steps = Number(stepsAsString)

    switch (type) {
      case 'forward':
        horizontalPosition += steps
        break
      case 'down':
        depth += steps
        break
      case 'up':
        depth -= steps
        break
      default:
        break
    }
  })
  return depth * horizontalPosition
}

const inputPath = fileURLToPath(new URL('./input.txt', import.meta.url))

const data = await readInput(inputPath)

console.log(solve(data))

```

## Solving Part Two
The second part of the problem requires calculating depth and horizontal position in a slightly different way using new rules and an aim value.
```js
let aim = 0
```

The new rules can be written in JavaScript as follows:
```js
switch (type) {
  case 'forward':
    horizontalPosition += steps
    depth += aim * steps
    break
  case 'down':
    aim += steps
    break
  case 'up':
    aim -= steps
    break
  default:
    break
}
```

The final solution for part two is on my [GitHub repository](https://github.com/mancristiana/advent-of-code-2021/blob/main/src/day-02-dive/two.js).

## Conclusion
Day 2 was about calculating values such as horizontal position and depth of a moving submarine based on a set of given commands. I had fun solving the challenge and the utility functions added on Day 1 helped solve the second challenge somewhat faster.