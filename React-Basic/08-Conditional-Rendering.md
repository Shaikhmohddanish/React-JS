# Conditional Rendering in React

## Introduction to Conditional Rendering

Conditional rendering is one of the most powerful features in React. It allows you to create dynamic user interfaces that display different content based on various conditions. With conditional rendering, you can:

- Show or hide elements
- Render different components based on state
- Display alternative content while data is loading
- Create permission-based UI elements
- Handle error states gracefully

In this comprehensive guide, we'll explore various techniques for conditional rendering in React, from basic if statements to advanced patterns.

## Why Conditional Rendering Matters

Conditional rendering is essential for building interactive applications because:

1. **Dynamic UIs**: Users expect interfaces that respond to their actions and show relevant information
2. **Data-Driven Content**: Applications need to display different content based on changing data
3. **Progressive Disclosure**: Revealing information gradually improves user experience
4. **Error Handling**: Proper error states help users understand when something goes wrong
5. **Permission Management**: Different users should see different parts of the application

## Basic Conditional Rendering Techniques

Let's start with the fundamental methods for conditional rendering in React.

### 1. If Statements Outside JSX

The most straightforward approach is to use if statements before the return statement:

```jsx
function UserGreeting({ isLoggedIn, username }) {
  if (isLoggedIn) {
    return <h1>Welcome back, {username}!</h1>;
  }
  
  return <h1>Please sign in</h1>;
}
```

This approach works well when:
- The condition creates completely different outputs
- The conditional logic is complex
- You need to return early under certain conditions

### 2. Ternary Operators

For simpler conditions that can be embedded directly in JSX, the ternary operator is ideal:

```jsx
function UserGreeting({ isLoggedIn, username }) {
  return (
    <h1>
      {isLoggedIn ? `Welcome back, ${username}!` : 'Please sign in'}
    </h1>
  );
}
```

The ternary operator follows this syntax: `condition ? expressionIfTrue : expressionIfFalse`

It's perfect for:
- Simple conditional expressions within JSX
- Toggling between two different elements
- Applying conditional props or styles

### 3. Logical && Operator

For cases where you want to render something or nothing, the logical && operator is concise:

```jsx
function Notifications({ messages }) {
  return (
    <div>
      <h2>Recent Notifications</h2>
      {messages.length > 0 && (
        <ul>
          {messages.map(message => (
            <li key={message.id}>{message.text}</li>
          ))}
        </ul>
      )}
      {messages.length === 0 && (
        <p>You have no new notifications</p>
      )}
    </div>
  );
}
```

How it works:
- If the left side of `&&` evaluates to true, the right side is rendered
- If the left side evaluates to false, React skips the right side

This technique is useful for:
- Conditional rendering of elements
- Showing content only when data is available
- Implementing "if without else" logic

**Important Note**: Be careful with numbers as the left operand. If the left side evaluates to 0, React will render 0 instead of nothing. For example:

```jsx
function Counter({ count }) {
  return (
    <div>
      {count && <p>Count: {count}</p>}
    </div>
  );
}
```

If `count` is 0, this will display just "0" instead of nothing. To avoid this, use a Boolean conversion or comparison:

```jsx
function Counter({ count }) {
  return (
    <div>
      {count > 0 && <p>Count: {count}</p>}
    </div>
  );
}
```

### 4. Switch Statements

For multiple conditions, switch statements provide a clean approach:

```jsx
function StatusMessage({ status }) {
  let message;
  
  switch (status) {
    case 'loading':
      message = <p>Loading data...</p>;
      break;
    case 'success':
      message = <p className="success">Data loaded successfully!</p>;
      break;
    case 'error':
      message = <p className="error">Error loading data. Please try again.</p>;
      break;
    default:
      message = <p>Please fetch data</p>;
  }
  
  return (
    <div className="status-container">
      {message}
    </div>
  );
}
```

Switch statements work well when:
- You have multiple possible states
- Each state maps to different content
- The conditions are based on the same variable

### 5. Object Maps

For an elegant alternative to switch statements, you can use object maps:

