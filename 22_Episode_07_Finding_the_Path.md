# Episode 07 - Finding the Path

In this episode, we focus on implementing routing in our React application using react-router-dom. We'll create multiple pages, implement dynamic routing, and understand the difference between client-side and server-side routing.

## Key Concepts Covered

1. React Router and Client-Side Routing
2. Creating Multiple Pages with React Router
3. Dynamic Routing with Parameters
4. Error Handling in Routing
5. Navigation with Link Component
6. Nested Routes with Outlet
7. Single Page Applications (SPAs)

## Setting Up React Router

### Installing and Configuring React Router

The first step is to install react-router-dom:

```bash
npm install react-router-dom
```

Then, we configure our router in App.js:

```jsx
import { createBrowserRouter, RouterProvider, Outlet } from 'react-router-dom';

const appRouter = createBrowserRouter([
  {
    path: '/',
    element: <AppLayout />,
    children: [
      {
        path: '/',
        element: <Body />,
      },
      {
        path: '/about',
        element: <About />,
      },
      {
        path: '/contact',
        element: <Contact />,
      },
      {
        path: '/restaurants/:resId',
        element: <RestaurantMenu />,
      },
    ],
    errorElement: <Error />,
  },
]);

root.render(<RouterProvider router={appRouter} />);
```

### Creating the AppLayout with Outlet

The `Outlet` component is where the child routes will be rendered:

```jsx
const AppLayout = () => {
  return (
    <div className="app">
      <Header />
      <Outlet />
      <Footer />
    </div>
  );
};
```

## Client-Side Routing vs. Server-Side Routing

### Server-Side Routing

In traditional server-side routing:
1. Browser sends a request to the server for a new page
2. Server processes the request
3. Server sends back the full HTML for the new page
4. Browser loads the entire page from scratch
5. All resources are reloaded (CSS, JS, images)

Pros:
- Better for SEO (historically)
- Works without JavaScript
- Initial page load might be faster

Cons:
- Full page refresh on every navigation
- More server load
- Can feel slower for users

### Client-Side Routing

With client-side routing in SPAs:
1. Initial request loads the application shell
2. JavaScript intercepts navigation requests
3. React Router updates the UI without a page refresh
4. Only the data needed for the new view is fetched

Pros:
- Smoother, faster transitions between pages
- No full page reloads
- Reduced server load
- Better user experience

Cons:
- Initial load might be slower
- Requires JavaScript to be enabled
- More complex to implement

## Navigation with Link Component

Instead of using regular `<a>` tags which cause page refreshes, we use React Router's `Link` component:

```jsx
import { Link } from 'react-router-dom';

<div className="nav-items">
  <ul>
    <li>
      <Link to="/" className="links">
        Home
      </Link>
    </li>
    <li>
      <Link to="/about" className="links">
        About Us
      </Link>
    </li>
    <li>
      <Link to="/contact" className="links">
        Contact Us
      </Link>
    </li>
  </ul>
</div>
```

This allows for navigation without page refreshes, maintaining the SPA experience.

## Dynamic Routing with Parameters

We can create dynamic routes by using parameters in our route paths:

```jsx
{
  path: '/restaurants/:resId',
  element: <RestaurantMenu />,
}
```

Then, in our component, we access the parameter using the `useParams` hook:

```jsx
import { useParams } from 'react-router-dom';

const RestaurantMenu = () => {
  const { resId } = useParams();
  
  // Now we can use resId to fetch data for this specific restaurant
  // ...
}
```

### Creating the Restaurant Menu Page

The Restaurant Menu page uses the `resId` parameter to fetch specific restaurant data:

```jsx
const RestaurantMenu = () => {
  const [resInfo, setResInfo] = useState(null);
  const { resId } = useParams();

  useEffect(() => {
    fetchMenu();
  }, []);

  const fetchMenu = async () => {
    const data = await fetch(MENU_API + resId);
    const json = await data.json();
    setResInfo(json.data);
  };

  if (resInfo === null) return <ShimmerMenu />;

  // Extract restaurant information
  const {
    name,
    cuisines,
    costForTwoMessage,
    cloudinaryImageId,
    avgRating,
    deliveryTime,
  } = resInfo?.cards[0]?.card?.card?.info;

  // Extract menu items
  const { itemCards } =
    resInfo?.cards[2]?.groupedCard?.cardGroupMap?.REGULAR?.cards[1]?.card?.card;

  return (
    <div className="menu">
      {/* Restaurant header */}
      <header className="menu-header">
        <div className="menu-header-left">
          <img src={CDN_URL + cloudinaryImageId} alt="Restaurant Info" />
        </div>
        <div className="menu-header-right">
          <div className="top">
            <h1>{name}</h1>
            <h3>{cuisines.join(', ')}</h3>
          </div>
          <div className="bottom">
            <h4 className="avg-rating">
              <span className="icons"><AiOutlineStar /></span>
              <span>{avgRating}</span>
            </h4>
            <h4 className="time">
              <span className="icons"><FiClock /></span>
              <span> {deliveryTime} MINS</span>
            </h4>
            <h3>{costForTwoMessage}</h3>
          </div>
        </div>
      </header>

      {/* Menu items */}
      <div className="menu-main">
        <h2>Menu</h2>
        <h3 className="items">{itemCards.length} items</h3>
        <div className="menu-main-card-container">
          {itemCards.map((item) => (
            <div key={item.card.info.id} className="menu-card">
              <div className="menu-card-left">
                <h2 className="menu-name">{item.card.info.name}</h2>
                <h3 className="menu-price">
                  ₹{item.card.info.price / 100 || item.card.info.defaultPrice / 100}
                </h3>
                <h4 className="menu-description">{item.card.info.description}</h4>
              </div>
              <div className="menu-card-right">
                <img src={CDN_URL + item.card.info.imageId} alt="Menu Info" />
              </div>
            </div>
          ))}
        </div>
      </div>
    </div>
  );
};
```

