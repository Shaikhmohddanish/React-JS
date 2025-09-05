# React Best Practices - A Complete Guide

## Introduction to React Best Practices

Writing clean, maintainable, and performant React code requires following established best practices that have evolved from the React community's collective experience. This comprehensive guide covers essential practices for building scalable React applications, from component design patterns to performance optimization, testing strategies, and project organization.

## Why Best Practices Matter

Following React best practices is crucial because they:

1. **Improve Code Quality**: Lead to more reliable and bug-free applications
2. **Enhance Maintainability**: Make code easier to understand, modify, and extend
3. **Boost Performance**: Optimize rendering and reduce unnecessary operations
4. **Facilitate Collaboration**: Create consistent patterns that team members can follow
5. **Reduce Technical Debt**: Prevent accumulation of problematic code patterns
6. **Improve Developer Experience**: Make development more enjoyable and productive
7. **Future-Proof Applications**: Ensure compatibility with React updates and new features

## Component Design and Architecture

### 1. Component Composition over Inheritance

React favors composition over inheritance. Build complex UIs by combining simple components:

```jsx
// ❌ Bad: Trying to use inheritance-like patterns
class BaseButton extends React.Component {
  render() {
    return <button className="btn">{this.props.children}</button>;
  }
}

class PrimaryButton extends BaseButton {
  render() {
    return <button className="btn btn-primary">{this.props.children}</button>;
  }
}

// ✅ Good: Using composition
function Button({ variant = 'default', children, ...props }) {
  const className = `btn btn-${variant}`;
  
  return (
    <button className={className} {...props}>
      {children}
    </button>
  );
}

function PrimaryButton({ children, ...props }) {
  return (
    <Button variant="primary" {...props}>
      {children}
    </Button>
  );
}

// Even better: Using a more flexible approach
function Button({ variant = 'default', size = 'medium', children, className = '', ...props }) {
  const classes = [
    'btn',
    `btn-${variant}`,
    `btn-${size}`,
    className
  ].filter(Boolean).join(' ');
  
  return (
    <button className={classes} {...props}>
      {children}
    </button>
  );
}
```

### 2. Single Responsibility Principle

Each component should have a single, well-defined responsibility:

```jsx
// ❌ Bad: Component doing too many things
function UserProfilePage({ userId }) {
  const [user, setUser] = useState(null);
  const [posts, setPosts] = useState([]);
  const [followers, setFollowers] = useState([]);
  const [loading, setLoading] = useState(true);
  const [editMode, setEditMode] = useState(false);
  const [formData, setFormData] = useState({});

  // Lots of mixed logic for different concerns...

  return (
    <div>
      {/* Complex render logic mixing user info, posts, edit form */}
    </div>
  );
}

// ✅ Good: Separated responsibilities
function UserProfilePage({ userId }) {
  return (
    <div className="user-profile-page">
      <UserProfileHeader userId={userId} />
      <UserProfileContent userId={userId} />
    </div>
  );
}

function UserProfileHeader({ userId }) {
  const { user, loading, error } = useUser(userId);
  const [editMode, setEditMode] = useState(false);

  if (loading) return <UserProfileSkeleton />;
  if (error) return <ErrorMessage error={error} />;

  return (
    <header className="user-profile-header">
      {editMode ? (
        <UserEditForm user={user} onCancel={() => setEditMode(false)} />
      ) : (
        <UserInfo user={user} onEdit={() => setEditMode(true)} />
      )}
    </header>
  );
}

function UserProfileContent({ userId }) {
  return (
    <div className="user-profile-content">
      <UserPosts userId={userId} />
      <UserFollowers userId={userId} />
    </div>
  );
}
```

### 3. Props Design Patterns

Design props for clarity, flexibility, and type safety:

```jsx
// ✅ Good: Clear, well-designed props
interface ButtonProps {
  variant?: 'primary' | 'secondary' | 'danger' | 'ghost';
  size?: 'small' | 'medium' | 'large';
  disabled?: boolean;
  loading?: boolean;
  fullWidth?: boolean;
  onClick?: (event: React.MouseEvent<HTMLButtonElement>) => void;
  children: React.ReactNode;
  className?: string;
  'data-testid'?: string;
}

function Button({
  variant = 'primary',
  size = 'medium',
  disabled = false,
  loading = false,
  fullWidth = false,
  onClick,
  children,
  className = '',
  'data-testid': testId,
  ...rest
}: ButtonProps) {
  const classes = [
    'btn',
    `btn-${variant}`,
    `btn-${size}`,
    fullWidth && 'btn-full-width',
    loading && 'btn-loading',
    className
  ].filter(Boolean).join(' ');

  return (
    <button
      className={classes}
      disabled={disabled || loading}
      onClick={onClick}
      data-testid={testId}
      {...rest}
    >
      {loading ? <Spinner size="small" /> : children}
    </button>
  );
}

// ✅ Good: Render props pattern for flexibility
interface DataFetcherProps<T> {
  url: string;
  children: (props: {
    data: T | null;
    loading: boolean;
    error: Error | null;
    refetch: () => void;
  }) => React.ReactNode;
}

function DataFetcher<T>({ url, children }: DataFetcherProps<T>) {
  const { data, loading, error, refetch } = useFetch<T>(url);
  
  return children({ data, loading, error, refetch });
}

// Usage
function UserProfile({ userId }: { userId: string }) {
  return (
    <DataFetcher<User> url={`/api/users/${userId}`}>
      {({ data: user, loading, error, refetch }) => (
        <div>
          {loading && <Skeleton />}
          {error && <ErrorMessage error={error} onRetry={refetch} />}
          {user && <UserInfo user={user} />}
        </div>
      )}
    </DataFetcher>
  );
}
```

