# [Throttling and Debouncing](https://www.greatfrontend.com/questions/quiz/explain-the-concept-of-debouncing-and-throttling)

## Throttling

**Throttling** is a technique used to ensure that a function is called at most once in a specified time interval. This is useful in scenarios where you want to limit the number of times a function is called, such as when handling events like window resizing or scrolling.

**Example use case:**  
Imagine you have a function that updates the position of elements on the screen based on the window size. Without throttling, this function could be called many times per second as the user resizes the window, leading to performance issues. Throttling ensures that the function is only called at most once in a specified time interval.

**Example:**
```js
function throttle(func, limit) {
  let inThrottle;
  return function (...args) {
    if (!inThrottle) {
      func.apply(this, args);
      inThrottle = true;
      setTimeout(() => (inThrottle = false), limit);
    }
  };
}

// Usage
const handleResize = throttle(() => {
  // Update element positions
  console.log('Window resized');
}, 100);

window.addEventListener('resize', handleResize);

```



## Debouncing

**Debouncing** is a technique used to ensure that a function is only executed after a certain amount of time has passed since it was last invoked. This is particularly useful in scenarios where you want to limit the number of times a function is called, such as when handling user input events like keypresses or mouse movements.

**Example use case:**  
Imagine you have a search input field and you want to make an API call to fetch search results. Without debouncing, an API call would be made every time the user types a character, which could lead to a large number of unnecessary calls. Debouncing ensures that the API call is only made after the user has stopped typing for a specified amount of time.

**Example:**
```js
function debounce(func, delay) {
  let timeoutId;
  return function (...args) {
    clearTimeout(timeoutId);
    timeoutId = setTimeout(() => func.apply(this, args), delay);
  };
}

// Usage
const handleSearch = debounce((query) => {
  // Make API call
  console.log('API call with query:', query);
}, 300);

document.getElementById('searchInput').addEventListener('input', (event) => {
  handleSearch(event.target.value);
});
```

## Bind vs Call vs Apply
```js
let nameObj = {
  name: "Tony",
};

let printName = {
  name: "steve",
  sayHi: function (age) {
    console.log(this.name + " age is " + age);
  },
};
```
#### Bind Usage
The Bind() Method creates a new function and when that new function is called it set this keyword to the first argument which is passed to the bind method, and if any other sequences of arguments preceding the first argument are passed to the bind method then they are passed as an argument to the new function when the new function is called.
```js
const hiFun = printName.sayHi.bind(nameObj);
hiFun();
```
#### Bind Polyfill
```js
Object.prototype.customBind = function (obj, ...args) {
  obj.myMethod = this;
  return function () {
    obj.myMethod(...args);
  };
};

const hiFunc = printName.sayHi.customBind(nameObj, 42);
hiFunc();
```

#### Call Usage
The Call() Method calls the function directly and sets this to the first argument passed to the call method and if any other sequences of arguments preceding the first argument are passed to the call method then they are passed as an argument to the function.
```js
printName.sayHi.call(nameObj, 42);
```

#### Call Polyfill
```js
Object.prototype.customCall = function (obj, ...args) {
  obj.myMethod = this;
  obj.myMethod(...args);
};
printName.sayHi.customCall(nameObj,42);
```

#### Apply Usage 
The Apply() Method calls the function directly and sets this to the first argument passed to the apply method and if any other arguments provided as an array are passed to the call method then they are passed as an argument to the function.
```js
printName.sayHi.apply(nameObj, [42]);
```
#### Apply Polyfill
```js
Object.prototype.customApply = function (obj, args) {
  obj.myMethod = this;
  obj.myMethod(...args);
};
printName.sayHi.customApply(nameObj,[42]);
```

## Array Methods Polyfill

