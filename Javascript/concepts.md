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