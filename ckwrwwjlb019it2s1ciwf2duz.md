## Advent of Code 2021: Day 4

This blog post is fourth in the Advent of Code 2021 series and shows a JavaScript-based solution to the problem described in [Day 4](https://adventofcode.com/2021/day/4). The stories never stop surprising me. This one, in particular, is about playing bingo with a giant squid! Consider me entertained 😎

All solutions are in this [GitHub repository](https://github.com/mancristiana/advent-of-code-2021).

## Solving Part One
We need to find the first winning bingo board and calculate the board's score. A winning bingo board contains an entire row or column of extracted random numbers.

The strategy of breaking down the problem into smaller, more manageable chunks worked well for day 3, so let's explore the steps for solving the fourth challenge:
- read the input file
- mark bingo numbers on each board
- check if a row is complete
- check if a column is complete
- calculate the score

### Read the input file
The input contains the extracted random numbers on the first line, followed by 5 x 5 boards of numbers separated by empty lines.  We can extract the bingo numbers and boards as strings using the double Newline separator.
```js
const [bingoNumbersAsString, ...boardsAsString] = data.split('\n\n')
```
We can extract the array of bingo numbers by using the `,` as a separator and parsing each number.
```js
const bingoNumbers = bingoNumbersAsString
    .split(',')
    .map((numberAsString) => Number(numberAsString))
```
Next, we must store board numbers in a data structure. I opted to keep each board as an array of 25 objects containing the number and a `marked` boolean. 

```js
{
  number: 4,
  marked: false,
}
```

The `marked` property is set to `false` by default and `true` whenever the number matches the played bingo number. A matrix with objects of the same structure would have worked well too. However, I thought it would be easier to traverse an array to find the matching bingo number rather than a matrix.

Here is the mapping of the input to the desired data structure.
```js
const boards = boardsAsString.map((boardAsString) =>
  boardAsString
    .replaceAll('\n', ' ')
    .split(' ')
    .filter(Boolean)
    .map((numberAsString) => ({
      number: Number(numberAsString),
      marked: false,
    }))
)
```

All new lines are concatenated separated by a space using ` .replaceAll('\n', ' ')`, then `.split(' ')` creates an array using the space character as a separator. Since some of the numbers have additional spaces, some of the items in the array are empty strings.  The `.filter(Boolean)` function removes all the empty strings from the array. Finally, each number is parsed and mapped into the data structure.

### Mark bingo numbers on each board
We can use the `forEach` function to iterate through bingo numbers and boards. However, `forEach` goes through all elements, whereas we should stop whenever we find a winning board. The `some` function returns true when finding a winning board.

```js
bingoNumbers.some((bingoNumber) => {
    return boards.some((board) => {
      // mark the bingo number on the board
      // determine if it is a winning board
      return isWinner
    })
```
To find and mark the bingo number on a board we can use the `find` function and set the marked property to `true` when the board contains the number.
```js
bingoNumbers.some((bingoNumber) => {
    return boards.some((board) => {
      const bingoNumberOnBoard = board.find((item) => item.number === bingoNumber)
      if (!bingoNumberOnBoard) {
        return
      }
      bingoNumberOnBoard.marked = true
      // determine if it is a winning board
      return isWinner
    })
```
To determine whether the board is winning, we must check if the row or column of the marked number is now complete. Both checks require the index of the `bingoNumberOnBoard`.
```js
const bingoNumberIndexOnBoard = board.findIndex(item) => item === bingoNumberOnBoard)
```

### Check if a row is complete
Given that each board is 5 x 5, we can use the index of the bingo number in the array to determine indices of the elements of the row. 
Each board has the following indices:
```
 0  1  2  3  4
 5  6  7  8  9
10 11 12 13 14
15 16 17 18 19
20 21 22 23 24
```

We can determine the indices of the row of the bingo number with the formula `rowIndex * 5 + columnIndex`. The `rowIndex` is the quotient of the bingo number index divided by `5` and the column indices are always `0`, `1`, `2`, `3`, `4` . For example, for  the index 13, the row index is `13 ÷ 5` which is `2`, thus the indices are `2 * 5 + 0` (`10`), `2 * 5 + 1` (`11`), `2 * 5 + 2` (`12`), `2 * 5 + 3` (`13`) and `2 * 5 + 4` (`14`).

Then, we only need to count how many marked elements there are on the row and return `true` if there are `5` marked elements and `false` otherwise.
```js
const checkRow = (board, bingoNumberIndexOnBoard) => {
  const rowIndex = Math.floor(bingoNumberIndexOnBoard / 5)
  const reducer = (markedTotal, columnIndex) => {
    const boardIndex = rowIndex * 5 + columnIndex
    if (board[boardIndex].marked) {
      return markedTotal + 1
    }
    return markedTotal
  }
  const markedTotal = [0, 1, 2, 3, 4].reduce(reducer, 0)
  return markedTotal === 5
}
```

In the `solve` function we can call the `checkRow` function like this:
```js
const isRow = checkRow(board, bingoNumberIndexOnBoard)
```


### Check if a column is complete
The column check is calculated in a similar manner to the row, except we determine the indices slightly differently. The row indices are always `0`, `1`, `2`, `3`, `4` and the column index is the remainder of the bingo number index divided by `5`.
```js
const checkColumn = (board, bingoNumberIndexOnBoard) => {
  const columnIndex = bingoNumberIndexOnBoard % 5
  const reducer = (markedTotal, rowIndex) => {
    const boardIndex = rowIndex * 5 + columnIndex
    if (board[boardIndex].marked) {
      return markedTotal + 1
    }
    return markedTotal
  }
  const markedTotal = [0, 1, 2, 3, 4].reduce(reducer, 0)
  return markedTotal === 5
}
``` 

In the `solve` function we can call the `checkColumn` function like this:
```js
const isColumn = checkColumn(board, bingoNumberIndexOnBoard)
```

### Calculate the score
The score is calculated by summing all the unmarked numbers and multiplying the sum with the bingo number. We can use the `reduce` function as follows:
```js
const calculateScore = (board, bingoNumber) => {
  const boardScore = board.reduce((boardScore, item) => {
    if (item.marked) {
      return boardScore
    }
    return boardScore + item.number
  }, 0)
  return boardScore * bingoNumber
}
```

In the `solve` function we can call the `calculateScore` function if we found a winning board:
```js
const isWinner = isRow || isColumn
if (isWinner) {
  score = calculateScore(board, bingoNumber)
}
```

You can see the final solution for part one on my [GitHub repository](https://github.com/mancristiana/advent-of-code-2021/blob/main/src/day-04-giant-squid/one.js).

## Solving Part Two
We need to find the last winning bingo board. We can reuse most of the code defined in part one, such as reading input, checking whether a row or column is complete and calculating the score.

Here is how we could break down the differences:
- iterate through all bingo numbers
- mark and skip winning boards

### Iterate through all bingo numbers
The main difference from part one is that instead of stopping after finding the first winning board, we must go through all bingo numbers until all boards have at least one complete row or column. Thus, `forEach` works well in this case.
```js
bingoNumbers.forEach((bingoNumber) => {
  boards.forEach((board) => {
     // ...
  })
})
```

### Mark and skip winning boards
We must keep track of winning boards to prevent continued markings and secondary winning. We can add a `won` property to each board.
```js
boards = boards.map((board) => ({
  won: false,
  board
}))
```

We must refactor the code to account for this data structure change. In addition, we skip marking numbers on winning boards and set the `won` property to `true` whenever we find a new winning board.
```js
let lastWinningScore = 0

bingoNumbers.forEach((bingoNumber) => {
  boards.forEach((boardObject) => {
    const { board, won } = boardObject
    if (won) {
        return
      }
    // ...
    
    if (isWinner) {
      boardObject.won = true
      lastWinningScore = calculateScore(board, bingoNumber)
    }
  })
})
```

You can see the final solution for part two on my [GitHub repository](https://github.com/mancristiana/advent-of-code-2021/blob/main/src/day-04-giant-squid/two.js).

## Conclusion
We wrote some code that simulates playing bingo. We determined the first and last winning boards, and we got to use a variety of JavaScript array functions, such as `some`, `forEach`, `reduce`, `map`, etc. As usual, I had fun with this challenge, and I am looking forward to day 5.