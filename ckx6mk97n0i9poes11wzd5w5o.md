## Advent of Code 2021: Day 14


This blog post is fourteenth in the Advent of Code 2021 series and shows a JavaScript-based solution to the problem described in [Day 14](https://adventofcode.com/2021/day/14). This challenge was about optimizing an algorithm that generates exponentially larger and larger strings. I found it tricky and had to try multiple solutions until I reached the optimized solution described in this blog post.

All solutions are in this [GitHub repository](https://github.com/mancristiana/advent-of-code-2021).

## Optimally Solving Part One and Two
Given a template of polymers represented through letters and a set of rules, we must insert a new polymer between each pair of the initial template according to the rules. This insertion represents a single first step that produces the template for the next step. Part one requires running the insertion algorithm ten times, whereas part two is 40 times. Then, we must determine the least and most common polymer and subtract their occurrence number. 

My initial solution was manipulating arrays of characters to insert new elements according to the set of given rules. However, this solution only worked for part one and was not optimized for exponentially growing arrays (sized 2^40) which eventually exceeded the limits (2^32). After attempting a recursive algorithm on each initial pair, I got a similar result.

The strategy that solved this challenge was to store pairs in an occurrence hash map. Each insertion step would determine the occurrence of new pairs based on the previous occurrence map. Finally, we can derive the polymer occurrence from the pair occurrence. 

We can break down the solution as follows:
- read polymer template and insertion rules
- create a pair occurrence map for the template
- update pair occurrence map 10/40 times
- find polymer occurrence map
- subtract the least from the most common polymer

### Read polymer template and insertion rules
The template and pair insertion rules are separated by an empty line, thus we can the `split` function with the `\n\n` separator to get these two sections from the input.
```js
const [template, pairInsertionRulesData] = data.split('\n\n')
```
We process each line from the input and add the corresponding rule to the `pairInsertionRules` hash map. We use the pair as key and polymer to be inserted as value.
```js
const pairInsertionRules = {}
pairInsertionRulesData.split('\n').forEach((line) => {
  const [pair, result] = line.split(' -> ')
  pairInsertionRules[pair] = result
})
}
```
### Create a pair occurrence map for the template
The initial template from the example is `NNCB` and has the following pair occurrence hash map:
```js
{
  'NN': 1,
  'NC': 1,
  'CB': 1
}
```
We can create the pair occurrence map using the following algorithm:
```js
const getPairOccurenceMap = (template) => {
  const pairs = {}
  for (let i = 0; i < template.length - 1; ++i) {
    addToOccurenceMap(pairs, `${template.charAt(i)}${template.charAt(i + 1)}`)
  }

  return pairs
}
```
I extracted the logic for adding a key / value to an occurrence map into `addToOccurenceMap` function. This is useful in some of the next steps.
```js
const addToOccurenceMap = (occurenceMap, key, value = 1) => {
  if (occurenceMap[key]) {
    occurenceMap[key] += value
  } else {
    occurenceMap[key] = value
  }
}
```

### Update pair occurrence map 10/40 times
As mentioned earlier in this article, the strategy is to update the pair occurrence map rather than build a very large array or string exceeding the maximum limits. Thus, for part one we update 10 times and for part two 40 times.
```js
for (let i = 0; i < 10; ++i) {
  pairOccurenceMap = updatePairOccurenceMap(pairOccurenceMap,  pairInsertionRules)
}
```

To build the new pair occurrence map, we iterate through each pair using the `Object.keys` function. Using the pair as an index, we can determine the pair's occurrences from `pairOccurenceMap` and the new polymer to insert from the `pairInsertionRules`. 

If a rule exists for the pair, then the new pair occurrence map will contain two new pairs rather than the original pair. For example, if the rule `NC -> B` for the pair `NC` exists, the new pair occurrence map will have pairs `NB` and `CB` instead of `NC`. Otherwise, if a rule does not exist, we add the original pair to the new occurrence map. 

To build the new pairs we can use the string template literals ``${pair.charAt(0)}${polymer}`` for the left side pair and ``${polymer}${pair.charAt(1)}`` for the right side pair.

We add the value representing the occurrences of the original pair to each new pair.

```js
const updatePairOccurenceMap = (pairOccurenceMap, pairInsertionRules) => {
  const newPairOccurenceMap = {}

  Object.keys(pairOccurenceMap).forEach((pair) => {
    const occurences = pairOccurenceMap[pair]
    const polymer = pairInsertionRules[pair]
    if (polymer) {
      addToOccurenceMap(newPairOccurenceMap, `${pair.charAt(0)}${polymer}`, occurences)
      addToOccurenceMap(newPairOccurenceMap, `${polymer}${pair.charAt(1)}`, occurences)
    } else {
      addToOccurenceMap(newPairOccurenceMap, pair, occurences)
    }
  })
  return newPairOccurenceMap
}

```
### Find polymer occurrence map
We can determine the polymer occurrence map based on the pair occurrence map and the first and last letter of the initial template. Each polymer, except the first and last one in the initial template, is counted twice in pairs. Thus, we must add the first and last polymers one more time, then divide each polymer occurrence by 2 to determine the correct polymer occurrence.

```js
onst getPolymerOccurenceMap = (pairOccurenceMap, template) => {
  const occurenceMap = {}
  Object.keys(pairOccurenceMap).forEach(pair => {
    addToOccurenceMap(occurenceMap, pair.charAt(0), pairOccurenceMap[pair])
    addToOccurenceMap(occurenceMap, pair.charAt(1), pairOccurenceMap[pair])
  })

  occurenceMap[template.charAt(0)]++
  occurenceMap[template.charAt(template.length - 1)]++

  Object.keys(occurenceMap).forEach(key => {
    occurenceMap[key] = Math.floor(occurenceMap[key] / 2)
  })
  return occurenceMap
}

```
### Subtract the least from the most common polymer
We can calculate the least and most common polymers by iterating through the keys of the polymer occurence map.
```js
const getLeastAndMostCommon = (occurenceMap) => {
  let leastCommon
  let mostCommon

  Object.keys(occurenceMap).forEach((polymer) => {
    if (!leastCommon || occurenceMap[leastCommon] > occurenceMap[polymer]) {
      leastCommon = polymer
    }
    if (!mostCommon || occurenceMap[mostCommon] < occurenceMap[polymer]) {
      mostCommon = polymer
    }
  })

  return { leastCommon, mostCommon }
}
```

Then we can subtract the occurence of the least most common polymers:
```js
polymerOccurenceMap[mostCommon] - polymerOccurenceMap[leastCommon]
```

The final solution for Day 14 is in my [GitHub repository](https://github.com/mancristiana/advent-of-code-2021/tree/main/src/day-14-extended-polymerization).


## Conclusion
Day 14 was tricky and required optimally determining the least and most common letter occurrences in an exponentially growing string. An occurrence hash map storing pairs helped define the next set of occurrences, prevented storing a massive string and made it possible to solve the challenge.