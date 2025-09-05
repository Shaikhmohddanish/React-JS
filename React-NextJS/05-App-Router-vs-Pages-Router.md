# App Router vs Pages Router - A Complete Comparison Guide

## Introduction

Next.js offers two routing systems: the newer **App Router** (introduced in Next.js 13) and the traditional **Pages Router**. Understanding the differences between these two approaches is crucial for choosing the right one for your project and planning migrations. This guide provides a comprehensive comparison, migration strategies, and best practices for both systems.

## Overview of Both Routing Systems

### Pages Router (Traditional)

The Pages Router has been the backbone of Next.js since its inception:

```
pages/
├── _app.tsx             # Custom App component
├── _document.tsx        # Custom Document
├── index.tsx            # Home page (/)
├── about.tsx            # /about
├── blog/
│   ├── index.tsx        # /blog
│   └── [slug].tsx       # /blog/[slug]
├── api/
│   └── posts.ts         # /api/posts
└── 404.tsx              # Custom 404 page
```

### App Router (Modern)

The App Router is the future of Next.js routing:

```
src/app/
├── layout.tsx           # Root layout
├── page.tsx             # Home page (/)
├── loading.tsx          # Loading UI
├── error.tsx            # Error UI
├── about/
│   └── page.tsx         # /about
├── blog/
│   ├── page.tsx         # /blog
│   ├── layout.tsx       # Blog layout
│   └── [slug]/
│       └── page.tsx     # /blog/[slug]
└── api/
    └── posts/
        └── route.ts     # /api/posts
```

## Key Differences

### 1. File Structure and Conventions

**Pages Router:**
```tsx
// pages/blog/[slug].tsx
import { GetStaticProps, GetStaticPaths } from 'next';

interface Props {
  post: Post;
}

export default function BlogPost({ post }: Props) {
  return (
    <div>
      <h1>{post.title}</h1>
      <p>{post.content}</p>
    </div>
  );
}

export const getStaticPaths: GetStaticPaths = async () => {
  const posts = await getPosts();
  const paths = posts.map((post) => ({ params: { slug: post.slug } }));
  return { paths, fallback: false };
};

export const getStaticProps: GetStaticProps = async ({ params }) => {
  const post = await getPost(params!.slug as string);
  return { props: { post } };
};
```

**App Router:**
```tsx
// src/app/blog/[slug]/page.tsx
interface Props {
  params: { slug: string };
}

export default async function BlogPost({ params }: Props) {
  const post = await getPost(params.slug);
  
  return (
    <div>
      <h1>{post.title}</h1>
      <p>{post.content}</p>
    </div>
  );
}

export async function generateStaticParams() {
  const posts = await getPosts();
  return posts.map((post) => ({ slug: post.slug }));
}
```

### 2. Data Fetching

**Pages Router - Multiple Methods:**
```tsx
// Static Generation
export const getStaticProps: GetStaticProps = async (context) => {
  const data = await fetchData();
  return {
    props: { data },
    revalidate: 3600, // ISR
  };
};

// Server-Side Rendering
export const getServerSideProps: GetServerSideProps = async (context) => {
  const data = await fetchData();
  return { props: { data } };
};

// Client-Side Fetching
function MyComponent() {
  const [data, setData] = useState(null);
  
  useEffect(() => {
    fetchData().then(setData);
  }, []);
  
  return <div>{data ? <Content data={data} /> : 'Loading...'}</div>;
}
```

**App Router - Simplified:**
```tsx
// Server Components (default) - SSG/SSR automatically determined
export default async function MyPage() {
  const data = await fetchData(); // Server-side fetch
  return <div><Content data={data} /></div>;
}

// Client Components when needed
'use client';

import { useState, useEffect } from 'react';

export default function MyClientComponent() {
  const [data, setData] = useState(null);
  
  useEffect(() => {
    fetchData().then(setData);
  }, []);
  
  return <div>{data ? <Content data={data} /> : 'Loading...'}</div>;
}

// Revalidation
export const revalidate = 3600; // ISR
```

### 3. Layouts

