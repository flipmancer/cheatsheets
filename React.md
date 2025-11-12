# React Cheatsheet

> Complete guide to building React applications. From components and hooks to performance optimization and best practices.

## Table of Contents

- [React Cheatsheet](#react-cheatsheet)
	- [Table of Contents](#table-of-contents)
	- [Overview](#overview)
	- [Project Setup](#project-setup)
	- [Core React Concepts](#core-react-concepts)
		- [1. Component Structure](#1-component-structure)
		- [2. State Management (`useState`)](#2-state-management-usestate)
		- [3. Props (Parent → Child Communication)](#3-props-parent--child-communication)
		- [4. Event Handling](#4-event-handling)
		- [5. Conditional Rendering](#5-conditional-rendering)
		- [6. Lists \& Iteration](#6-lists--iteration)
		- [7. useEffect Hook (Side Effects)](#7-useeffect-hook-side-effects)
		- [8. useRef Hook (Persistent References)](#8-useref-hook-persistent-references)
		- [9. useCallback Hook (Memoize Functions)](#9-usecallback-hook-memoize-functions)
		- [10. useMemo Hook (Memoize Values)](#10-usememo-hook-memoize-values)
	- [API Integration](#api-integration)
		- [11. Fetch Data (REST API)](#11-fetch-data-rest-api)
		- [12. POST Request (Form Submission)](#12-post-request-form-submission)
		- [13. WebSocket Connection](#13-websocket-connection)
	- [Forms](#forms)
		- [14. Controlled Inputs](#14-controlled-inputs)
		- [15. Input Types](#15-input-types)
	- [Styling](#styling)
		- [16. CSS Classes](#16-css-classes)
		- [17. Inline Styles](#17-inline-styles)
	- [Component Patterns](#component-patterns)
		- [18. Extracting Components](#18-extracting-components)
		- [19. Custom Hooks (Reusable Logic)](#19-custom-hooks-reusable-logic)
	- [Common Patterns](#common-patterns)
		- [20. Loading States](#20-loading-states)
		- [21. Error Handling](#21-error-handling)
		- [22. Debouncing User Input](#22-debouncing-user-input)
		- [23. Polling for Updates](#23-polling-for-updates)
	- [Environment Variables](#environment-variables)
	- [Quick Reference](#quick-reference)
	- [Debugging Tips](#debugging-tips)
	- [File Structure Best Practice](#file-structure-best-practice)
	- [Performance Tips](#performance-tips)
	- [Security \& Safety](#security--safety)
	- [Next Steps](#next-steps)
	- [Resources](#resources)

## Overview

- Purpose: Quick reference for React fundamentals, hooks, patterns, and modern best practices
- Audience: Beginner to Advanced
- Scope: React core concepts, hooks, state management, API integration, performance. Excludes framework-specific routing (Next.js, Remix) and state libraries (Redux, Zustand).
- Updated: 2025-11-09

---

## Project Setup

```shell
# Create new React app
npx create-react-app my-app
cd my-app

# Install dependencies
npm install

# Start development server (auto-reload on save)
npm start

# Build for production
npm run build

# Run tests
npm test
```

---

## Core React Concepts

### 1. Component Structure

Short explanation:

- What it is: Reusable UI building blocks that accept props and return JSX
- Why it matters: Modular, maintainable code - build complex UIs from simple pieces
- Mental model: Like LEGO blocks - combine small pieces to build larger structures

```jsx
import { useState, useEffect } from 'react';

function MyComponent({ propName }) {
  // State
  const [count, setCount] = useState(0);

  // Side effects
  useEffect(() => {
    // Runs after component renders
    console.log('Component mounted or updated');

    return () => {
      // Cleanup function (optional)
      console.log('Component will unmount');
    };
  }, [count]); // Re-run when 'count' changes

  // Event handler
  const handleClick = () => {
    setCount(count + 1);
  };

  // JSX return
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={handleClick}>Increment</button>
    </div>
  );
}

export default MyComponent;
```

Common pitfalls:

- Forgetting dependency array in `useEffect` (causes infinite loops)
- Mutating state directly instead of using setter functions
- Not returning cleanup function in `useEffect` for subscriptions/timers

---

### 2. State Management (`useState`)

Short explanation:

- What it is: Hook to add local state to functional components
- Why it matters: Makes components interactive and dynamic
- Mental model: Like a variable that triggers re-render when changed

```jsx
// Primitive values
const [name, setName] = useState('');
const [age, setAge] = useState(0);
const [isActive, setIsActive] = useState(false);

// Objects (use spread for immutability)
const [user, setUser] = useState({ name: '', email: '' });
setUser({ ...user, name: 'John' }); // Correct
// user.name = 'John'; // Wrong - mutates state

// Arrays
const [items, setItems] = useState([]);
setItems([...items, newItem]); // Add item
setItems(items.filter(item => item.id !== id)); // Remove item

// Functional updates (when new state depends on old state)
setCount(prevCount => prevCount + 1);
```

Common pitfalls:

- Mutating state directly instead of creating new objects/arrays
- Using stale state in callbacks (use functional updates)
- Not initializing state properly (causes `undefined` errors)

---

### 3. Props (Parent → Child Communication)

Short explanation:

- What it is: Data passed from parent to child components
- Why it matters: Enable component reusability and composition
- Mental model: Like function arguments - customize behavior without changing code

```jsx
// Parent component
function App() {
  return <ChildComponent title="Hello" count={5} onAction={handleAction} />;
}

// Child component with destructuring
function ChildComponent({ title, count, onAction }) {
  return (
    <div>
      <h1>{title}: {count}</h1>
      <button onClick={onAction}>Click</button>
    </div>
  );
}

// Default props
function ChildComponent({ title = 'Default', count = 0 }) {
  return <h1>{title}: {count}</h1>;
}

// Children prop (special prop)
function Container({ children }) {
  return <div className="container">{children}</div>;
}

// Usage
<Container>
  <p>This is the children content</p>
</Container>
```

---

### 4. Event Handling

Short explanation:

- What it is: Respond to user interactions (clicks, typing, etc.)
- When to use: Any interactive UI element
- Gotchas: Event handlers receive synthetic events, not native DOM events

```jsx
// Click events
<button onClick={() => console.log('Clicked')}>Click Me</button>
<button onClick={handleClick}>Click Me</button>

// Form events
const handleSubmit = (e) => {
  e.preventDefault(); // Prevent page reload
  console.log('Form submitted');
};

const handleChange = (e) => {
  setValue(e.target.value); // Get input value
};

// Event with parameters
const handleDelete = (id) => {
  console.log('Delete item:', id);
};

<button onClick={() => handleDelete(123)}>Delete</button>

// Complete form example
<form onSubmit={handleSubmit}>
  <input type="text" onChange={handleChange} value={value} />
  <button type="submit">Submit</button>
</form>
```

---

### 5. Conditional Rendering

Short explanation:

- What it is: Show/hide UI elements based on conditions
- Mental model: Like if/else statements but in JSX

```jsx
// Ternary operator
{isLoggedIn ? <Dashboard /> : <Login />}

// Logical AND (short-circuit)
{error && <ErrorMessage text={error} />}
{count > 0 && <p>Items: {count}</p>}

// Logical OR (fallback)
{username || 'Guest'}

// If/else with variables
let content;
if (isLoading) {
  content = <Spinner />;
} else if (error) {
  content = <Error message={error} />;
} else {
  content = <Data items={data} />;
}

return <div>{content}</div>;

// Switch-style with object
const statusComponents = {
  loading: <Spinner />,
  error: <Error />,
  success: <Data />
};

return statusComponents[status];
```

---

### 6. Lists & Iteration

Short explanation:

- What it is: Render multiple elements from arrays
- Why it matters: Dynamic UIs with variable content
- Gotchas: Must provide unique `key` prop for each item

```jsx
const markets = ['BTC', 'ETH', 'SOL'];

return (
  <ul>
    {markets.map((market, index) => (
      <li key={market}>{market}</li>
    ))}
  </ul>
);

// With objects (use unique ID as key)
const users = [
  { id: 1, name: 'Alice' },
  { id: 2, name: 'Bob' }
];

return (
  <ul>
    {users.map(user => (
      <li key={user.id}>
        {user.name}
        <button onClick={() => handleDelete(user.id)}>Delete</button>
      </li>
    ))}
  </ul>
);

// Filter and map
const activeUsers = users.filter(u => u.isActive);
return (
  <ul>
    {activeUsers.map(user => (
      <li key={user.id}>{user.name}</li>
    ))}
  </ul>
);
```

Common pitfalls:

- Using array index as key (causes bugs when items reorder)
- Forgetting key prop (React warning)
- Not using unique keys (causes rendering issues)

---

### 7. useEffect Hook (Side Effects)

Short explanation:

- What it is: Run code after render (data fetching, subscriptions, timers)
- Why it matters: Synchronize components with external systems
- Mental model: "Do something after React finishes updating the screen"

```jsx
// Run once on mount
useEffect(() => {
  fetchData();
}, []); // Empty dependency array = run once

// Run on every render
useEffect(() => {
  console.log('Component rendered');
}); // No dependency array = run every render

// Run when specific value changes
useEffect(() => {
  console.log('Count changed:', count);
}, [count]); // Run when count changes

// Cleanup function (important!)
useEffect(() => {
  const timer = setInterval(() => {
    console.log('Tick');
  }, 1000);

  return () => clearInterval(timer); // Cleanup on unmount
}, []);

// Multiple dependencies
useEffect(() => {
  console.log('User or page changed');
}, [user, page]);

// Async operations
useEffect(() => {
  async function loadData() {
    const data = await fetchData();
    setData(data);
  }
  loadData();
}, []);
```

Common pitfalls:

- Forgetting dependency array (infinite loops)
- Missing dependencies (stale closures)
- Not cleaning up subscriptions/timers (memory leaks)
- Using async function directly in useEffect (wrap in async IIFE)

---

### 8. useRef Hook (Persistent References)

Short explanation:

- What it is: Store mutable values that persist across renders without causing re-renders
- When to use: DOM access, storing timers/intervals, previous values, third-party library instances
- Mental model: Like a box that React ignores - you can change contents without triggering updates

```jsx
import { useRef } from 'react';

// Access DOM elements
const inputRef = useRef(null);

useEffect(() => {
  inputRef.current.focus(); // Direct DOM access
}, []);

<input ref={inputRef} type="text" />

// Store mutable values (no re-render)
const countRef = useRef(0);

const handleClick = () => {
  countRef.current += 1; // Update without re-rendering
  console.log('Clicked', countRef.current, 'times');
};

// Store previous value
const prevValueRef = useRef();

useEffect(() => {
  prevValueRef.current = value;
}, [value]);

console.log('Previous value:', prevValueRef.current);

// Store timer/interval ID
const timerRef = useRef(null);

const startTimer = () => {
  timerRef.current = setInterval(() => {
    console.log('Tick');
  }, 1000);
};

const stopTimer = () => {
  clearInterval(timerRef.current);
};

// Third-party library instances (ag-Grid, Chart.js, etc.)
const gridRef = useRef(null);

const scrollToTop = () => {
  gridRef.current.api.ensureIndexVisible(0, 'top');
};
```

**When to use:**

- DOM access (focus, scroll, measure)
- Mutable values without re-render
- Timers, intervals, WebSocket connections
- Third-party library instances
- Previous values for comparison

---

### 9. useCallback Hook (Memoize Functions)

Short explanation:

- What it is: Memoize functions to prevent unnecessary re-creations
- Why it matters: Prevents child components from re-rendering when props haven't changed
- Mental model: "Remember this function unless dependencies change"

```jsx
import { useCallback } from 'react';

// Without useCallback: new function every render
const handleClick = () => console.log('click');

// With useCallback: same function instance
const handleClick = useCallback(() => {
  console.log('Clicked:', count);
}, [count]); // Recreate only when count changes

// Prevent child re-renders
const handleSubmit = useCallback((data) => {
  console.log('Submitting:', data);
}, []); // Never recreated

// Pass to memoized child
const MemoizedChild = React.memo(ChildComponent);

<MemoizedChild onSubmit={handleSubmit} />

// With dependencies
const handleFilter = useCallback((term) => {
  return items.filter(item => item.name.includes(term));
}, [items]); // Recreate when items change

// In useEffect dependencies
useEffect(() => {
  fetchData();
}, [fetchData]); // fetchData should be useCallback
```

**When to use:**

- Pass to `React.memo` children
- Include in `useEffect` dependencies
- Expensive event handlers
- Optimize performance (measure first!)

---

### 10. useMemo Hook (Memoize Values)

Short explanation:

- What it is: Memoize expensive computations
- When to use: Expensive calculations, derived state, object/array creation
- Mental model: "Remember this result unless dependencies change"

```jsx
import { useMemo } from 'react';

// Expensive calculation
const expensiveValue = useMemo(() => {
  return items.reduce((sum, item) => sum + item.price, 0);
}, [items]); // Recalculate only when items change

// Filtering/sorting large lists
const filteredItems = useMemo(() => {
  return items.filter(item => item.category === selectedCategory);
}, [items, selectedCategory]);

// Stable object reference (prevent re-renders)
const config = useMemo(() => ({
  theme: 'dark',
  language: 'en'
}), []); // Same object instance

// Derived state
const stats = useMemo(() => {
  return {
    total: items.length,
    completed: items.filter(i => i.done).length,
    pending: items.filter(i => !i.done).length
  };
}, [items]);
```

**When to use:**

- Expensive calculations (loops over large arrays)
- Derived state from props/state
- Stable object/array references for child props
- Only optimize when you measure performance issues

---

## API Integration

### 11. Fetch Data (REST API)

Short explanation:

- What it is: Load data from external sources
- When to use: Component needs server data
- Mental model: "Ask the server, show loading, handle result or error"

```jsx
import { useState, useEffect } from 'react';

function DataFetcher() {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    fetch('http://localhost:3000/api/markets')
      .then(res => {
        if (!res.ok) throw new Error('Network error');
        return res.json();
      })
      .then(data => {
        setData(data);
        setLoading(false);
      })
      .catch(err => {
        setError(err.message);
        setLoading(false);
      });
  }, []);

  if (loading) return <p>Loading...</p>;
  if (error) return <p>Error: {error}</p>;

  return <div>{JSON.stringify(data)}</div>;
}

// With async/await
useEffect(() => {
  async function loadData() {
    try {
      setLoading(true);
      const response = await fetch('/api/data');
      if (!response.ok) throw new Error('Failed to fetch');
      const result = await response.json();
      setData(result);
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  }
  loadData();
}, []);
```

---

### 12. POST Request (Form Submission)

```jsx
const handleSubmit = async (e) => {
  e.preventDefault();

  try {
    const response = await fetch('http://localhost:3000/api/order', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({ price, size }),
    });

    if (!response.ok) throw new Error('Failed to submit');

    const result = await response.json();
    console.log('Success:', result);
  } catch (error) {
    console.error('Error:', error);
  }
};
```

---

### 13. WebSocket Connection

Short explanation:

- What it is: Real-time bidirectional communication
- When to use: Live updates, chat, notifications
- Mental model: "Open phone line - receive updates as they happen"

```jsx
useEffect(() => {
  const ws = new WebSocket('ws://localhost:3000/ws');

  ws.onopen = () => {
    console.log('WebSocket connected');
    ws.send(JSON.stringify({ type: 'subscribe', channel: 'market' }));
  };

  ws.onmessage = (event) => {
    const data = JSON.parse(event.data);
    setLiveData(data);
  };

  ws.onerror = (error) => {
    console.error('WebSocket error:', error);
  };

  ws.onclose = () => {
    console.log('WebSocket disconnected');
  };

  return () => ws.close(); // Cleanup
}, []);

// WebSocket with useRef (persistent connection)
const wsRef = useRef(null);

useEffect(() => {
  wsRef.current = new WebSocket('ws://localhost:3000/ws');
  wsRef.current.onmessage = (event) => {
    setData(prev => [JSON.parse(event.data), ...prev].slice(0, 100));
  };
  return () => wsRef.current?.close();
}, []);

// Send from outside useEffect
const sendMessage = useCallback(() => {
  if (wsRef.current?.readyState === WebSocket.OPEN) {
    wsRef.current.send(JSON.stringify({ type: 'ping' }));
  }
}, []);
```

---

## Forms

### 14. Controlled Inputs

Short explanation:

- What it is: Form inputs controlled by React state
- Why it matters: React controls the value, enabling validation and manipulation
- Mental model: State is the "source of truth" for input values

```jsx
function Form() {
  const [formData, setFormData] = useState({
    name: '',
    email: '',
    age: 0
  });

  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormData({
      ...formData,
      [name]: value
    });
  };

  const handleSubmit = (e) => {
    e.preventDefault();
    console.log('Form data:', formData);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        name="name"
        value={formData.name}
        onChange={handleChange}
        placeholder="Name"
      />
      <input
        name="email"
        type="email"
        value={formData.email}
        onChange={handleChange}
        placeholder="Email"
      />
      <input
        name="age"
        type="number"
        value={formData.age}
        onChange={handleChange}
        placeholder="Age"
      />
      <button type="submit">Submit</button>
    </form>
  );
}
```

---

### 15. Input Types

```jsx
// Text input
<input type="text" value={text} onChange={(e) => setText(e.target.value)} />

// Number input
<input type="number" step="0.01" value={price} onChange={(e) => setPrice(e.target.value)} />

// Checkbox
<input type="checkbox" checked={isChecked} onChange={(e) => setIsChecked(e.target.checked)} />

// Radio buttons
<input
  type="radio"
  name="option"
  value="1"
  checked={option === '1'}
  onChange={(e) => setOption(e.target.value)}
/>

// Select dropdown
<select value={selected} onChange={(e) => setSelected(e.target.value)}>
  <option value="btc">Bitcoin</option>
  <option value="eth">Ethereum</option>
</select>

// Textarea
<textarea value={text} onChange={(e) => setText(e.target.value)} rows={4} />

// File input (uncontrolled)
const handleFileChange = (e) => {
  const file = e.target.files[0];
  console.log(file);
};

<input type="file" onChange={handleFileChange} />
```

---

## Styling

### 16. CSS Classes

```jsx
// Import CSS file
import './App.css';

// Apply class
<div className="container">Content</div>

// Conditional classes
<div className={isActive ? 'active' : 'inactive'}>Content</div>

// Multiple classes
<div className={`base-class ${isActive ? 'active' : ''}`}>Content</div>

// Template literals for complex conditions
<div className={`
  base
  ${isLarge ? 'large' : 'small'}
  ${isActive && 'active'}
`}>Content</div>

// CSS Modules (scoped styles)
import styles from './Component.module.css';
<div className={styles.container}>Content</div>
```

---

### 17. Inline Styles

```jsx
const style = {
  color: 'red',
  fontSize: '16px',
  backgroundColor: '#f0f0f0'
};

<div style={style}>Styled content</div>

// Direct inline
<div style={{ color: 'blue', padding: '10px' }}>Content</div>

// Dynamic styles
const dynamicStyle = {
  color: isActive ? 'green' : 'gray',
  fontSize: `${size}px`
};

<div style={dynamicStyle}>Content</div>
```

---

## Component Patterns

### 18. Extracting Components

Short explanation:

- What it is: Split large components into smaller, focused ones
- Why it matters: Reusability, testability, maintainability
- Mental model: "Single responsibility - each component does one thing well"

```jsx
// Before: Everything in one component
function App() {
  const [reward, setReward] = useState(0);
  return (
    <div>
      <div className="results">
        <h2>Reward: ${reward}</h2>
      </div>
      <form>...</form>
    </div>
  );
}

// After: Separated concerns
function ResultsDisplay({ reward }) {
  return (
    <div className="results">
      <h2>Reward: ${reward}</h2>
    </div>
  );
}

function RewardsForm({ onCalculate }) {
  return <form onSubmit={onCalculate}>...</form>;
}

function App() {
  const [reward, setReward] = useState(0);
  return (
    <div>
      <ResultsDisplay reward={reward} />
      <RewardsForm onCalculate={(val) => setReward(val)} />
    </div>
  );
}
```

---

### 19. Custom Hooks (Reusable Logic)

Short explanation:

- What it is: Extract stateful logic into reusable functions
- When to use: Share logic between components, organize complex effects
- Mental model: "Functions that use hooks - package behavior for reuse"

```jsx
// Custom hook for API calls
function useFetch(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    async function loadData() {
      try {
        const response = await fetch(url);
        if (!response.ok) throw new Error('Failed to fetch');
        const result = await response.json();
        setData(result);
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    }
    loadData();
  }, [url]);

  return { data, loading, error };
}

// Usage
function MyComponent() {
  const { data, loading, error } = useFetch('http://localhost:3000/api/markets');

  if (loading) return <p>Loading...</p>;
  if (error) return <p>Error: {error}</p>;
  return <div>{JSON.stringify(data)}</div>;
}

// Custom hook for local storage
function useLocalStorage(key, initialValue) {
  const [value, setValue] = useState(() => {
    const stored = localStorage.getItem(key);
    return stored ? JSON.parse(stored) : initialValue;
  });

  useEffect(() => {
    localStorage.setItem(key, JSON.stringify(value));
  }, [key, value]);

  return [value, setValue];
}

// Usage
function Settings() {
  const [theme, setTheme] = useLocalStorage('theme', 'light');
  return (
    <button onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}>
      Current theme: {theme}
    </button>
  );
}
```

---

## Common Patterns

### 20. Loading States

```jsx
function Dashboard() {
  const [data, setData] = useState(null);
  const [isLoading, setIsLoading] = useState(true);

  useEffect(() => {
    fetchData().then(result => {
      setData(result);
      setIsLoading(false);
    });
  }, []);

  if (isLoading) {
    return <div className="spinner">Loading...</div>;
  }

  return <div>{/* Render data */}</div>;
}
```

---

### 21. Error Handling

```jsx
function SafeComponent() {
  const [error, setError] = useState(null);

  const handleAction = async () => {
    try {
      const result = await riskyOperation();
      setData(result);
    } catch (err) {
      setError(err.message);
    }
  };

  if (error) {
    return <div className="error">Error: {error}</div>;
  }

  return <button onClick={handleAction}>Do Action</button>;
}

// Error Boundary (class component)
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true };
  }

  componentDidCatch(error, errorInfo) {
    console.error('Error:', error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return <h1>Something went wrong.</h1>;
    }
    return this.props.children;
  }
}

// Usage
<ErrorBoundary>
  <MyComponent />
</ErrorBoundary>
```

---

### 22. Debouncing User Input

Short explanation:

- What it is: Delay execution until user stops typing
- When to use: Search inputs, autocomplete, API calls on keystroke
- Mental model: "Wait for pause before acting"

```jsx
import { useState, useEffect } from 'react';

function SearchBox() {
  const [query, setQuery] = useState('');
  const [debouncedQuery, setDebouncedQuery] = useState('');

  // Debounce: wait 500ms after user stops typing
  useEffect(() => {
    const timer = setTimeout(() => {
      setDebouncedQuery(query);
    }, 500);

    return () => clearTimeout(timer);
  }, [query]);

  // Trigger search when debounced value changes
  useEffect(() => {
    if (debouncedQuery) {
      searchAPI(debouncedQuery);
    }
  }, [debouncedQuery]);

  return <input value={query} onChange={(e) => setQuery(e.target.value)} />;
}

// Custom hook
function useDebounce(value, delay) {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const timer = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);

    return () => clearTimeout(timer);
  }, [value, delay]);

  return debouncedValue;
}

// Usage
const debouncedSearchTerm = useDebounce(searchTerm, 500);
```

---

### 23. Polling for Updates

```jsx
useEffect(() => {
  const interval = setInterval(() => {
    fetchLatestData().then(data => setData(data));
  }, 5000); // Poll every 5 seconds

  return () => clearInterval(interval); // Cleanup
}, []);
```

---

## Environment Variables

```jsx
// .env file (create in project root)
REACT_APP_API_URL=http://localhost:3000
REACT_APP_WS_URL=ws://localhost:3000/ws

// Access in code
const apiUrl = process.env.REACT_APP_API_URL;
const wsUrl = process.env.REACT_APP_WS_URL;

fetch(`${apiUrl}/markets/metadata`);
```

**Important:**

- Must prefix with `REACT_APP_` in Create React App
- Restart dev server after changing .env
- Don't commit secrets to .env (use .env.local)

---

## Quick Reference

Essential React rules and conventions:

1. **Single root element**: Wrap in `<div>` or `<>...</>` (fragment)
2. **Close all tags**: `<img />`, `<br />`, `<input />`
3. **className not class**: `<div className="container">`
4. **camelCase attributes**: `onClick`, `onChange`, `onSubmit`
5. **JavaScript expressions in {}**: `{variable}`, `{2 + 2}`, `{myFunction()}`
6. **Comments**: `{/* Comment here */}`
7. **Keys in lists**: Always provide unique `key` prop
8. **Props are read-only**: Never modify props directly

---

## Debugging Tips

```jsx
// Console log in JSX
{console.log('Current state:', data) || null}

// Conditional breakpoint
if (condition) debugger;

// React DevTools (Chrome extension): Inspect component state/props
// Use breakpoints in browser DevTools
// Check Network tab for API calls
// Check Console for errors and warnings
```

Checklist:

- Check console for errors and warnings
- Use React DevTools to inspect component tree
- Verify props are passed correctly
- Check state updates (use DevTools)
- Validate API responses in Network tab
- Isolate issues with minimal reproduction

---

## File Structure Best Practice

```txt
src/
├── components/
│   ├── common/
│   │   ├── Button.jsx
│   │   └── Input.jsx
│   ├── features/
│   │   ├── RewardsForm.jsx
│   │   └── ResultsDisplay.jsx
│   └── layout/
│       ├── Header.jsx
│       └── Footer.jsx
├── hooks/
│   ├── useFetch.js
│   └── useLocalStorage.js
├── services/
│   └── api.js          // Centralize all API calls
├── utils/
│   └── calculations.js
├── styles/
│   ├── App.css
│   └── variables.css
├── App.jsx
└── index.js
```

Notes:

- Keep components small and focused (single responsibility)
- Co-locate related files (component + styles + tests)
- Group by feature or domain, not by type
- Use index.js for cleaner imports

---

## Performance Tips

1. **Use `React.memo`**: Prevent re-renders for components with same props
2. **Use `useCallback`**: Memoize functions passed to children
3. **Use `useMemo`**: Memoize expensive calculations
4. **Use `useRef`**: For values that don't need re-renders
5. **Keys in lists**: Stable keys help React optimize updates
6. **Lazy loading**: `React.lazy()` and `Suspense` for code splitting
7. **Virtualization**: Use react-window for large lists
8. **Avoid inline objects/functions**: Creates new reference every render
9. **Debounce inputs**: Reduce API calls and expensive operations
10. **Profile before optimizing**: Use React DevTools Profiler

---

## Security & Safety

- **Sanitize user input**: Prevent XSS attacks (React escapes by default, but be careful with `dangerouslySetInnerHTML`)
- **Validate on backend**: Never trust client-side validation alone
- **Use HTTPS**: Encrypt data in transit
- **Don't expose secrets**: Use environment variables, never commit API keys
- **Content Security Policy**: Add CSP headers
- **Dependency audits**: Run `npm audit` regularly
- **Update dependencies**: Keep React and libraries up to date

---

## Next Steps

- **Learn React Router**: Client-side routing for SPAs
- **State management**: Redux Toolkit, Zustand, or Context API
- **Server-side rendering**: Next.js or Remix
- **Testing**: Jest, React Testing Library, Cypress
- **TypeScript**: Add type safety to React
- **Build tools**: Vite (faster than Create React App)
- **UI libraries**: Material-UI, Chakra UI, shadcn/ui

---

## Resources

- Official docs: [React Documentation](https://react.dev)
- Interactive tutorial: [React Tutorial](https://react.dev/learn)
- Hooks reference: [Hooks API Reference](https://react.dev/reference/react)
- Patterns: [Patterns.dev](https://www.patterns.dev/posts/react-patterns)
- Testing: [React Testing Library](https://testing-library.com/react)
- TypeScript: [React TypeScript Cheatsheet](https://react-typescript-cheatsheet.netlify.app/)
