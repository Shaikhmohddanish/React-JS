# API Calls and Data Fetching - A Complete Guide

## Introduction to API Calls in React

Modern React applications rarely exist in isolation. They need to communicate with external services, fetch data from APIs, submit forms to servers, and handle real-time updates. This guide covers all the essential techniques for making API calls and managing data in React applications, from basic fetch requests to advanced patterns with custom hooks and state management.

## Why Data Fetching Matters in React

Understanding data fetching is crucial for React developers because:

1. **Dynamic Content**: Most applications need to display data that changes over time
2. **User Interactions**: Forms, searches, and user actions require server communication
3. **Real-time Updates**: Many applications need to sync with live data
4. **Performance**: Efficient data fetching improves user experience
5. **Error Handling**: Robust applications gracefully handle network failures
6. **User Experience**: Loading states and optimistic updates enhance perceived performance
7. **Scalability**: Proper data management patterns scale with application complexity

## Basic Data Fetching with Fetch API

### Simple GET Request

```jsx
import React, { useState, useEffect } from 'react';

function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const fetchUser = async () => {
      try {
        setLoading(true);
        setError(null);
        
        const response = await fetch(`https://api.example.com/users/${userId}`);
        
        if (!response.ok) {
          throw new Error(`Error: ${response.status} ${response.statusText}`);
        }
        
        const userData = await response.json();
        setUser(userData);
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    };

    if (userId) {
      fetchUser();
    }
  }, [userId]);

  if (loading) return <div className="loading">Loading user...</div>;
  if (error) return <div className="error">Error: {error}</div>;
  if (!user) return <div className="no-data">User not found</div>;

  return (
    <div className="user-profile">
      <img src={user.avatar} alt={user.name} />
      <h2>{user.name}</h2>
      <p>{user.email}</p>
      <p>Joined: {new Date(user.createdAt).toLocaleDateString()}</p>
    </div>
  );
}

export default UserProfile;
```

### POST Request with Form Data

```jsx
import React, { useState } from 'react';

function CreateUser() {
  const [formData, setFormData] = useState({
    name: '',
    email: '',
    phone: ''
  });
  const [loading, setLoading] = useState(false);
  const [success, setSuccess] = useState(false);
  const [error, setError] = useState(null);

  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormData(prev => ({
      ...prev,
      [name]: value
    }));
  };

  const handleSubmit = async (e) => {
    e.preventDefault();
    
    try {
      setLoading(true);
      setError(null);
      setSuccess(false);

      const response = await fetch('https://api.example.com/users', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify(formData)
      });

      if (!response.ok) {
        const errorData = await response.json();
        throw new Error(errorData.message || `Error: ${response.status}`);
      }

      const newUser = await response.json();
      setSuccess(true);
      setFormData({ name: '', email: '', phone: '' }); // Reset form
      
      console.log('User created:', newUser);
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  };

  return (
    <form onSubmit={handleSubmit} className="create-user-form">
      <h2>Create New User</h2>
      
      {error && <div className="error-message">{error}</div>}
      {success && <div className="success-message">User created successfully!</div>}

      <div className="form-group">
        <label htmlFor="name">Name:</label>
        <input
          type="text"
          id="name"
          name="name"
          value={formData.name}
          onChange={handleChange}
          required
          disabled={loading}
        />
      </div>

      <div className="form-group">
        <label htmlFor="email">Email:</label>
        <input
          type="email"
          id="email"
          name="email"
          value={formData.email}
          onChange={handleChange}
          required
          disabled={loading}
        />
      </div>

      <div className="form-group">
        <label htmlFor="phone">Phone:</label>
        <input
          type="tel"
          id="phone"
          name="phone"
          value={formData.phone}
          onChange={handleChange}
          disabled={loading}
        />
      </div>

      <button type="submit" disabled={loading} className="submit-button">
        {loading ? 'Creating...' : 'Create User'}
      </button>
    </form>
  );
}

export default CreateUser;
```

## Custom Hooks for Data Fetching

### Basic useFetch Hook

```jsx
import { useState, useEffect } from 'react';

function useFetch(url, options = {}) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const fetchData = async () => {
      try {
        setLoading(true);
        setError(null);

        const response = await fetch(url, {
          headers: {
            'Content-Type': 'application/json',
            ...options.headers
          },
          ...options
        });

        if (!response.ok) {
          throw new Error(`Error: ${response.status} ${response.statusText}`);
        }

        const result = await response.json();
        setData(result);
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    };

    if (url) {
      fetchData();
    }
  }, [url, JSON.stringify(options)]);

  return { data, loading, error };
}

