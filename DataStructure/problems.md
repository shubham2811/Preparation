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