```js
const sample = [1, 2, 3, 4, 5]

//Map Polyfill
Array.prototype.customMap = function (callback) {
  const result = []
  for (let i = 0; i < this.length; i++) {
    result.push(callback(this[i], i, this))
  }
  return result
}
//Filter Polyfill
Array.prototype.customFilter = function (callback) {
  const result = []
  for (let i = 0; i < this.length; i++) {
    if (callback(this[i], i, this)) {
      result.push(this[i])
    }
  }
  return result
}
//Reduce Polyfill
const value = sample.customReduce((acc, currentValue) => acc + currentValue, 4)
Array.prototype.customReduce = function (callback, initialValue) {
  let accumulator = initialValue === undefined ? undefined : initialValue

  for (let i = 0; i < this.length; i++) {
    if (accumulator !== undefined) {
      accumulator = callback(accumulator, this[i], i, this)
    } else {
      accumulator = this[i]//set initial value (1)
    }
  }
  return accumulator
} 
//Find Polyfill
Array.prototype.customFind = function (callback) {
  for (let i = 0; i < this.length; i++) {
    if (callback(this[i], i, this)) {
     return this[i]
    }
  }
 return undefined;
}
// ForEach Polyfill
Array.prototype.myForEach = function(callback){
    for(let i=0; i<this.length; i++){
      callback(this[i],i,this)
    }
}
// every Polyfill
Array.prototype.myEvery = function(callback){
   for(let i=0; i<this.length; i++){
     if(!callback(this[i],i,this)){
       return false;
     }
   }
   return true;
}
// some Polyfill
Array.prototype.mySome = function(callback){
   for(let i=0; i<this.length; i++){
     if(callback(this[i],i,this)){
       return true;
     }
   }
   return false;
}

```

## Promises

#### [Promise Polyfill simple](https://medium.com/swlh/implement-a-simple-promise-in-javascript-20c9705f197a)
```js
const customPromise = function (handler) {
  let status = "pending";
  let value = null;
  let onFulfilledCallbacks = [];
  let onRejectedCallbacks = [];


  const resolve = (val) => {
    if (status === "pending") {
      status = "fulfilled";
      value = val;
      onFulfilledCallbacks.forEach((fn) => fn(value));
    }
  };

  const reject = (val) => {
    if (status === "pending") {
      status = "rejected";
      value = val;
      onRejectedCallbacks.forEach((fn) => fn(value));
    }
  };

  this.then = (onFulfilled, onRejected) => {
    onFulfilledCallbacks.push(onFulfilled);
    onRejectedCallbacks.push(onRejected);
  
    if (status === "fulfilled") {
      onFulfilled(value);
    }
    if (status === "rejected") {
      onRejected(value);
    }
  };

  this.catch = function(onRejected) {
    return this.then(null, onRejected);
  };

  try {
    handler(resolve, reject);
  } catch (err) {
    reject(err);
  }
};

//calling
const p3 = new customPromise((resolve, reject) => {
    setTimeout(() => reject('rejected!'), 1000);
});
p3.then((res) => {
    console.log(res);
}, (err) => {
    console.log(err);
});

```

#### Promise Chaining
Modify the then method only for promise chaining
```js
  this.then = (onFulfilled, onRejected) => {
    return new customPromise((resolve, reject) => {
      if (status === "pending") {
        onFulfilledCallbacks.push(() => {
          try {
            const fulfilledFromLastPromise = onFulfilled(value);
            if (fulfilledFromLastPromise instanceof Promise) {
              fulfilledFromLastPromise.then(resolve, reject);
            } else {
              resolve(fulfilledFromLastPromise);
            }
          } catch (err) {
            reject(err);
          }
        });

        onRejectedCallbacks.push(() => {
          try {
            const rejectedFromLastPromise = onRejected(value);
            if (rejectedFromLastPromise instanceof Promise) {
              rejectedFromLastPromise.then(resolve, reject);
            } else {
              reject(rejectedFromLastPromise);
            }
          } catch (err) {
            reject(err);
          }
        });
      }

      if (status === "fulfilled") {
        try {
          const fulfilledFromLastPromise = onFulfilled(value);
          if (
            fulfilledFromLastPromise &&
            typeof fulfilledFromLastPromise.then === "function"
          ) {
            fulfilledFromLastPromise.then(resolve, reject);
          } else {
            resolve(fulfilledFromLastPromise);
          }
        } catch (err) {
          reject(err);
        }
      }

      if (status === "rejected") {
        try {
          const rejectedFromLastPromise = onRejected(value);
          if (
            rejectedFromLastPromise &&
            typeof rejectedFromLastPromise.then === "function"
          ) {
            rejectedFromLastPromise.then(resolve, reject);
          } else {
            reject(rejectedFromLastPromise);
          }
        } catch (err) {
          reject(err);
        }
      }
    });
  };
  // testing code
const p1 = new customPromise((resolve, reject) => {
  setTimeout(() => resolve("resolve!"), 1000);
});
p1.then((res) => {
  console.log(res);
  return new Promise((resolve) => {
    setTimeout(() => resolve("resolved second one"), 1000);
  });
}).then((res) => {
  console.log(res);
});
```
#### Promise all polyfill
```js
customPromise.resolve = function (promise) {
  return new customPromise((resolve, reject) => resolve(promise));
};
customPromise.reject = function (promise) {
  return new customPromise((resolve, reject) => reject(promise));
};
// All method which is a function that takes array of promises
// Promise.all polyfill
customPromise.all = function (promises) {
  return new customPromise((resolve, reject) => {
    if (!Array.isArray(promises)) {
      reject(new TypeError("customPromise.all expects an array"));
      return;
    }

    if (promises.length === 0) {
      resolve([]);
      return;
    }

    const results = [];
    let completedPromises = 0;
    const totalPromises = promises.length;

    promises.forEach((promise, index) => {
      // Convert non-promise values to resolved promises
      const promiseToResolve =
        promise && typeof promise.then === "function"
          ? promise
          : new customPromise((resolve) => resolve(promise));

      promiseToResolve.then(
        (value) => {
          results[index] = value;
          completedPromises++;

          if (completedPromises === totalPromises) {
            resolve(results);
          }
        },
        (error) => {
          reject(error);
        },
      );
    });
  });
};
```