// Usage
function UserList() {
  const { data: users, loading, error } = useFetch('https://api.example.com/users');

  if (loading) return <div>Loading users...</div>;
  if (error) return <div>Error: {error}</div>;

  return (
    <ul>
      {users?.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```

### Advanced useApi Hook

```jsx
import { useState, useEffect, useCallback, useRef } from 'react';

function useApi(url, options = {}) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  const controllerRef = useRef(null);

  // Function to make the API call
  const execute = useCallback(async (customUrl = url, customOptions = {}) => {
    try {
      // Cancel previous request
      if (controllerRef.current) {
        controllerRef.current.abort();
      }

      // Create new AbortController
      controllerRef.current = new AbortController();

      setLoading(true);
      setError(null);

      const response = await fetch(customUrl, {
        signal: controllerRef.current.signal,
        headers: {
          'Content-Type': 'application/json',
          ...options.headers,
          ...customOptions.headers
        },
        ...options,
        ...customOptions
      });

      if (!response.ok) {
        const errorData = await response.text();
        throw new Error(errorData || `Error: ${response.status}`);
      }

      const result = await response.json();
      setData(result);
      return result;
    } catch (err) {
      if (err.name === 'AbortError') {
        console.log('Request was cancelled');
        return null;
      }
      setError(err.message);
      throw err;
    } finally {
      setLoading(false);
    }
  }, [url, JSON.stringify(options)]);

  // Auto-execute on mount if immediate option is true
  useEffect(() => {
    if (options.immediate !== false && url) {
      execute();
    }

    // Cleanup function to cancel request
    return () => {
      if (controllerRef.current) {
        controllerRef.current.abort();
      }
    };
  }, [execute, options.immediate, url]);

  // Reset function
  const reset = useCallback(() => {
    setData(null);
    setError(null);
    setLoading(false);
  }, []);

  return {
    data,
    loading,
    error,
    execute,
    reset
  };
}

// Usage
function ProductDetail({ productId }) {
  const {
    data: product,
    loading,
    error,
    execute: fetchProduct
  } = useApi(`https://api.example.com/products/${productId}`, {
    immediate: false // Don't auto-fetch
  });

  const {
    loading: updating,
    execute: updateProduct
  } = useApi(null, { immediate: false });

  useEffect(() => {
    if (productId) {
      fetchProduct();
    }
  }, [productId, fetchProduct]);

  const handleUpdate = async (updatedData) => {
    try {
      await updateProduct(`https://api.example.com/products/${productId}`, {
        method: 'PUT',
        body: JSON.stringify(updatedData)
      });
      
      // Refetch the product data
      await fetchProduct();
    } catch (err) {
      console.error('Failed to update product:', err);
    }
  };

  if (loading) return <div>Loading product...</div>;
  if (error) return <div>Error: {error}</div>;
  if (!product) return <div>Product not found</div>;

  return (
    <div>
      <h1>{product.name}</h1>
      <p>{product.description}</p>
      <button 
        onClick={() => handleUpdate({ ...product, name: 'Updated Name' })}
        disabled={updating}
      >
        {updating ? 'Updating...' : 'Update Product'}
      </button>
    </div>
  );
}
```

## Using Axios for HTTP Requests

Axios is a popular HTTP client library that provides additional features over the fetch API.

### Installation and Setup

```bash
npm install axios
```

### Basic Axios Usage

```jsx
import React, { useState, useEffect } from 'react';
import axios from 'axios';

// Create an axios instance with base configuration
const api = axios.create({
  baseURL: 'https://api.example.com',
  timeout: 10000,
  headers: {
    'Content-Type': 'application/json',
  },
});

// Add request interceptor for authentication
api.interceptors.request.use(
  (config) => {
    const token = localStorage.getItem('authToken');
    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }
    return config;
  },
  (error) => {
    return Promise.reject(error);
  }
);

// Add response interceptor for error handling
api.interceptors.response.use(
  (response) => response,
  (error) => {
    if (error.response?.status === 401) {
      // Handle unauthorized access
      localStorage.removeItem('authToken');
      window.location.href = '/login';
    }
    return Promise.reject(error);
  }
);

function UserManagement() {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  // Fetch users
  useEffect(() => {
    const fetchUsers = async () => {
      try {
        setLoading(true);
        const response = await api.get('/users');
        setUsers(response.data);
      } catch (err) {
        setError(err.response?.data?.message || err.message);
      } finally {
        setLoading(false);
      }
    };

    fetchUsers();
  }, []);

  // Create user
  const createUser = async (userData) => {
    try {
      const response = await api.post('/users', userData);
      setUsers(prev => [...prev, response.data]);
      return response.data;
    } catch (err) {
      throw new Error(err.response?.data?.message || err.message);
    }
  };

  // Update user
  const updateUser = async (userId, userData) => {
    try {
      const response = await api.put(`/users/${userId}`, userData);
      setUsers(prev => prev.map(user => 
        user.id === userId ? response.data : user
      ));
      return response.data;
    } catch (err) {
      throw new Error(err.response?.data?.message || err.message);
    }
  };

  // Delete user
  const deleteUser = async (userId) => {
    try {
      await api.delete(`/users/${userId}`);
      setUsers(prev => prev.filter(user => user.id !== userId));
    } catch (err) {
      throw new Error(err.response?.data?.message || err.message);
    }
  };

  if (loading) return <div>Loading users...</div>;
  if (error) return <div>Error: {error}</div>;

  return (
    <div>
      <h1>User Management</h1>
      {users.map(user => (
        <UserCard
          key={user.id}
          user={user}
          onUpdate={updateUser}
          onDelete={deleteUser}
        />
      ))}
    </div>
  );
}
```

### Custom Axios Hook

```jsx
import { useState, useEffect } from 'react';
import axios from 'axios';

function useAxios(url, options = {}) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    let source = axios.CancelToken.source();

    const fetchData = async () => {
      try {
        setLoading(true);
        setError(null);

        const response = await axios({
          url,
          cancelToken: source.token,
          ...options
        });

        setData(response.data);
      } catch (err) {
        if (!axios.isCancel(err)) {
          setError(err.response?.data?.message || err.message);
        }
      } finally {
        setLoading(false);
      }
    };

    if (url) {
      fetchData();
    }

    return () => {
      source.cancel('Request cancelled');
    };
  }, [url, JSON.stringify(options)]);

  return { data, loading, error };
}

