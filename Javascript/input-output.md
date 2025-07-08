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


