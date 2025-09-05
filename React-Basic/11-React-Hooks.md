# React Hooks - A Complete Guide

## Introduction to React Hooks

React Hooks are functions that allow you to use state and other React features in functional components. Introduced in React 16.8, hooks revolutionized how we write React applications by eliminating the need for class components in most cases. This comprehensive guide will cover all the essential hooks and show you how to create custom hooks for maximum code reusability.

## Why Hooks Matter

Before hooks, functional components were stateless and couldn't access lifecycle methods. Class components were required for any complex logic. Hooks changed this by providing:

1. **Simpler Code**: Functional components with hooks are often more concise than class components
2. **Better Logic Reuse**: Custom hooks allow sharing stateful logic between components
3. **Easier Testing**: Functional components are generally easier to test
4. **Better Performance**: Hooks can lead to better optimization opportunities
5. **Cleaner Architecture**: Hooks encourage better separation of concerns

## Rules of Hooks

Before diving into specific hooks, it's crucial to understand the rules that govern their usage:

### Rule 1: Only Call Hooks at the Top Level

Never call hooks inside loops, conditions, or nested functions. Always use hooks at the top level of your React function.

```jsx
// ❌ Wrong - hooks inside conditions
function BadExample({ shouldUseHook }) {
  if (shouldUseHook) {
    const [count, setCount] = useState(0); // This breaks the rules!
  }
  
  return <div>Bad example</div>;
}

// ✅ Correct - hooks at top level
function GoodExample({ shouldUseHook }) {
  const [count, setCount] = useState(0);
  
  if (shouldUseHook) {
    // Use the hook value here instead
  }
  
  return <div>Good example</div>;
}
```

### Rule 2: Only Call Hooks from React Functions

Only call hooks from:
- React functional components
- Custom hooks

```jsx
// ❌ Wrong - calling hook from regular function
function regularFunction() {
  const [state, setState] = useState(0); // This breaks the rules!
}

// ✅ Correct - calling hook from React component
function ReactComponent() {
  const [state, setState] = useState(0);
  return <div>{state}</div>;
}

// ✅ Correct - calling hook from custom hook
function useCustomHook() {
  const [state, setState] = useState(0);
  return [state, setState];
}
```

## useState Hook

The `useState` hook allows you to add state to functional components.

### Basic Usage

```jsx
import React, { useState } from 'react';

function Counter() {
  // Declare a state variable called 'count' with initial value 0
  const [count, setCount] = useState(0);
  
  const increment = () => {
    setCount(count + 1);
  };
  
  const decrement = () => {
    setCount(count - 1);
  };
  
  const reset = () => {
    setCount(0);
  };
  
  return (
    <div>
      <h2>Count: {count}</h2>
      <button onClick={increment}>+</button>
      <button onClick={decrement}>-</button>
      <button onClick={reset}>Reset</button>
    </div>
  );
}
```

### Multiple State Variables

You can use multiple `useState` calls for different pieces of state:

```jsx
function UserProfile() {
  const [name, setName] = useState('');
  const [email, setEmail] = useState('');
  const [age, setAge] = useState(0);
  const [isEditing, setIsEditing] = useState(false);
  
  const handleSave = () => {
    // Save logic here
    setIsEditing(false);
  };
  
  const handleEdit = () => {
    setIsEditing(true);
  };
  
  return (
    <div>
      {isEditing ? (
        <div>
          <input
            type="text"
            value={name}
            onChange={(e) => setName(e.target.value)}
            placeholder="Name"
          />
          <input
            type="email"
            value={email}
            onChange={(e) => setEmail(e.target.value)}
            placeholder="Email"
          />
          <input
            type="number"
            value={age}
            onChange={(e) => setAge(parseInt(e.target.value))}
            placeholder="Age"
          />
          <button onClick={handleSave}>Save</button>
        </div>
      ) : (
        <div>
          <h3>{name}</h3>
          <p>Email: {email}</p>
          <p>Age: {age}</p>
          <button onClick={handleEdit}>Edit</button>
        </div>
      )}
    </div>
  );
}
```

### Object State

When your state is an object, you need to spread the previous state to maintain immutability:

