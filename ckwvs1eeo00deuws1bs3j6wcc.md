## Advent of Code 2021: Day 7

This blog post is seventh in the Advent of Code 2021 series and shows a JavaScript-based solution to the problem described in [Day 7](https://adventofcode.com/2021/day/7). Given many horizontal positions for submarines navigated by crabs, we must align them using the least amount of fuel. Yes, it is another entertaining story!

All solutions are in this [GitHub repository](https://github.com/mancristiana/advent-of-code-2021).

## Solving Part One
The alignment position must be between the minimum and maximum horizontal submarine positions. We can then calculate the needed fuel for each alignment position between these thresholds. 

We can break down the solution as follows:
- read positions from input
- find the minimum and maximum positions
- calculate the fuel corresponding to each alignment
- find the minimum fuel

### Read positions from input
The `readInput` function created on [Day 1](https://blog.cristiana.tech/advent-of-code-2021-day-1) is useful for getting the file `data` as string. We can then map this data to a number array. 
```js
const positions = data.split(',').map((n) => Number(n))
```
### Find the minimum and maximum positions
Since solving Advent of Code challenges, I have noticed the `reduce` function is very handy for many calculations based on elements in an array. Finding the minimum and maximum number in an array is one of those cases. I also relied on `Math.min` and `Math.max` to avoid writing conditional statements, thus additional lines of code. 
```js
const [minAlignPos, maxAlignPos] = positions.reduce(
  ([min, max], pos) => [Math.min(min, pos), Math.max(max, pos)],
  [positions[0], positions[0]]
)
```
### Calculate the fuel corresponding to each alignment
We can iterate through each alignment between the maximum and minimum positions using a for loop.
```js
for (let alignPos = minAlignPos; alignPos <= maxAlignPos; alignPos++) {
  const fuel = calculateFuel(positions, alignPos)
}
```

We can use the reduce function again to calculate the fuel required for aligning all positions. We must sum the differences between each position and the alignment position.
```js
const calculateFuel = (positions, alignPos) => {
  return positions.reduce((fuel, pos) => fuel + Math.abs(pos - alignPos), 0)
}
```

### Find the minimum fuel
After calculating the fuel for each alignment, we can determine the minimum fuel needed by comparing each fuel amount with the `minFuel` variable and updating it accordingly.
```js
let minFuel

for (let alignPos = minAlignPos; alignPos <= maxAlignPos; alignPos++) {
  const fuel = calculateFuel(positions, alignPos)
  if (!minFuel || fuel < minFuel) {
    minFuel = fuel
  }
}

return minFuel
```

The final solution for part one is in my [GitHub repository](https://github.com/mancristiana/advent-of-code-2021/blob/main/src/day-07-the-treachery-of-whales/one.js).

## Solving Part Two
In part two, the problem describes a different way of calculating the fuel. Instead of a constant amount of fuel, the required fuel increases with 1 unit each step. Thus, aligning `n` steps would require `1 + 2 + 3 + ... + n` amount of fuel. We can use Gauss's summation and calculate the steps as follows `n * ( n + 1 ) / 2`.

```
const calculateFuelForSteps = (n) => (n * (n + 1)) / 2

const calculateFuel = (positions, alignPos) => {
  return positions.reduce(
    (fuel, pos) => fuel + calculateFuelForSteps(Math.abs(pos - alignPos)),
    0
  )
}
```

The final solution for part two is in my [GitHub repository](https://github.com/mancristiana/advent-of-code-2021/blob/main/src/day-07-the-treachery-of-whales/two.js).

## Conclusion
We found the most fuel-efficient way of aligning a set of crab submarines given their horizontal positions. I had fun reading and solving the challenge of Day 7 and am looking forward to Day 8!