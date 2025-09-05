# Episode 06 - Exploring The World

In this episode, we expand our food delivery app by connecting it to real APIs and introducing important React features like useEffect, conditional rendering, and shimmer UI. We'll also implement search functionality and a login/logout button.

## Key Concepts Covered

1. useEffect Hook
2. API Calls in React
3. Conditional Rendering
4. Shimmer UI for Better UX
5. Search Functionality
6. useState for Toggle Functionality
7. async/await for API Calls
8. Optional Chaining

## Fetching Data from APIs

### useEffect Hook

The useEffect Hook allows you to perform side effects in functional components, such as data fetching, subscriptions, or manually changing the DOM.

```javascript
useEffect(() => {
  fetchData();
}, []);
```

The empty dependency array `[]` means this effect will run only once after the initial render.

### API Integration

We replaced our mock data with actual API calls to fetch real restaurant data:

```javascript
const fetchData = async () => {
  const data = await fetch(
    'https://www.swiggy.com/dapi/restaurants/list/v5?lat=12.9351929&lng=77.624480699999999&page_type=DESKTOP_WEB_LISTING'
  );

  const json = await data.json();

  // Using optional chaining to safely access nested properties
  setListOfRestaurants(json?.data?.cards[2]?.data?.data?.cards);
  setFilteredRestaurant(json?.data?.cards[2]?.data?.data?.cards);
};
```

Key concepts in this code:
- **async/await**: Modern JavaScript syntax for handling promises more cleanly
- **fetch API**: Browser API for making HTTP requests
- **json()**: Method that returns a promise that resolves with the result of parsing the response body text as JSON
- **optional chaining (`?.`)**: Safely accesses nested properties without causing errors

## Shimmer UI for Better UX

Shimmer UI is a placeholder that mimics the layout of the page before the content loads, providing a better user experience:

```javascript
// Shimmer.js
const Shimmer = () => {
  return (
    <div className="shimmer-container">
      <div className="shimmer-card"></div>
      <div className="shimmer-card"></div>
      <div className="shimmer-card"></div>
      {/* Multiple shimmer cards */}
    </div>
  );
};
```

### Conditional Rendering

We implemented conditional rendering to show the Shimmer UI while data is loading:

```javascript
// Using ternary operator for conditional rendering
return listOfRestaurants.length === 0 ? (
  <Shimmer />
) : (
  // Main UI content
);

// Alternatively, using if statement (commented out in the code)
// if (listOfRestaurants.length === 0) {
//   return <Shimmer />;
// }
```

Conditional rendering allows us to:
- Show different UI based on the application state
- Handle loading states gracefully
- Provide better feedback to users

## Search Functionality

We implemented a search feature to filter restaurants by name:

```javascript
const [searchText, setSearchText] = useState('');
const [filteredRestaurant, setFilteredRestaurant] = useState([]);

// Input field with controlled component pattern
<input
  type="text"
  placeholder="Search a restaurant you want..."
  className="searchBox"
  value={searchText}
  onChange={(e) => {
    setSearchText(e.target.value);
  }}
/>

// Search button
<button
  onClick={() => {
    const filteredRestaurant = listOfRestaurants.filter((res) =>
      res.data.name.toLowerCase().includes(searchText.toLowerCase())
    );

    setFilteredRestaurant(filteredRestaurant);
  }}
>
  Search
</button>
```

Key concepts:
- **Controlled Components**: The input field's value is controlled by React state
- **Filter Method**: JavaScript array method that creates a new array with elements that pass a test
- **toLowerCase()**: Makes the search case-insensitive
- **includes()**: Checks if a string contains another string

## Login/Logout Toggle

We implemented a toggle button that switches between "Login" and "Logout" states:

```javascript
// In Header.js
const [btnNameReact, setBtnNameReact] = useState('Login');

<button
  className="loginBtn"
  onClick={() => {
    btnNameReact === 'Login'
      ? setBtnNameReact('Logout')
      : setBtnNameReact('Login');
  }}
>
  {btnNameReact}
</button>
```

This demonstrates:
- Using state to track UI elements
- Toggling between different states
- Conditional logic in event handlers

## Understanding useEffect Behavior

The useEffect Hook runs after the component renders:

```javascript
useEffect(() => {
  fetchData();
}, []);

console.log('Body rendered');
```

