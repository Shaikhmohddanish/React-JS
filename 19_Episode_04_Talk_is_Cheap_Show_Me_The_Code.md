# Episode 04 - Talk is Cheap, Show Me the Code!

In this episode, we dive into building a real-world food delivery app. The focus is on practical React development, applying all the concepts we've learned so far to create a working application with multiple components.

## Project Overview: Food Ordering App

We'll create a food ordering application with the following components:

```
Components of Our Food-Order App
- Header
  - Logo
  - Nav Items
- Body
  - Search Bar
  - Restaurant-Container
    - Restaurant-Card
      - Img
      - Name of Res, Star Rating, cuisine, delivery time
- Footer
  - Copyright
  - Links
  - Address
  - Contact
```

## Key Concepts Covered

1. Component Composition
2. Props and Data Flow
3. Dynamic Rendering with map()
4. Config-driven UI
5. Keys in React Lists
6. Destructuring in React
7. Rendering Dynamic Data

## Building the Application

### 1. Header Component

The Header component contains the logo and navigation menu:

```jsx
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

### 2. Restaurant Card Component

This component displays information about a single restaurant:

```jsx
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
```

### 3. Body Component

The Body component contains the search bar and restaurant list:

```jsx
const Body = () => {
  return (
    <div className="body">
      <div className="search-container">
        <input type="text" placeholder="Search Food or Restaurant" />
        <button>Search</button>
      </div>
      <div className="res-container">
        {resList.map((restaurant) => (
          <RestaurantCard key={restaurant.data.id} resData={restaurant} />
        ))}
      </div>
    </div>
  );
};
```

### 4. Footer Component

The Footer component displays copyright information:

```jsx
const currYear = new Date().getFullYear();

const Footer = () => {
  return (
    <footer className="footer">
      <p>
        Copyright &copy; {currYear}, Made with 💗 by <strong>Vasu</strong>
      </p>
    </footer>
  );
};
```

### 5. AppLayout Component

This is the main component that brings together all the other components:

```jsx
const AppLayout = () => {
  return (
    <div className="app">
      <Header />
      <Body />
      <Footer />
    </div>
  );
};
```

### 6. Rendering the App

Finally, we render the AppLayout component to the DOM:

```jsx
const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(<AppLayout />);
```

## Important Concepts Explained

### Props in React

Props (short for "properties") are how we pass data from parent components to child components:

```jsx
// Passing props
<RestaurantCard resData={restaurant} />

// Receiving props
const RestaurantCard = (props) => {
  const { resData } = props;
  // Now we can use resData
}
```

Props are just JavaScript objects, and we can destructure them for cleaner code:

```jsx
// Destructuring on the fly
const RestaurantCard = ({ resData }) => {
  // Use resData directly
}

// Nested destructuring
const { cloudinaryImageId, name, cuisines } = resData?.data;
```

### Rendering Lists in React

When rendering lists in React, we use the `map()` method to transform data into React elements:

```jsx
{resList.map((restaurant) => (
  <RestaurantCard key={restaurant.data.id} resData={restaurant} />
))}
```

### Keys in React Lists

React needs a unique "key" prop for each item in a list to efficiently update the UI:

```jsx
// Good practice: Use a unique ID
<RestaurantCard key={restaurant.data.id} resData={restaurant} />

// Not recommended practice: Using index as key
{resList.map((restaurant, index) => (
  <RestaurantCard key={index} resData={restaurant} />
))}
```

**Why keys are important:**
- Keys help React identify which items have changed, been added, or been removed
- They give elements a stable identity
- They help React optimize rendering by re-using elements

Best practices for keys:
1. Use unique and stable IDs from your data
2. Avoid using indexes as keys (except as a last resort)
3. Keys should be unique among siblings, not globally

### Config-Driven UI

A "config-driven UI" is a user interface that is built and configured using a declarative configuration file or data structure, rather than being hardcoded. In our app, the restaurant data structure determines what gets displayed.

This approach:
- Makes the application dynamic
- Allows for different UIs for different users (personalization)
- Enables A/B testing
- Makes the application maintainable and scalable

### Data Layer vs UI Layer

A good frontend engineer understands both:
- UI Layer: Components, styling, and interactions
- Data Layer: State management, data flow, and data transformation

## Restaurant Data Structure

Our application uses a structured data format for restaurants:

```jsx
const resList = [
  {
    type: 'restaurant',
    data: {
      id: '121603',
      name: 'Kannur Food Point',
      cloudinaryImageId: 'bmwn4n4bn6n1tcpc8x2h',
      cuisines: ['Kerala', 'Chinese'],
      costForTwo: 30000,
      deliveryTime: 24,
      avgRating: '4.3',
      // ... other properties
    }
  },
  // ... more restaurants
];
```

## Summary

In this episode, we've built a complete food ordering app with multiple components that demonstrates:

1. Component hierarchy and composition
2. Passing data through props
3. Destructuring for cleaner code
4. Rendering lists with map()
5. Using keys properly
6. Building a config-driven UI

The application structure follows a clear separation of concerns, with each component responsible for a specific part of the UI, making our code maintainable and scalable.
