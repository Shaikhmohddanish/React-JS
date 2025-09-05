# React Components

Components are the building blocks of React applications. They let you split the UI into independent, reusable pieces, and think about each piece in isolation.

## Types of React Components

### 1. Functional Components

Also known as "stateless components" or "function components," these are defined using JavaScript functions.

```jsx
// Simple functional component with regular function syntax
const Title = function () {
  return <h1 className="heading">Hello React using JSX🚀</h1>;
};

// Using arrow function syntax with parentheses
const HeadingComponent = () => (
  <h1 className="heading">
    Hello React from Functional Component
  </h1>
);

// Using arrow function syntax with return statement
const HeadingComponent = () => {
  return (
    <div className="container">
      <h1>Hello React from Functional Component</h1>
    </div>
  );
};

// Header component example
const Header = () => {
  return (
    <div className="header">
      <div className="logo-container">
        <img
          src="https://png.pngtree.com/png-vector/20230217/ourmid/pngtree-food-logo-design-for-restaurant-and-business-png-image_6604922.png"
          alt="App Logo"
          className="logo"
        />
      </div>
      <div className="nav-items">
        <ul>
          <li>Home</li>
          <li>About Us</li>
          <li>Contact Us</li>
          <li>Cart</li>
        </ul>
      </div>
    </div>
  );
};
```

Functional components:
- Are simpler and easier to understand
- Were traditionally used for presentational components (before Hooks)
- With the introduction of Hooks, can now use state and other React features
- Have better performance in some cases

### 2. Class Components

Class components are ES6 classes that extend `React.Component`.

```jsx
import React from 'react';

class UserClass extends React.Component {
  constructor(props) {
    super(props);

    this.state = {
      userInfo: {
        name: 'Dummy',
        location: 'Default',
      },
    };
  }

  async componentDidMount() {
    // * API call
    const data = await fetch(
      'https://api.github.com/users/Shaikhmohddanish'
    );
    const json = await data.json();

    this.setState({
      userInfo: json,
    });

    console.log(json);
  }

  componentDidUpdate() {
    console.log('Component Did Update');
  }

  componentWillUnmount() {
    console.log('Component Will Unmount');
  }

  render() {
    const { name, location, avatar_url } = this.state.userInfo;

    return (
      <div className="user-card">
        <img src={avatar_url} alt={name} />
        <h2>Name: {name}</h2>
        <h3>Location: {location}</h3>
        <h4>Contact: @vaasuk24</h4>
      </div>
    );
  }
}

export default UserClass;

/* ****************************************************************
 *
 *
 * ----- Mounting CYCLE -----
 *   Constructor (dummy)
 *   Render (dummy)
 *       <HTML Dummy></HTML>
 *   Component Did Mount
 *       <API Call>
 *       <this.setState> - State variable is updated
 *
 * ----- UPDATE CYCLE -----
 *       render(API data)
 *       <HTML (new API data)>
 *   Component Did Update
 *   Component Will Unmount
 *
 *
 * Life Cycle Diagram Website Reference: https://projects.wojtekmaj.pl/react-lifecycle-methods-diagram/
 */
```

Class components:
- Have access to lifecycle methods
- Can hold and manage local state with `this.state`
- Use `this` keyword to access props and state
- Can use refs
- Can create and access instance methods

## Component Composition

One of React's powerful features is the ability to compose components together:

```jsx
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}

function App() {
  return (
    <div>
      <Welcome name="Alice" />
      <Welcome name="Bob" />
      <Welcome name="Charlie" />
    </div>
  );
}
```

## Props: Component Inputs

Props (short for "properties") are inputs to React components. They are passed from parent to child components.

### Passing Props

```jsx
// Restaurant card component receiving props
const RestaurantCard = (props) => {
  const { resData } = props;

  const {
    cloudinaryImageId,
    name,
    cuisines,
    avgRating,
    costForTwo,
    deliveryTime,
  } = resData?.data;

  return (
    <div
      className="res-card"
      style={{
        backgroundColor: '#f0f0f0',
      }}
    >
      <img
        className="res-logo"
        src={
          'https://res.cloudinary.com/swiggy/image/upload/fl_lossy,f_auto,q_auto,w_264,h_288,c_fill/' +
          cloudinaryImageId
        }
        alt="Biryani"
      />
      <h3>{name}</h3>
      <h4>{cuisines.join(', ')}</h4>
      <h4>{avgRating} stars</h4>
      <h4>₹{costForTwo / 100} FOR TWO</h4>
      <h4>{deliveryTime} minutes</h4>
    </div>
  );
};

// Parent component using RestaurantCard with props
<div className="res-container">
  {filteredRestaurant.map((restaurant) => (
    <RestaurantCard key={restaurant.data.id} resData={restaurant} />
  ))}
</div>
```

### Props are Read-Only

A component must never modify its own props. React components must act like pure functions with respect to their props.

### Default Props

You can specify default values for props:

