# Static Site Generation (SSG) in Next.js

## Table of Contents
1. [Introduction to SSG](#introduction)
2. [getStaticProps Function](#getstaticprops)
3. [getStaticPaths Function](#getstaticpaths)
4. [Dynamic Routes with SSG](#dynamic-routes)
5. [Incremental Static Regeneration (ISR)](#isr)
6. [Data Sources for SSG](#data-sources)
7. [Optimizing SSG Performance](#performance)
8. [SSG vs SSR vs CSR](#comparison)
9. [Best Practices](#best-practices)
10. [Real-world Examples](#examples)

## Introduction to SSG {#introduction}

Static Site Generation (SSG) is one of Next.js's most powerful features. It pre-renders pages at build time, creating static HTML files that can be served incredibly fast from a CDN.

### Benefits of SSG

```typescript
interface SSGBenefits {
  performance: string[];
  seo: string[];
  reliability: string[];
  cost: string[];
}

const ssgBenefits: SSGBenefits = {
  performance: [
    'Fastest loading times',
    'Minimal server processing',
    'CDN-friendly',
    'Excellent Core Web Vitals'
  ],
  seo: [
    'Pre-rendered HTML for crawlers',
    'Meta tags available immediately',
    'Social media sharing optimized',
    'No JavaScript required for content'
  ],
  reliability: [
    'No server dependencies for content',
    'Works even if backend is down',
    'Resistant to traffic spikes',
    'Simple hosting requirements'
  ],
  cost: [
    'Lower server costs',
    'Efficient bandwidth usage',
    'Reduced API calls',
    'Scalable without infrastructure changes'
  ]
};
```

### When to Use SSG

```typescript
// Ideal use cases for SSG
const ssgUseCases = [
  'Blog posts and articles',
  'Product catalogs',
  'Documentation sites',
  'Marketing pages',
  'Portfolio websites',
  'E-commerce product pages',
  'Company websites',
  'News and magazine sites'
];

// Not ideal for SSG
const ssgNotIdeal = [
  'Highly personalized content',
  'Real-time data displays',
  'User-specific dashboards',
  'Chat applications',
  'Admin panels with frequent updates'
];
```

## getStaticProps Function {#getstaticprops}

`getStaticProps` runs at build time and fetches data needed for pre-rendering.

### Basic getStaticProps Example

```typescript
// pages/blog/index.tsx
import { GetStaticProps } from 'next';

interface BlogPost {
  id: string;
  title: string;
  excerpt: string;
  date: string;
  author: string;
  slug: string;
}

interface BlogPageProps {
  posts: BlogPost[];
  totalPosts: number;
}

export default function BlogPage({ posts, totalPosts }: BlogPageProps) {
  return (
    <div className="container mx-auto px-4 py-8">
      <h1 className="text-3xl font-bold mb-8">Blog Posts ({totalPosts})</h1>
      <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
        {posts.map((post) => (
          <article key={post.id} className="bg-white rounded-lg shadow-md p-6">
            <h2 className="text-xl font-semibold mb-2">{post.title}</h2>
            <p className="text-gray-600 mb-4">{post.excerpt}</p>
            <div className="flex justify-between items-center text-sm text-gray-500">
              <span>By {post.author}</span>
              <span>{new Date(post.date).toLocaleDateString()}</span>
            </div>
            <a 
              href={`/blog/${post.slug}`}
              className="mt-4 inline-block text-blue-600 hover:text-blue-800"
            >
              Read more →
            </a>
          </article>
        ))}
      </div>
    </div>
  );
}

export const getStaticProps: GetStaticProps<BlogPageProps> = async () => {
  try {
    // Fetch data from your data source
    const response = await fetch('https://api.example.com/posts');
    const posts: BlogPost[] = await response.json();
    
    return {
      props: {
        posts: posts.slice(0, 12), // Limit to 12 posts
        totalPosts: posts.length,
      },
      // Revalidate every hour (3600 seconds)
      revalidate: 3600,
    };
  } catch (error) {
    console.error('Failed to fetch posts:', error);
    
    return {
      props: {
        posts: [],
        totalPosts: 0,
      },
      revalidate: 60, // Retry more frequently on error
    };
  }
};
```

### Advanced getStaticProps with Multiple Data Sources

```typescript
// pages/products/index.tsx
import { GetStaticProps } from 'next';

interface Product {
  id: string;
  name: string;
  price: number;
  category: string;
  image: string;
  rating: number;
  inStock: boolean;
}

interface Category {
  id: string;
  name: string;
  slug: string;
  productCount: number;
}

interface ProductsPageProps {
  products: Product[];
  categories: Category[];
  featuredProducts: Product[];
  siteSettings: {
    siteName: string;
    description: string;
  };
}

export default function ProductsPage({ 
  products, 
  categories, 
  featuredProducts,
  siteSettings 
}: ProductsPageProps) {
  return (
    <div>
      <head>
        <title>{siteSettings.siteName} - Products</title>
        <meta name="description" content={siteSettings.description} />
      </head>
      
      <main className="container mx-auto px-4 py-8">
        {/* Featured Products */}
        <section className="mb-12">
          <h2 className="text-2xl font-bold mb-6">Featured Products</h2>
          <div className="grid grid-cols-1 md:grid-cols-3 gap-6">
            {featuredProducts.map((product) => (
              <ProductCard key={product.id} product={product} />
            ))}
          </div>
        </section>
        
        {/* Categories */}
        <section className="mb-12">
          <h2 className="text-2xl font-bold mb-6">Shop by Category</h2>
          <div className="grid grid-cols-2 md:grid-cols-4 gap-4">
            {categories.map((category) => (
              <CategoryCard key={category.id} category={category} />
            ))}
          </div>
        </section>
        
        {/* All Products */}
        <section>
          <h2 className="text-2xl font-bold mb-6">All Products</h2>
          <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6">
            {products.map((product) => (
              <ProductCard key={product.id} product={product} />
            ))}
          </div>
        </section>
      </main>
    </div>
  );
}

export const getStaticProps: GetStaticProps<ProductsPageProps> = async () => {
  try {
    // Fetch multiple data sources in parallel
    const [productsRes, categoriesRes, settingsRes] = await Promise.all([
      fetch('https://api.example.com/products'),
      fetch('https://api.example.com/categories'),
      fetch('https://api.example.com/site-settings')
    ]);
    
    const [products, categories, siteSettings] = await Promise.all([
      productsRes.json(),
      categoriesRes.json(),
      settingsRes.json()
    ]);
    
    // Filter featured products
    const featuredProducts = products.filter((product: Product) => product.rating >= 4.5);
    
    return {
      props: {
        products,
        categories,
        featuredProducts: featuredProducts.slice(0, 3),
        siteSettings,
      },
      revalidate: 1800, // Revalidate every 30 minutes
    };
  } catch (error) {
    console.error('Failed to fetch data:', error);
    
    return {
      notFound: true, // Return 404 page if data fetching fails
    };
  }
};

// Helper components
function ProductCard({ product }: { product: Product }) {
  return (
    <div className="bg-white rounded-lg shadow-md overflow-hidden">
      <img 
        src={product.image} 
        alt={product.name}
        className="w-full h-48 object-cover"
      />
      <div className="p-4">
        <h3 className="font-semibold text-lg mb-2">{product.name}</h3>
        <div className="flex justify-between items-center">
          <span className="text-2xl font-bold text-green-600">
            ${product.price.toFixed(2)}
          </span>
          <div className="flex items-center">
            <span className="text-yellow-500">★</span>
            <span className="ml-1 text-sm text-gray-600">{product.rating}</span>
          </div>
        </div>
        <div className="mt-2">
          {product.inStock ? (
            <span className="text-green-600 text-sm">In Stock</span>
          ) : (
            <span className="text-red-600 text-sm">Out of Stock</span>
          )}
        </div>
      </div>
    </div>
  );
}

function CategoryCard({ category }: { category: Category }) {
  return (
    <a 
      href={`/products/category/${category.slug}`}
      className="block bg-gray-100 rounded-lg p-4 text-center hover:bg-gray-200 transition-colors"
    >
      <h3 className="font-semibold">{category.name}</h3>
      <p className="text-sm text-gray-600">{category.productCount} products</p>
    </a>
  );
}
```

## getStaticPaths Function {#getstaticpaths}

`getStaticPaths` is used with dynamic routes to specify which paths should be pre-rendered.

### Basic getStaticPaths Example

```typescript
// pages/blog/[slug].tsx
import { GetStaticPaths, GetStaticProps } from 'next';

interface BlogPost {
  id: string;
  title: string;
  content: string;
  excerpt: string;
  date: string;
  author: {
    name: string;
    avatar: string;
    bio: string;
  };
  slug: string;
  tags: string[];
  readingTime: number;
}

interface BlogPostPageProps {
  post: BlogPost;
  relatedPosts: BlogPost[];
}

export default function BlogPostPage({ post, relatedPosts }: BlogPostPageProps) {
  return (
    <article className="container mx-auto px-4 py-8 max-w-4xl">
      {/* Article Header */}
      <header className="mb-8">
        <h1 className="text-4xl font-bold mb-4">{post.title}</h1>
        <div className="flex items-center space-x-4 text-gray-600 mb-4">
          <img 
            src={post.author.avatar} 
            alt={post.author.name}
            className="w-10 h-10 rounded-full"
          />
          <div>
            <p className="font-medium">{post.author.name}</p>
            <p className="text-sm">
              {new Date(post.date).toLocaleDateString()} • {post.readingTime} min read
            </p>
          </div>
        </div>
        <div className="flex flex-wrap gap-2">
          {post.tags.map((tag) => (
            <span 
              key={tag}
              className="px-3 py-1 bg-blue-100 text-blue-800 rounded-full text-sm"
            >
              {tag}
            </span>
          ))}
        </div>
      </header>
      
      {/* Article Content */}
      <div 
        className="prose prose-lg max-w-none mb-12"
        dangerouslySetInnerHTML={{ __html: post.content }}
      />
      
      {/* Author Bio */}
      <section className="bg-gray-50 rounded-lg p-6 mb-12">
        <div className="flex items-start space-x-4">
          <img 
            src={post.author.avatar} 
            alt={post.author.name}
            className="w-16 h-16 rounded-full"
          />
          <div>
            <h3 className="text-lg font-semibold mb-2">{post.author.name}</h3>
            <p className="text-gray-600">{post.author.bio}</p>
          </div>
        </div>
      </section>
      
      {/* Related Posts */}
      {relatedPosts.length > 0 && (
        <section>
          <h2 className="text-2xl font-bold mb-6">Related Posts</h2>
          <div className="grid grid-cols-1 md:grid-cols-2 gap-6">
            {relatedPosts.map((relatedPost) => (
              <a 
                key={relatedPost.id}
                href={`/blog/${relatedPost.slug}`}
                className="block bg-white rounded-lg shadow-md p-6 hover:shadow-lg transition-shadow"
              >
                <h3 className="text-lg font-semibold mb-2">{relatedPost.title}</h3>
                <p className="text-gray-600 text-sm mb-3">{relatedPost.excerpt}</p>
                <p className="text-xs text-gray-500">
                  {new Date(relatedPost.date).toLocaleDateString()}
                </p>
              </a>
            ))}
          </div>
        </section>
      )}
    </article>
  );
}

export const getStaticPaths: GetStaticPaths = async () => {
  try {
    // Fetch all blog posts to get their slugs
    const response = await fetch('https://api.example.com/posts');
    const posts: BlogPost[] = await response.json();
    
    // Generate paths for all posts
    const paths = posts.map((post) => ({
      params: { slug: post.slug }
    }));
    
    return {
      paths,
      fallback: 'blocking' // Generate pages on-demand for new slugs
    };
  } catch (error) {
    console.error('Failed to fetch post slugs:', error);
    
    return {
      paths: [],
      fallback: 'blocking'
    };
  }
};

export const getStaticProps: GetStaticProps<BlogPostPageProps> = async ({ params }) => {
  try {
    const slug = params?.slug as string;
    
    // Fetch the specific post and related posts in parallel
    const [postResponse, allPostsResponse] = await Promise.all([
      fetch(`https://api.example.com/posts/${slug}`),
      fetch('https://api.example.com/posts')
    ]);
    
    if (!postResponse.ok) {
      return {
        notFound: true,
      };
    }
    
    const post: BlogPost = await postResponse.json();
    const allPosts: BlogPost[] = await allPostsResponse.json();
    
    // Find related posts based on shared tags
    const relatedPosts = allPosts
      .filter((p) => 
        p.id !== post.id && 
        p.tags.some(tag => post.tags.includes(tag))
      )
      .slice(0, 2);
    
    return {
      props: {
        post,
        relatedPosts,
      },
      revalidate: 3600, // Revalidate every hour
    };
  } catch (error) {
    console.error('Failed to fetch post:', error);
    
    return {
      notFound: true,
    };
  }
};
```

### Advanced getStaticPaths with Nested Routes

```typescript
// pages/products/[category]/[product].tsx
import { GetStaticPaths, GetStaticProps } from 'next';

interface Product {
  id: string;
  name: string;
  slug: string;
  category: {
    name: string;
    slug: string;
  };
  price: number;
  description: string;
  images: string[];
  specifications: Record<string, string>;
  reviews: {
    id: string;
    rating: number;
    comment: string;
    author: string;
    date: string;
  }[];
}

interface ProductPageProps {
  product: Product;
  similarProducts: Product[];
}

export default function ProductPage({ product, similarProducts }: ProductPageProps) {
  return (
    <div className="container mx-auto px-4 py-8">
      {/* Breadcrumb */}
      <nav className="mb-6">
        <ol className="flex space-x-2 text-sm text-gray-600">
          <li><a href="/" className="hover:text-blue-600">Home</a></li>
          <li>/</li>
          <li><a href="/products" className="hover:text-blue-600">Products</a></li>
          <li>/</li>
          <li><a href={`/products/${product.category.slug}`} className="hover:text-blue-600">
            {product.category.name}
          </a></li>
          <li>/</li>
          <li className="text-gray-900">{product.name}</li>
        </ol>
      </nav>
      
      <div className="grid grid-cols-1 lg:grid-cols-2 gap-12">
        {/* Product Images */}
        <div>
          <div className="aspect-w-1 aspect-h-1 mb-4">
            <img 
              src={product.images[0]} 
              alt={product.name}
              className="w-full h-96 object-cover rounded-lg"
            />
          </div>
          <div className="grid grid-cols-4 gap-2">
            {product.images.slice(1, 5).map((image, index) => (
              <img 
                key={index}
                src={image} 
                alt={`${product.name} ${index + 2}`}
                className="w-full h-20 object-cover rounded cursor-pointer hover:opacity-80"
              />
            ))}
          </div>
        </div>
        
        {/* Product Details */}
        <div>
          <h1 className="text-3xl font-bold mb-4">{product.name}</h1>
          <p className="text-2xl font-bold text-green-600 mb-6">
            ${product.price.toFixed(2)}
          </p>
          <p className="text-gray-600 mb-6">{product.description}</p>
          
          {/* Specifications */}
          <div className="mb-6">
            <h3 className="text-lg font-semibold mb-3">Specifications</h3>
            <dl className="grid grid-cols-1 gap-2">
              {Object.entries(product.specifications).map(([key, value]) => (
                <div key={key} className="flex justify-between py-2 border-b border-gray-200">
                  <dt className="font-medium text-gray-600">{key}</dt>
                  <dd className="text-gray-900">{value}</dd>
                </div>
              ))}
            </dl>
          </div>
          
          <button className="w-full bg-blue-600 text-white py-3 rounded-lg hover:bg-blue-700 transition-colors">
            Add to Cart
          </button>
        </div>
      </div>
      
      {/* Reviews */}
      <section className="mt-12">
        <h2 className="text-2xl font-bold mb-6">Customer Reviews</h2>
        <div className="space-y-6">
          {product.reviews.map((review) => (
            <div key={review.id} className="border-b border-gray-200 pb-6">
              <div className="flex items-center mb-2">
                <div className="flex text-yellow-500 mr-2">
                  {Array.from({ length: 5 }).map((_, i) => (
                    <span key={i}>{i < review.rating ? '★' : '☆'}</span>
                  ))}
                </div>
                <span className="font-medium">{review.author}</span>
                <span className="text-gray-500 ml-2">
                  {new Date(review.date).toLocaleDateString()}
                </span>
              </div>
              <p className="text-gray-600">{review.comment}</p>
            </div>
          ))}
        </div>
      </section>
      
      {/* Similar Products */}
      {similarProducts.length > 0 && (
        <section className="mt-12">
          <h2 className="text-2xl font-bold mb-6">Similar Products</h2>
          <div className="grid grid-cols-1 md:grid-cols-3 gap-6">
            {similarProducts.map((similarProduct) => (
              <a 
                key={similarProduct.id}
                href={`/products/${similarProduct.category.slug}/${similarProduct.slug}`}
                className="block bg-white rounded-lg shadow-md overflow-hidden hover:shadow-lg transition-shadow"
              >
                <img 
                  src={similarProduct.images[0]} 
                  alt={similarProduct.name}
                  className="w-full h-48 object-cover"
                />
                <div className="p-4">
                  <h3 className="font-semibold mb-2">{similarProduct.name}</h3>
                  <p className="text-green-600 font-bold">
                    ${similarProduct.price.toFixed(2)}
                  </p>
                </div>
              </a>
            ))}
          </div>
        </section>
      )}
    </div>
  );
}

export const getStaticPaths: GetStaticPaths = async () => {
  try {
    // Fetch all products with their categories
    const response = await fetch('https://api.example.com/products?include=category');
    const products: Product[] = await response.json();
    
    // Generate paths for all products
    const paths = products.map((product) => ({
      params: { 
        category: product.category.slug,
        product: product.slug 
      }
    }));
    
    return {
      paths,
      fallback: 'blocking'
    };
  } catch (error) {
    console.error('Failed to fetch product paths:', error);
    
    return {
      paths: [],
      fallback: 'blocking'
    };
  }
};

export const getStaticProps: GetStaticProps<ProductPageProps> = async ({ params }) => {
  try {
    const categorySlug = params?.category as string;
    const productSlug = params?.product as string;
    
    // Fetch the specific product
    const productResponse = await fetch(
      `https://api.example.com/products/${categorySlug}/${productSlug}?include=reviews,specifications`
    );
    
    if (!productResponse.ok) {
      return { notFound: true };
    }
    
    const product: Product = await productResponse.json();
    
    // Fetch similar products in the same category
    const similarResponse = await fetch(
      `https://api.example.com/products?category=${categorySlug}&exclude=${product.id}&limit=3`
    );
    const similarProducts: Product[] = await similarResponse.json();
    
    return {
      props: {
        product,
        similarProducts,
      },
      revalidate: 1800, // Revalidate every 30 minutes
    };
  } catch (error) {
    console.error('Failed to fetch product:', error);
    return { notFound: true };
  }
};
```

## Incremental Static Regeneration (ISR) {#isr}

ISR allows you to update static content after you've built your site, without rebuilding the entire site.

### Basic ISR Implementation

```typescript
// pages/news/index.tsx
import { GetStaticProps } from 'next';

interface NewsArticle {
  id: string;
  title: string;
  summary: string;
  publishedAt: string;
  category: string;
  slug: string;
  views: number;
}

interface NewsPageProps {
  articles: NewsArticle[];
  lastUpdated: string;
}

export default function NewsPage({ articles, lastUpdated }: NewsPageProps) {
  return (
    <div className="container mx-auto px-4 py-8">
      <div className="flex justify-between items-center mb-8">
        <h1 className="text-3xl font-bold">Latest News</h1>
        <p className="text-sm text-gray-500">
          Last updated: {new Date(lastUpdated).toLocaleString()}
        </p>
      </div>
      
      <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
        {articles.map((article) => (
          <article key={article.id} className="bg-white rounded-lg shadow-md p-6">
            <div className="flex justify-between items-start mb-3">
              <span className="px-2 py-1 bg-blue-100 text-blue-800 text-xs rounded">
                {article.category}
              </span>
              <span className="text-xs text-gray-500">
                {article.views.toLocaleString()} views
              </span>
            </div>
            <h2 className="text-xl font-semibold mb-3">{article.title}</h2>
            <p className="text-gray-600 mb-4">{article.summary}</p>
            <div className="flex justify-between items-center">
              <time className="text-sm text-gray-500">
                {new Date(article.publishedAt).toLocaleDateString()}
              </time>
              <a 
                href={`/news/${article.slug}`}
                className="text-blue-600 hover:text-blue-800 text-sm font-medium"
              >
                Read more →
              </a>
            </div>
          </article>
        ))}
      </div>
    </div>
  );
}

export const getStaticProps: GetStaticProps<NewsPageProps> = async () => {
  try {
    const response = await fetch('https://api.example.com/news?sort=publishedAt&limit=12');
    const articles: NewsArticle[] = await response.json();
    
    return {
      props: {
        articles,
        lastUpdated: new Date().toISOString(),
      },
      // Revalidate every 5 minutes
      revalidate: 300,
    };
  } catch (error) {
    console.error('Failed to fetch news:', error);
    
    return {
      props: {
        articles: [],
        lastUpdated: new Date().toISOString(),
      },
      revalidate: 60, // Retry more frequently on error
    };
  }
};
```

### Advanced ISR with On-Demand Revalidation

```typescript
// pages/api/revalidate.ts
import { NextApiRequest, NextApiResponse } from 'next';

export default async function handler(req: NextApiRequest, res: NextApiResponse) {
  // Check for secret to confirm this is a valid request
  if (req.query.secret !== process.env.REVALIDATION_SECRET) {
    return res.status(401).json({ message: 'Invalid token' });
  }
  
  try {
    const { paths } = req.body;
    
    if (!paths || !Array.isArray(paths)) {
      return res.status(400).json({ message: 'Paths array is required' });
    }
    
    // Revalidate multiple paths
    const revalidationPromises = paths.map(path => res.revalidate(path));
    await Promise.all(revalidationPromises);
    
    return res.json({ 
      revalidated: true,
      paths,
      timestamp: new Date().toISOString()
    });
  } catch (err) {
    console.error('Error revalidating:', err);
    return res.status(500).json({ message: 'Error revalidating' });
  }
}
```

```typescript
// utils/revalidation.ts
export async function triggerRevalidation(paths: string[]) {
  try {
    const response = await fetch('/api/revalidate', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        paths,
        secret: process.env.REVALIDATION_SECRET,
      }),
    });
    
    if (!response.ok) {
      throw new Error('Failed to trigger revalidation');
    }
    
    const result = await response.json();
    console.log('Revalidation triggered:', result);
    return result;
  } catch (error) {
    console.error('Revalidation error:', error);
    throw error;
  }
}

// Usage in a webhook or admin action
export async function handleContentUpdate(contentType: string, slug: string) {
  const pathsToRevalidate = [];
  
  switch (contentType) {
    case 'blog':
      pathsToRevalidate.push('/blog', `/blog/${slug}`);
      break;
    case 'product':
      pathsToRevalidate.push('/products', `/products/${slug}`);
      break;
    case 'news':
      pathsToRevalidate.push('/news', `/news/${slug}`);
      break;
  }
  
  if (pathsToRevalidate.length > 0) {
    await triggerRevalidation(pathsToRevalidate);
  }
}
```

## Best Practices {#best-practices}

### 1. Optimize Build Performance

```typescript
// next.config.js
/** @type {import('next').NextConfig} */
const nextConfig = {
  experimental: {
    // Optimize build performance for large sites
    workerThreads: false,
    cpus: Math.max(1, (require('os').cpus().length || 1) - 1),
  },
  
  // Configure build optimization
  typescript: {
    // Only type-check in development
    ignoreBuildErrors: process.env.NODE_ENV === 'production',
  },
  
  // Optimize images
  images: {
    domains: ['api.example.com', 'cdn.example.com'],
    formats: ['image/webp', 'image/avif'],
  },
  
  // Configure webpack for better bundling
  webpack: (config, { buildId, dev, isServer, defaultLoaders }) => {
    if (!dev && !isServer) {
      // Optimize client-side bundles
      config.optimization.splitChunks.cacheGroups = {
        ...config.optimization.splitChunks.cacheGroups,
        commons: {
          name: 'commons',
          chunks: 'all',
          minChunks: 2,
        },
      };
    }
    
    return config;
  },
};

module.exports = nextConfig;
```

### 2. Error Handling and Fallbacks

```typescript
// utils/staticGeneration.ts
export async function safeStaticGeneration<T>(
  fetcher: () => Promise<T>,
  fallback: T,
  revalidateOnError = 60
) {
  try {
    const data = await fetcher();
    return {
      props: data,
      revalidate: 3600, // Normal revalidation
    };
  } catch (error) {
    console.error('Static generation error:', error);
    
    return {
      props: fallback,
      revalidate: revalidateOnError, // Retry more frequently
    };
  }
}

// Usage in getStaticProps
export const getStaticProps: GetStaticProps = async () => {
  return safeStaticGeneration(
    async () => {
      const response = await fetch('https://api.example.com/data');
      const data = await response.json();
      return { data };
    },
    { data: [] }, // Fallback data
    60 // Revalidate every minute on error
  );
};
```

### 3. Content Management Integration

```typescript
// lib/cms.ts
interface CMSContent {
  id: string;
  title: string;
  content: string;
  publishedAt: string;
  updatedAt: string;
  slug: string;
}

class CMSClient {
  private baseUrl: string;
  private apiKey: string;
  
  constructor() {
    this.baseUrl = process.env.CMS_API_URL!;
    this.apiKey = process.env.CMS_API_KEY!;
  }
  
  private async fetch(endpoint: string, options: RequestInit = {}) {
    const response = await fetch(`${this.baseUrl}${endpoint}`, {
      ...options,
      headers: {
        'Authorization': `Bearer ${this.apiKey}`,
        'Content-Type': 'application/json',
        ...options.headers,
      },
    });
    
    if (!response.ok) {
      throw new Error(`CMS API error: ${response.status} ${response.statusText}`);
    }
    
    return response.json();
  }
  
  async getAllContent(contentType: string): Promise<CMSContent[]> {
    return this.fetch(`/${contentType}?published=true&sort=publishedAt:desc`);
  }
  
  async getContentBySlug(contentType: string, slug: string): Promise<CMSContent> {
    const results = await this.fetch(`/${contentType}?slug=${slug}&published=true`);
    if (results.length === 0) {
      throw new Error(`Content not found: ${slug}`);
    }
    return results[0];
  }
  
  async getStaticPaths(contentType: string) {
    const content = await this.getAllContent(contentType);
    return content.map(item => ({ params: { slug: item.slug } }));
  }
}

export const cms = new CMSClient();
```

## Exercises

### Exercise 1: Blog with Categories
Create a blog system with:
- Blog listing page with categories filter
- Individual blog post pages
- Category pages showing filtered posts
- Related posts based on categories
- Search functionality (static)

### Exercise 2: E-commerce Product Catalog
Build a product catalog with:
- Product listing with filtering
- Category and subcategory pages
- Individual product pages
- Related products
- Price ranges and specifications

### Exercise 3: News Portal with ISR
Create a news website with:
- Latest news with ISR (5-minute revalidation)
- Category-based news pages
- Individual article pages
- Trending articles
- On-demand revalidation API

### Exercise 4: Documentation Site
Build a documentation site with:
- Nested documentation structure
- Search functionality
- Code syntax highlighting
- Table of contents generation
- Previous/next navigation

## Summary

Static Site Generation in Next.js provides:

- **Performance**: Fastest possible loading times
- **SEO**: Perfect for search engine optimization
- **Scalability**: CDN-friendly static files
- **Developer Experience**: Simple API with powerful features

Key concepts:
- `getStaticProps` for data fetching at build time
- `getStaticPaths` for dynamic route generation
- ISR for updating static content without rebuilds
- Fallback strategies for handling new pages

Next, we'll explore **Server-Side Rendering (SSR)** and learn when and how to render pages on each request.
