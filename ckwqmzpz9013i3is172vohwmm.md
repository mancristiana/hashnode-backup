## Advent of Code 2021: Day 3

This blog post is third in the Advent of Code 2021 series and shows a JavaScript-based solution to the problem described in [Day 3](https://adventofcode.com/2021/day/3).

All solutions are in this [GitHub repository](https://github.com/mancristiana/advent-of-code-2021).

## Solving Part One
The input file consists of binary numbers on separate lines. We must find the so-called gamma rate and epsilon rate, multiply them and return the resulting number in decimal format. Gamma rate is a binary number where each bit represents the most common bit in its position among all given numbers. Similarly, the epsilon rate contains the least common bit for each position.

We can break this problem into smaller tasks:
- read lines from the input file
- find how many bits each number has
- find the most common bit for a given position 
- find the least common bit for a given position
- concatenate found bits
- parse the binary string to decimal

### Read lines from the input file
I wrote about the `readInput` function in my (Advent of Code: Day 1)[https://blog.cristiana.tech/advent-of-code-2021-day-1] blog post. If you haven't already, give it a read!

```js
import { URL, fileURLToPath } from 'url'
import { readInput } from '../utils/readInput.js'

const inputPath = fileURLToPath(new URL('./input.txt', import.meta.url))
const data = await readInput(inputPath)
```

We can then get each line containing a binary number by splitting the input data using the new line character `\n`
```js
const lines = data.split('\n')
```

### Find how many bits each number has
One of the constraints of this problem is that each number is represented using the same number of bits. Thus, we can calculate the length of the first line to find how many bits each number has.
```js
const numberOfBits = lines[0].length
```
Now for each position, we must find the most and least common bits
```js
for (let i = 0; i < numberOfBits; i++) {
  // find most common bit for position i
  // find least common bit for position i
}
```
### Find the most common bit for a given position 
We must iterate through each line and investigate the bit at the given position. To get the bit on a given position of a line we can use `line.charAt(index)`. We can then count the occurrences of `0` and `1` and compare them in the end to determine the most common bit.

```js
const getMostCommonBit = (lines, index) => {
  let occurenceOf0 = 0
  let occurenceOf1 = 0
  lines.forEach((line) => {
    const bit = line.charAt(index)
    if (bit === '0') {
      occurenceOf0 += 1
    } else {
      occurenceOf1 += 1
    }
  })
  return occurenceOf0 > occurenceOf1 ? '0' : '1'
}
```

Since `occurenceOf0 > occurenceOf1` can be written `occurenceOf0 - occurenceOf1 > 0` we don't actually need the exact count of occurrences, only the difference. Thus, we can refactor the code like so:

```js
const getMostCommonBit = (lines, index) => {
  let occurenceDifference = 0
  lines.forEach((line) => {
    const bit = line.charAt(index)
    if (bit === '0') {
      occurenceDifference += 1
    } else {
      occurenceDifference -= 1
    }
  })
  return occurenceDifference > 0 ? '0' : '1'
}
```
### Find the least common bit for a given position 
After finding the most common bit, we can determine the least common bit as the opposite. For example, if the most common bit is `0` we know the least common is `1` and vice versa. 
```js
const mostCommonBit = getMostCommonBit(lines, i)
const leastCommonBit = mostCommonBit === '0' ? '1' : '0'
```

### Concatenate found bits
```js
let gammaRate = ''
let epsilonRate = ''

for (let i = 0; i < numberOfBits; i++) {
  const mostCommonBit = getMostCommonBit(lines, i)
  gammaRate += mostCommonBit
  epsilonRate += mostCommonBit === '0' ? '1' : '0'
}
```
### Parse the binary string to decimal
After finding the `gammaRate` and `epsilonRate` we can parse them to decimal using the `parseInt` function.
```js
parseInt(gammaRate, 2)
parseInt(epsilonRate, 2)
```

You can see the final solution for part one my [GitHub repository](https://github.com/mancristiana/advent-of-code-2021/blob/main/src/day-03-binary-diagnostic/one.js)

## Solving Part Two
I suggest thoroughly reading and understanding the [problem](https://adventofcode.com/2021/day/3#part2) before reading further. 
To solve the second part of the challenge, we need to repeatedly filter the given binary numbers based on a bit criteria until there a single number is left. We must find the so-called oxygen generator rating (`oxygenRate`) and the CO2 scrubber rating (`co2Rate`) using this filtering method.

There are a few steps to solve the challenge:
- find the bit criteria for `oxygenRate` and `co2Rate`
- filter numbers until there is a single item left
- parse the binary string to decimal

### Find the bit criteria
The bit criteria for `oxygenRate` is the most common bit in position X among filtered numbers, where X starts at 0 and increases each filtering round. Similarly, the bit criteria for `co2Rate` is the least common bit.

We can reuse the `getMostCommonBit` function created for part one, but with changes to accommodate the additional rule - if there is an equal number of bits, the criteria should default to `1` for `oxygenRate` and `0` for `co2Rate`.  

```js
const getOccurenceDifference = (lines, index) => {
  let occurenceDifference = 0
  lines.forEach((line) => {
    const bit = line.split('')[index]
    if (bit === '0') {
      occurenceDifference += 1
    } else {
      occurenceDifference -= 1
    }
  })
  return occurenceDifference
}
```

The `getOccurenceDifference` returns a positive number if `0` is most common, a negative number if `1` is most common or `0` if both bits are equally common. 
Thus, the bit criteria for `oxygenRate` is defined as follows and represents the most common bit or if both bits are equally common defaults to `1`.
```js
getOccurenceDifference(oxygenRates, index) > 0 ? '0' : '1'
```
The bit criteria for `co2Rate` represents the least common bit or if both bits are equally common defaults to `0`.
```js
getOccurenceDifference(co2Rates, index) > 0 ? '1' : '0'
```

### Filter numbers until there is a single item left
The first filtering round starts with all lines for both `oxygenRates` and `co2Rates`. We can copy the lines representing the binary numbers into each corresponding array.

```js
let oxygenRates = [...lines]
let co2Rates = [...lines]
```
We repeatedly filter these numbers until there is a single item left. Each time we must increase the index of bit we compare with `criteriaBit`.
```js
let index = 0
while (oxygenRates.length > 1) {
  const bitCriteria = getOccurenceDifference(oxygenRates, index) > 0 ? '0' : '1'
  oxygenRates = oxygenRates.filter((line) => line.charAt(index) === bitCriteria)
  index++
}
```
Similarly, we filter the `co2Rates`.
```js
index = 0
while (co2Rates.length > 1) {
  const bitCriteria = getOccurenceDifference(co2Rates, index) > 0 ? '1' : '0'
  co2Rates = co2Rates.filter((line) => line.charAt(index) === bitCriteria)
  index++
}
```

### Parse the binary string to decimal

Finally, we can parse the first and only element in `oxygenRates` and `co2Rates` in the same manner we did for part one.

```js
parseInt(oxygenRates[0], 2)
parseInt(co2Rates[0], 2)
```

You can see the final solution for part two my [GitHub repository](https://github.com/mancristiana/advent-of-code-2021/blob/main/src/day-03-binary-diagnostic/two.js)

## Conclusion
Day 3 was about thoroughly reading the problem, understanding it and breaking it down into smaller parts. We processed strings representing binary numbers and relied on JavaScript functions like `parseInt(binaryString, 2)`, `String.charAt(index)`, `array.filter(condition)` to solve the challenges.