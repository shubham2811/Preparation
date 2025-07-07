# React Diff Algorithm and Its Phases

## What is the React Diff Algorithm?

The React diff algorithm is a comparison mechanism that React uses to efficiently update the DOM by comparing the current virtual DOM tree with the previous virtual DOM tree. Instead of re-rendering the entire DOM, React identifies what has changed and updates only those specific parts.

## Key Principles

1. **Two trees are never compared if they are of different component types**
2. **Stable keys help React identify which items have changed, been added, or removed**
3. **React assumes that if two elements have the same type and key, the content is likely the same**

## Phases of the React Diff Algorithm

### 1. **Tree Comparison Phase**

#### Element Type Comparison
```jsx
// Different types - complete re-render
<div>
  <Counter />
</div>

// vs

<span>
  <Counter />
</span>
```

When React compares elements of different types (div vs span), it:
- Destroys the old tree completely
- Builds a new tree from scratch
- Triggers `componentWillUnmount()` for old components
- Triggers `componentDidMount()` for new components

#### Same Type Element Comparison
```jsx
// Before
<div className="before" title="stuff" />

// After  
<div className="after" title="stuff" />
```

When comparing same type elements, React:
- Keeps the same underlying DOM node
- Updates only changed attributes (className in this example)
- Recurses on children

### 2. **Component Comparison Phase**

#### Same Type Components
```jsx
// Component state/props change
<Button color="red" />
// to
<Button color="blue" />
```

For same component types, React:
- Updates the props
- Calls `componentDidUpdate()` or effect hooks
- Recurses on the render output

#### Different Type Components
```jsx
// Complete replacement
<Button />
// to  
<Link />
```

Different component types result in:
- Unmounting the old component
- Mounting the new component
- Complete subtree replacement

### 3. **List Reconciliation Phase**

#### Without Keys (Inefficient)
```jsx
// Before
<ul>
  <li>Duke</li>
  <li>Villanova</li>
</ul>

// After - adding at beginning
<ul>
  <li>Connecticut</li>
  <li>Duke</li>
  <li>Villanova</li>
</ul>
```

Without keys, React will:
- Mutate every child (inefficient)
- Recreate elements unnecessarily

#### With Keys (Efficient)
```jsx
// Before
<ul>
  <li key="2015">Duke</li>
  <li key="2016">Villanova</li>
</ul>

// After
<ul>
  <li key="2014">Connecticut</li>
  <li key="2015">Duke</li>
  <li key="2016">Villanova</li>
</ul>
```

With keys, React can:
- Identify elements that moved
- Reuse existing DOM nodes
- Only insert the new element

## Diff Algorithm Strategies

### 1. **Level-by-Level Comparison**
```jsx
// React compares trees level by level
     A              A
   /   \          /   \
  B     C   =>   B     D
 /             /   \
D             C     E
```

React won't detect that `D` moved - it will destroy and recreate it.

### 2. **Sibling Comparison**
```jsx
// React processes siblings in order
[A, B, C] => [A, C, B]
```

React uses keys to optimize sibling reordering.

## Performance Optimizations

### 1. **React.memo()**
```jsx
const MyComponent = React.memo(function MyComponent({ name }) {
  return <div>Hello {name}</div>;
});
```

Prevents re-rendering if props haven't changed.

### 2. **useMemo() and useCallback()**
```jsx
const ExpensiveComponent = () => {
  const expensiveValue = useMemo(() => {
    return computeExpensiveValue(a, b);
  }, [a, b]);

  const memoizedCallback = useCallback(() => {
    doSomething(a, b);
  }, [a, b]);

  return <div>{expensiveValue}</div>;
};
```

### 3. **Proper Key Usage**
```jsx
// Good - stable, unique keys
{items.map(item => 
  <Item key={item.id} data={item} />
)}

// Bad - array index as key (for dynamic lists)
{items.map((item, index) => 
  <Item key={index} data={item} />
)}

// Bad - generating keys during render
{items.map(item => 
  <Item key={Math.random()} data={item} />
)}
```

## Time Complexity

- **Traditional diff algorithms**: O(n³)
- **React's diff algorithm**: O(n)

React achieves O(n) complexity through its heuristic approach:
1. Only comparing elements at the same level
2. Using component types and keys for quick identification
3. Not attempting to find optimal solution for tree transformation

## Best Practices

1. **Use stable keys**: Avoid using array indices for dynamic lists
2. **Keep component structure stable**: Avoid changing component types frequently
3. **Use React.memo wisely**: For components with expensive renders
4. **Minimize unnecessary re-renders**: Use useCallback and useMemo appropriately
5. **Avoid creating objects in render**: This breaks referential equality

## Example: Optimized List Component

