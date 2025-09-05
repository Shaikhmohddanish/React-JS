# React Router - A Complete Guide

## Introduction to React Router

React Router is a standard library for routing in React applications. It enables navigation among views in a React application, allowing developers to build single-page applications with navigation without the page refreshing as the user navigates through the app. React Router uses component structure to call components, which display the appropriate user interface when the URL matches the route path.

## Why Routing Matters in React Applications

Understanding routing is essential for building modern web applications because it:

1. **Improves User Experience**: Enables navigation without page reloads, making the application feel smoother and more responsive
2. **Organizes Application Structure**: Helps divide your application into logical sections and views
3. **Supports Bookmarking**: Allows users to bookmark specific states of your application
4. **Enables Sharing Links**: Makes it possible to share direct links to specific pages or views
5. **Facilitates Deep Linking**: Allows users to navigate directly to nested content
6. **Supports Navigation History**: Works with the browser's history API for back/forward navigation
7. **Enables Code Splitting**: Allows for loading components only when they're needed, improving performance

## Getting Started with React Router

### Installation

To add React Router to your project, install it using npm or yarn:

```bash
npm install react-router-dom
```

or

```bash
yarn add react-router-dom
```

### Basic Setup

Here's a basic example of setting up React Router in a React application:

```jsx
import React from 'react';
import { BrowserRouter, Routes, Route } from 'react-router-dom';

// Import your components
import Home from './components/Home';
import About from './components/About';
import Contact from './components/Contact';
import NotFound from './components/NotFound';
import Navbar from './components/Navbar';

function App() {
  return (
    <BrowserRouter>
      <div className="app">
        <Navbar />
        <div className="content">
          <Routes>
            <Route path="/" element={<Home />} />
            <Route path="/about" element={<About />} />
            <Route path="/contact" element={<Contact />} />
            <Route path="*" element={<NotFound />} />
          </Routes>
        </div>
      </div>
    </BrowserRouter>
  );
}

export default App;
```

### Creating Navigation Links

The `Link` component is used to create links to different routes:

```jsx
import { Link } from 'react-router-dom';

function Navbar() {
  return (
    <nav>
      <ul>
        <li>
          <Link to="/">Home</Link>
        </li>
        <li>
          <Link to="/about">About</Link>
        </li>
        <li>
          <Link to="/contact">Contact</Link>
        </li>
      </ul>
    </nav>
  );
}

export default Navbar;
```

## Core Components of React Router

### 1. Router Components

React Router provides several router components:

#### BrowserRouter

Uses the HTML5 history API to keep your UI in sync with the URL:

```jsx
import { BrowserRouter } from 'react-router-dom';

function App() {
  return (
    <BrowserRouter>
      {/* Your app components */}
    </BrowserRouter>
  );
}
```

#### HashRouter

Uses the hash portion of the URL (`window.location.hash`) to keep your UI in sync with the URL:

```jsx
import { HashRouter } from 'react-router-dom';

function App() {
  return (
    <HashRouter>
      {/* Your app components */}
    </HashRouter>
  );
}
```

#### MemoryRouter