// Usage
function Posts() {
  const { data: posts, loading, error } = useAxios('https://api.example.com/posts');

  if (loading) return <div>Loading posts...</div>;
  if (error) return <div>Error: {error}</div>;

  return (
    <div>
      {posts?.map(post => (
        <article key={post.id}>
          <h2>{post.title}</h2>
          <p>{post.content}</p>
        </article>
      ))}
    </div>
  );
}
```

## Advanced Data Fetching Patterns

### Pagination

```jsx
import React, { useState, useEffect, useCallback } from 'react';

function usePagination(url, initialPage = 1, pageSize = 10) {
  const [data, setData] = useState([]);
  const [totalCount, setTotalCount] = useState(0);
  const [currentPage, setCurrentPage] = useState(initialPage);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  const fetchPage = useCallback(async (page) => {
    try {
      setLoading(true);
      setError(null);

      const response = await fetch(
        `${url}?page=${page}&limit=${pageSize}`
      );

      if (!response.ok) {
        throw new Error(`Error: ${response.status}`);
      }

      const result = await response.json();
      setData(result.data);
      setTotalCount(result.total);
      setCurrentPage(page);
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  }, [url, pageSize]);

  useEffect(() => {
    fetchPage(currentPage);
  }, [fetchPage, currentPage]);

  const goToPage = (page) => {
    if (page >= 1 && page <= Math.ceil(totalCount / pageSize)) {
      setCurrentPage(page);
    }
  };

  const nextPage = () => goToPage(currentPage + 1);
  const prevPage = () => goToPage(currentPage - 1);

  const totalPages = Math.ceil(totalCount / pageSize);
  const hasNextPage = currentPage < totalPages;
  const hasPrevPage = currentPage > 1;

  return {
    data,
    loading,
    error,
    currentPage,
    totalPages,
    totalCount,
    hasNextPage,
    hasPrevPage,
    goToPage,
    nextPage,
    prevPage,
    refetch: () => fetchPage(currentPage)
  };
}

function ProductList() {
  const {
    data: products,
    loading,
    error,
    currentPage,
    totalPages,
    hasNextPage,
    hasPrevPage,
    nextPage,
    prevPage,
    goToPage
  } = usePagination('https://api.example.com/products', 1, 12);

  if (loading && products.length === 0) {
    return <div>Loading products...</div>;
  }

  if (error) {
    return <div>Error: {error}</div>;
  }

  return (
    <div>
      <div className="product-grid">
        {products.map(product => (
          <div key={product.id} className="product-card">
            <img src={product.image} alt={product.name} />
            <h3>{product.name}</h3>
            <p>${product.price}</p>
          </div>
        ))}
      </div>

      <div className="pagination">
        <button onClick={prevPage} disabled={!hasPrevPage}>
          Previous
        </button>
        
        {Array.from({ length: totalPages }, (_, i) => i + 1).map(page => (
          <button
            key={page}
            onClick={() => goToPage(page)}
            className={page === currentPage ? 'active' : ''}
          >
            {page}
          </button>
        ))}
        
        <button onClick={nextPage} disabled={!hasNextPage}>
          Next
        </button>
      </div>
      
      {loading && <div className="loading-overlay">Loading...</div>}
    </div>
  );
}
```

### Infinite Scrolling

```jsx
import React, { useState, useEffect, useCallback, useRef } from 'react';

function useInfiniteScroll(url, pageSize = 20) {
  const [data, setData] = useState([]);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  const [hasMore, setHasMore] = useState(true);
  const [page, setPage] = useState(1);

  const fetchMoreData = useCallback(async () => {
    if (loading || !hasMore) return;

    try {
      setLoading(true);
      setError(null);

      const response = await fetch(
        `${url}?page=${page}&limit=${pageSize}`
      );

      if (!response.ok) {
        throw new Error(`Error: ${response.status}`);
      }

      const result = await response.json();
      
      if (result.data.length === 0) {
        setHasMore(false);
      } else {
        setData(prev => [...prev, ...result.data]);
        setPage(prev => prev + 1);
        setHasMore(result.data.length === pageSize);
      }
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  }, [url, page, pageSize, loading, hasMore]);

  // Load initial data
  useEffect(() => {
    fetchMoreData();
  }, []); // Empty dependency array for initial load only

  const reset = () => {
    setData([]);
    setPage(1);
    setHasMore(true);
    setError(null);
  };

  return {
    data,
    loading,
    error,
    hasMore,
    fetchMoreData,
    reset
  };
}

function useIntersectionObserver(callback, options = {}) {
  const targetRef = useRef(null);

  useEffect(() => {
    const target = targetRef.current;
    if (!target) return;

    const observer = new IntersectionObserver(([entry]) => {
      if (entry.isIntersecting) {
        callback();
      }
    }, options);

    observer.observe(target);

    return () => {
      observer.unobserve(target);
    };
  }, [callback, options]);

  return targetRef;
}

function InfinitePostList() {
  const {
    data: posts,
    loading,
    error,
    hasMore,
    fetchMoreData
  } = useInfiniteScroll('https://api.example.com/posts');

  const loadMoreRef = useIntersectionObserver(fetchMoreData, {
    threshold: 0.1
  });

  if (error) {
    return <div>Error: {error}</div>;
  }

  return (
    <div className="infinite-list">
      {posts.map((post, index) => (
        <article key={`${post.id}-${index}`} className="post-card">
          <h2>{post.title}</h2>
          <p>{post.content}</p>
          <div className="post-meta">
            <span>By {post.author}</span>
            <span>{new Date(post.createdAt).toLocaleDateString()}</span>
          </div>
        </article>
      ))}

      {hasMore && (
        <div ref={loadMoreRef} className="load-more-trigger">
          {loading && <div>Loading more posts...</div>}
        </div>
      )}

      {!hasMore && posts.length > 0 && (
        <div className="end-message">
          You've reached the end of the posts!
        </div>
      )}
    </div>
  );
}
```

### Search and Debouncing

```jsx
import React, { useState, useEffect, useMemo } from 'react';

function useDebounce(value, delay) {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const handler = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);

    return () => {
      clearTimeout(handler);
    };
  }, [value, delay]);

  return debouncedValue;
}

