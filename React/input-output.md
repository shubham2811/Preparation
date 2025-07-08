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


# [React Timer Counter](https://github.com/shubham2811/TimerCounter)

```js
const [count, setCount] = useState(0);

  let timer;
  useEffect(() => {
    if (count) {
      timer = setTimeout(handleStart, 1000);
    }
  }, [count]);

  const handleStart = () => {
    setCount((count) => count + 1);
  };
  const handleStop = () => {
    clearTimeout(timer);
  };

  const handleReset = () => {
    setCount(0);
    clearTimeout(timer);
  };

  return (
    <div className="App">
      <h1>Count:{count}</h1>
      <button onClick={handleStart}>start</button>
      <button onClick={handleStop}>pause</button>
      <button onClick={handleReset}>reset</button>
    </div>
  );

```
# [React Countdown Timer](https://github.com/shubham2811/React-Countdown-Timer)
```js
import "./styles.css";
import { useEffect, useState } from "react";
const START_MINUTES = "01";
const START_SECONDS = "00";
const START_DURATION = 10;
export default function App() {
  const [minutes, setMinutes] = useState(START_MINUTES);
  const [seconds, setSeconds] = useState(START_SECONDS);
  const [duration, setDuration] = useState(START_DURATION);
  const [isRunning, setIsRunning] = useState(false);

  useEffect(() => {
    if (isRunning) {
      console.log("isRunning", isRunning);
      let timer = duration;
      const intervalId = setInterval(() => {
        console.log("timer", timer);
        if (--timer <= 0) {
          handleReset();
        } else {
          const min = parseInt(timer / 60, 10); //quotient
          const sec = parseInt(timer % 60, 10); //reminder
          setMinutes(min < 10 ? "0" + min : min);
          setSeconds(sec < 10 ? "0" + sec : sec);
        }
      }, 1000);

      return () => {
        clearInterval(intervalId);
      };
    }
  }, [isRunning]);

  const handleStart = () => {
    setDuration(parseInt(seconds, 10) + 60 * parseInt(minutes, 10));
    setIsRunning(true);
  };
  const handleStop = () => {
    const newDuration = parseInt(minutes, 10) * 60 + parseInt(seconds, 10);
    setDuration(newDuration);
    setIsRunning(!isRunning);
  };

  const handleReset = () => {
    setMinutes(START_MINUTES);
    setSeconds(START_SECONDS);
    setDuration(START_DURATION);
    setIsRunning(false);
  };

  return (
    <div className="App">
      <h1>
        {minutes}:{seconds}
      </h1>
      <button onClick={handleStart}>start</button>
      <button onClick={handleStop}>pause</button>
      <button onClick={handleReset}>reset</button>
    </div>
  );
}

```
# [Undo Redo Counter](https://www.notion.so/Map-Async-1fb057ee7160803aac61e344b75ee347)
```javascript
import { useState, useEffect } from "react";
export default function App() {
  const [count, setCount] = useState(0);
  const [history, setHistory] = useState([]);
  const [redoStack, setRedoStack] = useState([]);

  const handleChange = (delta) => {
    const before = count;
    const after = count + delta;

    const action = {
      label: delta > 0 ? `+${delta}` : `${delta}`,
      before,
      after,
    };

    setCount(after);
    setHistory((prev) => [action, ...prev].slice(0, 50)); // keep max 50
    setRedoStack([]); // clear redo on new action
  };

  const handleUndo = () => {
    if (history.length === 0) return;
    const [lastAction, ...rest] = history;
    setCount(lastAction.before);
    setHistory(rest);
    setRedoStack((r) => [lastAction, ...r]);
  };

  const handleRedo = () => {
    if (redoStack.length === 0) return;
    const [redoAction, ...rest] = redoStack;
    setCount(redoAction.after);
    setHistory((h) => [redoAction, ...h]);
    setRedoStack(rest);
  };
  return (
    <div style={{ padding: "1rem", fontFamily: "Arial", maxWidth: 400 }}>
      <h2>Count: {count}</h2>
      <div style={{ display: "flex", gap: "8px", marginBottom: "1rem" }}>
        {[1, 10, 100].map((n) => (
          <button key={`+${n}`} onClick={() => handleChange(n)}>
            +{n}
          </button>
        ))}
        {[1, 10, 100].map((n) => (
          <button key={`-${n}`} onClick={() => handleChange(-n)}>
            -{n}
          </button>
        ))}
      </div>
      <div style={{ display: "flex", gap: "8px", marginBottom: "1rem" }}>
        <button onClick={handleUndo} disabled={history.length === 0}>
          Undo
        </button>
        <button onClick={handleRedo} disabled={redoStack.length === 0}>
          Redo
        </button>
      </div>
      <h3>History</h3>
      <ul>
        {history.map((action, idx) => (
          <li key={idx}>
            {action.label} ({action.before} → {action.after})
          </li>
        ))}
      </ul>
    </div>
  );
}
```