### 4. Component Patterns

#### Compound Components Pattern

```jsx
// ✅ Great: Compound component pattern for complex UI
interface TabsContextType {
  activeTab: string;
  setActiveTab: (tab: string) => void;
}

const TabsContext = React.createContext<TabsContextType | null>(null);

function Tabs({ defaultTab, children }: { defaultTab: string; children: React.ReactNode }) {
  const [activeTab, setActiveTab] = useState(defaultTab);
  
  return (
    <TabsContext.Provider value={{ activeTab, setActiveTab }}>
      <div className="tabs">{children}</div>
    </TabsContext.Provider>
  );
}

function TabList({ children }: { children: React.ReactNode }) {
  return <div className="tab-list" role="tablist">{children}</div>;
}

function Tab({ value, children }: { value: string; children: React.ReactNode }) {
  const context = useContext(TabsContext);
  if (!context) throw new Error('Tab must be used within Tabs');
  
  const { activeTab, setActiveTab } = context;
  const isActive = activeTab === value;
  
  return (
    <button
      className={`tab ${isActive ? 'tab-active' : ''}`}
      role="tab"
      aria-selected={isActive}
      onClick={() => setActiveTab(value)}
    >
      {children}
    </button>
  );
}

function TabPanels({ children }: { children: React.ReactNode }) {
  return <div className="tab-panels">{children}</div>;
}

function TabPanel({ value, children }: { value: string; children: React.ReactNode }) {
  const context = useContext(TabsContext);
  if (!context) throw new Error('TabPanel must be used within Tabs');
  
  const { activeTab } = context;
  
  if (activeTab !== value) return null;
  
  return (
    <div className="tab-panel" role="tabpanel">
      {children}
    </div>
  );
}

// Attach components as properties
Tabs.List = TabList;
Tabs.Tab = Tab;
Tabs.Panels = TabPanels;
Tabs.Panel = TabPanel;

// Usage
function App() {
  return (
    <Tabs defaultTab="profile">
      <Tabs.List>
        <Tabs.Tab value="profile">Profile</Tabs.Tab>
        <Tabs.Tab value="settings">Settings</Tabs.Tab>
        <Tabs.Tab value="notifications">Notifications</Tabs.Tab>
      </Tabs.List>
      
      <Tabs.Panels>
        <Tabs.Panel value="profile">
          <UserProfile />
        </Tabs.Panel>
        <Tabs.Panel value="settings">
          <UserSettings />
        </Tabs.Panel>
        <Tabs.Panel value="notifications">
          <NotificationSettings />
        </Tabs.Panel>
      </Tabs.Panels>
    </Tabs>
  );
}
```

## State Management Best Practices

### 1. Local vs Global State

Keep state as local as possible and only lift it up when necessary:

```jsx
// ❌ Bad: Unnecessary global state
function App() {
  const [users, setUsers] = useState([]);
  const [selectedUser, setSelectedUser] = useState(null);
  const [userEditMode, setUserEditMode] = useState(false);
  const [userFormData, setUserFormData] = useState({});
  
  return (
    <UserContext.Provider value={{
      users, setUsers,
      selectedUser, setSelectedUser,
      userEditMode, setUserEditMode,
      userFormData, setUserFormData
    }}>
      <UserManagement />
    </UserContext.Provider>
  );
}

// ✅ Good: Local state where appropriate
function UserManagement() {
  const [users, setUsers] = useState([]);
  
  return (
    <div>
      <UserList users={users} onUserSelect={handleUserSelect} />
      <UserDetail userId={selectedUserId} />
    </div>
  );
}

function UserDetail({ userId }) {
  const [editMode, setEditMode] = useState(false);
  const [formData, setFormData] = useState({});
  
  // State is local to this component where it's used
}
```

### 2. State Structure Design

Design state structures that are easy to update and reason about:

```jsx
// ❌ Bad: Nested state that's hard to update
const [state, setState] = useState({
  user: {
    profile: {
      personalInfo: {
        name: '',
        email: '',
        address: {
          street: '',
          city: '',
          country: ''
        }
      }
    }
  }
});

// Updating nested state is complex
const updateCity = (newCity) => {
  setState(prevState => ({
    ...prevState,
    user: {
      ...prevState.user,
      profile: {
        ...prevState.user.profile,
        personalInfo: {
          ...prevState.user.profile.personalInfo,
          address: {
            ...prevState.user.profile.personalInfo.address,
            city: newCity
          }
        }
      }
    }
  }));
};

// ✅ Good: Flat state structure with useReducer for complex state
interface UserState {
  name: string;
  email: string;
  street: string;
  city: string;
  country: string;
}

type UserAction =
  | { type: 'UPDATE_FIELD'; field: keyof UserState; value: string }
  | { type: 'RESET_FORM' }
  | { type: 'LOAD_USER'; user: UserState };

function userReducer(state: UserState, action: UserAction): UserState {
  switch (action.type) {
    case 'UPDATE_FIELD':
      return { ...state, [action.field]: action.value };
    case 'RESET_FORM':
      return { name: '', email: '', street: '', city: '', country: '' };
    case 'LOAD_USER':
      return action.user;
    default:
      return state;
  }
}

function UserForm() {
  const [state, dispatch] = useReducer(userReducer, {
    name: '', email: '', street: '', city: '', country: ''
  });
  
  const updateField = (field: keyof UserState, value: string) => {
    dispatch({ type: 'UPDATE_FIELD', field, value });
  };
  
  return (
    <form>
      <input
        value={state.name}
        onChange={(e) => updateField('name', e.target.value)}
      />
      <input
        value={state.city}
        onChange={(e) => updateField('city', e.target.value)}
      />
      {/* Much simpler updates */}
    </form>
  );
}
```

