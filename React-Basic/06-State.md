# State in React

## What is State?

State is one of the most important concepts in React. It's the data that a component manages internally, which can change over time in response to user actions, network responses, or anything else.

Unlike props, which are passed down from parent components and are read-only, state is managed within the component itself and can be changed. When state changes, React automatically re-renders the component to reflect those changes.

Think of state as the "memory" of a component - it's what allows your UI to be interactive and dynamic.

## Why State Matters

State is crucial for creating interactive applications because:

1. **Interactivity**: State allows components to respond to user input and change what's displayed
2. **Dynamic UI**: State enables the creation of dynamic user interfaces that update based on data changes
3. **Memory**: State acts as component memory, preserving information between renders
4. **User Experience**: State helps maintain the user's context during their interaction with your app

## Understanding State vs. Props

Before diving deeper into state, let's clarify the differences between state and props:

| State | Props |
|-------|-------|
| Internal to the component | Passed from parent components |
| Can be changed by the component | Read-only (cannot be modified) |
| Used for data that changes over time | Used for configuration and passing data down |
| Causes re-renders when changed | Causes re-renders when received new values from parent |
| Maintained by the component | Maintained by the parent component |

Understanding when to use state versus props is key to designing efficient React applications.

## Types of State Management in React

React offers several ways to manage state:

1. **Component State**: Local state managed within a single component
2. **Lifted State**: State lifted to a common ancestor component and shared via props
3. **Context API**: State shared across many components without prop drilling
4. **External State Management**: Libraries like Redux, MobX, or Recoil for complex state management

Let's explore each of these in detail, starting with component state.

## Component State in Functional Components (Hooks)

Since React 16.8, functional components can use state through the `useState` hook. This is now the recommended way to use state in React.

### Basic useState Example

