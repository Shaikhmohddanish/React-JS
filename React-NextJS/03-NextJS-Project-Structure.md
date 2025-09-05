# Next.js Project Structure - A Complete Guide

## Introduction to Next.js Project Structure

Understanding the project structure is crucial for building maintainable and scalable Next.js applications. Next.js follows specific conventions and provides flexibility in organizing your code. This guide will walk you through every aspect of Next.js project organization, from the basic file structure to advanced patterns.

## Default Project Structure

When you create a new Next.js project with `create-next-app`, you get this structure:

```
my-nextjs-app/
├── .eslintrc.json          # ESLint configuration
├── .gitignore             # Git ignore rules
├── README.md              # Project documentation
├── next.config.js         # Next.js configuration
├── package.json           # Node.js dependencies and scripts
├── tailwind.config.js     # Tailwind CSS configuration
├── tsconfig.json          # TypeScript configuration
├── public/                # Static assets
│   ├── next.svg
│   ├── vercel.svg
│   └── favicon.ico
└── src/                   # Source code (optional but recommended)
    └── app/               # App Router (Next.js 13+)
        ├── favicon.ico
        ├── globals.css
        ├── layout.tsx
        ├── page.tsx
        └── loading.tsx
```

## Core Directories and Files

### 1. The `app` Directory (App Router)

The `app` directory is the modern way to structure Next.js applications (introduced in Next.js 13):

```
src/app/
├── layout.tsx             # Root layout component
├── page.tsx              # Home page
├── loading.tsx           # Loading UI
├── error.tsx             # Error UI
├── not-found.tsx         # 404 page
├── globals.css           # Global styles
├── about/                # About page route
│   ├── page.tsx         # /about
│   └── loading.tsx      # Loading UI for /about
├── blog/                 # Blog section
│   ├── page.tsx         # /blog
│   ├── layout.tsx       # Blog layout
│   └── [slug]/          # Dynamic route
│       ├── page.tsx     # /blog/[slug]
│       └── loading.tsx  # Loading UI for blog posts
├── api/                  # API routes
│   ├── auth/
│   │   └── route.ts     # /api/auth
│   └── posts/
│       ├── route.ts     # /api/posts
│       └── [id]/
│           └── route.ts # /api/posts/[id]
└── (dashboard)/          # Route groups
    ├── analytics/
    │   └── page.tsx     # /analytics
    └── settings/
        └── page.tsx     # /settings
```

### 2. The `pages` Directory (Pages Router)

The traditional Pages Router structure (still supported):

```
pages/
├── _app.tsx              # Custom App component
├── _document.tsx         # Custom Document
├── _error.tsx            # Custom Error page
├── index.tsx             # Home page (/)
├── about.tsx             # About page (/about)
├── blog/
│   ├── index.tsx         # Blog index (/blog)
│   └── [slug].tsx        # Dynamic route (/blog/post-1)
├── api/                  # API routes
│   ├── auth.ts          # /api/auth
│   └── posts/
│       ├── index.ts     # /api/posts
│       └── [id].ts      # /api/posts/123
└── 404.tsx              # Custom 404 page
```

### 3. The `public` Directory

Static assets that are served directly:

```
public/
├── favicon.ico           # Favicon
├── robots.txt           # SEO robots file
├── sitemap.xml          # SEO sitemap
├── manifest.json        # PWA manifest
├── images/              # Image assets
│   ├── hero.jpg
│   ├── logo.png
│   └── icons/
│       ├── icon-192.png
│       └── icon-512.png
├── videos/              # Video assets
└── documents/           # Documents (PDFs, etc.)
```

### 4. The `src` Directory (Optional but Recommended)

Using the `src` directory helps separate source code from configuration:

```
src/
├── app/                 # App Router files
├── components/          # Reusable components
├── lib/                # Utility functions
├── hooks/              # Custom React hooks
├── types/              # TypeScript type definitions
├── styles/             # Stylesheets
├── utils/              # Helper functions
├── store/              # State management
└── middleware.ts       # Next.js middleware
```

