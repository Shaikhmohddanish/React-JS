# Components and Layouts in Next.js - A Complete Guide

## Introduction to Next.js Components and Layouts

Components and layouts are the building blocks of any Next.js application. While components encapsulate reusable pieces of UI logic, layouts provide consistent structure across multiple pages. Next.js enhances React's component system with server and client components, built-in layout support, and powerful composition patterns. This guide covers everything you need to know about building scalable component architectures in Next.js.

## Understanding Server vs Client Components

### Server Components (Default in App Router)

Server components run on the server and render to static HTML, reducing the JavaScript bundle sent to the client:

```tsx
// src/components/ProductList.tsx (Server Component by default)
import { getProducts } from '@/lib/api';
import { ProductCard } from './ProductCard';

// This component runs on the server
export async function ProductList() {
  // Data fetching happens on the server
  const products = await getProducts();
  
  return (
    <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
      {products.map((product) => (
        <ProductCard key={product.id} product={product} />
      ))}
    </div>
  );
}
```

### Client Components

Client components run in the browser and have access to React hooks and browser APIs:

```tsx
// src/components/AddToCartButton.tsx
'use client'; // This directive makes it a client component

import { useState } from 'react';
import { useCart } from '@/hooks/useCart';
import { Button } from '@/components/ui/Button';

interface AddToCartButtonProps {
  productId: string;
  productName: string;
  price: number;
}

export function AddToCartButton({ productId, productName, price }: AddToCartButtonProps) {
  const [isLoading, setIsLoading] = useState(false);
  const { addItem } = useCart();

  const handleAddToCart = async () => {
    setIsLoading(true);
    
    try {
      await addItem({
        id: productId,
        name: productName,
        price,
        quantity: 1,
      });
      
      // Show success message
      toast.success(`${productName} added to cart!`);
    } catch (error) {
      toast.error('Failed to add item to cart');
    } finally {
      setIsLoading(false);
    }
  };

  return (
    <Button 
      onClick={handleAddToCart}
      disabled={isLoading}
      className="w-full"
    >
      {isLoading ? 'Adding...' : 'Add to Cart'}
    </Button>
  );
}
```

### Hybrid Component Patterns

Combine server and client components for optimal performance:

```tsx
// src/components/ProductCard.tsx (Server Component)
import { AddToCartButton } from './AddToCartButton';
import { WishlistButton } from './WishlistButton';
import Image from 'next/image';

interface ProductCardProps {
  product: {
    id: string;
    name: string;
    price: number;
    image: string;
    description: string;
    inStock: boolean;
  };
}

// Server component - no JavaScript sent to client
export function ProductCard({ product }: ProductCardProps) {
  return (
    <div className="bg-white rounded-lg shadow-md overflow-hidden">
      <div className="relative aspect-square">
        <Image
          src={product.image}
          alt={product.name}
          fill
          className="object-cover"
          sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw"
        />
        
        {/* Client component for interactivity */}
        <WishlistButton productId={product.id} />
      </div>
      
      <div className="p-4">
        <h3 className="text-lg font-semibold mb-2">{product.name}</h3>
        <p className="text-gray-600 text-sm mb-3">{product.description}</p>
        
        <div className="flex justify-between items-center">
          <span className="text-xl font-bold">${product.price}</span>
          <span className={`text-sm ${product.inStock ? 'text-green-600' : 'text-red-600'}`}>
            {product.inStock ? 'In Stock' : 'Out of Stock'}
          </span>
        </div>
        
        <div className="mt-4 space-y-2">
          {/* Client component for cart functionality */}
          <AddToCartButton
            productId={product.id}
            productName={product.name}
            price={product.price}
          />
        </div>
      </div>
    </div>
  );
}
```

## Layout System in Next.js

### Root Layout (Required)

Every Next.js app must have a root layout:

