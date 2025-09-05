# Functional Components

## What are Functional Components?

Functional components are JavaScript functions that return React elements. They're a simpler way to write components that only contain a render method and don't have their own state (until the introduction of hooks).

## Basic Syntax

A functional component is simply a JavaScript function that returns JSX:

```jsx
function Greeting() {
  return <h1>Hello, World!</h1>;
}
```

Or using arrow function syntax:

```jsx
const Greeting = () => {
  return <h1>Hello, World!</h1>;
};
```

With implicit return for one-line components:

```jsx
const Greeting = () => <h1>Hello, World!</h1>;
```

## Using Props in Functional Components

Props (short for properties) are passed to components as function arguments:

```jsx
function Greeting(props) {
  return <h1>Hello, {props.name}!</h1>;
}
```

You can use object destructuring for cleaner code:

```jsx
function Greeting({ name, role }) {
  return (
    <div>
      <h1>Hello, {name}!</h1>
      <p>Your role is: {role}</p>
    </div>
  );
}
```

## Using a Functional Component

To use a functional component, you include it in JSX as if it were an HTML tag:

```jsx
// Define the component
function App() {
  return (
    <div>
      <Greeting name="John" role="Developer" />
      <Greeting name="Sarah" role="Designer" />
    </div>
  );
}

// Render the component
root.render(<App />);
```

## Default Props

You can define default values for props:

```jsx
function Greeting({ name = "Guest", role = "Visitor" }) {
  return (
    <div>
      <h1>Hello, {name}!</h1>
      <p>Your role is: {role}</p>
    </div>
  );
}
```

Or using the defaultProps property:

```jsx
function Greeting(props) {
  return (
    <div>
      <h1>Hello, {props.name}!</h1>
      <p>Your role is: {props.role}</p>
    </div>
  );
}

Greeting.defaultProps = {
  name: "Guest",
  role: "Visitor"
};
```

## Children Props

The `children` prop is a special prop that contains any elements included between the opening and closing tags of a component:

```jsx
function Container({ children }) {
  return (
    <div className="container">
      {children}
    </div>
  );
}

function App() {
  return (
    <Container>
      <h1>This is a heading</h1>
      <p>This is a paragraph</p>
    </Container>
  );
}
```

## Functional Components vs. Class Components

### Functional Components:

```jsx
function Welcome({ name }) {
  return <h1>Hello, {name}!</h1>;
}
```

### Equivalent Class Component:

```jsx
class Welcome extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}!</h1>;
  }
}
```

### Advantages of Functional Components:

1. **Simpler syntax**: Less boilerplate code
2. **Easier to understand**: Just a function that returns JSX
3. **Better performance**: Slightly faster than class components
4. **Hooks support**: Modern React features are designed for functional components
5. **Easier to test**: Pure functions are easier to test

## Adding State to Functional Components with Hooks

Before React 16.8, functional components couldn't have state. Now with hooks, they can:

```jsx
import React, { useState } from 'react';

function Counter() {
  // Declare a state variable named "count" with initial value 0
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

## Converting a Functional Component to its React.createElement Form

When your JSX is transformed, functional components become:

**JSX Version**:
```jsx
function Greeting({ name }) {
  return <h1>Hello, {name}!</h1>;
}

function App() {
  return <Greeting name="World" />;
}
```

**Transformed Version**:
```javascript
function Greeting({ name }) {
  return React.createElement("h1", null, "Hello, ", name, "!");
}

function App() {
  return React.createElement(Greeting, { name: "World" });
}
```

Note how:
- JSX elements become `React.createElement()` calls
- Component usage becomes `React.createElement(ComponentName, props)`

## Understanding How React Processes Functional Components

When React encounters a component element like `<Greeting name="World" />`:

1. React calls the `Greeting` function with `{ name: "World" }` as the props
2. The `Greeting` function returns a React element (from JSX)
3. React processes the returned element
4. The result is added to the DOM

## Best Practices for Functional Components

1. **Use destructuring for props**: Makes the code cleaner and shows which props are used
2. **Keep components small and focused**: Each component should do one thing well
3. **Use meaningful component names**: Always use PascalCase names
4. **Make components pure when possible**: Avoid side effects
5. **Extract complex logic**: Move complex operations outside the component body
6. **Use composition**: Build larger components from smaller ones

In the next section, we'll explore how to compose components and elements together in more complex ways.
