# Understanding useEffect and Shimmer UI in React

## Table of Contents
1. [useEffect Hook](#useeffect-hook)
   - [Introduction to useEffect](#introduction-to-useeffect)
   - [Basic Syntax](#basic-syntax)
   - [Dependency Array](#dependency-array)
   - [Cleanup Function](#cleanup-function)
   - [Common Use Cases](#common-use-cases)
   - [Best Practices](#useeffect-best-practices)
   - [Examples](#useeffect-examples)

2. [Shimmer UI](#shimmer-ui)
   - [What is Shimmer UI?](#what-is-shimmer-ui)
   - [Benefits of Shimmer UI](#benefits-of-shimmer-ui)
   - [Implementation Strategies](#implementation-strategies)
   - [Examples](#shimmer-ui-examples)
   - [Best Practices](#shimmer-ui-best-practices)

## useEffect Hook

### Introduction to useEffect

The `useEffect` hook is one of the most important hooks in React's functional components. It allows you to perform side effects in your components. Side effects are operations that affect something outside the scope of the function being executed, such as:

- Data fetching
- DOM manipulation
- Subscription setup and cleanup
- Timer functions
- Logging

`useEffect` essentially replaces several lifecycle methods that were used in class components, including:
- `componentDidMount`
- `componentDidUpdate`
- `componentWillUnmount`

By using this hook, you can handle side effects in a more organized and declarative way.

### Basic Syntax

The basic syntax of `useEffect` is:

```jsx
import React, { useEffect } from 'react';

function MyComponent() {
  useEffect(() => {
    // Code to run after render
    // This is where side effects go
    
    // Optional: return a cleanup function
    return () => {
      // Code for cleanup
    };
  }, [/* dependency array */]);
  
  return <div>My Component</div>;
}
```

### Dependency Array

The second argument to `useEffect` is the dependency array, which controls when the effect should run:

1. **No dependency array provided**: The effect runs after every render
   ```jsx
   useEffect(() => {
     console.log('This runs after every render');
   });
   ```

2. **Empty dependency array `[]`**: The effect runs only once after the initial render (similar to `componentDidMount`)
   ```jsx
   useEffect(() => {
     console.log('This runs only once after the initial render');
   }, []);
   ```

3. **Array with dependencies**: The effect runs after the initial render and whenever any of the dependencies change
   ```jsx
   useEffect(() => {
     console.log(`This runs when count changes: ${count}`);
   }, [count]);
   ```

### Cleanup Function

The function returned from the effect is called the cleanup function. It runs before the component unmounts and before the effect runs again (if it depends on changing values).

```jsx
useEffect(() => {
  // Setup code (e.g., add event listener)
  const handler = () => console.log('Window resized');
  window.addEventListener('resize', handler);
  
  // Cleanup function
  return () => {
    window.removeEventListener('resize', handler);
  };
}, []);
```

This cleanup function is crucial for:
- Preventing memory leaks
- Canceling network requests
- Clearing timers
- Unsubscribing from subscriptions

### Common Use Cases

#### 1. Data Fetching

```jsx
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    const fetchUser = async () => {
      setLoading(true);
      try {
        const response = await fetch(`https://api.example.com/users/${userId}`);
        const data = await response.json();
        setUser(data);
      } catch (error) {
        console.error('Error fetching user:', error);
      } finally {
        setLoading(false);
      }
    };
    
    fetchUser();
    
    // Cleanup function to handle component unmounting during fetch
    return () => {
      // This could be used with an AbortController to cancel the fetch
    };
  }, [userId]); // Re-run the effect when userId changes
  
  if (loading) return <div>Loading...</div>;
  if (!user) return <div>User not found</div>;
  
  return (
    <div>
      <h1>{user.name}</h1>
      <p>Email: {user.email}</p>
    </div>
  );
}
```

#### 2. Subscriptions

```jsx
function ChatRoom({ roomId }) {
  const [messages, setMessages] = useState([]);
  
  useEffect(() => {
    const subscription = chatService.subscribe(roomId, (message) => {
      setMessages(prev => [...prev, message]);
    });
    
    // Cleanup function to unsubscribe when component unmounts or roomId changes
    return () => {
      subscription.unsubscribe();
    };
  }, [roomId]);
  
  return (
    <div>
      <h2>Chat Room: {roomId}</h2>
      <ul>
        {messages.map((msg, index) => (
          <li key={index}>{msg.text}</li>
        ))}
      </ul>
    </div>
  );
}
```

#### 3. DOM Manipulation

```jsx
function AutoFocusInput() {
  const inputRef = useRef(null);
  
  useEffect(() => {
    // Focus the input element after component mounts
    if (inputRef.current) {
      inputRef.current.focus();
    }
  }, []); // Empty dependency array means this runs once after initial render
  
  return <input ref={inputRef} type="text" />;
}
```

#### 4. Timer Functions

```jsx
function Countdown({ seconds }) {
  const [timeLeft, setTimeLeft] = useState(seconds);
  
  useEffect(() => {
    if (timeLeft <= 0) return;
    
    const timerId = setInterval(() => {
      setTimeLeft(time => time - 1);
    }, 1000);
    
    // Clear the interval when component unmounts or when timeLeft changes
    return () => clearInterval(timerId);
  }, [timeLeft]);
  
  return <div>Time remaining: {timeLeft} seconds</div>;
}
```

### useEffect Best Practices

1. **Keep effects focused on a single concern**
   - Split unrelated code into separate `useEffect` calls

2. **Properly specify dependencies**
   - Don't lie to React about dependencies
   - Include all values from the component scope that change over time

3. **Handle cleanup properly**
   - Always clean up subscriptions, timers, and event listeners

4. **Use functional updates for state**
   - When the new state depends on the previous state, use the functional form of `setState`
   ```jsx
   useEffect(() => {
     const timer = setInterval(() => {
       setCount(prevCount => prevCount + 1); // Functional update
     }, 1000);
     return () => clearInterval(timer);
   }, []);
   ```

5. **Use refs for values you want to read in effects without triggering re-runs**
   ```jsx
   function Example() {
     const [count, setCount] = useState(0);
     const latestCount = useRef(count);
     
     useEffect(() => {
       latestCount.current = count;
     });
     
     useEffect(() => {
       const interval = setInterval(() => {
         console.log(`Current count: ${latestCount.current}`);
       }, 1000);
       return () => clearInterval(interval);
     }, []); // Empty deps means this only runs once
     
     return <button onClick={() => setCount(count + 1)}>Increment</button>;
   }
   ```

### useEffect Examples

#### Syncing with External Systems

```jsx
function ThemeToggler() {
  const [darkMode, setDarkMode] = useState(false);
  
  useEffect(() => {
    // Apply theme to document body
    document.body.classList.toggle('dark-theme', darkMode);
    
    // Store user preference
    localStorage.setItem('darkMode', darkMode.toString());
  }, [darkMode]);
  
  return (
    <button onClick={() => setDarkMode(!darkMode)}>
      {darkMode ? 'Switch to Light Mode' : 'Switch to Dark Mode'}
    </button>
  );
}
```

#### Handling Window Events

```jsx
function WindowSizeTracker() {
  const [windowSize, setWindowSize] = useState({
    width: window.innerWidth,
    height: window.innerHeight
  });
  
  useEffect(() => {
    function handleResize() {
      setWindowSize({
        width: window.innerWidth,
        height: window.innerHeight
      });
    }
    
    window.addEventListener('resize', handleResize);
    
    return () => window.removeEventListener('resize', handleResize);
  }, []);
  
  return (
    <div>
      <p>Window width: {windowSize.width}px</p>
      <p>Window height: {windowSize.height}px</p>
    </div>
  );
}
```

## Shimmer UI

### What is Shimmer UI?

Shimmer UI (also known as skeleton screens) is a design technique used to indicate a loading state in an application. Rather than showing traditional loading spinners, Shimmer UI displays a placeholder that resembles the page's layout before the content has fully loaded.

The "shimmer" or "pulse" effect is an animation that moves across these placeholder elements, giving users a sense that the application is actively loading content. This approach:

1. Provides immediate visual feedback
2. Creates a perception of speed
3. Reduces perceived wait time
4. Gives users a preview of the content structure

### Benefits of Shimmer UI

1. **Improved User Experience**
   - Reduces perceived loading time
   - Provides context about the upcoming content
   - Feels more responsive than traditional loaders

2. **Reduced Cognitive Load**
   - Users can mentally prepare for the content layout
   - Minimizes layout shifts when content loads
   - Decreases the jarring effect of sudden content appearance

3. **Enhanced Perception of Performance**
   - Creates an illusion of speed
   - Makes the application feel faster even if actual load times are the same
   - Reduces user abandonment during loading

### Implementation Strategies

#### 1. CSS-Based Shimmer Effect

The shimmer effect can be created using CSS animations with linear gradients:

```css
.shimmer-element {
  background: #f6f7f8;
  background-image: linear-gradient(
    to right,
    #f6f7f8 0%,
    #edeef1 20%,
    #f6f7f8 40%,
    #f6f7f8 100%
  );
  background-repeat: no-repeat;
  background-size: 800px 104px;
  animation-duration: 1.5s;
  animation-fill-mode: forwards;
  animation-iteration-count: infinite;
  animation-name: shimmer;
  animation-timing-function: linear;
}

@keyframes shimmer {
  0% {
    background-position: -468px 0;
  }
  100% {
    background-position: 468px 0;
  }
}
```

#### 2. React Implementation with Custom Components

```jsx
// ShimmerCard.jsx
import React from 'react';
import './Shimmer.css';

const ShimmerCard = () => {
  return (
    <div className="shimmer-card">
      <div className="shimmer-img shimmer-element"></div>
      <div className="shimmer-title shimmer-element"></div>
      <div className="shimmer-text shimmer-element"></div>
      <div className="shimmer-text shimmer-element" style={{ width: '70%' }}></div>
    </div>
  );
};

export default ShimmerCard;
```

#### 3. Implementation with Libraries

Several libraries provide ready-to-use shimmer components:

**Using react-loading-skeleton:**

```jsx
import Skeleton from 'react-loading-skeleton';
import 'react-loading-skeleton/dist/skeleton.css';

function ProductCardSkeleton() {
  return (
    <div className="product-card">
      <Skeleton height={200} width="100%" />
      <h2><Skeleton width="70%" /></h2>
      <p><Skeleton count={3} /></p>
      <div className="price">
        <Skeleton width={100} />
      </div>
    </div>
  );
}

// Usage in a component
function ProductList() {
  const [products, setProducts] = useState([]);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    fetch('https://api.example.com/products')
      .then(res => res.json())
      .then(data => {
        setProducts(data);
        setLoading(false);
      });
  }, []);
  
  return (
    <div className="product-list">
      {loading ? (
        // Show 6 skeleton cards while loading
        Array(6).fill().map((_, index) => (
          <ProductCardSkeleton key={index} />
        ))
      ) : (
        // Show actual product cards when data is loaded
        products.map(product => (
          <ProductCard key={product.id} product={product} />
        ))
      )}
    </div>
  );
}
```

### Shimmer UI Examples

#### Restaurant Card Shimmer (for food delivery app)

```jsx
// RestaurantCardShimmer.jsx
import React from 'react';
import './Shimmer.css';

const RestaurantCardShimmer = () => {
  return (
    <div className="restaurant-card-shimmer">
      <div className="shimmer-img shimmer-element"></div>
      <div className="shimmer-title shimmer-element"></div>
      <div className="shimmer-tags shimmer-element"></div>
      <div className="shimmer-details">
        <div className="shimmer-rating shimmer-element"></div>
        <div className="shimmer-distance shimmer-element"></div>
        <div className="shimmer-time shimmer-element"></div>
      </div>
    </div>
  );
};

// ShimmerList component to show multiple shimmer cards
const ShimmerList = () => {
  return (
    <div className="restaurant-list">
      {Array(8).fill().map((_, index) => (
        <RestaurantCardShimmer key={index} />
      ))}
    </div>
  );
};

export default ShimmerList;
```

#### Social Media Post Shimmer

```jsx
// PostShimmer.jsx
import React from 'react';
import './Shimmer.css';

const PostShimmer = () => {
  return (
    <div className="post-shimmer">
      <div className="post-header">
        <div className="shimmer-avatar shimmer-element"></div>
        <div className="shimmer-user-info">
          <div className="shimmer-name shimmer-element"></div>
          <div className="shimmer-timestamp shimmer-element"></div>
        </div>
      </div>
      <div className="shimmer-content">
        <div className="shimmer-text shimmer-element"></div>
        <div className="shimmer-text shimmer-element"></div>
        <div className="shimmer-text shimmer-element" style={{ width: '60%' }}></div>
      </div>
      <div className="shimmer-image shimmer-element"></div>
      <div className="shimmer-actions">
        <div className="shimmer-action shimmer-element"></div>
        <div className="shimmer-action shimmer-element"></div>
        <div className="shimmer-action shimmer-element"></div>
      </div>
    </div>
  );
};

export default PostShimmer;
```

#### Complete Example with useEffect and Shimmer UI

```jsx
import React, { useState, useEffect } from 'react';
import './App.css';

// Shimmer component
const ProductCardShimmer = () => {
  return (
    <div className="product-card shimmer">
      <div className="shimmer-img shimmer-element"></div>
      <div className="shimmer-title shimmer-element"></div>
      <div className="shimmer-price shimmer-element"></div>
      <div className="shimmer-button shimmer-element"></div>
    </div>
  );
};

// Actual product card component
const ProductCard = ({ product }) => {
  return (
    <div className="product-card">
      <img src={product.image} alt={product.title} />
      <h3>{product.title}</h3>
      <p>${product.price}</p>
      <button>Add to Cart</button>
    </div>
  );
};

// Main component
function App() {
  const [products, setProducts] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    // Simulate a realistic API fetch delay
    const fetchProducts = async () => {
      try {
        // Artificial delay to demonstrate shimmer UI (remove in production)
        await new Promise(resolve => setTimeout(resolve, 2000));
        
        const response = await fetch('https://fakestoreapi.com/products?limit=8');
        if (!response.ok) {
          throw new Error('Network response was not ok');
        }
        
        const data = await response.json();
        setProducts(data);
        setLoading(false);
      } catch (error) {
        setError(error.message);
        setLoading(false);
      }
    };

    fetchProducts();
    
    // Cleanup function
    return () => {
      // This could abort the fetch request in a real application
      console.log('Component unmounted, cleanup performed');
    };
  }, []); // Empty dependency array means this effect runs once after initial render

  if (error) {
    return <div className="error-message">Error: {error}</div>;
  }

  return (
    <div className="App">
      <header>
        <h1>Product Catalog</h1>
      </header>
      
      <div className="product-grid">
        {loading ? (
          // Show shimmer UI while loading
          Array(8).fill().map((_, index) => (
            <ProductCardShimmer key={index} />
          ))
        ) : (
          // Show actual products when loaded
          products.map(product => (
            <ProductCard key={product.id} product={product} />
          ))
        )}
      </div>
    </div>
  );
}

export default App;
```

### Shimmer UI Best Practices

1. **Match the Skeleton to Real Content**
   - The shimmer UI should closely resemble the actual content layout
   - Use similar sizing and proportions to minimize layout shifts

2. **Keep Animations Subtle**
   - The shimmer effect should be gentle and not distracting
   - Use subtle gradients and smooth transitions

3. **Show Progressive Loading**
   - Consider loading critical UI elements first
   - Implement a cascading effect where different parts load sequentially

4. **Handle Different Screen Sizes**
   - Make your shimmer UI responsive to match your actual content's behavior
   - Test on various device sizes

5. **Set Reasonable Timeouts**
   - If loading takes too long, consider showing an error or alternate message
   - Don't let users stare at shimmer effects indefinitely

6. **Accessibility Considerations**
   - Ensure screen readers can understand that content is loading
   - Consider using aria-busy="true" on loading containers

7. **Performance**
   - Use hardware-accelerated animations (transform, opacity) for better performance
   - Minimize DOM nodes for complex shimmer UI patterns

---

By implementing useEffect for handling side effects and Shimmer UI for loading states, you can create React applications that are both functionally powerful and provide an excellent user experience during data loading and processing.