## App Router File Conventions

### Special Files in App Router

```tsx
// app/layout.tsx - Root Layout (Required)
import type { Metadata } from 'next';
import { Inter } from 'next/font/google';
import './globals.css';

const inter = Inter({ subsets: ['latin'] });

export const metadata: Metadata = {
  title: 'My Next.js App',
  description: 'Built with Next.js and TypeScript',
};

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body className={inter.className}>
        <header>
          <nav>Navigation</nav>
        </header>
        <main>{children}</main>
        <footer>Footer</footer>
      </body>
    </html>
  );
}
```

```tsx
// app/page.tsx - Home Page
export default function HomePage() {
  return (
    <div>
      <h1>Welcome to My Next.js App</h1>
      <p>This is the home page.</p>
    </div>
  );
}
```

```tsx
// app/loading.tsx - Loading UI
export default function Loading() {
  return (
    <div className="flex items-center justify-center min-h-screen">
      <div className="animate-spin rounded-full h-32 w-32 border-b-2 border-gray-900"></div>
    </div>
  );
}
```

```tsx
// app/error.tsx - Error UI
'use client';

import { useEffect } from 'react';

export default function Error({
  error,
  reset,
}: {
  error: Error & { digest?: string };
  reset: () => void;
}) {
  useEffect(() => {
    console.error(error);
  }, [error]);

  return (
    <div className="flex flex-col items-center justify-center min-h-screen">
      <h2 className="text-2xl font-bold mb-4">Something went wrong!</h2>
      <button
        onClick={() => reset()}
        className="px-4 py-2 bg-blue-500 text-white rounded hover:bg-blue-600"
      >
        Try again
      </button>
    </div>
  );
}
```

```tsx
// app/not-found.tsx - 404 Page
import Link from 'next/link';

export default function NotFound() {
  return (
    <div className="flex flex-col items-center justify-center min-h-screen">
      <h2 className="text-4xl font-bold mb-4">Not Found</h2>
      <p className="mb-4">Could not find the requested resource</p>
      <Link 
        href="/"
        className="px-4 py-2 bg-blue-500 text-white rounded hover:bg-blue-600"
      >
        Return Home
      </Link>
    </div>
  );
}
```

### Route Groups and Organization

Use parentheses to group routes without affecting the URL:

```
app/
├── (marketing)/          # Route group
│   ├── layout.tsx       # Layout for marketing pages
│   ├── about/
│   │   └── page.tsx     # /about
│   └── contact/
│       └── page.tsx     # /contact
├── (dashboard)/          # Another route group
│   ├── layout.tsx       # Layout for dashboard pages
│   ├── analytics/
│   │   └── page.tsx     # /analytics
│   └── settings/
│       └── page.tsx     # /settings
└── layout.tsx           # Root layout
```

```tsx
// app/(marketing)/layout.tsx
export default function MarketingLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <div className="marketing-layout">
      <nav className="marketing-nav">
        {/* Marketing navigation */}
      </nav>
      {children}
    </div>
  );
}
```

```tsx
// app/(dashboard)/layout.tsx
export default function DashboardLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <div className="dashboard-layout">
      <aside className="sidebar">
        {/* Dashboard sidebar */}
      </aside>
      <main className="main-content">
        {children}
      </main>
    </div>
  );
}
```

## Components Organization

### Recommended Component Structure

