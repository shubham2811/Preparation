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

# Find Pivot Index

## Overview
The pivot index algorithm finds the index where the sum of elements to the left equals the sum of elements to the right. This is a classic array problem that demonstrates prefix sum techniques and efficient array traversal.

## Problem Statement
Given an array of integers `nums`, find the pivot index where:
- The sum of numbers to the left of the index equals the sum of numbers to the right of the index
- If no such index exists, return -1
- If there are multiple pivot indices, return the leftmost one

## Function Description

### `pivotIndex(nums)`

Finds the pivot index in an array where left sum equals right sum.

**Parameters:**
- `nums` (array): Array of integers to find pivot index in

**Returns:**
- `number`: The pivot index (0-based), or -1 if no pivot exists

## Algorithm Explanation

The algorithm uses a **prefix sum approach** with a single pass through the array:

1. **Calculate Total Sum**: First, compute the total sum of all elements
2. **Track Left Sum**: Maintain a running sum of elements to the left of current index
3. **Calculate Right Sum**: For each position, calculate right sum as `total - leftSum - currentElement`
4. **Check Balance**: If left sum equals right sum, return current index
5. **Update Left Sum**: Add current element to left sum for next iteration

## Step-by-Step Process

For each index `i`:
1. `rightSum = total - leftSum - nums[i]`
2. If `leftSum == rightSum`, return `i`
3. `leftSum += nums[i]` (prepare for next iteration)

## Example Execution

For input array `[1, 7, 3, 6, 5, 6]`:

```
Index 0: leftSum=0, rightSum=27, nums[0]=1 → 0 ≠ 27
Index 1: leftSum=1, rightSum=19, nums[1]=7 → 1 ≠ 19  
Index 2: leftSum=8, rightSum=12, nums[2]=3 → 8 ≠ 12
Index 3: leftSum=11, rightSum=11, nums[3]=6 → 11 = 11 ✓
```

**Result**: Index 3 is the pivot (left sum = right sum = 11)

## Time & Space Complexity

- **Time Complexity**: O(n) where n is the length of the array
  - One pass to calculate total sum: O(n)
  - One pass to find pivot: O(n)
  - Total: O(n)
- **Space Complexity**: O(1) 
  - Only uses constant extra space for variables

## Key Concepts

- **Prefix Sum**: Running sum technique to avoid recalculating sums
- **Mathematical Relationship**: `rightSum = total - leftSum - currentElement`
- **Early Termination**: Returns immediately when pivot is found
- **Edge Cases**: Handles empty arrays and single-element arrays

## Edge Cases

1. **Empty Array**: Returns -1
2. **Single Element**: Index 0 is pivot (left=0, right=0)
3. **No Pivot**: Returns -1 when no balance point exists
4. **Multiple Pivots**: Returns the leftmost pivot index

## Implementation

```js
/**
 * @param {number[]} nums
 * @return {number}
 */
var pivotIndex = function (nums) {
    let leftSum = 0
    let rightSum = 0
    const total = nums.reduce((acc, item) => acc + item)
    for (let i = 0; i < nums.length; i++) {
        rightSum = total - leftSum - nums[i]
        if (leftSum === rightSum) {
            return i
        }
        leftSum += nums[i]

    }
    return -1
};
```

# Game Outcome - Score Difference

## Overview
This algorithm simulates a two-player game where players take turns picking numbers an array. The game has a special rule: when a player picks an even number, the picking direction reverses. The function calculates the score difference between Player 1 and Player 2.

## Function Description

### `gameScoreDifference(numSeq)`

Simulates the game and returns the score difference between the two players.

**Parameters:**
- `numSeq` (array): Array of integers representing the game sequence

**Returns:**
- `number`: Score difference (Player 1 score - Player 2 score)

## Algorithm Explanation

The algorithm uses a **two-pointer approach** with state tracking:

1. **Initialize Variables**: 
   - Player scores (`p1`, `p2`)
   - Turn tracker (`turn`: 0 for P1, 1 for P2)
   - Array boundaries (`left`, `right`)
   - Direction flag (`reversed`)

2. **Game Loop**: Continue while there are numbers to pick
3. **Pick Number**: Based on current direction (left or right end)
4. **Update Score**: Add picked number to current player's score
5. **Check Even Rule**: If picked number is even, reverse direction
6. **Switch Turn**: Alternate between players

## Step-by-Step Process

For each turn:
1. Pick number from current end (left if not reversed, right if reversed)
2. Add to current player's score
3. If number is even, toggle `reversed` flag
4. Switch to next player
5. Move pointer (left++ or right--)

## Example Execution

For input array `[3, 6, 2, 3, 5]`:

```
Turn 1 (P1): Pick 3 from left → P1=3, P2=0, direction stays left (3 is odd)
Turn 2 (P2): Pick 6 from left → P1=3, P2=6, direction reverses (6 is even)
Turn 3 (P1): Pick 5 from right → P1=8, P2=6, direction stays right (5 is odd)
Turn 4 (P2): Pick 3 from right → P1=8, P2=9, direction stays right (3 is odd)
Turn 5 (P1): Pick 2 from right → P1=10, P2=9, direction reverses (2 is even)
```

**Result**: Score difference = 10 - 9 = 1

## Time & Space Complexity

- **Time Complexity**: O(n) where n is the length of the array
  - Single pass through all elements
- **Space Complexity**: O(1)
  - Only uses constant extra space for variables

## Key Concepts

- **Two Pointers**: Uses left and right pointers to track array boundaries
- **State Machine**: Direction reversal based on even numbers
- **Game Theory**: Turn-based gameplay simulation
- **Conditional Logic**: Different behavior based on number parity

## Edge Cases

1. **Single Element**: One player gets the element, other gets 0
2. **All Even Numbers**: Direction keeps reversing
3. **All Odd Numbers**: Direction never changes
4. **Empty Array**: Both players score 0

## Implementation

```js
function gameScoreDifference(numSeq) {
  let p1 = 0, p2 = 0;
  let turn = 0; // 0 for P1, 1 for P2

  let left = 0, right = numSeq.length - 1;
  let reversed = false;

  while (left <= right) {
    let current;
    if (!reversed) {
      current = numSeq[left++];
    } else {
      current = numSeq[right--];
    }

    if (turn === 0) {
      p1 += current;
    } else {
      p2 += current;
    }

    if (current % 2 === 0) {
      reversed = !reversed;
    }

    turn = 1 - turn;
  }

  return p1 - p2;
}

// Example usage
console.log(gameScoreDifference([3, 6, 2, 3, 5])); // Output: 1
```

## Game Strategy Notes

- **Even Numbers**: Create strategic opportunities by changing direction
- **Positioning**: Players must consider both immediate gain and direction change
- **Endgame**: Final few moves become critical when direction matters most
- **Optimal Play**: This implementation simulates actual gameplay, not optimal strategy
