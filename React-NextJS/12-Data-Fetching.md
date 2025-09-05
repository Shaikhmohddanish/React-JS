# Data Fetching in Next.js

## Table of Contents
1. [Introduction to Data Fetching](#introduction)
2. [Client-Side Data Fetching](#client-side)
3. [SWR for Data Fetching](#swr)
4. [React Query (TanStack Query)](#react-query)
5. [Hybrid Data Fetching Strategies](#hybrid)
6. [Real-time Data with WebSockets](#realtime)
7. [Optimistic Updates](#optimistic)
8. [Caching Strategies](#caching)
9. [Error Handling and Retry Logic](#error-handling)
10. [Performance Optimization](#performance)
11. [Best Practices](#best-practices)

## Introduction to Data Fetching {#introduction}

Next.js provides multiple strategies for fetching data, each suited for different use cases. Understanding when and how to use each approach is crucial for building performant applications.

### Data Fetching Strategies Overview

```typescript
interface DataFetchingStrategy {
  name: string;
  timing: 'build' | 'request' | 'client';
  performance: 'fast' | 'medium' | 'slow';
  freshness: 'static' | 'fresh' | 'stale-while-revalidate';
  useCase: string[];
  seoFriendly: boolean;
}

const strategies: DataFetchingStrategy[] = [
  {
    name: 'Static Site Generation (SSG)',
    timing: 'build',
    performance: 'fast',
    freshness: 'static',
    useCase: ['Marketing pages', 'Documentation', 'Blogs'],
    seoFriendly: true,
  },
  {
    name: 'Server-Side Rendering (SSR)',
    timing: 'request',
    performance: 'medium',
    freshness: 'fresh',
    useCase: ['User dashboards', 'Personalized content'],
    seoFriendly: true,
  },
  {
    name: 'Incremental Static Regeneration (ISR)',
    timing: 'build',
    performance: 'fast',
    freshness: 'stale-while-revalidate',
    useCase: ['News sites', 'E-commerce', 'Content with periodic updates'],
    seoFriendly: true,
  },
  {
    name: 'Client-Side Rendering (CSR)',
    timing: 'client',
    performance: 'slow',
    freshness: 'fresh',
    useCase: ['Interactive dashboards', 'Real-time data', 'Private content'],
    seoFriendly: false,
  },
];
```

### Choosing the Right Strategy

```typescript
// Decision tree for data fetching strategy
function chooseDataFetchingStrategy(requirements: {
  seoRequired: boolean;
  dataFreshness: 'static' | 'periodic' | 'realtime';
  userSpecific: boolean;
  buildTimeKnown: boolean;
}): string {
  if (requirements.seoRequired) {
    if (requirements.dataFreshness === 'static' && requirements.buildTimeKnown) {
      return 'SSG (getStaticProps)';
    }
    if (requirements.dataFreshness === 'periodic') {
      return 'ISR (getStaticProps with revalidate)';
    }
    if (requirements.userSpecific || requirements.dataFreshness === 'realtime') {
      return 'SSR (getServerSideProps)';
    }
  }
  
  return 'CSR (Client-side with SWR/React Query)';
}
```

## Client-Side Data Fetching {#client-side}

### Basic Client-Side Fetching with useEffect

```typescript
// hooks/useApi.ts
import { useState, useEffect, useCallback } from 'react';

interface UseApiState<T> {
  data: T | null;
  loading: boolean;
  error: string | null;
}

interface UseApiOptions {
  immediate?: boolean;
  onSuccess?: (data: any) => void;
  onError?: (error: string) => void;
}

export function useApi<T>(
  url: string, 
  options: UseApiOptions = {}
): UseApiState<T> & { refetch: () => Promise<void> } {
  const [state, setState] = useState<UseApiState<T>>({
    data: null,
    loading: false,
    error: null,
  });

  const fetchData = useCallback(async () => {
    setState(prev => ({ ...prev, loading: true, error: null }));

    try {
      const response = await fetch(url);
      
      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }

      const data = await response.json();
      
      setState({
        data,
        loading: false,
        error: null,
      });

      if (options.onSuccess) {
        options.onSuccess(data);
      }

    } catch (error) {
      const errorMessage = error instanceof Error ? error.message : 'An error occurred';
      
      setState({
        data: null,
        loading: false,
        error: errorMessage,
      });

      if (options.onError) {
        options.onError(errorMessage);
      }
    }
  }, [url, options.onSuccess, options.onError]);

  useEffect(() => {
    if (options.immediate !== false) {
      fetchData();
    }
  }, [fetchData, options.immediate]);

  return {
    ...state,
    refetch: fetchData,
  };
}
```

### Advanced API Hook with Caching

```typescript
// hooks/useApiWithCache.ts
import { useState, useEffect, useCallback, useRef } from 'react';

interface CacheEntry<T> {
  data: T;
  timestamp: number;
  expiry: number;
}

class APICache {
  private cache = new Map<string, CacheEntry<any>>();
  private defaultTTL = 5 * 60 * 1000; // 5 minutes

  set<T>(key: string, data: T, ttl?: number): void {
    const now = Date.now();
    this.cache.set(key, {
      data,
      timestamp: now,
      expiry: now + (ttl || this.defaultTTL),
    });
  }

  get<T>(key: string): T | null {
    const entry = this.cache.get(key);
    
    if (!entry) {
      return null;
    }

    if (Date.now() > entry.expiry) {
      this.cache.delete(key);
      return null;
    }

    return entry.data;
  }

  invalidate(key: string): void {
    this.cache.delete(key);
  }

  clear(): void {
    this.cache.clear();
  }

  // Get stale data even if expired (for stale-while-revalidate)
  getStale<T>(key: string): T | null {
    const entry = this.cache.get(key);
    return entry ? entry.data : null;
  }

  isStale(key: string): boolean {
    const entry = this.cache.get(key);
    return entry ? Date.now() > entry.expiry : true;
  }
}

const apiCache = new APICache();

interface UseApiWithCacheOptions {
  ttl?: number; // Time to live in milliseconds
  staleWhileRevalidate?: boolean;
  immediate?: boolean;
  onSuccess?: (data: any) => void;
  onError?: (error: string) => void;
}

export function useApiWithCache<T>(
  url: string,
  options: UseApiWithCacheOptions = {}
) {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);
  const [isStale, setIsStale] = useState(false);
  const abortControllerRef = useRef<AbortController | null>(null);

  const fetchData = useCallback(async (bypassCache = false) => {
    // Cancel previous request
    if (abortControllerRef.current) {
      abortControllerRef.current.abort();
    }

    const cacheKey = url;
    
    // Check cache first
    if (!bypassCache) {
      const cachedData = apiCache.get<T>(cacheKey);
      if (cachedData) {
        setData(cachedData);
        setError(null);
        setIsStale(false);
        return;
      }

      // Use stale data while revalidating
      if (options.staleWhileRevalidate) {
        const staleData = apiCache.getStale<T>(cacheKey);
        if (staleData) {
          setData(staleData);
          setIsStale(true);
        }
      }
    }

    setLoading(true);
    setError(null);

    // Create new abort controller
    const abortController = new AbortController();
    abortControllerRef.current = abortController;

    try {
      const response = await fetch(url, {
        signal: abortController.signal,
      });

      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }

      const fetchedData = await response.json();

      // Cache the data
      apiCache.set(cacheKey, fetchedData, options.ttl);

      setData(fetchedData);
      setLoading(false);
      setError(null);
      setIsStale(false);

      if (options.onSuccess) {
        options.onSuccess(fetchedData);
      }

    } catch (error) {
      // Ignore abort errors
      if (error.name === 'AbortError') {
        return;
      }

      const errorMessage = error instanceof Error ? error.message : 'An error occurred';
      
      setLoading(false);
      setError(errorMessage);

      if (options.onError) {
        options.onError(errorMessage);
      }
    }
  }, [url, options.ttl, options.staleWhileRevalidate, options.onSuccess, options.onError]);

  const invalidateCache = useCallback(() => {
    apiCache.invalidate(url);
  }, [url]);

  const refetch = useCallback(() => {
    return fetchData(true);
  }, [fetchData]);

  useEffect(() => {
    if (options.immediate !== false) {
      fetchData();
    }

    // Cleanup on unmount
    return () => {
      if (abortControllerRef.current) {
        abortControllerRef.current.abort();
      }
    };
  }, [fetchData, options.immediate]);

  return {
    data,
    loading,
    error,
    isStale,
    refetch,
    invalidateCache,
  };
}
```

### User Dashboard with Client-Side Fetching

```typescript
// components/UserDashboard.tsx
import { useState, useEffect } from 'react';
import { useApiWithCache } from '../hooks/useApiWithCache';

interface UserStats {
  totalOrders: number;
  totalSpent: number;
  favoriteProducts: number;
  loyaltyPoints: number;
}

interface RecentOrder {
  id: string;
  date: string;
  total: number;
  status: string;
  items: Array<{
    name: string;
    quantity: number;
    price: number;
  }>;
}

interface Notification {
  id: string;
  message: string;
  type: 'info' | 'warning' | 'success';
  timestamp: string;
  read: boolean;
}

export default function UserDashboard() {
  const [user, setUser] = useState(null);

  // Fetch user stats with 10-minute cache
  const {
    data: stats,
    loading: statsLoading,
    error: statsError,
    isStale: statsStale,
  } = useApiWithCache<UserStats>('/api/user/stats', {
    ttl: 10 * 60 * 1000, // 10 minutes
    staleWhileRevalidate: true,
  });

  // Fetch recent orders with 5-minute cache
  const {
    data: orders,
    loading: ordersLoading,
    error: ordersError,
    refetch: refetchOrders,
  } = useApiWithCache<RecentOrder[]>('/api/user/orders?limit=5', {
    ttl: 5 * 60 * 1000, // 5 minutes
    staleWhileRevalidate: true,
  });

  // Fetch notifications with 1-minute cache (more frequent updates)
  const {
    data: notifications,
    loading: notificationsLoading,
    error: notificationsError,
    refetch: refetchNotifications,
  } = useApiWithCache<Notification[]>('/api/user/notifications', {
    ttl: 60 * 1000, // 1 minute
    staleWhileRevalidate: true,
  });

  // Get user info from localStorage or context
  useEffect(() => {
    const userInfo = localStorage.getItem('user');
    if (userInfo) {
      setUser(JSON.parse(userInfo));
    }
  }, []);

  const handleMarkNotificationRead = async (notificationId: string) => {
    try {
      await fetch(`/api/user/notifications/${notificationId}`, {
        method: 'PATCH',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ read: true }),
      });
      
      // Refresh notifications
      refetchNotifications();
    } catch (error) {
      console.error('Failed to mark notification as read:', error);
    }
  };

  if (!user) {
    return (
      <div className="flex items-center justify-center h-64">
        <div className="text-center">
          <div className="animate-spin rounded-full h-12 w-12 border-b-2 border-blue-600 mx-auto mb-4"></div>
          <p>Loading user information...</p>
        </div>
      </div>
    );
  }

  return (
    <div className="container mx-auto px-4 py-8">
      {/* Header */}
      <div className="mb-8">
        <h1 className="text-3xl font-bold text-gray-900 mb-2">
          Welcome back, {user.name}!
        </h1>
        <p className="text-gray-600">Here's what's happening with your account</p>
      </div>

      {/* Stats Grid */}
      <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6 mb-8">
        {statsLoading && !stats ? (
          // Loading skeleton
          Array.from({ length: 4 }).map((_, i) => (
            <div key={i} className="bg-white p-6 rounded-lg shadow-md animate-pulse">
              <div className="h-4 bg-gray-200 rounded w-3/4 mb-4"></div>
              <div className="h-8 bg-gray-200 rounded w-1/2"></div>
            </div>
          ))
        ) : statsError ? (
          <div className="col-span-full bg-red-50 border border-red-200 rounded-lg p-4 text-red-800">
            Failed to load stats: {statsError}
          </div>
        ) : stats ? (
          <>
            <div className="bg-white p-6 rounded-lg shadow-md">
              <div className="flex items-center justify-between">
                <div>
                  <h3 className="text-lg font-semibold text-gray-900">Total Orders</h3>
                  <p className="text-3xl font-bold text-blue-600">{stats.totalOrders}</p>
                </div>
                {statsStale && (
                  <div className="text-xs text-yellow-600 bg-yellow-100 px-2 py-1 rounded">
                    Updating...
                  </div>
                )}
              </div>
            </div>
            
            <div className="bg-white p-6 rounded-lg shadow-md">
              <h3 className="text-lg font-semibold text-gray-900">Total Spent</h3>
              <p className="text-3xl font-bold text-green-600">${stats.totalSpent.toFixed(2)}</p>
            </div>
            
            <div className="bg-white p-6 rounded-lg shadow-md">
              <h3 className="text-lg font-semibold text-gray-900">Favorites</h3>
              <p className="text-3xl font-bold text-purple-600">{stats.favoriteProducts}</p>
            </div>
            
            <div className="bg-white p-6 rounded-lg shadow-md">
              <h3 className="text-lg font-semibold text-gray-900">Loyalty Points</h3>
              <p className="text-3xl font-bold text-orange-600">{stats.loyaltyPoints}</p>
            </div>
          </>
        ) : null}
      </div>

      <div className="grid grid-cols-1 lg:grid-cols-2 gap-8">
        {/* Recent Orders */}
        <div className="bg-white rounded-lg shadow-md">
          <div className="p-6 border-b border-gray-200">
            <div className="flex items-center justify-between">
              <h2 className="text-xl font-semibold text-gray-900">Recent Orders</h2>
              <button
                onClick={refetchOrders}
                className="text-blue-600 hover:text-blue-800 text-sm"
              >
                Refresh
              </button>
            </div>
          </div>
          
          <div className="p-6">
            {ordersLoading && !orders ? (
              <div className="space-y-4">
                {Array.from({ length: 3 }).map((_, i) => (
                  <div key={i} className="animate-pulse">
                    <div className="h-4 bg-gray-200 rounded w-3/4 mb-2"></div>
                    <div className="h-3 bg-gray-200 rounded w-1/2"></div>
                  </div>
                ))}
              </div>
            ) : ordersError ? (
              <p className="text-red-600">Failed to load orders: {ordersError}</p>
            ) : orders && orders.length > 0 ? (
              <div className="space-y-4">
                {orders.map((order) => (
                  <div key={order.id} className="border-b border-gray-100 pb-4 last:border-b-0">
                    <div className="flex justify-between items-start mb-2">
                      <div>
                        <p className="font-semibold">Order #{order.id}</p>
                        <p className="text-sm text-gray-600">
                          {new Date(order.date).toLocaleDateString()}
                        </p>
                      </div>
                      <div className="text-right">
                        <p className="font-semibold">${order.total.toFixed(2)}</p>
                        <span className={`text-xs px-2 py-1 rounded-full ${
                          order.status === 'delivered' ? 'bg-green-100 text-green-800' :
                          order.status === 'shipped' ? 'bg-blue-100 text-blue-800' :
                          'bg-yellow-100 text-yellow-800'
                        }`}>
                          {order.status}
                        </span>
                      </div>
                    </div>
                    <div className="text-sm text-gray-600">
                      {order.items.slice(0, 2).map((item, index) => (
                        <span key={index}>
                          {item.quantity}x {item.name}
                          {index < Math.min(order.items.length - 1, 1) && ', '}
                        </span>
                      ))}
                      {order.items.length > 2 && ` and ${order.items.length - 2} more`}
                    </div>
                  </div>
                ))}
              </div>
            ) : (
              <p className="text-gray-500 text-center py-8">No orders found</p>
            )}
          </div>
        </div>

        {/* Notifications */}
        <div className="bg-white rounded-lg shadow-md">
          <div className="p-6 border-b border-gray-200">
            <div className="flex items-center justify-between">
              <h2 className="text-xl font-semibold text-gray-900">Notifications</h2>
              <button
                onClick={refetchNotifications}
                className="text-blue-600 hover:text-blue-800 text-sm"
              >
                Refresh
              </button>
            </div>
          </div>
          
          <div className="p-6">
            {notificationsLoading && !notifications ? (
              <div className="space-y-3">
                {Array.from({ length: 4 }).map((_, i) => (
                  <div key={i} className="animate-pulse">
                    <div className="h-4 bg-gray-200 rounded w-full mb-2"></div>
                    <div className="h-3 bg-gray-200 rounded w-1/3"></div>
                  </div>
                ))}
              </div>
            ) : notificationsError ? (
              <p className="text-red-600">Failed to load notifications: {notificationsError}</p>
            ) : notifications && notifications.length > 0 ? (
              <div className="space-y-3">
                {notifications.map((notification) => (
                  <div
                    key={notification.id}
                    className={`p-3 rounded-lg border ${
                      notification.read 
                        ? 'bg-gray-50 border-gray-200' 
                        : 'bg-blue-50 border-blue-200'
                    }`}
                  >
                    <div className="flex justify-between items-start">
                      <div className="flex-1">
                        <p className={`text-sm ${
                          notification.read ? 'text-gray-700' : 'text-gray-900 font-medium'
                        }`}>
                          {notification.message}
                        </p>
                        <p className="text-xs text-gray-500 mt-1">
                          {new Date(notification.timestamp).toLocaleString()}
                        </p>
                      </div>
                      {!notification.read && (
                        <button
                          onClick={() => handleMarkNotificationRead(notification.id)}
                          className="text-xs text-blue-600 hover:text-blue-800 ml-2"
                        >
                          Mark as read
                        </button>
                      )}
                    </div>
                  </div>
                ))}
              </div>
            ) : (
              <p className="text-gray-500 text-center py-8">No notifications</p>
            )}
          </div>
        </div>
      </div>
    </div>
  );
}
```

## SWR for Data Fetching {#swr}

SWR (Stale-While-Revalidate) is a popular data fetching library that provides features like caching, revalidation, and error handling out of the box.

### Setting up SWR

```bash
npm install swr
```

```typescript
// lib/fetcher.ts
export const fetcher = async (url: string) => {
  const response = await fetch(url);
  
  if (!response.ok) {
    const error = new Error('An error occurred while fetching the data.');
    // Attach extra info to the error object
    error.info = await response.json();
    error.status = response.status;
    throw error;
  }
  
  return response.json();
};

// Custom fetcher with authentication
export const authenticatedFetcher = async (url: string) => {
  const token = localStorage.getItem('authToken');
  
  const response = await fetch(url, {
    headers: {
      'Authorization': token ? `Bearer ${token}` : '',
      'Content-Type': 'application/json',
    },
  });
  
  if (!response.ok) {
    if (response.status === 401) {
      // Handle authentication error
      localStorage.removeItem('authToken');
      window.location.href = '/login';
      return;
    }
    
    const error = new Error('An error occurred while fetching the data.');
    error.info = await response.json();
    error.status = response.status;
    throw error;
  }
  
  return response.json();
};
```

### SWR Configuration

```typescript
// pages/_app.tsx
import { SWRConfig } from 'swr';
import { fetcher } from '../lib/fetcher';

export default function App({ Component, pageProps }) {
  return (
    <SWRConfig
      value={{
        fetcher,
        refreshInterval: 60000, // Refresh every minute
        refreshWhenHidden: false, // Don't refresh when tab is hidden
        refreshWhenOffline: false, // Don't refresh when offline
        revalidateOnFocus: true, // Revalidate when window gets focused
        revalidateOnReconnect: true, // Revalidate when reconnected
        dedupingInterval: 2000, // Dedupe requests within 2 seconds
        errorRetryCount: 3, // Retry failed requests 3 times
        errorRetryInterval: 5000, // Wait 5 seconds between retries
        onError: (error) => {
          console.error('SWR Error:', error);
          // You can send error to monitoring service here
        },
      }}
    >
      <Component {...pageProps} />
    </SWRConfig>
  );
}
```

### Advanced SWR Usage

```typescript
// hooks/useProducts.ts
import useSWR from 'swr';
import { authenticatedFetcher } from '../lib/fetcher';

interface Product {
  id: string;
  name: string;
  price: number;
  category: string;
  inStock: boolean;
  rating: number;
}

interface ProductsResponse {
  products: Product[];
  total: number;
  page: number;
  hasMore: boolean;
}

interface UseProductsOptions {
  category?: string;
  search?: string;
  page?: number;
  limit?: number;
}

export function useProducts(options: UseProductsOptions = {}) {
  const { category, search, page = 1, limit = 20 } = options;
  
  // Build query string
  const params = new URLSearchParams();
  if (category) params.append('category', category);
  if (search) params.append('search', search);
  params.append('page', page.toString());
  params.append('limit', limit.toString());
  
  const url = `/api/products?${params.toString()}`;
  
  const {
    data,
    error,
    isValidating,
    mutate,
  } = useSWR<ProductsResponse>(url, authenticatedFetcher, {
    // Revalidate every 5 minutes for product data
    refreshInterval: 5 * 60 * 1000,
    // Keep previous data while loading new data
    keepPreviousData: true,
    // Revalidate when user comes back to the tab
    revalidateOnFocus: true,
  });

  return {
    products: data?.products || [],
    total: data?.total || 0,
    hasMore: data?.hasMore || false,
    loading: !error && !data,
    error,
    isValidating,
    refetch: mutate,
  };
}

// Product search with debouncing
import { useState, useEffect, useMemo } from 'react';
import { useDebouncedCallback } from 'use-debounce';

export function useProductSearch() {
  const [searchTerm, setSearchTerm] = useState('');
  const [debouncedSearchTerm, setDebouncedSearchTerm] = useState('');
  
  // Debounce search term updates
  const updateDebouncedSearchTerm = useDebouncedCallback(
    (value: string) => {
      setDebouncedSearchTerm(value);
    },
    300 // 300ms delay
  );

  useEffect(() => {
    updateDebouncedSearchTerm(searchTerm);
  }, [searchTerm, updateDebouncedSearchTerm]);

  // Use products hook with debounced search term
  const {
    products,
    total,
    loading,
    error,
    refetch,
  } = useProducts({
    search: debouncedSearchTerm,
  });

  return {
    searchTerm,
    setSearchTerm,
    products,
    total,
    loading: loading || searchTerm !== debouncedSearchTerm, // Show loading during debounce
    error,
    refetch,
  };
}
```

### Real-time Comments with SWR

```typescript
// components/CommentsSection.tsx
import { useState } from 'react';
import useSWR from 'swr';
import { authenticatedFetcher } from '../lib/fetcher';

interface Comment {
  id: string;
  content: string;
  author: {
    id: string;
    name: string;
    avatar: string;
  };
  createdAt: string;
  updatedAt: string;
  replies?: Comment[];
}

interface CommentsProps {
  postId: string;
}

export default function CommentsSection({ postId }: CommentsProps) {
  const [newComment, setNewComment] = useState('');
  const [isSubmitting, setIsSubmitting] = useState(false);

  // Fetch comments with automatic revalidation
  const {
    data: comments,
    error,
    isValidating,
    mutate,
  } = useSWR<Comment[]>(
    `/api/posts/${postId}/comments`,
    authenticatedFetcher,
    {
      refreshInterval: 30000, // Refresh every 30 seconds for real-time feel
      revalidateOnFocus: true,
      revalidateOnReconnect: true,
    }
  );

  const handleSubmitComment = async (e: React.FormEvent) => {
    e.preventDefault();
    
    if (!newComment.trim() || isSubmitting) return;

    setIsSubmitting(true);

    try {
      // Optimistic update
      const optimisticComment: Comment = {
        id: `temp-${Date.now()}`,
        content: newComment.trim(),
        author: {
          id: 'current-user',
          name: 'You',
          avatar: '/default-avatar.png',
        },
        createdAt: new Date().toISOString(),
        updatedAt: new Date().toISOString(),
      };

      // Update local data immediately
      mutate(
        (currentComments) => currentComments ? [...currentComments, optimisticComment] : [optimisticComment],
        false // Don't revalidate immediately
      );

      // Submit to server
      const response = await fetch(`/api/posts/${postId}/comments`, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'Authorization': `Bearer ${localStorage.getItem('authToken')}`,
        },
        body: JSON.stringify({ content: newComment.trim() }),
      });

      if (!response.ok) {
        throw new Error('Failed to post comment');
      }

      // Clear form
      setNewComment('');
      
      // Revalidate to get the real comment from server
      mutate();

    } catch (error) {
      console.error('Failed to post comment:', error);
      
      // Revert optimistic update on error
      mutate();
      
      alert('Failed to post comment. Please try again.');
    } finally {
      setIsSubmitting(false);
    }
  };

  const handleDeleteComment = async (commentId: string) => {
    if (!confirm('Are you sure you want to delete this comment?')) return;

    try {
      // Optimistic update - remove comment immediately
      mutate(
        (currentComments) => currentComments?.filter(c => c.id !== commentId),
        false
      );

      const response = await fetch(`/api/posts/${postId}/comments/${commentId}`, {
        method: 'DELETE',
        headers: {
          'Authorization': `Bearer ${localStorage.getItem('authToken')}`,
        },
      });

      if (!response.ok) {
        throw new Error('Failed to delete comment');
      }

      // Revalidate to confirm deletion
      mutate();

    } catch (error) {
      console.error('Failed to delete comment:', error);
      
      // Revert optimistic update on error
      mutate();
      
      alert('Failed to delete comment. Please try again.');
    }
  };

  if (error) {
    return (
      <div className="bg-red-50 border border-red-200 rounded-lg p-4 text-red-800">
        Failed to load comments: {error.message}
      </div>
    );
  }

  return (
    <div className="space-y-6">
      <h3 className="text-xl font-semibold">
        Comments ({comments?.length || 0})
        {isValidating && (
          <span className="ml-2 text-sm text-gray-500">(updating...)</span>
        )}
      </h3>

      {/* Comment Form */}
      <form onSubmit={handleSubmitComment} className="space-y-4">
        <textarea
          value={newComment}
          onChange={(e) => setNewComment(e.target.value)}
          placeholder="Write a comment..."
          className="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent resize-none"
          rows={3}
          disabled={isSubmitting}
        />
        <button
          type="submit"
          disabled={!newComment.trim() || isSubmitting}
          className="px-4 py-2 bg-blue-600 text-white rounded-lg hover:bg-blue-700 disabled:opacity-50 disabled:cursor-not-allowed"
        >
          {isSubmitting ? 'Posting...' : 'Post Comment'}
        </button>
      </form>

      {/* Comments List */}
      <div className="space-y-4">
        {!comments && !error ? (
          // Loading skeleton
          Array.from({ length: 3 }).map((_, i) => (
            <div key={i} className="animate-pulse">
              <div className="flex space-x-3">
                <div className="w-10 h-10 bg-gray-200 rounded-full"></div>
                <div className="flex-1 space-y-2">
                  <div className="h-4 bg-gray-200 rounded w-1/4"></div>
                  <div className="h-4 bg-gray-200 rounded w-3/4"></div>
                  <div className="h-4 bg-gray-200 rounded w-1/2"></div>
                </div>
              </div>
            </div>
          ))
        ) : comments && comments.length > 0 ? (
          comments.map((comment) => (
            <div key={comment.id} className="flex space-x-3">
              <img
                src={comment.author.avatar || '/default-avatar.png'}
                alt={comment.author.name}
                className="w-10 h-10 rounded-full"
              />
              <div className="flex-1">
                <div className="bg-gray-50 rounded-lg p-3">
                  <div className="flex items-center justify-between mb-1">
                    <span className="font-medium text-gray-900">
                      {comment.author.name}
                    </span>
                    <div className="flex items-center space-x-2">
                      <time className="text-xs text-gray-500">
                        {new Date(comment.createdAt).toLocaleDateString()}
                      </time>
                      {comment.author.id === 'current-user' && (
                        <button
                          onClick={() => handleDeleteComment(comment.id)}
                          className="text-xs text-red-600 hover:text-red-800"
                        >
                          Delete
                        </button>
                      )}
                    </div>
                  </div>
                  <p className="text-gray-700">{comment.content}</p>
                </div>
              </div>
            </div>
          ))
        ) : (
          <p className="text-gray-500 text-center py-8">
            No comments yet. Be the first to comment!
          </p>
        )}
      </div>
    </div>
  );
}
```

## React Query (TanStack Query) {#react-query}

React Query is a powerful data fetching library that provides advanced features like infinite queries, optimistic updates, and sophisticated caching.

### Setting up React Query

```bash
npm install @tanstack/react-query @tanstack/react-query-devtools
```

```typescript
// pages/_app.tsx
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { ReactQueryDevtools } from '@tanstack/react-query-devtools';
import { useState } from 'react';

export default function App({ Component, pageProps }) {
  const [queryClient] = useState(() => new QueryClient({
    defaultOptions: {
      queries: {
        staleTime: 5 * 60 * 1000, // 5 minutes
        cacheTime: 10 * 60 * 1000, // 10 minutes
        retry: 3,
        retryDelay: (attemptIndex) => Math.min(1000 * 2 ** attemptIndex, 30000),
        refetchOnWindowFocus: false,
        refetchOnReconnect: true,
      },
      mutations: {
        retry: 1,
      },
    },
  }));

  return (
    <QueryClientProvider client={queryClient}>
      <Component {...pageProps} />
      <ReactQueryDevtools initialIsOpen={false} />
    </QueryClientProvider>
  );
}
```

### Advanced Infinite Scroll with React Query

```typescript
// hooks/useInfiniteProducts.ts
import { useInfiniteQuery } from '@tanstack/react-query';
import { authenticatedFetcher } from '../lib/fetcher';

interface Product {
  id: string;
  name: string;
  price: number;
  image: string;
  category: string;
}

interface ProductsPage {
  products: Product[];
  nextCursor?: string;
  hasMore: boolean;
}

interface UseInfiniteProductsOptions {
  category?: string;
  search?: string;
  sortBy?: string;
}

export function useInfiniteProducts(options: UseInfiniteProductsOptions = {}) {
  const { category, search, sortBy = 'name' } = options;

  const queryKey = ['products', 'infinite', { category, search, sortBy }];

  return useInfiniteQuery({
    queryKey,
    queryFn: async ({ pageParam = '' }) => {
      const params = new URLSearchParams();
      if (category) params.append('category', category);
      if (search) params.append('search', search);
      params.append('sortBy', sortBy);
      if (pageParam) params.append('cursor', pageParam);
      params.append('limit', '20');

      const url = `/api/products/infinite?${params.toString()}`;
      return authenticatedFetcher(url) as Promise<ProductsPage>;
    },
    getNextPageParam: (lastPage) => lastPage.nextCursor,
    staleTime: 5 * 60 * 1000, // 5 minutes
    cacheTime: 10 * 60 * 1000, // 10 minutes
  });
}
```

### Infinite Scroll Component

```typescript
// components/InfiniteProductList.tsx
import { useEffect, useRef } from 'react';
import { useInfiniteProducts } from '../hooks/useInfiniteProducts';

interface InfiniteProductListProps {
  category?: string;
  search?: string;
  sortBy?: string;
}

export default function InfiniteProductList({ 
  category, 
  search, 
  sortBy 
}: InfiniteProductListProps) {
  const {
    data,
    error,
    fetchNextPage,
    hasNextPage,
    isFetching,
    isFetchingNextPage,
    isLoading,
  } = useInfiniteProducts({ category, search, sortBy });

  const loadMoreRef = useRef<HTMLDivElement>(null);

  // Intersection Observer for infinite scroll
  useEffect(() => {
    if (!hasNextPage || isFetchingNextPage) return;

    const observer = new IntersectionObserver(
      (entries) => {
        if (entries[0].isIntersecting) {
          fetchNextPage();
        }
      },
      { threshold: 0.1 }
    );

    const currentRef = loadMoreRef.current;
    if (currentRef) {
      observer.observe(currentRef);
    }

    return () => {
      if (currentRef) {
        observer.unobserve(currentRef);
      }
    };
  }, [hasNextPage, isFetchingNextPage, fetchNextPage]);

  if (isLoading) {
    return (
      <div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 gap-6">
        {Array.from({ length: 8 }).map((_, i) => (
          <div key={i} className="bg-white rounded-lg shadow-md overflow-hidden animate-pulse">
            <div className="h-48 bg-gray-200"></div>
            <div className="p-4 space-y-3">
              <div className="h-4 bg-gray-200 rounded w-3/4"></div>
              <div className="h-4 bg-gray-200 rounded w-1/2"></div>
              <div className="h-6 bg-gray-200 rounded w-1/3"></div>
            </div>
          </div>
        ))}
      </div>
    );
  }

  if (error) {
    return (
      <div className="text-center py-12">
        <div className="bg-red-50 border border-red-200 rounded-lg p-6 inline-block">
          <h3 className="text-lg font-medium text-red-800 mb-2">
            Failed to load products
          </h3>
          <p className="text-red-600 mb-4">
            {error.message || 'Something went wrong'}
          </p>
          <button
            onClick={() => window.location.reload()}
            className="px-4 py-2 bg-red-600 text-white rounded-lg hover:bg-red-700"
          >
            Try Again
          </button>
        </div>
      </div>
    );
  }

  const allProducts = data?.pages.flatMap(page => page.products) || [];

  if (allProducts.length === 0) {
    return (
      <div className="text-center py-12">
        <div className="text-gray-500">
          <h3 className="text-lg font-medium mb-2">No products found</h3>
          <p>Try adjusting your search criteria</p>
        </div>
      </div>
    );
  }

  return (
    <div>
      {/* Products Grid */}
      <div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 gap-6">
        {allProducts.map((product) => (
          <div key={product.id} className="bg-white rounded-lg shadow-md overflow-hidden hover:shadow-lg transition-shadow">
            <img
              src={product.image}
              alt={product.name}
              className="w-full h-48 object-cover"
            />
            <div className="p-4">
              <h3 className="font-semibold text-lg mb-2 line-clamp-2">
                {product.name}
              </h3>
              <p className="text-gray-600 text-sm mb-2">{product.category}</p>
              <div className="flex justify-between items-center">
                <span className="text-lg font-bold text-green-600">
                  ${product.price.toFixed(2)}
                </span>
                <button className="px-3 py-1 bg-blue-600 text-white text-sm rounded hover:bg-blue-700">
                  Add to Cart
                </button>
              </div>
            </div>
          </div>
        ))}
      </div>

      {/* Load More Trigger */}
      <div ref={loadMoreRef} className="py-8">
        {isFetchingNextPage && (
          <div className="text-center">
            <div className="inline-flex items-center space-x-2">
              <div className="animate-spin rounded-full h-6 w-6 border-b-2 border-blue-600"></div>
              <span className="text-gray-600">Loading more products...</span>
            </div>
          </div>
        )}
        
        {!hasNextPage && allProducts.length > 0 && (
          <div className="text-center text-gray-500">
            <p>You've reached the end of the list!</p>
          </div>
        )}
      </div>

      {/* Global Loading Indicator */}
      {isFetching && !isFetchingNextPage && (
        <div className="fixed top-4 right-4 bg-blue-600 text-white px-3 py-2 rounded-lg shadow-lg">
          <div className="flex items-center space-x-2">
            <div className="animate-spin rounded-full h-4 w-4 border-b-2 border-white"></div>
            <span className="text-sm">Updating...</span>
          </div>
        </div>
      )}
    </div>
  );
}
```

## Best Practices {#best-practices}

### 1. Data Fetching Strategy Decision Matrix

```typescript
// utils/dataFetchingStrategy.ts
interface PageRequirements {
  seoRequired: boolean;
  realTimeData: boolean;
  userSpecific: boolean;
  highTraffic: boolean;
  dataChangesFrequently: boolean;
  buildTimeDataAvailable: boolean;
}

export function recommendDataFetchingStrategy(requirements: PageRequirements): string[] {
  const strategies: string[] = [];

  // Primary strategy
  if (requirements.seoRequired) {
    if (requirements.buildTimeDataAvailable && !requirements.realTimeData) {
      if (requirements.dataChangesFrequently) {
        strategies.push('ISR (Incremental Static Regeneration)');
      } else {
        strategies.push('SSG (Static Site Generation)');
      }
    } else if (requirements.userSpecific || requirements.realTimeData) {
      strategies.push('SSR (Server-Side Rendering)');
    }
  } else {
    strategies.push('CSR (Client-Side Rendering)');
  }

  // Secondary strategy (for client-side enhancements)
  if (!requirements.realTimeData) {
    strategies.push('SWR for background updates');
  } else {
    strategies.push('React Query with short stale time');
    strategies.push('WebSocket for real-time updates');
  }

  return strategies;
}
```

### 2. Optimized Data Fetching Patterns

```typescript
// hooks/useOptimizedData.ts
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { useCallback } from 'react';

interface OptimizedDataOptions<T> {
  queryKey: string[];
  fetcher: () => Promise<T>;
  staleTime?: number;
  cacheTime?: number;
  optimisticUpdates?: boolean;
}

export function useOptimizedData<T>({
  queryKey,
  fetcher,
  staleTime = 5 * 60 * 1000,
  cacheTime = 10 * 60 * 1000,
  optimisticUpdates = false,
}: OptimizedDataOptions<T>) {
  const queryClient = useQueryClient();

  // Main query
  const query = useQuery({
    queryKey,
    queryFn: fetcher,
    staleTime,
    cacheTime,
    // Enable background refetch
    refetchOnWindowFocus: true,
    refetchOnReconnect: true,
    // Retry configuration
    retry: (failureCount, error: any) => {
      // Don't retry on 4xx errors
      if (error?.status >= 400 && error?.status < 500) {
        return false;
      }
      return failureCount < 3;
    },
  });

  // Optimistic update helper
  const updateOptimistically = useCallback(
    (updater: (oldData: T | undefined) => T) => {
      if (!optimisticUpdates) return;

      queryClient.setQueryData(queryKey, updater);
    },
    [queryClient, queryKey, optimisticUpdates]
  );

  // Invalidate and refetch
  const invalidate = useCallback(() => {
    queryClient.invalidateQueries({ queryKey });
  }, [queryClient, queryKey]);

  // Prefetch related data
  const prefetch = useCallback(
    (relatedQueryKey: string[], relatedFetcher: () => Promise<any>) => {
      queryClient.prefetchQuery({
        queryKey: relatedQueryKey,
        queryFn: relatedFetcher,
        staleTime: staleTime / 2, // Shorter stale time for prefetched data
      });
    },
    [queryClient, staleTime]
  );

  return {
    ...query,
    updateOptimistically,
    invalidate,
    prefetch,
  };
}
```

### 3. Error Boundary for Data Fetching

```typescript
// components/DataErrorBoundary.tsx
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

export class DataErrorBoundary extends Component<Props, State> {
  constructor(props: Props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error: Error): State {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, errorInfo: React.ErrorInfo) {
    console.error('Data fetching error:', error, errorInfo);
    
    if (this.props.onError) {
      this.props.onError(error, errorInfo);
    }

    // Send to error monitoring service
    // errorMonitoringService.captureException(error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      if (this.props.fallback) {
        return this.props.fallback;
      }

      return (
        <div className="bg-red-50 border border-red-200 rounded-lg p-6 text-center">
          <h2 className="text-lg font-semibold text-red-800 mb-2">
            Something went wrong
          </h2>
          <p className="text-red-600 mb-4">
            We encountered an error while loading the data.
          </p>
          <button
            onClick={() => window.location.reload()}
            className="px-4 py-2 bg-red-600 text-white rounded-lg hover:bg-red-700"
          >
            Reload Page
          </button>
        </div>
      );
    }

    return this.props.children;
  }
}

// Usage wrapper component
export function withDataErrorBoundary<P extends object>(
  Component: React.ComponentType<P>
) {
  return function WrappedComponent(props: P) {
    return (
      <DataErrorBoundary>
        <Component {...props} />
      </DataErrorBoundary>
    );
  };
}
```

## Exercises

### Exercise 1: Advanced Dashboard
Create a comprehensive dashboard with:
- Multiple data sources with different refresh intervals
- Real-time notifications using SWR
- Infinite scroll for activity feed
- Optimistic updates for user actions
- Advanced caching strategies

### Exercise 2: E-commerce Product Browser
Build a product browsing experience with:
- Infinite scroll product grid using React Query
- Advanced filtering and search
- Shopping cart with optimistic updates
- Product recommendations
- Performance optimization for large datasets

### Exercise 3: Social Media Feed
Create a social media feed with:
- Infinite scroll posts
- Real-time updates for likes and comments
- Optimistic UI for user interactions
- Image lazy loading and caching
- Background sync for offline support

### Exercise 4: Real-time Analytics Dashboard
Build an analytics dashboard with:
- Multiple chart components with different data sources
- Real-time data updates using WebSockets
- Data export functionality
- Advanced filtering and date range selection
- Performance monitoring and error handling

## Summary

Data fetching in Next.js provides multiple strategies:

- **Server-side methods**: SSG, SSR, ISR for SEO and performance
- **Client-side libraries**: SWR and React Query for dynamic UIs
- **Hybrid approaches**: Combining multiple strategies for optimal UX
- **Performance optimization**: Caching, background updates, optimistic UI

Key principles:
- Choose the right strategy based on requirements
- Implement proper error handling and loading states
- Use optimistic updates for better UX
- Cache data appropriately to reduce server load
- Provide fallbacks and retry mechanisms

Next, we'll explore **Image Optimization** and learn how to deliver performant images in Next.js applications!
