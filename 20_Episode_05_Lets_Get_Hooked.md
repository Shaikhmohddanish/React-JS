# Episode 05 - Let's Get Hooked!

In this episode, we focus on organizing our codebase and introducing React Hooks. We'll learn how to structure our React application properly and implement state management using the useState Hook.

## Key Concepts Covered

1. File and Folder Structure
2. Import and Export Methods
3. Introduction to React Hooks
4. useState Hook for State Management
5. Config-Driven UI with Constants
6. Mock Data for Development

## Organizing Our App

### Folder Structure

One of the major improvements in this episode is reorganizing our code into a proper folder structure:

```
src/
├── components/
│   ├── Body.js
│   ├── Header.js
│   └── RestaurantCard.js
├── utils/
│   ├── constants.js
│   └── mockData.js
├── App.js
├── index.css
└── index.html
```

This structure helps with:
- **Maintainability**: Makes it easier to find and update code
- **Scalability**: Allows the app to grow without becoming unwieldy
- **Reusability**: Components can be reused across the application
- **Separation of Concerns**: Each file has a specific responsibility

## Import and Export Types

In React, there are multiple ways to export and import code between files:

### 1. Default Export/Import

When a module exports a single main value:

```javascript
// Exporting
const Header = () => {
  // Component code
};

export default Header;

// Importing
import Header from './components/Header';
```

### 2. Named Export/Import

When a module exports multiple values:

```javascript
// Exporting
export const CDN_URL = 'https://res.cloudinary.com/swiggy/image/upload/fl_lossy,f_auto,q_auto,w_264,h_288,c_fill/';
export const LOGO_URL = 'https://png.pngtree.com/png-vector/20230217/ourmid/pngtree-food-logo-design-for-restaurant-and-business-png-image_6604922.png';

// Importing
import { CDN_URL, LOGO_URL } from '../utils/constants';
```

### 3. Import All as an Object

When you want to import all exports as a single object:

```javascript
// Importing all exports as a namespace
import * as Constants from '../utils/constants';

// Usage
const imageUrl = Constants.CDN_URL + cloudinaryImageId;
```

## Introduction to React Hooks

React Hooks are JavaScript functions that let you "hook into" React state and lifecycle features from function components.

### Why Hooks?

Before Hooks:
- Stateful logic had to be in class components
- Complex components became hard to understand
- Classes can be confusing (this, binding, etc.)

With Hooks:
- Use state and other React features without writing a class
- Reuse stateful logic between components
- Organize related code together (vs. splitting across lifecycle methods)

## useState Hook

The `useState` Hook is a function that lets you add React state to functional components.

```javascript
import { useState } from 'react';

// Using useState
const [listOfRestaurants, setListOfRestaurants] = useState(resList);
```

When you call `useState`, it returns an array with two elements:
1. The current state value (`listOfRestaurants`)
2. A function to update the state (`setListOfRestaurants`)

### Implementing the Filter Button

We implemented a "Top Rated Restaurants" filter button using useState:

```javascript
<div className="filter">
  <button
    className="filter-btn"
    onClick={() => {
      // Filter logic
      const filteredList = listOfRestaurants.filter(
        (res) => res.data.avgRating > 4
      );

      setListOfRestaurants(filteredList);
    }}
  >
    Top Rated Restaurants
  </button>
</div>
```

When the button is clicked:
1. We filter the restaurant list to only include restaurants with a rating above 4
2. We update the state with the filtered list using `setListOfRestaurants`
3. React automatically re-renders the component with the new state

### Normal JS Variable vs. State Variable

If we had used a normal JavaScript variable instead of a state variable:

```javascript
let listOfRestaurants = resList;

// Later in the code
listOfRestaurants = filteredList; // This won't trigger a re-render!
```

The UI wouldn't update because React doesn't know the variable changed. With state variables, React knows when to re-render the component.

## Config-Driven UI with Constants

We moved hardcoded values to a constants file:

```javascript
// constants.js
export const CDN_URL = 'https://res.cloudinary.com/swiggy/image/upload/fl_lossy,f_auto,q_auto,w_264,h_288,c_fill/';
export const LOGO_URL = 'https://png.pngtree.com/png-vector/20230217/ourmid/pngtree-food-logo-design-for-restaurant-and-business-png-image_6604922.png';
```

This approach:
- Avoids repetition
- Makes changes easier (change once, affects everywhere)
- Improves maintainability
- Enhances readability by using meaningful constant names

## Component Updates

### Updated RestaurantCard

```jsx
import { CDN_URL } from '../utils/constants';

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
    <div className="res-card" style={{ backgroundColor: '#f0f0f0' }}>
      <img
        className="res-logo"
        src={CDN_URL + cloudinaryImageId}
        alt="Restaurant"
      />
      <h3>{name}</h3>
      <h4>{cuisines.join(', ')}</h4>
      <h4>{avgRating} stars</h4>
      <h4>₹{costForTwo / 100} FOR TWO</h4>
      <h4>{deliveryTime} minutes</h4>
    </div>
  );
};

export default RestaurantCard;
```

### Updated Header

```jsx
import { LOGO_URL } from '../utils/constants';

const Header = () => {
  return (
    <div className="header">
      <div className="logo-container">
        <img src={LOGO_URL} alt="App Logo" className="logo" />
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

export default Header;
```

### Updated Body with useState

```jsx
import { useState } from 'react';
import RestaurantCard from './RestaurantCard';
import resList from '../utils/mockData';

const Body = () => {
  const [listOfRestaurants, setListOfRestaurants] = useState(resList);

  return (
    <div className="body">
      <div className="filter">
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
        {listOfRestaurants.map((restaurant) => (
          <RestaurantCard key={restaurant.data.id} resData={restaurant} />
        ))}
      </div>
    </div>
  );
};

export default Body;
```

## Summary

In this episode, we:

1. Reorganized our app with a proper folder structure
2. Learned different import/export methods
3. Introduced React Hooks, specifically useState
4. Implemented state management for the restaurant list
5. Created a filter button that updates the UI based on state changes
6. Improved our code with constants and config-driven UI

The key takeaway is that React Hooks provide a way to use state and other React features without writing classes, making our code cleaner and more maintainable.