```jsx
// For functional components
function Button({ text = "Click me" }) {
  return <button>{text}</button>;
}

// For class components
class Button extends React.Component {
  render() {
    return <button>{this.props.text}</button>;
  }
}

Button.defaultProps = {
  text: "Click me"
};
```

### Children Props

The `children` prop is a special prop that contains any elements included between the opening and closing tags of a component:

```jsx
function Container(props) {
  return <div className="container">{props.children}</div>;
}

// Usage
<Container>
  <h1>Title</h1>
  <p>Content</p>
</Container>
```

## State: Component Memory

State allows React components to change their output over time in response to user actions, network responses, and anything else.

### State in Class Components

```jsx
class Counter extends React.Component {
  constructor(props) {
    super(props);
    this.state = { count: 0 }; // Initialize state
  }
  
  incrementCount = () => {
    // Never modify state directly: this.state.count = this.state.count + 1
    this.setState({ count: this.state.count + 1 });
  }
  
  render() {
    return (
      <div>
        <p>Count: {this.state.count}</p>
        <button onClick={this.incrementCount}>Increment</button>
      </div>
    );
  }
}
```

### State in Functional Components (using Hooks)

```jsx
import { useEffect, useState } from 'react';
import RestaurantCard from './RestaurantCard';
import Shimmer from './Shimmer';

const Body = () => {
  // * State Variables - Super Powerful variables
  const [listOfRestaurants, setListOfRestaurants] = useState([]);
  const [filteredRestaurant, setFilteredRestaurant] = useState([]);
  const [searchText, setSearchText] = useState('');

  // * Whenever a state variable updates or changes, react triggers a reconciliation cycle(re-renders the component)
  console.log('Body rendered');

  useEffect(() => {
    fetchData();
  }, []);

  const fetchData = async () => {
    const data = await fetch(
      'https://www.swiggy.com/dapi/restaurants/list/v5?lat=12.9351929&lng=77.624480699999999&page_type=DESKTOP_WEB_LISTING'
    );

    const json = await data.json();

    console.log(json);
    // * optional chaining
    setListOfRestaurants(json?.data?.cards[2]?.data?.data?.cards);
    setFilteredRestaurant(json?.data?.cards[2]?.data?.data?.cards);
  };

  // * Conditional Rendering
  return listOfRestaurants.length === 0 ? (
    <Shimmer />
  ) : (
    <div className="body">
      <div className="filter">
        <div className="search">
          <input
            type="text"
            placeholder="Search a restaurant you want..."
            className="searchBox"
            value={searchText}
            onChange={(e) => {
              setSearchText(e.target.value);
            }}
          />
          <button
            onClick={() => {
              // * Filter the restaurant cards and update the UI
              console.log(searchText);

              const filteredRestaurant = listOfRestaurants.filter((res) =>
                res.data.name.toLowerCase().includes(searchText.toLowerCase())
              );

              setFilteredRestaurant(filteredRestaurant);
            }}
          >
            Search
          </button>
        </div>
        <button
          className="filter-btn"
          onClick={() => {
            // * Filter logic
            const filteredList = listOfRestaurants.filter(
              (res) => res.data.avgRating > 4
            );

            setListOfRestaurants(filteredList);
            console.log(filteredList);
          }}
        >
          Top Rated Restaurants
        </button>
      </div>
      <div className="res-container">
        {filteredRestaurant.map((restaurant) => (
          <RestaurantCard key={restaurant.data.id} resData={restaurant} />
        ))}
      </div>
    </div>
  );
};

export default Body;
```

## Component Lifecycle

Components go through a lifecycle of events:

### Class Component Lifecycle Methods

1. **Mounting** - Component is being created and inserted into the DOM
   - `constructor()`
   - `static getDerivedStateFromProps()`
   - `render()`
   - `componentDidMount()`

2. **Updating** - Component is being re-rendered due to changes in props or state
   - `static getDerivedStateFromProps()`
   - `shouldComponentUpdate()`
   - `render()`
   - `getSnapshotBeforeUpdate()`
   - `componentDidUpdate()`

3. **Unmounting** - Component is being removed from the DOM
   - `componentWillUnmount()`

Here's a complete example showing the lifecycle methods:

```jsx
import React from "react";

class Profile extends React.Component {
  constructor(props) {
    super(props);
    // State initialization
    this.state = {
      userInfo: {
        name: "Dummy Name",
        location: "Dummy Location",
      },
    };
    console.log("Constructor");
  }

  async componentDidMount() {
    console.log("Component Did Mount");
    
    // API call
    const data = await fetch("https://api.github.com/users/Shaikhmohddanish7");
    const json = await data.json();
    
    this.setState({
      userInfo: json,
    });
    
    console.log("componentDidMount completed");
    
    // Setting up timers (cleanup needed in componentWillUnmount)
    this.timer = setInterval(() => {
      console.log("Interval running");
    }, 1000);
  }

  componentDidUpdate(prevProps, prevState) {
    console.log("Component Did Update");
    
    // Comparing previous props/state with current
    if (this.state.count !== prevState.count) {
      // Some specific update logic
    }
  }

  componentWillUnmount() {
    console.log("Component Will Unmount");
    // Cleanup - important to prevent memory leaks
    clearInterval(this.timer);
  }

  render() {
    console.log("Render");
    
    const { name, location, avatar_url } = this.state.userInfo;
    
    return (
      <div className="profile-card">
        <img src={avatar_url} alt="avatar" />
        <h2>Name: {name}</h2>
        <h3>Location: {location}</h3>
      </div>
    );
  }
}

export default Profile;
```

