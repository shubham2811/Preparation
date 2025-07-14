# Create mapAsync(arrar,asyncDouble) such that it resolved promise for every item and return response

### problem
```js
const asyncDouble = (x)=>{
    return new Promise((resolve)=>{
        setTimeout(()=>resolve(x*2),10)
    })
}

const doubled = await mapAsync([1,2],asyncDouble);
console.log(doubled)//[2,4]
```

### Solution

```js
// Method 1: Using Promise.all with map
const mapAsync = async (array, asyncFn) => {
    const promises = array.map(asyncFn);
    return await Promise.all(promises);
}

// Method 2: Using for...of loop (sequential execution)
const mapAsyncSequential = async (array, asyncFn) => {
    const result = [];
    for (const item of array) {
        const value = await asyncFn(item);
        result.push(value);
    }
    return result;
}

// Method 3: Using reduce (sequential execution)
const mapAsyncReduce = async (array, asyncFn) => {
    return await array.reduce(async (accPromise, item) => {
        const acc = await accPromise;
        const value = await asyncFn(item);
        acc.push(value);
        return acc;
    }, Promise.resolve([]));
}

// Test the solutions
const asyncDouble = (x) => {
    return new Promise((resolve) => {
        setTimeout(() => resolve(x * 2), 10);
    });
}

// Using the mapAsync function
(async () => {
    const doubled = await mapAsync([1, 2, 3, 4], asyncDouble);
    console.log(doubled); // [2, 4, 6, 8]
    
    const sequential = await mapAsyncSequential([1, 2, 3], asyncDouble);
    console.log(sequential); // [2, 4, 6]
})();
```

### Explanation

1. **Method 1 (Recommended)**: Uses `Promise.all()` with `array.map()` to execute all async operations concurrently. This is the most efficient approach when order doesn't matter and you want parallel execution.

2. **Method 2**: Uses a `for...of` loop to execute operations sequentially. Each async operation waits for the previous one to complete before starting.

3. **Method 3**: Uses `reduce()` to build the result array sequentially, similar to Method 2 but with a functional programming approach.

**Key Differences:**
- **Concurrent execution** (Method 1): All promises start at the same time, faster overall execution
- **Sequential execution** (Methods 2 & 3): Promises execute one after another, slower but useful when operations depend on previous results or you need to limit concurrent operations

# JavaScript Execution Order - Event Loop, Promises & Async/Await

## Overview
This example demonstrates the execution order of synchronous code, setTimeout, promises, and async/await functions in JavaScript. Understanding this order is crucial for mastering asynchronous JavaScript programming.

## Problem Statement
Predict and understand the execution order of the following code that combines:
- `setTimeout` with 0ms delay
- Promise resolution
- Promise constructor execution
- Async/await functions
- Synchronous console.log statements

## Code Example

```js
setTimeout(() => console.log('timeout'), 0)

const myPromise = Promise.resolve('resolved')

const myPro = new Promise((resolve, reject) => {
    console.log("Data in 2nd promise");
    resolve("Success")
})

async function myAsync() {
    console.log('async started')
    const result = await myPromise;
    const result1 = await myPro;
    console.log(result)
    console.log(result1)
    console.log('async ended')
}

console.log('before async')
myAsync()
console.log('after async')
```

## Expected Output

```
Data in 2nd promise
before async
async started
after async
resolved
Success
async ended
timeout
```

## Execution Order Explanation

### 1. **Synchronous Code Execution (Call Stack)**
```
1. setTimeout is registered → goes to Web APIs/Timer
2. myPromise = Promise.resolve('resolved') → creates resolved promise
3. Promise constructor executes → "Data in 2nd promise" (synchronous)
4. myAsync function is defined
5. "before async" is logged
6. myAsync() is called → "async started" is logged
7. First await hits → function pauses, control returns
8. "after async" is logged
```