```jsx
function StatusMessage({ status }) {
  const statusMessages = {
    loading: <p>Loading data...</p>,
    success: <p className="success">Data loaded successfully!</p>,
    error: <p className="error">Error loading data. Please try again.</p>,
    default: <p>Please fetch data</p>
  };
  
  return (
    <div className="status-container">
      {statusMessages[status] || statusMessages.default}
    </div>
  );
}
```

This approach:
- Is more concise than switch statements
- Makes it easy to add or modify conditions
- Can be defined outside the component for reuse

## Conditional Styling and Props

Conditions can control not just what is rendered, but also how it looks or behaves.

### Conditional CSS Classes

You can conditionally apply CSS classes:

```jsx
function Button({ isActive, onClick, children }) {
  return (
    <button 
      className={`btn ${isActive ? 'btn-active' : 'btn-inactive'}`}
      onClick={onClick}
    >
      {children}
    </button>
  );
}
```

### Conditional Inline Styles

Similarly, inline styles can be conditional:

```jsx
function ProgressBar({ percentage }) {
  return (
    <div className="progress-container">
      <div 
        className="progress-bar"
        style={{ 
          width: `${percentage}%`,
          backgroundColor: percentage < 30 ? 'red' : 
                           percentage < 70 ? 'yellow' : 'green'
        }}
      />
    </div>
  );
}
```

### Conditional Props

You can also pass props conditionally:

```jsx
function TextInput({ isDisabled, isReadOnly, ...props }) {
  return (
    <input
      type="text"
      disabled={isDisabled}
      {...(isReadOnly && { readOnly: true })}
      {...props}
    />
  );
}
```

In this example, the `readOnly` prop is only included when `isReadOnly` is true, using the spread operator with a conditional object.

## Advanced Conditional Rendering Patterns

Now let's explore some more sophisticated patterns for conditional rendering.

### 1. Component Variables

For complex conditions, you can use variables to store elements:

```jsx
function UserDashboard({ user, isLoading, error }) {
  let content;
  
  if (isLoading) {
    content = <LoadingSpinner />;
  } else if (error) {
    content = <ErrorMessage error={error} />;
  } else if (user) {
    content = (
      <div>
        <h1>Welcome, {user.name}</h1>
        <UserStats stats={user.stats} />
        <RecentActivity activities={user.recentActivities} />
      </div>
    );
  } else {
    content = <p>No user data available</p>;
  }
  
  return (
    <div className="dashboard">
      {content}
    </div>
  );
}
```

This pattern is useful when:
- You have complex nested conditions
- The conditional content is extensive
- You want to keep the return statement clean

### 2. Immediately-Invoked Function Expressions (IIFE)

For complex conditional logic within JSX, you can use an immediately-invoked function expression:

```jsx
function ProductList({ products, category, isOnSale }) {
  return (
    <div>
      <h2>Products</h2>
      <ul>
        {(() => {
          const filteredProducts = products
            .filter(product => {
              if (category && product.category !== category) return false;
              if (isOnSale && !product.onSale) return false;
              return true;
            });
            
          if (filteredProducts.length === 0) {
            return <p>No products found matching your criteria.</p>;
          }
          
          return filteredProducts.map(product => (
            <li key={product.id}>
              {product.name} - ${product.price}
              {product.onSale && <span className="sale-badge">On Sale!</span>}
            </li>
          ));
        })()}
      </ul>
    </div>
  );
}
```

This pattern:
- Allows for complex logic within JSX
- Can combine filtering, mapping, and conditional rendering
- Keeps the logic contained within the component

However, it can make code harder to read, so use it sparingly.

### 3. Conditional Rendering with Higher-Order Components (HOCs)

You can create HOCs that conditionally render components:

```jsx
// Higher-Order Component
function withAuth(Component) {
  return function AuthenticatedComponent(props) {
    const { isAuthenticated, user } = useAuth(); // Custom hook for auth
    
    if (!isAuthenticated) {
      return <LoginPrompt />;
    }
    
    if (!user.hasPermission('access:feature')) {
      return <UnauthorizedMessage />;
    }
    
    return <Component {...props} user={user} />;
  };
}

// Usage
const ProtectedDashboard = withAuth(Dashboard);

function App() {
  return (
    <div>
      <Header />
      <ProtectedDashboard />
      <Footer />
    </div>
  );
}
```

This pattern is powerful for:
- Authentication and authorization checks
- Feature flags and permission-based UI
- Loading states that apply to multiple components

### 4. Conditional Rendering with Render Props

The render props pattern provides another flexible approach:

```jsx
function Toggleable({ render }) {
  const [isVisible, setIsVisible] = React.useState(false);
  
  const toggle = () => setIsVisible(!isVisible);
  
  return render({ isVisible, toggle });
}

// Usage
function App() {
  return (
    <Toggleable
      render={({ isVisible, toggle }) => (
        <div>
          <button onClick={toggle}>
            {isVisible ? 'Hide Details' : 'Show Details'}
          </button>
          
          {isVisible && (
            <div className="details-panel">
              <h2>Additional Information</h2>
              <p>This content is toggleable.</p>
            </div>
          )}
        </div>
      )}
    />
  );
}
```

This pattern:
- Provides great flexibility in how components are rendered
- Allows the parent component to control the rendering logic
- Can be combined with other patterns for complex UIs

### 5. Compound Components with Context

For advanced cases, compound components with context can provide an elegant solution:

```jsx
// Create a context
const TabContext = React.createContext();

// Parent component
function Tabs({ children, defaultIndex = 0 }) {
  const [activeIndex, setActiveIndex] = React.useState(defaultIndex);
  
  return (
    <TabContext.Provider value={{ activeIndex, setActiveIndex }}>
      <div className="tabs-container">
        {children}
      </div>
    </TabContext.Provider>
  );
}

// Child components
function TabList({ children }) {
  return <div className="tab-list">{children}</div>;
}

function Tab({ index, children }) {
  const { activeIndex, setActiveIndex } = React.useContext(TabContext);
  const isActive = index === activeIndex;
  
  return (
    <div
      className={`tab ${isActive ? 'active' : ''}`}
      onClick={() => setActiveIndex(index)}
    >
      {children}
    </div>
  );
}

function TabPanel({ index, children }) {
  const { activeIndex } = React.useContext(TabContext);
  
  if (index !== activeIndex) return null;
  
  return <div className="tab-panel">{children}</div>;
}

// Set up component hierarchy
Tabs.TabList = TabList;
Tabs.Tab = Tab;
Tabs.TabPanel = TabPanel;

// Usage
function App() {
  return (
    <Tabs>
      <Tabs.TabList>
        <Tabs.Tab index={0}>Profile</Tabs.Tab>
        <Tabs.Tab index={1}>Settings</Tabs.Tab>
        <Tabs.Tab index={2}>Notifications</Tabs.Tab>
      </Tabs.TabList>
      
      <Tabs.TabPanel index={0}>
        <h2>Profile Content</h2>
        <p>User profile information goes here.</p>
      </Tabs.TabPanel>
      
      <Tabs.TabPanel index={1}>
        <h2>Settings Content</h2>
        <p>User settings go here.</p>
      </Tabs.TabPanel>
      
      <Tabs.TabPanel index={2}>
        <h2>Notifications Content</h2>
        <p>User notifications go here.</p>
      </Tabs.TabPanel>
    </Tabs>
  );
}
```

This pattern:
- Creates a clear, declarative API
- Hides the conditional rendering implementation details
- Allows for complex component relationships
- Makes it easy to understand the component structure

## Handling Loading, Error, and Empty States

A common use case for conditional rendering is handling different data states.

### Loading States