#### Compare two Object deep and nested

```js
function deepEqual(obj1, obj2) {
  if (obj1 === obj2) {
    return true;
  }

  if (
    obj1 === null ||
    typeof obj1 !== "object" ||
    obj2 === null ||
    typeof obj2 !== "object"
  )
    return false;

  const keys1 = Object.keys(obj1);
  const keys2 = Object.keys(obj2);
  if (keys1.length !== keys2.length) return false;

  for (let key of keys1) {
    if (!obj2.hasOwnProperty(key) || !deepEqual(obj1[key], obj2[key])) {
      return false;
    }
  }
  return true;
}
const obj1 = {
  fName: "shubham",
  lName: "sharma",
};

const obj2 = {
  fName: "shubham",
  lName: "sharma",
};
console.log(deepEqual(obj1, obj2));

```

---

# Understanding `void(0)` in JavaScript

## What is `void(0)`?

`void(0)` is a JavaScript expression that **always returns `undefined`**, regardless of the operand. The `void` operator evaluates any expression and returns `undefined`.

## Basic Syntax and Examples

```js
// All of these return undefined
console.log(void 0);          // undefined
console.log(void(0));         // undefined
console.log(void 1);          // undefined
console.log(void "hello");    // undefined
console.log(void true);       // undefined
console.log(void {});         // undefined

// The expression is evaluated but the result is discarded
let x = 5;
console.log(void x++);        // undefined
console.log(x);               // 6 (x was incremented)
```

## Why Use `void(0)` Instead of `undefined`?

### 1. **Historical Safety (Pre-ES5)**
```js
// In older JavaScript (before ES5), undefined was mutable
undefined = "not undefined anymore!"; // This was possible!
console.log(undefined);                // "not undefined anymore!"

// But void(0) always returns true undefined
console.log(void(0));                  // undefined (always reliable)
```

### 2. **Minification Benefits**
```js
// Original code
if (someVariable === undefined) {
  // do something
}

// Minified version is shorter
if (someVariable === void 0) {
  // do something
}
// "void 0" is shorter than "undefined" (7 chars vs 9 chars)
```

## Common Use Cases

### 1. **Hyperlinks with JavaScript**
```html
<!-- Prevents page navigation while executing JavaScript -->
<a href="javascript:void(0)" onclick="doSomething()">Click me</a>

<!-- Alternative approaches -->
<a href="javascript:;" onclick="doSomething()">Click me</a>
<a href="#" onclick="doSomething(); return false;">Click me</a>
```

### 2. **Safe Undefined Comparison**
```js
// Safe way to check for undefined
function isUndefined(value) {
  return value === void 0;
}

// Usage
console.log(isUndefined());           // true
console.log(isUndefined(null));       // false
console.log(isUndefined(undefined));  // true
```

### 3. **Function Default Parameters (Legacy)**
```js
// Before ES6 default parameters
function greet(name) {
  name = name !== void 0 ? name : "Anonymous";
  console.log("Hello, " + name);
}

// Modern ES6 way
function greet(name = "Anonymous") {
  console.log("Hello, " + name);
}
```

### 4. **Immediately Invoked Function Expressions (IIFE)**
```js
// Using void to make function expression
void function() {
  console.log("IIFE executed!");
}();

// Traditional IIFE
(function() {
  console.log("IIFE executed!");
})();
```

## Advanced Examples

