## Advent of Code 2021: Day 6

This blog post is sixth in the Advent of Code 2021 series and shows a JavaScript-based solution to the problem described in [Day 6](https://adventofcode.com/2021/day/6). The challenge was creating and optimizing an algorithm for calculating the lanternfish population after a number of days.

All solutions are in this [GitHub repository](https://github.com/mancristiana/advent-of-code-2021).

## Solving Part One
We have an initial set of fish with given internal times until they multiply. 
According to the problem, after creating a new fish, each fish resets their internal time to 6 days since they need a week to make a new fish. A new fish starts with an internal time of 9 days until multiplying the first time. The challenge is to calculate the amount of fish after 80 days.

We can solve the problem with the following steps:
- read the input
- update the state of each fish after a day
- add new fish
- repeat for 80 days and count fish

### Read the input
The input contains comma-separated numbers, which represent the state of each fish. 
We can use an array to store each fish as a number.
```js
let fish = data.split(',').map((n) => Number(n))
```

### Update the state of each fish after a day
Without considering new fish for now, after a day, each fish's internal time decreases by one until it reaches 0, at which point it resets to 6.
```js
const newFishPopulation = fish.map((f) => 
{
  if (f === 0) {
    return 6
  }
  return f - 1
})
```

### Add new fish
We can count the amount of fish that reset to 6 and add that amount to the list of new fish population.
```js
const getNewFishPopulation = (fish) => {
  let newFishCount = 0
  const newFishPopulation = fish.map((f) => {
    if (f === 0) {
      newFishCount++
      return 6
    }
    return f - 1
  })
  newFishPopulation.push(...Array(newFishCount).fill(8))
  return newFishPopulation
}
```
### Repeat for 80 days and count fish
The challenge is to find the amount of fish after 80 days, so we can write a for loop which would update the fish array every day.

```js
 for (let day = 1; day <= 80; day++) {
  fish = getNewFishPopulation(fish)
 }
 return fish.length
```

You can see the final solution for part one on my [GitHub repository](https://github.com/mancristiana/advent-of-code-2021/blob/main/src/day-06-lanternfish/one.js).

## Solving Part Two
Part two is about optimizing the algorithm to calculate the fish population after 256 days. The solution from part one is not efficient in terms of memory. It exceeds the maximum number of elements allowed in an array, thus throwing the following range error.
```
fish.push(...Array(newFishCount).fill(8))
         ^
RangeError: Maximum call stack size exceeded
```
While thinking about this problem, it occurred to me that fish with the same state produces the same amount of ancestors. For example, two fishes left with three days until reproduction create the same amount of ancestors after 80 or any amount of days. 
Thus, it would be enough to keep track of the amount of fish in a given state each day. So I opted for an occurrence array with nine elements, where each index represents the state of days until reproduction and stores the amount of fish with that corresponding state.

We can break the problem like this:
- initialize occurrence array from input
- shift occurrence array one position
- add fish corresponding to state 6
- add new fish
- repeat for 256 days and count fish

```js
import { URL, fileURLToPath } from 'url'
import { readInput } from '../utils/readInput.js'

const sumFish = (occurenceArray) => {
  return occurenceArray.reduce((sum, fishCount) => sum + fishCount, 0)
}

const solve = (data) => {
  const fish = data.split(',').map((n) => Number(n))
  const fishOccurence = Array(9).fill(0)

  fish.forEach((f) => fishOccurence[f]++)

  for (let day = 1; day <= 256; day++) {
    const newFishCount = fishOccurence.shift()
    fishOccurence[6] += newFishCount
    fishOccurence.push(newFishCount)
  }
  return sumFish(fishOccurence)
}

const inputPath = fileURLToPath(new URL('./input.txt', import.meta.url))

const data = await readInput(inputPath)

console.log(solve(data))

```

## Conclusion
Day 6 offered a challenge in terms of optimizing an algorithm. Although it stored duplicate elements, a simple solution worked well for part one. However, part two required using an occurrence array instead.