```jsx
function UserProfile({ userId }) {
  const [user, setUser] = React.useState(null);
  const [isLoading, setIsLoading] = React.useState(true);
  const [error, setError] = React.useState(null);
  
  React.useEffect(() => {
    const fetchUser = async () => {
      try {
        setIsLoading(true);
        setError(null);
        
        // Simulating API call
        const response = await fetch(`/api/users/${userId}`);
        
        if (!response.ok) {
          throw new Error('Failed to fetch user data');
        }
        
        const userData = await response.json();
        setUser(userData);
      } catch (err) {
        setError(err.message);
      } finally {
        setIsLoading(false);
      }
    };
    
    fetchUser();
  }, [userId]);
  
  if (isLoading) {
    return (
      <div className="loading-container">
        <div className="spinner"></div>
        <p>Loading user profile...</p>
      </div>
    );
  }
  
  if (error) {
    return (
      <div className="error-container">
        <p className="error-message">Error: {error}</p>
        <button onClick={() => window.location.reload()}>Retry</button>
      </div>
    );
  }
  
  if (!user) {
    return <p>No user data found.</p>;
  }
  
  return (
    <div className="user-profile">
      <h1>{user.name}</h1>
      <p>Email: {user.email}</p>
      <p>Member since: {new Date(user.joinDate).toLocaleDateString()}</p>
    </div>
  );
}
```

### Generic LoadingWrapper Component

For consistency across your application, create reusable components for loading and error states:

```jsx
function LoadingWrapper({ isLoading, error, onRetry, loadingMessage = 'Loading...', children }) {
  if (isLoading) {
    return (
      <div className="loading-container">
        <div className="spinner"></div>
        <p>{loadingMessage}</p>
      </div>
    );
  }
  
  if (error) {
    return (
      <div className="error-container">
        <p className="error-message">Error: {error}</p>
        {onRetry && <button onClick={onRetry}>Retry</button>}
      </div>
    );
  }
  
  return children;
}

// Usage
function ProductList() {
  const [products, setProducts] = React.useState([]);
  const [isLoading, setIsLoading] = React.useState(true);
  const [error, setError] = React.useState(null);
  
  const fetchProducts = async () => {
    try {
      setIsLoading(true);
      setError(null);
      // API call logic...
      const data = await fetchFromAPI('/products');
      setProducts(data);
    } catch (err) {
      setError(err.message);
    } finally {
      setIsLoading(false);
    }
  };
  
  React.useEffect(() => {
    fetchProducts();
  }, []);
  
  return (
    <LoadingWrapper
      isLoading={isLoading}
      error={error}
      onRetry={fetchProducts}
      loadingMessage="Loading products..."
    >
      <div className="product-list">
        {products.length > 0 ? (
          products.map(product => (
            <ProductCard key={product.id} product={product} />
          ))
        ) : (
          <p>No products found.</p>
        )}
      </div>
    </LoadingWrapper>
  );
}
```

### Empty States

Well-designed empty states improve user experience:

```jsx
function SearchResults({ results, searchQuery }) {
  if (!searchQuery) {
    return (
      <div className="empty-state">
        <img src="/images/search-icon.svg" alt="Search" />
        <h2>Search for something</h2>
        <p>Enter a search term to find results</p>
      </div>
    );
  }
  
  if (results.length === 0) {
    return (
      <div className="empty-state">
        <img src="/images/no-results.svg" alt="No results" />
        <h2>No results found</h2>
        <p>We couldn't find any results for "{searchQuery}"</p>
        <p>Try using different keywords or check your spelling</p>
      </div>
    );
  }
  
  return (
    <div className="search-results">
      <h2>{results.length} results for "{searchQuery}"</h2>
      <ul>
        {results.map(result => (
          <li key={result.id}>
            <h3>{result.title}</h3>
            <p>{result.description}</p>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

## Optimizing Conditional Rendering

Conditional rendering can sometimes impact performance. Here are some techniques to optimize it:

### 1. Memoization with React.memo

Prevent unnecessary re-renders of conditionally rendered components:

```jsx
const ExpensiveComponent = React.memo(function ExpensiveComponent({ data }) {
  // Expensive rendering logic...
  return <div>{/* Complex UI */}</div>;
});