```jsx
function ContactForm() {
  const [formData, setFormData] = useState({
    firstName: '',
    lastName: '',
    email: '',
    phone: ''
  });
  
  const handleInputChange = (field, value) => {
    setFormData(prevData => ({
      ...prevData,
      [field]: value
    }));
  };
  
  const handleSubmit = (e) => {
    e.preventDefault();
    console.log('Submitted:', formData);
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input
        type="text"
        placeholder="First Name"
        value={formData.firstName}
        onChange={(e) => handleInputChange('firstName', e.target.value)}
      />
      <input
        type="text"
        placeholder="Last Name"
        value={formData.lastName}
        onChange={(e) => handleInputChange('lastName', e.target.value)}
      />
      <input
        type="email"
        placeholder="Email"
        value={formData.email}
        onChange={(e) => handleInputChange('email', e.target.value)}
      />
      <input
        type="tel"
        placeholder="Phone"
        value={formData.phone}
        onChange={(e) => handleInputChange('phone', e.target.value)}
      />
      <button type="submit">Submit</button>
    </form>
  );
}
```

### Functional Updates

When the new state depends on the previous state, use functional updates:

```jsx
function CounterWithHistory() {
  const [count, setCount] = useState(0);
  const [history, setHistory] = useState([]);
  
  const increment = () => {
    setCount(prevCount => {
      const newCount = prevCount + 1;
      setHistory(prevHistory => [...prevHistory, `Incremented to ${newCount}`]);
      return newCount;
    });
  };
  
  const decrement = () => {
    setCount(prevCount => {
      const newCount = prevCount - 1;
      setHistory(prevHistory => [...prevHistory, `Decremented to ${newCount}`]);
      return newCount;
    });
  };
  
  return (
    <div>
      <h2>Count: {count}</h2>
      <button onClick={increment}>+</button>
      <button onClick={decrement}>-</button>
      
      <h3>History:</h3>
      <ul>
        {history.map((entry, index) => (
          <li key={index}>{entry}</li>
        ))}
      </ul>
    </div>
  );
}
```

### Lazy Initial State

For expensive initial state calculations, use lazy initialization:

```jsx
function ExpensiveComponent() {
  // This function only runs once, on the initial render
  const [data, setData] = useState(() => {
    console.log('Expensive calculation running...');
    return Array.from({ length: 1000 }, (_, i) => ({
      id: i,
      value: Math.random()
    }));
  });
  
  return (
    <div>
      <p>Data items: {data.length}</p>
      <button onClick={() => setData([])}>Clear Data</button>
    </div>
  );
}
```

## useEffect Hook

The `useEffect` hook lets you perform side effects in functional components. It combines the functionality of `componentDidMount`, `componentDidUpdate`, and `componentWillUnmount` from class components.

### Basic Usage

```jsx
import React, { useState, useEffect } from 'react';

function DocumentTitle() {
  const [count, setCount] = useState(0);
  
  // Effect runs after every render
  useEffect(() => {
    document.title = `Count: ${count}`;
  });
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>
        Increment
      </button>
    </div>
  );
}
```

### Effect with Dependencies

Use the dependency array to control when effects run:

```jsx
function UserData({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  // Effect runs only when userId changes
  useEffect(() => {
    if (!userId) return;
    
    setLoading(true);
    setError(null);
    
    // Simulate API call
    const fetchUser = async () => {
      try {
        const response = await fetch(`/api/users/${userId}`);
        if (!response.ok) {
          throw new Error('Failed to fetch user');
        }
        const userData = await response.json();
        setUser(userData);
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    };
    
    fetchUser();
  }, [userId]); // Only re-run when userId changes
  
  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;
  if (!user) return <div>No user found</div>;
  
  return (
    <div>
      <h2>{user.name}</h2>
      <p>Email: {user.email}</p>
    </div>
  );
}
```

### Effect Cleanup

Use cleanup to prevent memory leaks and cancel ongoing operations:

```jsx
function Timer() {
  const [seconds, setSeconds] = useState(0);
  const [isRunning, setIsRunning] = useState(false);
  
  useEffect(() => {
    let intervalId;
    
    if (isRunning) {
      intervalId = setInterval(() => {
        setSeconds(prevSeconds => prevSeconds + 1);
      }, 1000);
    }
    
    // Cleanup function
    return () => {
      if (intervalId) {
        clearInterval(intervalId);
      }
    };
  }, [isRunning]);
  
  const handleStart = () => setIsRunning(true);
  const handleStop = () => setIsRunning(false);
  const handleReset = () => {
    setIsRunning(false);
    setSeconds(0);
  };
  
  return (
    <div>
      <h2>Timer: {seconds}s</h2>
      <button onClick={handleStart} disabled={isRunning}>
        Start
      </button>
      <button onClick={handleStop} disabled={!isRunning}>
        Stop
      </button>
      <button onClick={handleReset}>Reset</button>
    </div>
  );
}
```

### Multiple Effects

You can use multiple `useEffect` hooks to separate concerns:

```jsx
function BlogPost({ postId }) {
  const [post, setPost] = useState(null);
  const [comments, setComments] = useState([]);
  const [viewCount, setViewCount] = useState(0);
  
  // Effect for fetching the post
  useEffect(() => {
    fetch(`/api/posts/${postId}`)
      .then(response => response.json())
      .then(setPost);
  }, [postId]);
  
  // Effect for fetching comments
  useEffect(() => {
    fetch(`/api/posts/${postId}/comments`)
      .then(response => response.json())
      .then(setComments);
  }, [postId]);
  
  // Effect for tracking view count
  useEffect(() => {
    const incrementViewCount = () => {
      fetch(`/api/posts/${postId}/view`, { method: 'POST' })
        .then(response => response.json())
        .then(data => setViewCount(data.viewCount));
    };
    
    incrementViewCount();
  }, [postId]);
  
  // Effect for setting up scroll tracking
  useEffect(() => {
    const handleScroll = () => {
      const scrollPercentage = (window.scrollY / (document.body.scrollHeight - window.innerHeight)) * 100;
      if (scrollPercentage > 80) {
        // Track that user read most of the post
        console.log('User read 80% of the post');
      }
    };
    
    window.addEventListener('scroll', handleScroll);
    
    return () => {
      window.removeEventListener('scroll', handleScroll);
    };
  }, []);
  
  if (!post) return <div>Loading post...</div>;
  
  return (
    <article>
      <h1>{post.title}</h1>
      <p>Views: {viewCount}</p>
      <div>{post.content}</div>
      
      <section>
        <h3>Comments ({comments.length})</h3>
        {comments.map(comment => (
          <div key={comment.id}>
            <strong>{comment.author}</strong>: {comment.text}
          </div>
        ))}
      </section>
    </article>
  );
}
```

## useContext Hook

The `useContext` hook provides a way to pass data through the component tree without having to pass props down manually at every level.

### Basic Context Usage

```jsx
import React, { createContext, useContext, useState } from 'react';

// Create a context
const ThemeContext = createContext();

// Theme provider component
function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');
  
  const toggleTheme = () => {
    setTheme(prevTheme => prevTheme === 'light' ? 'dark' : 'light');
  };
  
  const value = {
    theme,
    toggleTheme
  };
  
  return (
    <ThemeContext.Provider value={value}>
      {children}
    </ThemeContext.Provider>
  );
}

// Custom hook for using theme context
function useTheme() {
  const context = useContext(ThemeContext);
  
  if (!context) {
    throw new Error('useTheme must be used within a ThemeProvider');
  }
  
  return context;
}

// Component that uses the theme
function Header() {
  const { theme, toggleTheme } = useTheme();
  
  return (
    <header className={`header header--${theme}`}>
      <h1>My App</h1>
      <button onClick={toggleTheme}>
        Switch to {theme === 'light' ? 'dark' : 'light'} theme
      </button>
    </header>
  );
}

// Another component that uses the theme
function MainContent() {
  const { theme } = useTheme();
  
  return (
    <main className={`main main--${theme}`}>
      <p>This is the main content area.</p>
      <p>Current theme: {theme}</p>
    </main>
  );
}

// App component
function App() {
  return (
    <ThemeProvider>
      <div className="app">
        <Header />
        <MainContent />
      </div>
    </ThemeProvider>
  );
}
```

### Complex Context with Reducer

For more complex state management, combine `useContext` with `useReducer`:

```jsx
import React, { createContext, useContext, useReducer } from 'react';

// Action types
const ACTIONS = {
  ADD_ITEM: 'ADD_ITEM',
  REMOVE_ITEM: 'REMOVE_ITEM',
  UPDATE_QUANTITY: 'UPDATE_QUANTITY',
  CLEAR_CART: 'CLEAR_CART'
};

// Reducer function
function cartReducer(state, action) {
  switch (action.type) {
    case ACTIONS.ADD_ITEM:
      const existingItem = state.items.find(item => item.id === action.payload.id);
      
      if (existingItem) {
        return {
          ...state,
          items: state.items.map(item =>
            item.id === action.payload.id
              ? { ...item, quantity: item.quantity + 1 }
              : item
          )
        };
      }
      
      return {
        ...state,
        items: [...state.items, { ...action.payload, quantity: 1 }]
      };
      
    case ACTIONS.REMOVE_ITEM:
      return {
        ...state,
        items: state.items.filter(item => item.id !== action.payload)
      };
      
    case ACTIONS.UPDATE_QUANTITY:
      return {
        ...state,
        items: state.items.map(item =>
          item.id === action.payload.id
            ? { ...item, quantity: action.payload.quantity }
            : item
        )
      };
      
    case ACTIONS.CLEAR_CART:
      return {
        ...state,
        items: []
      };
      
    default:
      return state;
  }
}

// Create context
const CartContext = createContext();

// Cart provider
function CartProvider({ children }) {
  const [state, dispatch] = useReducer(cartReducer, {
    items: []
  });
  
  const addItem = (item) => {
    dispatch({ type: ACTIONS.ADD_ITEM, payload: item });
  };
  
  const removeItem = (id) => {
    dispatch({ type: ACTIONS.REMOVE_ITEM, payload: id });
  };
  
  const updateQuantity = (id, quantity) => {
    if (quantity <= 0) {
      removeItem(id);
    } else {
      dispatch({ type: ACTIONS.UPDATE_QUANTITY, payload: { id, quantity } });
    }
  };
  
  const clearCart = () => {
    dispatch({ type: ACTIONS.CLEAR_CART });
  };
  
  const getTotalItems = () => {
    return state.items.reduce((total, item) => total + item.quantity, 0);
  };
  
  const getTotalPrice = () => {
    return state.items.reduce((total, item) => total + (item.price * item.quantity), 0);
  };
  
  const value = {
    items: state.items,
    addItem,
    removeItem,
    updateQuantity,
    clearCart,
    getTotalItems,
    getTotalPrice
  };
  
  return (
    <CartContext.Provider value={value}>
      {children}
    </CartContext.Provider>
  );
}

// Custom hook
function useCart() {
  const context = useContext(CartContext);
  
  if (!context) {
    throw new Error('useCart must be used within a CartProvider');
  }
  
  return context;
}

// Product component
function Product({ product }) {
  const { addItem } = useCart();
  
  return (
    <div className="product">
      <h3>{product.name}</h3>
      <p>Price: ${product.price}</p>
      <button onClick={() => addItem(product)}>
        Add to Cart
      </button>
    </div>
  );
}

// Cart component
function Cart() {
  const { items, removeItem, updateQuantity, clearCart, getTotalItems, getTotalPrice } = useCart();
  
  if (items.length === 0) {
    return <div>Your cart is empty</div>;
  }
  
  return (
    <div className="cart">
      <h2>Shopping Cart ({getTotalItems()} items)</h2>
      
      {items.map(item => (
        <div key={item.id} className="cart-item">
          <span>{item.name}</span>
          <span>${item.price}</span>
          <input
            type="number"
            min="0"
            value={item.quantity}
            onChange={(e) => updateQuantity(item.id, parseInt(e.target.value))}
          />
          <button onClick={() => removeItem(item.id)}>Remove</button>
        </div>
      ))}
      
      <div className="cart-total">
        <strong>Total: ${getTotalPrice().toFixed(2)}</strong>
      </div>
      
      <button onClick={clearCart}>Clear Cart</button>
    </div>
  );
}
```

## useReducer Hook

The `useReducer` hook is an alternative to `useState` for managing complex state logic.

### Basic useReducer

```jsx
import React, { useReducer } from 'react';

// Reducer function
function counterReducer(state, action) {
  switch (action.type) {
    case 'increment':
      return { count: state.count + 1 };
    case 'decrement':
      return { count: state.count - 1 };
    case 'reset':
      return { count: 0 };
    case 'set':
      return { count: action.payload };
    default:
      throw new Error(`Unknown action type: ${action.type}`);
  }
}

function Counter() {
  // useReducer returns [state, dispatch]
  const [state, dispatch] = useReducer(counterReducer, { count: 0 });
  
  return (
    <div>
      <h2>Count: {state.count}</h2>
      <button onClick={() => dispatch({ type: 'increment' })}>
        +
      </button>
      <button onClick={() => dispatch({ type: 'decrement' })}>
        -
      </button>
      <button onClick={() => dispatch({ type: 'reset' })}>
        Reset
      </button>
      <button onClick={() => dispatch({ type: 'set', payload: 10 })}>
        Set to 10
      </button>
    </div>
  );
}
```

### Complex State with useReducer

