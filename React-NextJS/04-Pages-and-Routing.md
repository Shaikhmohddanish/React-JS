# Pages and Routing in Next.js - A Complete Guide

## Introduction to Next.js Routing

Next.js provides a powerful file-based routing system that eliminates the need for complex routing configuration. Instead of defining routes in a separate file, you create pages by simply adding files to specific directories. This approach makes routing intuitive and easy to understand while providing advanced features like dynamic routes, nested layouts, and route groups.

## File-Based Routing Fundamentals

### How File-Based Routing Works

In Next.js, the file structure directly maps to your application's routes:

```
src/app/
├── page.tsx              # / (home page)
├── about/
│   └── page.tsx         # /about
├── blog/
│   ├── page.tsx         # /blog
│   └── [slug]/
│       └── page.tsx     # /blog/[slug] (dynamic route)
├── products/
│   ├── page.tsx         # /products
│   ├── [id]/
│   │   └── page.tsx     # /products/[id]
│   └── [...slug]/
│       └── page.tsx     # /products/[...slug] (catch-all route)
└── api/
    └── users/
        └── route.ts     # /api/users (API route)
```

### Basic Page Creation

```tsx
// src/app/page.tsx - Home page (/)
export default function HomePage() {
  return (
    <div>
      <h1>Welcome to My Website</h1>
      <p>This is the home page.</p>
    </div>
  );
}
```

```tsx
// src/app/about/page.tsx - About page (/about)
import { Metadata } from 'next';

export const metadata: Metadata = {
  title: 'About Us',
  description: 'Learn more about our company and mission.',
};

export default function AboutPage() {
  return (
    <div>
      <h1>About Us</h1>
      <p>We are a company dedicated to building amazing web applications.</p>
    </div>
  );
}
```

```tsx
// src/app/contact/page.tsx - Contact page (/contact)
export default function ContactPage() {
  return (
    <div>
      <h1>Contact Us</h1>
      <form>
        <div>
          <label htmlFor="name">Name:</label>
          <input type="text" id="name" name="name" required />
        </div>
        <div>
          <label htmlFor="email">Email:</label>
          <input type="email" id="email" name="email" required />
        </div>
        <div>
          <label htmlFor="message">Message:</label>
          <textarea id="message" name="message" required></textarea>
        </div>
        <button type="submit">Send Message</button>
      </form>
    </div>
  );
}
```

## Dynamic Routes

### Single Dynamic Segments

Dynamic routes allow you to create pages that can handle multiple URLs with a similar pattern:

```tsx
// src/app/blog/[slug]/page.tsx
interface Props {
  params: { slug: string };
  searchParams: { [key: string]: string | string[] | undefined };
}

export default async function BlogPost({ params, searchParams }: Props) {
  const { slug } = params;
  
  // Fetch post data based on slug
  const post = await getPostBySlug(slug);
  
  if (!post) {
    return <div>Post not found</div>;
  }

  return (
    <article>
      <h1>{post.title}</h1>
      <p className="text-gray-600">Published on {post.publishedAt}</p>
      <div className="prose">
        {post.content}
      </div>
    </article>
  );
}

// Generate static params for static generation
export async function generateStaticParams() {
  const posts = await getAllPosts();
  
  return posts.map((post) => ({
    slug: post.slug,
  }));
}

// Generate metadata dynamically
export async function generateMetadata({ params }: Props): Promise<Metadata> {
  const post = await getPostBySlug(params.slug);
  
  if (!post) {
    return {
      title: 'Post Not Found',
    };
  }

  return {
    title: post.title,
    description: post.excerpt,
    openGraph: {
      title: post.title,
      description: post.excerpt,
      images: [post.coverImage],
    },
  };
}
```

```tsx
// src/app/users/[id]/page.tsx
interface Props {
  params: { id: string };
}

export default async function UserProfile({ params }: Props) {
  const { id } = params;
  
  try {
    const user = await getUserById(id);
    
    return (
      <div>
        <h1>{user.name}</h1>
        <p>Email: {user.email}</p>
        <p>Joined: {new Date(user.createdAt).toLocaleDateString()}</p>
        
        <div>
          <h2>Recent Posts</h2>
          <UserPosts userId={id} />
        </div>
      </div>
    );
  } catch (error) {
    return <div>User not found</div>;
  }
}

// Component for user posts
async function UserPosts({ userId }: { userId: string }) {
  const posts = await getPostsByUserId(userId);
  
  return (
    <div>
      {posts.map((post) => (
        <div key={post.id}>
          <h3>{post.title}</h3>
          <p>{post.excerpt}</p>
        </div>
      ))}
    </div>
  );
}
```

