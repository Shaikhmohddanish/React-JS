# Styling in Next.js

## Table of Contents
1. [Introduction to Styling in Next.js](#introduction)
2. [Global CSS](#global-css)
3. [CSS Modules](#css-modules)
4. [Styled Components](#styled-components)
5. [Tailwind CSS](#tailwind-css)
6. [CSS-in-JS Solutions](#css-in-js)
7. [Sass/SCSS Support](#sass-scss)
8. [PostCSS Configuration](#postcss)
9. [Performance Considerations](#performance)
10. [Best Practices](#best-practices)

## Introduction to Styling in Next.js {#introduction}

Next.js provides multiple ways to style your applications, each with its own advantages and use cases. Understanding these different approaches will help you choose the right styling solution for your project.

### Styling Options Overview

```typescript
// Different styling approaches in Next.js
interface StylingOption {
  name: string;
  type: 'global' | 'scoped' | 'css-in-js';
  performance: 'high' | 'medium' | 'low';
  bundle_size: 'small' | 'medium' | 'large';
  learning_curve: 'easy' | 'medium' | 'steep';
}

const stylingOptions: StylingOption[] = [
  {
    name: 'Global CSS',
    type: 'global',
    performance: 'high',
    bundle_size: 'small',
    learning_curve: 'easy'
  },
  {
    name: 'CSS Modules',
    type: 'scoped',
    performance: 'high',
    bundle_size: 'small',
    learning_curve: 'easy'
  },
  {
    name: 'Tailwind CSS',
    type: 'global',
    performance: 'high',
    bundle_size: 'medium',
    learning_curve: 'medium'
  },
  {
    name: 'Styled Components',
    type: 'css-in-js',
    performance: 'medium',
    bundle_size: 'medium',
    learning_curve: 'medium'
  }
];
```

## Global CSS {#global-css}

Global CSS is the traditional way of styling web applications. In Next.js, global styles must be imported in the `_app.js` or `_app.tsx` file.

### Setting Up Global CSS

```css
/* styles/globals.css */
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

html,
body {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
  line-height: 1.6;
  color: #333;
}

a {
  color: #0070f3;
  text-decoration: none;
}

a:hover {
  text-decoration: underline;
}

.container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 20px;
}

.btn {
  display: inline-block;
  padding: 12px 24px;
  background-color: #0070f3;
  color: white;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  transition: background-color 0.2s ease;
}

.btn:hover {
  background-color: #0051cc;
}

.btn-secondary {
  background-color: #eaeaea;
  color: #333;
}

.btn-secondary:hover {
  background-color: #d1d1d1;
}
```

```typescript
// pages/_app.tsx
import '../styles/globals.css';
import type { AppProps } from 'next/app';

export default function App({ Component, pageProps }: AppProps) {
  return <Component {...pageProps} />;
}
```

### Using Global Styles

```typescript
// pages/index.tsx
export default function HomePage() {
  return (
    <div className="container">
      <h1>Welcome to Next.js</h1>
      <p>This is styled with global CSS.</p>
      <button className="btn">Primary Button</button>
      <button className="btn btn-secondary">Secondary Button</button>
    </div>
  );
}
```

## CSS Modules {#css-modules}

CSS Modules provide locally scoped CSS by default. They're perfect for component-level styling without worrying about class name conflicts.

### Creating CSS Modules

```css
/* components/Button.module.css */
.button {
  display: inline-block;
  padding: 12px 24px;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  font-size: 16px;
  font-weight: 500;
  transition: all 0.2s ease;
}

.primary {
  background-color: #0070f3;
  color: white;
}

.primary:hover {
  background-color: #0051cc;
  transform: translateY(-1px);
}

.secondary {
  background-color: #eaeaea;
  color: #333;
}

.secondary:hover {
  background-color: #d1d1d1;
}

.large {
  padding: 16px 32px;
  font-size: 18px;
}

.small {
  padding: 8px 16px;
  font-size: 14px;
}

.disabled {
  opacity: 0.6;
  cursor: not-allowed;
  pointer-events: none;
}
```

```typescript
// components/Button.tsx
import styles from './Button.module.css';
import { ReactNode, ButtonHTMLAttributes } from 'react';

interface ButtonProps extends ButtonHTMLAttributes<HTMLButtonElement> {
  variant?: 'primary' | 'secondary';
  size?: 'small' | 'medium' | 'large';
  children: ReactNode;
}

export default function Button({ 
  variant = 'primary', 
  size = 'medium', 
  children, 
  className = '',
  disabled,
  ...props 
}: ButtonProps) {
  const buttonClasses = [
    styles.button,
    styles[variant],
    size !== 'medium' && styles[size],
    disabled && styles.disabled,
    className
  ].filter(Boolean).join(' ');

  return (
    <button 
      className={buttonClasses} 
      disabled={disabled}
      {...props}
    >
      {children}
    </button>
  );
}
```

### Advanced CSS Modules Example

```css
/* components/Card.module.css */
.card {
  background: white;
  border-radius: 12px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  overflow: hidden;
  transition: transform 0.2s ease, box-shadow 0.2s ease;
}

.card:hover {
  transform: translateY(-4px);
  box-shadow: 0 8px 25px rgba(0, 0, 0, 0.15);
}

.header {
  padding: 20px;
  border-bottom: 1px solid #eaeaea;
}

.title {
  margin: 0 0 8px 0;
  font-size: 24px;
  font-weight: 600;
  color: #333;
}

.subtitle {
  margin: 0;
  font-size: 16px;
  color: #666;
}

.content {
  padding: 20px;
}

.footer {
  padding: 20px;
  background-color: #fafafa;
  border-top: 1px solid #eaeaea;
}

.featured {
  border: 2px solid #0070f3;
}

.featured .title {
  color: #0070f3;
}
```

```typescript
// components/Card.tsx
import styles from './Card.module.css';
import { ReactNode } from 'react';

interface CardProps {
  title: string;
  subtitle?: string;
  children: ReactNode;
  footer?: ReactNode;
  featured?: boolean;
  className?: string;
}

export default function Card({ 
  title, 
  subtitle, 
  children, 
  footer, 
  featured = false,
  className = ''
}: CardProps) {
  const cardClasses = [
    styles.card,
    featured && styles.featured,
    className
  ].filter(Boolean).join(' ');

  return (
    <div className={cardClasses}>
      <div className={styles.header}>
        <h2 className={styles.title}>{title}</h2>
        {subtitle && <p className={styles.subtitle}>{subtitle}</p>}
      </div>
      <div className={styles.content}>
        {children}
      </div>
      {footer && (
        <div className={styles.footer}>
          {footer}
        </div>
      )}
    </div>
  );
}
```

## Styled Components {#styled-components}

Styled Components is a popular CSS-in-JS library that allows you to write CSS directly in your JavaScript/TypeScript files.

### Installation and Setup

```bash
npm install styled-components
npm install --save-dev @types/styled-components
```

```typescript
// next.config.js
/** @type {import('next').NextConfig} */
const nextConfig = {
  compiler: {
    styledComponents: true,
  },
}

module.exports = nextConfig
```

### Basic Styled Components

```typescript
// components/StyledButton.tsx
import styled, { css } from 'styled-components';

interface ButtonProps {
  variant?: 'primary' | 'secondary' | 'danger';
  size?: 'small' | 'medium' | 'large';
  fullWidth?: boolean;
}

const StyledButton = styled.button<ButtonProps>`
  display: inline-flex;
  align-items: center;
  justify-content: center;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  font-weight: 500;
  transition: all 0.2s ease;
  text-decoration: none;
  
  ${props => {
    switch (props.size) {
      case 'small':
        return css`
          padding: 8px 16px;
          font-size: 14px;
        `;
      case 'large':
        return css`
          padding: 16px 32px;
          font-size: 18px;
        `;
      default:
        return css`
          padding: 12px 24px;
          font-size: 16px;
        `;
    }
  }}
  
  ${props => {
    switch (props.variant) {
      case 'secondary':
        return css`
          background-color: #eaeaea;
          color: #333;
          
          &:hover {
            background-color: #d1d1d1;
          }
        `;
      case 'danger':
        return css`
          background-color: #ff4757;
          color: white;
          
          &:hover {
            background-color: #ff3838;
          }
        `;
      default:
        return css`
          background-color: #0070f3;
          color: white;
          
          &:hover {
            background-color: #0051cc;
          }
        `;
    }
  }}
  
  ${props => props.fullWidth && css`
    width: 100%;
  `}
  
  &:disabled {
    opacity: 0.6;
    cursor: not-allowed;
  }
`;

export default StyledButton;
```

### Advanced Styled Components with Theming

```typescript
// styles/theme.ts
export const theme = {
  colors: {
    primary: '#0070f3',
    primaryDark: '#0051cc',
    secondary: '#eaeaea',
    secondaryDark: '#d1d1d1',
    danger: '#ff4757',
    dangerDark: '#ff3838',
    success: '#2ed573',
    successDark: '#1abc9c',
    text: '#333',
    textLight: '#666',
    background: '#ffffff',
    backgroundLight: '#fafafa',
    border: '#eaeaea',
  },
  spacing: {
    xs: '4px',
    sm: '8px',
    md: '16px',
    lg: '24px',
    xl: '32px',
    xxl: '48px',
  },
  borderRadius: {
    sm: '4px',
    md: '6px',
    lg: '12px',
    xl: '20px',
  },
  shadows: {
    sm: '0 2px 4px rgba(0, 0, 0, 0.1)',
    md: '0 4px 6px rgba(0, 0, 0, 0.1)',
    lg: '0 8px 25px rgba(0, 0, 0, 0.15)',
  },
  breakpoints: {
    sm: '640px',
    md: '768px',
    lg: '1024px',
    xl: '1280px',
  },
};

export type Theme = typeof theme;
```

```typescript
// pages/_app.tsx
import { ThemeProvider } from 'styled-components';
import { theme } from '../styles/theme';
import type { AppProps } from 'next/app';

export default function App({ Component, pageProps }: AppProps) {
  return (
    <ThemeProvider theme={theme}>
      <Component {...pageProps} />
    </ThemeProvider>
  );
}
```

```typescript
// components/ThemedCard.tsx
import styled from 'styled-components';
import { ReactNode } from 'react';

const CardContainer = styled.div`
  background: ${props => props.theme.colors.background};
  border-radius: ${props => props.theme.borderRadius.lg};
  box-shadow: ${props => props.theme.shadows.md};
  overflow: hidden;
  transition: all 0.2s ease;
  
  &:hover {
    transform: translateY(-4px);
    box-shadow: ${props => props.theme.shadows.lg};
  }
`;

const CardHeader = styled.div`
  padding: ${props => props.theme.spacing.lg};
  border-bottom: 1px solid ${props => props.theme.colors.border};
`;

const CardTitle = styled.h2`
  margin: 0 0 ${props => props.theme.spacing.sm} 0;
  font-size: 24px;
  font-weight: 600;
  color: ${props => props.theme.colors.text};
`;

const CardSubtitle = styled.p`
  margin: 0;
  font-size: 16px;
  color: ${props => props.theme.colors.textLight};
`;

const CardContent = styled.div`
  padding: ${props => props.theme.spacing.lg};
`;

const CardFooter = styled.div`
  padding: ${props => props.theme.spacing.lg};
  background-color: ${props => props.theme.colors.backgroundLight};
  border-top: 1px solid ${props => props.theme.colors.border};
`;

interface ThemedCardProps {
  title: string;
  subtitle?: string;
  children: ReactNode;
  footer?: ReactNode;
}

export default function ThemedCard({ title, subtitle, children, footer }: ThemedCardProps) {
  return (
    <CardContainer>
      <CardHeader>
        <CardTitle>{title}</CardTitle>
        {subtitle && <CardSubtitle>{subtitle}</CardSubtitle>}
      </CardHeader>
      <CardContent>
        {children}
      </CardContent>
      {footer && (
        <CardFooter>
          {footer}
        </CardFooter>
      )}
    </CardContainer>
  );
}
```

## Tailwind CSS {#tailwind-css}

Tailwind CSS is a utility-first CSS framework that provides low-level utility classes to build custom designs.

### Installation and Setup

```bash
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
```

```javascript
// tailwind.config.js
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: [
    './pages/**/*.{js,ts,jsx,tsx,mdx}',
    './components/**/*.{js,ts,jsx,tsx,mdx}',
    './app/**/*.{js,ts,jsx,tsx,mdx}',
  ],
  theme: {
    extend: {
      colors: {
        primary: {
          50: '#eff6ff',
          100: '#dbeafe',
          500: '#3b82f6',
          600: '#2563eb',
          700: '#1d4ed8',
          900: '#1e3a8a',
        },
      },
      spacing: {
        '18': '4.5rem',
        '88': '22rem',
      },
      fontFamily: {
        sans: ['Inter', 'sans-serif'],
      },
    },
  },
  plugins: [],
}
```

```css
/* styles/globals.css */
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base {
  html {
    @apply scroll-smooth;
  }
  
  body {
    @apply font-sans text-gray-900 bg-white;
  }
}

@layer components {
  .btn {
    @apply inline-flex items-center justify-center px-6 py-3 border border-transparent text-base font-medium rounded-md transition-colors duration-200 focus:outline-none focus:ring-2 focus:ring-offset-2;
  }
  
  .btn-primary {
    @apply bg-primary-600 text-white hover:bg-primary-700 focus:ring-primary-500;
  }
  
  .btn-secondary {
    @apply bg-gray-200 text-gray-900 hover:bg-gray-300 focus:ring-gray-500;
  }
  
  .card {
    @apply bg-white rounded-lg shadow-md overflow-hidden transition-transform duration-200 hover:scale-105;
  }
}

@layer utilities {
  .text-gradient {
    @apply bg-gradient-to-r from-primary-600 to-purple-600 bg-clip-text text-transparent;
  }
}
```

### Tailwind Component Examples

```typescript
// components/TailwindButton.tsx
import { ReactNode, ButtonHTMLAttributes } from 'react';
import { cva, type VariantProps } from 'class-variance-authority';

const buttonVariants = cva(
  'inline-flex items-center justify-center rounded-md text-sm font-medium transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 disabled:opacity-50 disabled:pointer-events-none',
  {
    variants: {
      variant: {
        default: 'bg-primary-600 text-white hover:bg-primary-700',
        destructive: 'bg-red-500 text-white hover:bg-red-600',
        outline: 'border border-gray-300 bg-white hover:bg-gray-50',
        secondary: 'bg-gray-200 text-gray-900 hover:bg-gray-300',
        ghost: 'hover:bg-gray-100',
        link: 'underline-offset-4 hover:underline text-primary-600',
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

interface ButtonProps
  extends ButtonHTMLAttributes<HTMLButtonElement>,
    VariantProps<typeof buttonVariants> {
  children: ReactNode;
}

export default function TailwindButton({ 
  className, 
  variant, 
  size, 
  children,
  ...props 
}: ButtonProps) {
  return (
    <button
      className={buttonVariants({ variant, size, className })}
      {...props}
    >
      {children}
    </button>
  );
}
```

```typescript
// components/TailwindCard.tsx
interface TailwindCardProps {
  title: string;
  description: string;
  image?: string;
  badge?: string;
  action?: React.ReactNode;
}

export default function TailwindCard({ 
  title, 
  description, 
  image, 
  badge, 
  action 
}: TailwindCardProps) {
  return (
    <div className="group relative bg-white rounded-xl shadow-md hover:shadow-xl transition-all duration-300 overflow-hidden">
      {image && (
        <div className="aspect-w-16 aspect-h-9">
          <img
            src={image}
            alt={title}
            className="w-full h-48 object-cover group-hover:scale-105 transition-transform duration-300"
          />
        </div>
      )}
      
      <div className="p-6">
        <div className="flex items-start justify-between mb-3">
          <h3 className="text-xl font-semibold text-gray-900 group-hover:text-primary-600 transition-colors">
            {title}
          </h3>
          {badge && (
            <span className="inline-flex items-center px-2.5 py-0.5 rounded-full text-xs font-medium bg-primary-100 text-primary-800">
              {badge}
            </span>
          )}
        </div>
        
        <p className="text-gray-600 text-sm leading-relaxed mb-4">
          {description}
        </p>
        
        {action && (
          <div className="flex justify-end">
            {action}
          </div>
        )}
      </div>
    </div>
  );
}
```

## Best Practices {#best-practices}

### 1. Choose the Right Styling Solution

```typescript
// Decision Matrix for Styling Solutions
interface StyleDecision {
  useCase: string;
  recommended: string[];
  avoid: string[];
}

const stylingDecisions: StyleDecision[] = [
  {
    useCase: "Small project with simple styling",
    recommended: ["Global CSS", "CSS Modules"],
    avoid: ["Styled Components", "Complex CSS-in-JS"]
  },
  {
    useCase: "Component library development",
    recommended: ["CSS Modules", "Styled Components"],
    avoid: ["Global CSS", "Tailwind (for reusable components)"]
  },
  {
    useCase: "Rapid prototyping",
    recommended: ["Tailwind CSS"],
    avoid: ["Custom CSS frameworks"]
  },
  {
    useCase: "Design system implementation",
    recommended: ["Styled Components with theming", "CSS Modules"],
    avoid: ["Global CSS"]
  },
  {
    useCase: "Server-side rendering optimization",
    recommended: ["CSS Modules", "Global CSS"],
    avoid: ["Runtime CSS-in-JS"]
  }
];
```

### 2. Performance Optimization

```typescript
// Optimized component loading
import dynamic from 'next/dynamic';

// Lazy load heavy styled components
const HeavyStyledComponent = dynamic(
  () => import('../components/HeavyStyledComponent'),
  {
    loading: () => <div>Loading...</div>,
    ssr: false // Only load on client if styling is not critical
  }
);

// CSS splitting for better performance
const CriticalStyles = dynamic(
  () => import('../styles/critical.module.css'),
  { ssr: true }
);

const NonCriticalStyles = dynamic(
  () => import('../styles/non-critical.module.css'),
  { ssr: false }
);
```

### 3. Responsive Design Patterns

```css
/* CSS Modules responsive approach */
.container {
  padding: 1rem;
}

@media (min-width: 640px) {
  .container {
    padding: 1.5rem;
  }
}

@media (min-width: 1024px) {
  .container {
    padding: 2rem;
  }
}
```

```typescript
// Styled Components responsive approach
const ResponsiveContainer = styled.div`
  padding: 1rem;
  
  ${props => props.theme.breakpoints.sm} {
    padding: 1.5rem;
  }
  
  ${props => props.theme.breakpoints.lg} {
    padding: 2rem;
  }
`;
```

```jsx
// Tailwind responsive approach
<div className="p-4 sm:p-6 lg:p-8">
  {/* Content */}
</div>
```

### 4. CSS Organization

```
styles/
├── globals.css              # Global styles and resets
├── components/              # Component-specific styles
│   ├── Button.module.css
│   ├── Card.module.css
│   └── Navigation.module.css
├── layouts/                 # Layout-specific styles
│   ├── Header.module.css
│   └── Footer.module.css
├── pages/                   # Page-specific styles
│   ├── Home.module.css
│   └── About.module.css
├── utilities/               # Utility classes
│   ├── spacing.css
│   └── typography.css
└── theme/                   # Theme and design tokens
    ├── colors.css
    ├── typography.css
    └── spacing.css
```

## Exercises

### Exercise 1: CSS Modules Practice
Create a reusable `ProductCard` component using CSS Modules with the following features:
- Product image with hover effects
- Title and description
- Price display with currency formatting
- Add to cart button
- Rating display with stars
- Responsive design for mobile and desktop

### Exercise 2: Styled Components Theme
Build a complete theme system using Styled Components:
- Define color palette with primary, secondary, and semantic colors
- Create typography scale
- Implement spacing system
- Build responsive breakpoints
- Create dark/light mode toggle

### Exercise 3: Tailwind Component Library
Create a mini component library using Tailwind CSS:
- Button component with multiple variants and sizes
- Input field with validation states
- Modal component with backdrop
- Navigation component with responsive menu
- Card component with different layouts

### Exercise 4: Performance Optimization
Optimize styling performance:
- Implement critical CSS extraction
- Set up CSS purging for production
- Create efficient responsive images with styling
- Implement lazy loading for non-critical styles

## Summary

Styling in Next.js offers multiple approaches, each suited for different use cases:

- **Global CSS**: Simple, performant, good for traditional approaches
- **CSS Modules**: Scoped styling, excellent for component-based architecture
- **Styled Components**: Dynamic styling, great for theme systems
- **Tailwind CSS**: Utility-first, rapid development, consistent design

Choose your styling approach based on project requirements, team preferences, and performance needs. Many projects successfully combine multiple approaches for different use cases.

Next, we'll explore **Static Site Generation (SSG)** and learn how to build fast, pre-rendered pages with Next.js.