### 3. Derived State

Avoid storing derived state; compute it on the fly:

```jsx
// ❌ Bad: Storing derived state
function ShoppingCart() {
  const [items, setItems] = useState([]);
  const [totalPrice, setTotalPrice] = useState(0);
  const [itemCount, setItemCount] = useState(0);
  
  const addItem = (item) => {
    const newItems = [...items, item];
    setItems(newItems);
    setTotalPrice(newItems.reduce((sum, item) => sum + item.price, 0));
    setItemCount(newItems.length);
  };
  
  // Risk of state getting out of sync
}

// ✅ Good: Computing derived state
function ShoppingCart() {
  const [items, setItems] = useState([]);
  
  // Derived state computed on each render
  const totalPrice = useMemo(
    () => items.reduce((sum, item) => sum + item.price * item.quantity, 0),
    [items]
  );
  
  const itemCount = useMemo(
    () => items.reduce((sum, item) => sum + item.quantity, 0),
    [items]
  );
  
  const addItem = (item) => {
    setItems(prev => {
      const existingIndex = prev.findIndex(i => i.id === item.id);
      if (existingIndex >= 0) {
        const newItems = [...prev];
        newItems[existingIndex] = {
          ...newItems[existingIndex],
          quantity: newItems[existingIndex].quantity + 1
        };
        return newItems;
      }
      return [...prev, { ...item, quantity: 1 }];
    });
  };
  
  return (
    <div>
      <div>Items: {itemCount}</div>
      <div>Total: ${totalPrice.toFixed(2)}</div>
      {/* Cart items */}
    </div>
  );
}
```

## Performance Optimization

### 1. Memoization Strategies

Use React.memo, useMemo, and useCallback strategically:

```jsx
// ✅ Good: Strategic use of React.memo
interface UserCardProps {
  user: User;
  onEdit: (user: User) => void;
  onDelete: (userId: string) => void;
}

const UserCard = React.memo<UserCardProps>(({ user, onEdit, onDelete }) => {
  const handleEdit = useCallback(() => {
    onEdit(user);
  }, [user, onEdit]);
  
  const handleDelete = useCallback(() => {
    onDelete(user.id);
  }, [user.id, onDelete]);
  
  return (
    <div className="user-card">
      <img src={user.avatar} alt={user.name} />
      <h3>{user.name}</h3>
      <p>{user.email}</p>
      <button onClick={handleEdit}>Edit</button>
      <button onClick={handleDelete}>Delete</button>
    </div>
  );
});

// ✅ Good: Memoizing expensive calculations
function DataVisualization({ data }) {
  const processedData = useMemo(() => {
    // Expensive data processing
    return data.map(item => ({
      ...item,
      processed: complexCalculation(item),
      formatted: formatData(item)
    })).sort((a, b) => b.processed - a.processed);
  }, [data]);
  
  const chartConfig = useMemo(() => ({
    type: 'line',
    data: processedData,
    options: {
      responsive: true,
      scales: {
        x: { type: 'time' },
        y: { beginAtZero: true }
      }
    }
  }), [processedData]);
  
  return <Chart config={chartConfig} />;
}

// ✅ Good: Memoizing stable callbacks
function TodoList({ todos, onToggle, onDelete }) {
  const handleToggle = useCallback((id: string) => {
    onToggle(id);
  }, [onToggle]);
  
  const handleDelete = useCallback((id: string) => {
    onDelete(id);
  }, [onDelete]);
  
  return (
    <ul>
      {todos.map(todo => (
        <TodoItem
          key={todo.id}
          todo={todo}
          onToggle={handleToggle}
          onDelete={handleDelete}
        />
      ))}
    </ul>
  );
}
```

### 2. Code Splitting and Lazy Loading

Implement code splitting for better initial load performance:

```jsx
// ✅ Good: Route-based code splitting
import { lazy, Suspense } from 'react';
import { BrowserRouter, Routes, Route } from 'react-router-dom';

const Home = lazy(() => import('./pages/Home'));
const Dashboard = lazy(() => import('./pages/Dashboard'));
const Profile = lazy(() => import('./pages/Profile'));
const Settings = lazy(() => import('./pages/Settings'));

function App() {
  return (
    <BrowserRouter>
      <Suspense fallback={<PageLoadingSpinner />}>
        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/dashboard" element={<Dashboard />} />
          <Route path="/profile" element={<Profile />} />
          <Route path="/settings" element={<Settings />} />
        </Routes>
      </Suspense>
    </BrowserRouter>
  );
}

// ✅ Good: Component-based code splitting
function DataVisualization({ type }) {
  const [ChartComponent, setChartComponent] = useState(null);
  
  useEffect(() => {
    const loadChart = async () => {
      try {
        const module = await import(`./charts/${type}Chart`);
        setChartComponent(() => module.default);
      } catch (error) {
        console.error('Failed to load chart component:', error);
      }
    };
    
    loadChart();
  }, [type]);
  
  if (!ChartComponent) {
    return <ChartLoadingSkeleton />;
  }
  
  return <ChartComponent />;
}

// ✅ Good: Preloading components
function ProductCard({ product }) {
  const [isPreloaded, setIsPreloaded] = useState(false);
  
  const preloadProductDetail = useCallback(() => {
    if (!isPreloaded) {
      import('./ProductDetail').then(() => {
        setIsPreloaded(true);
      });
    }
  }, [isPreloaded]);
  
  return (
    <div 
      className="product-card"
      onMouseEnter={preloadProductDetail}
    >
      <img src={product.image} alt={product.name} />
      <h3>{product.name}</h3>
      <p>${product.price}</p>
    </div>
  );
}
```