### Multiple Dynamic Segments

```tsx
// src/app/shop/[category]/[product]/page.tsx
interface Props {
  params: { 
    category: string;
    product: string;
  };
}

export default async function ProductPage({ params }: Props) {
  const { category, product } = params;
  
  const productData = await getProduct(category, product);
  
  return (
    <div>
      <nav>
        <a href="/shop">Shop</a> / 
        <a href={`/shop/${category}`}>{category}</a> / 
        {product}
      </nav>
      
      <h1>{productData.name}</h1>
      <p>Category: {productData.category}</p>
      <p>Price: ${productData.price}</p>
      <p>{productData.description}</p>
    </div>
  );
}

export async function generateStaticParams() {
  const products = await getAllProducts();
  
  return products.map((product) => ({
    category: product.category,
    product: product.slug,
  }));
}
```

## Catch-All Routes

### Basic Catch-All Routes

Catch-all routes capture multiple segments in a single dynamic route:

```tsx
// src/app/docs/[...slug]/page.tsx
interface Props {
  params: { slug: string[] };
}

export default async function DocsPage({ params }: Props) {
  const { slug } = params;
  
  // slug is an array: ['getting-started', 'installation'] for /docs/getting-started/installation
  const docPath = slug.join('/');
  const doc = await getDocByPath(docPath);
  
  if (!doc) {
    return <div>Documentation not found</div>;
  }

  return (
    <div className="docs-layout">
      <aside>
        <DocsNavigation currentPath={docPath} />
      </aside>
      
      <main>
        <Breadcrumbs segments={slug} />
        <h1>{doc.title}</h1>
        <div dangerouslySetInnerHTML={{ __html: doc.content }} />
      </main>
    </div>
  );
}

// Breadcrumbs component
function Breadcrumbs({ segments }: { segments: string[] }) {
  return (
    <nav className="breadcrumbs">
      <a href="/docs">Docs</a>
      {segments.map((segment, index) => {
        const path = `/docs/${segments.slice(0, index + 1).join('/')}`;
        const isLast = index === segments.length - 1;
        
        return (
          <span key={segment}>
            {' / '}
            {isLast ? (
              segment
            ) : (
              <a href={path}>{segment}</a>
            )}
          </span>
        );
      })}
    </nav>
  );
}
```

### Optional Catch-All Routes

Optional catch-all routes also match the parent route without any segments:

```tsx
// src/app/shop/[[...filters]]/page.tsx
interface Props {
  params: { filters?: string[] };
  searchParams: { [key: string]: string | string[] | undefined };
}

export default async function ShopPage({ params, searchParams }: Props) {
  const { filters = [] } = params;
  const { sort, search } = searchParams;
  
  // Build filter object from URL segments
  const filterObject = buildFilters(filters);
  
  const products = await getProducts({
    filters: filterObject,
    sort: sort as string,
    search: search as string,
  });

  return (
    <div>
      <h1>Shop</h1>
      
      <div className="shop-layout">
        <aside>
          <ShopFilters 
            currentFilters={filterObject}
            onFilterChange={handleFilterChange}
          />
        </aside>
        
        <main>
          <div className="controls">
            <SearchInput defaultValue={search as string} />
            <SortSelect defaultValue={sort as string} />
          </div>
          
          <ProductGrid products={products} />
        </main>
      </div>
    </div>
  );
}

// This route handles:
// /shop - All products
// /shop/electronics - Electronics category
// /shop/electronics/phones - Electronics > Phones
// /shop/electronics/phones/apple - Electronics > Phones > Apple
function buildFilters(filters: string[]): FilterObject {
  const filterObj: FilterObject = {};
  
  if (filters.length > 0) filterObj.category = filters[0];
  if (filters.length > 1) filterObj.subcategory = filters[1];
  if (filters.length > 2) filterObj.brand = filters[2];
  
  return filterObj;
}
```

