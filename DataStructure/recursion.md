# Subsequence Generation using Recursion

## Overview
This code demonstrates a recursive algorithm to generate all possible subsequences of an array using the "take or not take" approach. A subsequence is a sequence that can be derived from another sequence by deleting some or no elements without changing the order of the remaining elements.

## Function Description

### `subsequence(i, result, array)`

Generates all possible subsequences of an array using recursion and backtracking.

**Parameters:**
- `i` (number): Current index in the array being processed
- `result` (array): Current subsequence being built (passed by reference)
- `array` (array): Input array to generate subsequences from

**Algorithm:**
The function uses a binary recursive approach where at each element, there are two choices:
1. **Take** the current element (include it in the subsequence)
2. **Not take** the current element (exclude it from the subsequence)

## Code Flow

1. **Base Case**: When `i == n` (reached end of array), print the current subsequence
2. **Take Path**: 
   - Add current element to result using `push()`
   - Recursively call with next index (`i + 1`)
   - Remove element using `pop()` when returning (backtrack)
3. **Not Take Path**: 
   - Recursively call with next index without adding current element

## Example Execution

For input array `[3, 1, 2]`, the function generates all 8 possible subsequences:

```
[3, 1, 2]  // Take all elements
[3, 1]     // Take first two elements
[3, 2]     // Take first and third elements
[3]        // Take only first element
[1, 2]     // Take last two elements
[1]        // Take only second element
[2]        // Take only third element
[]         // Take no elements (empty subsequence)
```

## Time & Space Complexity
- **Time Complexity**: O(2^n) where n is the length of the array
  - Each element has 2 choices (include/exclude), resulting in 2^n total subsequences
- **Space Complexity**: O(n) for the recursion stack depth
  - Maximum depth of recursion is n (length of array)

## Key Concepts
- **Backtracking**: Uses `result.pop()` to undo choices when returning from recursion
- **Binary Recursion**: Each element has exactly two choices (include/exclude)
- **Complete Search**: Explores all possible combinations systematically
- **Reference Passing**: The `result` array is passed by reference and modified in-place

## Implementation

```js
const array = [3, 1, 2]
const result = []
function subsequence(i, result, array) {
  const n = array.length
  if (i == n) {
    console.log("result", result) //[3, 1, 2] [3, 1], [3, 2], [3], [1, 2], [1], [2], []
    return
  }
  //take condition
  result.push(array[i]) //
  subsequence(i + 1, result, array)
  result.pop() // removing because coming back you have to remove last index so that next step will be not take

  //not take condition
  subsequence(i + 1, result, array)
}
subsequence(0, result, array)
```
