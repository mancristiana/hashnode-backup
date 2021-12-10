## Advent of Code 2021: Day 10

This blog post is the tenth in the Advent of Code 2021 series and shows a JavaScript-based solution to the problem described in [Day 10](https://adventofcode.com/2021/day/10).

All solutions are in this [GitHub repository](https://github.com/mancristiana/advent-of-code-2021).

## Solving Part One
Given a set of lines with opening and closing brackets, we must find syntax errors and calculate their score.

My strategy for solving this challenge is using a FILO (First In, Last Out) data structure that stores open brackets. We add new open brackets to the array. If a bracket is closed correctly, we pop the last open bracket from the array. Otherwise, if the closing bracket does not correspond to the previous open bracket, we have found a syntax error, and we can calculate its score.

We can break down the solution like so:
- outline the algorithm that sums syntax error scores
- add open brackets to the FILO data structure
- remove open brackets when the corresponding closing bracket matches
- calculate illegal bracket score when the corresponding closing bracket does not match

### Outline the algorithm that sums syntax error scores
```js
const lines = data.split('\n')
return lines.reduce((sum, line) => sum + getErrorScore(line), 0)
```

```js
const getErrorScore = (line) => {
  const brackets = line.split('')
  let score = 0
  brackets.some(bracket => {
    // If there is a syntax error overwrite score
  })

  return score
}
```

### Add open brackets to the FILO data structure
To check whether we reached an open bracket we can use the `include` function on the open brackets array like so:
```js
const isOpeningBracket = (bracket) => ["{", "[", "(", "<"].includes(bracket)
```

We can then use the `isOpeningBracket` function to determine whether to add a new item to the `openBrackets` FILO data structure.

```js
const getErrorScore = (line) => {
  const brackets = line.split('')
  const openBrackets = []
  let score = 0
  brackets.some(bracket => {
    if (isOpeningBracket(bracket)) {
      openBrackets.push(bracket)
    }
    // If there is a syntax error overwrite score
  })
  return score
}
```

### Remove open brackets when the corresponding closing bracket matches

We can define matching brackets like so:
```js
const matchingBrackets = {
  "(": ")",
  "[": "]",
  "{": "}",
  "<": ">"
}
```
We can use the following function to check whether a closing bracket matches an existing open bracket. We also check whether the opening bracket exists because a closing bracket written before any opening bracket represents a syntax error.
```js
const isCorrectClosingBracket = (openingBracket, closingBracket) => {
  if (!openingBracket) {
    return false
  } 
  return matchingBrackets[openingBracket] === closingBracket
}
```

We can use the `pop` function to remove the last element in the FILO structure.
```js
if (isOpeningBracket(bracket)) {
  openBrackets.push(bracket)
} else if(isCorrectClosingBracket(openBrackets[openBrackets.length - 1], bracket)) {
  openBrackets.pop()
}
```
### Calculate illegal bracket score when the corresponding closing bracket does not match
The illegal bracket has a defined score as follows:
```js
const illegalBracketScoring = {
  ")": 3,
  "]": 57,
  "}": 1197,
  ">": 25137
}
```

We can pop the last element of the `openBrackets` regardless of whether the closed bracket is correct or not. Thus, we can rewrite the conditions from the `getErrorScore` function as follows:
```js
let score = 0
brackets.some(bracket => {
  if (isOpeningBracket(bracket)) {
    openBrackets.push(bracket)
  } else if(!isCorrectClosingBracket(openBrackets.pop(), bracket)) {
    score = illegalBracketScoring[bracket]
    return true 
  }
})
return score
```

The final solution for part one is in my [GitHub repository](https://github.com/mancristiana/advent-of-code-2021/blob/main/src/day-10-syntax-scoring/one.js).

## Solving Part Two
Part two is about filtering the corrupted lines identified in part one and focusing on the incomplete lines. The goal is to find the missing brackets of incomplete lines and calculate a score corresponding to each completed line. We must sort all scores to return the expected score in the middle.

We can reuse a great deal of the solutions from part one with a few additions:
- filter corrupted lines using the error score
- use opening brackets to calculate the completion score
- find the middle completion score

### Filter corrupted lines using the error score
We can reuse the `getErrorScore` function from part one, with a few adjustments:
- remove the score calculation since it is unnecessary for part two
- return a boolean `hasError` set to `true` when the line has a syntax error
- return the array of open brackets
- rename function to `getOpeningBrackets` to better fit the goal of part two

```js
const getOpeningBrackets = (line) => {
  const brackets = line.split('')
  const openingBrackets = []
  const hasError = brackets.some((bracket) => {
    if (isOpeningBracket(bracket)) {
      openingBrackets.push(bracket)
    } else if (!isCorrectClosingBracket(openingBrackets.pop(), bracket)) {
      return true
    }
  })

  return [hasError, openingBrackets]
}
```

We can call the `getOpeningBrackets` function to filter the lines with syntax errors. The `reduce` function returns the array of completed line scores by adding a calculated score for each line without a syntax error.

```js
const completedLineScores = lines.reduce((completed, line) => {
  const [hasError, openingBrackets] = getOpeningBrackets(line)
  if (hasError) {
    return completed
  } else {
    return [...completed, getCompletionScore(openingBrackets)]
  }
}, [])
```

### Use opening brackets to calculate the completion score
Each closing bracket has the following score:
```js
const completionScoring = {
  ')': 1,
  ']': 2,
  '}': 3,
  '>': 4,
}
```

We must traverse open brackets in reverse order to find the matching closing bracket and calculate its corresponding score.
```js
const getCompletionScore = (openingBrackets) => {
  return openingBrackets
    .reverse()
    .reduce(
      (score, bracket) =>
        score * 5 + completionScoring[matchingBrackets[bracket]],
      0
    )
}
```
### Find the middle completion score
Once we have sorted the array of scores, we can determine the middle score using the middle of the array length as index.
```js
completedLineScores.sort((a, b) => a - b)
const middleIndex = Math.floor(completedLineScores.length / 2)
return completedLineScores[middleIndex]
```

The final solution for part two is in my [GitHub repository](https://github.com/mancristiana/advent-of-code-2021/blob/main/src/day-10-syntax-scoring/two.js).

## Conclusion
Day 10 was about processing lines of brackets, filtering lines with syntax errors and completing lines without errors. A FILO (First In, Last Out) data structure helped match open and close brackets.