```jsx
import React, { useState } from 'react';

function Counter() {
  // Declare a state variable called "count" with initial value 0
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

Let's break down what's happening:

1. We import the `useState` hook from React
2. We call `useState(0)` inside our component to declare a state variable:
   - `count` is the current state value
   - `setCount` is a function to update that value
   - `0` is the initial state value
3. When the button is clicked, we call `setCount(count + 1)` to update the state
4. React re-renders the component with the new count value

### Why Hooks Are Important

Hooks revolutionized React by allowing functional components to:
- Use state (previously only possible in class components)
- Access lifecycle features
- Reuse stateful logic between components

This made functional components the preferred way to write React code.

### Multiple State Variables

You can declare multiple state variables in a component:

```jsx
function UserForm() {
  const [name, setName] = useState('');
  const [email, setEmail] = useState('');
  const [age, setAge] = useState(0);
  
  return (
    <form>
      <input
        type="text"
        placeholder="Name"
        value={name}
        onChange={(e) => setName(e.target.value)}
      />
      <input
        type="email"
        placeholder="Email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
      />
      <input
        type="number"
        placeholder="Age"
        value={age}
        onChange={(e) => setAge(Number(e.target.value))}
      />
      <div>
        <p>Name: {name}</p>
        <p>Email: {email}</p>
        <p>Age: {age}</p>
      </div>
    </form>
  );
}
```

### Using Object State

For related state values, you can use an object:

```jsx
function UserForm() {
  const [user, setUser] = useState({
    name: '',
    email: '',
    age: 0
  });
  
  const handleChange = (e) => {
    const { name, value } = e.target;
    setUser({
      ...user, // Important! Spread the existing state
      [name]: name === 'age' ? Number(value) : value
    });
  };
  
  return (
    <form>
      <input
        type="text"
        name="name"
        placeholder="Name"
        value={user.name}
        onChange={handleChange}
      />
      <input
        type="email"
        name="email"
        placeholder="Email"
        value={user.email}
        onChange={handleChange}
      />
      <input
        type="number"
        name="age"
        placeholder="Age"
        value={user.age}
        onChange={handleChange}
      />
      <div>
        <p>Name: {user.name}</p>
        <p>Email: {user.email}</p>
        <p>Age: {user.age}</p>
      </div>
    </form>
  );
}
```

Notice the use of the spread operator (`...user`) - this is crucial when updating object state in React. Unlike class components, the `useState` hook doesn't automatically merge object properties.

### Using Array State

Working with arrays in state is also common:

```jsx
function TodoList() {
  const [todos, setTodos] = useState([]);
  const [input, setInput] = useState('');
  
  const addTodo = () => {
    if (input.trim() === '') return;
    
    setTodos([...todos, {
      id: Date.now(),
      text: input,
      completed: false
    }]);
    setInput(''); // Clear input after adding
  };
  
  const toggleTodo = (id) => {
    setTodos(todos.map(todo => 
      todo.id === id ? { ...todo, completed: !todo.completed } : todo
    ));
  };
  
  const deleteTodo = (id) => {
    setTodos(todos.filter(todo => todo.id !== id));
  };
  
  return (
    <div>
      <div>
        <input
          value={input}
          onChange={(e) => setInput(e.target.value)}
          placeholder="Add a todo"
        />
        <button onClick={addTodo}>Add</button>
      </div>
      <ul>
        {todos.map(todo => (
          <li key={todo.id}>
            <span
              style={{ textDecoration: todo.completed ? 'line-through' : 'none' }}
              onClick={() => toggleTodo(todo.id)}
            >
              {todo.text}
            </span>
            <button onClick={() => deleteTodo(todo.id)}>Delete</button>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

Key patterns for array state manipulation:

1. **Adding an item**: `setArray([...array, newItem])`
2. **Removing an item**: `setArray(array.filter(item => item.id !== idToRemove))`
3. **Updating an item**: `setArray(array.map(item => item.id === idToUpdate ? { ...item, property: newValue } : item))`
4. **Reordering items**: Use `array.slice()` and spread operator to create a new array with the desired order

### Lazy Initial State

If computing the initial state is expensive, you can pass a function to `useState`:

```jsx
function ExpensiveInitialState() {
  // This function only runs during the initial render
  const [state, setState] = useState(() => {
    console.log('Computing initial state...');
    
    // Simulate expensive calculation
    let result = 0;
    for (let i = 0; i < 1000000; i++) {
      result += i;
    }
    
    return result;
  });
  
  return (
    <div>
      <p>Expensive calculation result: {state}</p>
      <button onClick={() => setState(state + 1)}>Increment</button>
    </div>
  );
}
```

The function passed to `useState` is only called during the initial render, not on re-renders.

### Functional Updates

When the new state depends on the previous state, it's better to use the functional update form:

```jsx
function Counter() {
  const [count, setCount] = useState(0);
  
  // Not ideal if setCount is called multiple times
  const incrementRegular = () => {
    setCount(count + 1);
  };
  
  // Better - uses functional update form
  const incrementFunctional = () => {
    setCount(prevCount => prevCount + 1);
  };
  
  // This will increment by 3 correctly
  const incrementThree = () => {
    incrementFunctional();
    incrementFunctional();
    incrementFunctional();
  };
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={incrementRegular}>Increment (Regular)</button>
      <button onClick={incrementFunctional}>Increment (Functional)</button>
      <button onClick={incrementThree}>Increment by 3</button>
    </div>
  );
}
```

The functional update form is necessary because state updates in React might be batched for performance reasons. Using a function guarantees you're working with the latest state.

## State in Class Components

Although functional components with hooks are now preferred, you may encounter class components in existing codebases. Here's how state works in class components:

```jsx
import React, { Component } from 'react';

class Counter extends Component {
  constructor(props) {
    super(props);
    // Initialize state in the constructor
    this.state = {
      count: 0
    };
  }
  
  increment = () => {
    // Use setState to update state
    this.setState({ count: this.state.count + 1 });
  };
  
  // Using previous state (functional update)
  incrementSafe = () => {
    this.setState(prevState => ({
      count: prevState.count + 1
    }));
  };
  
  render() {
    return (
      <div>
        <p>Count: {this.state.count}</p>
        <button onClick={this.increment}>Increment</button>
        <button onClick={this.incrementSafe}>Increment (Safe)</button>
      </div>
    );
  }
}
```

### Key Differences in Class Component State

1. **Initialization**: State is initialized in the constructor with `this.state = {...}`
2. **Updates**: State is updated with `this.setState()` method
3. **Merging**: `setState` automatically merges object properties (unlike `useState`)
4. **Asynchronous**: `setState` is asynchronous, which is why functional updates are important
5. **Single Object**: All state is stored in a single object

### Handling Asynchronous setState

Since `setState` is asynchronous in class components, you might need to use the callback form or the second callback parameter:

```jsx
class AsyncExample extends Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0
    };
  }
  
  // Use the function form for state that depends on previous state
  increment = () => {
    this.setState(prevState => ({
      count: prevState.count + 1
    }));
  };
  
  // Use the callback to run code after state is updated
  incrementAndLog = () => {
    this.setState(
      prevState => ({ count: prevState.count + 1 }),
      () => {
        // This callback runs after state is updated and component re-renders
        console.log('Updated count:', this.state.count);
      }
    );
  };
  
  render() {
    return (
      <div>
        <p>Count: {this.state.count}</p>
        <button onClick={this.increment}>Increment</button>
        <button onClick={this.incrementAndLog}>Increment and Log</button>
      </div>
    );
  }
}
```

## Lifting State Up

Often, you'll need to share state between components. The React way to do this is to "lift the state up" to their closest common ancestor.

### Example of Lifting State Up

Let's say we have a temperature calculator with two inputs: Celsius and Fahrenheit. When one input changes, the other should update accordingly.

```jsx
import React, { useState } from 'react';

// Child component for temperature input
function TemperatureInput({ scale, temperature, onTemperatureChange }) {
  const scaleNames = {
    c: 'Celsius',
    f: 'Fahrenheit'
  };
  
  return (
    <fieldset>
      <legend>Enter temperature in {scaleNames[scale]}:</legend>
      <input
        value={temperature}
        onChange={(e) => onTemperatureChange(e.target.value)}
      />
    </fieldset>
  );
}

// Parent component that manages the shared state
function Calculator() {
  const [temperature, setTemperature] = useState('');
  const [scale, setScale] = useState('c');
  
  const handleCelsiusChange = (temperature) => {
    setScale('c');
    setTemperature(temperature);
  };
  
  const handleFahrenheitChange = (temperature) => {
    setScale('f');
    setTemperature(temperature);
  };
  
  // Conversion functions
  const toCelsius = (fahrenheit) => {
    return (fahrenheit - 32) * 5 / 9;
  };
  
  const toFahrenheit = (celsius) => {
    return (celsius * 9 / 5) + 32;
  };
  
  const tryConvert = (temperature, convert) => {
    const input = parseFloat(temperature);
    if (Number.isNaN(input)) {
      return '';
    }
    const output = convert(input);
    const rounded = Math.round(output * 1000) / 1000;
    return rounded.toString();
  };
  
  const celsius = scale === 'f'
    ? tryConvert(temperature, toCelsius)
    : temperature;
    
  const fahrenheit = scale === 'c'
    ? tryConvert(temperature, toFahrenheit)
    : temperature;
  
  return (
    <div>
      <TemperatureInput
        scale="c"
        temperature={celsius}
        onTemperatureChange={handleCelsiusChange}
      />
      <TemperatureInput
        scale="f"
        temperature={fahrenheit}
        onTemperatureChange={handleFahrenheitChange}
      />
      <p>
        {parseFloat(celsius) >= 100
          ? 'The water would boil.'
          : 'The water would not boil.'}
      </p>
    </div>
  );
}
```

In this example:
1. The `Calculator` component manages the shared state (temperature and scale)
2. It passes the state down to child `TemperatureInput` components as props
3. When either input changes, it calls a callback function that updates the parent's state
4. The parent component recalculates both temperature values and passes them back down

This pattern, known as "lifting state up," allows related components to stay in sync.

## State Management with Context API

For more complex applications, passing props through many component layers (prop drilling) becomes cumbersome. React's Context API provides a way to share state across components without explicitly passing props.

### Creating and Using Context

```jsx
import React, { createContext, useState, useContext } from 'react';

// Create a context
const ThemeContext = createContext();

// Provider component that wraps the app
function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');
  
  const toggleTheme = () => {
    setTheme(prevTheme => prevTheme === 'light' ? 'dark' : 'light');
  };
  
  // The value prop contains the state and any functions we want to expose
  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

// Custom hook to use the theme context
function useTheme() {
  const context = useContext(ThemeContext);
  if (context === undefined) {
    throw new Error('useTheme must be used within a ThemeProvider');
  }
  return context;
}

// Components that consume the theme
function ThemedButton() {
  const { theme, toggleTheme } = useTheme();
  
  return (
    <button
      onClick={toggleTheme}
      style={{
        backgroundColor: theme === 'light' ? '#fff' : '#333',
        color: theme === 'light' ? '#333' : '#fff',
        border: '1px solid',
        padding: '8px 16px',
        borderRadius: '4px'
      }}
    >
      Toggle Theme
    </button>
  );
}

function ThemedHeader() {
  const { theme } = useTheme();
  
  return (
    <header
      style={{
        backgroundColor: theme === 'light' ? '#f0f0f0' : '#222',
        color: theme === 'light' ? '#333' : '#fff',
        padding: '16px'
      }}
    >
      <h1>Themed App</h1>
    </header>
  );
}

// App that uses the theme
function App() {
  return (
    <ThemeProvider>
      <div style={{ padding: '20px' }}>
        <ThemedHeader />
        <div style={{ marginTop: '20px' }}>
          <ThemedButton />
        </div>
      </div>
    </ThemeProvider>
  );
}
```

In this example:
1. We create a context with `createContext()`
2. We create a provider component that manages state and makes it available to children
3. We create a custom hook for easier context consumption
4. Child components can access the state and functions without props

Context is ideal for global state like:
- Theme (dark/light mode)
- User authentication
- Language/localization
- UI state that affects multiple components

## Advanced State Management with useReducer

For complex state logic, React provides the `useReducer` hook, which is similar to how Redux works:

```jsx
import React, { useReducer } from 'react';

// Define action types as constants to avoid typos
const ACTIONS = {
  INCREMENT: 'increment',
  DECREMENT: 'decrement',
  RESET: 'reset',
  SET_VALUE: 'set_value'
};

// Reducer function: takes current state and action, returns new state
function counterReducer(state, action) {
  switch (action.type) {
    case ACTIONS.INCREMENT:
      return { count: state.count + 1 };
    case ACTIONS.DECREMENT:
      return { count: state.count - 1 };
    case ACTIONS.RESET:
      return { count: 0 };
    case ACTIONS.SET_VALUE:
      return { count: action.payload };
    default:
      throw new Error(`Unhandled action type: ${action.type}`);
  }
}

function Counter() {
  // useReducer takes a reducer function and initial state
  const [state, dispatch] = useReducer(counterReducer, { count: 0 });
  
  return (
    <div>
      <h2>Count: {state.count}</h2>
      <button onClick={() => dispatch({ type: ACTIONS.INCREMENT })}>
        Increment
      </button>
      <button onClick={() => dispatch({ type: ACTIONS.DECREMENT })}>
        Decrement
      </button>
      <button onClick={() => dispatch({ type: ACTIONS.RESET })}>
        Reset
      </button>
      <button onClick={() => dispatch({ 
        type: ACTIONS.SET_VALUE, 
        payload: 10 
      })}>
        Set to 10
      </button>
    </div>
  );
}
```

### When to Use useReducer vs. useState

- **Use useState when**:
  - State is simple (primitives, simple objects)
  - State updates are straightforward
  - You have few state transitions
  - Components are small

- **Use useReducer when**:
  - State is complex (nested objects, arrays)
  - Next state depends on previous state
  - State transitions are numerous or complex
  - Related state transitions should be grouped together
  - You want to centralize state update logic

### Combining useReducer and Context

For more sophisticated state management, you can combine `useReducer` with Context:

```jsx
import React, { createContext, useReducer, useContext } from 'react';

// Action types
const ACTIONS = {
  ADD_TODO: 'add-todo',
  TOGGLE_TODO: 'toggle-todo',
  DELETE_TODO: 'delete-todo'
};

// Reducer function
function todosReducer(state, action) {
  switch (action.type) {
    case ACTIONS.ADD_TODO:
      return [...state, {
        id: Date.now(),
        text: action.payload,
        completed: false
      }];
    case ACTIONS.TOGGLE_TODO:
      return state.map(todo => 
        todo.id === action.payload
          ? { ...todo, completed: !todo.completed }
          : todo
      );
    case ACTIONS.DELETE_TODO:
      return state.filter(todo => todo.id !== action.payload);
    default:
      return state;
  }
}

// Create context
const TodosContext = createContext();

// Provider component
function TodosProvider({ children }) {
  const [todos, dispatch] = useReducer(todosReducer, []);
  
  return (
    <TodosContext.Provider value={{ todos, dispatch }}>
      {children}
    </TodosContext.Provider>
  );
}

// Custom hook to use the todos context
function useTodos() {
  const context = useContext(TodosContext);
  if (context === undefined) {
    throw new Error('useTodos must be used within a TodosProvider');
  }
  return context;
}

// Components
function TodoList() {
  const { todos, dispatch } = useTodos();
  
  return (
    <ul>
      {todos.map(todo => (
        <li key={todo.id}>
          <span
            style={{ 
              textDecoration: todo.completed ? 'line-through' : 'none',
              cursor: 'pointer'
            }}
            onClick={() => dispatch({ 
              type: ACTIONS.TOGGLE_TODO, 
              payload: todo.id 
            })}
          >
            {todo.text}
          </span>
          <button
            onClick={() => dispatch({ 
              type: ACTIONS.DELETE_TODO, 
              payload: todo.id 
            })}
          >
            Delete
          </button>
        </li>
      ))}
    </ul>
  );
}

function AddTodo() {
  const { dispatch } = useTodos();
  const [text, setText] = React.useState('');
  
  const handleSubmit = (e) => {
    e.preventDefault();
    if (text.trim()) {
      dispatch({ type: ACTIONS.ADD_TODO, payload: text });
      setText('');
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input
        value={text}
        onChange={(e) => setText(e.target.value)}
        placeholder="Add a todo"
      />
      <button type="submit">Add</button>
    </form>
  );
}

function TodoApp() {
  return (
    <TodosProvider>
      <h1>Todo List</h1>
      <AddTodo />
      <TodoList />
    </TodosProvider>
  );
}
```

This pattern provides a Redux-like state management solution but using only React's built-in features.

## Common State Management Pitfalls

### 1. Modifying State Directly

Never modify state directly - always use the state updater function:

```jsx
// WRONG
function Counter() {
  const [count, setCount] = useState(0);
  
  const increment = () => {
    count++; // This won't trigger a re-render
    console.log(count); // The value changes in the console but not in the UI
  };
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={increment}>Increment</button>
    </div>
  );
}

// RIGHT
function Counter() {
  const [count, setCount] = useState(0);
  
  const increment = () => {
    setCount(count + 1); // This triggers a re-render
  };
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={increment}>Increment</button>
    </div>
  );
}
```

### 2. Not Using Functional Updates When Needed

When the new state depends on the previous state, always use functional updates:

```jsx
// PROBLEMATIC
function Counter() {
  const [count, setCount] = useState(0);
  
  const increment = () => {
    // These might batch together and only increment once
    setCount(count + 1);
    setCount(count + 1);
  };
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={increment}>Increment</button>
    </div>
  );
}

// CORRECT
function Counter() {
  const [count, setCount] = useState(0);
  
  const increment = () => {
    // This will correctly increment twice
    setCount(prevCount => prevCount + 1);
    setCount(prevCount => prevCount + 1);
  };
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={increment}>Increment</button>
    </div>
  );
}
```

### 3. Object/Array Mutations in State

When working with objects or arrays in state, make sure to create new copies instead of mutating the originals:

```jsx
// WRONG - mutating array in state
function TodoList() {
  const [todos, setTodos] = useState([]);
  
  const addTodo = (text) => {
    todos.push({ id: Date.now(), text }); // Mutation!
    setTodos(todos); // Same reference, no re-render
  };
  
  // Other functions...
}

// RIGHT - creating new arrays
function TodoList() {
  const [todos, setTodos] = useState([]);
  
  const addTodo = (text) => {
    setTodos([...todos, { id: Date.now(), text }]); // New array
  };
  
  // Other functions...
}
```

For objects:

```jsx
// WRONG
function UserProfile() {
  const [user, setUser] = useState({
    name: 'John',
    email: 'john@example.com'
  });
  
  const updateEmail = (newEmail) => {
    user.email = newEmail; // Mutation!
    setUser(user); // Same reference, no re-render
  };
  
  // Rest of component...
}

// RIGHT
function UserProfile() {
  const [user, setUser] = useState({
    name: 'John',
    email: 'john@example.com'
  });
  
  const updateEmail = (newEmail) => {
    setUser({ ...user, email: newEmail }); // New object
  };
  
  // Rest of component...
}
```

### 4. Overusing State

Not everything needs to be in state. Derived values should be calculated during render:

```jsx
// UNNECESSARY STATE
function TodoList() {
  const [todos, setTodos] = useState([]);
  const [completedCount, setCompletedCount] = useState(0);
  
  const addTodo = (text) => {
    setTodos([...todos, { id: Date.now(), text, completed: false }]);
  };
  
  const toggleTodo = (id) => {
    const newTodos = todos.map(todo => 
      todo.id === id ? { ...todo, completed: !todo.completed } : todo
    );
    setTodos(newTodos);
    
    // Now we need to update completedCount too
    setCompletedCount(newTodos.filter(todo => todo.completed).length);
  };
  
  // Component rendering...
}

// BETTER APPROACH
function TodoList() {
  const [todos, setTodos] = useState([]);
  
  // Calculate this during render
  const completedCount = todos.filter(todo => todo.completed).length;
  
  const addTodo = (text) => {
    setTodos([...todos, { id: Date.now(), text, completed: false }]);
  };
  
  const toggleTodo = (id) => {
    setTodos(todos.map(todo => 
      todo.id === id ? { ...todo, completed: !todo.completed } : todo
    ));
    // No need to update completedCount manually
  };
  
  // Component rendering...
}
```

### 5. State Initialization Mistakes

Be careful with state initialization, especially with functions:

```jsx
// PROBLEMATIC - function runs on every render
function SearchResults() {
  // This expensive function runs on EVERY render
  const [results, setResults] = useState(fetchInitialResults());
  
  // Component code...
}

// BETTER - lazy initialization
function SearchResults() {
  // The function only runs once during initial render
  const [results, setResults] = useState(() => fetchInitialResults());
  
  // Component code...
}
```

## State Management Best Practices

1. **Keep state as local as possible**
   - Only lift state up when necessary
   - Prefer component state for UI-specific state

2. **Use the appropriate state tool for the job**
   - Simple state: `useState`
   - Complex state logic: `useReducer`
   - Shared state: Context API or external libraries

3. **Normalize complex state**
   - For collections of items, prefer objects with IDs as keys over arrays
   - This makes updates, lookups, and removals more efficient

   ```jsx
   // Instead of an array:
   const [todos, setTodos] = useState([
     { id: 1, text: 'Learn React', completed: false },
     { id: 2, text: 'Build app', completed: true }
   ]);
   
   // Consider an object:
   const [todos, setTodos] = useState({
     1: { id: 1, text: 'Learn React', completed: false },
     2: { id: 2, text: 'Build app', completed: true }
   });
   ```

4. **Use immutable update patterns**
   - Always create new objects/arrays when updating state
   - Consider using libraries like Immer for complex updates

5. **Split complex state**
   - Divide complex state into smaller, manageable pieces
   - Group related state with custom hooks

6. **Document your state structure**
   - Add comments explaining the shape of complex state
   - Use TypeScript for type-safe state

7. **Handle loading and error states**
   - Include loading and error flags in your state
   - Provide meaningful feedback to users

   ```jsx
   function DataFetcher() {
     const [state, setState] = useState({
       data: null,
       isLoading: true,
       error: null
     });
     
     // Fetch data and update state accordingly
   }
   ```

## Stateful vs. Stateless Components

Understanding the difference between stateful and stateless components helps in designing better component hierarchies:

### Stateful Components (Container Components)

- Manage and update state
- Handle business logic and data fetching
- Pass data down to stateless components
- Often higher in the component tree

Example:

```jsx
function UserDashboard() {
  const [user, setUser] = useState(null);
  const [isLoading, setIsLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    async function fetchUser() {
      try {
        setIsLoading(true);
        const response = await fetch('/api/user');
        const userData = await response.json();
        setUser(userData);
        setIsLoading(false);
      } catch (err) {
        setError(err);
        setIsLoading(false);
      }
    }
    
    fetchUser();
  }, []);
  
  if (isLoading) return <LoadingSpinner />;
  if (error) return <ErrorMessage error={error} />;
  if (!user) return <NotFound />;
  
  return <UserProfile user={user} />;
}
```

### Stateless Components (Presentational Components)

- Receive data via props
- Focus on presentation/UI
- Don't manage state (or minimal UI state)
- More reusable and easier to test
- Often lower in the component tree

Example:

```jsx
function UserProfile({ user }) {
  return (
    <div className="user-profile">
      <h2>{user.name}</h2>
      <p>Email: {user.email}</p>
      <div className="stats">
        <div className="stat">
          <span className="stat-value">{user.followers}</span>
          <span className="stat-label">Followers</span>
        </div>
        <div className="stat">
          <span className="stat-value">{user.following}</span>
          <span className="stat-label">Following</span>
        </div>
      </div>
    </div>
  );
}
```

This separation makes your code more maintainable and testable. Stateless components are easier to reason about because they're pure functions: given the same props, they always render the same UI.

## Third-Party State Management Libraries

While React's built-in state management is sufficient for many applications, larger projects might benefit from dedicated state management libraries:

### Redux

Redux provides predictable state management with a single store and unidirectional data flow:

```jsx
// Action types
const INCREMENT = 'INCREMENT';
const DECREMENT = 'DECREMENT';

// Action creators
function increment() {
  return { type: INCREMENT };
}

function decrement() {
  return { type: DECREMENT };
}

// Reducer
function counterReducer(state = { count: 0 }, action) {
  switch (action.type) {
    case INCREMENT:
      return { count: state.count + 1 };
    case DECREMENT:
      return { count: state.count - 1 };
    default:
      return state;
  }
}

// Store
const store = createStore(counterReducer);

// Component with Redux
function Counter() {
  const count = useSelector(state => state.count);
  const dispatch = useDispatch();
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => dispatch(increment())}>Increment</button>
      <button onClick={() => dispatch(decrement())}>Decrement</button>
    </div>
  );
}
```

### MobX

MobX uses observable state and automatic reactions:

```jsx
// Store with MobX
class CounterStore {
  @observable count = 0;
  
  @action increment() {
    this.count++;
  }
  
  @action decrement() {
    this.count--;
  }
}

const counterStore = new CounterStore();

// Component with MobX
const Counter = observer(() => {
  return (
    <div>
      <p>Count: {counterStore.count}</p>
      <button onClick={() => counterStore.increment()}>Increment</button>
      <button onClick={() => counterStore.decrement()}>Decrement</button>
    </div>
  );
});
```

### Recoil

Recoil is a state management library by Facebook that works with React's concurrent mode:

```jsx
// Define an atom (piece of state)
const countState = atom({
  key: 'countState',
  default: 0
});

// Component using Recoil
function Counter() {
  const [count, setCount] = useRecoilState(countState);
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
      <button onClick={() => setCount(count - 1)}>Decrement</button>
    </div>
  );
}
```

### Zustand

Zustand is a small, fast state management solution:

```jsx
// Create a store
const useCounterStore = create(set => ({
  count: 0,
  increment: () => set(state => ({ count: state.count + 1 })),
  decrement: () => set(state => ({ count: state.count - 1 }))
}));

// Component using Zustand
function Counter() {
  const { count, increment, decrement } = useCounterStore();
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={increment}>Increment</button>
      <button onClick={decrement}>Decrement</button>
    </div>
  );
}
```

## Summary

State is a fundamental concept in React that allows components to create interactive UIs. Key takeaways:

1. State represents data that changes over time and affects a component's rendering
2. Functional components use the `useState` and `useReducer` hooks for state management
3. Class components use `this.state` and `this.setState()`
4. Always treat state as immutable, creating new objects/arrays rather than mutating existing ones
5. Lift state up to common ancestors when multiple components need to share it
6. Use Context API for state that needs to be accessed by many components
7. Choose the right state management solution based on your application's complexity
8. Keep state as local as possible and derive values where you can

Understanding state management is crucial for building responsive and maintainable React applications.

## Practice Exercise

Build a shopping cart application with the following features:

1. Display a list of products with prices
2. Allow users to add items to a cart
3. Allow users to adjust quantities in the cart
4. Calculate the total price
5. Apply a discount code

Try implementing this using:
- Local component state with `useState`
- Lifted state to a parent component
- Context API for global cart state
- `useReducer` for complex cart operations

This exercise will help you understand the different approaches to state management and when to use each one.

## Additional Resources

- [React Official Documentation on State](https://react.dev/learn/managing-state)
- [React Hooks API Reference](https://react.dev/reference/react)
- [Redux Documentation](https://redux.js.org/)
- [MobX Documentation](https://mobx.js.org/README.html)
- [Recoil Documentation](https://recoiljs.org/)
- [Zustand Documentation](https://github.com/pmndrs/zustand)
