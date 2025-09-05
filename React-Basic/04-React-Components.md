# React Components

Components are the building blocks of any React application. They are reusable, self-contained pieces of code that return markup. In this guide, we'll explore everything you need to know about React components as a beginner.

## What Are Components and Why Use Them?

A component in React is a JavaScript function or class that:
- Accepts inputs called "props" (short for properties)
- Returns React elements describing what should appear on the screen

### Why Components Matter

1. **Reusability**: Build components once and use them in multiple places
2. **Separation of Concerns**: Each component handles a specific part of the UI
3. **Maintainability**: Changes to one component don't affect others
4. **Testability**: Individual components can be tested in isolation
5. **Composition**: Complex UIs can be built by combining simpler components

Think of components like LEGO pieces - small, modular blocks that you can combine to build complex structures.

## Types of React Components

React supports two types of components:

1. **Functional Components** (also called Function Components)
2. **Class Components**

Since React 16.8 introduced Hooks, functional components have become the preferred way to write React components. However, understanding both types is important, especially when working with existing codebases.

### Functional Components

Functional components are JavaScript functions that accept props as an argument and return React elements.

#### Syntax:

```jsx
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}
```

#### Using ES6 Arrow Functions:

```jsx
const Welcome = (props) => {
  return <h1>Hello, {props.name}</h1>;
};
```

#### With Destructuring:

```jsx
const Welcome = ({ name }) => {
  return <h1>Hello, {name}</h1>;
};
```

#### One-Line Arrow Function (Implicit Return):

```jsx
const Welcome = ({ name }) => <h1>Hello, {name}</h1>;
```

#### Why Use Functional Components?

- Simpler and more concise syntax
- Easier to understand and test
- Better performance in most cases
- Access to Hooks (useState, useEffect, etc.)
- Recommended by the React team for new code

### Class Components

Class components are ES6 classes that extend from `React.Component` and implement a render method.

#### Syntax:

```jsx
import React, { Component } from 'react';

class Welcome extends Component {
  render() {
    return <h1>Hello, {this.props.name}</h1>;
  }
}
```

#### Class Components with State:

```jsx
import React, { Component } from 'react';

class Counter extends Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0
    };
  }
  
  increment = () => {
    this.setState({ count: this.state.count + 1 });
  }
  
  render() {
    return (
      <div>
        <p>Count: {this.state.count}</p>
        <button onClick={this.increment}>Increment</button>
      </div>
    );
  }
}
```

#### Why Use Class Components?

- Necessary for older React codebases
- Required before Hooks were introduced
- Still have some niche use cases for error boundaries
- More familiar to developers coming from OOP backgrounds

### When to Use Each Type

**Use Functional Components When:**
- You're writing new React code
- You need to use React Hooks
- Your component doesn't need complex state management
- You want simpler, more readable code

**Use Class Components When:**
- You're working with older React code
- You need to use lifecycle methods that don't have Hook equivalents
- You need to implement error boundaries

## Creating Your First Component

Let's create a simple component that displays a greeting:

```jsx
// Greeting.js
import React from 'react';

function Greeting() {
  return (
    <div className="greeting">
      <h1>Hello, World!</h1>
      <p>Welcome to React!</p>
    </div>
  );
}

export default Greeting;
```

To use this component in another file:

```jsx
// App.js
import React from 'react';
import Greeting from './Greeting';

function App() {
  return (
    <div className="app">
      <Greeting />
    </div>
  );
}

export default App;
```

Let's break down what's happening:

1. We create a function called `Greeting` that returns JSX
2. We export this function as the default export from the file
3. We import the `Greeting` component in our `App.js` file
4. We use the component as a custom HTML-like tag: `<Greeting />`

## Props: Passing Data to Components

Props (short for properties) are inputs that allow you to pass data from parent components to child components. They work like HTML attributes but can pass any JavaScript value: strings, numbers, arrays, objects, functions, and even other React components.

### Passing Props

```jsx
// Parent component
function App() {
  return (
    <div className="app">
      <Greeting name="Alice" age={25} isAdmin={true} />
    </div>
  );
}
```

### Receiving Props in Functional Components