### 3. List Rendering Optimization

Optimize rendering of large lists:

```jsx
// ✅ Good: Virtualized list for large datasets
import { FixedSizeList as List } from 'react-window';

interface VirtualizedListProps {
  items: any[];
  renderItem: (item: any, index: number) => React.ReactNode;
  itemHeight: number;
  height: number;
}

function VirtualizedList({ items, renderItem, itemHeight, height }: VirtualizedListProps) {
  const Row = ({ index, style }) => (
    <div style={style}>
      {renderItem(items[index], index)}
    </div>
  );
  
  return (
    <List
      height={height}
      itemCount={items.length}
      itemSize={itemHeight}
      itemData={items}
    >
      {Row}
    </List>
  );
}

// ✅ Good: Optimized list with proper keys and memoization
const ProductListItem = React.memo<{ product: Product; onSelect: (id: string) => void }>(
  ({ product, onSelect }) => {
    const handleClick = useCallback(() => {
      onSelect(product.id);
    }, [product.id, onSelect]);
    
    return (
      <div className="product-item" onClick={handleClick}>
        <img src={product.thumbnail} alt={product.name} loading="lazy" />
        <h3>{product.name}</h3>
        <p>${product.price}</p>
      </div>
    );
  }
);

function ProductList({ products, onProductSelect }) {
  const handleProductSelect = useCallback((productId: string) => {
    onProductSelect(productId);
  }, [onProductSelect]);
  
  return (
    <div className="product-list">
      {products.map(product => (
        <ProductListItem
          key={product.id}
          product={product}
          onSelect={handleProductSelect}
        />
      ))}
    </div>
  );
}
```

## Error Handling and Resilience

### 1. Error Boundaries

Implement comprehensive error boundaries:

```jsx
interface ErrorBoundaryState {
  hasError: boolean;
  error: Error | null;
  errorInfo: React.ErrorInfo | null;
}

class ErrorBoundary extends React.Component<
  React.PropsWithChildren<{
    fallback?: React.ComponentType<{ error: Error; resetError: () => void }>;
    onError?: (error: Error, errorInfo: React.ErrorInfo) => void;
  }>,
  ErrorBoundaryState
> {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null, errorInfo: null };
  }

  static getDerivedStateFromError(error: Error): Partial<ErrorBoundaryState> {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, errorInfo: React.ErrorInfo) {
    this.setState({ errorInfo });
    this.props.onError?.(error, errorInfo);
    
    // Log to error reporting service
    console.error('ErrorBoundary caught an error:', error, errorInfo);
  }

  resetError = () => {
    this.setState({ hasError: false, error: null, errorInfo: null });
  };

  render() {
    if (this.state.hasError) {
      const FallbackComponent = this.props.fallback || DefaultErrorFallback;
      return (
        <FallbackComponent 
          error={this.state.error!} 
          resetError={this.resetError} 
        />
      );
    }

    return this.props.children;
  }
}

function DefaultErrorFallback({ error, resetError }: { error: Error; resetError: () => void }) {
  return (
    <div className="error-boundary">
      <h2>Something went wrong</h2>
      <details style={{ whiteSpace: 'pre-wrap' }}>
        {error && error.toString()}
      </details>
      <button onClick={resetError}>Try again</button>
    </div>
  );
}

// Usage with specific error handling
function App() {
  return (
    <ErrorBoundary
      fallback={CustomErrorPage}
      onError={(error, errorInfo) => {
        // Send to error reporting service
        errorReportingService.captureException(error, { extra: errorInfo });
      }}
    >
      <Router>
        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/dashboard" element={
            <ErrorBoundary fallback={DashboardErrorFallback}>
              <Dashboard />
            </ErrorBoundary>
          } />
        </Routes>
      </Router>
    </ErrorBoundary>
  );
}
```

### 2. Graceful Error Handling in Components

```jsx
// ✅ Good: Comprehensive error handling
function UserProfile({ userId }: { userId: string }) {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);
  const [retryCount, setRetryCount] = useState(0);

  const fetchUser = useCallback(async () => {
    try {
      setLoading(true);
      setError(null);
      
      const userData = await userService.getUser(userId);
      setUser(userData);
    } catch (err) {
      const errorMessage = err instanceof Error ? err.message : 'Unknown error occurred';
      setError(errorMessage);
      
      // Log error for debugging
      console.error('Failed to fetch user:', err);
    } finally {
      setLoading(false);
    }
  }, [userId]);

  useEffect(() => {
    fetchUser();
  }, [fetchUser]);

  const handleRetry = () => {
    setRetryCount(prev => prev + 1);
    fetchUser();
  };

  if (loading) {
    return <UserProfileSkeleton />;
  }

  if (error) {
    return (
      <ErrorMessage
        title="Failed to load user profile"
        message={error}
        onRetry={handleRetry}
        retryCount={retryCount}
      />
    );
  }

  if (!user) {
    return <EmptyState message="User not found" />;
  }

  return <UserInfo user={user} />;
}

// Reusable error message component
interface ErrorMessageProps {
  title: string;
  message: string;
  onRetry?: () => void;
  retryCount?: number;
  maxRetries?: number;
}

function ErrorMessage({ 
  title, 
  message, 
  onRetry, 
  retryCount = 0, 
  maxRetries = 3 
}: ErrorMessageProps) {
  const canRetry = onRetry && retryCount < maxRetries;
  
  return (
    <div className="error-message">
      <h3>{title}</h3>
      <p>{message}</p>
      {canRetry && (
        <button onClick={onRetry}>
          Retry {retryCount > 0 && `(${retryCount}/${maxRetries})`}
        </button>
      )}
      {retryCount >= maxRetries && (
        <p className="max-retries-message">
          Maximum retry attempts reached. Please try again later.
        </p>
      )}
    </div>
  );
}
```