```jsx
function todoReducer(state, action) {
  switch (action.type) {
    case 'ADD_TODO':
      return {
        ...state,
        todos: [
          ...state.todos,
          {
            id: Date.now(),
            text: action.payload,
            completed: false,
            createdAt: new Date().toISOString()
          }
        ]
      };
      
    case 'TOGGLE_TODO':
      return {
        ...state,
        todos: state.todos.map(todo =>
          todo.id === action.payload
            ? { ...todo, completed: !todo.completed }
            : todo
        )
      };
      
    case 'DELETE_TODO':
      return {
        ...state,
        todos: state.todos.filter(todo => todo.id !== action.payload)
      };
      
    case 'SET_FILTER':
      return {
        ...state,
        filter: action.payload
      };
      
    case 'CLEAR_COMPLETED':
      return {
        ...state,
        todos: state.todos.filter(todo => !todo.completed)
      };
      
    default:
      return state;
  }
}

function TodoApp() {
  const [state, dispatch] = useReducer(todoReducer, {
    todos: [],
    filter: 'all' // 'all', 'active', 'completed'
  });
  
  const [inputValue, setInputValue] = useState('');
  
  const handleSubmit = (e) => {
    e.preventDefault();
    if (inputValue.trim()) {
      dispatch({ type: 'ADD_TODO', payload: inputValue.trim() });
      setInputValue('');
    }
  };
  
  const filteredTodos = state.todos.filter(todo => {
    switch (state.filter) {
      case 'active':
        return !todo.completed;
      case 'completed':
        return todo.completed;
      default:
        return true;
    }
  });
  
  const completedCount = state.todos.filter(todo => todo.completed).length;
  const activeCount = state.todos.length - completedCount;
  
  return (
    <div>
      <h1>Todo App</h1>
      
      <form onSubmit={handleSubmit}>
        <input
          type="text"
          value={inputValue}
          onChange={(e) => setInputValue(e.target.value)}
          placeholder="Add a new todo..."
        />
        <button type="submit">Add</button>
      </form>
      
      <div className="stats">
        <span>Total: {state.todos.length}</span>
        <span>Active: {activeCount}</span>
        <span>Completed: {completedCount}</span>
      </div>
      
      <div className="filters">
        <button
          className={state.filter === 'all' ? 'active' : ''}
          onClick={() => dispatch({ type: 'SET_FILTER', payload: 'all' })}
        >
          All
        </button>
        <button
          className={state.filter === 'active' ? 'active' : ''}
          onClick={() => dispatch({ type: 'SET_FILTER', payload: 'active' })}
        >
          Active
        </button>
        <button
          className={state.filter === 'completed' ? 'active' : ''}
          onClick={() => dispatch({ type: 'SET_FILTER', payload: 'completed' })}
        >
          Completed
        </button>
      </div>
      
      <ul className="todo-list">
        {filteredTodos.map(todo => (
          <li key={todo.id} className={todo.completed ? 'completed' : ''}>
            <input
              type="checkbox"
              checked={todo.completed}
              onChange={() => dispatch({ type: 'TOGGLE_TODO', payload: todo.id })}
            />
            <span>{todo.text}</span>
            <button
              onClick={() => dispatch({ type: 'DELETE_TODO', payload: todo.id })}
            >
              Delete
            </button>
          </li>
        ))}
      </ul>
      
      {completedCount > 0 && (
        <button
          onClick={() => dispatch({ type: 'CLEAR_COMPLETED' })}
        >
          Clear Completed
        </button>
      )}
    </div>
  );
}
```

## Other Built-in Hooks

### useMemo

The `useMemo` hook memoizes expensive calculations:

```jsx
import React, { useState, useMemo } from 'react';

function ExpensiveCalculation({ items }) {
  const [multiplier, setMultiplier] = useState(1);
  
  // This expensive calculation only runs when items or multiplier changes
  const expensiveValue = useMemo(() => {
    console.log('Calculating expensive value...');
    return items.reduce((sum, item) => sum + item.value, 0) * multiplier;
  }, [items, multiplier]);
  
  return (
    <div>
      <p>Expensive calculation result: {expensiveValue}</p>
      <input
        type="number"
        value={multiplier}
        onChange={(e) => setMultiplier(Number(e.target.value))}
      />
    </div>
  );
}
```

### useCallback

The `useCallback` hook memoizes functions:

```jsx
import React, { useState, useCallback, memo } from 'react';

// Memoized child component
const ChildComponent = memo(({ onButtonClick, children }) => {
  console.log('ChildComponent rendered');
  return (
    <div>
      <button onClick={onButtonClick}>Click me</button>
      <p>{children}</p>
    </div>
  );
});

function Parent() {
  const [count, setCount] = useState(0);
  const [otherState, setOtherState] = useState(0);
  
  // Without useCallback, this function would be recreated on every render
  const handleButtonClick = useCallback(() => {
    setCount(prevCount => prevCount + 1);
  }, []); // Empty dependency array means this function never changes
  
  // This function depends on count, so it will be recreated when count changes
  const handleAnotherAction = useCallback(() => {
    console.log(`Current count is ${count}`);
  }, [count]);
  
  return (
    <div>
      <h2>Count: {count}</h2>
      <h2>Other state: {otherState}</h2>
      
      {/* This child won't re-render when otherState changes */}
      <ChildComponent onButtonClick={handleButtonClick}>
        Child content
      </ChildComponent>
      
      <button onClick={() => setOtherState(otherState + 1)}>
        Change other state
      </button>
      
      <button onClick={handleAnotherAction}>
        Log current count
      </button>
    </div>
  );
}
```