### 2. **Microtask Queue Processing**
```
9. Microtasks (resolved promises) are processed
10. myPromise resolves → "resolved" is logged
11. myPro resolves → "Success" is logged  
12. "async ended" is logged
```

### 3. **Macrotask Queue Processing**
```
13. setTimeout callback executes → "timeout" is logged
```

## Key Concepts

### **Event Loop Priority Order**
1. **Call Stack** (synchronous code)
2. **Microtask Queue** (Promises, queueMicrotask)
3. **Macrotask Queue** (setTimeout, setInterval, I/O)

### **Promise Constructor Behavior**
- The executor function runs **immediately and synchronously**
- Only the `.then()` handlers are asynchronous

### **Async/Await Behavior**
- `async` functions return promises
- `await` pauses function execution until promise resolves
- Code after `await` is equivalent to `.then()` callback

### **setTimeout with 0ms**
- Doesn't execute immediately
- Goes to macrotask queue after minimum delay
- Always executes after all microtasks

## Detailed Step-by-Step Breakdown

| Step | Code Executed | Queue | Output |
|------|---------------|-------|---------|
| 1 | `setTimeout(...)` | Timer → Macrotask | - |
| 2 | `Promise.resolve(...)` | - | - |
| 3 | `new Promise(...)` | Synchronous | "Data in 2nd promise" |
| 4 | `console.log('before async')` | Call Stack | "before async" |
| 5 | `myAsync()` called | Call Stack | "async started" |
| 6 | First `await` hit | Function pauses | - |
| 7 | `console.log('after async')` | Call Stack | "after async" |
| 8 | Process microtasks | Microtask Queue | "resolved" |
| 9 | Continue microtasks | Microtask Queue | "Success" |
| 10 | Resume async function | Microtask Queue | "async ended" |
| 11 | Process macrotasks | Macrotask Queue | "timeout" |

## Important Notes

- **Promise constructors execute synchronously**
- **Microtasks always run before macrotasks**
- **Async functions pause at await, resume via microtasks**
- **setTimeout(0) is not truly 0ms - minimum ~4ms in browsers**
- **Event loop processes all microtasks before moving to macrotasks**

## Common Interview Questions

1. **Why does "timeout" log last despite 0ms delay?**
   - Because microtasks (promises) have higher priority than macrotasks (setTimeout)

2. **Why does "Data in 2nd promise" log first?**
   - Promise constructor executor runs synchronously, not asynchronously

3. **When does the async function resume?**
   - After current call stack is empty and microtasks are processed

## Practical Applications

- **Understanding promise resolution timing**
- **Debugging async code execution order**
- **Optimizing performance by understanding queue priorities**
- **Avoiding race conditions in complex async flows**

# Node.js Event Loop - process.nextTick() vs setImmediate()

## Overview
This example demonstrates the execution order of `process.nextTick()` and `setImmediate()` in Node.js, which have different priorities in the event loop compared to browser JavaScript.

## Problem Statement
Predict and understand the execution order of Node.js-specific asynchronous functions:
- `process.nextTick()` - highest priority microtask
- `setImmediate()` - macrotask that runs after I/O events
- Synchronous console.log statements

## Code Example

```js
console.log("Start");

process.nextTick(() => {
    console.log("nextTick callback");
});

setImmediate(() => {
    console.log("setImmediate callback");
});

console.log("End");
```

## Expected Output

```
Start
End
nextTick callback
setImmediate callback
```

## Execution Order Explanation

### 1. **Synchronous Code Execution (Call Stack)**
```
1. "Start" is logged immediately
2. process.nextTick() callback is queued → goes to NextTick Queue
3. setImmediate() callback is queued → goes to Check Phase Queue
4. "End" is logged immediately
```

### 2. **NextTick Queue Processing**
```
5. All process.nextTick() callbacks execute → "nextTick callback"
```

### 3. **Event Loop Phases**
```
6. setImmediate() callbacks execute in Check Phase → "setImmediate callback"
```