function useSearch(searchFunction, dependencies = []) {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  const debouncedQuery = useDebounce(query, 300);

  useEffect(() => {
    if (debouncedQuery.trim() === '') {
      setResults([]);
      return;
    }

    const performSearch = async () => {
      try {
        setLoading(true);
        setError(null);
        const searchResults = await searchFunction(debouncedQuery);
        setResults(searchResults);
      } catch (err) {
        setError(err.message);
        setResults([]);
      } finally {
        setLoading(false);
      }
    };

    performSearch();
  }, [debouncedQuery, searchFunction]);

  return {
    query,
    setQuery,
    results,
    loading,
    error,
    hasQuery: debouncedQuery.trim() !== ''
  };
}

function ProductSearch() {
  const searchProducts = async (query) => {
    const response = await fetch(
      `https://api.example.com/products/search?q=${encodeURIComponent(query)}`
    );
    
    if (!response.ok) {
      throw new Error('Search failed');
    }
    
    const data = await response.json();
    return data.products;
  };

  const {
    query,
    setQuery,
    results,
    loading,
    error,
    hasQuery
  } = useSearch(searchProducts);

  return (
    <div className="product-search">
      <div className="search-input-container">
        <input
          type="text"
          placeholder="Search products..."
          value={query}
          onChange={(e) => setQuery(e.target.value)}
          className="search-input"
        />
        {loading && <div className="search-spinner">🔄</div>}
      </div>

      {error && (
        <div className="search-error">
          Error: {error}
        </div>
      )}

      {hasQuery && !loading && (
        <div className="search-results">
          <p>{results.length} results found</p>
          
          {results.length > 0 ? (
            <div className="results-grid">
              {results.map(product => (
                <div key={product.id} className="result-card">
                  <img src={product.image} alt={product.name} />
                  <h3>{product.name}</h3>
                  <p>${product.price}</p>
                </div>
              ))}
            </div>
          ) : (
            <div className="no-results">
              No products found for "{query}"
            </div>
          )}
        </div>
      )}
    </div>
  );
}
```

## Real-time Data with WebSockets

```jsx
import React, { useState, useEffect, useRef } from 'react';

function useWebSocket(url) {
  const [socket, setSocket] = useState(null);
  const [lastMessage, setLastMessage] = useState(null);
  const [connectionStatus, setConnectionStatus] = useState('Disconnected');
  const [error, setError] = useState(null);

  useEffect(() => {
    try {
      const ws = new WebSocket(url);
      
      ws.onopen = () => {
        setConnectionStatus('Connected');
        setError(null);
      };
      
      ws.onmessage = (event) => {
        const message = JSON.parse(event.data);
        setLastMessage(message);
      };
      
      ws.onclose = () => {
        setConnectionStatus('Disconnected');
      };
      
      ws.onerror = (error) => {
        setError('WebSocket error occurred');
        setConnectionStatus('Error');
      };
      
      setSocket(ws);
      
      return () => {
        ws.close();
      };
    } catch (err) {
      setError('Failed to connect to WebSocket');
      setConnectionStatus('Error');
    }
  }, [url]);

  const sendMessage = (message) => {
    if (socket && socket.readyState === WebSocket.OPEN) {
      socket.send(JSON.stringify(message));
    } else {
      console.error('WebSocket is not connected');
    }
  };

  return {
    lastMessage,
    connectionStatus,
    error,
    sendMessage
  };
}