### 1. **Preventing Return Values**
```js
// Useful in arrow functions where you don't want to return anything
const numbers = [1, 2, 3, 4, 5];

// Without void - returns mapped array
const result1 = numbers.map(n => console.log(n));
console.log(result1); // [undefined, undefined, undefined, undefined, undefined]

// With void - explicitly returns undefined
const result2 = numbers.map(n => void console.log(n));
console.log(result2); // [undefined, undefined, undefined, undefined, undefined]
```

### 2. **Bookmarklet Safety**
```js
// Bookmarklet that doesn't interfere with page
javascript:(function(){
  alert('Hello from bookmarklet!');
})();void(0);
```

### 3. **Checking for Undefined Properties**
```js
const obj = {
  name: "John",
  age: undefined,
  city: "New York"
};

// Check if property exists vs is undefined
console.log(obj.age === void 0);        // true
console.log(obj.missing === void 0);    // true
console.log(obj.hasOwnProperty('age')); // true
console.log(obj.hasOwnProperty('missing')); // false

// Better approach for checking undefined
function isPropertyUndefined(obj, prop) {
  return obj.hasOwnProperty(prop) && obj[prop] === void 0;
}
```

## Modern Alternatives

### 1. **Using `undefined` Directly (Safe in Modern JS)**
```js
// Modern JavaScript (ES5+) - undefined is read-only
if (value === undefined) {
  // This is safe and preferred
}

// Still works but unnecessary
if (value === void 0) {
  // This works but is less readable
}
```

### 2. **Using `typeof` for Safety**
```js
// Ultra-safe way to check for undefined
if (typeof value === 'undefined') {
  // This works even if 'value' is not declared
}
```

### 3. **Destructuring with Defaults**
```js
// Modern way to handle undefined values
const { name = "Anonymous", age = 0 } = userData || {};

// Instead of
const name = userData && userData.name !== void 0 ? userData.name : "Anonymous";
```

## Practical Examples

### 1. **Safe Object Property Access**
```js
// Utility function using void(0)
function safeGet(obj, path, defaultValue = void 0) {
  return path.split('.').reduce((current, key) => {
    return current && current[key] !== void 0 ? current[key] : defaultValue;
  }, obj);
}

// Usage
const user = {
  profile: {
    name: "John",
    settings: {
      theme: "dark"
    }
  }
};

console.log(safeGet(user, 'profile.name'));           // "John"
console.log(safeGet(user, 'profile.missing'));       // undefined
console.log(safeGet(user, 'profile.missing', 'N/A')); // "N/A"
```

### 2. **Event Handler Prevention**
```js
// Prevent default behavior in event handlers
function handleClick(event) {
  // Do something
  console.log('Button clicked');
  
  // Prevent default and return undefined
  event.preventDefault();
  return void 0;
}

// In HTML
// <button onclick="return handleClick(event)">Click me</button>
```

### 3. **Conditional Execution**
```js
// Execute code conditionally and return undefined
const result = condition ? doSomething() : void 0;

// More explicit than
const result = condition ? doSomething() : undefined;
```

## Browser Console Examples

```js
// Try these in browser console
console.log(void(0));                    // undefined
console.log(void(0) === undefined);     // true
console.log(typeof void(0));            // "undefined"

// Comparison with other falsy values
console.log(void(0) === null);          // false
console.log(void(0) === false);         // false
console.log(void(0) === 0);             // false
console.log(void(0) === "");            // false

// With expressions
console.log(void(1 + 2));               // undefined (but 1 + 2 was calculated)
console.log(void(console.log("hi")));   // undefined (but "hi" was logged)
```

## Summary

| Aspect | Details |
|--------|---------|
| **Purpose** | Always returns `undefined` |
| **When to use** | Legacy code, minification, href attributes |
| **Modern relevance** | Limited - `undefined` is safer now |
| **Best practice** | Use `undefined` directly in modern code |
| **Still useful for** | Bookmarklets, IIFE, preventing return values |

**Key Takeaway**: While `void(0)` was crucial for safety in older JavaScript, modern code should prefer `undefined` directly for better readability. However, understanding `void(0)` is important for reading legacy code and specific use cases like bookmarklets.

---

# Understanding WeakMap in JavaScript

## What is WeakMap?

A `WeakMap` is a collection of key-value pairs where **keys must be objects** and the references to keys are **weak**. This means that if there are no other references to a key object, it can be garbage collected even if it's still in the WeakMap.

## Key Characteristics