### useRef

The `useRef` hook creates a mutable reference that persists across renders:

```jsx
import React, { useRef, useEffect, useState } from 'react';

function FocusInput() {
  const inputRef = useRef(null);
  const [value, setValue] = useState('');
  
  const focusInput = () => {
    inputRef.current.focus();
  };
  
  // Focus the input when component mounts
  useEffect(() => {
    inputRef.current.focus();
  }, []);
  
  return (
    <div>
      <input
        ref={inputRef}
        type="text"
        value={value}
        onChange={(e) => setValue(e.target.value)}
        placeholder="Type something..."
      />
      <button onClick={focusInput}>Focus Input</button>
    </div>
  );
}

// useRef for storing previous values
function PreviousValue({ value }) {
  const prevValueRef = useRef();
  
  useEffect(() => {
    prevValueRef.current = value;
  });
  
  const prevValue = prevValueRef.current;
  
  return (
    <div>
      <p>Current value: {value}</p>
      <p>Previous value: {prevValue}</p>
    </div>
  );
}

// useRef for storing mutable values that don't trigger re-renders
function Timer() {
  const [count, setCount] = useState(0);
  const intervalRef = useRef(null);
  
  const startTimer = () => {
    if (intervalRef.current !== null) return;
    
    intervalRef.current = setInterval(() => {
      setCount(prevCount => prevCount + 1);
    }, 1000);
  };
  
  const stopTimer = () => {
    if (intervalRef.current !== null) {
      clearInterval(intervalRef.current);
      intervalRef.current = null;
    }
  };
  
  const resetTimer = () => {
    stopTimer();
    setCount(0);
  };
  
  // Cleanup on unmount
  useEffect(() => {
    return () => {
      if (intervalRef.current !== null) {
        clearInterval(intervalRef.current);
      }
    };
  }, []);
  
  return (
    <div>
      <h2>Timer: {count}</h2>
      <button onClick={startTimer}>Start</button>
      <button onClick={stopTimer}>Stop</button>
      <button onClick={resetTimer}>Reset</button>
    </div>
  );
}
```

## Custom Hooks

Custom hooks allow you to extract component logic into reusable functions.

### useLocalStorage Hook

```jsx
import { useState, useEffect } from 'react';

function useLocalStorage(key, initialValue) {
  // Get value from localStorage or use initial value
  const [storedValue, setStoredValue] = useState(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.error(`Error reading localStorage key "${key}":`, error);
      return initialValue;
    }
  });
  
  // Return a wrapped version of useState's setter function that persists the new value to localStorage
  const setValue = (value) => {
    try {
      // Allow value to be a function so we have the same API as useState
      const valueToStore = value instanceof Function ? value(storedValue) : value;
      setStoredValue(valueToStore);
      window.localStorage.setItem(key, JSON.stringify(valueToStore));
    } catch (error) {
      console.error(`Error setting localStorage key "${key}":`, error);
    }
  };
  
  return [storedValue, setValue];
}

// Usage example
function Settings() {
  const [theme, setTheme] = useLocalStorage('theme', 'light');
  const [language, setLanguage] = useLocalStorage('language', 'en');
  
  return (
    <div>
      <h2>Settings</h2>
      
      <div>
        <label>
          Theme:
          <select value={theme} onChange={(e) => setTheme(e.target.value)}>
            <option value="light">Light</option>
            <option value="dark">Dark</option>
          </select>
        </label>
      </div>
      
      <div>
        <label>
          Language:
          <select value={language} onChange={(e) => setLanguage(e.target.value)}>
            <option value="en">English</option>
            <option value="es">Spanish</option>
            <option value="fr">French</option>
          </select>
        </label>
      </div>
      
      <p>Current theme: {theme}</p>
      <p>Current language: {language}</p>
    </div>
  );
}
```

### useFetch Hook

```jsx
import { useState, useEffect } from 'react';

function useFetch(url, options = {}) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    const fetchData = async () => {
      try {
        setLoading(true);
        setError(null);
        
        const response = await fetch(url, options);
        
        if (!response.ok) {
          throw new Error(`HTTP error! status: ${response.status}`);
        }
        
        const result = await response.json();
        setData(result);
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    };
    
    fetchData();
  }, [url, JSON.stringify(options)]);
  
  return { data, loading, error };
}

// Usage example
function UserProfile({ userId }) {
  const { data: user, loading, error } = useFetch(`/api/users/${userId}`);
  
  if (loading) return <div>Loading user...</div>;
  if (error) return <div>Error: {error}</div>;
  if (!user) return <div>No user found</div>;
  
  return (
    <div>
      <h2>{user.name}</h2>
      <p>Email: {user.email}</p>
      <p>Location: {user.location}</p>
    </div>
  );
}
```