function ParentComponent({ showExpensive, data }) {
  return (
    <div>
      {showExpensive && <ExpensiveComponent data={data} />}
    </div>
  );
}
```

### 2. Lazy Loading with React.lazy and Suspense

For components that are conditionally rendered but large, use lazy loading:

```jsx
const AdminDashboard = React.lazy(() => import('./AdminDashboard'));
const UserDashboard = React.lazy(() => import('./UserDashboard'));

function Dashboard({ userRole }) {
  return (
    <React.Suspense fallback={<div>Loading dashboard...</div>}>
      {userRole === 'admin' ? <AdminDashboard /> : <UserDashboard />}
    </React.Suspense>
  );
}
```

### 3. Avoid Expensive Operations in Conditions

Be careful about performing expensive operations within the condition itself:

```jsx
// Not ideal - filterExpensiveData runs on every render
function ProductList({ products }) {
  return (
    <div>
      {filterExpensiveData(products).length > 0 ? (
        <ProductGrid products={filterExpensiveData(products)} />
      ) : (
        <NoProducts />
      )}
    </div>
  );
}

// Better - compute once and reuse
function ProductList({ products }) {
  const filteredProducts = React.useMemo(
    () => filterExpensiveData(products),
    [products]
  );
  
  return (
    <div>
      {filteredProducts.length > 0 ? (
        <ProductGrid products={filteredProducts} />
      ) : (
        <NoProducts />
      )}
    </div>
  );
}
```

### 4. Extract Conditional Components

For cleaner code and potential performance benefits, extract complex conditional rendering into separate components:

```jsx
// Before extraction
function UserInterface({ user, permissions, theme }) {
  return (
    <div className={`theme-${theme}`}>
      <Header user={user} />
      {user.isAdmin && permissions.includes('viewAdminPanel') && (
        <div className="admin-panel">
          <h2>Admin Panel</h2>
          <AdminControls />
          <UserManagement />
          <SystemSettings />
        </div>
      )}
      <MainContent user={user} />
      <Footer />
    </div>
  );
}

// After extraction
function AdminPanel({ user, permissions }) {
  if (!user.isAdmin || !permissions.includes('viewAdminPanel')) {
    return null;
  }
  
  return (
    <div className="admin-panel">
      <h2>Admin Panel</h2>
      <AdminControls />
      <UserManagement />
      <SystemSettings />
    </div>
  );
}