```jsx
// Child component
function Greeting(props) {
  return (
    <div className="greeting">
      <h1>Hello, {props.name}!</h1>
      <p>You are {props.age} years old.</p>
      {props.isAdmin && <p>You have admin privileges.</p>}
    </div>
  );
}
```

### Receiving Props in Class Components

```jsx
// Child component (class-based)
class Greeting extends React.Component {
  render() {
    return (
      <div className="greeting">
        <h1>Hello, {this.props.name}!</h1>
        <p>You are {this.props.age} years old.</p>
        {this.props.isAdmin && <p>You have admin privileges.</p>}
      </div>
    );
  }
}
```

### Destructuring Props

For cleaner code, you can destructure props:

```jsx
// With destructuring
function Greeting({ name, age, isAdmin }) {
  return (
    <div className="greeting">
      <h1>Hello, {name}!</h1>
      <p>You are {age} years old.</p>
      {isAdmin && <p>You have admin privileges.</p>}
    </div>
  );
}
```

### Default Props

You can specify default values for props in case they're not provided:

```jsx
function Greeting({ name = "Guest", age = 0, isAdmin = false }) {
  return (
    <div className="greeting">
      <h1>Hello, {name}!</h1>
      <p>You are {age} years old.</p>
      {isAdmin && <p>You have admin privileges.</p>}
    </div>
  );
}
```

Alternatively, you can use the `defaultProps` property:

```jsx
function Greeting(props) {
  return (
    <div className="greeting">
      <h1>Hello, {props.name}!</h1>
      <p>You are {props.age} years old.</p>
      {props.isAdmin && <p>You have admin privileges.</p>}
    </div>
  );
}

Greeting.defaultProps = {
  name: "Guest",
  age: 0,
  isAdmin: false
};
```

### Props are Read-Only

In React, props are immutable (read-only). A component must never modify its own props. This ensures that components behave predictably. If you need to modify data, use state instead (covered in a later section).

```jsx
// Incorrect - never modify props
function Greeting(props) {
  props.name = "Changed"; // This is wrong!
  return <h1>Hello, {props.name}</h1>;
}

// Correct - treat props as read-only
function Greeting(props) {
  return <h1>Hello, {props.name}</h1>;
}
```

## Component Composition

Component composition is a fundamental concept in React. It refers to the pattern of combining smaller, more focused components to build complex UIs.

### Parent-Child Composition

The most common form of composition is parent-child, where one component renders other components:

```jsx
function App() {
  return (
    <div className="app">
      <Header />
      <MainContent />
      <Footer />
    </div>
  );
}

function Header() {
  return <header>This is the header</header>;
}

function MainContent() {
  return (
    <main>
      <h1>Main Content</h1>
      <p>This is the main content of the page.</p>
    </main>
  );
}

function Footer() {
  return <footer>This is the footer</footer>;
}
```

### Containment with children Prop

Sometimes, you want a component to act as a container for other components without knowing what those components will be in advance. You can use the special `children` prop for this:

```jsx
function Card({ title, children }) {
  return (
    <div className="card">
      <h2>{title}</h2>
      <div className="card-content">
        {children}
      </div>
    </div>
  );
}

function App() {
  return (
    <div className="app">
      <Card title="Welcome">
        <p>This is some content inside the card.</p>
        <button>Click me</button>
      </Card>
      
      <Card title="Features">
        <ul>
          <li>Feature 1</li>
          <li>Feature 2</li>
          <li>Feature 3</li>
        </ul>
      </Card>
    </div>
  );
}
```

In the example above, the `Card` component doesn't know what children it will receive. The content inside the `<Card>` tags is passed as the `children` prop, allowing for flexible composition.

### Specialized Components

You can create specialized versions of components by composing them with specific props:

```jsx
function Button({ children, color, size, onClick }) {
  return (
    <button 
      className={`btn btn-${color} btn-${size}`} 
      onClick={onClick}
    >
      {children}
    </button>
  );
}

// Create specialized button components
function PrimaryButton(props) {
  return <Button color="primary" {...props} />;
}

function DangerButton(props) {
  return <Button color="danger" {...props} />;
}

function SmallButton(props) {
  return <Button size="small" {...props} />;
}

// Usage
function App() {
  return (
    <div>
      <PrimaryButton onClick={() => console.log('Saved')}>Save</PrimaryButton>
      <DangerButton onClick={() => console.log('Deleted')}>Delete</DangerButton>
      <SmallButton color="success">Small Success</SmallButton>
    </div>
  );
}
```