**Pages Router - Custom App:**
```tsx
// pages/_app.tsx
import type { AppProps } from 'next/app';
import Layout from '../components/Layout';

export default function MyApp({ Component, pageProps }: AppProps) {
  return (
    <Layout>
      <Component {...pageProps} />
    </Layout>
  );
}

// components/Layout.tsx
export default function Layout({ children }: { children: React.ReactNode }) {
  return (
    <div>
      <header>Header</header>
      <main>{children}</main>
      <footer>Footer</footer>
    </div>
  );
}
```

**App Router - Native Layouts:**
```tsx
// src/app/layout.tsx (Root Layout)
export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body>
        <header>Header</header>
        <main>{children}</main>
        <footer>Footer</footer>
      </body>
    </html>
  );
}

// src/app/blog/layout.tsx (Nested Layout)
export default function BlogLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <div className="blog-layout">
      <aside>Blog Sidebar</aside>
      <div>{children}</div>
    </div>
  );
}
```

### 4. API Routes

**Pages Router:**
```typescript
// pages/api/posts.ts
import type { NextApiRequest, NextApiResponse } from 'next';

export default function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method === 'GET') {
    // Handle GET
    res.status(200).json({ posts: [] });
  } else if (req.method === 'POST') {
    // Handle POST
    res.status(201).json({ success: true });
  } else {
    res.setHeader('Allow', ['GET', 'POST']);
    res.status(405).end(`Method ${req.method} Not Allowed`);
  }
}
```

**App Router:**
```typescript
// src/app/api/posts/route.ts
import { NextRequest, NextResponse } from 'next/server';

export async function GET(request: NextRequest) {
  return NextResponse.json({ posts: [] });
}

export async function POST(request: NextRequest) {
  const body = await request.json();
  return NextResponse.json({ success: true }, { status: 201 });
}
```

### 5. Metadata and SEO

**Pages Router:**
```tsx
import Head from 'next/head';

export default function BlogPost({ post }: { post: Post }) {
  return (
    <>
      <Head>
        <title>{post.title}</title>
        <meta name="description" content={post.excerpt} />
        <meta property="og:title" content={post.title} />
      </Head>
      
      <article>
        <h1>{post.title}</h1>
        <p>{post.content}</p>
      </article>
    </>
  );
}
```

**App Router:**
```tsx
import { Metadata } from 'next';

export async function generateMetadata({ params }: Props): Promise<Metadata> {
  const post = await getPost(params.slug);
  
  return {
    title: post.title,
    description: post.excerpt,
    openGraph: {
      title: post.title,
      description: post.excerpt,
    },
  };
}

export default function BlogPost({ post }: { post: Post }) {
  return (
    <article>
      <h1>{post.title}</h1>
      <p>{post.content}</p>
    </article>
  );
}
```

## Feature Comparison Table

| Feature | Pages Router | App Router |
|---------|-------------|------------|
| **Routing** | File-based | File-based |
| **Layouts** | Custom `_app.tsx` | Native nested layouts |
| **Data Fetching** | `getStaticProps`, `getServerSideProps` | Server Components, async/await |
| **Client Components** | Default | Explicit `'use client'` |
| **Server Components** | Not available | Default |
| **Streaming** | Limited support | Native support |
| **Suspense** | Limited | Full support |
| **Error Boundaries** | Custom implementation | Built-in `error.tsx` |
| **Loading States** | Custom implementation | Built-in `loading.tsx` |
| **Parallel Routes** | Not supported | Supported |
| **Intercepting Routes** | Not supported | Supported |
| **Middleware** | Supported | Enhanced support |
| **TypeScript** | Good support | Enhanced support |

## When to Use Each Router

### Use Pages Router When:

1. **Existing Projects**: Large existing codebases that would be expensive to migrate
2. **Team Familiarity**: Team is deeply familiar with Pages Router patterns
3. **Specific Dependencies**: Using libraries that don't support App Router yet
4. **Gradual Adoption**: Want to stay on a stable, proven system
5. **SSG-Heavy Sites**: Simple static sites with occasional SSR needs