function UserInterface({ user, permissions, theme }) {
  return (
    <div className={`theme-${theme}`}>
      <Header user={user} />
      <AdminPanel user={user} permissions={permissions} />
      <MainContent user={user} />
      <Footer />
    </div>
  );
}
```

## Common Conditional Rendering Patterns

Let's explore some common patterns you'll encounter in React applications.

### Feature Flags

Feature flags allow you to enable or disable features without deploying new code:

```jsx
function FeatureFlaggedComponent({ flags }) {
  return (
    <div>
      <h1>My Application</h1>
      
      {/* Basic feature flag */}
      {flags.showNewDashboard ? (
        <NewDashboard />
      ) : (
        <LegacyDashboard />
      )}
      
      {/* Feature with percentage rollout */}
      {flags.newCheckout.enabled && Math.random() < flags.newCheckout.percentage && (
        <NewCheckoutProcess />
      )}
      
      {/* Feature enabled for specific users */}
      {flags.betaFeatures.enabledFor.includes(currentUser.id) && (
        <BetaFeatures />
      )}
    </div>
  );
}
```

### Role-Based Access Control

Control what users see based on their roles:

```jsx
function AdminPage({ user }) {
  // Early return for unauthorized users
  if (!user || user.role !== 'admin') {
    return <UnauthorizedPage />;
  }
  
  return (
    <div>
      <h1>Admin Dashboard</h1>
      
      <section>
        <h2>User Management</h2>
        <UserTable />
      </section>
      
      {/* Nested permission check */}
      {user.permissions.includes('manage_billing') && (
        <section>
          <h2>Billing Settings</h2>
          <BillingControls />
        </section>
      )}
      
      {/* Multiple permission check */}
      {(user.permissions.includes('view_metrics') || user.role === 'super_admin') && (
        <section>
          <h2>Analytics</h2>
          <MetricsDashboard />
        </section>
      )}
    </div>
  );
}
```

### Responsive UI

Conditionally render different layouts based on screen size:

```jsx
function ResponsiveLayout() {
  const [windowWidth, setWindowWidth] = React.useState(window.innerWidth);
  
  React.useEffect(() => {
    function handleResize() {
      setWindowWidth(window.innerWidth);
    }
    
    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
  }, []);
  
  const isMobile = windowWidth < 768;
  const isTablet = windowWidth >= 768 && windowWidth < 1024;
  const isDesktop = windowWidth >= 1024;
  
  return (
    <div>
      {/* Completely different layouts */}
      {isMobile && <MobileNavigation />}
      {(isTablet || isDesktop) && <DesktopNavigation />}
      
      <main>
        {/* Conditional content arrangement */}
        {isMobile ? (
          <div className="mobile-layout">
            <Content />
            <Sidebar />
          </div>
        ) : (
          <div className="desktop-layout">
            <Content />
            <aside>
              <Sidebar />
              {isDesktop && <AdditionalInfo />}
            </aside>
          </div>
        )}
      </main>
    </div>
  );
}
```

For more sophisticated responsive handling, consider using a dedicated library like `react-responsive` or CSS media queries.

### Multi-Step Forms

Conditional rendering is perfect for multi-step forms:

```jsx
function MultiStepForm() {
  const [step, setStep] = React.useState(1);
  const [formData, setFormData] = React.useState({
    personalInfo: {
      name: '',
      email: ''
    },
    addressInfo: {
      street: '',
      city: '',
      zipCode: ''
    },
    paymentInfo: {
      cardNumber: '',
      expiryDate: '',
      cvv: ''
    }
  });
  
  const updateFormData = (section, field, value) => {
    setFormData({
      ...formData,
      [section]: {
        ...formData[section],
        [field]: value
      }
    });
  };
  
  const nextStep = () => setStep(step + 1);
  const prevStep = () => setStep(step - 1);
  
  return (
    <div className="multi-step-form">
      <div className="progress-bar">
        <div 
          className="progress" 
          style={{ width: `${(step / 3) * 100}%` }}
        ></div>
      </div>
      
      <div className="step-indicators">
        <div className={`step ${step >= 1 ? 'active' : ''}`}>Personal</div>
        <div className={`step ${step >= 2 ? 'active' : ''}`}>Address</div>
        <div className={`step ${step >= 3 ? 'active' : ''}`}>Payment</div>
      </div>
      
      {/* Step 1: Personal Information */}
      {step === 1 && (
        <div className="form-step">
          <h2>Personal Information</h2>
          <input
            type="text"
            placeholder="Full Name"
            value={formData.personalInfo.name}
            onChange={(e) => updateFormData('personalInfo', 'name', e.target.value)}
          />
          <input
            type="email"
            placeholder="Email Address"
            value={formData.personalInfo.email}
            onChange={(e) => updateFormData('personalInfo', 'email', e.target.value)}
          />
          <button onClick={nextStep}>Next</button>
        </div>
      )}
      
      {/* Step 2: Address Information */}
      {step === 2 && (
        <div className="form-step">
          <h2>Address Information</h2>
          <input
            type="text"
            placeholder="Street Address"
            value={formData.addressInfo.street}
            onChange={(e) => updateFormData('addressInfo', 'street', e.target.value)}
          />
          <input
            type="text"
            placeholder="City"
            value={formData.addressInfo.city}
            onChange={(e) => updateFormData('addressInfo', 'city', e.target.value)}
          />
          <input
            type="text"
            placeholder="ZIP Code"
            value={formData.addressInfo.zipCode}
            onChange={(e) => updateFormData('addressInfo', 'zipCode', e.target.value)}
          />
          <div className="buttons">
            <button onClick={prevStep}>Previous</button>
            <button onClick={nextStep}>Next</button>
          </div>
        </div>
      )}
      
      {/* Step 3: Payment Information */}
      {step === 3 && (
        <div className="form-step">
          <h2>Payment Information</h2>
          <input
            type="text"
            placeholder="Card Number"
            value={formData.paymentInfo.cardNumber}
            onChange={(e) => updateFormData('paymentInfo', 'cardNumber', e.target.value)}
          />
          <input
            type="text"
            placeholder="Expiry Date (MM/YY)"
            value={formData.paymentInfo.expiryDate}
            onChange={(e) => updateFormData('paymentInfo', 'expiryDate', e.target.value)}
          />
          <input
            type="text"
            placeholder="CVV"
            value={formData.paymentInfo.cvv}
            onChange={(e) => updateFormData('paymentInfo', 'cvv', e.target.value)}
          />
          <div className="buttons">
            <button onClick={prevStep}>Previous</button>
            <button onClick={() => alert('Form submitted!')}>Submit</button>
          </div>
        </div>
      )}
    </div>
  );
}
```

A more maintainable approach for complex forms would be to extract each step into its own component.

## Common Pitfalls and How to Avoid Them

### 1. Forgetting to Handle the "Else" Case

Always consider all possible states:

```jsx
// Problematic - what if isLoading is false but data is still null?
function DataDisplay({ data, isLoading }) {
  if (isLoading) {
    return <LoadingSpinner />;
  }
  
  return <div>{data.name}</div>; // This will crash if data is null
}

