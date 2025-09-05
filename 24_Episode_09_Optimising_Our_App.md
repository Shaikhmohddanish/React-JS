# Episode 09 - Optimising Our App

In this episode, we focus on optimizing our React application using several advanced techniques like lazy loading, code splitting, and custom hooks. These optimizations help improve performance and maintainability of our food delivery app.

## Key Concepts Covered

1. Code Splitting and Lazy Loading
2. React Suspense for Loading States
3. Custom Hooks for Logic Reuse
4. Online Status Detection
5. Abstracting API Calls into Hooks
6. Performance Optimization Techniques

## Code Splitting with React.lazy()

### What is Code Splitting?

Code splitting is a technique that allows you to split your JavaScript bundle into smaller chunks that can be loaded on demand, rather than loading the entire application upfront. This improves initial load time and performance.

React.lazy() is a function that enables code splitting in React applications:

```jsx
import React, { lazy, Suspense } from 'react';

// Instead of regular import
// import Grocery from './components/Grocery';

// Use lazy loading
const Grocery = lazy(() => import('./components/Grocery'));
```

### Why Use lazy()?

1. **Reduced Initial Bundle Size**: Only load what's necessary for the initial render
2. **Faster Initial Load**: Users see and can interact with your app more quickly
3. **Better Resource Utilization**: Loads components only when needed
4. **Improved Performance for Low-End Devices**: Less JavaScript to parse and execute on initial load

## React Suspense

Suspense is a React feature that allows components to "wait" for something before rendering, showing a fallback content (like a loading indicator) while waiting.

```jsx
<Suspense fallback={<h1>Loading...</h1>}>
  <Grocery />
</Suspense>
```

When the Grocery component is being loaded, the fallback UI will be displayed. Once the component is ready, it will replace the fallback content.

### Why Do We Need Suspense?

Without Suspense, when a lazily loaded component is rendered, React would throw an error because the component isn't available yet. Suspense provides a way to handle this asynchronous loading gracefully.

### The "A component was suspended" Error

This error occurs when a component suspends (e.g., when lazy loading) during a synchronous update, like during initial render or an event handler. React needs to know how to handle this suspension, which is what Suspense provides.

The error message suggests using `startTransition` for updates that might suspend, which is another React feature that helps manage asynchronous updates by marking them as non-urgent.

## Custom Hooks

Custom Hooks are JavaScript functions that use React Hooks and can be shared across components. They allow you to extract component logic into reusable functions.

### Online Status Hook

We created a custom hook to check if the user is online:

```jsx
import { useEffect, useState } from 'react';

const useOnlineStatus = () => {
  const [onlineStatus, setOnlineStatus] = useState(true);

  useEffect(() => {
    window.addEventListener('offline', () => {
      setOnlineStatus(false);
    });

    window.addEventListener('online', () => {
      setOnlineStatus(true);
    });
  }, []);

  return onlineStatus;
};

export default useOnlineStatus;
```

Using the hook in a component:

```jsx
import useOnlineStatus from '../utils/useOnlineStatus';

const Header = () => {
  const onlineStatus = useOnlineStatus();
  
  return (
    // ...
    <li>Online Status: {onlineStatus ? '✅' : '⛔'}</li>
    // ...
  );
};
```

### Restaurant Menu Hook

We also created a hook to fetch restaurant menu data:

```jsx
import { useEffect, useState } from 'react';
import { MENU_API } from '../utils/constants';

const useRestaurantMenu = (resId) => {
  const [resInfo, setResInfo] = useState(null);

  useEffect(() => {
    fetchData();
  }, []);

  const fetchData = async () => {
    const data = await fetch(MENU_API + resId);
    const json = await data.json();
    setResInfo(json.data);
  };

  return resInfo;
};

export default useRestaurantMenu;
```

Using the hook in the RestaurantMenu component:

```jsx
import useRestaurantMenu from '../utils/useRestaurantMenu';
import { useParams } from 'react-router-dom';

const RestaurantMenu = () => {
  const { resId } = useParams();
  const resInfo = useRestaurantMenu(resId);

  if (resInfo === null) return <ShimmerMenu />;
  
  // Rest of the component
};
```