```
src/components/
├── ui/                   # Basic UI components
│   ├── Button/
│   │   ├── Button.tsx
│   │   ├── Button.test.tsx
│   │   ├── Button.stories.tsx
│   │   ├── Button.module.css
│   │   └── index.ts
│   ├── Input/
│   ├── Modal/
│   └── index.ts         # Barrel exports
├── layout/               # Layout components
│   ├── Header/
│   │   ├── Header.tsx
│   │   ├── Navigation.tsx
│   │   └── index.ts
│   ├── Footer/
│   └── Sidebar/
├── forms/                # Form components
│   ├── ContactForm/
│   ├── LoginForm/
│   └── SearchForm/
├── features/             # Feature-specific components
│   ├── auth/
│   │   ├── LoginForm.tsx
│   │   ├── SignupForm.tsx
│   │   └── UserProfile.tsx
│   ├── blog/
│   │   ├── PostCard.tsx
│   │   ├── PostList.tsx
│   │   └── PostDetail.tsx
│   └── dashboard/
└── common/               # Shared components
    ├── LoadingSpinner.tsx
    ├── ErrorBoundary.tsx
    └── SEOHead.tsx
```

### Component File Example

```tsx
// src/components/ui/Button/Button.tsx
import React from 'react';
import { cva, type VariantProps } from 'class-variance-authority';
import { cn } from '@/lib/utils';

const buttonVariants = cva(
  'inline-flex items-center justify-center rounded-md text-sm font-medium transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 disabled:opacity-50 disabled:pointer-events-none ring-offset-background',
  {
    variants: {
      variant: {
        default: 'bg-primary text-primary-foreground hover:bg-primary/90',
        destructive: 'bg-destructive text-destructive-foreground hover:bg-destructive/90',
        outline: 'border border-input hover:bg-accent hover:text-accent-foreground',
        secondary: 'bg-secondary text-secondary-foreground hover:bg-secondary/80',
        ghost: 'hover:bg-accent hover:text-accent-foreground',
        link: 'underline-offset-4 hover:underline text-primary',
      },
      size: {
        default: 'h-10 py-2 px-4',
        sm: 'h-9 px-3 rounded-md',
        lg: 'h-11 px-8 rounded-md',
        icon: 'h-10 w-10',
      },
    },
    defaultVariants: {
      variant: 'default',
      size: 'default',
    },
  }
);

export interface ButtonProps
  extends React.ButtonHTMLAttributes<HTMLButtonElement>,
    VariantProps<typeof buttonVariants> {
  asChild?: boolean;
}

const Button = React.forwardRef<HTMLButtonElement, ButtonProps>(
  ({ className, variant, size, asChild = false, ...props }, ref) => {
    return (
      <button
        className={cn(buttonVariants({ variant, size, className }))}
        ref={ref}
        {...props}
      />
    );
  }
);

Button.displayName = 'Button';

export { Button, buttonVariants };
```

```ts
// src/components/ui/Button/index.ts
export { Button, buttonVariants, type ButtonProps } from './Button';
```

```ts
// src/components/ui/index.ts
export { Button, type ButtonProps } from './Button';
export { Input, type InputProps } from './Input';
export { Modal, type ModalProps } from './Modal';
```

## Utility and Library Organization

### Lib Directory Structure

```
src/lib/
├── auth.ts              # Authentication utilities
├── db.ts                # Database connection
├── email.ts             # Email utilities
├── env.ts               # Environment validation
├── stripe.ts            # Payment processing
├── utils.ts             # General utilities
├── validations.ts       # Zod schemas
├── constants.ts         # Application constants
└── types.ts             # Shared TypeScript types
```

### Example Utility Files

```ts
// src/lib/utils.ts
import { type ClassValue, clsx } from 'clsx';
import { twMerge } from 'tailwind-merge';

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}

export function formatDate(date: Date): string {
  return new Intl.DateTimeFormat('en-US', {
    year: 'numeric',
    month: 'long',
    day: 'numeric',
  }).format(date);
}

export function slugify(text: string): string {
  return text
    .toLowerCase()
    .replace(/[^\w ]+/g, '')
    .replace(/ +/g, '-');
}

export function capitalize(text: string): string {
  return text.charAt(0).toUpperCase() + text.slice(1);
}
```