## Error Handling in Routing

We can create a custom error page to handle routing errors:

```jsx
import { useRouteError } from 'react-router-dom';

const Error = () => {
  const err = useRouteError();
  
  return (
    <div>
      <h1>Oops❗</h1>
      <h2>Something went wrong❗</h2>
      <h3>
        {err.status}: {err.statusText}
      </h3>
    </div>
  );
};
```

This is configured in our router with the `errorElement` property:

```jsx
{
  path: '/',
  element: <AppLayout />,
  children: [...],
  errorElement: <Error />,
}
```

## Updating the Body Component for Routing

Our Body component now wraps each RestaurantCard in a Link to navigate to the restaurant details:

```jsx
{filteredRestaurant.map((restaurant) => (
  <Link
    style={{
      textDecoration: 'none',
      color: '#000',
    }}
    key={restaurant.data.id}
    to={'/restaurants/' + restaurant.data.id}
  >
    <RestaurantCard resData={restaurant} />
  </Link>
))}
```

## Single Page Applications (SPAs)

A Single Page Application (SPA) is a web application that loads a single HTML page and dynamically updates that page as the user interacts with the app.

Key characteristics of SPAs:
1. They load all necessary HTML, CSS, and JavaScript in the initial page load
2. They use client-side routing to navigate between "pages" without full page refreshes
3. They dynamically fetch data and update the DOM as needed
4. They provide a more fluid user experience similar to native applications

Our food delivery app is now a SPA because:
- Users can navigate between Home, About, Contact, and Restaurant pages without full page refreshes
- React Router handles the URL changes and component rendering
- The app shell (Header and Footer) stays consistent while only the content changes

## Adding Images in React

There are several ways to add images in React:

### 1. Using Public Folder

Images in the public folder can be accessed directly:

```jsx
<img src="/logo.png" alt="Logo" />
```

### 2. Importing Images

For images that are part of your source code:

```jsx
import logoImg from '../assets/logo.png';

// Then in your component
<img src={logoImg} alt="Logo" />
```

### 3. Using External URLs

For images hosted elsewhere (like our Swiggy images):

```jsx
<img 
  src={CDN_URL + cloudinaryImageId} 
  alt="Restaurant Logo" 
/>
```

### 4. Using Icons from Libraries

We're using React Icons in our app:

```jsx
import { AiOutlineStar } from 'react-icons/ai';
import { FiClock } from 'react-icons/fi';

// Then in your component
<span className="icons"><AiOutlineStar /></span>
```

## Understanding React Hooks Behavior

### useState() Behavior

If we `console.log(useState())`, we would see:
```
[undefined, function]
```

This is because useState returns an array with:
1. The current state value (initially undefined if no default is provided)
2. A function to update that state

### useEffect Dependencies

The useEffect Hook behavior changes based on its dependency array:

1. **No dependency array**: Runs after every render
   ```jsx
   useEffect(() => {
     console.log("This runs after every render");
   });
   ```

2. **Empty dependency array**: Runs only on the initial render
   ```jsx
   useEffect(() => {
     console.log("This runs only once after initial render");
   }, []);
   ```

3. **With dependencies**: Runs whenever any dependency changes
   ```jsx
   useEffect(() => {
     console.log("This runs when searchText changes");
   }, [searchText]);
   ```

## Summary

In this episode, we:

1. Implemented routing in our React application using react-router-dom
2. Created multiple pages including Home, About, Contact, and Restaurant Menu
3. Implemented dynamic routing for restaurant details
4. Created an error page for handling routing errors
5. Used the Link component for client-side navigation
6. Implemented nested routes with Outlet
7. Transformed our app into a Single Page Application (SPA)

The key takeaway is that client-side routing with React Router enables a smooth, app-like experience without full page refreshes, while still maintaining the ability to directly access specific pages through URLs.