**Example Use Cases:**
- Existing e-commerce platforms
- Marketing websites
- Documentation sites
- Simple blogs

### Use App Router When:

1. **New Projects**: Starting fresh projects
2. **Modern Features**: Need streaming, Suspense, or Server Components
3. **Complex Layouts**: Require nested layouts and parallel routes
4. **Performance Critical**: Need the latest performance optimizations
5. **Future-Proofing**: Want to use the latest Next.js features

**Example Use Cases:**
- Modern web applications
- Complex dashboards
- Social media platforms
- Real-time applications

## Migration Strategies

### 1. Gradual Migration (Recommended)

You can run both routers simultaneously during migration:

```
my-app/
├── pages/              # Existing Pages Router
│   ├── _app.tsx
│   ├── index.tsx
│   └── about.tsx
├── src/app/            # New App Router
│   ├── layout.tsx
│   ├── blog/
│   │   └── page.tsx
│   └── dashboard/
│       └── page.tsx
└── next.config.js
```

```javascript
// next.config.js
/** @type {import('next').NextConfig} */
const nextConfig = {
  experimental: {
    appDir: true,
  },
};

module.exports = nextConfig;
```

### 2. Page-by-Page Migration

**Step 1: Migrate Simple Static Pages**

```tsx
// Before (pages/about.tsx)
export default function About() {
  return (
    <div>
      <h1>About Us</h1>
      <p>Learn more about our company.</p>
    </div>
  );
}

// After (src/app/about/page.tsx)
export default function About() {
  return (
    <div>
      <h1>About Us</h1>
      <p>Learn more about our company.</p>
    </div>
  );
}
```

**Step 2: Migrate Pages with Static Data**

```tsx
// Before (pages/blog/index.tsx)
import { GetStaticProps } from 'next';

interface Props {
  posts: Post[];
}

export default function Blog({ posts }: Props) {
  return (
    <div>
      <h1>Blog</h1>
      {posts.map(post => (
        <article key={post.id}>
          <h2>{post.title}</h2>
          <p>{post.excerpt}</p>
        </article>
      ))}
    </div>
  );
}

export const getStaticProps: GetStaticProps = async () => {
  const posts = await getPosts();
  return { props: { posts } };
};

// After (src/app/blog/page.tsx)
export default async function Blog() {
  const posts = await getPosts();
  
  return (
    <div>
      <h1>Blog</h1>
      {posts.map(post => (
        <article key={post.id}>
          <h2>{post.title}</h2>
          <p>{post.excerpt}</p>
        </article>
      ))}
    </div>
  );
}
```

**Step 3: Migrate Dynamic Routes**

```tsx
// Before (pages/blog/[slug].tsx)
import { GetStaticProps, GetStaticPaths } from 'next';

interface Props {
  post: Post;
}

export default function BlogPost({ post }: Props) {
  return (
    <article>
      <h1>{post.title}</h1>
      <div dangerouslySetInnerHTML={{ __html: post.content }} />
    </article>
  );
}

export const getStaticPaths: GetStaticPaths = async () => {
  const posts = await getPosts();
  const paths = posts.map(post => ({ params: { slug: post.slug } }));
  return { paths, fallback: 'blocking' };
};

export const getStaticProps: GetStaticProps = async ({ params }) => {
  const post = await getPost(params!.slug as string);
  
  if (!post) {
    return { notFound: true };
  }
  
  return { 
    props: { post },
    revalidate: 3600,
  };
};

// After (src/app/blog/[slug]/page.tsx)
import { notFound } from 'next/navigation';

interface Props {
  params: { slug: string };
}

export default async function BlogPost({ params }: Props) {
  const post = await getPost(params.slug);
  
  if (!post) {
    notFound();
  }
  
  return (
    <article>
      <h1>{post.title}</h1>
      <div dangerouslySetInnerHTML={{ __html: post.content }} />
    </article>
  );
}

export async function generateStaticParams() {
  const posts = await getPosts();
  return posts.map(post => ({ slug: post.slug }));
}

export const revalidate = 3600;
```