In the console, you would see "Body rendered" first, then the API data, confirming that useEffect runs after the component renders.

### Dependency Array in useEffect

The dependency array controls when the effect runs:

- `[]`: Run once after initial render
- `[dep1, dep2]`: Run when any dependency changes
- No dependency array: Run after every render

## Advanced Concepts

### Optional Chaining

Optional chaining (`?.`) is a JavaScript feature that allows you to safely access deeply nested properties without causing errors:

```javascript
// Without optional chaining (may cause errors if any property is undefined)
setListOfRestaurants(json.data.cards[2].data.data.cards);

// With optional chaining (safely handles undefined properties)
setListOfRestaurants(json?.data?.cards[2]?.data?.data?.cards);
```

### JS Expression vs Statement

In React JSX:
- **Expressions** can be used inside curly braces `{}`
- **Statements** cannot be used inside JSX

Examples:
- Expression (valid in JSX): `{btnNameReact === 'Login' ? 'Logout' : 'Login'}`
- Statement (invalid in JSX): `{if (btnNameReact === 'Login') { return 'Logout'; } else { return 'Login'; }}`

## CORS (Cross-Origin Resource Sharing)

CORS is a security mechanism that restricts web pages from making requests to a different domain than the one that served the original page. When building applications that call APIs from different domains, you may encounter CORS errors.

Solutions include:
- Using a CORS proxy
- Configuring the server to allow cross-origin requests
- Making the request from your backend instead of directly from the browser

## Updated Body Component

The Body component now handles API data, search functionality, and conditional rendering:

```jsx
import { useEffect, useState } from 'react';
import RestaurantCard from './RestaurantCard';
import Shimmer from './Shimmer';

const Body = () => {
  const [listOfRestaurants, setListOfRestaurants] = useState([]);
  const [filteredRestaurant, setFilteredRestaurant] = useState([]);
  const [searchText, setSearchText] = useState('');

  console.log('Body rendered');

  useEffect(() => {
    fetchData();
  }, []);

  const fetchData = async () => {
    const data = await fetch(
      'https://www.swiggy.com/dapi/restaurants/list/v5?lat=12.9351929&lng=77.624480699999999&page_type=DESKTOP_WEB_LISTING'
    );

    const json = await data.json();
    setListOfRestaurants(json?.data?.cards[2]?.data?.data?.cards);
    setFilteredRestaurant(json?.data?.cards[2]?.data?.data?.cards);
  };

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
            const filteredList = listOfRestaurants.filter(
              (res) => res.data.avgRating > 4
            );
            setListOfRestaurants(filteredList);
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

## Monolithic vs Microservice Architecture

### Monolithic Architecture

A monolithic architecture is a traditional software development approach where all components of an application are interconnected and interdependent:

- **Single Codebase**: All application code is in one codebase
- **Single Deployment Unit**: The entire application is deployed as a single unit
- **Tightly Coupled Components**: Changes in one part can affect the entire application
- **Simpler Development**: Initially easier to develop and test
- **Scaling Challenges**: Must scale the entire application, not just busy components

### Microservice Architecture

A microservice architecture breaks an application into smaller, independent services:

- **Multiple Codebases**: Each service has its own codebase
- **Independent Deployment**: Services can be deployed independently
- **Loosely Coupled**: Services communicate via well-defined APIs
- **Technology Diversity**: Different services can use different technologies
- **Scalability**: Can scale individual services based on demand
- **Resilience**: Failure in one service doesn't bring down the entire application

### Key Differences

1. **Development Complexity**: Microservices are more complex to develop initially
2. **Deployment**: Monoliths deploy as a single unit; microservices deploy independently
3. **Scaling**: Monoliths scale as a whole; microservices scale individually
4. **Technology**: Monoliths use a single technology stack; microservices can use multiple
5. **Team Organization**: Monoliths often have a single team; microservices can have multiple teams focused on specific services

## Summary

In this episode, we:

1. Integrated real API data into our application
2. Implemented loading states with Shimmer UI
3. Used conditional rendering to improve user experience
4. Created a search feature for filtering restaurants
5. Added a login/logout toggle button
6. Learned about useEffect for side effects
7. Explored advanced concepts like optional chaining and async/await

The key takeaway is that connecting to external APIs requires understanding asynchronous JavaScript, handling loading states, and implementing proper error handling to create a robust user experience.