### Functional Component Lifecycle (using Hooks)

The `useEffect` hook replaces multiple lifecycle methods:

```jsx
import { useState, useEffect } from "react";

const Profile = (props) => {
  const [profile, setProfile] = useState({
    name: "Dummy Name",
    location: "Dummy Location",
  });
  
  useEffect(() => {
    // This runs after first render (similar to componentDidMount)
    console.log("useEffect callback - similar to componentDidMount");
    
    // API call
    const fetchData = async () => {
      const data = await fetch("https://api.github.com/users/Shaikhmohddanish7");
      const json = await data.json();
      setProfile(json);
    };
    
    fetchData();
    
    const timer = setInterval(() => {
      console.log("Interval running in functional component");
    }, 1000);
    
    // Cleanup function (similar to componentWillUnmount)
    return () => {
      console.log("useEffect cleanup - similar to componentWillUnmount");
      clearInterval(timer);
    };
  }, []); // Empty dependency array means run only once after first render
  
  // This useEffect runs on every state update of searchText
  useEffect(() => {
    console.log("searchText changed useEffect");
  }, [props.searchText]); // Dependency array with values - runs when those values change
  
  console.log("render");
  
  return (
    <div className="profile-card">
      <img src={profile.avatar_url} alt="avatar" />
      <h2>Name: {profile.name}</h2>
      <h3>Location: {profile.location}</h3>
    </div>
  );
};

export default Profile;
```

## Component Patterns

### 1. Container and Presentational Components

Separating logic and presentation:

```jsx
// Container component (handles logic)
function UserListContainer() {
  const [users, setUsers] = useState([]);
  
  useEffect(() => {
    fetchUsers().then(data => setUsers(data));
  }, []);
  
  return <UserList users={users} />;
}

// Presentational component (handles UI)
function UserList({ users }) {
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

Functions that take a component and return a new enhanced component:

```jsx
function withLogger(WrappedComponent) {
  return function(props) {
    console.log('Props:', props);
    return <WrappedComponent {...props} />;
  }
}

// Usage
const EnhancedComponent = withLogger(MyComponent);
```

### 3. Render Props

Sharing code between components using a prop whose value is a function:

```jsx
class MouseTracker extends React.Component {
  state = { x: 0, y: 0 };
  
  handleMouseMove = (event) => {
    this.setState({
      x: event.clientX,
      y: event.clientY
    });
  };
  
  render() {
    return (
      <div onMouseMove={this.handleMouseMove}>
        {this.props.render(this.state)}
      </div>
    );
  }
}

// Usage
<MouseTracker
  render={({ x, y }) => (
    <h1>The mouse position is ({x}, {y})</h1>
  )}
/>
```

### 4. Hooks (Function Components)

Custom hooks allow sharing logic between components:

```jsx
// Custom hook
function useWindowSize() {
  const [size, setSize] = useState({ width: 0, height: 0 });
  
  useEffect(() => {
    const handleResize = () => {
      setSize({ width: window.innerWidth, height: window.innerHeight });
    };
    
    window.addEventListener('resize', handleResize);
    handleResize(); // Initial size
    
    return () => window.removeEventListener('resize', handleResize);
  }, []);
  
  return size;
}

// Usage in components
function MyComponent() {
  const { width, height } = useWindowSize();
  return <div>Window size: {width} x {height}</div>;
}
```

## Best Practices for Components

1. **Keep components small and focused** - Each component should do one thing well

2. **Use composition over inheritance** - Build complex UIs by composing simpler components

3. **Extract reusable logic** into custom hooks or higher-order components

4. **Use PropTypes or TypeScript** for type checking component props

5. **Name components clearly** - Use descriptive, specific names

6. **Use PascalCase for component names** - To differentiate from regular HTML elements

7. **Follow the Single Responsibility Principle** - Components should have only one reason to change

8. **Destructure props** for cleaner code:
   ```jsx
   function Profile({ name, age, bio }) {
     return (
       <div>
         <h2>{name}</h2>
         <p>Age: {age}</p>
         <p>{bio}</p>
       </div>
     );
   }
   ```

## Conclusion

Components are the heart of React. Understanding how to create and compose components effectively is essential for building maintainable React applications. Whether you choose functional or class components, the component-based architecture of React encourages reuse, separation of concerns, and cleaner code.