## Route Groups

Route groups allow you to organize routes without affecting the URL structure:

```
src/app/
├── (marketing)/
│   ├── layout.tsx       # Layout for marketing pages
│   ├── page.tsx         # / (home page)
│   ├── about/
│   │   └── page.tsx     # /about
│   └── contact/
│       └── page.tsx     # /contact
├── (dashboard)/
│   ├── layout.tsx       # Layout for dashboard pages
│   ├── analytics/
│   │   └── page.tsx     # /analytics
│   ├── settings/
│   │   └── page.tsx     # /settings
│   └── profile/
│       └── page.tsx     # /profile
└── layout.tsx           # Root layout
```

```tsx
// src/app/(marketing)/layout.tsx
export default function MarketingLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <div className="marketing-layout">
      <header className="marketing-header">
        <nav>
          <a href="/">Home</a>
          <a href="/about">About</a>
          <a href="/contact">Contact</a>
        </nav>
      </header>
      
      <main>{children}</main>
      
      <footer className="marketing-footer">
        <p>&copy; 2024 My Company. All rights reserved.</p>
      </footer>
    </div>
  );
}
```

```tsx
// src/app/(dashboard)/layout.tsx
import { Sidebar } from '@/components/dashboard/Sidebar';
import { DashboardHeader } from '@/components/dashboard/Header';

export default function DashboardLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <div className="dashboard-layout">
      <DashboardHeader />
      
      <div className="dashboard-content">
        <Sidebar />
        <main className="dashboard-main">
          {children}
        </main>
      </div>
    </div>
  );
}
```

## Navigation and Links

### Using the Link Component

```tsx
import Link from 'next/link';

export function Navigation() {
  return (
    <nav>
      {/* Basic link */}
      <Link href="/">Home</Link>
      
      {/* Link with custom styling */}
      <Link 
        href="/about" 
        className="nav-link"
      >
        About
      </Link>
      
      {/* Dynamic link */}
      <Link href={`/blog/${post.slug}`}>
        {post.title}
      </Link>
      
      {/* External link (use regular anchor tag) */}
      <a 
        href="https://example.com" 
        target="_blank" 
        rel="noopener noreferrer"
      >
        External Link
      </a>
      
      {/* Prefetch disabled (for large pages) */}
      <Link 
        href="/heavy-page" 
        prefetch={false}
      >
        Heavy Page
      </Link>
      
      {/* Replace history instead of pushing */}
      <Link 
        href="/login" 
        replace
      >
        Login
      </Link>
    </nav>
  );
}
```

### Programmatic Navigation

```tsx
'use client';

import { useRouter } from 'next/navigation';
import { useTransition } from 'react';

export function NavigationExample() {
  const router = useRouter();
  const [isPending, startTransition] = useTransition();

  const handleNavigation = () => {
    startTransition(() => {
      router.push('/dashboard');
    });
  };

  const handleBack = () => {
    router.back();
  };

  const handleReplace = () => {
    router.replace('/new-page');
  };

  const handleRefresh = () => {
    router.refresh();
  };

  return (
    <div>
      <button 
        onClick={handleNavigation}
        disabled={isPending}
      >
        {isPending ? 'Navigating...' : 'Go to Dashboard'}
      </button>
      
      <button onClick={handleBack}>
        Go Back
      </button>
      
      <button onClick={handleReplace}>
        Replace Current Page
      </button>
      
      <button onClick={handleRefresh}>
        Refresh Page
      </button>
    </div>
  );
}
```

### Active Link Component

```tsx
'use client';

import Link from 'next/link';
import { usePathname } from 'next/navigation';
import { ReactNode } from 'react';

interface ActiveLinkProps {
  href: string;
  children: ReactNode;
  activeClassName?: string;
  exactMatch?: boolean;
}

export function ActiveLink({ 
  href, 
  children, 
  activeClassName = 'active',
  exactMatch = false 
}: ActiveLinkProps) {
  const pathname = usePathname();
  
  const isActive = exactMatch 
    ? pathname === href
    : pathname.startsWith(href);

  return (
    <Link 
      href={href}
      className={isActive ? activeClassName : ''}
    >
      {children}
    </Link>
  );
}

// Usage
export function Navigation() {
  return (
    <nav>
      <ActiveLink href="/" exactMatch>
        Home
      </ActiveLink>
      <ActiveLink href="/blog">
        Blog
      </ActiveLink>
      <ActiveLink href="/about" exactMatch>
        About
      </ActiveLink>
    </nav>
  );
}
```