## Advantages of Custom Hooks

1. **Reusability**: Logic can be shared across components
2. **Separation of Concerns**: UI and logic are separated
3. **Cleaner Components**: Components focus on rendering, not logic
4. **Easier Testing**: Logic can be tested independently
5. **Composition**: Multiple hooks can be combined for complex behavior

## Implementation in Our App

### Adding Lazy Loading to the Router

We updated our router configuration to use lazy loading for the Grocery component:

```jsx
const appRouter = createBrowserRouter([
  {
    path: '/',
    element: <AppLayout />,
    children: [
      // Other routes...
      {
        path: '/grocery',
        element: (
          <Suspense fallback={<h1>Loading...</h1>}>
            <Grocery />
          </Suspense>
        ),
      },
    ],
    errorElement: <Error />,
  },
]);
```

### Displaying Online Status

We added an online status indicator to the Header component:

```jsx
const Header = () => {
  const onlineStatus = useOnlineStatus();
  
  return (
    <div className="header">
      {/* Logo */}
      <div className="nav-items">
        <ul>
          <li>Online Status: {onlineStatus ? '✅' : '⛔'}</li>
          {/* Other nav items */}
        </ul>
      </div>
    </div>
  );
};
```

### Simplifying RestaurantMenu with a Custom Hook

We replaced the direct API fetch logic in the RestaurantMenu component with our custom hook:

```jsx
// Before: Component had API fetch logic
useEffect(() => {
  fetchMenu();
}, []);

const fetchMenu = async () => {
  const data = await fetch(MENU_API + resId);
  const json = await data.json();
  setResInfo(json.data);
};

// After: Using custom hook
const resInfo = useRestaurantMenu(resId);
```

## Advantages and Disadvantages of Code Splitting

### Advantages

1. **Faster Initial Load**: Smaller initial bundle size means faster loading
2. **Reduced Memory Usage**: Only load what's needed
3. **Better Caching**: Smaller, more focused bundles can be cached more effectively
4. **Better User Experience**: App becomes interactive faster
5. **On-Demand Loading**: Features are loaded only when needed

### Disadvantages

1. **Additional Complexity**: More moving parts to manage
2. **Potential for Too Many Requests**: If not done carefully, could lead to too many small chunks
3. **Loading States**: Need to handle loading states for lazy-loaded components
4. **Browser Support**: Older browsers might need polyfills
5. **SEO Considerations**: Content in lazy-loaded components might not be immediately available for crawlers

## When to Use Code Splitting

Code splitting is most beneficial for:

1. **Large Components**: Components with a lot of code
2. **Infrequently Used Features**: Features that aren't needed by most users
3. **Heavy Libraries**: Third-party libraries that are only used in certain parts of your app
4. **Routes**: Different routes in your application
5. **Modal Windows and Dialogs**: UI elements that aren't immediately visible

For our app, we applied code splitting to the Grocery route because:
- It's a separate section of our app
- It's not part of the core restaurant browsing experience
- It potentially contains many components that aren't needed for other parts of the app

## Best Practices for Optimization

1. **Route-Based Code Splitting**: Split code at the route level first
2. **Component-Based Code Splitting**: Then consider heavy components
3. **Strategic Suspense Placement**: Place Suspense boundaries wisely
4. **Custom Hooks for Shared Logic**: Extract and reuse logic with custom hooks
5. **Performance Monitoring**: Measure the impact of optimizations
6. **Avoid Over-Optimization**: Focus on areas with measurable impact

## Summary

In this episode, we:

1. Implemented code splitting using React.lazy()
2. Used Suspense to handle loading states
3. Created custom hooks for online status and restaurant menu data
4. Applied these optimizations to our food delivery app
5. Learned when and why to use these techniques

These optimizations help make our application more performant and maintainable by:
- Reducing initial load time
- Extracting and reusing logic
- Handling network status
- Simplifying components

By applying these advanced React techniques, we've taken our app to the next level in terms of both performance and code quality.