Keeps the history of your "URL" in memory (doesn't read or write to the address bar). Useful for testing and non-browser environments:

```jsx
import { MemoryRouter } from 'react-router-dom';

function App() {
  return (
    <MemoryRouter>
      {/* Your app components */}
    </MemoryRouter>
  );
}
```

### 2. Route Components

#### Routes and Route

The `Routes` component is used to wrap multiple `Route` components, which define the mapping between URL paths and components:

```jsx
import { Routes, Route } from 'react-router-dom';

function App() {
  return (
    <Routes>
      <Route path="/" element={<Home />} />
      <Route path="/about" element={<About />} />
      <Route path="/contact" element={<Contact />} />
    </Routes>
  );
}
```

### 3. Navigation Components

#### Link

Creates a link to a different route without reloading the page:

```jsx
import { Link } from 'react-router-dom';

function Navbar() {
  return (
    <nav>
      <Link to="/">Home</Link>
      <Link to="/about">About</Link>
    </nav>
  );
}
```

#### NavLink

A special version of `Link` that adds styling attributes to the rendered element when it matches the current URL:

```jsx
import { NavLink } from 'react-router-dom';

function Navbar() {
  return (
    <nav>
      <NavLink to="/" className={({ isActive }) => 
        isActive ? "active-link" : "link"
      }>
        Home
      </NavLink>
      <NavLink to="/about" className={({ isActive }) => 
        isActive ? "active-link" : "link"
      }>
        About
      </NavLink>
    </nav>
  );
}
```

#### Navigate

Used for programmatic navigation:

```jsx
import { Navigate } from 'react-router-dom';

function ProtectedRoute({ isAuthenticated, children }) {
  if (!isAuthenticated) {
    return <Navigate to="/login" replace />;
  }
  
  return children;
}
```

## Advanced Routing Techniques

### 1. URL Parameters

URL parameters allow you to capture values from the URL:

```jsx
import { Routes, Route, useParams } from 'react-router-dom';

function ProductDetail() {
  const { id } = useParams();
  
  return (
    <div>
      <h2>Product Details</h2>
      <p>Product ID: {id}</p>
      {/* Fetch and display product details based on the ID */}
    </div>
  );
}

function App() {
  return (
    <Routes>
      <Route path="/" element={<Home />} />
      <Route path="/products/:id" element={<ProductDetail />} />
    </Routes>
  );
}
```

To link to a product:

```jsx
<Link to={`/products/${product.id}`}>{product.name}</Link>
```

### 2. Nested Routes

React Router allows you to nest routes within other routes:

```jsx
import { Routes, Route, Outlet } from 'react-router-dom';

function Dashboard() {
  return (
    <div>
      <h1>Dashboard</h1>
      <nav>
        <Link to="/dashboard/stats">Stats</Link>
        <Link to="/dashboard/settings">Settings</Link>
      </nav>
      
      {/* The Outlet component renders the matching child route */}
      <Outlet />
    </div>
  );
}

function App() {
  return (
    <Routes>
      <Route path="/" element={<Home />} />
      <Route path="/dashboard" element={<Dashboard />}>
        <Route index element={<DashboardHome />} />
        <Route path="stats" element={<Stats />} />
        <Route path="settings" element={<Settings />} />
      </Route>
    </Routes>
  );
}
```

### 3. Index Routes

Index routes render in the parent's outlet at the parent's URL:

```jsx
<Routes>
  <Route path="/dashboard" element={<Dashboard />}>
    <Route index element={<DashboardHome />} />
    <Route path="stats" element={<Stats />} />
  </Route>
</Routes>
```

In this example, when the URL is "/dashboard", it will render both the `Dashboard` component and the `DashboardHome` component.

### 4. Route Layouts and Outlet

The `Outlet` component is used to render child routes:

```jsx
import { Outlet } from 'react-router-dom';

function Layout() {
  return (
    <div>
      <header>
        <nav>{/* Navigation links */}</nav>
      </header>
      
      <main>
        {/* Child routes will be rendered here */}
        <Outlet />
      </main>
      
      <footer>
        {/* Footer content */}
      </footer>
    </div>
  );
}

function App() {
  return (
    <Routes>
      <Route path="/" element={<Layout />}>
        <Route index element={<Home />} />
        <Route path="/about" element={<About />} />
        <Route path="/contact" element={<Contact />} />
      </Route>
    </Routes>
  );
}
```

### 5. Multiple Routes Layouts

You can have different layouts for different sections of your app:

```jsx
function App() {
  return (
    <Routes>
      {/* Main public routes with main layout */}
      <Route path="/" element={<MainLayout />}>
        <Route index element={<Home />} />
        <Route path="about" element={<About />} />
        <Route path="contact" element={<Contact />} />
      </Route>
      
      {/* Dashboard routes with dashboard layout */}
      <Route path="/dashboard" element={<DashboardLayout />}>
        <Route index element={<DashboardHome />} />
        <Route path="stats" element={<Stats />} />
        <Route path="settings" element={<Settings />} />
      </Route>
      
      {/* Auth routes with minimal layout */}
      <Route path="/auth" element={<AuthLayout />}>
        <Route path="login" element={<Login />} />
        <Route path="register" element={<Register />} />
        <Route path="forgot-password" element={<ForgotPassword />} />
      </Route>
    </Routes>
  );
}
```

## Navigation and Redirection

### 1. Programmatic Navigation with useNavigate

The `useNavigate` hook provides programmatic navigation:

```jsx
import { useNavigate } from 'react-router-dom';

function LoginForm() {
  const navigate = useNavigate();
  
  const handleSubmit = async (e) => {
    e.preventDefault();
    
    try {
      // Perform login API call
      await loginUser(formData);
      
      // Redirect to dashboard after successful login
      navigate('/dashboard');
    } catch (error) {
      // Handle error
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      {/* Form fields */}
      <button type="submit">Log In</button>
    </form>
  );
}
```

You can also pass options to `navigate`:

```jsx
// Replace the current entry in the history stack
navigate('/dashboard', { replace: true });

// Pass state to the new location
navigate('/dashboard', { state: { from: 'login' } });
```

### 2. Redirects with Navigate Component

For declarative redirects:

```jsx
import { Navigate } from 'react-router-dom';

function ProtectedRoute({ isAuthenticated, children }) {
  if (!isAuthenticated) {
    // Redirect to login if not authenticated
    return <Navigate to="/login" replace />;
  }
  
  return children;
}

function App() {
  const isAuthenticated = checkAuth(); // Your auth check logic
  
  return (
    <Routes>
      <Route path="/login" element={<Login />} />
      <Route 
        path="/dashboard" 
        element={
          <ProtectedRoute isAuthenticated={isAuthenticated}>
            <Dashboard />
          </ProtectedRoute>
        } 
      />
    </Routes>
  );
}
```

### 3. Handling Navigation Events

```jsx
import { useNavigationType, useLocation } from 'react-router-dom';

function NavigationLogger() {
  const navigationType = useNavigationType(); // Returns 'POP', 'PUSH', or 'REPLACE'
  const location = useLocation();
  
  useEffect(() => {
    console.log(`Navigation type: ${navigationType}`);
    console.log('Current location:', location);
    
    // Analytics tracking example
    if (navigationType !== 'POP') {
      trackPageView(location.pathname);
    }
  }, [location, navigationType]);
  
  return null; // This is a utility component that doesn't render anything
}
```

## Route Params and Query Params

### 1. URL Parameters with useParams

```jsx
import { useParams } from 'react-router-dom';

function UserProfile() {
  const { userId } = useParams();
  
  // Fetch user data based on userId
  
  return (
    <div>
      <h1>User Profile</h1>
      <p>User ID: {userId}</p>
      {/* Display user information */}
    </div>
  );
}

// In your routes configuration:
<Route path="/users/:userId" element={<UserProfile />} />
```

### 2. Optional Parameters

```jsx
<Route path="/products/:category?/:productId?" element={<Products />} />
```

In the component:

```jsx
function Products() {
  const { category, productId } = useParams();
  
  if (productId) {
    return <ProductDetail productId={productId} />;
  }
  
  if (category) {
    return <ProductCategory category={category} />;
  }
  
  return <AllProducts />;
}
```

### 3. Query Parameters with useSearchParams

```jsx
import { useSearchParams } from 'react-router-dom';

function ProductList() {
  const [searchParams, setSearchParams] = useSearchParams();
  
  const page = parseInt(searchParams.get('page') || '1', 10);
  const sort = searchParams.get('sort') || 'name';
  const filter = searchParams.get('filter') || '';
  
  // Use page, sort, and filter to fetch and display products
  
  const goToNextPage = () => {
    setSearchParams({ page: (page + 1).toString(), sort, filter });
  };
  
  const updateSort = (newSort) => {
    setSearchParams({ page: page.toString(), sort: newSort, filter });
  };
  
  const updateFilter = (newFilter) => {
    setSearchParams({ page: '1', sort, filter: newFilter });
  };
  
  return (
    <div>
      <h1>Products</h1>
      
      <div className="filters">
        <input 
          type="text" 
          placeholder="Filter products..." 
          value={filter}
          onChange={(e) => updateFilter(e.target.value)}
        />
        
        <select 
          value={sort} 
          onChange={(e) => updateSort(e.target.value)}
        >
          <option value="name">Sort by Name</option>
          <option value="price">Sort by Price</option>
          <option value="rating">Sort by Rating</option>
        </select>
      </div>
      
      {/* Product list */}
      
      <div className="pagination">
        <button 
          disabled={page === 1}
          onClick={() => setSearchParams({ page: (page - 1).toString(), sort, filter })}
        >
          Previous
        </button>
        <span>Page {page}</span>
        <button onClick={goToNextPage}>Next</button>
      </div>
    </div>
  );
}
```

### 4. Accessing Location State

```jsx
import { useLocation } from 'react-router-dom';

function Dashboard() {
  const location = useLocation();
  const { from } = location.state || { from: 'direct access' };
  
  return (
    <div>
      <h1>Dashboard</h1>
      <p>You came from: {from}</p>
    </div>
  );
}

// Navigating with state
navigate('/dashboard', { state: { from: 'login' } });
```

## Protected Routes and Authentication

### 1. Basic Protected Route

```jsx
import { Navigate, useLocation } from 'react-router-dom';

function RequireAuth({ children }) {
  const isAuthenticated = useAuth(); // Your auth hook
  const location = useLocation();
  
  if (!isAuthenticated) {
    // Redirect to login and remember where they were trying to go
    return <Navigate to="/login" state={{ from: location }} replace />;
  }
  
  return children;
}

function App() {
  return (
    <Routes>
      <Route path="/login" element={<Login />} />
      <Route path="/register" element={<Register />} />
      
      <Route
        path="/dashboard"
        element={
          <RequireAuth>
            <Dashboard />
          </RequireAuth>
        }
      />
      
      <Route
        path="/profile"
        element={
          <RequireAuth>
            <UserProfile />
          </RequireAuth>
        }
      />
    </Routes>
  );
}
```

### 2. Role-Based Access Control

```jsx
function RequireRole({ children, requiredRole }) {
  const { user, isAuthenticated } = useAuth();
  const location = useLocation();
  
  if (!isAuthenticated) {
    return <Navigate to="/login" state={{ from: location }} replace />;
  }
  
  if (user.role !== requiredRole) {
    return <Navigate to="/unauthorized" replace />;
  }
  
  return children;
}

function App() {
  return (
    <Routes>
      <Route path="/login" element={<Login />} />
      <Route path="/unauthorized" element={<Unauthorized />} />
      
      {/* Admin routes */}
      <Route
        path="/admin"
        element={
          <RequireRole requiredRole="admin">
            <AdminDashboard />
          </RequireRole>
        }
      />
      
      {/* User routes */}
      <Route
        path="/dashboard"
        element={
          <RequireRole requiredRole="user">
            <UserDashboard />
          </RequireRole>
        }
      />
    </Routes>
  );
}
```

### 3. Redirecting After Login

```jsx
import { useNavigate, useLocation } from 'react-router-dom';

function Login() {
  const navigate = useNavigate();
  const location = useLocation();
  
  // Get the redirect path from location state, or default to homepage
  const from = location.state?.from?.pathname || '/';
  
  const handleLogin = async (formData) => {
    try {
      await loginUser(formData);
      
      // Redirect to the originally requested page
      navigate(from, { replace: true });
    } catch (error) {
      // Handle login error
    }
  };
  
  return (
    <div>
      <h1>Login</h1>
      <LoginForm onSubmit={handleLogin} />
    </div>
  );
}
```

## Loading and Error States

### 1. Loading States During Navigation

```jsx
import { Suspense } from 'react';
import { Routes, Route } from 'react-router-dom';

// Lazy load components
const Dashboard = React.lazy(() => import('./Dashboard'));
const UserProfile = React.lazy(() => import('./UserProfile'));
const Settings = React.lazy(() => import('./Settings'));

function LoadingSpinner() {
  return <div className="spinner">Loading...</div>;
}

function App() {
  return (
    <Suspense fallback={<LoadingSpinner />}>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/dashboard" element={<Dashboard />} />
        <Route path="/profile" element={<UserProfile />} />
        <Route path="/settings" element={<Settings />} />
      </Routes>
    </Suspense>
  );
}
```

### 2. Data Loading with React Router Loaders (v6.4+)

```jsx
import {
  createBrowserRouter,
  RouterProvider,
  useLoaderData
} from 'react-router-dom';

// Loader function
async function dashboardLoader() {
  const stats = await fetchDashboardStats();
  const notifications = await fetchNotifications();
  
  return { stats, notifications };
}

function Dashboard() {
  // Access the loaded data
  const { stats, notifications } = useLoaderData();
  
  return (
    <div>
      <h1>Dashboard</h1>
      <StatsDisplay stats={stats} />
      <NotificationList notifications={notifications} />
    </div>
  );
}

// Create router with loaders
const router = createBrowserRouter([
  {
    path: '/',
    element: <Home />
  },
  {
    path: '/dashboard',
    element: <Dashboard />,
    loader: dashboardLoader
  }
]);

function App() {
  return <RouterProvider router={router} />;
}
```

### 3. Error Handling with ErrorBoundary

```jsx
import { ErrorBoundary } from 'react-error-boundary';

function ErrorFallback({ error, resetErrorBoundary }) {
  return (
    <div className="error-container">
      <h2>Something went wrong</h2>
      <pre>{error.message}</pre>
      <button onClick={resetErrorBoundary}>Try again</button>
    </div>
  );
}

function App() {
  return (
    <ErrorBoundary FallbackComponent={ErrorFallback}>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/dashboard" element={<Dashboard />} />
        <Route path="/profile" element={<UserProfile />} />
      </Routes>
    </ErrorBoundary>
  );
}
```

### 4. Route-Specific Error Handling (v6.4+)

```jsx
// Error boundary component
function DashboardError() {
  const error = useRouteError();
  
  return (
    <div className="error-container">
      <h2>Dashboard Error</h2>
      <p>{error.message || 'An unexpected error occurred'}</p>
      <Link to="/">Go to Home</Link>
    </div>
  );
}

// Create router with error elements
const router = createBrowserRouter([
  {
    path: '/',
    element: <Home />
  },
  {
    path: '/dashboard',
    element: <Dashboard />,
    loader: dashboardLoader,
    errorElement: <DashboardError />
  }
]);
```

## Code Splitting and Lazy Loading

React Router works well with React's lazy loading for code splitting:

```jsx
import React, { Suspense, lazy } from 'react';
import { BrowserRouter, Routes, Route } from 'react-router-dom';

// Lazy load components
const Home = lazy(() => import('./components/Home'));
const About = lazy(() => import('./components/About'));
const Contact = lazy(() => import('./components/Contact'));
const Dashboard = lazy(() => import('./components/Dashboard'));

function App() {
  return (
    <BrowserRouter>
      <Suspense fallback={<div>Loading...</div>}>
        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/about" element={<About />} />
          <Route path="/contact" element={<Contact />} />
          <Route path="/dashboard/*" element={<Dashboard />} />
        </Routes>
      </Suspense>
    </BrowserRouter>
  );
}
```

## Advanced Topics

### 1. Custom Routing Hooks

Create custom hooks for common routing patterns:

```jsx
// useConfirmNavigation.js
import { useCallback, useEffect } from 'react';
import { useBlocker } from 'react-router-dom';

export function useConfirmNavigation(
  when = true,
  message = 'Are you sure you want to leave? Your changes may not be saved.'
) {
  const blocker = useBlocker(when);
  
  const handleBeforeUnload = useCallback(
    (event) => {
      if (when) {
        event.preventDefault();
        event.returnValue = message;
        return message;
      }
    },
    [when, message]
  );
  
  useEffect(() => {
    if (when) {
      window.addEventListener('beforeunload', handleBeforeUnload);
      
      return () => {
        window.removeEventListener('beforeunload', handleBeforeUnload);
      };
    }
  }, [when, handleBeforeUnload]);
  
  useEffect(() => {
    if (blocker.state === 'blocked') {
      const confirmed = window.confirm(message);
      
      if (confirmed) {
        blocker.proceed();
      } else {
        blocker.reset();
      }
    }
  }, [blocker, message]);
}

// Usage
function ContactForm() {
  const [formData, setFormData] = useState(initialFormData);
  const [isDirty, setIsDirty] = useState(false);
  
  // Prompt user if they try to navigate away with unsaved changes
  useConfirmNavigation(isDirty);
  
  // Rest of component
}
```

### 2. Scroll Restoration

Restore scroll position when navigating:

```jsx
import { useEffect } from 'react';
import { useLocation } from 'react-router-dom';

function ScrollToTop() {
  const { pathname } = useLocation();
  
  useEffect(() => {
    window.scrollTo(0, 0);
  }, [pathname]);
  
  return null;
}

function App() {
  return (
    <BrowserRouter>
      <ScrollToTop />
      {/* Your routes */}
    </BrowserRouter>
  );
}
```

### 3. Animated Transitions

Use a library like Framer Motion with React Router:

```jsx
import { AnimatePresence, motion } from 'framer-motion';
import { Routes, Route, useLocation } from 'react-router-dom';

function AnimatedRoutes() {
  const location = useLocation();
  
  return (
    <AnimatePresence mode="wait">
      <Routes location={location} key={location.pathname}>
        <Route
          path="/"
          element={
            <motion.div
              initial={{ opacity: 0, y: 20 }}
              animate={{ opacity: 1, y: 0 }}
              exit={{ opacity: 0, y: -20 }}
              transition={{ duration: 0.3 }}
            >
              <Home />
            </motion.div>
          }
        />
        <Route
          path="/about"
          element={
            <motion.div
              initial={{ opacity: 0, y: 20 }}
              animate={{ opacity: 1, y: 0 }}
              exit={{ opacity: 0, y: -20 }}
              transition={{ duration: 0.3 }}
            >
              <About />
            </motion.div>
          }
        />
        {/* Other routes */}
      </Routes>
    </AnimatePresence>
  );
}
```

### 4. Handling 404 Pages

Create a catch-all route for 404 pages:

```jsx
function NotFound() {
  return (
    <div className="not-found">
      <h1>404 - Page Not Found</h1>
      <p>The page you are looking for does not exist.</p>
      <Link to="/">Go to Home</Link>
    </div>
  );
}

function App() {
  return (
    <Routes>
      <Route path="/" element={<Home />} />
      <Route path="/about" element={<About />} />
      <Route path="/contact" element={<Contact />} />
      {/* This will match when no other routes match */}
      <Route path="*" element={<NotFound />} />
    </Routes>
  );
}
```

## Complete Application Example

Let's put everything together in a complete example:

```jsx
import React, { Suspense, lazy, useState, useEffect } from 'react';
import {
  BrowserRouter,
  Routes,
  Route,
  Link,
  NavLink,
  Outlet,
  useParams,
  useNavigate,
  useLocation,
  Navigate
} from 'react-router-dom';

// Lazy loaded components
const Home = lazy(() => import('./pages/Home'));
const About = lazy(() => import('./pages/About'));
const Contact = lazy(() => import('./pages/Contact'));
const ProductList = lazy(() => import('./pages/ProductList'));
const ProductDetail = lazy(() => import('./pages/ProductDetail'));
const Dashboard = lazy(() => import('./pages/Dashboard'));
const Profile = lazy(() => import('./pages/Profile'));
const Settings = lazy(() => import('./pages/Settings'));
const Login = lazy(() => import('./pages/Login'));
const Register = lazy(() => import('./pages/Register'));
const NotFound = lazy(() => import('./pages/NotFound'));

// Auth context
const AuthContext = React.createContext();

function AuthProvider({ children }) {
  const [user, setUser] = useState(null);
  
  // Check for saved user on mount
  useEffect(() => {
    const savedUser = localStorage.getItem('user');
    if (savedUser) {
      setUser(JSON.parse(savedUser));
    }
  }, []);
  
  const login = (userData) => {
    setUser(userData);
    localStorage.setItem('user', JSON.stringify(userData));
  };
  
  const logout = () => {
    setUser(null);
    localStorage.removeItem('user');
  };
  
  const value = {
    user,
    isAuthenticated: !!user,
    login,
    logout
  };
  
  return (
    <AuthContext.Provider value={value}>
      {children}
    </AuthContext.Provider>
  );
}

// Custom hook to use auth context
function useAuth() {
  return React.useContext(AuthContext);
}

// Protected route component
function RequireAuth({ children }) {
  const { isAuthenticated } = useAuth();
  const location = useLocation();
  
  if (!isAuthenticated) {
    return <Navigate to="/login" state={{ from: location }} replace />;
  }
  
  return children;
}

// Layout components
function MainLayout() {
  const { isAuthenticated, logout } = useAuth();
  const navigate = useNavigate();
  
  const handleLogout = () => {
    logout();
    navigate('/login');
  };
  
  return (
    <div className="app">
      <header className="app-header">
        <div className="logo">
          <Link to="/">MyApp</Link>
        </div>
        
        <nav className="main-nav">
          <NavLink to="/" className={({ isActive }) => 
            isActive ? 'nav-link active' : 'nav-link'
          }>
            Home
          </NavLink>
          <NavLink to="/products" className={({ isActive }) => 
            isActive ? 'nav-link active' : 'nav-link'
          }>
            Products
          </NavLink>
          <NavLink to="/about" className={({ isActive }) => 
            isActive ? 'nav-link active' : 'nav-link'
          }>
            About
          </NavLink>
          <NavLink to="/contact" className={({ isActive }) => 
            isActive ? 'nav-link active' : 'nav-link'
          }>
            Contact
          </NavLink>
        </nav>
        
        <div className="auth-nav">
          {isAuthenticated ? (
            <>
              <NavLink to="/dashboard" className={({ isActive }) => 
                isActive ? 'nav-link active' : 'nav-link'
              }>
                Dashboard
              </NavLink>
              <button onClick={handleLogout} className="logout-btn">
                Logout
              </button>
            </>
          ) : (
            <>
              <NavLink to="/login" className={({ isActive }) => 
                isActive ? 'nav-link active' : 'nav-link'
              }>
                Login
              </NavLink>
              <NavLink to="/register" className={({ isActive }) => 
                isActive ? 'nav-link active' : 'nav-link'
              }>
                Register
              </NavLink>
            </>
          )}
        </div>
      </header>
      
      <main className="app-main">
        <Suspense fallback={<div className="loading">Loading...</div>}>
          <Outlet />
        </Suspense>
      </main>
      
      <footer className="app-footer">
        <p>&copy; 2025 MyApp. All rights reserved.</p>
      </footer>
    </div>
  );
}

function DashboardLayout() {
  return (
    <div className="dashboard">
      <aside className="dashboard-sidebar">
        <nav>
          <NavLink to="/dashboard" end className={({ isActive }) => 
            isActive ? 'nav-link active' : 'nav-link'
          }>
            Dashboard Home
          </NavLink>
          <NavLink to="/dashboard/profile" className={({ isActive }) => 
            isActive ? 'nav-link active' : 'nav-link'
          }>
            Profile
          </NavLink>
          <NavLink to="/dashboard/settings" className={({ isActive }) => 
            isActive ? 'nav-link active' : 'nav-link'
          }>
            Settings
          </NavLink>
        </nav>
      </aside>
      
      <div className="dashboard-content">
        <Suspense fallback={<div className="loading">Loading...</div>}>
          <Outlet />
        </Suspense>
      </div>
    </div>
  );
}

// Scroll to top on navigation
function ScrollToTop() {
  const { pathname } = useLocation();
  
  useEffect(() => {
    window.scrollTo(0, 0);
  }, [pathname]);
  
  return null;
}

function App() {
  return (
    <AuthProvider>
      <BrowserRouter>
        <ScrollToTop />
        <Routes>
          {/* Main routes with main layout */}
          <Route path="/" element={<MainLayout />}>
            <Route index element={<Home />} />
            <Route path="about" element={<About />} />
            <Route path="contact" element={<Contact />} />
            
            <Route path="products">
              <Route index element={<ProductList />} />
              <Route path=":productId" element={<ProductDetail />} />
            </Route>
            
            {/* Auth routes */}
            <Route path="login" element={<Login />} />
            <Route path="register" element={<Register />} />
            
            {/* Dashboard routes (protected) */}
            <Route
              path="dashboard"
              element={
                <RequireAuth>
                  <DashboardLayout />
                </RequireAuth>
              }
            >
              <Route index element={<Dashboard />} />
              <Route path="profile" element={<Profile />} />
              <Route path="settings" element={<Settings />} />
            </Route>
            
            {/* 404 route */}
            <Route path="*" element={<NotFound />} />
          </Route>
        </Routes>
      </BrowserRouter>
    </AuthProvider>
  );
}

export default App;
```

### Product Detail Page Example

```jsx
import React, { useEffect, useState } from 'react';
import { useParams, useNavigate, Link } from 'react-router-dom';

function ProductDetail() {
  const { productId } = useParams();
  const navigate = useNavigate();
  const [product, setProduct] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    async function fetchProduct() {
      try {
        setLoading(true);
        // In a real app, fetch from an API
        const response = await fetch(`/api/products/${productId}`);
        
        if (!response.ok) {
          throw new Error('Product not found');
        }
        
        const data = await response.json();
        setProduct(data);
        setLoading(false);
      } catch (err) {
        setError(err.message);
        setLoading(false);
      }
    }
    
    fetchProduct();
  }, [productId]);
  
  const handleGoBack = () => {
    navigate(-1); // Go back to previous page
  };
  
  if (loading) {
    return <div className="loading">Loading product...</div>;
  }
  
  if (error) {
    return (
      <div className="error">
        <h2>Error</h2>
        <p>{error}</p>
        <button onClick={handleGoBack}>Go Back</button>
      </div>
    );
  }
  
  return (
    <div className="product-detail">
      <button onClick={handleGoBack} className="back-button">
        &larr; Back
      </button>
      
      <div className="product-content">
        <div className="product-image">
          <img src={product.image} alt={product.name} />
        </div>
        
        <div className="product-info">
          <h1>{product.name}</h1>
          <p className="product-price">${product.price.toFixed(2)}</p>
          <div className="product-rating">
            Rating: {product.rating}/5 ({product.reviews} reviews)
          </div>
          
          <p className="product-description">{product.description}</p>
          
          <div className="product-actions">
            <button className="add-to-cart-btn">Add to Cart</button>
            <button className="buy-now-btn">Buy Now</button>
          </div>
        </div>
      </div>
      
      <div className="related-products">
        <h2>Related Products</h2>
        <div className="product-grid">
          {product.relatedProducts.map(relatedProduct => (
            <div key={relatedProduct.id} className="product-card">
              <Link to={`/products/${relatedProduct.id}`}>
                <img src={relatedProduct.image} alt={relatedProduct.name} />
                <h3>{relatedProduct.name}</h3>
                <p>${relatedProduct.price.toFixed(2)}</p>
              </Link>
            </div>
          ))}
        </div>
      </div>
    </div>
  );
}

export default ProductDetail;
```

### Login Page Example

```jsx
import React, { useState } from 'react';
import { useNavigate, useLocation, Link } from 'react-router-dom';
import { useAuth } from '../contexts/AuthContext';

function Login() {
  const navigate = useNavigate();
  const location = useLocation();
  const { login } = useAuth();
  
  const [formData, setFormData] = useState({
    email: '',
    password: ''
  });
  const [errors, setErrors] = useState({});
  const [isLoading, setIsLoading] = useState(false);
  
  // Get the redirect path from location state, or default to homepage
  const from = location.state?.from?.pathname || '/dashboard';
  
  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormData(prevData => ({
      ...prevData,
      [name]: value
    }));
    
    // Clear error when user starts typing
    if (errors[name]) {
      setErrors(prevErrors => ({
        ...prevErrors,
        [name]: null
      }));
    }
  };
  
  const validate = () => {
    const newErrors = {};
    
    if (!formData.email) {
      newErrors.email = 'Email is required';
    } else if (!/\S+@\S+\.\S+/.test(formData.email)) {
      newErrors.email = 'Email is invalid';
    }
    
    if (!formData.password) {
      newErrors.password = 'Password is required';
    } else if (formData.password.length < 6) {
      newErrors.password = 'Password must be at least 6 characters';
    }
    
    setErrors(newErrors);
    return Object.keys(newErrors).length === 0;
  };
  
  const handleSubmit = async (e) => {
    e.preventDefault();
    
    if (!validate()) {
      return;
    }
    
    try {
      setIsLoading(true);
      
      // In a real app, you would call an API here
      // Simulating API call with timeout
      await new Promise(resolve => setTimeout(resolve, 1000));
      
      // Simulate successful login
      login({
        id: 1,
        name: 'John Doe',
        email: formData.email,
        role: 'user'
      });
      
      // Redirect to the originally requested page
      navigate(from, { replace: true });
    } catch (error) {
      setErrors({
        form: 'Invalid email or password'
      });
      setIsLoading(false);
    }
  };
  
  return (
    <div className="login-page">
      <div className="login-container">
        <h1>Login</h1>
        
        {errors.form && (
          <div className="error-message">{errors.form}</div>
        )}
        
        <form onSubmit={handleSubmit}>
          <div className="form-group">
            <label htmlFor="email">Email</label>
            <input
              type="email"
              id="email"
              name="email"
              value={formData.email}
              onChange={handleChange}
              className={errors.email ? 'error' : ''}
              disabled={isLoading}
            />
            {errors.email && (
              <div className="error-message">{errors.email}</div>
            )}
          </div>
          
          <div className="form-group">
            <label htmlFor="password">Password</label>
            <input
              type="password"
              id="password"
              name="password"
              value={formData.password}
              onChange={handleChange}
              className={errors.password ? 'error' : ''}
              disabled={isLoading}
            />
            {errors.password && (
              <div className="error-message">{errors.password}</div>
            )}
          </div>
          
          <button 
            type="submit" 
            className="submit-btn"
            disabled={isLoading}
          >
            {isLoading ? 'Logging in...' : 'Login'}
          </button>
        </form>
        
        <div className="login-footer">
          <p>
            Don't have an account? <Link to="/register">Register</Link>
          </p>
          <p>
            <Link to="/forgot-password">Forgot Password?</Link>
          </p>
        </div>
      </div>
    </div>
  );
}

export default Login;
```

## Best Practices and Tips

1. **Organize Routes Logically**: Group related routes together and use nested routes for better organization.

2. **Use Layouts**: Create layout components for different sections of your app to avoid repeating common UI elements.

3. **Component-Based Navigation**: Keep navigation components separate and reusable.

4. **Handle Loading States**: Always provide feedback during navigation and data loading.

5. **Error Handling**: Implement proper error handling for routes and data fetching.

6. **Use TypeScript**: Add type safety to your routing for better development experience:

```tsx
interface RouteParams {
  productId: string;
  categoryId?: string;
}

function ProductDetail() {
  const { productId, categoryId } = useParams<RouteParams>();
  // ...
}
```

7. **Test Routes**: Write tests for your routes and navigation logic:

```jsx
import { render, screen } from '@testing-library/react';
import { MemoryRouter } from 'react-router-dom';
import App from './App';

test('renders home page on default route', () => {
  render(
    <MemoryRouter initialEntries={['/']}>
      <App />
    </MemoryRouter>
  );
  
  expect(screen.getByText(/welcome to the home page/i)).toBeInTheDocument();
});

test('renders about page on /about route', () => {
  render(
    <MemoryRouter initialEntries={['/about']}>
      <App />
    </MemoryRouter>
  );
  
  expect(screen.getByText(/about us/i)).toBeInTheDocument();
});
```

8. **Code Splitting**: Use lazy loading for better performance, especially for large applications.

9. **Keep URLs Clean**: Design URLs to be descriptive, hierarchical, and shareable.

10. **Use Query Parameters for Filtering/Sorting**: Keep filter and sort state in the URL for shareable and bookmarkable pages.

## Summary

React Router is a powerful and flexible library for handling routing in React applications. It provides a declarative way to define your application's routes and navigation structure.

Key concepts covered in this guide:

1. **Basic Routing**: Setting up BrowserRouter, Routes, Route, and Link components
2. **Navigation**: Using Link, NavLink, and programmatic navigation with useNavigate
3. **Route Parameters**: Capturing dynamic segments from the URL with useParams
4. **Nested Routes**: Organizing routes hierarchically with parent/child relationships
5. **Layouts**: Creating consistent layouts with Outlet
6. **Protected Routes**: Implementing authentication and authorization
7. **Query Parameters**: Managing filter, sort, and pagination state
8. **Loading States**: Handling loading and error states during navigation
9. **Code Splitting**: Optimizing performance with lazy loading
10. **Advanced Topics**: Scroll restoration, animated transitions, and custom hooks

By mastering React Router, you can create sophisticated, multi-page applications with seamless navigation while maintaining the performance benefits of a single-page application.

## Practice Exercise

Build a small e-commerce application with the following routes and features:

1. Home page with featured products
2. Product listing page with filtering and sorting
3. Product detail page with dynamic route parameters
4. Shopping cart page
5. Checkout process with multiple steps using nested routes
6. User authentication (login/register)
7. Protected user dashboard with profile and order history
8. 404 page for non-existent routes
9. Implement loading states for all data fetching
10. Add route transitions using your preferred animation library

This exercise will help you apply the concepts covered in this guide to a real-world application scenario.

## Additional Resources

- [React Router Official Documentation](https://reactrouter.com/en/main)
- [React Router GitHub Repository](https://github.com/remix-run/react-router)
- [React Router v6 Tutorial](https://reactrouter.com/en/main/start/tutorial)
- [React Router v6 Migration Guide](https://reactrouter.com/en/main/upgrading/v5)