### useToggle Hook

```jsx
import { useState, useCallback } from 'react';

function useToggle(initialValue = false) {
  const [value, setValue] = useState(initialValue);
  
  const toggle = useCallback(() => {
    setValue(prev => !prev);
  }, []);
  
  const setTrue = useCallback(() => {
    setValue(true);
  }, []);
  
  const setFalse = useCallback(() => {
    setValue(false);
  }, []);
  
  return [value, toggle, setTrue, setFalse];
}

// Usage example
function ToggleExample() {
  const [isModalOpen, toggleModal, openModal, closeModal] = useToggle(false);
  const [isMenuOpen, toggleMenu] = useToggle(false);
  
  return (
    <div>
      <button onClick={toggleModal}>
        {isModalOpen ? 'Close' : 'Open'} Modal
      </button>
      
      <button onClick={toggleMenu}>
        {isMenuOpen ? 'Close' : 'Open'} Menu
      </button>
      
      {isModalOpen && (
        <div className="modal">
          <p>This is a modal</p>
          <button onClick={closeModal}>Close</button>
        </div>
      )}
      
      {isMenuOpen && (
        <nav className="menu">
          <ul>
            <li>Home</li>
            <li>About</li>
            <li>Contact</li>
          </ul>
        </nav>
      )}
    </div>
  );
}
```

### useDebounce Hook

```jsx
import { useState, useEffect } from 'react';

function useDebounce(value, delay) {
  const [debouncedValue, setDebouncedValue] = useState(value);
  
  useEffect(() => {
    const handler = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);
    
    return () => {
      clearTimeout(handler);
    };
  }, [value, delay]);
  
  return debouncedValue;
}

// Usage example
function SearchComponent() {
  const [searchTerm, setSearchTerm] = useState('');
  const [results, setResults] = useState([]);
  const [loading, setLoading] = useState(false);
  
  const debouncedSearchTerm = useDebounce(searchTerm, 300);
  
  useEffect(() => {
    if (debouncedSearchTerm) {
      setLoading(true);
      
      // Simulate API call
      setTimeout(() => {
        const mockResults = [
          `Result 1 for "${debouncedSearchTerm}"`,
          `Result 2 for "${debouncedSearchTerm}"`,
          `Result 3 for "${debouncedSearchTerm}"`
        ];
        setResults(mockResults);
        setLoading(false);
      }, 500);
    } else {
      setResults([]);
    }
  }, [debouncedSearchTerm]);
  
  return (
    <div>
      <input
        type="text"
        placeholder="Search..."
        value={searchTerm}
        onChange={(e) => setSearchTerm(e.target.value)}
      />
      
      {loading && <p>Searching...</p>}
      
      <ul>
        {results.map((result, index) => (
          <li key={index}>{result}</li>
        ))}
      </ul>
    </div>
  );
}
```

### useWindowSize Hook

```jsx
import { useState, useEffect } from 'react';

function useWindowSize() {
  const [windowSize, setWindowSize] = useState({
    width: undefined,
    height: undefined,
  });
  
  useEffect(() => {
    function handleResize() {
      setWindowSize({
        width: window.innerWidth,
        height: window.innerHeight,
      });
    }
    
    // Set initial size
    handleResize();
    
    window.addEventListener('resize', handleResize);
    
    return () => window.removeEventListener('resize', handleResize);
  }, []);
  
  return windowSize;
}

// Usage example
function ResponsiveComponent() {
  const { width, height } = useWindowSize();
  
  const isMobile = width < 768;
  const isTablet = width >= 768 && width < 1024;
  const isDesktop = width >= 1024;
  
  return (
    <div>
      <h2>Window Size: {width} x {height}</h2>
      
      <p>Device type: {
        isMobile ? 'Mobile' : 
        isTablet ? 'Tablet' : 
        isDesktop ? 'Desktop' : 'Unknown'
      }</p>
      
      {isMobile && <MobileComponent />}
      {isTablet && <TabletComponent />}
      {isDesktop && <DesktopComponent />}
    </div>
  );
}

function MobileComponent() {
  return <div>Mobile-specific content</div>;
}

function TabletComponent() {
  return <div>Tablet-specific content</div>;
}

function DesktopComponent() {
  return <div>Desktop-specific content</div>;
}
```

## Hook Best Practices

### 1. Keep Hooks at the Top Level