```tsx
// src/app/layout.tsx
import type { Metadata } from 'next';
import { Inter } from 'next/font/google';
import { ThemeProvider } from '@/components/providers/ThemeProvider';
import { CartProvider } from '@/components/providers/CartProvider';
import { Toaster } from '@/components/ui/Toaster';
import './globals.css';

const inter = Inter({ subsets: ['latin'] });

export const metadata: Metadata = {
  title: {
    default: 'My E-commerce Store',
    template: '%s | My E-commerce Store',
  },
  description: 'The best online shopping experience',
  keywords: ['e-commerce', 'shopping', 'online store'],
};

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en" suppressHydrationWarning>
      <body className={inter.className}>
        <ThemeProvider>
          <CartProvider>
            <div className="min-h-screen bg-background">
              {children}
            </div>
            <Toaster />
          </CartProvider>
        </ThemeProvider>
      </body>
    </html>
  );
}
```

### Nested Layouts

Create layouts for specific sections of your application:

```tsx
// src/app/(shop)/layout.tsx
import { ShopHeader } from '@/components/shop/ShopHeader';
import { ShopFooter } from '@/components/shop/ShopFooter';
import { ShopSidebar } from '@/components/shop/ShopSidebar';

export default function ShopLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <div className="shop-layout">
      <ShopHeader />
      
      <div className="flex min-h-screen">
        <aside className="w-64 bg-gray-50 p-4">
          <ShopSidebar />
        </aside>
        
        <main className="flex-1 p-6">
          {children}
        </main>
      </div>
      
      <ShopFooter />
    </div>
  );
}
```

```tsx
// src/app/dashboard/layout.tsx
import { DashboardNav } from '@/components/dashboard/DashboardNav';
import { DashboardSidebar } from '@/components/dashboard/DashboardSidebar';
import { auth } from '@/lib/auth';
import { redirect } from 'next/navigation';

export default async function DashboardLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  const session = await auth();
  
  if (!session) {
    redirect('/login');
  }

  return (
    <div className="dashboard-layout">
      <DashboardNav user={session.user} />
      
      <div className="flex">
        <DashboardSidebar />
        
        <main className="flex-1 p-6">
          <div className="max-w-7xl mx-auto">
            {children}
          </div>
        </main>
      </div>
    </div>
  );
}
```

### Layout with Multiple Slots (Parallel Routes)

Create layouts with multiple content areas:

```tsx
// src/app/dashboard/layout.tsx
export default function DashboardLayout({
  children,
  analytics,
  notifications,
}: {
  children: React.ReactNode;
  analytics: React.ReactNode;
  notifications: React.ReactNode;
}) {
  return (
    <div className="dashboard-grid">
      <header className="dashboard-header">
        <h1>Dashboard</h1>
        <div className="notifications-widget">
          {notifications}
        </div>
      </header>
      
      <aside className="dashboard-sidebar">
        <nav>Navigation</nav>
      </aside>
      
      <main className="dashboard-main">
        {children}
      </main>
      
      <aside className="dashboard-analytics">
        {analytics}
      </aside>
    </div>
  );
}
```

```tsx
// src/app/dashboard/@analytics/page.tsx
import { getAnalyticsData } from '@/lib/analytics';

export default async function AnalyticsSlot() {
  const data = await getAnalyticsData();
  
  return (
    <div className="analytics-panel">
      <h2>Analytics</h2>
      <div className="metrics-grid">
        <div className="metric">
          <h3>Total Sales</h3>
          <p>${data.totalSales}</p>
        </div>
        <div className="metric">
          <h3>New Customers</h3>
          <p>{data.newCustomers}</p>
        </div>
      </div>
    </div>
  );
}
```

## Component Architecture Patterns

### Compound Components

Build complex components with multiple related parts:

```tsx
// src/components/ui/Card.tsx
import React from 'react';
import { cn } from '@/lib/utils';

interface CardProps {
  children: React.ReactNode;
  className?: string;
}

interface CardHeaderProps {
  children: React.ReactNode;
  className?: string;
}

interface CardContentProps {
  children: React.ReactNode;
  className?: string;
}

interface CardFooterProps {
  children: React.ReactNode;
  className?: string;
}

// Main Card component
export function Card({ children, className }: CardProps) {
  return (
    <div className={cn('bg-white rounded-lg shadow-md', className)}>
      {children}
    </div>
  );
}

// Card subcomponents
function CardHeader({ children, className }: CardHeaderProps) {
  return (
    <div className={cn('px-6 py-4 border-b border-gray-200', className)}>
      {children}
    </div>
  );
}

function CardContent({ children, className }: CardContentProps) {
  return (
    <div className={cn('px-6 py-4', className)}>
      {children}
    </div>
  );
}

function CardFooter({ children, className }: CardFooterProps) {
  return (
    <div className={cn('px-6 py-4 border-t border-gray-200', className)}>
      {children}
    </div>
  );
}

// Attach subcomponents to main component
Card.Header = CardHeader;
Card.Content = CardContent;
Card.Footer = CardFooter;

// Usage example
export function ProductCard({ product }: { product: Product }) {
  return (
    <Card>
      <Card.Header>
        <h3 className="text-lg font-semibold">{product.name}</h3>
      </Card.Header>
      
      <Card.Content>
        <p className="text-gray-600">{product.description}</p>
        <p className="text-xl font-bold mt-2">${product.price}</p>
      </Card.Content>
      
      <Card.Footer>
        <AddToCartButton productId={product.id} />
      </Card.Footer>
    </Card>
  );
}
```

### Render Props Pattern

Pass rendering logic as props for maximum flexibility:

```tsx
// src/components/DataFetcher.tsx
import { useState, useEffect, ReactNode } from 'react';

interface DataFetcherProps<T> {
  url: string;
  children: (props: {
    data: T | null;
    loading: boolean;
    error: Error | null;
    refetch: () => void;
  }) => ReactNode;
}

export function DataFetcher<T>({ url, children }: DataFetcherProps<T>) {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);

  const fetchData = async () => {
    try {
      setLoading(true);
      setError(null);
      
      const response = await fetch(url);
      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }
      
      const result = await response.json();
      setData(result);
    } catch (err) {
      setError(err instanceof Error ? err : new Error('Unknown error'));
    } finally {
      setLoading(false);
    }
  };

  useEffect(() => {
    fetchData();
  }, [url]);

  return children({ data, loading, error, refetch: fetchData });
}

// Usage
export function UserProfile({ userId }: { userId: string }) {
  return (
    <DataFetcher<User> url={`/api/users/${userId}`}>
      {({ data: user, loading, error, refetch }) => {
        if (loading) return <div>Loading user...</div>;
        if (error) return <div>Error: {error.message}</div>;
        if (!user) return <div>User not found</div>;

        return (
          <div>
            <h1>{user.name}</h1>
            <p>{user.email}</p>
            <button onClick={refetch}>Refresh</button>
          </div>
        );
      }}
    </DataFetcher>
  );
}
```

### Higher-Order Components (HOCs)

Enhance components with additional functionality:

```tsx
// src/components/hoc/withAuth.tsx
import { auth } from '@/lib/auth';
import { redirect } from 'next/navigation';
import { ComponentType } from 'react';

interface WithAuthOptions {
  redirectTo?: string;
  roles?: string[];
}

export function withAuth<P extends object>(
  Component: ComponentType<P>,
  options: WithAuthOptions = {}
) {
  const { redirectTo = '/login', roles } = options;

  return async function AuthenticatedComponent(props: P) {
    const session = await auth();
    
    // Check if user is authenticated
    if (!session) {
      redirect(redirectTo);
    }
    
    // Check if user has required roles
    if (roles && !roles.some(role => session.user.roles.includes(role))) {
      redirect('/unauthorized');
    }

    return <Component {...props} />;
  };
}

// Usage
const ProtectedDashboard = withAuth(Dashboard, {
  roles: ['admin', 'user'],
});

const AdminPanel = withAuth(AdminPanelComponent, {
  roles: ['admin'],
  redirectTo: '/dashboard',
});
```

### Custom Hooks for Component Logic

Extract component logic into reusable hooks:

```tsx
// src/hooks/useLocalStorage.ts
'use client';

import { useState, useEffect } from 'react';

export function useLocalStorage<T>(
  key: string,
  initialValue: T
): [T, (value: T | ((prev: T) => T)) => void] {
  const [storedValue, setStoredValue] = useState<T>(initialValue);

  useEffect(() => {
    try {
      const item = window.localStorage.getItem(key);
      if (item) {
        setStoredValue(JSON.parse(item));
      }
    } catch (error) {
      console.warn(`Error reading localStorage key "${key}":`, error);
    }
  }, [key]);

  const setValue = (value: T | ((prev: T) => T)) => {
    try {
      const valueToStore = value instanceof Function ? value(storedValue) : value;
      setStoredValue(valueToStore);
      window.localStorage.setItem(key, JSON.stringify(valueToStore));
    } catch (error) {
      console.warn(`Error setting localStorage key "${key}":`, error);
    }
  };

  return [storedValue, setValue];
}

// Usage in component
'use client';

import { useLocalStorage } from '@/hooks/useLocalStorage';

export function Settings() {
  const [theme, setTheme] = useLocalStorage('theme', 'light');
  const [language, setLanguage] = useLocalStorage('language', 'en');

  return (
    <div>
      <select value={theme} onChange={(e) => setTheme(e.target.value)}>
        <option value="light">Light</option>
        <option value="dark">Dark</option>
      </select>
      
      <select value={language} onChange={(e) => setLanguage(e.target.value)}>
        <option value="en">English</option>
        <option value="es">Spanish</option>
      </select>
    </div>
  );
}
```

## Responsive Design Patterns

### Mobile-First Components

Design components that work well on all screen sizes:

```tsx
// src/components/ResponsiveNav.tsx
'use client';

import { useState } from 'react';
import Link from 'next/link';
import { Menu, X } from 'lucide-react';

export function ResponsiveNav() {
  const [isOpen, setIsOpen] = useState(false);

  const navItems = [
    { href: '/', label: 'Home' },
    { href: '/products', label: 'Products' },
    { href: '/about', label: 'About' },
    { href: '/contact', label: 'Contact' },
  ];

  return (
    <nav className="bg-white shadow-lg">
      <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
        <div className="flex justify-between h-16">
          {/* Logo */}
          <div className="flex items-center">
            <Link href="/" className="text-xl font-bold">
              My Store
            </Link>
          </div>

          {/* Desktop Navigation */}
          <div className="hidden md:flex items-center space-x-8">
            {navItems.map((item) => (
              <Link
                key={item.href}
                href={item.href}
                className="text-gray-700 hover:text-blue-600 transition-colors"
              >
                {item.label}
              </Link>
            ))}
          </div>

          {/* Mobile menu button */}
          <div className="md:hidden flex items-center">
            <button
              onClick={() => setIsOpen(!isOpen)}
              className="text-gray-700 hover:text-blue-600"
            >
              {isOpen ? <X size={24} /> : <Menu size={24} />}
            </button>
          </div>
        </div>

        {/* Mobile Navigation */}
        {isOpen && (
          <div className="md:hidden">
            <div className="px-2 pt-2 pb-3 space-y-1 sm:px-3">
              {navItems.map((item) => (
                <Link
                  key={item.href}
                  href={item.href}
                  className="block px-3 py-2 text-gray-700 hover:text-blue-600 hover:bg-gray-50 rounded-md"
                  onClick={() => setIsOpen(false)}
                >
                  {item.label}
                </Link>
              ))}
            </div>
          </div>
        )}
      </div>
    </nav>
  );
}
```

### Container and Grid Components

Create flexible layout components:

```tsx
// src/components/layout/Container.tsx
import { ReactNode } from 'react';
import { cn } from '@/lib/utils';

interface ContainerProps {
  children: ReactNode;
  size?: 'sm' | 'md' | 'lg' | 'xl' | 'full';
  className?: string;
}

export function Container({ children, size = 'lg', className }: ContainerProps) {
  const sizeClasses = {
    sm: 'max-w-3xl',
    md: 'max-w-5xl',
    lg: 'max-w-7xl',
    xl: 'max-w-screen-2xl',
    full: 'max-w-full',
  };

  return (
    <div className={cn('mx-auto px-4 sm:px-6 lg:px-8', sizeClasses[size], className)}>
      {children}
    </div>
  );
}

// src/components/layout/Grid.tsx
import { ReactNode } from 'react';
import { cn } from '@/lib/utils';

interface GridProps {
  children: ReactNode;
  cols?: 1 | 2 | 3 | 4 | 5 | 6;
  gap?: 2 | 4 | 6 | 8;
  className?: string;
}

export function Grid({ children, cols = 3, gap = 6, className }: GridProps) {
  const colsClasses = {
    1: 'grid-cols-1',
    2: 'grid-cols-1 md:grid-cols-2',
    3: 'grid-cols-1 md:grid-cols-2 lg:grid-cols-3',
    4: 'grid-cols-1 md:grid-cols-2 lg:grid-cols-4',
    5: 'grid-cols-1 md:grid-cols-3 lg:grid-cols-5',
    6: 'grid-cols-1 md:grid-cols-3 lg:grid-cols-6',
  };

  const gapClasses = {
    2: 'gap-2',
    4: 'gap-4',
    6: 'gap-6',
    8: 'gap-8',
  };

  return (
    <div className={cn('grid', colsClasses[cols], gapClasses[gap], className)}>
      {children}
    </div>
  );
}

// Usage
export function ProductGrid({ products }: { products: Product[] }) {
  return (
    <Container>
      <Grid cols={3} gap={6}>
        {products.map((product) => (
          <ProductCard key={product.id} product={product} />
        ))}
      </Grid>
    </Container>
  );
}
```

## Component Composition Patterns

### Provider Pattern

Create context providers for shared state:

```tsx
// src/components/providers/CartProvider.tsx
'use client';

import { createContext, useContext, useReducer, ReactNode } from 'react';

interface CartItem {
  id: string;
  name: string;
  price: number;
  quantity: number;
  image: string;
}

interface CartState {
  items: CartItem[];
  total: number;
  itemCount: number;
}

type CartAction = 
  | { type: 'ADD_ITEM'; item: Omit<CartItem, 'quantity'> }
  | { type: 'REMOVE_ITEM'; id: string }
  | { type: 'UPDATE_QUANTITY'; id: string; quantity: number }
  | { type: 'CLEAR_CART' };

const CartContext = createContext<{
  state: CartState;
  dispatch: React.Dispatch<CartAction>;
} | null>(null);

function cartReducer(state: CartState, action: CartAction): CartState {
  switch (action.type) {
    case 'ADD_ITEM': {
      const existingItem = state.items.find(item => item.id === action.item.id);
      
      if (existingItem) {
        const updatedItems = state.items.map(item =>
          item.id === action.item.id
            ? { ...item, quantity: item.quantity + 1 }
            : item
        );
        
        return {
          ...state,
          items: updatedItems,
          total: calculateTotal(updatedItems),
          itemCount: calculateItemCount(updatedItems),
        };
      }
      
      const newItems = [...state.items, { ...action.item, quantity: 1 }];
      return {
        ...state,
        items: newItems,
        total: calculateTotal(newItems),
        itemCount: calculateItemCount(newItems),
      };
    }
    
    case 'REMOVE_ITEM': {
      const filteredItems = state.items.filter(item => item.id !== action.id);
      return {
        ...state,
        items: filteredItems,
        total: calculateTotal(filteredItems),
        itemCount: calculateItemCount(filteredItems),
      };
    }
    
    case 'UPDATE_QUANTITY': {
      const updatedItems = state.items.map(item =>
        item.id === action.id
          ? { ...item, quantity: action.quantity }
          : item
      ).filter(item => item.quantity > 0);
      
      return {
        ...state,
        items: updatedItems,
        total: calculateTotal(updatedItems),
        itemCount: calculateItemCount(updatedItems),
      };
    }
    
    case 'CLEAR_CART':
      return {
        items: [],
        total: 0,
        itemCount: 0,
      };
    
    default:
      return state;
  }
}

function calculateTotal(items: CartItem[]): number {
  return items.reduce((sum, item) => sum + (item.price * item.quantity), 0);
}

function calculateItemCount(items: CartItem[]): number {
  return items.reduce((sum, item) => sum + item.quantity, 0);
}

export function CartProvider({ children }: { children: ReactNode }) {
  const [state, dispatch] = useReducer(cartReducer, {
    items: [],
    total: 0,
    itemCount: 0,
  });

  return (
    <CartContext.Provider value={{ state, dispatch }}>
      {children}
    </CartContext.Provider>
  );
}

export function useCart() {
  const context = useContext(CartContext);
  
  if (!context) {
    throw new Error('useCart must be used within a CartProvider');
  }
  
  const { state, dispatch } = context;
  
  const addItem = (item: Omit<CartItem, 'quantity'>) => {
    dispatch({ type: 'ADD_ITEM', item });
  };
  
  const removeItem = (id: string) => {
    dispatch({ type: 'REMOVE_ITEM', id });
  };
  
  const updateQuantity = (id: string, quantity: number) => {
    dispatch({ type: 'UPDATE_QUANTITY', id, quantity });
  };
  
  const clearCart = () => {
    dispatch({ type: 'CLEAR_CART' });
  };
  
  return {
    ...state,
    addItem,
    removeItem,
    updateQuantity,
    clearCart,
  };
}
```

