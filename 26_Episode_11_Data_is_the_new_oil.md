# Episode 11 - Data is the New Oil

In this episode, we explore data management in React applications, focusing on React Context API for state management. We learn how to share data across components without prop drilling and implement user authentication using context.

## Key Concepts Covered

1. Prop Drilling and Its Limitations
2. Lifting State Up
3. React Context API
4. Context Providers and Consumers
5. Default Values in Context
6. Higher Order Components (HOC)
7. User Authentication Flow

## Prop Drilling

Prop drilling refers to the process of passing data from a component high in the hierarchy to a component that's deeply nested through props.

### The Problem with Prop Drilling

```jsx
// Parent Component
const App = () => {
  const [user, setUser] = useState('John');
  
  return (
    <div>
      <Header user={user} />
      <Main user={user} />
      <Footer user={user} />
    </div>
  );
}

// Main Component
const Main = ({ user }) => {
  return (
    <div>
      <Sidebar user={user} />
      <Content user={user} />
    </div>
  );
}

// Content Component
const Content = ({ user }) => {
  return (
    <div>
      <ArticleList user={user} />
    </div>
  );
}

// ArticleList Component
const ArticleList = ({ user }) => {
  return (
    <div>
      <Article user={user} />
    </div>
  );
}

// Finally, the component that actually needs the data
const Article = ({ user }) => {
  return <div>Written by {user}</div>;
}
```

In this example, `user` is passed through multiple components that don't actually use it, just to reach the `Article` component.

### Drawbacks of Prop Drilling

1. **Component Coupling**: Components become tightly coupled
2. **Maintenance Challenges**: Adding or removing props requires changes in multiple places
3. **Readability Issues**: Props can become confusing in complex applications
4. **Performance Concerns**: Unnecessary re-renders when props change

## Lifting State Up

Lifting state up is a pattern where state is moved from a lower component to a higher one in the component tree to share it among multiple components.

```jsx
const Parent = () => {
  const [count, setCount] = useState(0);
  
  const increment = () => setCount(count + 1);
  
  return (
    <div>
      <ChildA count={count} />
      <ChildB increment={increment} />
    </div>
  );
}

const ChildA = ({ count }) => <div>Count: {count}</div>;
const ChildB = ({ increment }) => <button onClick={increment}>Increment</button>;
```

This works well for simple cases but can lead to prop drilling in more complex applications.

## React Context API

The Context API provides a way to share values like user authentication, theme preferences, or other global data without explicitly passing props through every level of the component tree.

### Creating a Context

```jsx
import { createContext } from 'react';

const UserContext = createContext({
  loggedInUser: 'Default User',
});

export default UserContext;
```

### Context Provider

The Provider component allows consuming components to subscribe to context changes:

```jsx
const AppLayout = () => {
  const [userName, setUserName] = useState();

  useEffect(() => {
    // Authentication API call
    const data = {
      name: 'Vasu K',
    };
    setUserName(data.name);
  }, []);

  return (
    <UserContext.Provider value={{ loggedInUser: userName, setUserName }}>
      <div className="app">
        <Header />
        <Outlet />
      </div>
    </UserContext.Provider>
  );
};
```

### Consuming Context in Functional Components

Using the `useContext` Hook:

```jsx
import { useContext } from 'react';
import UserContext from '../utils/UserContext';

const Header = () => {
  const { loggedInUser } = useContext(UserContext);
  
  return (
    <header>
      <div>User: {loggedInUser}</div>
    </header>
  );
};
```

### Consuming Context in Class Components

Using the Context Consumer:

```jsx
import UserContext from '../utils/UserContext';

class About extends Component {
  render() {
    return (
      <div className="about-page">
        <h1>About Class Component</h1>
        <div>
          LoggedInUser
          <UserContext.Consumer>
            {({ loggedInUser }) => <h1>{loggedInUser}</h1>}
          </UserContext.Consumer>
        </div>
      </div>
    );
  }
}
```

## Default Values in Context

When you create a context, you can provide a default value:

```jsx
const UserContext = createContext({
  loggedInUser: 'Default User',
});
```

This default value is used when:
1. A component calls `useContext` without a matching Provider above it in the tree
2. The Provider is present but doesn't pass a value prop

```jsx
// No Provider, so default value is used
const Component = () => {
  const { loggedInUser } = useContext(UserContext);
  // loggedInUser will be 'Default User'
};

// Provider with no value, so default value is used
<UserContext.Provider>
  <Component />
</UserContext.Provider>

// Provider with explicit value, so this value is used
<UserContext.Provider value={{ loggedInUser: 'John' }}>
  <Component />
</UserContext.Provider>
```

## Nested Contexts

You can nest multiple Context Providers to create a hierarchy of contexts:

```jsx
<ThemeContext.Provider value={{ theme: 'dark' }}>
  <UserContext.Provider value={{ loggedInUser: 'John' }}>
    <LanguageContext.Provider value={{ language: 'en' }}>
      <App />
    </LanguageContext.Provider>
  </UserContext.Provider>
</ThemeContext.Provider>
```

