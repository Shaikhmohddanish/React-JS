# Setting Up Next.js Environment - A Complete Guide

## Prerequisites

Before setting up Next.js, ensure you have the following installed on your development machine:

### Required Software

1. **Node.js** (version 18.17 or later)
2. **npm** (comes with Node.js) or **yarn** or **pnpm**
3. **Code Editor** (VS Code recommended)
4. **Git** (for version control)

### Checking Your Setup

Let's verify your environment is ready:

```bash
# Check Node.js version
node --version
# Should output v18.17.0 or higher

# Check npm version
npm --version
# Should output 9.0.0 or higher

# Check Git version
git --version
# Should output git version 2.x.x
```

If any of these commands fail or show older versions, you'll need to install or update them.

## Installing Node.js

### Option 1: Official Installer (Recommended for Beginners)

1. Visit [nodejs.org](https://nodejs.org/)
2. Download the LTS (Long Term Support) version
3. Run the installer and follow the setup wizard
4. Restart your terminal and verify installation

### Option 2: Using Node Version Manager (Recommended for Developers)

**On macOS/Linux:**
```bash
# Install nvm
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash

# Restart terminal or run:
source ~/.bashrc

# Install latest LTS Node.js
nvm install --lts
nvm use --lts
```

**On Windows:**
```bash
# Install nvm-windows from GitHub releases
# Then in Command Prompt:
nvm install lts
nvm use lts
```

## Creating Your First Next.js Application

### Method 1: Using create-next-app (Recommended)

The easiest way to start a Next.js project:

```bash
# Create a new Next.js app with TypeScript
npx create-next-app@latest my-nextjs-app

# You'll be prompted with configuration options:
# ✅ Would you like to use TypeScript? → Yes
# ✅ Would you like to use ESLint? → Yes  
# ✅ Would you like to use Tailwind CSS? → Yes
# ✅ Would you like to use `src/` directory? → Yes
# ✅ Would you like to use App Router? → Yes
# ✅ Would you like to customize the default import alias? → No

# Navigate to your project
cd my-nextjs-app

# Start the development server
npm run dev
```

### Method 2: Manual Setup

For a deeper understanding, let's set up Next.js manually:

```bash
# Create project directory
mkdir my-nextjs-app
cd my-nextjs-app

# Initialize package.json
npm init -y

# Install Next.js, React, and React DOM
npm install next react react-dom

# Install TypeScript and types (optional but recommended)
npm install -D typescript @types/react @types/node
```

Create the basic file structure:

```bash
# Create necessary directories
mkdir pages public styles

# Create basic files
touch pages/index.tsx
touch pages/_app.tsx
touch package.json
```

Update your `package.json` scripts:

```json
{
  "name": "my-nextjs-app",
  "version": "0.1.0",
  "private": true,
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint"
  },
  "dependencies": {
    "next": "14.0.0",
    "react": "18.2.0",
    "react-dom": "18.2.0"
  },
  "devDependencies": {
    "@types/node": "20.8.0",
    "@types/react": "18.2.0",
    "@types/react-dom": "18.2.0",
    "typescript": "5.2.0"
  }
}
```

## Project Structure Overview

Here's what a typical Next.js project structure looks like:

```
my-nextjs-app/
├── .next/                 # Build output (auto-generated)
├── .git/                  # Git repository
├── public/                # Static assets
│   ├── favicon.ico
│   ├── images/
│   └── icons/
├── src/                   # Source code (optional but recommended)
│   ├── app/              # App Router (Next.js 13+)
│   │   ├── layout.tsx
│   │   ├── page.tsx
│   │   ├── globals.css
│   │   └── favicon.ico
│   ├── components/       # Reusable components
│   │   ├── ui/          # UI components
│   │   └── layout/      # Layout components
│   ├── lib/             # Utility functions
│   ├── hooks/           # Custom React hooks
│   └── styles/          # Global styles
├── .env.local           # Environment variables
├── .env.example         # Environment variables template
├── .gitignore          # Git ignore rules
├── next.config.js      # Next.js configuration
├── package.json        # Project dependencies
├── README.md           # Project documentation
├── tailwind.config.js  # Tailwind CSS configuration
└── tsconfig.json       # TypeScript configuration
```

## Essential Configuration Files

### 1. Next.js Configuration (next.config.js)

```javascript
/** @type {import('next').NextConfig} */
const nextConfig = {
  // Enable experimental features
  experimental: {
    appDir: true, // Enable App Router
  },
  
  // Image optimization
  images: {
    domains: ['example.com', 'api.example.com'],
    formats: ['image/webp', 'image/avif'],
  },
  
  // Environment variables
  env: {
    CUSTOM_KEY: process.env.CUSTOM_KEY,
  },
  
  // Redirects
  async redirects() {
    return [
      {
        source: '/old-path',
        destination: '/new-path',
        permanent: true,
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
        ],
      },
    ];
  },
};

module.exports = nextConfig;
```

### 2. TypeScript Configuration (tsconfig.json)

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
      "@/styles/*": ["./src/styles/*"]
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

### 3. Environment Variables (.env.local)

```bash
# .env.local - Local environment variables (not committed to Git)
NEXT_PUBLIC_API_URL=http://localhost:3001
DATABASE_URL=postgresql://username:password@localhost:5432/mydb
NEXTAUTH_SECRET=your-secret-key
NEXTAUTH_URL=http://localhost:3000

# .env.example - Template for environment variables (committed to Git)
NEXT_PUBLIC_API_URL=
DATABASE_URL=
NEXTAUTH_SECRET=
NEXTAUTH_URL=
```

### 4. Git Ignore (.gitignore)

```bash
# Dependencies
/node_modules
/.pnp
.pnp.js

# Testing
/coverage

# Next.js
/.next/
/out/

# Production
/build

# Misc
.DS_Store
*.pem

# Debug
npm-debug.log*
yarn-debug.log*
yarn-error.log*

# Local env files
.env*.local

# Vercel
.vercel

# TypeScript
*.tsbuildinfo
next-env.d.ts

# IDE
.vscode/
.idea/
```

## Development Tools Setup

### 1. VS Code Extensions

Install these essential VS Code extensions:

```json
{
  "recommendations": [
    "bradlc.vscode-tailwindcss",
    "esbenp.prettier-vscode", 
    "ms-vscode.vscode-typescript-next",
    "formulahendry.auto-rename-tag",
    "christian-kohler.path-intellisense",
    "ms-vscode.vscode-json",
    "PKief.material-icon-theme"
  ]
}
```

### 2. VS Code Settings

Create `.vscode/settings.json`:

```json
{
  "typescript.preferences.importModuleSpecifier": "relative",
  "typescript.suggest.autoImports": true,
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true
  },
  "emmet.includeLanguages": {
    "typescript": "html",
    "typescriptreact": "html"
  }
}
```

### 3. Prettier Configuration

Create `.prettierrc`:

```json
{
  "semi": true,
  "trailingComma": "es5",
  "singleQuote": true,
  "printWidth": 80,
  "tabWidth": 2,
  "useTabs": false
}
```

### 4. ESLint Configuration

Next.js comes with ESLint configured. Extend it in `.eslintrc.json`:

```json
{
  "extends": [
    "next/core-web-vitals",
    "next/typescript"
  ],
  "rules": {
    "react/no-unescaped-entities": "off",
    "@next/next/no-page-custom-font": "off"
  }
}
```

## Creating Your First Pages

### 1. Home Page (src/app/page.tsx)

```tsx
import Image from 'next/image';
import Link from 'next/link';

export default function Home() {
  return (
    <main className="flex min-h-screen flex-col items-center justify-between p-24">
      <div className="z-10 max-w-5xl w-full items-center justify-between font-mono text-sm lg:flex">
        <p className="fixed left-0 top-0 flex w-full justify-center border-b border-gray-300 bg-gradient-to-b from-zinc-200 pb-6 pt-8 backdrop-blur-2xl dark:border-neutral-800 dark:bg-zinc-800/30 dark:from-inherit lg:static lg:w-auto lg:rounded-xl lg:border lg:bg-gray-200 lg:p-4 lg:dark:bg-zinc-800/30">
          Welcome to&nbsp;
          <code className="font-mono font-bold">Next.js!</code>
        </p>
      </div>

      <div className="relative flex place-items-center before:absolute before:h-[300px] before:w-[480px] before:-translate-x-1/2 before:translate-y-1/4 before:rounded-full before:bg-gradient-radial before:from-white before:to-transparent before:blur-2xl before:content-[''] after:absolute after:-z-20 after:h-[180px] after:w-[240px] after:translate-x-1/3 after:bg-gradient-conic after:from-sky-200 after:via-blue-200 after:blur-2xl after:content-[''] before:dark:bg-gradient-to-br before:dark:from-transparent before:dark:to-blue-700 before:dark:opacity-10 after:dark:from-sky-900 after:dark:via-[#0141ff] after:dark:opacity-40 before:lg:h-[360px] z-[-1]">
        <Image
          className="relative dark:drop-shadow-[0_0_0.3rem_#ffffff70] dark:invert"
          src="/next.svg"
          alt="Next.js Logo"
          width={180}
          height={37}
          priority
        />
      </div>

      <div className="mb-32 grid text-center lg:max-w-5xl lg:w-full lg:mb-0 lg:grid-cols-4 lg:text-left">
        <Link
          href="/about"
          className="group rounded-lg border border-transparent px-5 py-4 transition-colors hover:border-gray-300 hover:bg-gray-100 hover:dark:border-neutral-700 hover:dark:bg-neutral-800/30"
        >
          <h2 className={`mb-3 text-2xl font-semibold`}>
            About{' '}
            <span className="inline-block transition-transform group-hover:translate-x-1 motion-reduce:transform-none">
              -&gt;
            </span>
          </h2>
          <p className={`m-0 max-w-[30ch] text-sm opacity-50`}>
            Learn more about this Next.js application.
          </p>
        </Link>

        <Link
          href="/blog"
          className="group rounded-lg border border-transparent px-5 py-4 transition-colors hover:border-gray-300 hover:bg-gray-100 hover:dark:border-neutral-700 hover:dark:bg-neutral-800/30"
        >
          <h2 className={`mb-3 text-2xl font-semibold`}>
            Blog{' '}
            <span className="inline-block transition-transform group-hover:translate-x-1 motion-reduce:transform-none">
              -&gt;
            </span>
          </h2>
          <p className={`m-0 max-w-[30ch] text-sm opacity-50`}>
            Read our latest blog posts and tutorials.
          </p>
        </Link>

        <Link
          href="/contact"
          className="group rounded-lg border border-transparent px-5 py-4 transition-colors hover:border-gray-300 hover:bg-gray-100 hover:dark:border-neutral-700 hover:dark:bg-neutral-800/30"
        >
          <h2 className={`mb-3 text-2xl font-semibold`}>
            Contact{' '}
            <span className="inline-block transition-transform group-hover:translate-x-1 motion-reduce:transform-none">
              -&gt;
            </span>
          </h2>
          <p className={`m-0 max-w-[30ch] text-sm opacity-50`}>
            Get in touch with our team.
          </p>
        </Link>

        <a
          href="https://nextjs.org/docs"
          className="group rounded-lg border border-transparent px-5 py-4 transition-colors hover:border-gray-300 hover:bg-gray-100 hover:dark:border-neutral-700 hover:dark:bg-neutral-800/30"
          target="_blank"
          rel="noopener noreferrer"
        >
          <h2 className={`mb-3 text-2xl font-semibold`}>
            Docs{' '}
            <span className="inline-block transition-transform group-hover:translate-x-1 motion-reduce:transform-none">
              -&gt;
            </span>
          </h2>
          <p className={`m-0 max-w-[30ch] text-sm opacity-50`}>
            Find in-depth information about Next.js features and API.
          </p>
        </a>
      </div>
    </main>
  );
}
```

### 2. Root Layout (src/app/layout.tsx)

```tsx
import type { Metadata } from 'next';
import { Inter } from 'next/font/google';
import './globals.css';

const inter = Inter({ subsets: ['latin'] });

export const metadata: Metadata = {
  title: {
    default: 'My Next.js App',
    template: '%s | My Next.js App'
  },
  description: 'A comprehensive Next.js application built with TypeScript and Tailwind CSS',
  keywords: ['Next.js', 'React', 'TypeScript', 'Tailwind CSS'],
  authors: [{ name: 'Your Name' }],
  creator: 'Your Name',
  openGraph: {
    type: 'website',
    locale: 'en_US',
    url: 'https://your-domain.com',
    title: 'My Next.js App',
    description: 'A comprehensive Next.js application',
    siteName: 'My Next.js App',
  },
  twitter: {
    card: 'summary_large_image',
    title: 'My Next.js App',
    description: 'A comprehensive Next.js application',
    creator: '@yourusername',
  },
  robots: {
    index: true,
    follow: true,
  },
};

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body className={inter.className}>
        <div className="min-h-screen bg-background font-sans antialiased">
          {children}
        </div>
      </body>
    </html>
  );
}
```

### 3. About Page (src/app/about/page.tsx)

```tsx
import { Metadata } from 'next';

export const metadata: Metadata = {
  title: 'About',
  description: 'Learn more about our Next.js application',
};

export default function About() {
  return (
    <div className="container mx-auto px-4 py-16">
      <h1 className="text-4xl font-bold mb-8">About Our Application</h1>
      
      <div className="prose prose-lg max-w-none">
        <p className="text-xl text-gray-600 mb-6">
          Welcome to our Next.js application! This project showcases the power
          of modern web development with React, TypeScript, and Tailwind CSS.
        </p>
        
        <h2 className="text-2xl font-semibold mb-4">Features</h2>
        <ul className="list-disc list-inside space-y-2 mb-6">
          <li>Server-side rendering for optimal SEO</li>
          <li>Static site generation for fast loading times</li>
          <li>TypeScript for type-safe development</li>
          <li>Tailwind CSS for rapid UI development</li>
          <li>Responsive design that works on all devices</li>
        </ul>
        
        <h2 className="text-2xl font-semibold mb-4">Technology Stack</h2>
        <div className="grid md:grid-cols-2 gap-6">
          <div>
            <h3 className="text-lg font-medium mb-2">Frontend</h3>
            <ul className="space-y-1">
              <li>Next.js 14</li>
              <li>React 18</li>
              <li>TypeScript</li>
              <li>Tailwind CSS</li>
            </ul>
          </div>
          
          <div>
            <h3 className="text-lg font-medium mb-2">Development Tools</h3>
            <ul className="space-y-1">
              <li>ESLint</li>
              <li>Prettier</li>
              <li>VS Code</li>
              <li>Git</li>
            </ul>
          </div>
        </div>
      </div>
    </div>
  );
}
```

## Development Workflow

### 1. Starting Development

```bash
# Start development server
npm run dev

# Your app will be available at:
# http://localhost:3000
```

### 2. Building for Production

```bash
# Build the application
npm run build

# Start production server
npm run start

# Analyze bundle size
npm run build -- --analyze
```

### 3. Linting and Formatting

```bash
# Run ESLint
npm run lint

# Fix ESLint errors
npm run lint -- --fix

# Format code with Prettier
npx prettier --write .
```

## Common Development Commands

### Package Management

```bash
# Install a new dependency
npm install package-name

# Install a development dependency
npm install -D package-name

# Update all packages
npm update

# Remove a package
npm uninstall package-name

# Check for outdated packages
npm outdated
```

### Git Workflow

```bash
# Initialize Git repository
git init

# Add all files
git add .

# Commit changes
git commit -m "Initial commit"

# Add remote origin
git remote add origin https://github.com/username/repo.git

# Push to remote
git push -u origin main
```

## Troubleshooting Common Issues

### 1. Port Already in Use

```bash
# If port 3000 is busy, Next.js will automatically use the next available port
# Or specify a different port:
npm run dev -- -p 3001
```

### 2. Module Not Found Errors

```bash
# Clear Next.js cache
rm -rf .next

# Clear npm cache
npm cache clean --force

# Reinstall dependencies
rm -rf node_modules package-lock.json
npm install
```

### 3. TypeScript Errors

```bash
# Check TypeScript configuration
npx tsc --noEmit

# Restart TypeScript server in VS Code
# Command Palette > TypeScript: Restart TS Server
```

### 4. Image Optimization Issues

If you encounter image optimization errors:

```javascript
// next.config.js
/** @type {import('next').NextConfig} */
const nextConfig = {
  images: {
    unoptimized: true, // Disable image optimization for development
  },
};

module.exports = nextConfig;
```

## Performance Monitoring

### 1. Built-in Performance Metrics

```bash
# Run with performance analysis
npm run dev -- --turbo

# Build with bundle analyzer
npm install -D @next/bundle-analyzer
```

### 2. Lighthouse Testing

```bash
# Install Lighthouse CLI
npm install -g lighthouse

# Test your application
lighthouse http://localhost:3000 --view
```

## Environment-Specific Configuration

### Development Environment

```javascript
// next.config.js
const isDev = process.env.NODE_ENV === 'development';

/** @type {import('next').NextConfig} */
const nextConfig = {
  // Enable source maps in development
  ...(isDev && {
    webpack: (config) => {
      config.devtool = 'source-map';
      return config;
    },
  }),
  
  // Disable image optimization in development
  images: {
    ...(isDev && { unoptimized: true }),
  },
};

module.exports = nextConfig;
```

### Production Environment

```javascript
// next.config.js
const isProd = process.env.NODE_ENV === 'production';

/** @type {import('next').NextConfig} */
const nextConfig = {
  // Production optimizations
  ...(isProd && {
    swcMinify: true,
    compress: true,
    poweredByHeader: false,
  }),
  
  // Security headers for production
  ...(isProd && {
    async headers() {
      return [
        {
          source: '/(.*)',
          headers: [
            {
              key: 'X-Frame-Options',
              value: 'DENY',
            },
            {
              key: 'X-Content-Type-Options',
              value: 'nosniff',
            },
          ],
        },
      ];
    },
  }),
};

module.exports = nextConfig;
```

## Summary

You've successfully set up a complete Next.js development environment! Here's what we covered:

### ✅ Environment Setup:
- Node.js and npm installation
- Project creation with create-next-app
- Manual setup process
- Development tool configuration

### ✅ Project Structure:
- Understanding the Next.js file organization
- Essential configuration files
- Environment variables setup
- TypeScript configuration

### ✅ Development Workflow:
- Creating pages and layouts
- Development server usage
- Building and deploying
- Performance monitoring

### ✅ Best Practices:
- Code formatting with Prettier
- Linting with ESLint
- Version control with Git
- Troubleshooting common issues

### Key Takeaways:
1. **Use create-next-app** for quick project setup
2. **Configure TypeScript** for type safety
3. **Set up development tools** early in the project
4. **Use environment variables** for configuration
5. **Follow the recommended file structure**

Your development environment is now ready for building amazing Next.js applications! In the next guide, we'll explore the Next.js project structure in detail and understand how different parts work together.

## Practice Exercise

Create a new Next.js project and:

1. Set up the development environment following this guide
2. Create a simple navigation between multiple pages
3. Configure TypeScript and Tailwind CSS
4. Add some basic styling and components
5. Test the development and build processes

## Next Steps

Continue to [Next.js Project Structure](./03-NextJS-Project-Structure.md) to learn about the file organization and understand how Next.js projects are structured.