### Error Boundary Components

Handle errors gracefully in your component tree:

```tsx
// src/components/ErrorBoundary.tsx
'use client';

import React, { Component, ReactNode } from 'react';

interface Props {
  children: ReactNode;
  fallback?: ReactNode;
  onError?: (error: Error, errorInfo: React.ErrorInfo) => void;
}

interface State {
  hasError: boolean;
  error?: Error;
}

export class ErrorBoundary extends Component<Props, State> {
  constructor(props: Props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error: Error): State {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, errorInfo: React.ErrorInfo) {
    console.error('ErrorBoundary caught an error:', error, errorInfo);
    this.props.onError?.(error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      if (this.props.fallback) {
        return this.props.fallback;
      }

      return (
        <div className="error-boundary p-6 text-center">
          <h2 className="text-xl font-semibold text-red-600 mb-4">
            Something went wrong
          </h2>
          <p className="text-gray-600 mb-4">
            We're sorry, but something unexpected happened.
          </p>
          <button
            onClick={() => this.setState({ hasError: false })}
            className="bg-blue-500 text-white px-4 py-2 rounded hover:bg-blue-600"
          >
            Try again
          </button>
        </div>
      );
    }

    return this.props.children;
  }
}

// Usage
export function App() {
  return (
    <ErrorBoundary
      fallback={<CustomErrorPage />}
      onError={(error, errorInfo) => {
        // Send error to logging service
        console.error('Application error:', error, errorInfo);
      }}
    >
      <MyApplication />
    </ErrorBoundary>
  );
}
```

## Performance Optimization

### Lazy Loading Components

Load components only when needed:

```tsx
// src/components/LazyComponents.tsx
import dynamic from 'next/dynamic';
import { ComponentProps } from 'react';

// Lazy load heavy components
const Chart = dynamic(() => import('./Chart'), {
  loading: () => <div>Loading chart...</div>,
  ssr: false, // Disable server-side rendering for this component
});

const VideoPlayer = dynamic(() => import('./VideoPlayer'), {
  loading: () => <div className="aspect-video bg-gray-200 animate-pulse" />,
});

// Lazy load with props
const DynamicModal = dynamic(() => import('./Modal'), {
  loading: () => <div>Loading modal...</div>,
});

// Usage
export function Dashboard() {
  const [showChart, setShowChart] = useState(false);

  return (
    <div>
      <h1>Dashboard</h1>
      
      <button onClick={() => setShowChart(true)}>
        Show Chart
      </button>
      
      {showChart && <Chart data={chartData} />}
      
      <VideoPlayer src="/video.mp4" />
    </div>
  );
}
```

### Memoization Patterns

Optimize re-renders with proper memoization:

```tsx
// src/components/OptimizedList.tsx
import React, { memo, useMemo, useCallback } from 'react';

interface Item {
  id: string;
  name: string;
  price: number;
  category: string;
}

interface OptimizedListProps {
  items: Item[];
  filter: string;
  onItemClick: (id: string) => void;
}

// Memoized list item component
const ListItem = memo<{
  item: Item;
  onClick: () => void;
}>(({ item, onClick }) => {
  return (
    <div
      className="p-4 border border-gray-200 rounded cursor-pointer hover:bg-gray-50"
      onClick={onClick}
    >
      <h3 className="font-semibold">{item.name}</h3>
      <p className="text-gray-600">${item.price}</p>
      <span className="text-sm text-blue-600">{item.category}</span>
    </div>
  );
});

ListItem.displayName = 'ListItem';

export const OptimizedList = memo<OptimizedListProps>(
  ({ items, filter, onItemClick }) => {
    // Memoize filtered items
    const filteredItems = useMemo(() => {
      return items.filter(item =>
        item.name.toLowerCase().includes(filter.toLowerCase()) ||
        item.category.toLowerCase().includes(filter.toLowerCase())
      );
    }, [items, filter]);

    // Memoize click handlers
    const handleItemClick = useCallback(
      (id: string) => {
        onItemClick(id);
      },
      [onItemClick]
    );

    return (
      <div className="grid gap-4">
        {filteredItems.map((item) => (
          <ListItem
            key={item.id}
            item={item}
            onClick={() => handleItemClick(item.id)}
          />
        ))}
      </div>
    );
  }
);

OptimizedList.displayName = 'OptimizedList';
```

## Testing Components

### Component Testing with Testing Library

```tsx
// src/components/__tests__/AddToCartButton.test.tsx
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import { AddToCartButton } from '../AddToCartButton';
import { CartProvider } from '../providers/CartProvider';

const renderWithProvider = (component: React.ReactElement) => {
  return render(
    <CartProvider>
      {component}
    </CartProvider>
  );
};

describe('AddToCartButton', () => {
  const mockProduct = {
    productId: '1',
    productName: 'Test Product',
    price: 29.99,
  };

  it('renders correctly', () => {
    renderWithProvider(<AddToCartButton {...mockProduct} />);
    
    expect(screen.getByText('Add to Cart')).toBeInTheDocument();
  });

  it('shows loading state when clicked', async () => {
    renderWithProvider(<AddToCartButton {...mockProduct} />);
    
    const button = screen.getByText('Add to Cart');
    fireEvent.click(button);

    expect(screen.getByText('Adding...')).toBeInTheDocument();
    
    await waitFor(() => {
      expect(screen.getByText('Add to Cart')).toBeInTheDocument();
    });
  });

  it('disables button when loading', () => {
    renderWithProvider(<AddToCartButton {...mockProduct} />);
    
    const button = screen.getByText('Add to Cart');
    fireEvent.click(button);

    expect(button).toBeDisabled();
  });
});
```

## Summary

Components and layouts are the foundation of great Next.js applications. Here's what we covered:

### ✅ Core Concepts:
- **Server vs Client Components** for optimal performance
- **Layout system** with nested and parallel routes
- **Component patterns** for reusable and maintainable code
- **Responsive design** for all screen sizes

### ✅ Advanced Patterns:
- **Compound components** for complex UI elements
- **Provider pattern** for shared state management
- **Error boundaries** for graceful error handling
- **HOCs and custom hooks** for code reuse

### ✅ Performance:
- **Lazy loading** for code splitting
- **Memoization** to prevent unnecessary re-renders
- **Optimization strategies** for large component trees

### ✅ Best Practices:
- **Component composition** over inheritance
- **Proper TypeScript** types and interfaces
- **Testing strategies** for reliable components
- **Accessibility** considerations

### Key Takeaways:
1. **Server Components by default** reduce client-side JavaScript
2. **Use Client Components sparingly** for interactivity only
3. **Nested layouts** provide flexible page structures
4. **Component composition** creates maintainable architectures
5. **Performance optimization** should be built-in, not added later

Your component and layout knowledge is now comprehensive! In the next guide, we'll explore styling approaches in Next.js.

## Practice Exercise

Build a product catalog with:

1. **Server components** for product listings
2. **Client components** for interactive features
3. **Nested layouts** for different sections
4. **Compound components** for product cards
5. **Provider pattern** for cart state
6. **Responsive design** for mobile and desktop

## Next Steps

Continue to [Styling in Next.js](./07-Styling-in-NextJS.md) to learn about the various styling approaches and how to create beautiful, maintainable styles in your Next.js applications.
