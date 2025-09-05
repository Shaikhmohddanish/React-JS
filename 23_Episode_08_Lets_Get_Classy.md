# Episode 08 - Let's Get Classy

In this episode, we explore Class Components in React. While modern React development primarily uses Functional Components with Hooks, understanding Class Components is important for working with legacy code and grasping React's evolution.

## Key Concepts Covered

1. Class Components vs Functional Components
2. Component Lifecycle Methods
3. State Management in Class Components
4. Constructor and super(props)
5. Fetching Data in Class Components
6. Component Lifecycle Phases
7. Lifecycle Method Order of Execution

## Class Components in React

### Creating a Basic Class Component

Unlike functional components, class components are ES6 classes that extend from `React.Component`:

```jsx
import React from 'react';

class UserClass extends React.Component {
  render() {
    return (
      <div className="user-card">
        <h2>Name: {this.props.name}</h2>
        <h3>Location: {this.props.location}</h3>
        <h4>Contact: @vaasuk24</h4>
      </div>
    );
  }
}

export default UserClass;
```

### Constructor and Props

The constructor is where you initialize the component's state and bind methods:

```jsx
constructor(props) {
  super(props);
  
  this.state = {
    userInfo: {
      name: 'Dummy',
      location: 'Default',
    }
  };
}
```

### State Management in Class Components

In class components, state is managed through `this.state` and `this.setState()`:

```jsx
// Initializing state in constructor
this.state = {
  count: 0,
  count2: 2,
};

// Updating state
this.setState({
  count: this.state.count + 1,
});
```

Unlike functional components where each state variable has its own setter function, class components have a single state object that can be partially updated with `setState()`.

### Why super(props)?

The `super(props)` call in the constructor is necessary because:

1. When you extend a class in JavaScript, you need to call `super()` in the constructor before using `this`
2. Passing `props` to `super()` ensures that `this.props` is properly initialized in the constructor
3. Without it, `this.props` would be undefined in the constructor (though React would set it for other methods)

```jsx
constructor(props) {
  super(props); // This is necessary
  console.log(this.props); // Now this works correctly
}
```

## Component Lifecycle Methods

React class components have several lifecycle methods that are called at different stages of a component's existence.

### Mounting Phase

1. **constructor()** - Called when a component is initialized
2. **render()** - Called to generate the component's UI
3. **componentDidMount()** - Called after the component is added to the DOM

### Updating Phase

1. **render()** - Called when state or props change
2. **componentDidUpdate(prevProps, prevState)** - Called after the update is applied to the DOM

### Unmounting Phase

1. **componentWillUnmount()** - Called right before the component is removed from the DOM

### Order of Execution

When parent and child components are rendered together, the order of lifecycle method execution is:

```
1. Parent Constructor
2. Parent Render
   3. Child Constructor
   4. Child Render
   5. Child ComponentDidMount
6. Parent ComponentDidMount
```

This was illustrated in our About component:

```jsx
class About extends Component {
  constructor(props) {
    super(props);
    console.log('Parent Constructor');
  }

  componentDidMount() {
    console.log('Parent Component Did Mount');
  }

  render() {
    console.log('Parent Render');
    return (
      <div className="about-page">
        <UserClass name={'First'} location={'Badvel class'} />
      </div>
    );
  }
}
```

With multiple children, the order becomes:

```
1. Parent Constructor
2. Parent Render
   3. First Child Constructor
   4. First Child Render
   5. Second Child Constructor
   6. Second Child Render
   7. First Child ComponentDidMount
   8. Second Child ComponentDidMount
9. Parent ComponentDidMount
```

This batching of renders and updates is an optimization technique in React's reconciliation process.

## Fetching Data in Class Components

In functional components, we use the `useEffect` hook to fetch data. In class components, we use `componentDidMount`:

```jsx
async componentDidMount() {
  // API call
  const data = await fetch(
    'https://api.github.com/users/Shaikhmohddanish'
  );
  const json = await data.json();

  this.setState({
    userInfo: json,
  });
}
```

## Why Use componentDidMount for API Calls?

There are several reasons to fetch data in `componentDidMount`:

1. It runs after the component has been rendered, ensuring there's something visible to the user even before data loads
2. It only runs once after the initial render, which is ideal for API calls
3. It guarantees that the component is fully mounted, so any DOM manipulation will work correctly
4. It's the right place to set up subscriptions or timers

