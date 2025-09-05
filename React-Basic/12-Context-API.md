# Context API - A Complete Guide

## Introduction to Context API

The React Context API is a powerful feature that allows you to share data across multiple components without having to pass props down through every level of the component tree. This pattern is known as "prop drilling" and can make your code difficult to maintain as your application grows. Context provides a way to create a "global" state that can be accessed by any component in the tree.

## Why Context API Matters

Understanding and properly using the Context API is crucial because:

1. **Eliminates Prop Drilling**: No need to pass props through multiple component levels
2. **Global State Management**: Share data across distant components easily
3. **Theme Management**: Perfect for implementing light/dark themes
4. **User Authentication**: Manage user state throughout the application
5. **Localization**: Handle language and region settings globally
6. **Performance**: When used correctly, it can improve performance by reducing unnecessary re-renders

## When to Use Context

Context should be used when:
- Data needs to be accessible by many components at different nesting levels
- You want to avoid prop drilling
- The data is relatively stable (doesn't change frequently)
- You're implementing features like themes, authentication, or localization

Context should NOT be used when:
- The data changes frequently (can cause performance issues)
- Only a few components need the data
- Simple prop passing would suffice

## Basic Context Usage

### Creating a Context

```jsx
import React, { createContext, useContext, useState } from 'react';

// Create a context
const UserContext = createContext();

// Create a provider component
function UserProvider({ children }) {
  const [user, setUser] = useState(null);
  const [isAuthenticated, setIsAuthenticated] = useState(false);
  
  const login = (userData) => {
    setUser(userData);
    setIsAuthenticated(true);
  };
  
  const logout = () => {
    setUser(null);
    setIsAuthenticated(false);
  };
  
  const value = {
    user,
    isAuthenticated,
    login,
    logout
  };
  
  return (
    <UserContext.Provider value={value}>
      {children}
    </UserContext.Provider>
  );
}

// Custom hook to use the context
function useUser() {
  const context = useContext(UserContext);
  
  if (!context) {
    throw new Error('useUser must be used within a UserProvider');
  }
  
  return context;
}

// Component that uses the context
function UserProfile() {
  const { user, isAuthenticated, logout } = useUser();
  
  if (!isAuthenticated) {
    return <div>Please log in to view your profile</div>;
  }
  
  return (
    <div>
      <h2>Welcome, {user.name}!</h2>
      <p>Email: {user.email}</p>
      <button onClick={logout}>Logout</button>
    </div>
  );
}

// Login component
function LoginForm() {
  const { login, isAuthenticated } = useUser();
  const [formData, setFormData] = useState({ email: '', password: '' });
  
  const handleSubmit = (e) => {
    e.preventDefault();
    // Simulate login
    login({
      name: 'John Doe',
      email: formData.email
    });
  };
  
  if (isAuthenticated) {
    return <div>You are already logged in!</div>;
  }
  
  return (
    <form onSubmit={handleSubmit}>
      <h2>Login</h2>
      <input
        type="email"
        placeholder="Email"
        value={formData.email}
        onChange={(e) => setFormData({ ...formData, email: e.target.value })}
        required
      />
      <input
        type="password"
        placeholder="Password"
        value={formData.password}
        onChange={(e) => setFormData({ ...formData, password: e.target.value })}
        required
      />
      <button type="submit">Login</button>
    </form>
  );
}

// Main App component
function App() {
  return (
    <UserProvider>
      <div className="app">
        <header>
          <h1>My Application</h1>
        </header>
        <main>
          <LoginForm />
          <UserProfile />
        </main>
      </div>
    </UserProvider>
  );
}

export default App;
```

## Context with useReducer

For more complex state management, combine Context with useReducer:

```jsx
import React, { createContext, useContext, useReducer } from 'react';

// Action types
const CART_ACTIONS = {
  ADD_ITEM: 'ADD_ITEM',
  REMOVE_ITEM: 'REMOVE_ITEM',
  UPDATE_QUANTITY: 'UPDATE_QUANTITY',
  CLEAR_CART: 'CLEAR_CART',
  APPLY_DISCOUNT: 'APPLY_DISCOUNT'
};

// Reducer function
function cartReducer(state, action) {
  switch (action.type) {
    case CART_ACTIONS.ADD_ITEM: {
      const existingItem = state.items.find(item => item.id === action.payload.id);
      
      if (existingItem) {
        return {
          ...state,
          items: state.items.map(item =>
            item.id === action.payload.id
              ? { ...item, quantity: item.quantity + 1 }
              : item
          )
        };
      }
      
      return {
        ...state,
        items: [...state.items, { ...action.payload, quantity: 1 }]
      };
    }
    
    case CART_ACTIONS.REMOVE_ITEM:
      return {
        ...state,
        items: state.items.filter(item => item.id !== action.payload)
      };
    
    case CART_ACTIONS.UPDATE_QUANTITY: {
      const { id, quantity } = action.payload;
      
      if (quantity <= 0) {
        return {
          ...state,
          items: state.items.filter(item => item.id !== id)
        };
      }
      
      return {
        ...state,
        items: state.items.map(item =>
          item.id === id ? { ...item, quantity } : item
        )
      };
    }
    
    case CART_ACTIONS.CLEAR_CART:
      return {
        ...state,
        items: [],
        discount: 0
      };
    
    case CART_ACTIONS.APPLY_DISCOUNT:
      return {
        ...state,
        discount: action.payload
      };
    
    default:
      return state;
  }
}

// Initial state
const initialState = {
  items: [],
  discount: 0
};

// Create context
const CartContext = createContext();

// Cart provider
function CartProvider({ children }) {
  const [state, dispatch] = useReducer(cartReducer, initialState);
  
  // Action creators
  const addItem = (item) => {
    dispatch({ type: CART_ACTIONS.ADD_ITEM, payload: item });
  };
  
  const removeItem = (id) => {
    dispatch({ type: CART_ACTIONS.REMOVE_ITEM, payload: id });
  };
  
  const updateQuantity = (id, quantity) => {
    dispatch({ type: CART_ACTIONS.UPDATE_QUANTITY, payload: { id, quantity } });
  };
  
  const clearCart = () => {
    dispatch({ type: CART_ACTIONS.CLEAR_CART });
  };
  
  const applyDiscount = (discount) => {
    dispatch({ type: CART_ACTIONS.APPLY_DISCOUNT, payload: discount });
  };
  
  // Computed values
  const getTotalItems = () => {
    return state.items.reduce((total, item) => total + item.quantity, 0);
  };
  
  const getSubtotal = () => {
    return state.items.reduce((total, item) => total + (item.price * item.quantity), 0);
  };
  
  const getTotal = () => {
    const subtotal = getSubtotal();
    return subtotal - (subtotal * state.discount / 100);
  };
  
  const value = {
    items: state.items,
    discount: state.discount,
    addItem,
    removeItem,
    updateQuantity,
    clearCart,
    applyDiscount,
    getTotalItems,
    getSubtotal,
    getTotal
  };
  
  return (
    <CartContext.Provider value={value}>
      {children}
    </CartContext.Provider>
  );
}

// Custom hook
function useCart() {
  const context = useContext(CartContext);
  
  if (!context) {
    throw new Error('useCart must be used within a CartProvider');
  }
  
  return context;
}

// Product component
function Product({ product }) {
  const { addItem } = useCart();
  
  return (
    <div className="product">
      <img src={product.image} alt={product.name} />
      <h3>{product.name}</h3>
      <p>${product.price.toFixed(2)}</p>
      <button onClick={() => addItem(product)}>
        Add to Cart
      </button>
    </div>
  );
}

// Cart summary component
function CartSummary() {
  const { items, getTotalItems, getSubtotal, getTotal, discount } = useCart();
  
  return (
    <div className="cart-summary">
      <h3>Cart Summary</h3>
      <p>Items: {getTotalItems()}</p>
      <p>Subtotal: ${getSubtotal().toFixed(2)}</p>
      {discount > 0 && (
        <p>Discount ({discount}%): -${(getSubtotal() * discount / 100).toFixed(2)}</p>
      )}
      <p><strong>Total: ${getTotal().toFixed(2)}</strong></p>
    </div>
  );
}

// Cart items component
function CartItems() {
  const { items, updateQuantity, removeItem } = useCart();
  
  if (items.length === 0) {
    return <div>Your cart is empty</div>;
  }
  
  return (
    <div className="cart-items">
      <h3>Cart Items</h3>
      {items.map(item => (
        <div key={item.id} className="cart-item">
          <span>{item.name}</span>
          <span>${item.price.toFixed(2)}</span>
          <input
            type="number"
            min="0"
            value={item.quantity}
            onChange={(e) => updateQuantity(item.id, parseInt(e.target.value) || 0)}
          />
          <span>${(item.price * item.quantity).toFixed(2)}</span>
          <button onClick={() => removeItem(item.id)}>Remove</button>
        </div>
      ))}
    </div>
  );
}

// Discount component
function DiscountCode() {
  const { applyDiscount, discount } = useCart();
  const [code, setCode] = useState('');
  
  const handleApplyDiscount = () => {
    // Simple discount logic
    const discounts = {
      'SAVE10': 10,
      'SAVE20': 20,
      'WELCOME': 15
    };
    
    const discountValue = discounts[code.toUpperCase()];
    if (discountValue) {
      applyDiscount(discountValue);
      setCode('');
      alert(`Discount applied: ${discountValue}%`);
    } else {
      alert('Invalid discount code');
    }
  };
  
  return (
    <div className="discount-code">
      <h3>Discount Code</h3>
      <input
        type="text"
        value={code}
        onChange={(e) => setCode(e.target.value)}
        placeholder="Enter discount code"
      />
      <button onClick={handleApplyDiscount}>Apply</button>
      {discount > 0 && <p>Current discount: {discount}%</p>}
    </div>
  );
}

// Shopping app
function ShoppingApp() {
  const products = [
    { id: 1, name: 'Laptop', price: 999.99, image: '/laptop.jpg' },
    { id: 2, name: 'Phone', price: 699.99, image: '/phone.jpg' },
    { id: 3, name: 'Headphones', price: 199.99, image: '/headphones.jpg' }
  ];
  
  return (
    <CartProvider>
      <div className="shopping-app">
        <header>
          <h1>Online Store</h1>
          <CartSummary />
        </header>
        
        <main>
          <section className="products">
            <h2>Products</h2>
            <div className="product-grid">
              {products.map(product => (
                <Product key={product.id} product={product} />
              ))}
            </div>
          </section>
          
          <section className="cart">
            <CartItems />
            <DiscountCode />
          </section>
        </main>
      </div>
    </CartProvider>
  );
}
```

## Theme Context Example

A common use case for Context is theme management:

```jsx
import React, { createContext, useContext, useState, useEffect } from 'react';

// Theme configuration
const themes = {
  light: {
    name: 'light',
    colors: {
      primary: '#007bff',
      secondary: '#6c757d',
      background: '#ffffff',
      surface: '#f8f9fa',
      text: '#212529',
      textSecondary: '#6c757d',
      border: '#dee2e6'
    },
    spacing: {
      small: '8px',
      medium: '16px',
      large: '24px'
    }
  },
  dark: {
    name: 'dark',
    colors: {
      primary: '#0d6efd',
      secondary: '#adb5bd',
      background: '#121212',
      surface: '#1e1e1e',
      text: '#ffffff',
      textSecondary: '#adb5bd',
      border: '#404040'
    },
    spacing: {
      small: '8px',
      medium: '16px',
      large: '24px'
    }
  }
};

// Create theme context
const ThemeContext = createContext();

// Theme provider
function ThemeProvider({ children }) {
  const [currentTheme, setCurrentTheme] = useState('light');
  
  // Load theme from localStorage on mount
  useEffect(() => {
    const savedTheme = localStorage.getItem('theme');
    if (savedTheme && themes[savedTheme]) {
      setCurrentTheme(savedTheme);
    }
  }, []);
  
  // Save theme to localStorage when it changes
  useEffect(() => {
    localStorage.setItem('theme', currentTheme);
    
    // Apply theme to CSS custom properties
    const theme = themes[currentTheme];
    const root = document.documentElement;
    
    Object.entries(theme.colors).forEach(([key, value]) => {
      root.style.setProperty(`--color-${key}`, value);
    });
    
    Object.entries(theme.spacing).forEach(([key, value]) => {
      root.style.setProperty(`--spacing-${key}`, value);
    });
  }, [currentTheme]);
  
  const toggleTheme = () => {
    setCurrentTheme(prev => prev === 'light' ? 'dark' : 'light');
  };
  
  const setTheme = (themeName) => {
    if (themes[themeName]) {
      setCurrentTheme(themeName);
    }
  };
  
  const value = {
    theme: themes[currentTheme],
    themeName: currentTheme,
    toggleTheme,
    setTheme,
    availableThemes: Object.keys(themes)
  };
  
  return (
    <ThemeContext.Provider value={value}>
      {children}
    </ThemeContext.Provider>
  );
}

// Custom hook
function useTheme() {
  const context = useContext(ThemeContext);
  
  if (!context) {
    throw new Error('useTheme must be used within a ThemeProvider');
  }
  
  return context;
}

// Styled component using theme
function Button({ children, variant = 'primary', ...props }) {
  const { theme } = useTheme();
  
  const styles = {
    backgroundColor: theme.colors[variant],
    color: variant === 'primary' ? '#ffffff' : theme.colors.text,
    border: `1px solid ${theme.colors.border}`,
    padding: `${theme.spacing.small} ${theme.spacing.medium}`,
    borderRadius: '4px',
    cursor: 'pointer',
    fontSize: '14px'
  };
  
  return (
    <button style={styles} {...props}>
      {children}
    </button>
  );
}

// Card component using theme
function Card({ title, children }) {
  const { theme } = useTheme();
  
  const styles = {
    backgroundColor: theme.colors.surface,
    color: theme.colors.text,
    border: `1px solid ${theme.colors.border}`,
    borderRadius: '8px',
    padding: theme.spacing.large,
    margin: theme.spacing.medium
  };
  
  return (
    <div style={styles}>
      {title && <h3 style={{ margin: 0, marginBottom: theme.spacing.medium }}>{title}</h3>}
      {children}
    </div>
  );
}

// Theme toggle component
function ThemeToggle() {
  const { themeName, toggleTheme } = useTheme();
  
  return (
    <Button onClick={toggleTheme}>
      Switch to {themeName === 'light' ? 'dark' : 'light'} theme
    </Button>
  );
}

// Main app component
function ThemedApp() {
  const { theme } = useTheme();
  
  const appStyles = {
    backgroundColor: theme.colors.background,
    color: theme.colors.text,
    minHeight: '100vh',
    padding: theme.spacing.large
  };
  
  return (
    <div style={appStyles}>
      <header>
        <h1>Themed Application</h1>
        <ThemeToggle />
      </header>
      
      <main>
        <Card title="Welcome">
          <p>This is a themed application using React Context.</p>
          <p>The theme is applied globally and persisted in localStorage.</p>
        </Card>
        
        <Card title="Features">
          <ul>
            <li>Light and dark themes</li>
            <li>Persistent theme selection</li>
            <li>CSS custom properties integration</li>
            <li>Consistent styling across components</li>
          </ul>
        </Card>
        
        <Card title="Actions">
          <Button variant="primary" style={{ marginRight: '8px' }}>
            Primary Button
          </Button>
          <Button variant="secondary">
            Secondary Button
          </Button>
        </Card>
      </main>
    </div>
  );
}

// Root app with provider
function App() {
  return (
    <ThemeProvider>
      <ThemedApp />
    </ThemeProvider>
  );
}
```

## Multiple Contexts

Sometimes you need multiple contexts in your application:

```jsx
import React, { createContext, useContext, useState, useReducer } from 'react';

// User Context
const UserContext = createContext();

function UserProvider({ children }) {
  const [user, setUser] = useState(null);
  const [isAuthenticated, setIsAuthenticated] = useState(false);
  
  const login = (userData) => {
    setUser(userData);
    setIsAuthenticated(true);
  };
  
  const logout = () => {
    setUser(null);
    setIsAuthenticated(false);
  };
  
  return (
    <UserContext.Provider value={{ user, isAuthenticated, login, logout }}>
      {children}
    </UserContext.Provider>
  );
}

// Settings Context
const SettingsContext = createContext();

function settingsReducer(state, action) {
  switch (action.type) {
    case 'SET_LANGUAGE':
      return { ...state, language: action.payload };
    case 'SET_NOTIFICATIONS':
      return { ...state, notifications: action.payload };
    case 'SET_PRIVACY':
      return { ...state, privacy: { ...state.privacy, ...action.payload } };
    case 'RESET_SETTINGS':
      return {
        language: 'en',
        notifications: { email: true, push: false },
        privacy: { publicProfile: false, shareData: false }
      };
    default:
      return state;
  }
}

function SettingsProvider({ children }) {
  const [settings, dispatch] = useReducer(settingsReducer, {
    language: 'en',
    notifications: { email: true, push: false },
    privacy: { publicProfile: false, shareData: false }
  });
  
  const updateLanguage = (language) => {
    dispatch({ type: 'SET_LANGUAGE', payload: language });
  };
  
  const updateNotifications = (notifications) => {
    dispatch({ type: 'SET_NOTIFICATIONS', payload: notifications });
  };
  
  const updatePrivacy = (privacy) => {
    dispatch({ type: 'SET_PRIVACY', payload: privacy });
  };
  
  const resetSettings = () => {
    dispatch({ type: 'RESET_SETTINGS' });
  };
  
  return (
    <SettingsContext.Provider value={{
      settings,
      updateLanguage,
      updateNotifications,
      updatePrivacy,
      resetSettings
    }}>
      {children}
    </SettingsContext.Provider>
  );
}

// Notification Context
const NotificationContext = createContext();

function NotificationProvider({ children }) {
  const [notifications, setNotifications] = useState([]);
  
  const addNotification = (message, type = 'info') => {
    const id = Date.now();
    const notification = { id, message, type, timestamp: new Date() };
    
    setNotifications(prev => [...prev, notification]);
    
    // Auto-remove after 5 seconds
    setTimeout(() => {
      removeNotification(id);
    }, 5000);
  };
  
  const removeNotification = (id) => {
    setNotifications(prev => prev.filter(n => n.id !== id));
  };
  
  const clearAll = () => {
    setNotifications([]);
  };
  
  return (
    <NotificationContext.Provider value={{
      notifications,
      addNotification,
      removeNotification,
      clearAll
    }}>
      {children}
    </NotificationContext.Provider>
  );
}

// Custom hooks
function useUser() {
  const context = useContext(UserContext);
  if (!context) {
    throw new Error('useUser must be used within a UserProvider');
  }
  return context;
}

function useSettings() {
  const context = useContext(SettingsContext);
  if (!context) {
    throw new Error('useSettings must be used within a SettingsProvider');
  }
  return context;
}

function useNotifications() {
  const context = useContext(NotificationContext);
  if (!context) {
    throw new Error('useNotifications must be used within a NotificationProvider');
  }
  return context;
}

// Components using multiple contexts
function UserDashboard() {
  const { user, isAuthenticated, logout } = useUser();
  const { settings } = useSettings();
  const { addNotification } = useNotifications();
  
  const handleLogout = () => {
    logout();
    addNotification('Successfully logged out', 'success');
  };
  
  if (!isAuthenticated) {
    return <div>Please log in to access the dashboard</div>;
  }
  
  return (
    <div>
      <h2>Dashboard</h2>
      <p>Welcome, {user.name}!</p>
      <p>Language: {settings.language}</p>
      <p>Email notifications: {settings.notifications.email ? 'Enabled' : 'Disabled'}</p>
      <button onClick={handleLogout}>Logout</button>
    </div>
  );
}

function SettingsPanel() {
  const { settings, updateLanguage, updateNotifications } = useSettings();
  const { addNotification } = useNotifications();
  
  const handleLanguageChange = (language) => {
    updateLanguage(language);
    addNotification(`Language changed to ${language}`, 'success');
  };
  
  const handleNotificationToggle = (type) => {
    const newNotifications = {
      ...settings.notifications,
      [type]: !settings.notifications[type]
    };
    updateNotifications(newNotifications);
    addNotification('Notification settings updated', 'success');
  };
  
  return (
    <div>
      <h3>Settings</h3>
      
      <div>
        <label>Language:</label>
        <select
          value={settings.language}
          onChange={(e) => handleLanguageChange(e.target.value)}
        >
          <option value="en">English</option>
          <option value="es">Spanish</option>
          <option value="fr">French</option>
        </select>
      </div>
      
      <div>
        <label>
          <input
            type="checkbox"
            checked={settings.notifications.email}
            onChange={() => handleNotificationToggle('email')}
          />
          Email notifications
        </label>
      </div>
      
      <div>
        <label>
          <input
            type="checkbox"
            checked={settings.notifications.push}
            onChange={() => handleNotificationToggle('push')}
          />
          Push notifications
        </label>
      </div>
    </div>
  );
}

function NotificationList() {
  const { notifications, removeNotification, clearAll } = useNotifications();
  
  if (notifications.length === 0) {
    return null;
  }
  
  return (
    <div className="notification-list">
      <div className="notification-header">
        <h4>Notifications</h4>
        <button onClick={clearAll}>Clear All</button>
      </div>
      
      {notifications.map(notification => (
        <div
          key={notification.id}
          className={`notification notification--${notification.type}`}
        >
          <span>{notification.message}</span>
          <button onClick={() => removeNotification(notification.id)}>×</button>
        </div>
      ))}
    </div>
  );
}

// Combined provider component
function AppProviders({ children }) {
  return (
    <UserProvider>
      <SettingsProvider>
        <NotificationProvider>
          {children}
        </NotificationProvider>
      </SettingsProvider>
    </UserProvider>
  );
}

// Main app
function MultiContextApp() {
  const { login, isAuthenticated } = useUser();
  
  const handleLogin = () => {
    login({ name: 'John Doe', email: 'john@example.com' });
  };
  
  return (
    <div>
      <h1>Multi-Context Application</h1>
      
      <NotificationList />
      
      {!isAuthenticated ? (
        <button onClick={handleLogin}>Login</button>
      ) : (
        <>
          <UserDashboard />
          <SettingsPanel />
        </>
      )}
    </div>
  );
}

// Root app
function App() {
  return (
    <AppProviders>
      <MultiContextApp />
    </AppProviders>
  );
}
```

## Context Performance Optimization

Context can cause performance issues if not used carefully. Here are some optimization techniques:

### 1. Split Context by Concern

Instead of one large context, split it into smaller, focused contexts:

```jsx
// Instead of one large context
const AppContext = createContext();

function AppProvider({ children }) {
  const [user, setUser] = useState(null);
  const [theme, setTheme] = useState('light');
  const [settings, setSettings] = useState({});
  const [cart, setCart] = useState([]);
  
  // All components re-render when any value changes
  const value = { user, setUser, theme, setTheme, settings, setSettings, cart, setCart };
  
  return <AppContext.Provider value={value}>{children}</AppContext.Provider>;
}

// Split into separate contexts
const UserContext = createContext();
const ThemeContext = createContext();
const SettingsContext = createContext();
const CartContext = createContext();

// Each context only causes re-renders for components that use it
```

### 2. Memoize Context Values

Use useMemo to prevent unnecessary re-renders:

```jsx
function CartProvider({ children }) {
  const [items, setItems] = useState([]);
  const [discount, setDiscount] = useState(0);
  
  // Memoize the context value
  const value = useMemo(() => ({
    items,
    discount,
    addItem: (item) => setItems(prev => [...prev, item]),
    removeItem: (id) => setItems(prev => prev.filter(item => item.id !== id)),
    applyDiscount: (value) => setDiscount(value)
  }), [items, discount]);
  
  return (
    <CartContext.Provider value={value}>
      {children}
    </CartContext.Provider>
  );
}
```

### 3. Use Multiple Providers for Different Update Frequencies

Separate frequently changing data from stable data:

```jsx
// Stable data (changes rarely)
const ConfigContext = createContext();

function ConfigProvider({ children }) {
  const config = useMemo(() => ({
    apiUrl: process.env.REACT_APP_API_URL,
    theme: 'light',
    language: 'en'
  }), []);
  
  return (
    <ConfigContext.Provider value={config}>
      {children}
    </ConfigContext.Provider>
  );
}

// Frequently changing data
const CounterContext = createContext();

function CounterProvider({ children }) {
  const [count, setCount] = useState(0);
  
  const value = useMemo(() => ({
    count,
    increment: () => setCount(prev => prev + 1),
    decrement: () => setCount(prev => prev - 1)
  }), [count]);
  
  return (
    <CounterContext.Provider value={value}>
      {children}
    </CounterContext.Provider>
  );
}
```

### 4. Use React.memo for Components

Prevent re-renders of components that don't need to update:

```jsx
const ExpensiveComponent = React.memo(function ExpensiveComponent({ data }) {
  console.log('ExpensiveComponent rendered');
  
  return (
    <div>
      {/* Expensive rendering logic */}
      {data.map(item => (
        <div key={item.id}>{item.name}</div>
      ))}
    </div>
  );
});

function App() {
  const { user } = useUser(); // Only re-renders when user changes
  const [count, setCount] = useState(0); // Local state doesn't affect context
  
  return (
    <div>
      <button onClick={() => setCount(count + 1)}>Count: {count}</button>
      {/* This component won't re-render when count changes */}
      <ExpensiveComponent data={user?.preferences || []} />
    </div>
  );
}
```

## Context Best Practices

### 1. Always Provide Default Values

```jsx
const ThemeContext = createContext({
  theme: 'light',
  toggleTheme: () => {
    console.warn('toggleTheme called outside of ThemeProvider');
  }
});
```

### 2. Create Custom Hooks

Always create custom hooks for consuming context:

```jsx
function useTheme() {
  const context = useContext(ThemeContext);
  
  if (!context) {
    throw new Error('useTheme must be used within a ThemeProvider');
  }
  
  return context;
}
```

### 3. Keep Context Close to Where It's Used

Don't wrap your entire app in context if only a few components need it:

```jsx
// Instead of wrapping the entire app
function App() {
  return (
    <UserProvider>
      <Header />
      <Main />
      <Footer /> {/* Footer doesn't need user context */}
    </UserProvider>
  );
}

// Only wrap the components that need it
function App() {
  return (
    <>
      <Header />
      <UserProvider>
        <Main />
      </UserProvider>
      <Footer />
    </>
  );
}
```

### 4. Use TypeScript for Better Developer Experience

```typescript
interface User {
  id: string;
  name: string;
  email: string;
}

interface UserContextType {
  user: User | null;
  login: (user: User) => void;
  logout: () => void;
  isAuthenticated: boolean;
}

const UserContext = createContext<UserContextType | undefined>(undefined);

function useUser(): UserContextType {
  const context = useContext(UserContext);
  
  if (!context) {
    throw new Error('useUser must be used within a UserProvider');
  }
  
  return context;
}
```

## Common Patterns and Use Cases

### 1. Authentication Context

```jsx
function AuthProvider({ children }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    // Check if user is already authenticated
    const token = localStorage.getItem('authToken');
    if (token) {
      validateToken(token).then(userData => {
        setUser(userData);
        setLoading(false);
      }).catch(() => {
        localStorage.removeItem('authToken');
        setLoading(false);
      });
    } else {
      setLoading(false);
    }
  }, []);
  
  const login = async (credentials) => {
    const { user, token } = await authenticate(credentials);
    localStorage.setItem('authToken', token);
    setUser(user);
  };
  
  const logout = () => {
    localStorage.removeItem('authToken');
    setUser(null);
  };
  
  if (loading) {
    return <div>Loading...</div>;
  }
  
  return (
    <AuthContext.Provider value={{ user, login, logout, isAuthenticated: !!user }}>
      {children}
    </AuthContext.Provider>
  );
}
```

### 2. Form Context for Complex Forms

```jsx
function FormProvider({ children, onSubmit }) {
  const [formData, setFormData] = useState({});
  const [errors, setErrors] = useState({});
  const [isSubmitting, setIsSubmitting] = useState(false);
  
  const setFieldValue = (name, value) => {
    setFormData(prev => ({ ...prev, [name]: value }));
    // Clear error when user starts typing
    if (errors[name]) {
      setErrors(prev => ({ ...prev, [name]: null }));
    }
  };
  
  const setFieldError = (name, error) => {
    setErrors(prev => ({ ...prev, [name]: error }));
  };
  
  const handleSubmit = async () => {
    setIsSubmitting(true);
    try {
      await onSubmit(formData);
    } catch (error) {
      if (error.fieldErrors) {
        setErrors(error.fieldErrors);
      }
    } finally {
      setIsSubmitting(false);
    }
  };
  
  const value = {
    formData,
    errors,
    isSubmitting,
    setFieldValue,
    setFieldError,
    handleSubmit
  };
  
  return (
    <FormContext.Provider value={value}>
      {children}
    </FormContext.Provider>
  );
}
```

### 3. Modal Context

```jsx
function ModalProvider({ children }) {
  const [modals, setModals] = useState([]);
  
  const openModal = (component, props = {}) => {
    const id = Date.now().toString();
    setModals(prev => [...prev, { id, component, props }]);
    return id;
  };
  
  const closeModal = (id) => {
    setModals(prev => prev.filter(modal => modal.id !== id));
  };
  
  const closeAllModals = () => {
    setModals([]);
  };
  
  return (
    <ModalContext.Provider value={{ openModal, closeModal, closeAllModals }}>
      {children}
      {modals.map(modal => {
        const Component = modal.component;
        return (
          <Modal key={modal.id} onClose={() => closeModal(modal.id)}>
            <Component {...modal.props} onClose={() => closeModal(modal.id)} />
          </Modal>
        );
      })}
    </ModalContext.Provider>
  );
}
```

## Summary

The Context API is a powerful tool for state management in React applications. Key takeaways:

1. **Use Context for Global State**: Perfect for data that needs to be shared across many components
2. **Avoid Prop Drilling**: Context eliminates the need to pass props through multiple levels
3. **Combine with useReducer**: For complex state management, combine Context with useReducer
4. **Performance Matters**: Split contexts, memoize values, and use React.memo to optimize performance
5. **Custom Hooks**: Always create custom hooks for better developer experience
6. **Type Safety**: Use TypeScript for better development experience and error prevention

Common use cases include:
- Authentication and user management
- Theme and UI preferences
- Shopping cart and e-commerce state
- Form data management
- Modal and notification systems
- Localization and internationalization

By mastering the Context API, you can build more maintainable and scalable React applications with cleaner component hierarchies and better state management.

## Practice Exercise

Build a complete e-commerce application that demonstrates multiple contexts:

1. Create a UserContext for authentication
2. Implement a CartContext with useReducer for shopping cart management
3. Add a ThemeContext for light/dark mode switching
4. Create a NotificationContext for showing success/error messages
5. Implement a ProductContext for managing product data
6. Add proper TypeScript types for all contexts
7. Optimize performance with proper memoization

This exercise will help you understand how to architect a real-world application using multiple contexts effectively.

## Additional Resources

- [React Context Official Documentation](https://react.dev/reference/react/createContext)
- [When to use Context](https://react.dev/learn/passing-data-deeply-with-context)
- [Context API Performance Tips](https://blog.logrocket.com/react-context-api-deep-dive-examples/)
- [Context vs Redux](https://blog.isquaredsoftware.com/2021/01/context-redux-differences/)
