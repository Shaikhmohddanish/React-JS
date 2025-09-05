# Incremental Static Regeneration (ISR) in Next.js

## Table of Contents
1. [Introduction to ISR](#introduction)
2. [Basic ISR Implementation](#basic-isr)
3. [On-Demand Revalidation](#on-demand)
4. [ISR with Dynamic Routes](#dynamic-routes)
5. [Advanced ISR Patterns](#advanced-patterns)
6. [Performance Optimization](#performance)
7. [Monitoring and Analytics](#monitoring)
8. [Best Practices](#best-practices)
9. [Real-world Use Cases](#use-cases)
10. [Troubleshooting](#troubleshooting)

## Introduction to ISR {#introduction}

Incremental Static Regeneration (ISR) allows you to update static content after you've built your site, without rebuilding the entire application. It combines the benefits of Static Site Generation (SSG) and Server-Side Rendering (SSR).

### How ISR Works

```typescript
interface ISRWorkflow {
  step: number;
  description: string;
  timing: string;
}

const isrWorkflow: ISRWorkflow[] = [
  {
    step: 1,
    description: 'User requests a page',
    timing: 'Request time'
  },
  {
    step: 2,
    description: 'Serve cached static page immediately',
    timing: '< 100ms'
  },
  {
    step: 3,
    description: 'Check if page needs revalidation',
    timing: 'Background'
  },
  {
    step: 4,
    description: 'Regenerate page with fresh data',
    timing: 'Background'
  },
  {
    step: 5,
    description: 'Update cache with new version',
    timing: 'Background'
  },
  {
    step: 6,
    description: 'Subsequent requests get updated page',
    timing: 'Next request'
  }
];
```

### Benefits of ISR

```typescript
const isrBenefits = {
  performance: [
    'Fast initial page loads (static)',
    'No build time delays for updates',
    'CDN-friendly caching',
    'Reduced server load'
  ],
  scalability: [
    'Handle millions of pages',
    'Incremental updates only',
    'No need to rebuild entire site',
    'Efficient resource usage'
  ],
  freshness: [
    'Always serve the latest content eventually',
    'Stale-while-revalidate strategy',
    'Background regeneration',
    'Configurable update frequency'
  ],
  development: [
    'Simple implementation',
    'No complex caching logic needed',
    'Works with existing SSG code',
    'Easy to debug and monitor'
  ]
};
```

## Basic ISR Implementation {#basic-isr}

### Simple ISR with Time-based Revalidation

```typescript
// pages/blog/index.tsx
import { GetStaticProps } from 'next';

interface BlogPost {
  id: string;
  title: string;
  excerpt: string;
  content: string;
  author: {
    name: string;
    avatar: string;
  };
  publishedAt: string;
  updatedAt: string;
  slug: string;
  tags: string[];
  readingTime: number;
  viewCount: number;
}

interface BlogPageProps {
  posts: BlogPost[];
  featuredPost: BlogPost;
  totalPosts: number;
  lastUpdated: string;
}

export default function BlogPage({ posts, featuredPost, totalPosts, lastUpdated }: BlogPageProps) {
  return (
    <div className="container mx-auto px-4 py-8">
      {/* Page Header */}
      <header className="text-center mb-12">
        <h1 className="text-4xl font-bold text-gray-900 mb-4">
          Our Blog
        </h1>
        <p className="text-xl text-gray-600 mb-2">
          Discover insights, tutorials, and industry news
        </p>
        <p className="text-sm text-gray-500">
          {totalPosts} articles • Last updated: {new Date(lastUpdated).toLocaleString()}
        </p>
      </header>

      {/* Featured Post */}
      {featuredPost && (
        <section className="mb-12">
          <h2 className="text-2xl font-bold text-gray-900 mb-6">Featured Article</h2>
          <div className="bg-gradient-to-r from-blue-500 to-purple-600 rounded-xl overflow-hidden shadow-lg">
            <div className="p-8 text-white">
              <div className="flex items-center mb-4">
                <img 
                  src={featuredPost.author.avatar} 
                  alt={featuredPost.author.name}
                  className="w-10 h-10 rounded-full mr-3"
                />
                <div>
                  <p className="font-medium">{featuredPost.author.name}</p>
                  <p className="text-sm opacity-90">
                    {new Date(featuredPost.publishedAt).toLocaleDateString()} • 
                    {featuredPost.readingTime} min read
                  </p>
                </div>
              </div>
              <h3 className="text-2xl font-bold mb-3">{featuredPost.title}</h3>
              <p className="text-lg opacity-90 mb-4">{featuredPost.excerpt}</p>
              <div className="flex items-center justify-between">
                <div className="flex flex-wrap gap-2">
                  {featuredPost.tags.slice(0, 3).map((tag) => (
                    <span 
                      key={tag}
                      className="px-3 py-1 bg-white bg-opacity-20 rounded-full text-sm"
                    >
                      {tag}
                    </span>
                  ))}
                </div>
                <a 
                  href={`/blog/${featuredPost.slug}`}
                  className="px-6 py-3 bg-white text-blue-600 rounded-lg font-medium hover:bg-gray-100 transition-colors"
                >
                  Read Article
                </a>
              </div>
            </div>
          </div>
        </section>
      )}

      {/* Recent Posts Grid */}
      <section>
        <h2 className="text-2xl font-bold text-gray-900 mb-6">Recent Articles</h2>
        <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-8">
          {posts.map((post) => (
            <article key={post.id} className="bg-white rounded-lg shadow-md overflow-hidden hover:shadow-lg transition-shadow">
              <div className="p-6">
                <div className="flex items-center mb-3">
                  <img 
                    src={post.author.avatar} 
                    alt={post.author.name}
                    className="w-8 h-8 rounded-full mr-2"
                  />
                  <div className="text-sm text-gray-600">
                    <p className="font-medium">{post.author.name}</p>
                    <p>{new Date(post.publishedAt).toLocaleDateString()}</p>
                  </div>
                </div>
                
                <h3 className="text-xl font-semibold text-gray-900 mb-3 line-clamp-2">
                  {post.title}
                </h3>
                
                <p className="text-gray-600 mb-4 line-clamp-3">
                  {post.excerpt}
                </p>
                
                <div className="flex items-center justify-between text-sm text-gray-500 mb-4">
                  <span>{post.readingTime} min read</span>
                  <span>{post.viewCount.toLocaleString()} views</span>
                </div>
                
                <div className="flex flex-wrap gap-2 mb-4">
                  {post.tags.slice(0, 2).map((tag) => (
                    <span 
                      key={tag}
                      className="px-2 py-1 bg-gray-100 text-gray-700 rounded text-xs"
                    >
                      {tag}
                    </span>
                  ))}
                </div>
                
                <a 
                  href={`/blog/${post.slug}`}
                  className="text-blue-600 hover:text-blue-800 font-medium"
                >
                  Read more →
                </a>
              </div>
            </article>
          ))}
        </div>
      </section>

      {/* Load More */}
      <div className="text-center mt-12">
        <button className="px-8 py-3 bg-blue-600 text-white rounded-lg hover:bg-blue-700 transition-colors">
          Load More Articles
        </button>
      </div>
    </div>
  );
}

export const getStaticProps: GetStaticProps<BlogPageProps> = async () => {
  try {
    // Fetch blog posts from your data source
    const [postsResponse, featuredResponse] = await Promise.all([
      fetch(`${process.env.API_BASE_URL}/posts?limit=12&sort=publishedAt:desc`),
      fetch(`${process.env.API_BASE_URL}/posts/featured`)
    ]);

    if (!postsResponse.ok || !featuredResponse.ok) {
      throw new Error('Failed to fetch blog data');
    }

    const postsData = await postsResponse.json();
    const featuredData = await featuredResponse.json();

    return {
      props: {
        posts: postsData.posts,
        featuredPost: featuredData,
        totalPosts: postsData.total,
        lastUpdated: new Date().toISOString(),
      },
      // Revalidate every 5 minutes (300 seconds)
      revalidate: 300,
    };
  } catch (error) {
    console.error('Error fetching blog data:', error);
    
    // Return empty data on error, will retry on next request
    return {
      props: {
        posts: [],
        featuredPost: null,
        totalPosts: 0,
        lastUpdated: new Date().toISOString(),
      },
      // Retry more frequently on error
      revalidate: 60,
    };
  }
};
```

### ISR with Dynamic Content

```typescript
// pages/products/[category].tsx
import { GetStaticPaths, GetStaticProps } from 'next';

interface Product {
  id: string;
  name: string;
  description: string;
  price: number;
  originalPrice?: number;
  image: string;
  rating: number;
  reviewCount: number;
  inStock: boolean;
  slug: string;
  specifications: Record<string, string>;
}

interface Category {
  id: string;
  name: string;
  slug: string;
  description: string;
  image: string;
  productCount: number;
}

interface CategoryPageProps {
  category: Category;
  products: Product[];
  filters: {
    priceRanges: { min: number; max: number; count: number }[];
    brands: { name: string; count: number }[];
    ratings: { rating: number; count: number }[];
  };
  totalProducts: number;
  lastUpdated: string;
}

export default function CategoryPage({ 
  category, 
  products, 
  filters, 
  totalProducts,
  lastUpdated 
}: CategoryPageProps) {
  return (
    <div className="container mx-auto px-4 py-8">
      {/* Category Header */}
      <header className="relative mb-12">
        <div className="h-48 rounded-xl overflow-hidden mb-6">
          <img 
            src={category.image} 
            alt={category.name}
            className="w-full h-full object-cover"
          />
          <div className="absolute inset-0 bg-black bg-opacity-40 flex items-center justify-center">
            <div className="text-center text-white">
              <h1 className="text-4xl font-bold mb-2">{category.name}</h1>
              <p className="text-xl">{category.description}</p>
              <p className="text-sm mt-2 opacity-90">
                {totalProducts} products • Updated {new Date(lastUpdated).toLocaleTimeString()}
              </p>
            </div>
          </div>
        </div>
      </header>

      <div className="flex flex-col lg:flex-row gap-8">
        {/* Filters Sidebar */}
        <aside className="lg:w-1/4">
          <div className="bg-white rounded-lg shadow-md p-6 sticky top-4">
            <h2 className="text-lg font-semibold mb-6">Filters</h2>
            
            {/* Price Range Filter */}
            <div className="mb-6">
              <h3 className="font-medium mb-3">Price Range</h3>
              <div className="space-y-2">
                {filters.priceRanges.map((range, index) => (
                  <label key={index} className="flex items-center">
                    <input type="checkbox" className="mr-2" />
                    <span className="text-sm">
                      ${range.min} - ${range.max} ({range.count})
                    </span>
                  </label>
                ))}
              </div>
            </div>

            {/* Brand Filter */}
            <div className="mb-6">
              <h3 className="font-medium mb-3">Brands</h3>
              <div className="space-y-2 max-h-40 overflow-y-auto">
                {filters.brands.map((brand) => (
                  <label key={brand.name} className="flex items-center">
                    <input type="checkbox" className="mr-2" />
                    <span className="text-sm">
                      {brand.name} ({brand.count})
                    </span>
                  </label>
                ))}
              </div>
            </div>

            {/* Rating Filter */}
            <div>
              <h3 className="font-medium mb-3">Customer Rating</h3>
              <div className="space-y-2">
                {filters.ratings.map((ratingFilter) => (
                  <label key={ratingFilter.rating} className="flex items-center">
                    <input type="checkbox" className="mr-2" />
                    <span className="text-sm flex items-center">
                      <span className="flex text-yellow-500 mr-1">
                        {Array.from({ length: ratingFilter.rating }).map((_, i) => (
                          <span key={i}>★</span>
                        ))}
                      </span>
                      & up ({ratingFilter.count})
                    </span>
                  </label>
                ))}
              </div>
            </div>
          </div>
        </aside>

        {/* Products Grid */}
        <main className="lg:w-3/4">
          {/* Sort Options */}
          <div className="flex justify-between items-center mb-6">
            <p className="text-gray-600">
              Showing {products.length} of {totalProducts} products
            </p>
            <select className="px-3 py-2 border rounded text-sm">
              <option value="featured">Featured</option>
              <option value="price-asc">Price: Low to High</option>
              <option value="price-desc">Price: High to Low</option>
              <option value="rating">Customer Rating</option>
              <option value="newest">Newest</option>
            </select>
          </div>

          {/* Products Grid */}
          <div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-6">
            {products.map((product) => (
              <div key={product.id} className="bg-white rounded-lg shadow-md overflow-hidden hover:shadow-lg transition-shadow">
                <div className="relative">
                  <img 
                    src={product.image} 
                    alt={product.name}
                    className="w-full h-48 object-cover"
                  />
                  {product.originalPrice && product.originalPrice > product.price && (
                    <span className="absolute top-2 left-2 bg-red-500 text-white px-2 py-1 text-xs rounded">
                      SALE
                    </span>
                  )}
                  {!product.inStock && (
                    <div className="absolute inset-0 bg-black bg-opacity-50 flex items-center justify-center">
                      <span className="text-white font-semibold">Out of Stock</span>
                    </div>
                  )}
                </div>
                
                <div className="p-4">
                  <h3 className="font-semibold text-lg mb-2 line-clamp-2">
                    {product.name}
                  </h3>
                  
                  <p className="text-gray-600 text-sm mb-3 line-clamp-2">
                    {product.description}
                  </p>
                  
                  <div className="flex items-center mb-3">
                    <span className="flex text-yellow-500 mr-2">
                      {Array.from({ length: 5 }).map((_, i) => (
                        <span key={i}>
                          {i < Math.floor(product.rating) ? '★' : '☆'}
                        </span>
                      ))}
                    </span>
                    <span className="text-sm text-gray-600">
                      ({product.reviewCount})
                    </span>
                  </div>
                  
                  <div className="flex items-center justify-between">
                    <div>
                      <span className="text-xl font-bold text-green-600">
                        ${product.price.toFixed(2)}
                      </span>
                      {product.originalPrice && product.originalPrice > product.price && (
                        <span className="text-sm text-gray-500 line-through ml-2">
                          ${product.originalPrice.toFixed(2)}
                        </span>
                      )}
                    </div>
                    <button 
                      className={`px-4 py-2 rounded-lg text-sm font-medium ${
                        product.inStock 
                          ? 'bg-blue-600 text-white hover:bg-blue-700' 
                          : 'bg-gray-300 text-gray-500 cursor-not-allowed'
                      }`}
                      disabled={!product.inStock}
                    >
                      {product.inStock ? 'Add to Cart' : 'Out of Stock'}
                    </button>
                  </div>
                </div>
              </div>
            ))}
          </div>

          {/* Load More */}
          {products.length < totalProducts && (
            <div className="text-center mt-8">
              <button className="px-8 py-3 bg-blue-600 text-white rounded-lg hover:bg-blue-700 transition-colors">
                Load More Products
              </button>
            </div>
          )}
        </main>
      </div>
    </div>
  );
}

export const getStaticPaths: GetStaticPaths = async () => {
  try {
    // Fetch all categories for static generation
    const response = await fetch(`${process.env.API_BASE_URL}/categories`);
    const categories: Category[] = await response.json();
    
    // Generate paths for top categories only
    const topCategories = categories.filter(cat => cat.productCount > 10);
    const paths = topCategories.map((category) => ({
      params: { category: category.slug }
    }));
    
    return {
      paths,
      fallback: 'blocking', // Generate other categories on-demand
    };
  } catch (error) {
    console.error('Error generating category paths:', error);
    return {
      paths: [],
      fallback: 'blocking',
    };
  }
};

export const getStaticProps: GetStaticProps<CategoryPageProps> = async ({ params }) => {
  try {
    const categorySlug = params?.category as string;
    
    // Fetch category data and products in parallel
    const [categoryResponse, productsResponse, filtersResponse] = await Promise.all([
      fetch(`${process.env.API_BASE_URL}/categories/${categorySlug}`),
      fetch(`${process.env.API_BASE_URL}/categories/${categorySlug}/products?limit=24&sort=featured`),
      fetch(`${process.env.API_BASE_URL}/categories/${categorySlug}/filters`)
    ]);
    
    if (!categoryResponse.ok) {
      return { notFound: true };
    }
    
    const [category, productsData, filters] = await Promise.all([
      categoryResponse.json(),
      productsResponse.json(),
      filtersResponse.json()
    ]);
    
    return {
      props: {
        category,
        products: productsData.products,
        filters,
        totalProducts: productsData.total,
        lastUpdated: new Date().toISOString(),
      },
      // Revalidate every 10 minutes for product availability and pricing
      revalidate: 600,
    };
  } catch (error) {
    console.error('Error fetching category data:', error);
    return { notFound: true };
  }
};
```

## On-Demand Revalidation {#on-demand}

### API Route for On-Demand Revalidation

```typescript
// pages/api/revalidate.ts
import { NextApiRequest, NextApiResponse } from 'next';

interface RevalidationRequest {
  paths: string[];
  secret?: string;
  priority?: 'high' | 'normal' | 'low';
}

interface RevalidationResponse {
  revalidated: boolean;
  message: string;
  timestamp: string;
  paths?: string[];
  errors?: string[];
}

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse<RevalidationResponse>
) {
  // Only allow POST requests
  if (req.method !== 'POST') {
    return res.status(405).json({
      revalidated: false,
      message: 'Method not allowed',
      timestamp: new Date().toISOString(),
    });
  }

  try {
    const { paths, secret, priority = 'normal' }: RevalidationRequest = req.body;

    // Verify the secret token
    if (secret !== process.env.REVALIDATION_SECRET) {
      return res.status(401).json({
        revalidated: false,
        message: 'Invalid secret token',
        timestamp: new Date().toISOString(),
      });
    }

    // Validate paths
    if (!paths || !Array.isArray(paths) || paths.length === 0) {
      return res.status(400).json({
        revalidated: false,
        message: 'Paths array is required and must not be empty',
        timestamp: new Date().toISOString(),
      });
    }

    // Validate path format
    const invalidPaths = paths.filter(path => !path.startsWith('/'));
    if (invalidPaths.length > 0) {
      return res.status(400).json({
        revalidated: false,
        message: `Invalid paths (must start with /): ${invalidPaths.join(', ')}`,
        timestamp: new Date().toISOString(),
      });
    }

    console.log(`[Revalidation] Starting revalidation for ${paths.length} paths with priority: ${priority}`);

    // Revalidate paths based on priority
    const results = await Promise.allSettled(
      paths.map(async (path) => {
        try {
          await res.revalidate(path);
          console.log(`[Revalidation] Successfully revalidated: ${path}`);
          return { path, success: true };
        } catch (error) {
          console.error(`[Revalidation] Failed to revalidate ${path}:`, error);
          return { path, success: false, error: error.message };
        }
      })
    );

    // Collect results
    const successful = results.filter(r => r.status === 'fulfilled' && r.value.success)
                            .map(r => (r as PromiseFulfilledResult<any>).value.path);
    const failed = results.filter(r => r.status === 'rejected' || !r.value?.success)
                          .map(r => r.status === 'rejected' ? r.reason : r.value?.error);

    // Log results
    console.log(`[Revalidation] Completed: ${successful.length} successful, ${failed.length} failed`);

    return res.status(200).json({
      revalidated: true,
      message: `Revalidated ${successful.length} of ${paths.length} paths`,
      timestamp: new Date().toISOString(),
      paths: successful,
      errors: failed.length > 0 ? failed : undefined,
    });

  } catch (error) {
    console.error('[Revalidation] Unexpected error:', error);
    
    return res.status(500).json({
      revalidated: false,
      message: 'Internal server error during revalidation',
      timestamp: new Date().toISOString(),
    });
  }
}
```

### Webhook Integration for Content Updates

```typescript
// pages/api/webhooks/content-updated.ts
import { NextApiRequest, NextApiResponse } from 'next';
import crypto from 'crypto';

interface ContentUpdateWebhook {
  type: 'blog' | 'product' | 'category' | 'page';
  action: 'created' | 'updated' | 'deleted' | 'published';
  data: {
    id: string;
    slug: string;
    category?: string;
    tags?: string[];
  };
  timestamp: string;
}

function verifyWebhookSignature(payload: string, signature: string): boolean {
  const secret = process.env.WEBHOOK_SECRET!;
  const expectedSignature = crypto
    .createHmac('sha256', secret)
    .update(payload)
    .digest('hex');
  
  return crypto.timingSafeEqual(
    Buffer.from(signature),
    Buffer.from(expectedSignature)
  );
}

async function triggerRevalidation(paths: string[]): Promise<void> {
  try {
    const response = await fetch(`${process.env.NEXTAUTH_URL}/api/revalidate`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        paths,
        secret: process.env.REVALIDATION_SECRET,
        priority: 'high',
      }),
    });

    if (!response.ok) {
      throw new Error(`Revalidation failed: ${response.statusText}`);
    }

    const result = await response.json();
    console.log('[Webhook] Revalidation triggered:', result);
  } catch (error) {
    console.error('[Webhook] Revalidation error:', error);
    throw error;
  }
}

export default async function handler(req: NextApiRequest, res: NextApiResponse) {
  if (req.method !== 'POST') {
    return res.status(405).json({ message: 'Method not allowed' });
  }

  try {
    // Verify webhook signature
    const signature = req.headers['x-signature'] as string;
    const payload = JSON.stringify(req.body);
    
    if (!signature || !verifyWebhookSignature(payload, signature)) {
      console.warn('[Webhook] Invalid signature');
      return res.status(401).json({ message: 'Invalid signature' });
    }

    const webhook: ContentUpdateWebhook = req.body;
    console.log(`[Webhook] Received ${webhook.type} ${webhook.action}:`, webhook.data);

    // Determine paths to revalidate based on content type and action
    const pathsToRevalidate: string[] = [];

    switch (webhook.type) {
      case 'blog':
        pathsToRevalidate.push('/blog');
        if (webhook.action !== 'deleted') {
          pathsToRevalidate.push(`/blog/${webhook.data.slug}`);
        }
        // Revalidate tag pages
        if (webhook.data.tags) {
          webhook.data.tags.forEach(tag => {
            pathsToRevalidate.push(`/blog/tag/${tag}`);
          });
        }
        break;

      case 'product':
        pathsToRevalidate.push('/products');
        if (webhook.data.category) {
          pathsToRevalidate.push(`/products/${webhook.data.category}`);
        }
        if (webhook.action !== 'deleted') {
          pathsToRevalidate.push(`/products/${webhook.data.category}/${webhook.data.slug}`);
        }
        break;

      case 'category':
        pathsToRevalidate.push('/products');
        if (webhook.action !== 'deleted') {
          pathsToRevalidate.push(`/products/${webhook.data.slug}`);
        }
        break;

      case 'page':
        if (webhook.action !== 'deleted') {
          pathsToRevalidate.push(`/${webhook.data.slug}`);
        }
        break;
    }

    // Add homepage for significant updates
    if (['created', 'published'].includes(webhook.action)) {
      pathsToRevalidate.push('/');
    }

    // Remove duplicates
    const uniquePaths = [...new Set(pathsToRevalidate)];

    if (uniquePaths.length > 0) {
      await triggerRevalidation(uniquePaths);
      
      return res.status(200).json({
        message: 'Webhook processed successfully',
        revalidatedPaths: uniquePaths,
        timestamp: new Date().toISOString(),
      });
    } else {
      return res.status(200).json({
        message: 'No revalidation needed',
        timestamp: new Date().toISOString(),
      });
    }

  } catch (error) {
    console.error('[Webhook] Processing error:', error);
    
    return res.status(500).json({
      message: 'Webhook processing failed',
      error: error.message,
      timestamp: new Date().toISOString(),
    });
  }
}
```

### Client-side Utility for Manual Revalidation

```typescript
// utils/revalidation.ts
interface RevalidationOptions {
  paths: string[];
  priority?: 'high' | 'normal' | 'low';
  onSuccess?: (result: any) => void;
  onError?: (error: Error) => void;
}

export class RevalidationManager {
  private baseUrl: string;
  private secret: string;

  constructor() {
    this.baseUrl = process.env.NEXT_PUBLIC_APP_URL || '';
    this.secret = process.env.REVALIDATION_SECRET || '';
  }

  async revalidatePaths(options: RevalidationOptions): Promise<any> {
    try {
      const response = await fetch(`${this.baseUrl}/api/revalidate`, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({
          paths: options.paths,
          secret: this.secret,
          priority: options.priority || 'normal',
        }),
      });

      if (!response.ok) {
        throw new Error(`Revalidation failed: ${response.statusText}`);
      }

      const result = await response.json();
      
      if (options.onSuccess) {
        options.onSuccess(result);
      }

      return result;
    } catch (error) {
      if (options.onError) {
        options.onError(error as Error);
      }
      throw error;
    }
  }

  // Convenience methods for common revalidation scenarios
  async revalidateBlog(slug?: string): Promise<any> {
    const paths = ['/blog'];
    if (slug) {
      paths.push(`/blog/${slug}`);
    }
    
    return this.revalidatePaths({ paths });
  }

  async revalidateProduct(category: string, slug?: string): Promise<any> {
    const paths = ['/products', `/products/${category}`];
    if (slug) {
      paths.push(`/products/${category}/${slug}`);
    }
    
    return this.revalidatePaths({ paths });
  }

  async revalidateHomepage(): Promise<any> {
    return this.revalidatePaths({ paths: ['/'] });
  }

  // Batch revalidation with rate limiting
  async batchRevalidate(batches: string[][], delayMs = 1000): Promise<any[]> {
    const results = [];
    
    for (let i = 0; i < batches.length; i++) {
      const batch = batches[i];
      
      try {
        const result = await this.revalidatePaths({ paths: batch });
        results.push(result);
        
        // Add delay between batches to avoid overwhelming the server
        if (i < batches.length - 1) {
          await new Promise(resolve => setTimeout(resolve, delayMs));
        }
      } catch (error) {
        console.error(`Batch ${i + 1} failed:`, error);
        results.push({ error: error.message });
      }
    }
    
    return results;
  }
}

export const revalidationManager = new RevalidationManager();

// React hook for revalidation in components
import { useState, useCallback } from 'react';

export function useRevalidation() {
  const [isRevalidating, setIsRevalidating] = useState(false);
  const [error, setError] = useState<string | null>(null);

  const revalidate = useCallback(async (options: RevalidationOptions) => {
    setIsRevalidating(true);
    setError(null);

    try {
      const result = await revalidationManager.revalidatePaths(options);
      setIsRevalidating(false);
      return result;
    } catch (err) {
      const errorMessage = err instanceof Error ? err.message : 'Revalidation failed';
      setError(errorMessage);
      setIsRevalidating(false);
      throw err;
    }
  }, []);

  return {
    revalidate,
    isRevalidating,
    error,
  };
}
```

### Admin Interface for Content Management

```typescript
// components/AdminRevalidation.tsx
import { useState, useCallback } from 'react';
import { useRevalidation } from '../utils/revalidation';

interface RevalidationItem {
  id: string;
  label: string;
  paths: string[];
  description: string;
}

const COMMON_REVALIDATIONS: RevalidationItem[] = [
  {
    id: 'homepage',
    label: 'Homepage',
    paths: ['/'],
    description: 'Revalidate the main homepage'
  },
  {
    id: 'blog-index',
    label: 'Blog Index',
    paths: ['/blog'],
    description: 'Revalidate the blog listing page'
  },
  {
    id: 'products-index',
    label: 'Products Index',
    paths: ['/products'],
    description: 'Revalidate the products listing page'
  },
  {
    id: 'all-categories',
    label: 'All Product Categories',
    paths: ['/products/electronics', '/products/clothing', '/products/books'],
    description: 'Revalidate all main product category pages'
  }
];

export default function AdminRevalidation() {
  const [customPaths, setCustomPaths] = useState('');
  const [selectedItems, setSelectedItems] = useState<string[]>([]);
  const [results, setResults] = useState<any[]>([]);
  
  const { revalidate, isRevalidating, error } = useRevalidation();

  const handleQuickRevalidation = useCallback(async (item: RevalidationItem) => {
    try {
      const result = await revalidate({
        paths: item.paths,
        priority: 'high',
        onSuccess: (result) => {
          setResults(prev => [
            { ...result, label: item.label, timestamp: new Date() },
            ...prev.slice(0, 9) // Keep last 10 results
          ]);
        }
      });
    } catch (err) {
      console.error('Revalidation failed:', err);
    }
  }, [revalidate]);

  const handleBatchRevalidation = useCallback(async () => {
    const selectedRevalidations = COMMON_REVALIDATIONS.filter(
      item => selectedItems.includes(item.id)
    );

    for (const item of selectedRevalidations) {
      await handleQuickRevalidation(item);
    }

    setSelectedItems([]);
  }, [selectedItems, handleQuickRevalidation]);

  const handleCustomRevalidation = useCallback(async () => {
    const paths = customPaths
      .split('\n')
      .map(path => path.trim())
      .filter(path => path.length > 0 && path.startsWith('/'));

    if (paths.length === 0) {
      alert('Please enter valid paths (must start with /)');
      return;
    }

    try {
      const result = await revalidate({
        paths,
        priority: 'normal',
        onSuccess: (result) => {
          setResults(prev => [
            { ...result, label: 'Custom Paths', timestamp: new Date() },
            ...prev.slice(0, 9)
          ]);
          setCustomPaths('');
        }
      });
    } catch (err) {
      console.error('Custom revalidation failed:', err);
    }
  }, [customPaths, revalidate]);

  return (
    <div className="bg-white rounded-lg shadow-md p-6">
      <h2 className="text-2xl font-bold text-gray-900 mb-6">
        Page Revalidation Manager
      </h2>

      {/* Quick Actions */}
      <section className="mb-8">
        <h3 className="text-lg font-semibold text-gray-700 mb-4">Quick Actions</h3>
        <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
          {COMMON_REVALIDATIONS.map((item) => (
            <div key={item.id} className="border border-gray-200 rounded-lg p-4">
              <div className="flex items-start justify-between mb-2">
                <label className="flex items-center">
                  <input
                    type="checkbox"
                    checked={selectedItems.includes(item.id)}
                    onChange={(e) => {
                      if (e.target.checked) {
                        setSelectedItems(prev => [...prev, item.id]);
                      } else {
                        setSelectedItems(prev => prev.filter(id => id !== item.id));
                      }
                    }}
                    className="mr-2"
                  />
                  <span className="font-medium">{item.label}</span>
                </label>
                <button
                  onClick={() => handleQuickRevalidation(item)}
                  disabled={isRevalidating}
                  className="px-3 py-1 bg-blue-600 text-white text-sm rounded hover:bg-blue-700 disabled:opacity-50"
                >
                  Revalidate
                </button>
              </div>
              <p className="text-sm text-gray-600">{item.description}</p>
              <p className="text-xs text-gray-500 mt-1">
                Paths: {item.paths.join(', ')}
              </p>
            </div>
          ))}
        </div>

        {selectedItems.length > 0 && (
          <div className="mt-4">
            <button
              onClick={handleBatchRevalidation}
              disabled={isRevalidating}
              className="px-6 py-2 bg-green-600 text-white rounded-lg hover:bg-green-700 disabled:opacity-50"
            >
              Revalidate Selected ({selectedItems.length})
            </button>
          </div>
        )}
      </section>

      {/* Custom Paths */}
      <section className="mb-8">
        <h3 className="text-lg font-semibold text-gray-700 mb-4">Custom Paths</h3>
        <div className="space-y-4">
          <textarea
            value={customPaths}
            onChange={(e) => setCustomPaths(e.target.value)}
            placeholder="Enter paths to revalidate (one per line)&#10;/blog/my-post&#10;/products/electronics&#10;/about"
            className="w-full h-24 px-3 py-2 border border-gray-300 rounded-lg resize-none"
          />
          <button
            onClick={handleCustomRevalidation}
            disabled={isRevalidating || !customPaths.trim()}
            className="px-6 py-2 bg-purple-600 text-white rounded-lg hover:bg-purple-700 disabled:opacity-50"
          >
            Revalidate Custom Paths
          </button>
        </div>
      </section>

      {/* Status */}
      {isRevalidating && (
        <div className="mb-4 p-3 bg-blue-100 text-blue-800 rounded-lg">
          Revalidation in progress...
        </div>
      )}

      {error && (
        <div className="mb-4 p-3 bg-red-100 text-red-800 rounded-lg">
          Error: {error}
        </div>
      )}

      {/* Results */}
      {results.length > 0 && (
        <section>
          <h3 className="text-lg font-semibold text-gray-700 mb-4">Recent Results</h3>
          <div className="space-y-3 max-h-64 overflow-y-auto">
            {results.map((result, index) => (
              <div key={index} className="bg-gray-50 rounded-lg p-3">
                <div className="flex justify-between items-start mb-2">
                  <span className="font-medium">{result.label}</span>
                  <span className="text-xs text-gray-500">
                    {result.timestamp.toLocaleTimeString()}
                  </span>
                </div>
                <p className="text-sm text-gray-600 mb-1">{result.message}</p>
                {result.paths && (
                  <p className="text-xs text-gray-500">
                    Paths: {result.paths.join(', ')}
                  </p>
                )}
                {result.errors && result.errors.length > 0 && (
                  <p className="text-xs text-red-600 mt-1">
                    Errors: {result.errors.join(', ')}
                  </p>
                )}
              </div>
            ))}
          </div>
        </section>
      )}
    </div>
  );
}
```

## Best Practices {#best-practices}

### 1. Optimal Revalidation Intervals

```typescript
// utils/revalidationStrategy.ts
interface ContentType {
  name: string;
  revalidateInterval: number; // seconds
  priority: 'high' | 'medium' | 'low';
  reasoning: string;
}

const REVALIDATION_STRATEGIES: ContentType[] = [
  {
    name: 'Homepage',
    revalidateInterval: 300, // 5 minutes
    priority: 'high',
    reasoning: 'High traffic, frequently updated content'
  },
  {
    name: 'Blog Posts',
    revalidateInterval: 3600, // 1 hour
    priority: 'medium',
    reasoning: 'Content changes infrequently, good balance'
  },
  {
    name: 'Product Pages',
    revalidateInterval: 1800, // 30 minutes
    priority: 'high',
    reasoning: 'Pricing and inventory can change frequently'
  },
  {
    name: 'Category Pages',
    revalidateInterval: 600, // 10 minutes
    priority: 'medium',
    reasoning: 'New products added regularly'
  },
  {
    name: 'Documentation',
    revalidateInterval: 86400, // 24 hours
    priority: 'low',
    reasoning: 'Stable content, infrequent updates'
  },
  {
    name: 'User Profiles',
    revalidateInterval: 7200, // 2 hours
    priority: 'low',
    reasoning: 'Personal content, moderate update frequency'
  }
];

export function getRevalidationInterval(contentType: string): number {
  const strategy = REVALIDATION_STRATEGIES.find(s => 
    s.name.toLowerCase() === contentType.toLowerCase()
  );
  
  return strategy?.revalidateInterval || 3600; // Default to 1 hour
}
```

### 2. Error Handling and Fallbacks

```typescript
// lib/isrErrorHandling.ts
import { GetStaticProps } from 'next';

export function withISRErrorHandling<P extends Record<string, any>>(
  getStaticPropsFunc: (context: any) => Promise<{ props: P }>,
  fallbackProps: P,
  options: {
    revalidateOnError?: number;
    logError?: boolean;
    notifyOnError?: boolean;
  } = {}
): GetStaticProps<P> {
  return async (context) => {
    const {
      revalidateOnError = 300, // 5 minutes
      logError = true,
      notifyOnError = false
    } = options;

    try {
      const result = await getStaticPropsFunc(context);
      
      // Add success revalidation interval
      return {
        ...result,
        revalidate: getRevalidationInterval('default')
      };
    } catch (error) {
      if (logError) {
        console.error(`ISR Error for ${context.params?.slug || 'unknown'}:`, error);
      }

      if (notifyOnError) {
        // Send error notification (implement based on your monitoring setup)
        await notifyError(error, context);
      }

      // Return fallback data with shorter revalidation
      return {
        props: fallbackProps,
        revalidate: revalidateOnError,
      };
    }
  };
}

async function notifyError(error: any, context: any): Promise<void> {
  // Implement your error notification logic
  // e.g., send to monitoring service, Slack, email, etc.
  try {
    await fetch(process.env.ERROR_WEBHOOK_URL!, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        error: error.message,
        context: context.params,
        timestamp: new Date().toISOString(),
      }),
    });
  } catch (notificationError) {
    console.error('Failed to send error notification:', notificationError);
  }
}
```

### 3. Performance Monitoring

```typescript
// lib/isrMonitoring.ts
interface ISRMetrics {
  pageKey: string;
  revalidationTime: number;
  dataFreshness: number;
  errorRate: number;
  cacheHitRate: number;
}

class ISRMonitor {
  private metrics: Map<string, ISRMetrics> = new Map();

  trackRevalidation(pageKey: string, startTime: number): void {
    const revalidationTime = Date.now() - startTime;
    
    const currentMetrics = this.metrics.get(pageKey) || {
      pageKey,
      revalidationTime: 0,
      dataFreshness: 0,
      errorRate: 0,
      cacheHitRate: 0,
    };

    // Update metrics
    this.metrics.set(pageKey, {
      ...currentMetrics,
      revalidationTime,
    });

    // Log slow revalidations
    if (revalidationTime > 5000) { // 5 seconds
      console.warn(`Slow ISR revalidation for ${pageKey}: ${revalidationTime}ms`);
    }
  }

  trackError(pageKey: string): void {
    const currentMetrics = this.metrics.get(pageKey);
    if (currentMetrics) {
      currentMetrics.errorRate += 1;
    }
  }

  getMetrics(): ISRMetrics[] {
    return Array.from(this.metrics.values());
  }

  exportMetrics(): string {
    const metrics = this.getMetrics();
    return JSON.stringify(metrics, null, 2);
  }
}

export const isrMonitor = new ISRMonitor();

// Enhanced getStaticProps with monitoring
export function withISRMonitoring<P extends Record<string, any>>(
  getStaticPropsFunc: (context: any) => Promise<{ props: P }>,
  pageKey: string
): GetStaticProps<P> {
  return async (context) => {
    const startTime = Date.now();
    
    try {
      const result = await getStaticPropsFunc(context);
      isrMonitor.trackRevalidation(pageKey, startTime);
      return result;
    } catch (error) {
      isrMonitor.trackError(pageKey);
      throw error;
    }
  };
}
```

## Exercises

### Exercise 1: News Website with ISR
Create a news website using ISR:
- Homepage with latest articles (5-minute revalidation)
- Category pages (10-minute revalidation)
- Individual article pages (1-hour revalidation)
- Breaking news banner (1-minute revalidation)
- On-demand revalidation for urgent updates

### Exercise 2: E-commerce with Real-time Inventory
Build an e-commerce site with ISR:
- Product catalog with live pricing (15-minute revalidation)
- Inventory-based revalidation (when stock changes)
- Flash sales with dynamic pricing
- User reviews and ratings updates
- Stock alerts and notifications

### Exercise 3: Blog Platform with Content Management
Create a blog platform with:
- Admin dashboard for content management
- Automatic revalidation on publish
- Tag and category management
- SEO optimization with fresh meta tags
- Analytics tracking for page views

### Exercise 4: Documentation Site with Version Control
Build a documentation platform with:
- Version-controlled content
- Automatic revalidation on documentation updates
- Search functionality with fresh indices
- Multi-language support
- Contributor tracking and statistics

## Summary

Incremental Static Regeneration in Next.js provides:

- **Best of Both Worlds**: Static performance with dynamic freshness
- **Scalability**: Handle large sites without full rebuilds
- **Flexibility**: Mix of static and dynamic content strategies
- **User Experience**: Fast loading with fresh content

Key features:
- Time-based revalidation for automated updates
- On-demand revalidation for immediate content updates
- Fallback strategies for new pages
- Error handling and monitoring capabilities
- Webhook integration for content management systems

Next, we'll explore **API Routes** and learn how to build full-stack applications with Next.js.