### 3. Layout Migration

```tsx
// Before (pages/_app.tsx)
import type { AppProps } from 'next/app';
import Layout from '../components/Layout';
import '../styles/globals.css';

export default function MyApp({ Component, pageProps }: AppProps) {
  return (
    <Layout>
      <Component {...pageProps} />
    </Layout>
  );
}

// After (src/app/layout.tsx)
import './globals.css';

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body>
        <header>Header</header>
        <main>{children}</main>
        <footer>Footer</footer>
      </body>
    </html>
  );
}
```

### 4. API Routes Migration

```typescript
// Before (pages/api/posts/[id].ts)
import type { NextApiRequest, NextApiResponse } from 'next';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  const { id } = req.query;
  
  if (req.method === 'GET') {
    const post = await getPost(id as string);
    res.json(post);
  } else if (req.method === 'PUT') {
    const updatedPost = await updatePost(id as string, req.body);
    res.json(updatedPost);
  } else {
    res.setHeader('Allow', ['GET', 'PUT']);
    res.status(405).end(`Method ${req.method} Not Allowed`);
  }
}

// After (src/app/api/posts/[id]/route.ts)
import { NextRequest, NextResponse } from 'next/server';

interface Context {
  params: { id: string };
}

export async function GET(request: NextRequest, context: Context) {
  const post = await getPost(context.params.id);
  return NextResponse.json(post);
}

export async function PUT(request: NextRequest, context: Context) {
  const body = await request.json();
  const updatedPost = await updatePost(context.params.id, body);
  return NextResponse.json(updatedPost);
}
```

## Common Migration Challenges and Solutions

### 1. Client-Side Code Migration

**Challenge:** Converting client-side components
```tsx
// Pages Router - automatically client-side
import { useState, useEffect } from 'react';

export default function Counter() {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    console.log('Component mounted');
  }, []);
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>
        Increment
      </button>
    </div>
  );
}
```

**Solution:** Add 'use client' directive
```tsx
// App Router - explicit client component
'use client';

import { useState, useEffect } from 'react';

export default function Counter() {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    console.log('Component mounted');
  }, []);
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>
        Increment
      </button>
    </div>
  );
}
```

### 2. Router Hook Migration

```tsx
// Before (Pages Router)
import { useRouter } from 'next/router';

export default function MyComponent() {
  const router = useRouter();
  const { query, pathname } = router;
  
  const handleNavigation = () => {
    router.push('/new-page');
  };
  
  return <button onClick={handleNavigation}>Navigate</button>;
}

// After (App Router)
'use client';

import { useRouter, usePathname, useSearchParams } from 'next/navigation';

export default function MyComponent() {
  const router = useRouter();
  const pathname = usePathname();
  const searchParams = useSearchParams();
  
  const handleNavigation = () => {
    router.push('/new-page');
  };
  
  return <button onClick={handleNavigation}>Navigate</button>;
}
```

### 3. Data Fetching Migration

```tsx
// Before (Pages Router)
export default function ProductPage({ product }: { product: Product }) {
  return <ProductDetail product={product} />;
}

export const getServerSideProps: GetServerSideProps = async ({ params }) => {
  const product = await getProduct(params!.id as string);
  return { props: { product } };
};

// After (App Router)
interface Props {
  params: { id: string };
}

export default async function ProductPage({ params }: Props) {
  const product = await getProduct(params.id);
  return <ProductDetail product={product} />;
}
```

## Performance Considerations

### App Router Advantages

1. **Server Components**: Reduce client-side JavaScript bundle
2. **Streaming**: Improved perceived performance with streaming
3. **Automatic Code Splitting**: More granular code splitting
4. **Built-in Suspense**: Better loading states

```tsx
// App Router - Server Component (no JS sent to client)
export default async function ProductList() {
  const products = await getProducts();
  
  return (
    <div>
      {products.map(product => (
        <ProductCard key={product.id} product={product} />
      ))}
    </div>
  );
}

// Only interactive parts need 'use client'
'use client';

export function AddToCartButton({ productId }: { productId: string }) {
  const handleClick = () => {
    // Client-side logic
  };
  
  return <button onClick={handleClick}>Add to Cart</button>;
}
```