## Testing Best Practices

### 1. Component Testing

Write focused, maintainable tests:

```jsx
// ✅ Good: Well-structured component tests
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { UserProfile } from './UserProfile';
import { userService } from '../services/userService';

// Mock the service
jest.mock('../services/userService');
const mockedUserService = userService as jest.Mocked<typeof userService>;

describe('UserProfile', () => {
  const mockUser = {
    id: '1',
    name: 'John Doe',
    email: 'john@example.com',
    avatar: 'avatar.jpg'
  };

  beforeEach(() => {
    jest.clearAllMocks();
  });

  it('displays loading state initially', () => {
    mockedUserService.getUser.mockImplementation(() => 
      new Promise(() => {}) // Never resolves
    );

    render(<UserProfile userId="1" />);
    
    expect(screen.getByTestId('user-profile-skeleton')).toBeInTheDocument();
  });

  it('displays user information when loaded successfully', async () => {
    mockedUserService.getUser.mockResolvedValue(mockUser);

    render(<UserProfile userId="1" />);

    await waitFor(() => {
      expect(screen.getByText('John Doe')).toBeInTheDocument();
    });
    
    expect(screen.getByText('john@example.com')).toBeInTheDocument();
    expect(screen.getByAltText('John Doe')).toHaveAttribute('src', 'avatar.jpg');
  });

  it('displays error message when loading fails', async () => {
    const errorMessage = 'Failed to load user';
    mockedUserService.getUser.mockRejectedValue(new Error(errorMessage));

    render(<UserProfile userId="1" />);

    await waitFor(() => {
      expect(screen.getByText('Failed to load user profile')).toBeInTheDocument();
    });
    
    expect(screen.getByText(errorMessage)).toBeInTheDocument();
  });

  it('retries loading when retry button is clicked', async () => {
    mockedUserService.getUser
      .mockRejectedValueOnce(new Error('Network error'))
      .mockResolvedValue(mockUser);

    render(<UserProfile userId="1" />);

    // Wait for error state
    await waitFor(() => {
      expect(screen.getByText('Failed to load user profile')).toBeInTheDocument();
    });

    // Click retry
    const retryButton = screen.getByText('Retry');
    await userEvent.click(retryButton);

    // Wait for successful load
    await waitFor(() => {
      expect(screen.getByText('John Doe')).toBeInTheDocument();
    });

    expect(mockedUserService.getUser).toHaveBeenCalledTimes(2);
  });
});

// ✅ Good: Testing custom hooks
import { renderHook, act } from '@testing-library/react';
import { useLocalStorage } from './useLocalStorage';

describe('useLocalStorage', () => {
  beforeEach(() => {
    localStorage.clear();
  });

  it('returns initial value when no stored value exists', () => {
    const { result } = renderHook(() => useLocalStorage('test-key', 'default'));
    
    expect(result.current[0]).toBe('default');
  });

  it('returns stored value when it exists', () => {
    localStorage.setItem('test-key', JSON.stringify('stored-value'));
    
    const { result } = renderHook(() => useLocalStorage('test-key', 'default'));
    
    expect(result.current[0]).toBe('stored-value');
  });

  it('updates localStorage when value changes', () => {
    const { result } = renderHook(() => useLocalStorage('test-key', 'initial'));
    
    act(() => {
      result.current[1]('updated-value');
    });
    
    expect(result.current[0]).toBe('updated-value');
    expect(localStorage.getItem('test-key')).toBe('"updated-value"');
  });
});
```

### 2. Integration Testing

Test component interactions and data flow:

```jsx
// ✅ Good: Integration test
import { render, screen, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { BrowserRouter } from 'react-router-dom';
import { UserManagement } from './UserManagement';
import { userService } from '../services/userService';

jest.mock('../services/userService');
const mockedUserService = userService as jest.Mocked<typeof userService>;

function renderWithRouter(component: React.ReactElement) {
  return render(<BrowserRouter>{component}</BrowserRouter>);
}

describe('UserManagement Integration', () => {
  const mockUsers = [
    { id: '1', name: 'John Doe', email: 'john@example.com' },
    { id: '2', name: 'Jane Smith', email: 'jane@example.com' }
  ];

  beforeEach(() => {
    jest.clearAllMocks();
    mockedUserService.getUsers.mockResolvedValue({ users: mockUsers, total: 2 });
  });

  it('displays users and allows creating a new user', async () => {
    const newUser = { id: '3', name: 'Bob Wilson', email: 'bob@example.com' };
    mockedUserService.createUser.mockResolvedValue(newUser);

    renderWithRouter(<UserManagement />);

    // Wait for users to load
    await waitFor(() => {
      expect(screen.getByText('John Doe')).toBeInTheDocument();
    });

    // Click create user button
    const createButton = screen.getByText('Create User');
    await userEvent.click(createButton);

    // Fill out form
    const nameInput = screen.getByLabelText('Name');
    const emailInput = screen.getByLabelText('Email');
    
    await userEvent.type(nameInput, 'Bob Wilson');
    await userEvent.type(emailInput, 'bob@example.com');

    // Submit form
    const submitButton = screen.getByText('Create');
    await userEvent.click(submitButton);

    // Verify user was created
    await waitFor(() => {
      expect(screen.getByText('Bob Wilson')).toBeInTheDocument();
    });

    expect(mockedUserService.createUser).toHaveBeenCalledWith({
      name: 'Bob Wilson',
      email: 'bob@example.com'
    });
  });
});
```