```jsx
const TodoList = ({ todos, onToggle }) => {
  return (
    <ul>
      {todos.map(todo => (
        <TodoItem
          key={todo.id} // Stable key
          todo={todo}
          onToggle={onToggle}
        />
      ))}
    </ul>
  );
};

const TodoItem = React.memo(({ todo, onToggle }) => {
  const handleToggle = useCallback(() => {
    onToggle(todo.id);
  }, [todo.id, onToggle]);

  return (
    <li onClick={handleToggle}>
      {todo.text}
    </li>
  );
});
```

This demonstrates how the diff algorithm works efficiently with proper keys and memoization.

# [React Fiber Architecture and Its Phases](https://sunnychopper.medium.com/what-is-react-fiber-and-how-it-helps-you-build-a-high-performing-react-applications-57bceb706ff3)

## What is React Fiber?

React Fiber is a complete reimplementation of React's core algorithm introduced in React 16. It's a new reconciliation engine that enables incremental rendering, meaning React can split rendering work into chunks and spread it out over multiple frames. This allows React to pause, abort, or reuse work as new updates come in, making the UI more responsive.

With Fiber, however, the process of reconciliation and rendering is split up into two phases:

**Reconciliation** — React will figure out all the changes that need to occur based on the changes found in the DOM. It will then create a list of changes that need to occur. Since the algorithm uses the concepts of fibers, React is able to pause and resume this work at any time.

**Commit** — From there, React can decide on which set of changes to render and commit to one. Once the rendering process begins, however, it cannot be interrupted as you could with reconciliation.

## Key Goals of Fiber

1. **Incremental Rendering**: Ability to split rendering work into chunks
2. **Pause and Resume**: Ability to pause work and come back to it later
3. **Priority Assignment**: Assign priority to different types of updates
4. **Reuse Work**: Ability to reuse previously completed work
5. **Abort Work**: Ability to abort work if it's no longer needed

## Performance Benefits

1. **Time Slicing**: Large updates are broken into smaller chunks
2. **Prioritization**: Critical updates (user input) take precedence
3. **Interruptibility**: Low-priority work can be paused for urgent updates
4. **Batching**: Multiple state updates can be batched together
5. **Concurrent Rendering**: Multiple versions of the UI can be prepared simultaneously

React Fiber represents a fundamental shift in how React handles rendering, making it possible to build highly responsive user interfaces even with complex component trees and expensive computations.

# React useLayoutEffect Hook

## What is useLayoutEffect?

`useLayoutEffect` is a React hook that is similar to `useEffect`, but it fires synchronously after all DOM mutations. It's executed before the browser has a chance to paint, making it ideal for reading layout from the DOM and synchronously re-rendering.

## Syntax

```jsx
import { useLayoutEffect } from 'react';

function MyComponent() {
  useLayoutEffect(() => {
    // Effect code here
    
    return () => {
      // Cleanup code here (optional)
    };
  }, [dependencies]); // Dependencies array (optional)
}
```

## Key Differences: useEffect vs useLayoutEffect

| Aspect | useEffect | useLayoutEffect |
|--------|-----------|----------------|
| **Execution Timing** | Asynchronous, after paint | Synchronous, before paint |
| **Performance** | Better for most cases | Can block visual updates |
| **Use Case** | Side effects, API calls | DOM measurements, layout changes |
| **Browser Painting** | Doesn't block painting | Blocks painting until complete |

## When to Use useLayoutEffect

### ✅ Use useLayoutEffect when:

1. **Reading layout properties** (offsetWidth, offsetHeight, getBoundingClientRect)
2. **Making DOM changes** that users should not see
3. **Synchronously updating the DOM** before paint
4. **Implementing animations** that require immediate DOM updates
5. **Managing focus** or scroll position
6. **Preventing visual flicker** or layout shifts

### ❌ Avoid useLayoutEffect when:

1. **Making API calls** or async operations
2. **Updating state** that doesn't affect layout
3. **Performing heavy computations** (use useEffect instead)
4. **Side effects** that don't need to block painting

## Summary

`useLayoutEffect` is a powerful hook for synchronous DOM manipulations that need to happen before the browser paints. Use it judiciously for layout-related operations, but prefer `useEffect` for most other side effects to maintain good performance.

# React Synthetic Events

## What are Synthetic Events?

React Synthetic Events are a cross-browser wrapper around the browser's native events. They combine the behavior of different browsers into one API so that all events behave consistently across all browsers. Synthetic events have the same interface as native events, including `stopPropagation()` and `preventDefault()`, except the events work identically across all browsers.

## Key Features

1. **Cross-browser compatibility**: Works consistently across different browsers
2. **Same API as native events**: Familiar methods like `preventDefault()` and `stopPropagation()`
3. **Event pooling**: Reuses event objects for performance (React 16 and below)
4. **Event delegation**: Uses a single event listener at the document root
5. **Automatic cleanup**: Events are automatically cleaned up when components unmount