## Layouts and Templates

### Nested Layouts

```tsx
// src/app/layout.tsx - Root layout
export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body>
        <div className="app">
          {children}
        </div>
      </body>
    </html>
  );
}
```

```tsx
// src/app/blog/layout.tsx - Blog layout
import { BlogSidebar } from '@/components/blog/Sidebar';

export default function BlogLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <div className="blog-layout">
      <header className="blog-header">
        <h1>My Blog</h1>
        <nav>
          <a href="/blog">All Posts</a>
          <a href="/blog/categories">Categories</a>
          <a href="/blog/archive">Archive</a>
        </nav>
      </header>
      
      <div className="blog-content">
        <main className="blog-main">
          {children}
        </main>
        <aside className="blog-sidebar">
          <BlogSidebar />
        </aside>
      </div>
    </div>
  );
}
```

```tsx
// src/app/blog/[slug]/layout.tsx - Individual post layout
import { BlogPostHeader } from '@/components/blog/PostHeader';
import { BlogPostSidebar } from '@/components/blog/PostSidebar';

interface Props {
  children: React.ReactNode;
  params: { slug: string };
}

export default async function BlogPostLayout({ children, params }: Props) {
  const post = await getPostBySlug(params.slug);
  
  return (
    <article className="blog-post">
      <BlogPostHeader post={post} />
      
      <div className="blog-post-content">
        <div className="blog-post-body">
          {children}
        </div>
        
        <aside className="blog-post-sidebar">
          <BlogPostSidebar post={post} />
        </aside>
      </div>
    </article>
  );
}
```

### Templates vs Layouts

Templates create a new instance for each route, while layouts persist:

```tsx
// src/app/blog/template.tsx - Creates new instance for each navigation
'use client';

import { useEffect } from 'react';

export default function BlogTemplate({
  children,
}: {
  children: React.ReactNode;
}) {
  useEffect(() => {
    // This runs on every navigation to blog pages
    console.log('Blog template mounted');
  }, []);

  return (
    <div className="blog-template">
      {children}
    </div>
  );
}
```

## URL Parameters and Search Params

### Accessing URL Parameters

```tsx
// src/app/products/[id]/page.tsx
interface Props {
  params: { id: string };
  searchParams: { 
    color?: string;
    size?: string;
    variant?: string;
  };
}

export default function ProductPage({ params, searchParams }: Props) {
  const { id } = params;
  const { color, size, variant } = searchParams;

  return (
    <div>
      <h1>Product {id}</h1>
      {color && <p>Color: {color}</p>}
      {size && <p>Size: {size}</p>}
      {variant && <p>Variant: {variant}</p>}
      
      <ProductDetails 
        id={id}
        selectedColor={color}
        selectedSize={size}
        selectedVariant={variant}
      />
    </div>
  );
}
```

### Client-Side URL Manipulation

```tsx
'use client';

import { useRouter, useSearchParams, usePathname } from 'next/navigation';
import { useCallback } from 'react';

export function ProductFilters() {
  const router = useRouter();
  const pathname = usePathname();
  const searchParams = useSearchParams();

  const createQueryString = useCallback(
    (name: string, value: string) => {
      const params = new URLSearchParams(searchParams.toString());
      params.set(name, value);
      return params.toString();
    },
    [searchParams]
  );

  const removeQueryParam = useCallback(
    (name: string) => {
      const params = new URLSearchParams(searchParams.toString());
      params.delete(name);
      return params.toString();
    },
    [searchParams]
  );

  const handleColorChange = (color: string) => {
    router.push(pathname + '?' + createQueryString('color', color));
  };

  const handleSizeChange = (size: string) => {
    router.push(pathname + '?' + createQueryString('size', size));
  };

  const clearFilters = () => {
    router.push(pathname);
  };

  return (
    <div className="filters">
      <div>
        <h3>Color</h3>
        <button onClick={() => handleColorChange('red')}>Red</button>
        <button onClick={() => handleColorChange('blue')}>Blue</button>
        <button onClick={() => handleColorChange('green')}>Green</button>
      </div>
      
      <div>
        <h3>Size</h3>
        <button onClick={() => handleSizeChange('small')}>Small</button>
        <button onClick={() => handleSizeChange('medium')}>Medium</button>
        <button onClick={() => handleSizeChange('large')}>Large</button>
      </div>
      
      <button onClick={clearFilters}>Clear Filters</button>
    </div>
  );
}
```