```ts
// src/lib/validations.ts
import { z } from 'zod';

export const userSchema = z.object({
  name: z.string().min(2, 'Name must be at least 2 characters'),
  email: z.string().email('Please enter a valid email'),
  password: z.string().min(8, 'Password must be at least 8 characters'),
});

export const postSchema = z.object({
  title: z.string().min(1, 'Title is required'),
  content: z.string().min(10, 'Content must be at least 10 characters'),
  slug: z.string().min(1, 'Slug is required'),
  published: z.boolean().default(false),
});

export type User = z.infer<typeof userSchema>;
export type Post = z.infer<typeof postSchema>;
```

```ts
// src/lib/constants.ts
export const APP_CONFIG = {
  name: 'My Next.js App',
  description: 'A modern web application built with Next.js',
  url: process.env.NEXT_PUBLIC_APP_URL || 'http://localhost:3000',
  author: {
    name: 'Your Name',
    email: 'your.email@example.com',
    twitter: '@yourusername',
  },
} as const;

export const API_ENDPOINTS = {
  posts: '/api/posts',
  users: '/api/users',
  auth: '/api/auth',
} as const;

export const ROUTES = {
  home: '/',
  about: '/about',
  blog: '/blog',
  contact: '/contact',
  login: '/auth/login',
  signup: '/auth/signup',
} as const;
```

## API Routes Organization

### API Structure

```
src/app/api/
├── auth/
│   ├── login/
│   │   └── route.ts     # POST /api/auth/login
│   ├── logout/
│   │   └── route.ts     # POST /api/auth/logout
│   └── signup/
│       └── route.ts     # POST /api/auth/signup
├── posts/
│   ├── route.ts         # GET, POST /api/posts
│   └── [id]/
│       └── route.ts     # GET, PUT, DELETE /api/posts/[id]
├── users/
│   ├── route.ts         # GET, POST /api/users
│   ├── [id]/
│   │   └── route.ts     # GET, PUT, DELETE /api/users/[id]
│   └── profile/
│       └── route.ts     # GET, PUT /api/users/profile
└── upload/
    └── route.ts         # POST /api/upload
```

### API Route Examples

```ts
// src/app/api/posts/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { db } from '@/lib/db';
import { postSchema } from '@/lib/validations';

export async function GET(request: NextRequest) {
  try {
    const { searchParams } = new URL(request.url);
    const page = parseInt(searchParams.get('page') || '1');
    const limit = parseInt(searchParams.get('limit') || '10');
    const skip = (page - 1) * limit;

    const posts = await db.post.findMany({
      skip,
      take: limit,
      include: {
        author: {
          select: {
            name: true,
            email: true,
          },
        },
      },
      orderBy: {
        createdAt: 'desc',
      },
    });

    const total = await db.post.count();

    return NextResponse.json({
      posts,
      pagination: {
        page,
        limit,
        total,
        pages: Math.ceil(total / limit),
      },
    });
  } catch (error) {
    console.error('Error fetching posts:', error);
    return NextResponse.json(
      { error: 'Internal Server Error' },
      { status: 500 }
    );
  }
}

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const validatedData = postSchema.parse(body);

    const post = await db.post.create({
      data: validatedData,
    });

    return NextResponse.json(post, { status: 201 });
  } catch (error) {
    if (error instanceof z.ZodError) {
      return NextResponse.json(
        { error: 'Validation error', details: error.errors },
        { status: 400 }
      );
    }

    console.error('Error creating post:', error);
    return NextResponse.json(
      { error: 'Internal Server Error' },
      { status: 500 }
    );
  }
}
```

## Configuration Files

### Next.js Configuration