## Why Use componentWillUnmount?

The `componentWillUnmount` method is called right before a component is removed from the DOM. It's the perfect place for cleanup tasks:

```jsx
componentWillUnmount() {
  console.log('Component Will Unmount');
  // Clear any timers, cancel network requests, or clean up subscriptions
  clearInterval(this.timer);
}
```

Examples of cleanup tasks:
1. Clearing intervals or timeouts
2. Cancelling network requests
3. Removing event listeners
4. Unsubscribing from external services

### Example with Interval

Creating an interval without clearing it can cause memory leaks:

```jsx
componentDidMount() {
  this.timer = setInterval(() => {
    console.log("Timer running");
  }, 1000);
}

componentWillUnmount() {
  clearInterval(this.timer);
}
```

## Complete Lifecycle of a Class Component

Here's the complete flow of a class component:

### Mounting Phase
```
Constructor (with initial state)
Render (with initial/dummy data)
[React updates DOM]
ComponentDidMount
  - API calls
  - setState (triggers update cycle)
```

### Updating Phase
```
Render (with updated data)
[React updates DOM]
ComponentDidUpdate
```

### Unmounting Phase
```
ComponentWillUnmount
  - Cleanup
```

## Why Can't useEffect Callback Be Async?

In functional components, we can't make the `useEffect` callback directly async:

```jsx
// This is NOT allowed
useEffect(async () => {
  const data = await fetch('api/data');
  // ...
}, []);
```

This is because:

1. The `useEffect` callback should either return nothing or a cleanup function
2. An async function always returns a Promise, not a cleanup function
3. React wouldn't know how to handle this Promise

The solution is to define and call an async function inside useEffect:

```jsx
// This is correct
useEffect(() => {
  const fetchData = async () => {
    const data = await fetch('api/data');
    // ...
  };
  
  fetchData();
  
  // Can still return a cleanup function
  return () => {
    // cleanup code
  };
}, []);
```

## Comparing Class and Functional Components

### Class Component

```jsx
class UserClass extends React.Component {
  constructor(props) {
    super(props);
    this.state = { userInfo: { name: 'Dummy', location: 'Default' } };
  }

  async componentDidMount() {
    const data = await fetch('https://api.github.com/users/Shaikhmohddanish');
    const json = await data.json();
    this.setState({ userInfo: json });
  }

  render() {
    const { name, location, avatar_url } = this.state.userInfo;
    
    return (
      <div className="user-card">
        <img src={avatar_url} alt={name} />
        <h2>Name: {name}</h2>
        <h3>Location: {location}</h3>
      </div>
    );
  }
}
```

### Functional Component (Equivalent)

```jsx
const User = () => {
  const [userInfo, setUserInfo] = useState({
    name: 'Dummy',
    location: 'Default',
  });

  useEffect(() => {
    const fetchData = async () => {
      const data = await fetch('https://api.github.com/users/Shaikhmohddanish');
      const json = await data.json();
      setUserInfo(json);
    };
    
    fetchData();
  }, []);

  const { name, location, avatar_url } = userInfo;
  
  return (
    <div className="user-card">
      <img src={avatar_url} alt={name} />
      <h2>Name: {name}</h2>
      <h3>Location: {location}</h3>
    </div>
  );
};
```

## Lifecycle Methods and Hooks Mapping

| Class Component Lifecycle | Functional Component Hook |
|---------------------------|---------------------------|
| constructor | useState |
| render | Component function body |
| componentDidMount | useEffect(() => {}, []) |
| componentDidUpdate | useEffect(() => {}, [dependency]) |
| componentWillUnmount | useEffect(() => { return cleanup }, []) |
| shouldComponentUpdate | React.memo |
| getDerivedStateFromProps | No direct equivalent, use hooks pattern |
| getSnapshotBeforeUpdate | No direct equivalent |

## Summary

In this episode, we:

1. Created and used class components in React
2. Learned about component lifecycle methods and their execution order
3. Implemented state management in class components
4. Fetched data from an API in the componentDidMount method
5. Understood the importance of cleanup in componentWillUnmount
6. Compared class and functional components
7. Explored the reasons behind React's design decisions

While functional components with hooks are now the preferred approach for new React code, understanding class components remains valuable for maintaining existing codebases and appreciating React's evolution.