Components can then consume whichever contexts they need:

```jsx
const Component = () => {
  const { theme } = useContext(ThemeContext);
  const { loggedInUser } = useContext(UserContext);
  const { language } = useContext(LanguageContext);
  
  // Use theme, loggedInUser, and language
};
```

## Updating Context Values

Context can include functions to update its values:

```jsx
const Body = () => {
  const { loggedInUser, setUserName } = useContext(UserContext);
  
  return (
    <div>
      <label htmlFor="name">User Name: </label>
      <input
        id="name"
        className="border border-black p-2"
        value={loggedInUser}
        onChange={(e) => setUserName(e.target.value)}
      />
    </div>
  );
};
```

## Higher Order Components (HOC)

A Higher Order Component is a function that takes a component and returns a new enhanced component:

```jsx
export const withPromotedLabel = (RestaurantCard) => {
  return (props) => {
    return (
      <div>
        <label className="absolute bg-black text-white m-2 p-2 rounded-lg">
          Promoted
        </label>
        <RestaurantCard {...props} />
      </div>
    );
  };
};

// Usage
const RestaurantCardPromoted = withPromotedLabel(RestaurantCard);

// In JSX
{restaurant.data.promoted ? (
  <RestaurantCardPromoted resData={restaurant} />
) : (
  <RestaurantCard resData={restaurant} />
)}
```

## Implementation in Our Food Delivery App

### User Authentication with Context

We implemented a simple authentication flow using Context:

1. **Setting Up the Context**:
   ```jsx
   // UserContext.js
   import { createContext } from 'react';
   
   const UserContext = createContext({
     loggedInUser: 'Default User',
   });
   
   export default UserContext;
   ```

2. **Providing the Context**:
   ```jsx
   // App.js
   const AppLayout = () => {
     const [userName, setUserName] = useState();
   
     useEffect(() => {
       // Authentication API call
       const data = {
         name: 'Vasu K',
       };
       setUserName(data.name);
     }, []);
   
     return (
       <UserContext.Provider value={{ loggedInUser: userName, setUserName }}>
         <div className="app">
           <Header />
           <Outlet />
         </div>
       </UserContext.Provider>
     );
   };
   ```

3. **Consuming the Context in Header**:
   ```jsx
   // Header.js
   const Header = () => {
     const { loggedInUser } = useContext(UserContext);
   
     return (
       <header>
         <li className="px-4 font-bold">
           <Link className="links">{loggedInUser}</Link>
         </li>
       </header>
     );
   };
   ```

4. **Updating Context in Body**:
   ```jsx
   // Body.js
   const Body = () => {
     const { loggedInUser, setUserName } = useContext(UserContext);
   
     return (
       <div>
         <div className="search m-4 p-4 flex items-center">
           <label htmlFor="name">User Name: </label>
           <input
             id="name"
             className="border border-black p-2"
             value={loggedInUser}
             onChange={(e) => setUserName(e.target.value)}
           />
         </div>
       </div>
     );
   };
   ```

5. **Using Context in RestaurantCard**:
   ```jsx
   // RestaurantCard.js
   const RestaurantCard = (props) => {
     const { resData } = props;
     const { loggedInUser } = useContext(UserContext);
   
     return (
       <div className="restaurant-card">
         {/* Restaurant details */}
         <h4>User: {loggedInUser}</h4>
       </div>
     );
   };
   ```

## When to Use Context

Context is best for:

1. **Global Data**: Authentication, theme, language preferences
2. **Deeply Nested Components**: When many components need the same data
3. **Avoiding Prop Drilling**: When data needs to pass through many layers

Context might not be needed for:
1. **Simple Props**: For passing a few props down a small component tree
2. **Component Composition**: Sometimes passing components as props is cleaner
3. **Performance-Critical Areas**: Context can cause re-renders in large trees

## Context vs. Other State Management

### Context + useReducer

For more complex state logic, combine Context with useReducer:

```jsx
const initialState = { count: 0 };

function reducer(state, action) {
  switch (action.type) {
    case 'increment':
      return { count: state.count + 1 };
    case 'decrement':
      return { count: state.count - 1 };
    default:
      throw new Error();
  }
}

const CountContext = createContext();

const CountProvider = ({ children }) => {
  const [state, dispatch] = useReducer(reducer, initialState);
  return (
    <CountContext.Provider value={{ state, dispatch }}>
      {children}
    </CountContext.Provider>
  );
};
```

### Context vs. Redux

- **Context**: Built into React, simpler for small to medium apps
- **Redux**: More structure, better for complex state, better dev tools

## Summary

In this episode, we:

1. Identified the problems with prop drilling
2. Learned about the Context API for global state management
3. Implemented a user authentication system using Context
4. Created a context provider with default values
5. Consumed context in both functional and class components
6. Updated context values from child components
7. Implemented Higher Order Components for enhanced functionality

The key takeaway is that Context provides a powerful way to share data across components without prop drilling, making our application more maintainable and easier to understand. As the saying goes, "Data is the new oil" - and with Context, we can ensure our data flows efficiently throughout our application.