```javascript
// next.config.js
/** @type {import('next').NextConfig} */
const nextConfig = {
  // Enable experimental features
  experimental: {
    appDir: true,
  },
  
  // Image optimization
  images: {
    domains: ['example.com', 'images.unsplash.com'],
    formats: ['image/webp', 'image/avif'],
  },
  
  // Environment variables
  env: {
    CUSTOM_KEY: process.env.CUSTOM_KEY,
  },
  
  // Webpack configuration
  webpack: (config, { buildId, dev, isServer, defaultLoaders, webpack }) => {
    // Custom webpack rules
    config.module.rules.push({
      test: /\\.svg$/,
      use: ['@svgr/webpack'],
    });
    
    return config;
  },
  
  // Redirects
  async redirects() {
    return [
      {
        source: '/old-blog/:slug',
        destination: '/blog/:slug',
        permanent: true,
      },
    ];
  },
  
  // Rewrites
  async rewrites() {
    return [
      {
        source: '/api/v1/:path*',
        destination: '/api/:path*',
      },
    ];
  },
  
  // Headers
  async headers() {
    return [
      {
        source: '/api/:path*',
        headers: [
          {
            key: 'Access-Control-Allow-Origin',
            value: '*',
          },
          {
            key: 'Access-Control-Allow-Methods',
            value: 'GET, POST, PUT, DELETE, OPTIONS',
          },
          {
            key: 'Access-Control-Allow-Headers',
            value: 'Content-Type, Authorization',
          },
        ],
      },
    ];
  },
};

module.exports = nextConfig;
```

### TypeScript Configuration

```json
{
  "compilerOptions": {
    "target": "es5",
    "lib": ["dom", "dom.iterable", "es6"],
    "allowJs": true,
    "skipLibCheck": true,
    "strict": true,
    "noEmit": true,
    "esModuleInterop": true,
    "module": "esnext",
    "moduleResolution": "bundler",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "jsx": "preserve",
    "incremental": true,
    "plugins": [
      {
        "name": "next"
      }
    ],
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"],
      "@/components/*": ["./src/components/*"],
      "@/lib/*": ["./src/lib/*"],
      "@/hooks/*": ["./src/hooks/*"],
      "@/types/*": ["./src/types/*"],
      "@/styles/*": ["./src/styles/*"],
      "@/utils/*": ["./src/utils/*"]
    }
  },
  "include": [
    "next-env.d.ts",
    "**/*.ts",
    "**/*.tsx",
    ".next/types/**/*.ts"
  ],
  "exclude": ["node_modules"]
}
```

## Advanced Project Organization

### Feature-Based Structure

For larger applications, consider organizing by features:

```
src/
├── app/                 # App Router
├── features/            # Feature modules
│   ├── auth/
│   │   ├── components/
│   │   │   ├── LoginForm.tsx
│   │   │   └── SignupForm.tsx
│   │   ├── hooks/
│   │   │   ├── useAuth.ts
│   │   │   └── useLogin.ts
│   │   ├── services/
│   │   │   └── authService.ts
│   │   ├── types/
│   │   │   └── auth.types.ts
│   │   └── utils/
│   │       └── validation.ts
│   ├── blog/
│   │   ├── components/
│   │   ├── hooks/
│   │   ├── services/
│   │   └── types/
│   └── dashboard/
├── shared/              # Shared resources
│   ├── components/
│   ├── hooks/
│   ├── services/
│   ├── types/
│   └── utils/
├── lib/                 # Core utilities
└── styles/              # Global styles
```

### Monorepo Structure

For complex projects with multiple applications:

```
packages/
├── web/                 # Next.js app
│   ├── src/
│   ├── package.json
│   └── next.config.js
├── admin/               # Admin dashboard
│   ├── src/
│   ├── package.json
│   └── next.config.js
├── api/                 # Standalone API
│   ├── src/
│   └── package.json
├── ui/                  # Shared UI components
│   ├── src/
│   └── package.json
├── utils/               # Shared utilities
│   ├── src/
│   └── package.json
├── package.json         # Root package.json
└── turbo.json          # Turborepo config
```

## Best Practices for Project Structure

### 1. Consistent Naming Conventions