## Route Handlers (API Routes)

### Basic API Routes

```typescript
// src/app/api/posts/route.ts
import { NextRequest, NextResponse } from 'next/server';

export async function GET(request: NextRequest) {
  const { searchParams } = new URL(request.url);
  const page = searchParams.get('page') || '1';
  const limit = searchParams.get('limit') || '10';

  try {
    const posts = await getPosts({
      page: parseInt(page),
      limit: parseInt(limit),
    });

    return NextResponse.json({
      posts,
      pagination: {
        page: parseInt(page),
        limit: parseInt(limit),
        total: posts.length,
      },
    });
  } catch (error) {
    return NextResponse.json(
      { error: 'Failed to fetch posts' },
      { status: 500 }
    );
  }
}

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { title, content, author } = body;

    // Validate required fields
    if (!title || !content || !author) {
      return NextResponse.json(
        { error: 'Missing required fields' },
        { status: 400 }
      );
    }

    const newPost = await createPost({ title, content, author });

    return NextResponse.json(newPost, { status: 201 });
  } catch (error) {
    return NextResponse.json(
      { error: 'Failed to create post' },
      { status: 500 }
    );
  }
}
```

### Dynamic API Routes

```typescript
// src/app/api/posts/[id]/route.ts
import { NextRequest, NextResponse } from 'next/server';

interface Context {
  params: { id: string };
}

export async function GET(
  request: NextRequest,
  context: Context
) {
  const { id } = context.params;

  try {
    const post = await getPostById(id);
    
    if (!post) {
      return NextResponse.json(
        { error: 'Post not found' },
        { status: 404 }
      );
    }

    return NextResponse.json(post);
  } catch (error) {
    return NextResponse.json(
      { error: 'Failed to fetch post' },
      { status: 500 }
    );
  }
}

export async function PUT(
  request: NextRequest,
  context: Context
) {
  const { id } = context.params;

  try {
    const body = await request.json();
    const updatedPost = await updatePost(id, body);

    return NextResponse.json(updatedPost);
  } catch (error) {
    return NextResponse.json(
      { error: 'Failed to update post' },
      { status: 500 }
    );
  }
}

export async function DELETE(
  request: NextRequest,
  context: Context
) {
  const { id } = context.params;

  try {
    await deletePost(id);

    return NextResponse.json(
      { message: 'Post deleted successfully' },
      { status: 200 }
    );
  } catch (error) {
    return NextResponse.json(
      { error: 'Failed to delete post' },
      { status: 500 }
    );
  }
}
```

## Advanced Routing Patterns

### Parallel Routes

Parallel routes allow you to render multiple pages in the same layout:

```
src/app/dashboard/
├── layout.tsx
├── page.tsx
├── @analytics/
│   ├── page.tsx
│   └── loading.tsx
├── @team/
│   ├── page.tsx
│   └── error.tsx
└── settings/
    ├── page.tsx
    ├── @analytics/
    │   └── page.tsx
    └── @team/
        └── page.tsx
```

```tsx
// src/app/dashboard/layout.tsx
export default function DashboardLayout({
  children,
  analytics,
  team,
}: {
  children: React.ReactNode;
  analytics: React.ReactNode;
  team: React.ReactNode;
}) {
  return (
    <div className="dashboard">
      <div className="main-content">
        {children}
      </div>
      
      <div className="sidebar">
        <div className="analytics-section">
          {analytics}
        </div>
        
        <div className="team-section">
          {team}
        </div>
      </div>
    </div>
  );
}
```

### Intercepting Routes

Intercept routes to show content in a modal while keeping the URL:

```
src/app/
├── layout.tsx
├── page.tsx
├── photos/
│   ├── page.tsx
│   └── [id]/
│       └── page.tsx
├── @modal/
│   └── (..)photos/
│       └── [id]/
│           └── page.tsx
└── modal.tsx
```

