# Server-Side Rendering (SSR) in Next.js

## Table of Contents
1. [Introduction to SSR](#introduction)
2. [getServerSideProps Function](#getserversideprops)
3. [When to Use SSR vs SSG](#when-to-use)
4. [Dynamic Server-Side Rendering](#dynamic-ssr)
5. [Authentication with SSR](#authentication)
6. [Performance Optimization](#performance)
7. [Caching Strategies](#caching)
8. [Error Handling](#error-handling)
9. [Best Practices](#best-practices)
10. [Real-world Examples](#examples)

## Introduction to SSR {#introduction}

Server-Side Rendering (SSR) in Next.js renders pages on the server for each request, providing fresh data and dynamic content while maintaining SEO benefits.

### SSR vs Other Rendering Methods

```typescript
interface RenderingMethod {
  name: string;
  renderTime: 'build' | 'request' | 'client';
  dataFreshness: 'static' | 'fresh' | 'stale-while-revalidate';
  seoFriendly: boolean;
  performanceRating: 'fast' | 'medium' | 'slow';
  useCase: string[];
}

const renderingMethods: RenderingMethod[] = [
  {
    name: 'Static Site Generation (SSG)',
    renderTime: 'build',
    dataFreshness: 'static',
    seoFriendly: true,
    performanceRating: 'fast',
    useCase: ['blogs', 'marketing pages', 'documentation']
  },
  {
    name: 'Server-Side Rendering (SSR)',
    renderTime: 'request',
    dataFreshness: 'fresh',
    seoFriendly: true,
    performanceRating: 'medium',
    useCase: ['user dashboards', 'real-time data', 'personalized content']
  },
  {
    name: 'Client-Side Rendering (CSR)',
    renderTime: 'client',
    dataFreshness: 'fresh',
    seoFriendly: false,
    performanceRating: 'slow',
    useCase: ['interactive apps', 'admin panels', 'private content']
  },
  {
    name: 'Incremental Static Regeneration (ISR)',
    renderTime: 'build',
    dataFreshness: 'stale-while-revalidate',
    seoFriendly: true,
    performanceRating: 'fast',
    useCase: ['news sites', 'e-commerce', 'content with periodic updates']
  }
];
```

### Benefits of SSR

```typescript
const ssrBenefits = {
  seo: {
    description: 'Full HTML content available for crawlers',
    benefits: [
      'Meta tags rendered server-side',
      'Social media previews work correctly',
      'Search engines index complete content',
      'No JavaScript required for content visibility'
    ]
  },
  performance: {
    description: 'Faster initial page load',
    benefits: [
      'Content visible immediately',
      'Reduced Time to First Contentful Paint',
      'Better perceived performance',
      'Works without JavaScript enabled'
    ]
  },
  dataFreshness: {
    description: 'Always up-to-date content',
    benefits: [
      'Real-time data on every request',
      'User-specific personalization',
      'Dynamic content based on request context',
      'No stale content issues'
    ]
  }
};
```

## getServerSideProps Function {#getserversideprops}

`getServerSideProps` runs on every request and provides fresh data to your page components.

### Basic getServerSideProps Example

```typescript
// pages/dashboard/index.tsx
import { GetServerSideProps } from 'next';

interface UserStats {
  totalOrders: number;
  totalSpent: number;
  pendingOrders: number;
  favoriteProducts: number;
}

interface RecentOrder {
  id: string;
  date: string;
  total: number;
  status: 'pending' | 'shipped' | 'delivered' | 'cancelled';
  items: {
    name: string;
    quantity: number;
    price: number;
  }[];
}

interface DashboardProps {
  user: {
    id: string;
    name: string;
    email: string;
    avatar: string;
    memberSince: string;
  };
  stats: UserStats;
  recentOrders: RecentOrder[];
  notifications: {
    id: string;
    type: 'info' | 'warning' | 'success';
    message: string;
    timestamp: string;
  }[];
}

export default function Dashboard({ user, stats, recentOrders, notifications }: DashboardProps) {
  return (
    <div className="min-h-screen bg-gray-50">
      {/* Header */}
      <header className="bg-white shadow-sm">
        <div className="container mx-auto px-4 py-6">
          <div className="flex items-center justify-between">
            <div className="flex items-center space-x-4">
              <img 
                src={user.avatar} 
                alt={user.name}
                className="w-12 h-12 rounded-full"
              />
              <div>
                <h1 className="text-2xl font-bold text-gray-900">
                  Welcome back, {user.name}!
                </h1>
                <p className="text-gray-600">
                  Member since {new Date(user.memberSince).getFullYear()}
                </p>
              </div>
            </div>
            <div className="relative">
              {notifications.length > 0 && (
                <span className="absolute -top-2 -right-2 bg-red-500 text-white text-xs rounded-full h-5 w-5 flex items-center justify-center">
                  {notifications.length}
                </span>
              )}
              <button className="p-2 text-gray-600 hover:text-gray-900">
                🔔
              </button>
            </div>
          </div>
        </div>
      </header>

      <main className="container mx-auto px-4 py-8">
        {/* Notifications */}
        {notifications.length > 0 && (
          <div className="mb-8 space-y-3">
            {notifications.map((notification) => (
              <div 
                key={notification.id}
                className={`p-4 rounded-lg ${
                  notification.type === 'info' ? 'bg-blue-100 text-blue-800' :
                  notification.type === 'warning' ? 'bg-yellow-100 text-yellow-800' :
                  'bg-green-100 text-green-800'
                }`}
              >
                <div className="flex justify-between items-start">
                  <p>{notification.message}</p>
                  <span className="text-xs opacity-75">
                    {new Date(notification.timestamp).toLocaleTimeString()}
                  </span>
                </div>
              </div>
            ))}
          </div>
        )}

        {/* Stats Grid */}
        <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6 mb-8">
          <div className="bg-white p-6 rounded-lg shadow-md">
            <h3 className="text-lg font-semibold text-gray-900 mb-2">Total Orders</h3>
            <p className="text-3xl font-bold text-blue-600">{stats.totalOrders}</p>
          </div>
          <div className="bg-white p-6 rounded-lg shadow-md">
            <h3 className="text-lg font-semibold text-gray-900 mb-2">Total Spent</h3>
            <p className="text-3xl font-bold text-green-600">${stats.totalSpent.toFixed(2)}</p>
          </div>
          <div className="bg-white p-6 rounded-lg shadow-md">
            <h3 className="text-lg font-semibold text-gray-900 mb-2">Pending Orders</h3>
            <p className="text-3xl font-bold text-orange-600">{stats.pendingOrders}</p>
          </div>
          <div className="bg-white p-6 rounded-lg shadow-md">
            <h3 className="text-lg font-semibold text-gray-900 mb-2">Favorites</h3>
            <p className="text-3xl font-bold text-purple-600">{stats.favoriteProducts}</p>
          </div>
        </div>

        {/* Recent Orders */}
        <div className="bg-white rounded-lg shadow-md">
          <div className="p-6 border-b border-gray-200">
            <h2 className="text-xl font-semibold text-gray-900">Recent Orders</h2>
          </div>
          <div className="divide-y divide-gray-200">
            {recentOrders.map((order) => (
              <div key={order.id} className="p-6">
                <div className="flex justify-between items-start mb-3">
                  <div>
                    <p className="font-semibold">Order #{order.id}</p>
                    <p className="text-sm text-gray-600">
                      {new Date(order.date).toLocaleDateString()}
                    </p>
                  </div>
                  <div className="text-right">
                    <p className="font-semibold">${order.total.toFixed(2)}</p>
                    <span className={`inline-block px-2 py-1 text-xs rounded-full ${
                      order.status === 'delivered' ? 'bg-green-100 text-green-800' :
                      order.status === 'shipped' ? 'bg-blue-100 text-blue-800' :
                      order.status === 'pending' ? 'bg-yellow-100 text-yellow-800' :
                      'bg-red-100 text-red-800'
                    }`}>
                      {order.status.charAt(0).toUpperCase() + order.status.slice(1)}
                    </span>
                  </div>
                </div>
                <div className="space-y-1">
                  {order.items.map((item, index) => (
                    <p key={index} className="text-sm text-gray-600">
                      {item.quantity}x {item.name} - ${item.price.toFixed(2)}
                    </p>
                  ))}
                </div>
              </div>
            ))}
          </div>
        </div>
      </main>
    </div>
  );
}

export const getServerSideProps: GetServerSideProps<DashboardProps> = async (context) => {
  // Get user session/authentication
  const { req, res } = context;
  
  try {
    // In a real app, you'd get the user ID from the session/JWT
    const userId = req.cookies.userId || req.headers.authorization?.split(' ')[1];
    
    if (!userId) {
      return {
        redirect: {
          destination: '/login',
          permanent: false,
        },
      };
    }

    // Fetch user data, stats, and orders in parallel
    const [userResponse, statsResponse, ordersResponse, notificationsResponse] = await Promise.all([
      fetch(`${process.env.API_BASE_URL}/users/${userId}`),
      fetch(`${process.env.API_BASE_URL}/users/${userId}/stats`),
      fetch(`${process.env.API_BASE_URL}/users/${userId}/orders?limit=5&sort=date:desc`),
      fetch(`${process.env.API_BASE_URL}/users/${userId}/notifications?unread=true`)
    ]);

    // Check if user exists
    if (!userResponse.ok) {
      return {
        notFound: true,
      };
    }

    const [user, stats, recentOrders, notifications] = await Promise.all([
      userResponse.json(),
      statsResponse.json(),
      ordersResponse.json(),
      notificationsResponse.json()
    ]);

    return {
      props: {
        user,
        stats,
        recentOrders,
        notifications,
      },
    };
  } catch (error) {
    console.error('Dashboard data fetch error:', error);
    
    return {
      redirect: {
        destination: '/error',
        permanent: false,
      },
    };
  }
};
```

### Advanced getServerSideProps with Context

```typescript
// pages/search/index.tsx
import { GetServerSideProps } from 'next';
import { ParsedUrlQuery } from 'querystring';

interface SearchFilters {
  category?: string;
  priceMin?: number;
  priceMax?: number;
  rating?: number;
  brand?: string;
  sortBy?: 'price' | 'rating' | 'name' | 'date';
  sortOrder?: 'asc' | 'desc';
}

interface SearchResult {
  id: string;
  name: string;
  price: number;
  rating: number;
  image: string;
  category: string;
  brand: string;
  description: string;
}

interface SearchPageProps {
  query: string;
  results: SearchResult[];
  totalResults: number;
  currentPage: number;
  totalPages: number;
  filters: SearchFilters;
  availableFilters: {
    categories: { name: string; count: number }[];
    brands: { name: string; count: number }[];
    priceRange: { min: number; max: number };
  };
}

export default function SearchPage({ 
  query, 
  results, 
  totalResults, 
  currentPage, 
  totalPages,
  filters,
  availableFilters 
}: SearchPageProps) {
  return (
    <div className="container mx-auto px-4 py-8">
      {/* Search Header */}
      <div className="mb-8">
        <h1 className="text-3xl font-bold mb-2">
          Search Results for "{query}"
        </h1>
        <p className="text-gray-600">
          {totalResults.toLocaleString()} results found
        </p>
      </div>

      <div className="flex flex-col lg:flex-row gap-8">
        {/* Filters Sidebar */}
        <aside className="lg:w-1/4">
          <div className="bg-white rounded-lg shadow-md p-6">
            <h2 className="text-lg font-semibold mb-4">Filters</h2>
            
            {/* Category Filter */}
            <div className="mb-6">
              <h3 className="font-medium mb-3">Category</h3>
              <div className="space-y-2">
                {availableFilters.categories.map((category) => (
                  <label key={category.name} className="flex items-center">
                    <input 
                      type="checkbox" 
                      checked={filters.category === category.name}
                      className="mr-2"
                    />
                    <span className="text-sm">
                      {category.name} ({category.count})
                    </span>
                  </label>
                ))}
              </div>
            </div>

            {/* Price Range Filter */}
            <div className="mb-6">
              <h3 className="font-medium mb-3">Price Range</h3>
              <div className="flex space-x-2">
                <input 
                  type="number" 
                  placeholder="Min"
                  value={filters.priceMin || ''}
                  className="w-full px-3 py-2 border rounded text-sm"
                />
                <input 
                  type="number" 
                  placeholder="Max"
                  value={filters.priceMax || ''}
                  className="w-full px-3 py-2 border rounded text-sm"
                />
              </div>
              <p className="text-xs text-gray-500 mt-1">
                ${availableFilters.priceRange.min} - ${availableFilters.priceRange.max}
              </p>
            </div>

            {/* Brand Filter */}
            <div className="mb-6">
              <h3 className="font-medium mb-3">Brand</h3>
              <div className="space-y-2 max-h-40 overflow-y-auto">
                {availableFilters.brands.map((brand) => (
                  <label key={brand.name} className="flex items-center">
                    <input 
                      type="checkbox" 
                      checked={filters.brand === brand.name}
                      className="mr-2"
                    />
                    <span className="text-sm">
                      {brand.name} ({brand.count})
                    </span>
                  </label>
                ))}
              </div>
            </div>

            {/* Rating Filter */}
            <div>
              <h3 className="font-medium mb-3">Rating</h3>
              <div className="space-y-2">
                {[4, 3, 2, 1].map((rating) => (
                  <label key={rating} className="flex items-center">
                    <input 
                      type="radio" 
                      name="rating"
                      checked={filters.rating === rating}
                      className="mr-2"
                    />
                    <span className="text-sm flex items-center">
                      {Array.from({ length: rating }).map((_, i) => (
                        <span key={i} className="text-yellow-500">★</span>
                      ))}
                      <span className="ml-1">& up</span>
                    </span>
                  </label>
                ))}
              </div>
            </div>
          </div>
        </aside>

        {/* Results */}
        <main className="lg:w-3/4">
          {/* Sort Options */}
          <div className="flex justify-between items-center mb-6">
            <p className="text-gray-600">
              Showing {((currentPage - 1) * 12) + 1}-{Math.min(currentPage * 12, totalResults)} of {totalResults} results
            </p>
            <select className="px-3 py-2 border rounded text-sm">
              <option value="relevance">Sort by Relevance</option>
              <option value="price-asc">Price: Low to High</option>
              <option value="price-desc">Price: High to Low</option>
              <option value="rating">Customer Rating</option>
              <option value="newest">Newest First</option>
            </select>
          </div>

          {/* Product Grid */}
          <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6 mb-8">
            {results.map((product) => (
              <div key={product.id} className="bg-white rounded-lg shadow-md overflow-hidden">
                <img 
                  src={product.image} 
                  alt={product.name}
                  className="w-full h-48 object-cover"
                />
                <div className="p-4">
                  <h3 className="font-semibold mb-2 line-clamp-2">{product.name}</h3>
                  <p className="text-gray-600 text-sm mb-2 line-clamp-2">
                    {product.description}
                  </p>
                  <div className="flex justify-between items-center mb-2">
                    <span className="text-lg font-bold text-green-600">
                      ${product.price.toFixed(2)}
                    </span>
                    <div className="flex items-center">
                      <span className="text-yellow-500 mr-1">★</span>
                      <span className="text-sm text-gray-600">{product.rating}</span>
                    </div>
                  </div>
                  <p className="text-xs text-gray-500">{product.brand}</p>
                </div>
              </div>
            ))}
          </div>

          {/* Pagination */}
          {totalPages > 1 && (
            <div className="flex justify-center space-x-2">
              {Array.from({ length: Math.min(5, totalPages) }).map((_, i) => {
                const page = i + 1;
                return (
                  <button
                    key={page}
                    className={`px-3 py-2 rounded ${
                      page === currentPage 
                        ? 'bg-blue-600 text-white' 
                        : 'bg-white text-gray-700 border'
                    }`}
                  >
                    {page}
                  </button>
                );
              })}
            </div>
          )}
        </main>
      </div>
    </div>
  );
}

export const getServerSideProps: GetServerSideProps<SearchPageProps> = async (context) => {
  const { query: urlQuery } = context;
  
  try {
    // Parse query parameters
    const query = (urlQuery.q as string) || '';
    const page = parseInt((urlQuery.page as string) || '1');
    const category = urlQuery.category as string;
    const priceMin = urlQuery.priceMin ? parseFloat(urlQuery.priceMin as string) : undefined;
    const priceMax = urlQuery.priceMax ? parseFloat(urlQuery.priceMax as string) : undefined;
    const rating = urlQuery.rating ? parseInt(urlQuery.rating as string) : undefined;
    const brand = urlQuery.brand as string;
    const sortBy = (urlQuery.sortBy as string) || 'relevance';
    const sortOrder = (urlQuery.sortOrder as 'asc' | 'desc') || 'desc';

    if (!query) {
      return {
        redirect: {
          destination: '/',
          permanent: false,
        },
      };
    }

    // Build search parameters
    const searchParams = new URLSearchParams({
      q: query,
      page: page.toString(),
      limit: '12',
      sortBy,
      sortOrder,
    });

    // Add filters to search params
    if (category) searchParams.append('category', category);
    if (priceMin !== undefined) searchParams.append('priceMin', priceMin.toString());
    if (priceMax !== undefined) searchParams.append('priceMax', priceMax.toString());
    if (rating !== undefined) searchParams.append('rating', rating.toString());
    if (brand) searchParams.append('brand', brand);

    // Fetch search results and filters in parallel
    const [searchResponse, filtersResponse] = await Promise.all([
      fetch(`${process.env.API_BASE_URL}/search?${searchParams}`),
      fetch(`${process.env.API_BASE_URL}/search/filters?q=${encodeURIComponent(query)}`)
    ]);

    if (!searchResponse.ok) {
      throw new Error('Search API failed');
    }

    const searchData = await searchResponse.json();
    const filtersData = await filtersResponse.json();

    return {
      props: {
        query,
        results: searchData.results,
        totalResults: searchData.totalResults,
        currentPage: page,
        totalPages: Math.ceil(searchData.totalResults / 12),
        filters: {
          category,
          priceMin,
          priceMax,
          rating,
          brand,
          sortBy,
          sortOrder,
        },
        availableFilters: filtersData,
      },
    };
  } catch (error) {
    console.error('Search error:', error);
    
    return {
      props: {
        query: (urlQuery.q as string) || '',
        results: [],
        totalResults: 0,
        currentPage: 1,
        totalPages: 0,
        filters: {},
        availableFilters: {
          categories: [],
          brands: [],
          priceRange: { min: 0, max: 0 },
        },
      },
    };
  }
};
```

## Authentication with SSR {#authentication}

SSR is perfect for handling authentication and user-specific content.

### Protected Routes with SSR

```typescript
// lib/auth.ts
import jwt from 'jsonwebtoken';
import { GetServerSidePropsContext } from 'next';

export interface User {
  id: string;
  email: string;
  name: string;
  role: 'user' | 'admin' | 'moderator';
  avatar?: string;
  preferences: {
    theme: 'light' | 'dark';
    language: string;
    timezone: string;
  };
}

export async function getServerSideUser(context: GetServerSidePropsContext): Promise<User | null> {
  try {
    const { req } = context;
    
    // Get token from cookie or authorization header
    const token = req.cookies.authToken || 
                 req.headers.authorization?.replace('Bearer ', '');
    
    if (!token) {
      return null;
    }

    // Verify JWT token
    const decoded = jwt.verify(token, process.env.JWT_SECRET!) as { userId: string };
    
    // Fetch user data from database/API
    const userResponse = await fetch(`${process.env.API_BASE_URL}/users/${decoded.userId}`, {
      headers: {
        Authorization: `Bearer ${token}`,
      },
    });

    if (!userResponse.ok) {
      return null;
    }

    return await userResponse.json();
  } catch (error) {
    console.error('Authentication error:', error);
    return null;
  }
}

export function requireAuth<T extends Record<string, any>>(
  getServerSidePropsFunc: (context: GetServerSidePropsContext, user: User) => Promise<{ props: T }>,
  redirectTo = '/login'
) {
  return async (context: GetServerSidePropsContext) => {
    const user = await getServerSideUser(context);
    
    if (!user) {
      return {
        redirect: {
          destination: `${redirectTo}?returnUrl=${encodeURIComponent(context.resolvedUrl)}`,
          permanent: false,
        },
      };
    }

    return getServerSidePropsFunc(context, user);
  };
}

export function requireRole(
  roles: User['role'][],
  getServerSidePropsFunc: (context: GetServerSidePropsContext, user: User) => Promise<{ props: any }>,
  redirectTo = '/unauthorized'
) {
  return async (context: GetServerSidePropsContext) => {
    const user = await getServerSideUser(context);
    
    if (!user) {
      return {
        redirect: {
          destination: '/login',
          permanent: false,
        },
      };
    }

    if (!roles.includes(user.role)) {
      return {
        redirect: {
          destination: redirectTo,
          permanent: false,
        },
      };
    }

    return getServerSidePropsFunc(context, user);
  };
}
```

### User Profile Page with Authentication

```typescript
// pages/profile/index.tsx
import { GetServerSideProps } from 'next';
import { requireAuth, User } from '../../lib/auth';

interface UserProfile extends User {
  stats: {
    postsCount: number;
    followersCount: number;
    followingCount: number;
    joinDate: string;
  };
  recentActivity: {
    id: string;
    type: 'post' | 'comment' | 'like' | 'follow';
    description: string;
    timestamp: string;
  }[];
  settings: {
    isProfilePublic: boolean;
    emailNotifications: boolean;
    marketingEmails: boolean;
    twoFactorEnabled: boolean;
  };
}

interface ProfilePageProps {
  profile: UserProfile;
  isOwnProfile: boolean;
}

export default function ProfilePage({ profile, isOwnProfile }: ProfilePageProps) {
  return (
    <div className="min-h-screen bg-gray-50">
      <div className="container mx-auto px-4 py-8">
        {/* Profile Header */}
        <div className="bg-white rounded-lg shadow-md p-8 mb-8">
          <div className="flex flex-col md:flex-row items-start md:items-center space-y-4 md:space-y-0 md:space-x-6">
            <img 
              src={profile.avatar || '/default-avatar.png'} 
              alt={profile.name}
              className="w-24 h-24 rounded-full"
            />
            <div className="flex-1">
              <h1 className="text-3xl font-bold text-gray-900 mb-2">{profile.name}</h1>
              <p className="text-gray-600 mb-4">{profile.email}</p>
              <div className="flex space-x-6 text-sm text-gray-600">
                <span><strong>{profile.stats.postsCount}</strong> Posts</span>
                <span><strong>{profile.stats.followersCount}</strong> Followers</span>
                <span><strong>{profile.stats.followingCount}</strong> Following</span>
              </div>
              <p className="text-sm text-gray-500 mt-2">
                Member since {new Date(profile.stats.joinDate).toLocaleDateString()}
              </p>
            </div>
            {isOwnProfile && (
              <div className="flex space-x-3">
                <button className="px-4 py-2 bg-blue-600 text-white rounded-lg hover:bg-blue-700">
                  Edit Profile
                </button>
                <button className="px-4 py-2 border border-gray-300 text-gray-700 rounded-lg hover:bg-gray-50">
                  Settings
                </button>
              </div>
            )}
          </div>
        </div>

        <div className="grid grid-cols-1 lg:grid-cols-3 gap-8">
          {/* Recent Activity */}
          <div className="lg:col-span-2">
            <div className="bg-white rounded-lg shadow-md">
              <div className="p-6 border-b border-gray-200">
                <h2 className="text-xl font-semibold">Recent Activity</h2>
              </div>
              <div className="divide-y divide-gray-200">
                {profile.recentActivity.map((activity) => (
                  <div key={activity.id} className="p-6">
                    <div className="flex items-start space-x-3">
                      <div className={`w-3 h-3 rounded-full mt-2 ${
                        activity.type === 'post' ? 'bg-blue-500' :
                        activity.type === 'comment' ? 'bg-green-500' :
                        activity.type === 'like' ? 'bg-red-500' :
                        'bg-purple-500'
                      }`} />
                      <div className="flex-1">
                        <p className="text-gray-900">{activity.description}</p>
                        <p className="text-sm text-gray-500 mt-1">
                          {new Date(activity.timestamp).toLocaleString()}
                        </p>
                      </div>
                    </div>
                  </div>
                ))}
              </div>
            </div>
          </div>

          {/* Profile Info & Settings */}
          <div className="space-y-6">
            {/* Profile Stats */}
            <div className="bg-white rounded-lg shadow-md p-6">
              <h3 className="text-lg font-semibold mb-4">Profile Stats</h3>
              <div className="space-y-3">
                <div className="flex justify-between">
                  <span className="text-gray-600">Profile Views</span>
                  <span className="font-medium">1,234</span>
                </div>
                <div className="flex justify-between">
                  <span className="text-gray-600">Posts This Month</span>
                  <span className="font-medium">12</span>
                </div>
                <div className="flex justify-between">
                  <span className="text-gray-600">Average Likes</span>
                  <span className="font-medium">45</span>
                </div>
              </div>
            </div>

            {/* Privacy Settings (only for own profile) */}
            {isOwnProfile && (
              <div className="bg-white rounded-lg shadow-md p-6">
                <h3 className="text-lg font-semibold mb-4">Privacy & Settings</h3>
                <div className="space-y-4">
                  <div className="flex items-center justify-between">
                    <span className="text-gray-700">Public Profile</span>
                    <input 
                      type="checkbox" 
                      checked={profile.settings.isProfilePublic}
                      className="rounded"
                    />
                  </div>
                  <div className="flex items-center justify-between">
                    <span className="text-gray-700">Email Notifications</span>
                    <input 
                      type="checkbox" 
                      checked={profile.settings.emailNotifications}
                      className="rounded"
                    />
                  </div>
                  <div className="flex items-center justify-between">
                    <span className="text-gray-700">Marketing Emails</span>
                    <input 
                      type="checkbox" 
                      checked={profile.settings.marketingEmails}
                      className="rounded"
                    />
                  </div>
                  <div className="flex items-center justify-between">
                    <span className="text-gray-700">Two-Factor Auth</span>
                    <span className={`px-2 py-1 text-xs rounded-full ${
                      profile.settings.twoFactorEnabled 
                        ? 'bg-green-100 text-green-800' 
                        : 'bg-red-100 text-red-800'
                    }`}>
                      {profile.settings.twoFactorEnabled ? 'Enabled' : 'Disabled'}
                    </span>
                  </div>
                </div>
              </div>
            )}
          </div>
        </div>
      </div>
    </div>
  );
}

export const getServerSideProps: GetServerSideProps<ProfilePageProps> = requireAuth(
  async (context, user) => {
    try {
      // Get profile ID from URL or use current user
      const profileId = context.query.id as string || user.id;
      const isOwnProfile = profileId === user.id;

      // Fetch detailed profile data
      const profileResponse = await fetch(
        `${process.env.API_BASE_URL}/users/${profileId}/profile`,
        {
          headers: {
            Authorization: `Bearer ${context.req.cookies.authToken}`,
          },
        }
      );

      if (!profileResponse.ok) {
        return {
          notFound: true,
        };
      }

      const profile: UserProfile = await profileResponse.json();

      // Don't show private profiles to other users
      if (!isOwnProfile && !profile.settings.isProfilePublic) {
        return {
          notFound: true,
        };
      }

      return {
        props: {
          profile,
          isOwnProfile,
        },
      };
    } catch (error) {
      console.error('Profile fetch error:', error);
      return {
        notFound: true,
      };
    }
  }
);
```

## Performance Optimization {#performance}

### Caching Strategies

```typescript
// lib/cache.ts
import { GetServerSidePropsContext } from 'next';

interface CacheOptions {
  maxAge?: number;
  staleWhileRevalidate?: number;
  mustRevalidate?: boolean;
}

export function setCacheHeaders(
  context: GetServerSidePropsContext,
  options: CacheOptions = {}
) {
  const {
    maxAge = 60, // 1 minute default
    staleWhileRevalidate = 300, // 5 minutes default
    mustRevalidate = false
  } = options;

  const cacheControl = [
    `max-age=${maxAge}`,
    `s-maxage=${maxAge}`,
    `stale-while-revalidate=${staleWhileRevalidate}`,
    mustRevalidate ? 'must-revalidate' : '',
  ].filter(Boolean).join(', ');

  context.res.setHeader('Cache-Control', cacheControl);
}

// Usage in getServerSideProps
export const getServerSideProps: GetServerSideProps = async (context) => {
  // Set cache headers for 5 minutes with 1 hour stale-while-revalidate
  setCacheHeaders(context, {
    maxAge: 300,
    staleWhileRevalidate: 3600,
  });

  // Your data fetching logic here
  const data = await fetchData();

  return {
    props: { data },
  };
};
```

### Database Connection Optimization

```typescript
// lib/database.ts
import { Pool } from 'pg';

// Use connection pooling for database connections
const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
  max: 20, // Maximum number of connections
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 2000,
});

export async function queryDatabase<T>(
  query: string,
  params: any[] = []
): Promise<T[]> {
  const client = await pool.connect();
  try {
    const result = await client.query(query, params);
    return result.rows;
  } finally {
    client.release();
  }
}

// Example usage in getServerSideProps
export const getServerSideProps: GetServerSideProps = async (context) => {
  try {
    const userId = context.query.id as string;
    
    // Use parameterized queries for security
    const userData = await queryDatabase(
      'SELECT id, name, email, avatar FROM users WHERE id = $1',
      [userId]
    );

    if (userData.length === 0) {
      return { notFound: true };
    }

    return {
      props: {
        user: userData[0],
      },
    };
  } catch (error) {
    console.error('Database error:', error);
    return {
      props: {
        user: null,
      },
    };
  }
};
```

## Best Practices {#best-practices}

### 1. Error Handling and Fallbacks

```typescript
// utils/serverSideHelpers.ts
export async function safeServerSideProps<T>(
  dataFetcher: () => Promise<T>,
  fallbackData: T,
  options: {
    notFoundOnError?: boolean;
    redirectOnError?: string;
    cacheOnError?: boolean;
  } = {}
) {
  try {
    const data = await dataFetcher();
    return {
      props: data,
    };
  } catch (error) {
    console.error('Server-side props error:', error);
    
    if (options.notFoundOnError) {
      return { notFound: true };
    }
    
    if (options.redirectOnError) {
      return {
        redirect: {
          destination: options.redirectOnError,
          permanent: false,
        },
      };
    }
    
    return {
      props: fallbackData,
    };
  }
}
```

### 2. Request Optimization

```typescript
// lib/apiClient.ts
class APIClient {
  private baseURL: string;
  private defaultHeaders: Record<string, string>;

  constructor() {
    this.baseURL = process.env.API_BASE_URL!;
    this.defaultHeaders = {
      'Content-Type': 'application/json',
      'User-Agent': 'NextJS-SSR/1.0',
    };
  }

  async fetch<T>(
    endpoint: string,
    options: RequestInit & { timeout?: number } = {}
  ): Promise<T> {
    const { timeout = 5000, ...requestOptions } = options;
    
    const controller = new AbortController();
    const timeoutId = setTimeout(() => controller.abort(), timeout);

    try {
      const response = await fetch(`${this.baseURL}${endpoint}`, {
        ...requestOptions,
        headers: {
          ...this.defaultHeaders,
          ...requestOptions.headers,
        },
        signal: controller.signal,
      });

      clearTimeout(timeoutId);

      if (!response.ok) {
        throw new Error(`API Error: ${response.status} ${response.statusText}`);
      }

      return await response.json();
    } catch (error) {
      clearTimeout(timeoutId);
      throw error;
    }
  }

  async fetchMultiple<T>(endpoints: string[]): Promise<T[]> {
    const promises = endpoints.map(endpoint => this.fetch<T>(endpoint));
    return Promise.all(promises);
  }
}

export const apiClient = new APIClient();
```

## Exercises

### Exercise 1: Real-time Dashboard
Create a user dashboard with real-time data:
- User statistics and metrics
- Recent activities and notifications
- Live data updates
- Responsive design for mobile and desktop

### Exercise 2: E-commerce Product Page
Build a dynamic product page with:
- Product details with real-time inventory
- User reviews and ratings
- Personalized recommendations
- Recently viewed products
- Price comparison with competitors

### Exercise 3: Social Media Feed
Create a personalized social media feed:
- User-specific content based on follows
- Real-time posts and interactions
- Infinite scroll pagination
- Content filtering and preferences

### Exercise 4: Multi-tenant Application
Build a multi-tenant app with:
- Tenant-specific data and theming
- Role-based access control
- Dynamic subdomain routing
- Tenant-specific configurations

## Summary

Server-Side Rendering in Next.js provides:

- **Fresh Data**: Always up-to-date content on every request
- **SEO Benefits**: Complete HTML for search engines
- **Authentication**: Secure server-side user verification
- **Personalization**: User-specific content and experiences

Key considerations:
- Use SSR for dynamic, user-specific content
- Implement proper caching strategies
- Optimize database connections and API calls
- Handle errors gracefully with fallbacks
- Consider performance implications

Next, we'll explore **Incremental Static Regeneration (ISR)** and learn how to combine the benefits of SSG and SSR.