function ChatRoom({ roomId }) {
  const [messages, setMessages] = useState([]);
  const [newMessage, setNewMessage] = useState('');
  const [user] = useState({ id: 1, name: 'John Doe' }); // This would come from auth
  
  const {
    lastMessage,
    connectionStatus,
    error,
    sendMessage
  } = useWebSocket(`ws://localhost:8080/chat/${roomId}`);

  // Handle incoming messages
  useEffect(() => {
    if (lastMessage) {
      setMessages(prev => [...prev, lastMessage]);
    }
  }, [lastMessage]);

  const handleSendMessage = (e) => {
    e.preventDefault();
    
    if (newMessage.trim() === '') return;
    
    const message = {
      id: Date.now(),
      text: newMessage,
      user: user,
      timestamp: new Date().toISOString()
    };
    
    sendMessage(message);
    setNewMessage('');
  };

  return (
    <div className="chat-room">
      <div className="chat-header">
        <h2>Chat Room {roomId}</h2>
        <div className={`connection-status ${connectionStatus.toLowerCase()}`}>
          {connectionStatus}
        </div>
      </div>

      {error && (
        <div className="error-message">{error}</div>
      )}

      <div className="messages-container">
        {messages.map(message => (
          <div key={message.id} className="message">
            <div className="message-header">
              <span className="username">{message.user.name}</span>
              <span className="timestamp">
                {new Date(message.timestamp).toLocaleTimeString()}
              </span>
            </div>
            <div className="message-text">{message.text}</div>
          </div>
        ))}
      </div>

      <form onSubmit={handleSendMessage} className="message-form">
        <input
          type="text"
          value={newMessage}
          onChange={(e) => setNewMessage(e.target.value)}
          placeholder="Type a message..."
          disabled={connectionStatus !== 'Connected'}
        />
        <button 
          type="submit" 
          disabled={connectionStatus !== 'Connected' || !newMessage.trim()}
        >
          Send
        </button>
      </form>
    </div>
  );
}
```

## Error Handling and Retry Logic

```jsx
import React, { useState, useCallback } from 'react';

function useRetry(asyncFunction, maxRetries = 3, delay = 1000) {
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  const [retryCount, setRetryCount] = useState(0);

  const execute = useCallback(async (...args) => {
    let currentRetry = 0;
    
    while (currentRetry <= maxRetries) {
      try {
        setLoading(true);
        setError(null);
        setRetryCount(currentRetry);
        
        const result = await asyncFunction(...args);
        setRetryCount(0);
        return result;
      } catch (err) {
        if (currentRetry === maxRetries) {
          setError(err.message);
          throw err;
        }
        
        currentRetry++;
        await new Promise(resolve => setTimeout(resolve, delay * currentRetry));
      } finally {
        setLoading(false);
      }
    }
  }, [asyncFunction, maxRetries, delay]);

  return {
    execute,
    loading,
    error,
    retryCount
  };
}

function RobustDataFetcher({ userId }) {
  const [userData, setUserData] = useState(null);

  const fetchUser = async (id) => {
    const response = await fetch(`https://api.example.com/users/${id}`);
    
    if (!response.ok) {
      throw new Error(`Failed to fetch user: ${response.status}`);
    }
    
    return response.json();
  };

  const {
    execute: fetchUserWithRetry,
    loading,
    error,
    retryCount
  } = useRetry(fetchUser, 3, 1000);

  const handleFetchUser = async () => {
    try {
      const user = await fetchUserWithRetry(userId);
      setUserData(user);
    } catch (err) {
      console.error('Failed to fetch user after retries:', err);
    }
  };

  return (
    <div>
      <button onClick={handleFetchUser} disabled={loading}>
        {loading ? 'Loading...' : 'Fetch User'}
      </button>
      
      {retryCount > 0 && (
        <div className="retry-info">
          Retry attempt: {retryCount}
        </div>
      )}
      
      {error && (
        <div className="error">
          Error: {error}
          <button onClick={handleFetchUser}>Try Again</button>
        </div>
      )}
      
      {userData && (
        <div className="user-data">
          <h3>{userData.name}</h3>
          <p>{userData.email}</p>
        </div>
      )}
    </div>
  );
}
```

## Optimistic Updates

```jsx
import React, { useState, useOptimistic } from 'react';

