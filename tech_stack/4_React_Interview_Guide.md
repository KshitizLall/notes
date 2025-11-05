# React Interview Guide - Complete Basics

## Table of Contents
1. [React Fundamentals](#react-fundamentals)
2. [Components](#components)
3. [JSX](#jsx)
4. [State & Props](#state--props)
5. [Hooks](#hooks)
6. [Event Handling](#event-handling)
7. [Conditional Rendering](#conditional-rendering)
8. [Lists & Keys](#lists--keys)
9. [Forms](#forms)
10. [Lifecycle Methods](#lifecycle-methods)
11. [Context API](#context-api)
12. [Performance Optimization](#performance-optimization)
13. [Common Patterns](#common-patterns)
14. [Common Interview Questions](#common-interview-questions)

---

## React Fundamentals

### What is React?
React is a **JavaScript library** for building user interfaces using reusable components. It uses a **declarative** approach where you describe what the UI should look like, and React handles the updates.

### Key Features
- **Component-Based**: Break UI into reusable components
- **Declarative**: Describe what UI should be, not how to build it
- **Virtual DOM**: Efficient rendering through a virtual representation of the DOM
- **Unidirectional Data Flow**: Data flows from parent to child components
- **Fast**: React optimizes rendering through diffing algorithms

### React vs Other Frameworks

| Feature | React | Vue | Angular |
|---------|-------|-----|---------|
| Learning Curve | Moderate | Easy | Steep |
| Size | ~40KB | ~33KB | ~130KB |
| Type Safety | Optional (TS) | Optional (TS) | Built-in (TS) |
| Bundle Size | Smaller | Smaller | Larger |
| Component Reusability | High | High | High |
| Community | Large | Medium | Large |

---

## Components

### Functional Components (Modern Approach)
Functional components are JavaScript functions that return JSX.

```jsx
function Greeting({ name }) {
  return <h1>Hello, {name}!</h1>;
}

// Arrow function syntax
const Greeting = ({ name }) => {
  return <h1>Hello, {name}!</h1>;
};
```

### Class Components (Legacy)
Class components extend `React.Component` and have more features.

```jsx
class Greeting extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}!</h1>;
  }
}
```

### Functional vs Class Components

| Feature | Functional | Class |
|---------|-----------|-------|
| Syntax | Simple function | ES6 class |
| State | Via Hooks (useState) | this.state |
| Lifecycle | Via Hooks (useEffect) | Lifecycle methods |
| this binding | Not needed | Required |
| Performance | Better (optimized) | Slightly heavier |
| Modern Standard | Yes | Legacy |

### Component Composition
Components can be composed together:

```jsx
function App() {
  return (
    <div>
      <Header />
      <MainContent />
      <Footer />
    </div>
  );
}
```

---

## JSX

### What is JSX?
JSX is a syntax extension to JavaScript that looks like XML/HTML. It gets compiled to `React.createElement()` calls.

```jsx
// JSX
const element = <h1>Hello, World!</h1>;

// Compiles to
const element = React.createElement('h1', null, 'Hello, World!');
```

### JSX Rules

1. **Single Root Element**: Each component must return one root element
```jsx
// Wrong
return (
  <div>Hello</div>
  <div>World</div>
);

// Correct
return (
  <div>
    <div>Hello</div>
    <div>World</div>
  </div>
);

// Also correct (Fragment)
return (
  <>
    <div>Hello</div>
    <div>World</div>
  </>
);
```

2. **Close All Tags**: All elements must be self-closed or have closing tags
```jsx
// Correct
<input type="text" />
<input type="text"></input>
```

3. **className instead of class**
```jsx
<div className="container">Content</div>
```

4. **Expressions in Curly Braces**
```jsx
const name = "John";
const element = <h1>Hello, {name}!</h1>;
const age = <p>{2024 - 1990}</p>;
```

5. **Attributes are camelCase**
```jsx
<input
  onClick={handleClick}
  onChange={handleChange}
  onMouseEnter={handleHover}
/>
```

---

## State & Props

### Props (Properties)
Props are read-only inputs to a component. They allow you to pass data from parent to child.

```jsx
// Parent component
function App() {
  return <Greeting name="John" age={25} />;
}

// Child component
function Greeting({ name, age }) {
  return <p>{name} is {age} years old</p>;
}

// Alternative destructuring
function Greeting(props) {
  return <p>{props.name} is {props.age} years old</p>;
}
```

### Default Props
```jsx
function Greeting({ name = "Guest", age = 0 }) {
  return <p>{name} is {age} years old</p>;
}
```

### PropTypes (Type Checking)
```jsx
import PropTypes from 'prop-types';

function Greeting({ name, age }) {
  return <p>{name} is {age} years old</p>;
}

Greeting.propTypes = {
  name: PropTypes.string.isRequired,
  age: PropTypes.number,
};
```

### State (useState Hook)
State is mutable data that belongs to a component. Changes to state trigger re-renders.

```jsx
import { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}
```

### State vs Props

| Aspect | State | Props |
|--------|-------|-------|
| Mutable | Yes | No |
| Owned by | Component | Parent |
| Updated by | Component itself | Parent component |
| Purpose | Internal data | Pass data down |
| Triggers re-render | Yes | Yes |

### Lifting State Up
Move state to a common parent to share it between siblings:

```jsx
function Parent() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <Counter count={count} setCount={setCount} />
      <Display count={count} />
    </div>
  );
}

function Counter({ count, setCount }) {
  return <button onClick={() => setCount(count + 1)}>Inc</button>;
}

function Display({ count }) {
  return <p>Count: {count}</p>;
}
```

---

## Hooks

### What are Hooks?
Hooks are functions that let you use state and other React features in functional components.

### useState
Manages state in functional components.

```jsx
const [state, setState] = useState(initialValue);

// Example
const [name, setName] = useState("John");
const [count, setCount] = useState(0);

// Updating state
setName("Jane");
setCount(count + 1);
```

**Important**: Never call hooks conditionally or inside loops.

```jsx
// Wrong
if (condition) {
  const [state, setState] = useState(0);
}

// Correct
const [state, setState] = useState(0);
if (condition) {
  // use state
}
```

### useEffect
Performs side effects in components (API calls, subscriptions, etc).

```jsx
import { useEffect } from 'react';

useEffect(() => {
  // Side effect code here
  console.log('Component mounted or dependency changed');

  // Cleanup function (optional)
  return () => {
    console.log('Cleanup on unmount or before next effect');
  };
}, [dependency]); // Dependency array
```

#### Dependency Array Behavior

```jsx
// Runs on every render
useEffect(() => {});

// Runs once on mount and cleanup on unmount
useEffect(() => {
  return () => {};
}, []);

// Runs when dependency changes
useEffect(() => {}, [dependency]);

// Runs when any of the dependencies change
useEffect(() => {}, [dep1, dep2]);
```

### useContext
Access context values without wrapping in a Consumer component.

```jsx
const ThemeContext = React.createContext();

function App() {
  return (
    <ThemeContext.Provider value="dark">
      <Component />
    </ThemeContext.Provider>
  );
}

function Component() {
  const theme = useContext(ThemeContext);
  return <p>Theme: {theme}</p>;
}
```

### useReducer
Complex state logic with multiple sub-values.

```jsx
const initialState = { count: 0 };

function reducer(state, action) {
  switch (action.type) {
    case 'INCREMENT':
      return { count: state.count + 1 };
    case 'DECREMENT':
      return { count: state.count - 1 };
    default:
      return state;
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, initialState);

  return (
    <div>
      <p>Count: {state.count}</p>
      <button onClick={() => dispatch({ type: 'INCREMENT' })}>+</button>
      <button onClick={() => dispatch({ type: 'DECREMENT' })}>-</button>
    </div>
  );
}
```

### useMemo
Memoize expensive computations.

```jsx
const memoizedValue = useMemo(() => {
  return expensiveComputation(a, b);
}, [a, b]);
```

### useCallback
Memoize function definitions.

```jsx
const handleClick = useCallback(() => {
  doSomething(a, b);
}, [a, b]);
```

### useRef
Access DOM elements directly or store mutable values.

```jsx
function TextInput() {
  const inputRef = useRef(null);

  const focusInput = () => {
    inputRef.current.focus();
  };

  return (
    <>
      <input ref={inputRef} type="text" />
      <button onClick={focusInput}>Focus</button>
    </>
  );
}
```

### Custom Hooks
Create reusable stateful logic.

```jsx
function useFormInput(initialValue) {
  const [value, setValue] = useState(initialValue);

  return {
    value,
    bind: {
      value,
      onChange: e => setValue(e.target.value)
    },
    reset: () => setValue(initialValue)
  };
}

function LoginForm() {
  const email = useFormInput('');
  const password = useFormInput('');

  const handleSubmit = (e) => {
    e.preventDefault();
    console.log(email.value, password.value);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input type="email" {...email.bind} />
      <input type="password" {...password.bind} />
      <button type="submit">Login</button>
    </form>
  );
}
```

---

## Event Handling

### Basic Event Handling
```jsx
function Button() {
  const handleClick = () => {
    alert('Button clicked!');
  };

  return <button onClick={handleClick}>Click me</button>;
}
```

### Event Object
```jsx
function Form() {
  const handleChange = (e) => {
    console.log(e.target.value);
  };

  return <input onChange={handleChange} />;
}
```

### Common Events

```jsx
function EventExamples() {
  return (
    <div>
      <button onClick={() => console.log('click')}>Click</button>
      <input onChange={(e) => console.log(e.target.value)} />
      <input onBlur={(e) => console.log('blur')} />
      <div onMouseEnter={() => console.log('enter')}>Hover</div>
      <input onKeyDown={(e) => console.log(e.key)} />
      <form onSubmit={(e) => e.preventDefault()}>Form</form>
    </div>
  );
}
```

### Event Delegation
React handles events efficiently through event delegation.

```jsx
function List() {
  const handleClick = (e) => {
    if (e.target.tagName === 'BUTTON') {
      console.log('Button clicked:', e.target.textContent);
    }
  };

  return (
    <div onClick={handleClick}>
      <button>Item 1</button>
      <button>Item 2</button>
      <button>Item 3</button>
    </div>
  );
}
```

---

## Conditional Rendering

### if/else Statement
```jsx
function Greeting({ isLoggedIn }) {
  if (isLoggedIn) {
    return <h1>Welcome back!</h1>;
  } else {
    return <h1>Please sign in.</h1>;
  }
}
```

### Ternary Operator
```jsx
function Greeting({ isLoggedIn }) {
  return isLoggedIn ? <h1>Welcome!</h1> : <h1>Sign in</h1>;
}
```

### Logical && Operator
```jsx
function Alert({ showAlert, message }) {
  return (
    <>
      {showAlert && <div className="alert">{message}</div>}
    </>
  );
}
```

### Switch Statement
```jsx
function Component({ status }) {
  switch (status) {
    case 'loading':
      return <p>Loading...</p>;
    case 'success':
      return <p>Success!</p>;
    case 'error':
      return <p>Error occurred</p>;
    default:
      return <p>Unknown status</p>;
  }
}
```

---

## Lists & Keys

### Rendering Lists
```jsx
function TodoList() {
  const todos = ['Learn React', 'Build projects', 'Master state'];

  return (
    <ul>
      {todos.map((todo, index) => (
        <li key={index}>{todo}</li>
      ))}
    </ul>
  );
}
```

### Key Prop Importance
Keys help React identify which items have changed. Always use a unique, stable identifier.

```jsx
// Bad: Using index as key (if list can be reordered)
{todos.map((todo, index) => <li key={index}>{todo}</li>)}

// Good: Using unique ID
{todos.map((todo) => <li key={todo.id}>{todo.text}</li>)}
```

### Why Keys Matter
- Maintains component state when list reorders
- Improves performance
- Prevents bugs with form inputs and component state

```jsx
function List({ items }) {
  return (
    <ul>
      {items.map((item) => (
        <li key={item.id}>
          <input type="checkbox" />
          {item.name}
        </li>
      ))}
    </ul>
  );
}
```

---

## Forms

### Controlled Components
Form inputs are controlled by React state.

```jsx
function LoginForm() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');

  const handleSubmit = (e) => {
    e.preventDefault();
    console.log({ email, password });
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        placeholder="Email"
      />
      <input
        type="password"
        value={password}
        onChange={(e) => setPassword(e.target.value)}
        placeholder="Password"
      />
      <button type="submit">Login</button>
    </form>
  );
}
```

### Uncontrolled Components
Form data is managed by the DOM itself.

```jsx
function Form() {
  const emailRef = useRef(null);

  const handleSubmit = (e) => {
    e.preventDefault();
    console.log(emailRef.current.value);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input type="email" ref={emailRef} placeholder="Email" />
      <button type="submit">Submit</button>
    </form>
  );
}
```

### Multiple Input Fields
```jsx
function Form() {
  const [formData, setFormData] = useState({
    name: '',
    email: '',
    age: '',
  });

  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormData({
      ...formData,
      [name]: value,
    });
  };

  const handleSubmit = (e) => {
    e.preventDefault();
    console.log(formData);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="text"
        name="name"
        value={formData.name}
        onChange={handleChange}
        placeholder="Name"
      />
      <input
        type="email"
        name="email"
        value={formData.email}
        onChange={handleChange}
        placeholder="Email"
      />
      <input
        type="number"
        name="age"
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

## Lifecycle Methods

### Class Component Lifecycle

```jsx
class Component extends React.Component {
  constructor(props) {
    super(props);
    this.state = { count: 0 };
  }

  // Called before render
  componentWillMount() {}

  // Called after render (mounting)
  componentDidMount() {
    // API calls, subscriptions
  }

  // Called before update
  componentWillReceiveProps(nextProps) {}

  // Determines if component should update
  shouldComponentUpdate(nextProps, nextState) {
    return true;
  }

  // Called after update
  componentDidUpdate(prevProps, prevState) {
    // More API calls, comparisons
  }

  // Called before unmounting
  componentWillUnmount() {
    // Cleanup: unsubscribe, timers
  }

  render() {
    return <div>{this.state.count}</div>;
  }
}
```

### Functional Component Lifecycle (with Hooks)

```jsx
function Component() {
  // Mount & Update
  useEffect(() => {
    console.log('After render');

    return () => {
      console.log('Before unmount or re-effect');
    };
  }, []); // Mount only

  // Did Update
  useEffect(() => {
    console.log('After every render');
  });

  // Component Did Update with deps
  useEffect(() => {
    console.log('After render or dependency change');
  }, [dependency]);

  return <div>Component</div>;
}
```

---

## Context API

### Creating and Using Context

```jsx
// Create context
const ThemeContext = React.createContext('light');

// Provider component
function App() {
  const [theme, setTheme] = useState('light');

  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      <Header />
      <Main />
      <Footer />
    </ThemeContext.Provider>
  );
}

// Consumer component
function Header() {
  const { theme, setTheme } = useContext(ThemeContext);

  return (
    <header className={theme}>
      <button onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}>
        Toggle Theme
      </button>
    </header>
  );
}
```

### Multiple Contexts
```jsx
const ThemeContext = React.createContext();
const AuthContext = React.createContext();

function App() {
  const [theme, setTheme] = useState('light');
  const [user, setUser] = useState(null);

  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      <AuthContext.Provider value={{ user, setUser }}>
        <MainComponent />
      </AuthContext.Provider>
    </ThemeContext.Provider>
  );
}

function MainComponent() {
  const theme = useContext(ThemeContext);
  const auth = useContext(AuthContext);

  return <div>Theme: {theme.theme}, User: {auth.user}</div>;
}
```

---

## Performance Optimization

### React.memo
Memoize components to prevent unnecessary re-renders when props haven't changed.

```jsx
const Button = React.memo(({ onClick, label }) => {
  console.log('Button re-rendered');
  return <button onClick={onClick}>{label}</button>;
});

// With custom comparison
const Button = React.memo(
  ({ onClick, label }) => <button onClick={onClick}>{label}</button>,
  (prevProps, nextProps) => {
    return prevProps.label === nextProps.label;
  }
);
```

### useMemo
Memoize expensive computations.

```jsx
function List({ items }) {
  const sortedItems = useMemo(() => {
    return items.sort((a, b) => a.name.localeCompare(b.name));
  }, [items]);

  return <ul>{sortedItems.map(item => <li key={item.id}>{item.name}</li>)}</ul>;
}
```

### useCallback
Memoize function references to prevent child re-renders.

```jsx
function Parent() {
  const [count, setCount] = useState(0);

  const handleClick = useCallback(() => {
    setCount(count + 1);
  }, [count]);

  return <Button onClick={handleClick} />;
}
```

### Code Splitting
Lazy load components with React.lazy and Suspense.

```jsx
import React, { Suspense } from 'react';

const HeavyComponent = React.lazy(() => import('./HeavyComponent'));

function App() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <HeavyComponent />
    </Suspense>
  );
}
```

### shouldComponentUpdate (Class Components)
```jsx
class Component extends React.Component {
  shouldComponentUpdate(nextProps, nextState) {
    return nextProps.id !== this.props.id;
  }

  render() {
    return <div>{this.props.id}</div>;
  }
}
```

---

## Common Patterns

### Higher-Order Component (HOC)
A function that takes a component and returns an enhanced component.

```jsx
function withLogger(Component) {
  return (props) => {
    useEffect(() => {
      console.log(`${Component.name} mounted`);
      return () => console.log(`${Component.name} unmounted`);
    }, []);

    return <Component {...props} />;
  };
}

const LoggedButton = withLogger(Button);
```

### Render Props
A function prop that returns JSX.

```jsx
function Mouse({ children }) {
  const [position, setPosition] = useState({ x: 0, y: 0 });

  const handleMouseMove = (e) => {
    setPosition({ x: e.clientX, y: e.clientY });
  };

  return (
    <div onMouseMove={handleMouseMove}>
      {children(position)}
    </div>
  );
}

function App() {
  return (
    <Mouse>
      {({ x, y }) => <p>Mouse position: {x}, {y}</p>}
    </Mouse>
  );
}
```

### Compound Components
Components that work together as a unit.

```jsx
function Tabs({ children }) {
  const [activeTab, setActiveTab] = useState(0);

  return (
    <div>
      {children.map((child, index) => (
        React.cloneElement(child, { activeTab, setActiveTab })
      ))}
    </div>
  );
}

function Tab({ label, children, activeTab, setActiveTab, index }) {
  return (
    <>
      <button onClick={() => setActiveTab(index)}>{label}</button>
      {activeTab === index && <div>{children}</div>}
    </>
  );
}

function App() {
  return (
    <Tabs>
      <Tab label="Tab 1">Content 1</Tab>
      <Tab label="Tab 2">Content 2</Tab>
    </Tabs>
  );
}
```

### Provider Pattern with Custom Hook
```jsx
const DataContext = React.createContext();

function DataProvider({ children }) {
  const [data, setData] = useState(null);

  useEffect(() => {
    // Fetch data
    setData({ name: 'John' });
  }, []);

  return (
    <DataContext.Provider value={{ data, setData }}>
      {children}
    </DataContext.Provider>
  );
}

function useData() {
  const context = useContext(DataContext);
  if (!context) {
    throw new Error('useData must be used within DataProvider');
  }
  return context;
}

function App() {
  return (
    <DataProvider>
      <Component />
    </DataProvider>
  );
}

function Component() {
  const { data } = useData();
  return <p>{data?.name}</p>;
}
```

---

## Common Interview Questions

### 1. What is the Virtual DOM?
The Virtual DOM is a lightweight JavaScript representation of the actual DOM. React uses it to:
- Improve performance by batching updates
- Determine what changed using diffing algorithm
- Update only the changed parts in the real DOM

### 2. Explain the React Reconciliation Process
React compares the old Virtual DOM with the new one (diffing) and updates only the changed elements. This is more efficient than directly manipulating the DOM.

### 3. What is the difference between State and Props?
- **State**: Owned by the component, mutable, managed by setState/useState, changes trigger re-render
- **Props**: Passed from parent, immutable (from child perspective), read-only, changes trigger re-render

### 4. When would you use useEffect with an empty dependency array?
```jsx
useEffect(() => {
  // Runs once on mount
  console.log('Component mounted');

  return () => {
    // Cleanup on unmount
    console.log('Component unmounting');
  };
}, []); // Empty array = run only once
```

### 5. What is prop drilling? How do you avoid it?
Prop drilling is passing props through many intermediate components that don't need them. Avoid it by using:
- Context API
- Redux or state management libraries
- Higher-Order Components (HOC)

### 6. What is the key prop used for in lists?
Keys help React identify which items have changed, preventing bugs and improving performance. Use unique, stable identifiers:

```jsx
// Good
{items.map(item => <div key={item.id}>{item.name}</div>)}

// Bad (can cause issues with reordering)
{items.map((item, index) => <div key={index}>{item.name}</div>)}
```

### 7. Explain controlled vs uncontrolled components
- **Controlled**: Form data is managed by React state
- **Uncontrolled**: Form data is managed by the DOM

```jsx
// Controlled
<input value={state} onChange={(e) => setState(e.target.value)} />

// Uncontrolled
<input ref={inputRef} />
```

### 8. What are React Hooks and why were they introduced?
Hooks are functions that let you use state and lifecycle features in functional components. They were introduced to:
- Simplify state management
- Replace class component complexity
- Enable code reuse through custom hooks

### 9. Explain Suspense and lazy loading in React
Suspense allows you to handle code splitting and async data fetching gracefully:

```jsx
const LazyComponent = React.lazy(() => import('./LazyComponent'));

<Suspense fallback={<Loading />}>
  <LazyComponent />
</Suspense>
```

### 10. What is the difference between useCallback and useMemo?
- **useCallback**: Memoizes function references, prevents recreating the same function on every render
- **useMemo**: Memoizes computed values, prevents recalculating expensive computations

```jsx
const handleClick = useCallback(() => { /* ... */ }, [deps]);
const value = useMemo(() => expensiveComputation(), [deps]);
```

### 11. How does React handle events? Is it real DOM events?
React uses event delegation through a synthetic event system. Events are actually attached to the root element and bubbled appropriately. This provides:
- Cross-browser compatibility
- Better performance
- Consistent event handling

### 12. Explain the useReducer hook
useReducer is used for complex state logic with multiple related values. It takes a reducer function and initial state:

```jsx
const [state, dispatch] = useReducer(reducer, initialState);
dispatch({ type: 'ACTION_TYPE', payload: value });
```

### 13. What are fragments and when would you use them?
Fragments let you group elements without adding extra nodes to the DOM:

```jsx
// Using Fragment tag
<React.Fragment>
  <div>Hello</div>
  <div>World</div>
</React.Fragment>

// Using shorthand
<>
  <div>Hello</div>
  <div>World</div>
</>
```

### 14. How do you handle errors in React?
Use Error Boundaries (class components only):

```jsx
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true };
  }

  componentDidCatch(error, errorInfo) {
    console.log(error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return <h1>Something went wrong</h1>;
    }
    return this.props.children;
  }
}
```

### 15. What is the purpose of the dependency array in useEffect?
The dependency array tells React when to run the effect:
- **No array**: Run after every render
- **Empty array []**: Run once on mount
- **[dep1, dep2]**: Run when any dependency changes

```jsx
useEffect(() => {
  // Runs only when id changes
  fetchData(id);
}, [id]);
```

---

## Summary

### Key Takeaways for Interviews

1. **Understand the basics**: Components, JSX, Props, State
2. **Know Hooks**: useState, useEffect, useContext are most important
3. **Performance matters**: Know memo, useMemo, useCallback
4. **Unidirectional data flow**: Parents pass props down, children emit events up
5. **Component composition**: Break UI into small, reusable components
6. **Side effects**: Use useEffect for API calls, subscriptions, cleanup
7. **Conditional rendering**: Know when to use ternary vs && operator
8. **Forms**: Understand controlled vs uncontrolled components
9. **Keys in lists**: Always use unique, stable IDs
10. **Context vs State Management**: Use Context for simple cases, Redux/Zustand for complex apps

### Important Rules to Remember

✓ React components must be pure functions (with props)
✓ Never mutate state directly
✓ Never call hooks conditionally
✓ Always provide a key to list items
✓ Handle side effects in useEffect
✓ Clean up subscriptions and timers in useEffect cleanup
✓ Don't call hooks inside loops or conditions
✓ Import React when using JSX (in older versions)

---

## Quick Reference

### Most Used Hooks
```
useState - Manage component state
useEffect - Handle side effects
useContext - Access context values
useReducer - Complex state management
useRef - Direct DOM access or mutable values
useCallback - Memoize functions
useMemo - Memoize computations
```

### Common Component Patterns
```
Functional Component (modern)
Class Component (legacy)
Higher-Order Component (HOC)
Render Props
Custom Hooks
Context Provider
```

### Common Props
```
children - Content passed to component
className - CSS class names
key - List item identifier
ref - Direct element access
onClick, onChange, onSubmit, etc. - Event handlers
```

---

Good luck with your React interviews! Master these fundamentals and you'll be well-prepared.