```typescript
// Use PascalCase for components
const UserProfile = () => {};
const BlogPost = () => {};

// Use camelCase for variables and functions
const userData = {};
const fetchUserData = () => {};

// Use kebab-case for file names
// user-profile.tsx
// blog-post.tsx

// Use SCREAMING_SNAKE_CASE for constants
const API_BASE_URL = 'https://api.example.com';
const MAX_FILE_SIZE = 1024 * 1024;
```

### 2. Barrel Exports

Use index files to create clean import statements:

```ts
// components/ui/index.ts
export { Button } from './Button';
export { Input } from './Input';
export { Modal } from './Modal';

// Usage
import { Button, Input, Modal } from '@/components/ui';
```

### 3. Type Organization

```typescript
// types/api.ts
export interface ApiResponse<T> {
  data: T;
  message: string;
  success: boolean;
}

export interface PaginatedResponse<T> extends ApiResponse<T[]> {
  pagination: {
    page: number;
    limit: number;
    total: number;
    pages: number;
  };
}

// types/user.ts
export interface User {
  id: string;
  name: string;
  email: string;
  avatar?: string;
  createdAt: Date;
  updatedAt: Date;
}

export interface CreateUserRequest {
  name: string;
  email: string;
  password: string;
}

export interface UpdateUserRequest {
  name?: string;
  email?: string;
  avatar?: string;
}
```

### 4. Environment Configuration

```typescript
// lib/env.ts
import { z } from 'zod';

const envSchema = z.object({
  NODE_ENV: z.enum(['development', 'test', 'production']),
  NEXT_PUBLIC_APP_URL: z.string().url(),
  DATABASE_URL: z.string(),
  NEXTAUTH_SECRET: z.string(),
  NEXTAUTH_URL: z.string().url(),
});

export const env = envSchema.parse(process.env);
```

## Migration Between Structures

### From Pages to App Router

```typescript
// Before (Pages Router)
// pages/blog/[slug].tsx
import { GetStaticProps, GetStaticPaths } from 'next';

interface Props {
  post: Post;
}

export default function BlogPost({ post }: Props) {
  return <div>{post.title}</div>;
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

// After (App Router)
// app/blog/[slug]/page.tsx
interface Props {
  params: { slug: string };
}

export default async function BlogPost({ params }: Props) {
  const post = await getPost(params.slug);
  return <div>{post.title}</div>;
}

export async function generateStaticParams() {
  const posts = await getPosts();
  return posts.map((post) => ({ slug: post.slug }));
}
```

## Summary

Understanding Next.js project structure is essential for building maintainable applications. Here's what we covered:

### ✅ Core Concepts:
- **App Router vs Pages Router** structure differences
- **File-based routing** conventions
- **Special files** and their purposes
- **Route groups** and organization patterns

### ✅ Organization Patterns:
- **Component organization** by type and feature
- **API routes** structure and conventions
- **Utility and library** organization
- **Configuration files** and their roles

### ✅ Best Practices:
- **Consistent naming** conventions
- **Barrel exports** for clean imports
- **Type organization** for TypeScript projects
- **Environment configuration** management

### ✅ Advanced Patterns:
- **Feature-based** organization for large apps
- **Monorepo structure** for complex projects
- **Migration strategies** between routing systems

### Key Takeaways:
1. **Choose the right structure** based on your project size and complexity
2. **Be consistent** with naming and organization patterns
3. **Use the src directory** for better code organization
4. **Leverage TypeScript** for better development experience
5. **Plan for scalability** from the beginning

Your Next.js project is now well-organized and ready for development! In the next guide, we'll explore pages and routing in detail.

## Practice Exercise

Create a Next.js project with the following structure:

1. Set up an App Router project with proper folder organization
2. Create multiple pages with layouts
3. Add API routes for a simple blog
4. Organize components by feature
5. Set up proper TypeScript types and utilities

## Next Steps

Continue to [Pages and Routing](./04-Pages-and-Routing.md) to learn how Next.js handles routing and page creation.