Always call hooks at the top level of your React function:

```jsx
// ✅ Good
function MyComponent({ condition }) {
  const [state, setState] = useState(0);
  const [otherState, setOtherState] = useState('');
  
  if (condition) {
    // Use the state here
  }
  
  return <div>{state}</div>;
}

// ❌ Bad
function MyComponent({ condition }) {
  if (condition) {
    const [state, setState] = useState(0); // This breaks the rules!
  }
  
  return <div>Content</div>;
}
```

### 2. Use Custom Hooks for Reusable Logic

Extract common patterns into custom hooks:

```jsx
// Instead of repeating this pattern
function ComponentA() {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    fetch('/api/data-a')
      .then(response => response.json())
      .then(setData)
      .catch(setError)
      .finally(() => setLoading(false));
  }, []);
  
  // ... rest of component
}

// Create a reusable hook
function useApiData(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    fetch(url)
      .then(response => response.json())
      .then(setData)
      .catch(setError)
      .finally(() => setLoading(false));
  }, [url]);
  
  return { data, loading, error };
}

// Use it in components
function ComponentA() {
  const { data, loading, error } = useApiData('/api/data-a');
  // ... rest of component
}
```

### 3. Optimize with useMemo and useCallback

Use memoization hooks to prevent unnecessary re-renders:

```jsx
function OptimizedComponent({ items, onItemClick }) {
  const [filter, setFilter] = useState('');
  
  // Memoize expensive calculations
  const filteredItems = useMemo(() => {
    return items.filter(item => 
      item.name.toLowerCase().includes(filter.toLowerCase())
    );
  }, [items, filter]);
  
  // Memoize callback functions
  const handleItemClick = useCallback((id) => {
    onItemClick(id);
  }, [onItemClick]);
  
  return (
    <div>
      <input
        value={filter}
        onChange={(e) => setFilter(e.target.value)}
        placeholder="Filter items..."
      />
      
      {filteredItems.map(item => (
        <Item
          key={item.id}
          item={item}
          onClick={handleItemClick}
        />
      ))}
    </div>
  );
}
```

### 4. Handle Cleanup Properly

Always clean up side effects to prevent memory leaks:

```jsx
function ComponentWithSubscription() {
  const [data, setData] = useState(null);
  
  useEffect(() => {
    const subscription = someDataSource.subscribe(setData);
    
    // Cleanup function
    return () => {
      subscription.unsubscribe();
    };
  }, []);
  
  useEffect(() => {
    const intervalId = setInterval(() => {
      console.log('Interval running');
    }, 1000);
    
    return () => {
      clearInterval(intervalId);
    };
  }, []);
  
  return <div>{data}</div>;
}
```

### 5. Use the Dependency Array Correctly

Be precise with your dependency arrays:

```jsx
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  
  useEffect(() => {
    fetchUser(userId).then(setUser);
  }, [userId]); // Only re-run when userId changes
  
  const handleUpdate = useCallback((newData) => {
    updateUser(userId, newData);
  }, [userId]); // Include all dependencies
  
  return <div>User profile content</div>;
}
```

## Summary

React Hooks have fundamentally changed how we write React applications by:

1. **Simplifying Components**: Functional components can now handle all the functionality that previously required class components
2. **Improving Code Reuse**: Custom hooks allow sharing stateful logic between components
3. **Better Performance**: Hooks like `useMemo` and `useCallback` help optimize rendering
4. **Cleaner Code**: Effects and state management are more organized and easier to understand

Key hooks to master:
- `useState` for component state
- `useEffect` for side effects and lifecycle methods
- `useContext` for consuming context
- `useReducer` for complex state management
- `useMemo` and `useCallback` for performance optimization
- `useRef` for DOM references and persistent values

By understanding and properly using these hooks, you can build more efficient, maintainable, and powerful React applications.

## Practice Exercise

Build a todo application that demonstrates multiple hooks:

1. Use `useState` for managing todos and filters
2. Use `useEffect` for persistence (localStorage)
3. Create custom hooks for localStorage and debounced search
4. Use `useMemo` for filtered todo lists
5. Use `useCallback` for optimized event handlers
6. Implement `useContext` for theme management

This exercise will help you apply all the hook concepts in a practical project.

## Additional Resources

- [React Hooks Official Documentation](https://react.dev/reference/react)
- [Rules of Hooks](https://react.dev/warnings/invalid-hook-call-warning)
- [Building Your Own Hooks](https://react.dev/learn/reusing-logic-with-custom-hooks)
- [Hook Patterns and Best Practices](https://blog.logrocket.com/react-hooks-cheat-sheet-unlock-solutions-to-common-problems-af4caf699e70/)