## Component File Structure

Organizing your components is crucial for maintainability as your application grows. Here are some common approaches:

### 1. Component Per File

The most common approach is to have one component per file:

```
src/
├── components/
│   ├── App.js
│   ├── Header.js
│   ├── MainContent.js
│   ├── Footer.js
│   └── Button.js
```

### 2. Feature-Based Organization

For larger applications, organize components by feature:

```
src/
├── features/
│   ├── authentication/
│   │   ├── LoginForm.js
│   │   ├── SignupForm.js
│   │   └── AuthContext.js
│   ├── products/
│   │   ├── ProductList.js
│   │   ├── ProductDetail.js
│   │   └── ProductFilter.js
│   └── cart/
│       ├── CartItem.js
│       ├── CartSummary.js
│       └── CheckoutButton.js
├── components/
│   ├── common/
│   │   ├── Button.js
│   │   ├── Card.js
│   │   └── Modal.js
│   └── layout/
│       ├── Header.js
│       ├── Footer.js
│       └── Sidebar.js
```

### 3. Component with Related Files

For components with associated styles, tests, or other files:

```
src/
├── components/
│   ├── Button/
│   │   ├── index.js         # The component
│   │   ├── Button.css       # Styles
│   │   └── Button.test.js   # Tests
│   ├── Header/
│   │   ├── index.js
│   │   ├── Header.css
│   │   └── Header.test.js
```

Using an `index.js` file allows you to import the component without specifying the filename:

```jsx
// Both work the same way
import Button from './components/Button';
import Button from './components/Button/index.js';
```

### Best Practices

1. **Keep components small and focused**: Each component should do one thing well
2. **Use descriptive names**: Names should clearly indicate what the component does
3. **Export one component per file**: Avoid exporting multiple components from a single file (except for tightly related components)
4. **Co-locate related files**: Keep styles, tests, and other related files close to the component they belong to
5. **Use index.js for clean imports**: Export components from an index.js file for cleaner imports

## Component Lifecycle

Components go through different phases during their existence: mounting (creation), updating, and unmounting (removal). Understanding these phases is important for managing side effects and optimizing performance.

### Functional Components with Hooks

With the introduction of Hooks, functional components can handle lifecycle events:

```jsx
import React, { useState, useEffect } from 'react';

function Timer() {
  const [seconds, setSeconds] = useState(0);
  
  // Similar to componentDidMount and componentDidUpdate
  useEffect(() => {
    const interval = setInterval(() => {
      setSeconds(prev => prev + 1);
    }, 1000);
    
    // Similar to componentWillUnmount
    return () => {
      clearInterval(interval);
    };
  }, []); // Empty dependency array means run once on mount
  
  return <div>Seconds: {seconds}</div>;
}
```

### Class Component Lifecycle Methods

In class components, you can use specific lifecycle methods:

```jsx
import React, { Component } from 'react';

class Timer extends Component {
  constructor(props) {
    super(props);
    this.state = {
      seconds: 0
    };
  }
  
  // After component is inserted into the DOM
  componentDidMount() {
    this.interval = setInterval(() => {
      this.setState(prevState => ({
        seconds: prevState.seconds + 1
      }));
    }, 1000);
  }
  
  // Before component is removed from the DOM
  componentWillUnmount() {
    clearInterval(this.interval);
  }
  
  render() {
    return <div>Seconds: {this.state.seconds}</div>;
  }
}
```

## Common Component Patterns

### 1. Container and Presentational Components

This pattern separates data handling from presentation:

```jsx
// Container component (handles data and logic)
function UserListContainer() {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    // Fetch data
    fetch('https://api.example.com/users')
      .then(response => response.json())
      .then(data => {
        setUsers(data);
        setLoading(false);
      });
  }, []);
  
  return <UserList users={users} loading={loading} />;
}

// Presentational component (handles presentation)
function UserList({ users, loading }) {
  if (loading) {
    return <div>Loading...</div>;
  }
  
  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```

### 2. Higher-Order Components (HOCs)

HOCs are functions that take a component and return a new enhanced component:

```jsx
// HOC that adds loading functionality
function withLoading(Component) {
  return function WithLoadingComponent({ isLoading, ...props }) {
    if (isLoading) {
      return <div>Loading...</div>;
    }
    return <Component {...props} />;
  };
}

// Usage
const UserListWithLoading = withLoading(UserList);

function App() {
  const [loading, setLoading] = useState(true);
  const [users, setUsers] = useState([]);
  
  useEffect(() => {
    fetch('https://api.example.com/users')
      .then(response => response.json())
      .then(data => {
        setUsers(data);
        setLoading(false);
      });
  }, []);
  
  return <UserListWithLoading isLoading={loading} users={users} />;
}
```

### 3. Render Props

The render prop pattern involves passing a function as a prop that returns React elements:

```jsx
// Component with render prop
function DataFetcher({ url, render }) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    fetch(url)
      .then(response => response.json())
      .then(data => {
        setData(data);
        setLoading(false);
      })
      .catch(error => {
        setError(error);
        setLoading(false);
      });
  }, [url]);
  
  return render({ data, loading, error });
}

// Usage
function App() {
  return (
    <DataFetcher 
      url="https://api.example.com/users"
      render={({ data, loading, error }) => {
        if (loading) return <div>Loading...</div>;
        if (error) return <div>Error: {error.message}</div>;
        return (
          <ul>
            {data.map(user => (
              <li key={user.id}>{user.name}</li>
            ))}
          </ul>
        );
      }}
    />
  );
}
```

### 4. Custom Hooks

Hooks are a modern way to share logic between components:

```jsx
// Custom hook for data fetching
function useDataFetching(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    const fetchData = async () => {
      try {
        const response = await fetch(url);
        const json = await response.json();
        setData(json);
        setLoading(false);
      } catch (error) {
        setError(error);
        setLoading(false);
      }
    };
    
    fetchData();
  }, [url]);
  
  return { data, loading, error };
}

// Usage
function UserList({ url }) {
  const { data, loading, error } = useDataFetching(url);
  
  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;
  
  return (
    <ul>
      {data.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```

## Conditional Rendering in Components

Often you'll want to render different UI based on certain conditions:

### Using if Statements

```jsx
function LoginStatus({ isLoggedIn }) {
  if (isLoggedIn) {
    return <div>Welcome back!</div>;
  }
  return <div>Please log in</div>;
}
```

### Using Ternary Operators

```jsx
function LoginStatus({ isLoggedIn }) {
  return (
    <div>
      {isLoggedIn ? 'Welcome back!' : 'Please log in'}
    </div>
  );
}
```

### Using Logical && Operator

```jsx
function Notifications({ messages }) {
  return (
    <div>
      {messages.length > 0 && (
        <p>You have {messages.length} unread messages</p>
      )}
    </div>
  );
}
```

### Using Variables to Store Elements

```jsx
function LoginControl({ isLoggedIn }) {
  let button;
  
  if (isLoggedIn) {
    button = <LogoutButton />;
  } else {
    button = <LoginButton />;
  }
  
  return (
    <div>
      <div>{isLoggedIn ? 'Welcome back!' : 'Please log in'}</div>
      {button}
    </div>
  );
}
```

## Error Handling in Components

Components can sometimes fail due to various reasons. React provides a way to catch these errors using Error Boundaries:

```jsx
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }
  
  static getDerivedStateFromError(error) {
    // Update state so the next render will show the fallback UI
    return { hasError: true };
  }
  
  componentDidCatch(error, errorInfo) {
    // You can log the error to an error reporting service
    console.error("Error caught by boundary:", error, errorInfo);
  }
  
  render() {
    if (this.state.hasError) {
      // You can render any custom fallback UI
      return <h1>Something went wrong.</h1>;
    }
    
    return this.props.children;
  }
}

// Usage
function App() {
  return (
    <div>
      <ErrorBoundary>
        <ComponentThatMightError />
      </ErrorBoundary>
    </div>
  );
}
```

Note: Error boundaries only catch errors in the component tree below them. They don't catch errors in event handlers or asynchronous code.

## Performance Optimization

As your application grows, you might need to optimize performance. React provides several ways to do this:

### React.memo

`React.memo` is a higher-order component that memoizes the result of a component render, preventing unnecessary re-renders:

```jsx
const MemoizedComponent = React.memo(function MyComponent(props) {
  // Only re-renders if props change
  return <div>{props.value}</div>;
});
```

### useMemo Hook

The `useMemo` hook memoizes the result of a computation:

```jsx
function ExpensiveComponent({ data }) {
  // processData will only be called if data changes
  const processedData = useMemo(() => {
    return data.map(item => expensiveOperation(item));
  }, [data]);
  
  return <div>{processedData.length} items processed</div>;
}
```

### useCallback Hook

The `useCallback` hook memoizes a function, which is useful when passing callbacks to optimized child components:

```jsx
function ParentComponent() {
  const [count, setCount] = useState(0);
  
  // handleClick will only be recreated if count changes
  const handleClick = useCallback(() => {
    console.log(`Button clicked, count: ${count}`);
  }, [count]);
  
  return <ChildComponent onClick={handleClick} />;
}

// ChildComponent is wrapped in React.memo to prevent unnecessary re-renders
const ChildComponent = React.memo(function ChildComponent({ onClick }) {
  console.log("ChildComponent rendered");
  return <button onClick={onClick}>Click me</button>;
});
```

## Component Testing

Testing components is essential for ensuring reliability. Here's a basic example using Jest and React Testing Library:

```jsx
// Button.js
function Button({ onClick, children }) {
  return <button onClick={onClick}>{children}</button>;
}

// Button.test.js
import { render, screen, fireEvent } from '@testing-library/react';
import Button from './Button';

test('renders button with correct text', () => {
  render(<Button>Click me</Button>);
  expect(screen.getByText('Click me')).toBeInTheDocument();
});

test('calls onClick when clicked', () => {
  const handleClick = jest.fn();
  render(<Button onClick={handleClick}>Click me</Button>);
  fireEvent.click(screen.getByText('Click me'));
  expect(handleClick).toHaveBeenCalledTimes(1);
});
```

## Best Practices for Components

1. **Keep components small and focused**
   - Each component should have a single responsibility
   - If a component grows too large, break it down into smaller components

2. **Use functional components with hooks**
   - Prefer functional components over class components for new code
   - Use hooks for state and side effects

3. **Make components reusable**
   - Design components to be reusable across different parts of your application
   - Use props to customize behavior and appearance

4. **Follow naming conventions**
   - Use PascalCase for component names (e.g., `UserProfile`, not `userProfile`)
   - Use descriptive names that indicate what the component does

5. **Use prop types or TypeScript**
   - Define the expected props to catch bugs early
   - Consider using TypeScript for stronger type checking

6. **Keep state as local as possible**
   - Only lift state up when necessary
   - Prefer component state for UI state and context/redux for shared state

7. **Avoid deeply nested component trees**
   - Deeply nested components can make your application harder to understand
   - Consider using composition patterns to flatten the structure

8. **Write tests for your components**
   - Test component rendering, interactions, and edge cases
   - Use tools like React Testing Library to test components from a user's perspective

## Summary

Components are the foundation of React applications. They allow you to build complex UIs from small, reusable pieces. In this guide, we covered:

- The two types of components: functional and class components
- How to create and compose components
- Passing data via props
- Component file organization
- Component lifecycle and side effects
- Common component patterns
- Performance optimization
- Testing components

Understanding these concepts will help you build robust, maintainable React applications.

## Practice Exercise

Create a simple Todo application with the following components:

1. `TodoApp`: The main container component
2. `TodoForm`: For adding new todos
3. `TodoList`: For displaying the list of todos
4. `TodoItem`: For displaying a single todo item
5. `TodoFilter`: For filtering todos by status (all, active, completed)

Use the concepts you've learned:
- Pass data between components using props
- Use state to manage the todo list
- Implement event handlers for adding, toggling, and removing todos
- Use conditional rendering to show/hide elements based on the filter

## Additional Resources

- [Official React Components Documentation](https://react.dev/learn/your-first-component)
- [React Patterns on GitHub](https://reactpatterns.com/)
- [React Testing Library Documentation](https://testing-library.com/docs/react-testing-library/intro/)
- [React DevTools Browser Extension](https://chrome.google.com/webstore/detail/react-developer-tools/fmkadmapgofadopljbjfkapdkoienihi)