// Better
function DataDisplay({ data, isLoading }) {
  if (isLoading) {
    return <LoadingSpinner />;
  }
  
  if (!data) {
    return <div>No data available</div>;
  }
  
  return <div>{data.name}</div>;
}
```

### 2. Too Much Nesting

Deeply nested conditions can be hard to read and maintain:

```jsx
// Problematic
function UserProfile({ user, isLoading, error, isAdmin }) {
  if (isLoading) {
    return <LoadingSpinner />;
  } else {
    if (error) {
      return <ErrorMessage error={error} />;
    } else {
      if (!user) {
        return <p>No user found</p>;
      } else {
        return (
          <div>
            <h1>{user.name}</h1>
            {isAdmin ? (
              <div>
                <AdminPanel />
              </div>
            ) : null}
          </div>
        );
      }
    }
  }
}

// Better - flat conditions with early returns
function UserProfile({ user, isLoading, error, isAdmin }) {
  if (isLoading) {
    return <LoadingSpinner />;
  }
  
  if (error) {
    return <ErrorMessage error={error} />;
  }
  
  if (!user) {
    return <p>No user found</p>;
  }
  
  return (
    <div>
      <h1>{user.name}</h1>
      {isAdmin && <AdminPanel />}
    </div>
  );
}
```

### 3. Inconsistent Loading States

Inconsistent handling of loading, error, and empty states can create a confusing user experience:

```jsx
// Problematic - inconsistent loading UI
function Dashboard() {
  const [users, usersLoading, usersError] = useUsers();
  const [posts, postsLoading, postsError] = usePosts();
  const [comments, commentsLoading, commentsError] = useComments();
  
  return (
    <div>
      <h1>Dashboard</h1>
      
      <section>
        <h2>Users</h2>
        {usersLoading ? (
          <p>Loading users...</p>
        ) : usersError ? (
          <p>Error: {usersError}</p>
        ) : (
          <UserList users={users} />
        )}
      </section>
      
      <section>
        <h2>Posts</h2>
        {postsLoading ? (
          <div className="spinner" />
        ) : postsError ? (
          <div className="error-box">{postsError}</div>
        ) : (
          <PostGrid posts={posts} />
        )}
      </section>
      
      <section>
        <h2>Comments</h2>
        {commentsLoading ? (
          <LoadingSpinner />
        ) : commentsError ? (
          <ErrorMessage error={commentsError} />
        ) : (
          <CommentList comments={comments} />
        )}
      </section>
    </div>
  );
}

