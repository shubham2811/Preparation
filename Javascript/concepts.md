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