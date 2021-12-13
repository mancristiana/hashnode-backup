## Advent of Code 2021: Day 13

This blog post is thirteenth in the Advent of Code 2021 series and shows a JavaScript-based solution to the problem described in [Day 13](https://adventofcode.com/2021/day/13).

All solutions are in this [GitHub repository](https://github.com/mancristiana/advent-of-code-2021).

## Solving Part One
Given a list of `x`, `y` coordinates representing dots on a transparent Origami paper along with folding instructions, we must determine the number of visible dots after the first fold.

I broke down the solution as follows:
- read input into a dot coordinate hash map
- read the first instruction
- update dots after folding left
- update dots after folding up
- count dots

### Read input into a dot coordinate hash map
In terms of the data structure for the Origami paper, I opted for a hash map that stores dot coordinates. An alternative data structure is a matrix of characters `.` and `#`. However, since the problem doesn't specify constraints about the paper size and since the matrix size changes at each step, I deemed this approach more difficult to implement.

The first step of reading the input is separating the dot coordinates from the instructions using the empty line separator `\n\n`.
```js
const [dotsString, instructionsString] = data.split('\n\n')
```

Next, we add each dot to the hash map using the `x,y` string as a key and `{x, y}` as value.

```js
const dots = {}
dotsString.split('\n').map((dot) => {
  const [x, y] = dot.split(',').map((n) => parseInt(n))
  dots[dot] = { x, y }
})
```

For example, given the `1,7` coordinates, we store:
```js
const dots = {
  '1,7': {
    x: 1,
    y: 7
  }
}
```

### Read the first instruction
We can get the first line among instructions using the `split` function and the 0 index.
```js
const firstInstruction = instructionsString.split('\n')[0]
```

To interpret the folding instruction we need the axis (`x` or `y`) and the line coordinate. Thus, we can remove the `fold along ` string.

```js
const [axis, lineString] = firstInstruction
  .replace('fold along ', '')
  .split('=')

const line = parseInt(lineString)
```

According to the problem, axis `x` is a vertical line, thus we must fold the paper to the left, whereas given axis `y` is a horizontal line, we must fold the paper up. We define the `foldLeft` and `foldUp` functions next.

```js
if (axis === 'x') {
  foldLeft(dots, line)
} else {
  foldUp(dots, line)
}
```

### Update dots after folding left

We must inspect each dot coordinate. We can use the `Object.key` function to iterate through the elements of the dot hash map. We need to determine whether a dot is on the right half of the paper relative to the folding line. This condition is met when the `x` coordinate of the dot is larger than the `x` coordinate representing the line (`x > line`).

Each dot on the right half of the paper changes its `x` coordinate after folding the paper by twice the difference between itself and the line. We can express this as `const newX = x - 2 * (line - x)`
For example, the folding line `x=7` updates the `x` coordinate of dot `9,1` from `9` to `9 - 2 * (9 - 7) = 5` resulting in dot `5,1`. We can simplify the expression to `const newX = 2 * line - x`.

Once we have determined the new coordinates, we can add the new dot to the hash map and delete the dot with old coordinates.

```js
const foldLeft = (dots, line) => {
  Object.keys(dots)
  .forEach((key) => {
    const { x, y } = dots[key]
    if (x > line) {
      const newX = 2 * line - x
      dots[getKey(newX, y)] = { x: newX, y }
      delete dots[key]
    }
  })
}
```

### Update dots after folding up
To fold up, we can use the same algorithm as for folding to the left, except we update the `y` coordinate instead of `x`.

```js
const foldUp = (dots, line) => {
  Object.keys(dots)
    .forEach((key) => {
      const { x, y } = dots[key]
      if (y > line) {
        const newY = 2 * line - y
        dots[getKey(x, newY)] = { x, y: newY }
        delete dots[key]
      }
    })
}
```
### Count dots
We can rely on the `Object.keys` function to count the number of visible dots after folding the Origami paper.
```js
Object.keys(dots).length
```

The final solution for part one is in my [GitHub repository](https://github.com/mancristiana/advent-of-code-2021/blob/main/src/day-13-transparent-origami/one.js).

## Solving Part Two
Part two was about following all folding instructions, printing the final Origami result and revealing eight uppercase characters - the expected output.

To solve part two we can build on top of part one with the following additions:
- fold paper according to all instructions
- print folded Origami paper

### Fold paper according to all instructions
Unlike in part one, we must iterate through and follow all folding instructions one by one.
```js
 instructionsString.split('\n').forEach((instruction) => {
  const [axis, lineString] = instruction.replace('fold along ', '').split('=')
  const line = parseInt(lineString)
  if (axis === 'x') {
    foldLeft(dots, line)
  } else {
    foldUp(dots, line)
  }
})
```
### Print folded Origami paper
Once we have the final coordinates of dots on the folded Origami paper, we are able to print it.
We need to know the size of the folded Origami paper to be able to print it. Thus, we must find the maximum `x` and `y` coordinates.

```js
const [maxX, maxY] = Object.keys(dots).reduce(
  ([maxX, maxY], key) => [
    Math.max(maxX, dots[key].x),
    Math.max(maxY, dots[key].y),
  ],
  [0, 0]
)
```
Next, for each two coordinate between 0 and the size of the paper, we print either a `#` if a dot exists with those coordinates, or a space character ` `.
```js
for (let y = 0; y <= maxY; y++) {
  let line = ''
  for (let x = 0; x <= maxX; x++) {
    const character = dots[getKey(x, y)] ? '#' : ' '
    line += character
  }
  console.log(line)
}
```

The final solution for part two is in my [GitHub repository](https://github.com/mancristiana/advent-of-code-2021/blob/main/src/day-13-transparent-origami/two.js).

## Conclusion
Day 13 was about folding a transparent Origami paper, updating coordinates of dots on the paper and revealing a final eight-character message.