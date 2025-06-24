# React Batch update
```js
export default function App() {
  const [count, setCount] = useState(0);
  const handleCount = () => {
    setCount(5);
    setCount(2);
  };
  console.log("count", count);
  return (
    <div className="App">
      <button type="button" onClick={handleCount}>
        Click me
      </button>
      {count}
    </div>
  );
}
```
## When you click the button, the value of count will be 2.

#### Explanation:
Both setCount(5) and setCount(2) are called in the same event handler. React batches these state updates, and since both use a direct value (not a function), the last call (setCount(2)) overrides the previous one. So after the click, count becomes 2.

## To make the value 7, use the functional form of setCount so each update is based on the previous state:
```js
setCount(c => c + 5);
setCount(c => c + 2);
```



# Different ways of calling a function

```js
onClick={handleClick}
onClick={handleClick()}
onClick={()=>handleClick}
onClick={()=>handleClick()}
```
| Syntax                        | When is function called?         | Typical Use Case                          |
|-------------------------------|----------------------------------|-------------------------------------------|
| `onClick={handleClick}`        | On click                         | Simple handlers                           |
| `onClick={handleClick()}`      | On render                        | Not recommended                           |
| `onClick={() => handleClick}`  | Never (returns function ref)     | Rarely used                               |
| `onClick={() => handleClick()}`| On click                         | When passing arguments or logic           |