## Code Organization and Project Structure

### 1. Folder Structure

Organize your project for scalability:

```
src/
├── components/           # Reusable UI components
│   ├── common/          # Generic components (Button, Input, Modal)
│   │   ├── Button/
│   │   │   ├── Button.tsx
│   │   │   ├── Button.test.tsx
│   │   │   ├── Button.stories.tsx
│   │   │   └── index.ts
│   │   └── index.ts     # Barrel exports
│   ├── forms/           # Form-specific components
│   └── layout/          # Layout components (Header, Footer, Sidebar)
├── pages/               # Page components
│   ├── Home/
│   ├── Dashboard/
│   └── Profile/
├── hooks/               # Custom hooks
│   ├── useApi.ts
│   ├── useLocalStorage.ts
│   └── index.ts
├── services/            # API services and external integrations
│   ├── api/
│   │   ├── userService.ts
│   │   ├── productService.ts
│   │   └── index.ts
│   └── utils/
├── store/               # State management
│   ├── slices/          # Redux slices or Zustand stores
│   ├── providers/       # Context providers
│   └── index.ts
├── types/               # TypeScript type definitions
│   ├── api.ts
│   ├── common.ts
│   └── index.ts
├── utils/               # Utility functions
│   ├── helpers.ts
│   ├── constants.ts
│   ├── formatters.ts
│   └── validators.ts
├── styles/              # Global styles and themes
│   ├── globals.css
│   ├── variables.css
│   └── themes/
└── __tests__/           # Test utilities and setup
    ├── setup.ts
    ├── mocks/
    └── utils/
```

### 2. Component Export Patterns

Use consistent export patterns:

```jsx
// ✅ Good: Named exports with barrel files

// components/Button/Button.tsx
export interface ButtonProps {
  variant?: 'primary' | 'secondary';
  children: React.ReactNode;
}

export function Button({ variant = 'primary', children }: ButtonProps) {
  return <button className={`btn btn-${variant}`}>{children}</button>;
}

// components/Button/index.ts
export { Button, type ButtonProps } from './Button';

// components/index.ts (barrel file)
export { Button, type ButtonProps } from './Button';
export { Input, type InputProps } from './Input';
export { Modal, type ModalProps } from './Modal';

// Usage
import { Button, Input, Modal } from '../components';
```

### 3. Configuration and Environment

Centralize configuration:

```typescript
// config/index.ts
interface Config {
  apiUrl: string;
  environment: 'development' | 'staging' | 'production';
  features: {
    enableAnalytics: boolean;
    enableDebugMode: boolean;
    maxRetries: number;
  };
}

function getConfig(): Config {
  const environment = (process.env.NODE_ENV || 'development') as Config['environment'];
  
  const baseConfig: Config = {
    apiUrl: process.env.REACT_APP_API_URL || 'http://localhost:3001',
    environment,
    features: {
      enableAnalytics: environment === 'production',
      enableDebugMode: environment === 'development',
      maxRetries: parseInt(process.env.REACT_APP_MAX_RETRIES || '3', 10),
    },
  };

  // Environment-specific overrides
  const environmentConfigs: Partial<Record<Config['environment'], Partial<Config>>> = {
    development: {
      features: {
        ...baseConfig.features,
        enableDebugMode: true,
      },
    },
    staging: {
      features: {
        ...baseConfig.features,
        enableAnalytics: false,
      },
    },
  };

  return {
    ...baseConfig,
    ...environmentConfigs[environment],
  };
}

export const config = getConfig();

// utils/constants.ts
export const BREAKPOINTS = {
  mobile: '768px',
  tablet: '1024px',
  desktop: '1200px',
} as const;

export const COLORS = {
  primary: '#007bff',
  secondary: '#6c757d',
  success: '#28a745',
  danger: '#dc3545',
  warning: '#ffc107',
  info: '#17a2b8',
} as const;

export const API_ENDPOINTS = {
  users: '/users',
  products: '/products',
  orders: '/orders',
} as const;
```

## Accessibility Best Practices

### 1. Semantic HTML and ARIA

Use proper semantic elements and ARIA attributes:

```jsx
// ✅ Good: Accessible component design
interface DropdownProps {
  trigger: React.ReactNode;
  children: React.ReactNode;
  label?: string;
}

function Dropdown({ trigger, children, label }: DropdownProps) {
  const [isOpen, setIsOpen] = useState(false);
  const dropdownRef = useRef<HTMLDivElement>(null);
  const triggerRef = useRef<HTMLButtonElement>(null);

  // Handle keyboard navigation
  const handleKeyDown = (event: React.KeyboardEvent) => {
    switch (event.key) {
      case 'Escape':
        setIsOpen(false);
        triggerRef.current?.focus();
        break;
      case 'ArrowDown':
        event.preventDefault();
        if (!isOpen) {
          setIsOpen(true);
        } else {
          // Focus first menu item
          const firstItem = dropdownRef.current?.querySelector('[role="menuitem"]') as HTMLElement;
          firstItem?.focus();
        }
        break;
    }
  };

  return (
    <div className="dropdown" onKeyDown={handleKeyDown}>
      <button
        ref={triggerRef}
        aria-haspopup="menu"
        aria-expanded={isOpen}
        aria-label={label}
        onClick={() => setIsOpen(!isOpen)}
      >
        {trigger}
      </button>
      
      {isOpen && (
        <div
          ref={dropdownRef}
          role="menu"
          className="dropdown-menu"
          aria-label={label || 'Dropdown menu'}
        >
          {children}
        </div>
      )}
    </div>
  );
}

function DropdownItem({ children, onClick }: { children: React.ReactNode; onClick: () => void }) {
  const handleKeyDown = (event: React.KeyboardEvent) => {
    if (event.key === 'Enter' || event.key === ' ') {
      event.preventDefault();
      onClick();
    }
  };

  return (
    <div
      role="menuitem"
      tabIndex={0}
      className="dropdown-item"
      onClick={onClick}
      onKeyDown={handleKeyDown}
    >
      {children}
    </div>
  );
}

// ✅ Good: Form accessibility
function ContactForm() {
  const [errors, setErrors] = useState<Record<string, string>>({});
  
  return (
    <form aria-labelledby="contact-form-title">
      <h2 id="contact-form-title">Contact Us</h2>
      
      <div className="form-group">
        <label htmlFor="name">
          Name <span aria-label="required">*</span>
        </label>
        <input
          id="name"
          type="text"
          required
          aria-invalid={!!errors.name}
          aria-describedby={errors.name ? 'name-error' : undefined}
        />
        {errors.name && (
          <div id="name-error" role="alert" className="error-message">
            {errors.name}
          </div>
        )}
      </div>
      
      <button type="submit" aria-describedby="submit-help">
        Send Message
      </button>
      <div id="submit-help" className="help-text">
        We'll respond within 24 hours
      </div>
    </form>
  );
}
```

### 2. Focus Management

Implement proper focus management:

```jsx
// ✅ Good: Focus management in modals
function Modal({ isOpen, onClose, children, title }: ModalProps) {
  const modalRef = useRef<HTMLDivElement>(null);
  const previousActiveElementRef = useRef<HTMLElement | null>(null);

  useEffect(() => {
    if (isOpen) {
      // Store previously focused element
      previousActiveElementRef.current = document.activeElement as HTMLElement;
      
      // Focus the modal
      modalRef.current?.focus();
      
      // Trap focus within modal
      const trapFocus = (event: KeyboardEvent) => {
        if (event.key === 'Tab') {
          const focusableElements = modalRef.current?.querySelectorAll(
            'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
          );
          
          if (focusableElements) {
            const firstElement = focusableElements[0] as HTMLElement;
            const lastElement = focusableElements[focusableElements.length - 1] as HTMLElement;
            
            if (event.shiftKey && document.activeElement === firstElement) {
              lastElement.focus();
              event.preventDefault();
            } else if (!event.shiftKey && document.activeElement === lastElement) {
              firstElement.focus();
              event.preventDefault();
            }
          }
        }
      };
      
      document.addEventListener('keydown', trapFocus);
      
      return () => {
        document.removeEventListener('keydown', trapFocus);
        
        // Restore focus when modal closes
        if (previousActiveElementRef.current) {
          previousActiveElementRef.current.focus();
        }
      };
    }
  }, [isOpen]);

  if (!isOpen) return null;

  return (
    <div
      className="modal-overlay"
      onClick={(e) => e.target === e.currentTarget && onClose()}
    >
      <div
        ref={modalRef}
        role="dialog"
        aria-modal="true"
        aria-labelledby="modal-title"
        className="modal"
        tabIndex={-1}
      >
        <div className="modal-header">
          <h2 id="modal-title">{title}</h2>
          <button
            onClick={onClose}
            aria-label="Close modal"
            className="modal-close"
          >
            ×
          </button>
        </div>
        <div className="modal-content">
          {children}
        </div>
      </div>
    </div>
  );
}

// ✅ Good: Skip navigation link
function SkipNavigation() {
  return (
    <a
      href="#main-content"
      className="skip-nav"
      onFocus={(e) => e.target.style.transform = 'translateY(0)'}
      onBlur={(e) => e.target.style.transform = 'translateY(-100%)'}
    >
      Skip to main content
    </a>
  );
}
```

## Security Best Practices

### 1. Input Sanitization and Validation

Always validate and sanitize user input:

```jsx
// ✅ Good: Input validation and sanitization
import DOMPurify from 'dompurify';

interface CommentFormProps {
  onSubmit: (comment: string) => void;
}

function CommentForm({ onSubmit }: CommentFormProps) {
  const [comment, setComment] = useState('');
  const [errors, setErrors] = useState<string[]>([]);

  const validateComment = (text: string): string[] => {
    const errors: string[] = [];
    
    if (!text.trim()) {
      errors.push('Comment cannot be empty');
    }
    
    if (text.length > 1000) {
      errors.push('Comment must be less than 1000 characters');
    }
    
    // Check for potentially malicious content
    const suspiciousPatterns = [
      /<script/i,
      /javascript:/i,
      /on\w+\s*=/i,
    ];
    
    if (suspiciousPatterns.some(pattern => pattern.test(text))) {
      errors.push('Comment contains invalid content');
    }
    
    return errors;
  };

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    
    const validationErrors = validateComment(comment);
    setErrors(validationErrors);
    
    if (validationErrors.length === 0) {
      // Sanitize the comment before submitting
      const sanitizedComment = DOMPurify.sanitize(comment);
      onSubmit(sanitizedComment);
      setComment('');
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <div className="form-group">
        <label htmlFor="comment">Comment</label>
        <textarea
          id="comment"
          value={comment}
          onChange={(e) => setComment(e.target.value)}
          maxLength={1000}
          aria-describedby={errors.length > 0 ? 'comment-errors' : undefined}
        />
        <div className="character-count">
          {comment.length}/1000 characters
        </div>
      </div>
      
      {errors.length > 0 && (
        <div id="comment-errors" role="alert" className="error-list">
          {errors.map((error, index) => (
            <div key={index} className="error-message">
              {error}
            </div>
          ))}
        </div>
      )}
      
      <button type="submit" disabled={errors.length > 0}>
        Submit Comment
      </button>
    </form>
  );
}

// ✅ Good: Safe rendering of user content
interface UserContentProps {
  content: string;
  allowedTags?: string[];
}

function UserContent({ content, allowedTags = ['b', 'i', 'em', 'strong'] }: UserContentProps) {
  const sanitizedContent = useMemo(() => {
    return DOMPurify.sanitize(content, {
      ALLOWED_TAGS: allowedTags,
      ALLOWED_ATTR: [],
    });
  }, [content, allowedTags]);

  return (
    <div 
      className="user-content"
      dangerouslySetInnerHTML={{ __html: sanitizedContent }}
    />
  );
}
```

### 2. Environment Variable Security

Handle sensitive data properly:

```jsx
// ✅ Good: Environment variable handling
// config/security.ts
interface SecurityConfig {
  apiUrl: string;
  publicKey: string;
  // Never expose private keys or secrets in frontend
}

function getSecurityConfig(): SecurityConfig {
  // Validate required environment variables
  const requiredVars = {
    apiUrl: process.env.REACT_APP_API_URL,
    publicKey: process.env.REACT_APP_PUBLIC_KEY,
  };

  const missingVars = Object.entries(requiredVars)
    .filter(([_, value]) => !value)
    .map(([key]) => key);

  if (missingVars.length > 0) {
    throw new Error(`Missing required environment variables: ${missingVars.join(', ')}`);
  }

  return {
    apiUrl: requiredVars.apiUrl!,
    publicKey: requiredVars.publicKey!,
  };
}

export const securityConfig = getSecurityConfig();

// ✅ Good: API token handling
class ApiClient {
  private baseURL: string;
  private token: string | null = null;

  constructor(baseURL: string) {
    this.baseURL = baseURL;
  }

  setToken(token: string) {
    this.token = token;
  }

  clearToken() {
    this.token = null;
  }

  private getHeaders(): Record<string, string> {
    const headers: Record<string, string> = {
      'Content-Type': 'application/json',
    };

    if (this.token) {
      headers.Authorization = `Bearer ${this.token}`;
    }

    return headers;
  }

  async request<T>(endpoint: string, options: RequestInit = {}): Promise<T> {
    const url = `${this.baseURL}${endpoint}`;
    
    const response = await fetch(url, {
      ...options,
      headers: {
        ...this.getHeaders(),
        ...options.headers,
      },
    });

    if (!response.ok) {
      if (response.status === 401) {
        // Clear invalid token
        this.clearToken();
        // Redirect to login or trigger auth refresh
        window.location.href = '/login';
      }
      
      throw new Error(`API request failed: ${response.statusText}`);
    }

    return response.json();
  }
}
```

## Summary

This comprehensive guide covered essential React best practices:

1. **Component Design**: Composition over inheritance, single responsibility, and proper prop design
2. **State Management**: Local vs global state, flat structures, and derived state patterns
3. **Performance**: Strategic memoization, code splitting, and list optimization
4. **Error Handling**: Error boundaries, graceful degradation, and resilient components
5. **Testing**: Component testing, integration testing, and custom hook testing
6. **Code Organization**: Scalable folder structure, consistent exports, and configuration management
7. **Accessibility**: Semantic HTML, ARIA attributes, and focus management
8. **Security**: Input validation, sanitization, and secure API handling

Key principles to remember:

- **Keep components small and focused**
- **Favor composition over complex hierarchies**
- **Handle errors gracefully at every level**
- **Test behavior, not implementation details**
- **Make accessibility a priority from the start**
- **Optimize performance based on real metrics**
- **Maintain consistent code organization**
- **Never trust user input**

By following these practices, you'll build React applications that are maintainable, performant, accessible, and secure.

## Final Exercise

Create a complete task management application that demonstrates all the best practices covered:

1. **Component Architecture**: Well-structured, composable components
2. **State Management**: Proper local/global state with Context and useReducer
3. **Performance**: Optimized list rendering with virtualization
4. **Error Handling**: Comprehensive error boundaries and user feedback
5. **Testing**: Unit and integration tests with good coverage
6. **Accessibility**: Full keyboard navigation and screen reader support
7. **Security**: Input validation and XSS prevention
8. **Code Organization**: Clean folder structure and TypeScript types

This project will serve as a practical application of all the concepts covered in this React tutorial series.

## Additional Resources

- [React Official Documentation](https://react.dev/)
- [React Best Practices Guide](https://react.dev/learn/thinking-in-react)
- [Testing Library Documentation](https://testing-library.com/docs/react-testing-library/intro/)
- [React Performance Optimization](https://react.dev/learn/render-and-commit)
- [Accessibility in React](https://react.dev/learn/accessibility)
- [React Security Best Practices](https://snyk.io/blog/10-react-security-best-practices/)
- [React TypeScript Cheatsheet](https://react-typescript-cheatsheet.netlify.app/)