```js
// Basic WeakMap creation
const weakMap = new WeakMap();

// Keys must be objects (not primitives)
const obj1 = { name: "John" };
const obj2 = { name: "Jane" };
const func = function() {};

weakMap.set(obj1, "User data for John");
weakMap.set(obj2, "User data for Jane");
weakMap.set(func, "Function metadata");

// This would throw an error - keys must be objects
// weakMap.set("string", "value"); // TypeError
// weakMap.set(123, "value");      // TypeError
```

## WeakMap vs Map Comparison

| Feature | Map | WeakMap |
|---------|-----|---------|
| **Key Types** | Any type | Objects only |
| **Garbage Collection** | Prevents GC | Allows GC |
| **Enumerable** | Yes (has size, keys(), values()) | No |
| **Size Property** | Yes | No |
| **Iteration** | Yes | No |
| **Clear Method** | Yes | No |

```js
// Map example
const map = new Map();
map.set("key1", "value1");
map.set(obj1, "value2");
console.log(map.size); // 2
for (let [key, value] of map) {
  console.log(key, value);
}

// WeakMap example
const weakMap = new WeakMap();
weakMap.set(obj1, "value1");
// console.log(weakMap.size); // undefined - no size property
// for (let entry of weakMap) {} // TypeError - not iterable
```

## Primary Use Cases

### 1. **Private Data Storage**

```js
// Using WeakMap to store private data
const privateData = new WeakMap();

class User {
  constructor(name, email) {
    // Store private data
    privateData.set(this, {
      email: email,
      createdAt: new Date(),
      sessionToken: Math.random().toString(36)
    });
    
    this.name = name; // Public property
  }
  
  getEmail() {
    return privateData.get(this).email;
  }
  
  getSessionToken() {
    return privateData.get(this).sessionToken;
  }
  
  updateEmail(newEmail) {
    const data = privateData.get(this);
    data.email = newEmail;
  }
}

const user = new User("John", "john@example.com");
console.log(user.name);           // "John" (accessible)
console.log(user.email);          // undefined (private)
console.log(user.getEmail());     // "john@example.com" (via method)

// When user is garbage collected, private data is also cleaned up
```
### 2. **Caching and Memoization**

```js
// Cache expensive computations per object
const computationCache = new WeakMap();

function expensiveComputation(obj) {
  // Check if result is already cached
  if (computationCache.has(obj)) {
    console.log('Cache hit!');
    return computationCache.get(obj);
  }
  
  console.log('Computing...');
  // Simulate expensive operation
  const result = JSON.stringify(obj).length * Math.random();
  
  // Cache the result
  computationCache.set(obj, result);
  return result;
}

const data1 = { name: "John", age: 30 };
const data2 = { name: "Jane", age: 25 };

console.log(expensiveComputation(data1)); // Computing... then result
console.log(expensiveComputation(data1)); // Cache hit! then same result
console.log(expensiveComputation(data2)); // Computing... then result
```
## WeakMap Methods

```js
const wm = new WeakMap();
const obj = { id: 1 };

// Available methods
wm.set(obj, "value");           // Add key-value pair
console.log(wm.get(obj));       // "value" - Get value
console.log(wm.has(obj));       // true - Check if key exists
wm.delete(obj);                 // Remove key-value pair
console.log(wm.has(obj));       // false

// Methods NOT available (unlike Map)
// wm.size        // undefined
// wm.clear()     // Method doesn't exist
// wm.keys()      // Method doesn't exist
// wm.values()    // Method doesn't exist
// wm.entries()   // Method doesn't exist
```

## Best Practices

### 1. **Use WeakMap When**
- You need to associate data with objects without modifying them
- You want automatic cleanup when objects are garbage collected
- You're implementing private data for classes
- You need object-based caching or memoization

### 2. **Don't Use WeakMap When**
- You need to iterate over the collection
- You need to know the size of the collection
- Keys might be primitive values
- You need to clear all entries at once

## Summary

| Use Case | Description | Benefit |
|----------|-------------|---------|
| **Private Data** | Store private properties for objects | No property pollution |
| **Metadata** | Add data without modifying objects | Clean separation |
| **Caching** | Cache results per object | Automatic cleanup |
| **State Management** | Manage object state externally | Memory efficient |
| **Event Tracking** | Track events without memory leaks | Garbage collection |
| **Mixins** | Add functionality to objects | Non-intrusive |

**Key Benefits**: Automatic memory management, no object modification, and clean separation of concerns make WeakMap ideal for scenarios where you need to associate data with objects temporarily or privately.