// Better - consistent loading UI with a reusable component
function DataSection({ title, isLoading, error, data, renderData }) {
  return (
    <section>
      <h2>{title}</h2>
      {isLoading ? (
        <LoadingSpinner />
      ) : error ? (
        <ErrorMessage error={error} />
      ) : data.length === 0 ? (
        <EmptyState entity={title.toLowerCase()} />
      ) : (
        renderData(data)
      )}
    </section>
  );
}

function Dashboard() {
  const [users, usersLoading, usersError] = useUsers();
  const [posts, postsLoading, postsError] = usePosts();
  const [comments, commentsLoading, commentsError] = useComments();
  
  return (
    <div>
      <h1>Dashboard</h1>
      
      <DataSection
        title="Users"
        isLoading={usersLoading}
        error={usersError}
        data={users}
        renderData={(data) => <UserList users={data} />}
      />
      
      <DataSection
        title="Posts"
        isLoading={postsLoading}
        error={postsError}
        data={posts}
        renderData={(data) => <PostGrid posts={data} />}
      />
      
      <DataSection
        title="Comments"
        isLoading={commentsLoading}
        error={commentsError}
        data={comments}
        renderData={(data) => <CommentList comments={data} />}
      />
    </div>
  );
}
```

### 4. Forgetting Key Props

When conditionally rendering lists, always include key props:

```jsx
// Problematic - missing key prop
function TodoList({ todos, showCompleted }) {
  return (
    <ul>
      {todos.map(todo => {
        if (showCompleted || !todo.completed) {
          return <li>{todo.text}</li>; // Missing key prop
        }
        return null;
      })}
    </ul>
  );
}

// Better
function TodoList({ todos, showCompleted }) {
  return (
    <ul>
      {todos
        .filter(todo => showCompleted || !todo.completed)
        .map(todo => (
          <li key={todo.id}>{todo.text}</li>
        ))}
    </ul>
  );
}
```

### 5. Performance Issues with Complex Conditions

Complex conditional rendering can cause performance issues:

```jsx
// Problematic - recalculating on every render
function FilteredList({ items }) {
  return (
    <ul>
      {items
        .filter(item => {
          // Complex filtering logic that runs on every render
          return complexFilterFunction(item);
        })
        .map(item => (
          <li key={item.id}>{item.name}</li>
        ))}
    </ul>
  );
}

// Better - memoizing the filtered items
function FilteredList({ items }) {
  const filteredItems = React.useMemo(() => {
    return items.filter(item => complexFilterFunction(item));
  }, [items]);
  
  return (
    <ul>
      {filteredItems.map(item => (
        <li key={item.id}>{item.name}</li>
      ))}
    </ul>
  );
}
```

## Summary

Conditional rendering is a powerful technique in React that allows you to create dynamic, responsive user interfaces. By mastering various conditional rendering patterns, you can build applications that adapt to different states, user roles, and data conditions.

Key takeaways:
- Use simple techniques like if statements, ternary operators, and the logical && operator for basic conditions
- Apply more advanced patterns for complex scenarios
- Handle loading, error, and empty states consistently
- Extract complex conditional logic into separate components
- Be mindful of performance implications when conditions involve expensive computations
- Consider accessibility when conditionally rendering UI elements

With these techniques, you can create React applications that provide excellent user experiences across different scenarios and states.

## Practice Exercise

Build a user profile page with the following features:

1. Show a loading spinner while fetching user data
2. Display an error message if the data fetch fails
3. Show different sections based on the user's role (admin, regular user)
4. Implement a tabbed interface to switch between profile, settings, and activity
5. Show an empty state when there's no activity data
6. Include a feature flag for a new "dark mode" feature

This exercise will help you practice various conditional rendering techniques in a realistic scenario.

## Additional Resources

- [React Official Documentation on Conditional Rendering](https://react.dev/learn/conditional-rendering)
- [Patterns.dev - Conditional Rendering](https://www.patterns.dev/posts/conditional-rendering)
- [Kent C. Dodds - Use ternaries rather than && in JSX](https://kentcdodds.com/blog/use-ternaries-rather-than-and-and-in-jsx)