function TodoList() {
  const [todos, setTodos] = useState([]);
  const [optimisticTodos, addOptimisticTodo] = useOptimistic(
    todos,
    (state, newTodo) => [...state, newTodo]
  );

  const addTodo = async (text) => {
    const optimisticTodo = {
      id: Date.now(), // Temporary ID
      text,
      completed: false,
      optimistic: true
    };

    // Add optimistic update
    addOptimisticTodo(optimisticTodo);

    try {
      // Make API call
      const response = await fetch('/api/todos', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ text })
      });

      if (!response.ok) {
        throw new Error('Failed to add todo');
      }

      const newTodo = await response.json();
      
      // Update with real data
      setTodos(prev => [...prev, newTodo]);
    } catch (error) {
      // Optimistic update will be reverted automatically
      console.error('Failed to add todo:', error);
      // Show error message to user
    }
  };

  const toggleTodo = async (id) => {
    // Find the todo to update
    const todoToUpdate = todos.find(todo => todo.id === id);
    if (!todoToUpdate) return;

    // Optimistic update
    setTodos(prev => prev.map(todo =>
      todo.id === id ? { ...todo, completed: !todo.completed } : todo
    ));

    try {
      const response = await fetch(`/api/todos/${id}`, {
        method: 'PATCH',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ completed: !todoToUpdate.completed })
      });

      if (!response.ok) {
        throw new Error('Failed to update todo');
      }

      const updatedTodo = await response.json();
      
      // Update with server response
      setTodos(prev => prev.map(todo =>
        todo.id === id ? updatedTodo : todo
      ));
    } catch (error) {
      // Revert optimistic update
      setTodos(prev => prev.map(todo =>
        todo.id === id ? todoToUpdate : todo
      ));
      console.error('Failed to update todo:', error);
    }
  };

  return (
    <div>
      <h2>Optimistic Todo List</h2>
      
      <form onSubmit={(e) => {
        e.preventDefault();
        const formData = new FormData(e.target);
        addTodo(formData.get('text'));
        e.target.reset();
      }}>
        <input name="text" placeholder="Add a todo..." required />
        <button type="submit">Add</button>
      </form>

      <ul>
        {optimisticTodos.map(todo => (
          <li
            key={todo.id}
            className={todo.optimistic ? 'optimistic' : ''}
            style={{ opacity: todo.optimistic ? 0.7 : 1 }}
          >
            <label>
              <input
                type="checkbox"
                checked={todo.completed}
                onChange={() => toggleTodo(todo.id)}
                disabled={todo.optimistic}
              />
              {todo.text}
            </label>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

## Caching Strategies

```jsx
import React, { useState, useEffect, useCallback } from 'react';

// Simple in-memory cache
class Cache {
  constructor(maxSize = 100, ttl = 5 * 60 * 1000) { // 5 minutes default TTL
    this.cache = new Map();
    this.maxSize = maxSize;
    this.ttl = ttl;
  }

  set(key, value) {
    // Remove oldest items if cache is full
    if (this.cache.size >= this.maxSize) {
      const firstKey = this.cache.keys().next().value;
      this.cache.delete(firstKey);
    }

    this.cache.set(key, {
      value,
      timestamp: Date.now()
    });
  }

  get(key) {
    const item = this.cache.get(key);
    
    if (!item) {
      return null;
    }

    // Check if item has expired
    if (Date.now() - item.timestamp > this.ttl) {
      this.cache.delete(key);
      return null;
    }

    return item.value;
  }

  clear() {
    this.cache.clear();
  }

  delete(key) {
    this.cache.delete(key);
  }
}

const apiCache = new Cache();

function useCachedFetch(url, options = {}) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  const cacheKey = `${url}:${JSON.stringify(options)}`;

  const fetchData = useCallback(async (forceRefresh = false) => {
    // Check cache first
    if (!forceRefresh) {
      const cachedData = apiCache.get(cacheKey);
      if (cachedData) {
        setData(cachedData);
        setLoading(false);
        return cachedData;
      }
    }

    try {
      setLoading(true);
      setError(null);

      const response = await fetch(url, {
        headers: {
          'Content-Type': 'application/json',
          ...options.headers
        },
        ...options
      });

      if (!response.ok) {
        throw new Error(`Error: ${response.status}`);
      }

      const result = await response.json();
      
      // Cache the result
      apiCache.set(cacheKey, result);
      setData(result);
      
      return result;
    } catch (err) {
      setError(err.message);
      throw err;
    } finally {
      setLoading(false);
    }
  }, [url, cacheKey, JSON.stringify(options)]);

  useEffect(() => {
    fetchData();
  }, [fetchData]);

  const refresh = () => fetchData(true);
  const clearCache = () => apiCache.delete(cacheKey);

  return {
    data,
    loading,
    error,
    refresh,
    clearCache
  };
}

function CachedProductList() {
  const {
    data: products,
    loading,
    error,
    refresh,
    clearCache
  } = useCachedFetch('https://api.example.com/products');

  return (
    <div>
      <div className="controls">
        <button onClick={refresh} disabled={loading}>
          {loading ? 'Refreshing...' : 'Refresh'}
        </button>
        <button onClick={clearCache}>Clear Cache</button>
      </div>

      {error && <div className="error">Error: {error}</div>}

      {products && (
        <div className="product-list">
          {products.map(product => (
            <div key={product.id} className="product-item">
              <h3>{product.name}</h3>
              <p>${product.price}</p>
            </div>
          ))}
        </div>
      )}
    </div>
  );
}
```

## Complete Example: User Management Dashboard

```jsx
import React, { useState, useEffect, useCallback } from 'react';

// API service
class UserService {
  static baseURL = 'https://api.example.com';

  static async getUsers(page = 1, limit = 10, search = '') {
    const params = new URLSearchParams({
      page: page.toString(),
      limit: limit.toString(),
      ...(search && { search })
    });

    const response = await fetch(`${this.baseURL}/users?${params}`);
    
    if (!response.ok) {
      throw new Error(`Failed to fetch users: ${response.statusText}`);
    }
    
    return response.json();
  }

  static async createUser(userData) {
    const response = await fetch(`${this.baseURL}/users`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(userData)
    });

    if (!response.ok) {
      const errorData = await response.json();
      throw new Error(errorData.message || 'Failed to create user');
    }

    return response.json();
  }

  static async updateUser(id, userData) {
    const response = await fetch(`${this.baseURL}/users/${id}`, {
      method: 'PUT',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(userData)
    });

    if (!response.ok) {
      const errorData = await response.json();
      throw new Error(errorData.message || 'Failed to update user');
    }

    return response.json();
  }

  static async deleteUser(id) {
    const response = await fetch(`${this.baseURL}/users/${id}`, {
      method: 'DELETE'
    });

    if (!response.ok) {
      throw new Error('Failed to delete user');
    }
  }
}

// Custom hook for user management
function useUserManagement() {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  const [totalCount, setTotalCount] = useState(0);
  const [currentPage, setCurrentPage] = useState(1);
  const [searchQuery, setSearchQuery] = useState('');

  const fetchUsers = useCallback(async (page = currentPage, search = searchQuery) => {
    try {
      setLoading(true);
      setError(null);
      
      const result = await UserService.getUsers(page, 10, search);
      setUsers(result.users);
      setTotalCount(result.total);
      setCurrentPage(page);
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  }, [currentPage, searchQuery]);

  const createUser = async (userData) => {
    try {
      const newUser = await UserService.createUser(userData);
      setUsers(prev => [newUser, ...prev]);
      setTotalCount(prev => prev + 1);
      return newUser;
    } catch (err) {
      setError(err.message);
      throw err;
    }
  };

  const updateUser = async (id, userData) => {
    try {
      const updatedUser = await UserService.updateUser(id, userData);
      setUsers(prev => prev.map(user => 
        user.id === id ? updatedUser : user
      ));
      return updatedUser;
    } catch (err) {
      setError(err.message);
      throw err;
    }
  };

  const deleteUser = async (id) => {
    try {
      await UserService.deleteUser(id);
      setUsers(prev => prev.filter(user => user.id !== id));
      setTotalCount(prev => prev - 1);
    } catch (err) {
      setError(err.message);
      throw err;
    }
  };

  const search = (query) => {
    setSearchQuery(query);
    setCurrentPage(1);
    fetchUsers(1, query);
  };

  const goToPage = (page) => {
    setCurrentPage(page);
    fetchUsers(page, searchQuery);
  };

  // Initial load
  useEffect(() => {
    fetchUsers();
  }, []);

  return {
    users,
    loading,
    error,
    totalCount,
    currentPage,
    searchQuery,
    createUser,
    updateUser,
    deleteUser,
    search,
    goToPage,
    refresh: () => fetchUsers(currentPage, searchQuery)
  };
}

// User form component
function UserForm({ user, onSubmit, onCancel }) {
  const [formData, setFormData] = useState({
    name: user?.name || '',
    email: user?.email || '',
    role: user?.role || 'user'
  });
  const [loading, setLoading] = useState(false);

  const handleSubmit = async (e) => {
    e.preventDefault();
    setLoading(true);
    
    try {
      await onSubmit(formData);
    } finally {
      setLoading(false);
    }
  };

  return (
    <form onSubmit={handleSubmit} className="user-form">
      <h3>{user ? 'Edit User' : 'Create User'}</h3>
      
      <div className="form-group">
        <label>Name:</label>
        <input
          type="text"
          value={formData.name}
          onChange={(e) => setFormData(prev => ({ ...prev, name: e.target.value }))}
          required
          disabled={loading}
        />
      </div>

      <div className="form-group">
        <label>Email:</label>
        <input
          type="email"
          value={formData.email}
          onChange={(e) => setFormData(prev => ({ ...prev, email: e.target.value }))}
          required
          disabled={loading}
        />
      </div>

      <div className="form-group">
        <label>Role:</label>
        <select
          value={formData.role}
          onChange={(e) => setFormData(prev => ({ ...prev, role: e.target.value }))}
          disabled={loading}
        >
          <option value="user">User</option>
          <option value="admin">Admin</option>
        </select>
      </div>

      <div className="form-actions">
        <button type="submit" disabled={loading}>
          {loading ? 'Saving...' : (user ? 'Update' : 'Create')}
        </button>
        <button type="button" onClick={onCancel} disabled={loading}>
          Cancel
        </button>
      </div>
    </form>
  );
}

// Main dashboard component
function UserManagementDashboard() {
  const {
    users,
    loading,
    error,
    totalCount,
    currentPage,
    searchQuery,
    createUser,
    updateUser,
    deleteUser,
    search,
    goToPage
  } = useUserManagement();

  const [showForm, setShowForm] = useState(false);
  const [editingUser, setEditingUser] = useState(null);
  const [searchInput, setSearchInput] = useState('');

  const handleSearch = (e) => {
    e.preventDefault();
    search(searchInput);
  };

  const handleCreateUser = async (userData) => {
    await createUser(userData);
    setShowForm(false);
  };

  const handleUpdateUser = async (userData) => {
    await updateUser(editingUser.id, userData);
    setEditingUser(null);
    setShowForm(false);
  };

  const handleDeleteUser = async (id) => {
    if (window.confirm('Are you sure you want to delete this user?')) {
      await deleteUser(id);
    }
  };

  const totalPages = Math.ceil(totalCount / 10);

  return (
    <div className="user-dashboard">
      <h1>User Management Dashboard</h1>

      {error && (
        <div className="error-banner">
          Error: {error}
        </div>
      )}

      <div className="dashboard-controls">
        <form onSubmit={handleSearch} className="search-form">
          <input
            type="text"
            placeholder="Search users..."
            value={searchInput}
            onChange={(e) => setSearchInput(e.target.value)}
          />
          <button type="submit">Search</button>
        </form>

        <button 
          onClick={() => setShowForm(true)}
          className="create-button"
        >
          Create User
        </button>
      </div>

      {showForm && (
        <div className="modal-overlay">
          <div className="modal">
            <UserForm
              user={editingUser}
              onSubmit={editingUser ? handleUpdateUser : handleCreateUser}
              onCancel={() => {
                setShowForm(false);
                setEditingUser(null);
              }}
            />
          </div>
        </div>
      )}

      <div className="user-stats">
        <p>Total Users: {totalCount}</p>
        <p>Page {currentPage} of {totalPages}</p>
      </div>

      {loading && users.length === 0 ? (
        <div className="loading">Loading users...</div>
      ) : (
        <>
          <div className="user-table">
            <table>
              <thead>
                <tr>
                  <th>Name</th>
                  <th>Email</th>
                  <th>Role</th>
                  <th>Actions</th>
                </tr>
              </thead>
              <tbody>
                {users.map(user => (
                  <tr key={user.id}>
                    <td>{user.name}</td>
                    <td>{user.email}</td>
                    <td>{user.role}</td>
                    <td>
                      <button
                        onClick={() => {
                          setEditingUser(user);
                          setShowForm(true);
                        }}
                      >
                        Edit
                      </button>
                      <button
                        onClick={() => handleDeleteUser(user.id)}
                        className="delete-button"
                      >
                        Delete
                      </button>
                    </td>
                  </tr>
                ))}
              </tbody>
            </table>
          </div>

          {totalPages > 1 && (
            <div className="pagination">
              <button
                onClick={() => goToPage(currentPage - 1)}
                disabled={currentPage === 1}
              >
                Previous
              </button>
              
              {Array.from({ length: totalPages }, (_, i) => i + 1).map(page => (
                <button
                  key={page}
                  onClick={() => goToPage(page)}
                  className={page === currentPage ? 'active' : ''}
                >
                  {page}
                </button>
              ))}
              
              <button
                onClick={() => goToPage(currentPage + 1)}
                disabled={currentPage === totalPages}
              >
                Next
              </button>
            </div>
          )}
        </>
      )}

      {loading && users.length > 0 && (
        <div className="loading-overlay">Loading...</div>
      )}
    </div>
  );
}

export default UserManagementDashboard;
```

## Best Practices and Performance Tips

### 1. Request Cancellation

Always cancel ongoing requests when components unmount:

```jsx
function useApi(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const controller = new AbortController();

    const fetchData = async () => {
      try {
        const response = await fetch(url, {
          signal: controller.signal
        });
        const result = await response.json();
        setData(result);
      } catch (err) {
        if (err.name !== 'AbortError') {
          setError(err.message);
        }
      } finally {
        setLoading(false);
      }
    };

    fetchData();

    return () => {
      controller.abort();
    };
  }, [url]);

  return { data, loading, error };
}
```

### 2. Error Boundaries for API Errors

```jsx
class ApiErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }

  componentDidCatch(error, errorInfo) {
    console.error('API Error:', error, errorInfo);
    // Log to error reporting service
  }

  render() {
    if (this.state.hasError) {
      return (
        <div className="api-error">
          <h2>Something went wrong</h2>
          <button onClick={() => this.setState({ hasError: false, error: null })}>
            Try again
          </button>
        </div>
      );
    }

    return this.props.children;
  }
}
```

### 3. Request Deduplication

```jsx
const requestCache = new Map();

function useApiWithDeduplication(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const fetchData = async () => {
      // Check if request is already in progress
      if (requestCache.has(url)) {
        const cachedPromise = requestCache.get(url);
        try {
          const result = await cachedPromise;
          setData(result);
        } catch (err) {
          setError(err.message);
        } finally {
          setLoading(false);
        }
        return;
      }

      // Create new request promise
      const requestPromise = fetch(url).then(res => res.json());
      requestCache.set(url, requestPromise);

      try {
        const result = await requestPromise;
        setData(result);
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
        requestCache.delete(url);
      }
    };

    fetchData();
  }, [url]);

  return { data, loading, error };
}
```

## Summary

This guide covered comprehensive data fetching techniques in React:

1. **Basic Fetch Operations**: GET, POST, PUT, DELETE requests with proper error handling
2. **Custom Hooks**: Reusable hooks for common data fetching patterns
3. **Axios Integration**: Using a popular HTTP client library with interceptors
4. **Advanced Patterns**: Pagination, infinite scrolling, search with debouncing
5. **Real-time Data**: WebSocket integration for live updates
6. **Error Handling**: Retry logic, error boundaries, and user feedback
7. **Performance Optimization**: Caching, request cancellation, and deduplication
8. **Optimistic Updates**: Improving perceived performance with optimistic UI updates

Key principles for effective data fetching:

- Always handle loading and error states
- Implement proper request cancellation
- Use debouncing for search functionality
- Cache frequently accessed data
- Provide clear user feedback
- Handle network failures gracefully
- Optimize for performance with techniques like pagination and infinite scrolling

By mastering these data fetching techniques, you can build robust, performant React applications that provide excellent user experiences even when dealing with complex data requirements and unreliable network conditions.

## Practice Exercise

Build a comprehensive blog application with the following features:

1. Post listing with pagination and infinite scroll
2. Search functionality with debouncing
3. Real-time comments using WebSockets
4. Optimistic updates for likes and comments
5. Robust error handling with retry logic
6. Caching for frequently accessed posts
7. Image upload with progress indicators
8. Offline support with request queuing

This exercise will help you apply all the concepts covered in this guide to create a production-ready application.

## Additional Resources

- [Fetch API MDN Documentation](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API)
- [Axios Documentation](https://axios-http.com/docs/intro)
- [React Query Documentation](https://tanstack.com/query/latest)
- [SWR Documentation](https://swr.vercel.app/)
- [WebSocket API MDN](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API)
- [AbortController MDN](https://developer.mozilla.org/en-US/docs/Web/API/AbortController)