```tsx
// src/app/@modal/(..)photos/[id]/page.tsx
import { Modal } from '@/components/ui/Modal';
import { PhotoDetail } from '@/components/PhotoDetail';

interface Props {
  params: { id: string };
}

export default function PhotoModal({ params }: Props) {
  return (
    <Modal>
      <PhotoDetail photoId={params.id} />
    </Modal>
  );
}
```

## Route Validation and Error Handling

### Input Validation

```tsx
// src/app/api/posts/route.ts
import { z } from 'zod';

const createPostSchema = z.object({
  title: z.string().min(1, 'Title is required').max(100, 'Title too long'),
  content: z.string().min(10, 'Content too short'),
  tags: z.array(z.string()).optional(),
  published: z.boolean().default(false),
});

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    
    // Validate input
    const validatedData = createPostSchema.parse(body);
    
    const post = await createPost(validatedData);
    
    return NextResponse.json(post, { status: 201 });
  } catch (error) {
    if (error instanceof z.ZodError) {
      return NextResponse.json(
        { 
          error: 'Validation failed',
          details: error.errors 
        },
        { status: 400 }
      );
    }
    
    return NextResponse.json(
      { error: 'Internal server error' },
      { status: 500 }
    );
  }
}
```

### Custom Error Pages

```tsx
// src/app/blog/[slug]/not-found.tsx
import Link from 'next/link';

export default function BlogPostNotFound() {
  return (
    <div className="text-center py-16">
      <h1 className="text-4xl font-bold mb-4">Blog Post Not Found</h1>
      <p className="text-gray-600 mb-8">
        The blog post you're looking for doesn't exist.
      </p>
      <div className="space-x-4">
        <Link 
          href="/blog"
          className="bg-blue-500 text-white px-4 py-2 rounded"
        >
          Back to Blog
        </Link>
        <Link 
          href="/"
          className="bg-gray-500 text-white px-4 py-2 rounded"
        >
          Home
        </Link>
      </div>
    </div>
  );
}
```

```tsx
// src/app/blog/error.tsx
'use client';

interface Props {
  error: Error & { digest?: string };
  reset: () => void;
}

export default function BlogError({ error, reset }: Props) {
  return (
    <div className="text-center py-16">
      <h1 className="text-4xl font-bold mb-4">Something went wrong!</h1>
      <p className="text-gray-600 mb-8">
        There was an error loading the blog content.
      </p>
      <button
        onClick={reset}
        className="bg-blue-500 text-white px-4 py-2 rounded"
      >
        Try again
      </button>
    </div>
  );
}
```

## Summary

Next.js routing provides a powerful and intuitive way to build complex applications. Here's what we covered:

### ✅ Core Concepts:
- **File-based routing** eliminates manual route configuration
- **Dynamic routes** handle variable URL segments
- **Catch-all routes** capture multiple segments
- **Route groups** organize without affecting URLs

### ✅ Navigation:
- **Link component** for client-side navigation
- **Programmatic navigation** with useRouter
- **Active links** for navigation state
- **URL manipulation** with search params

### ✅ Advanced Features:
- **Nested layouts** for complex page structures
- **Parallel routes** for multiple content areas
- **Intercepting routes** for modal workflows
- **API routes** for backend functionality

### ✅ Best Practices:
- **Consistent naming** for route files
- **Proper error handling** with custom error pages
- **Input validation** for API routes
- **SEO optimization** with metadata

### Key Takeaways:
1. **File structure equals URL structure** in Next.js
2. **Dynamic routes** provide flexibility for content-driven apps
3. **Layouts** enable consistent page structure
4. **API routes** allow full-stack development
5. **Error boundaries** improve user experience

Your Next.js routing knowledge is now comprehensive! In the next guide, we'll compare the App Router with the Pages Router to help you choose the right approach.

## Practice Exercise

Build a blog application with:

1. **Home page** with featured posts
2. **Blog listing** with pagination
3. **Dynamic blog posts** with slugs
4. **Category pages** with catch-all routes
5. **API routes** for post management
6. **Proper navigation** and active links

## Next Steps

Continue to [App Router vs Pages Router](./05-App-Router-vs-Pages-Router.md) to understand the differences between Next.js routing systems and learn migration strategies.