### Pages Router Strengths

1. **Predictable**: Well-understood patterns
2. **Ecosystem**: Mature ecosystem support
3. **Documentation**: Extensive documentation and examples
4. **Stability**: Production-tested in many applications

## Best Practices for Both Routers

### Pages Router Best Practices

```tsx
// 1. Optimize data fetching
export const getStaticProps: GetStaticProps = async () => {
  const [posts, categories] = await Promise.all([
    getPosts(),
    getCategories(),
  ]);
  
  return {
    props: { posts, categories },
    revalidate: 3600, // ISR
  };
};

// 2. Use proper TypeScript types
interface PageProps {
  posts: Post[];
  categories: Category[];
}

// 3. Implement error boundaries
class ErrorBoundary extends React.Component {
  // Error boundary implementation
}
```

### App Router Best Practices

```tsx
// 1. Minimize client components
// Server component by default
export default async function Page() {
  const data = await fetchData();
  return <ServerSideContent data={data} />;
}

// 2. Use loading and error boundaries
// loading.tsx
export default function Loading() {
  return <Skeleton />;
}

// error.tsx
'use client';

export default function Error({ error, reset }: {
  error: Error;
  reset: () => void;
}) {
  return (
    <div>
      <h2>Something went wrong!</h2>
      <button onClick={reset}>Try again</button>
    </div>
  );
}

// 3. Optimize metadata
export async function generateMetadata({ params }: Props): Promise<Metadata> {
  const post = await getPost(params.slug);
  
  return {
    title: post.title,
    description: post.excerpt,
  };
}
```

## Migration Checklist

### Pre-Migration Assessment

- [ ] **Audit current pages** and identify complexity
- [ ] **Check dependencies** for App Router compatibility
- [ ] **Plan migration phases** (static pages first)
- [ ] **Set up development environment** with both routers
- [ ] **Create migration timeline** and milestones

### During Migration

- [ ] **Start with simple pages** (about, contact)
- [ ] **Migrate layouts** and shared components
- [ ] **Convert data fetching** patterns
- [ ] **Update navigation** and routing logic
- [ ] **Test thoroughly** at each phase

### Post-Migration

- [ ] **Remove Pages Router** files
- [ ] **Update documentation** and team knowledge
- [ ] **Monitor performance** metrics
- [ ] **Gather feedback** from development team
- [ ] **Plan for ongoing optimization**

## Summary

Understanding the differences between App Router and Pages Router is crucial for Next.js development:

### ✅ Key Differences:
- **Architecture**: Server Components vs traditional React
- **Data Fetching**: Simplified async/await vs specific methods
- **Layouts**: Native nested layouts vs custom `_app.tsx`
- **Performance**: Built-in optimizations vs manual optimization

### ✅ Migration Strategy:
- **Gradual approach** works best for existing applications
- **Start simple** with static pages
- **Test thoroughly** at each migration phase
- **Plan for team training** on new patterns

### ✅ Decision Factors:
- **New projects**: Choose App Router
- **Existing projects**: Evaluate migration cost vs benefits
- **Team expertise**: Consider learning curve
- **Project requirements**: Assess feature needs

### Key Takeaways:
1. **App Router is the future** of Next.js development
2. **Both routers can coexist** during migration
3. **Server Components** provide significant performance benefits
4. **Migration requires planning** but offers long-term benefits
5. **Choose based on project needs** and team capacity

Your understanding of both routing systems is now comprehensive! In the next guide, we'll explore components and layouts in detail.

## Practice Exercise

Create a simple blog application using both routing approaches:

1. **Build with Pages Router** first
2. **Migrate to App Router** page by page
3. **Compare performance** between both approaches
4. **Document lessons learned** during migration

## Next Steps

Continue to [Components and Layouts](./06-Components-and-Layouts.md) to learn how to build reusable components and create effective layout patterns in Next.js.
