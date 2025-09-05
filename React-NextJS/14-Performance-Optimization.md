# Performance Optimization in Next.js

## Table of Contents
1. [Introduction to Performance Optimization](#introduction)
2. [Core Web Vitals](#core-web-vitals)
3. [Bundle Optimization](#bundle-optimization)
4. [Caching Strategies](#caching-strategies)
5. [Code Splitting and Lazy Loading](#code-splitting)
6. [Runtime Performance](#runtime-performance)
7. [Database and API Optimization](#database-api)
8. [Memory Management](#memory-management)
9. [Performance Monitoring](#performance-monitoring)
10. [Advanced Optimization Techniques](#advanced-techniques)
11. [Performance Testing](#performance-testing)
12. [Best Practices](#best-practices)

## Introduction to Performance Optimization {#introduction}

Performance optimization in Next.js involves multiple layers: build-time optimizations, runtime performance, network efficiency, and user experience metrics. Understanding and optimizing each layer is crucial for delivering fast, responsive applications.

### Performance Metrics Overview

```typescript
// types/performance.ts
interface PerformanceMetrics {
  // Core Web Vitals
  lcp: number; // Largest Contentful Paint
  fid: number; // First Input Delay
  cls: number; // Cumulative Layout Shift
  
  // Additional metrics
  fcp: number; // First Contentful Paint
  ttfb: number; // Time to First Byte
  tti: number; // Time to Interactive
  
  // Custom metrics
  bundleSize: number;
  renderTime: number;
  hydrationTime: number;
}

interface PerformanceBudget {
  lcp: number; // < 2.5s
  fid: number; // < 100ms
  cls: number; // < 0.1
  bundleSize: number; // < 300KB
  imageSize: number; // < 500KB
  apiResponseTime: number; // < 500ms
}

const performanceBudget: PerformanceBudget = {
  lcp: 2500,
  fid: 100,
  cls: 0.1,
  bundleSize: 300 * 1024,
  imageSize: 500 * 1024,
  apiResponseTime: 500,
};
```

### Performance Optimization Strategy

```typescript
// lib/performanceStrategy.ts
interface OptimizationStrategy {
  phase: 'build' | 'runtime' | 'network' | 'user-experience';
  techniques: string[];
  impact: 'high' | 'medium' | 'low';
  complexity: 'low' | 'medium' | 'high';
}

export const optimizationStrategies: OptimizationStrategy[] = [
  {
    phase: 'build',
    techniques: ['Code splitting', 'Tree shaking', 'Bundle analysis', 'Static optimization'],
    impact: 'high',
    complexity: 'medium',
  },
  {
    phase: 'runtime',
    techniques: ['Component memoization', 'Virtual scrolling', 'State optimization'],
    impact: 'high',
    complexity: 'medium',
  },
  {
    phase: 'network',
    techniques: ['CDN usage', 'Compression', 'Caching', 'Resource hints'],
    impact: 'high',
    complexity: 'low',
  },
  {
    phase: 'user-experience',
    techniques: ['Progressive loading', 'Skeleton screens', 'Optimistic updates'],
    impact: 'medium',
    complexity: 'medium',
  },
];
```

## Core Web Vitals {#core-web-vitals}

### Measuring Core Web Vitals

```typescript
// lib/webVitals.ts
import { getCLS, getFID, getFCP, getLCP, getTTFB } from 'web-vitals';

interface VitalsData {
  name: string;
  value: number;
  delta: number;
  id: string;
  timestamp: number;
}

class WebVitalsTracker {
  private vitals: Map<string, VitalsData> = new Map();
  private callbacks: Array<(vital: VitalsData) => void> = [];

  constructor() {
    this.initializeTracking();
  }

  private initializeTracking() {
    // Track Core Web Vitals
    getCLS(this.handleVital.bind(this), true);
    getFID(this.handleVital.bind(this));
    getFCP(this.handleVital.bind(this));
    getLCP(this.handleVital.bind(this), true);
    getTTFB(this.handleVital.bind(this));
  }

  private handleVital(vital: any) {
    const vitalData: VitalsData = {
      name: vital.name,
      value: vital.value,
      delta: vital.delta,
      id: vital.id,
      timestamp: Date.now(),
    };

    this.vitals.set(vital.name, vitalData);
    this.notifyCallbacks(vitalData);
    this.sendToAnalytics(vitalData);
  }

  private notifyCallbacks(vital: VitalsData) {
    this.callbacks.forEach(callback => callback(vital));
  }

  private sendToAnalytics(vital: VitalsData) {
    // Send to Google Analytics
    if (typeof window !== 'undefined' && window.gtag) {
      window.gtag('event', vital.name, {
        event_category: 'Web Vitals',
        value: Math.round(vital.value),
        event_label: vital.id,
        non_interaction: true,
      });
    }

    // Send to custom analytics
    if (typeof window !== 'undefined') {
      fetch('/api/analytics/vitals', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          vital,
          page: window.location.pathname,
          userAgent: navigator.userAgent,
        }),
      }).catch(console.error);
    }
  }

  onVital(callback: (vital: VitalsData) => void) {
    this.callbacks.push(callback);
  }

  getVitals(): VitalsData[] {
    return Array.from(this.vitals.values());
  }

  getVital(name: string): VitalsData | undefined {
    return this.vitals.get(name);
  }

  isGoodPerformance(): boolean {
    const lcp = this.vitals.get('LCP');
    const fid = this.vitals.get('FID');
    const cls = this.vitals.get('CLS');

    return (
      (!lcp || lcp.value <= 2500) &&
      (!fid || fid.value <= 100) &&
      (!cls || cls.value <= 0.1)
    );
  }
}

export const webVitalsTracker = new WebVitalsTracker();

// Hook for using Web Vitals in components
export function useWebVitals() {
  const [vitals, setVitals] = useState<VitalsData[]>([]);
  const [isGoodPerformance, setIsGoodPerformance] = useState(true);

  useEffect(() => {
    const handleVitalUpdate = (vital: VitalsData) => {
      setVitals(prev => {
        const updated = prev.filter(v => v.name !== vital.name);
        return [...updated, vital];
      });
      setIsGoodPerformance(webVitalsTracker.isGoodPerformance());
    };

    webVitalsTracker.onVital(handleVitalUpdate);
    setVitals(webVitalsTracker.getVitals());
    setIsGoodPerformance(webVitalsTracker.isGoodPerformance());
  }, []);

  return { vitals, isGoodPerformance };
}
```

### LCP Optimization Component

```typescript
// components/LCPOptimizedHero.tsx
import Image from 'next/image';
import { useEffect, useRef, useState } from 'react';

interface LCPOptimizedHeroProps {
  title: string;
  subtitle: string;
  backgroundImage: string;
  ctaText: string;
  ctaHref: string;
}

export default function LCPOptimizedHero({
  title,
  subtitle,
  backgroundImage,
  ctaText,
  ctaHref,
}: LCPOptimizedHeroProps) {
  const [imageLoaded, setImageLoaded] = useState(false);
  const [lcpTime, setLcpTime] = useState<number | null>(null);
  const heroRef = useRef<HTMLElement>(null);

  useEffect(() => {
    // Measure LCP for this hero section
    const observer = new PerformanceObserver((list) => {
      const entries = list.getEntries();
      const lcpEntry = entries[entries.length - 1] as any;
      
      if (lcpEntry && heroRef.current?.contains(lcpEntry.element)) {
        setLcpTime(lcpEntry.startTime);
      }
    });

    observer.observe({ entryTypes: ['largest-contentful-paint'] });

    return () => observer.disconnect();
  }, []);

  // Preload critical resources
  useEffect(() => {
    // Preload hero image
    const link = document.createElement('link');
    link.rel = 'preload';
    link.as = 'image';
    link.href = backgroundImage;
    document.head.appendChild(link);

    // Preload fonts if needed
    const fontLink = document.createElement('link');
    fontLink.rel = 'preload';
    fontLink.as = 'font';
    fontLink.type = 'font/woff2';
    fontLink.href = '/fonts/primary-bold.woff2';
    fontLink.crossOrigin = 'anonymous';
    document.head.appendChild(fontLink);

    return () => {
      document.head.removeChild(link);
      document.head.removeChild(fontLink);
    };
  }, [backgroundImage]);

  return (
    <section 
      ref={heroRef}
      className="relative h-screen flex items-center justify-center overflow-hidden"
    >
      {/* Background Image - Optimized for LCP */}
      <div className="absolute inset-0 z-0">
        <Image
          src={backgroundImage}
          alt=""
          fill
          priority // Critical for LCP
          quality={85}
          sizes="100vw"
          style={{
            objectFit: 'cover',
          }}
          onLoad={() => setImageLoaded(true)}
          placeholder="blur"
          blurDataURL="data:image/jpeg;base64,/9j/4AAQSkZJRgABAQAAAQABAAD/2wBDAAYEBQYFBAYGBQYHBwYIChAKCgkJChQODwwQFxQYGBcUFhYaHSUfGhsjHBYWICwgIyYnKSopGR8tMC0oMCUoKSj/2wBDAQcHBwoIChMKChMoGhYaKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCj/wAARCAAIAAoDASIAAhEBAxEB/8QAFQABAQAAAAAAAAAAAAAAAAAAAAv/xAAhEAACAQMDBQAAAAAAAAAAAAABAgMABAUGIWGRkqGx0f/EABUBAQEAAAAAAAAAAAAAAAAAAAMF/8QAGhEAAgIDAAAAAAAAAAAAAAAAAAECEgMRkf/aAAwDAQACEQMRAD8AltJagyeH0AthI5xdrLcNM91BF5pX2HaH9bcfaSXWGaRmknyJckliyjqTzSlT54b6bk+h0R//2Q=="
        />
      </div>

      {/* Overlay */}
      <div className="absolute inset-0 bg-black bg-opacity-40 z-10" />

      {/* Content */}
      <div className="relative z-20 text-center text-white max-w-4xl mx-auto px-4">
        {/* Title - Optimized for rendering */}
        <h1 
          className="text-4xl md:text-6xl lg:text-7xl font-bold mb-6 leading-tight"
          style={{
            fontDisplay: 'swap', // Prevent invisible text during font load
            willChange: 'transform', // Optimize for animations
          }}
        >
          {title}
        </h1>

        {/* Subtitle */}
        <p className="text-xl md:text-2xl mb-8 opacity-90 leading-relaxed">
          {subtitle}
        </p>

        {/* CTA Button */}
        <a
          href={ctaHref}
          className="inline-block bg-blue-600 hover:bg-blue-700 text-white font-semibold px-8 py-4 rounded-lg text-lg transition-colors duration-200 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-offset-2 focus:ring-offset-black"
          // Preload the CTA destination
          onMouseEnter={() => {
            const link = document.createElement('link');
            link.rel = 'prefetch';
            link.href = ctaHref;
            document.head.appendChild(link);
          }}
        >
          {ctaText}
        </a>
      </div>

      {/* Performance indicator (development only) */}
      {process.env.NODE_ENV === 'development' && (
        <div className="absolute top-4 right-4 bg-black bg-opacity-75 text-white p-2 rounded text-sm z-30">
          <div>Image loaded: {imageLoaded ? '✅' : '⏳'}</div>
          {lcpTime && <div>LCP: {Math.round(lcpTime)}ms</div>}
        </div>
      )}
    </section>
  );
}
```

## Bundle Optimization {#bundle-optimization}

### Bundle Analysis and Optimization

```typescript
// scripts/analyzeBundles.js
const { BundleAnalyzerPlugin } = require('webpack-bundle-analyzer');
const { readFileSync, writeFileSync } = require('fs');
const path = require('path');

class BundleOptimizer {
  constructor() {
    this.thresholds = {
      maxBundleSize: 300 * 1024, // 300KB
      maxChunkSize: 100 * 1024, // 100KB
      maxDuplicates: 5, // Maximum duplicate packages
    };
  }

  analyzeBundles() {
    const buildDir = path.join(process.cwd(), '.next');
    const manifestPath = path.join(buildDir, 'build-manifest.json');
    
    if (!require('fs').existsSync(manifestPath)) {
      console.error('Build manifest not found. Run "npm run build" first.');
      return;
    }

    const manifest = JSON.parse(readFileSync(manifestPath, 'utf8'));
    const analysis = this.performAnalysis(manifest);
    
    this.generateReport(analysis);
    this.checkBudgets(analysis);
    
    return analysis;
  }

  performAnalysis(manifest) {
    const bundles = {};
    const duplicates = new Map();

    // Analyze each page bundle
    Object.entries(manifest.pages).forEach(([page, files]) => {
      const bundleSize = this.calculateBundleSize(files);
      const dependencies = this.extractDependencies(files);
      
      bundles[page] = {
        size: bundleSize,
        files,
        dependencies,
        gzipSize: bundleSize * 0.3, // Estimate gzip compression
      };

      // Track duplicates
      dependencies.forEach(dep => {
        if (duplicates.has(dep)) {
          duplicates.set(dep, duplicates.get(dep) + 1);
        } else {
          duplicates.set(dep, 1);
        }
      });
    });

    return {
      bundles,
      duplicates: Array.from(duplicates.entries()).filter(([_, count]) => count > 1),
      totalSize: Object.values(bundles).reduce((sum, bundle) => sum + bundle.size, 0),
    };
  }

  calculateBundleSize(files) {
    return files.reduce((size, file) => {
      try {
        const filePath = path.join(process.cwd(), '.next', file);
        const stats = require('fs').statSync(filePath);
        return size + stats.size;
      } catch {
        return size;
      }
    }, 0);
  }

  extractDependencies(files) {
    const dependencies = new Set();
    
    files.forEach(file => {
      if (file.includes('node_modules')) {
        const matches = file.match(/node_modules\/([^\/]+)/g);
        if (matches) {
          matches.forEach(match => {
            const dep = match.replace('node_modules/', '');
            dependencies.add(dep);
          });
        }
      }
    });

    return Array.from(dependencies);
  }

  generateReport(analysis) {
    const report = {
      timestamp: new Date().toISOString(),
      summary: {
        totalBundles: Object.keys(analysis.bundles).length,
        totalSize: analysis.totalSize,
        averageSize: analysis.totalSize / Object.keys(analysis.bundles).length,
        duplicatePackages: analysis.duplicates.length,
      },
      bundles: analysis.bundles,
      duplicates: analysis.duplicates,
      recommendations: this.generateRecommendations(analysis),
    };

    // Write report to file
    writeFileSync(
      path.join(process.cwd(), 'bundle-analysis.json'),
      JSON.stringify(report, null, 2)
    );

    console.log('Bundle Analysis Report Generated:');
    console.log(`Total Size: ${(report.summary.totalSize / 1024).toFixed(2)}KB`);
    console.log(`Average Bundle Size: ${(report.summary.averageSize / 1024).toFixed(2)}KB`);
    console.log(`Duplicate Packages: ${report.summary.duplicatePackages}`);
  }

  generateRecommendations(analysis) {
    const recommendations = [];

    // Check bundle sizes
    Object.entries(analysis.bundles).forEach(([page, bundle]) => {
      if (bundle.size > this.thresholds.maxBundleSize) {
        recommendations.push({
          type: 'bundle-size',
          page,
          message: `Bundle size (${(bundle.size / 1024).toFixed(2)}KB) exceeds limit`,
          suggestion: 'Consider code splitting or removing unused dependencies',
        });
      }
    });

    // Check duplicates
    analysis.duplicates.forEach(([package, count]) => {
      if (count > this.thresholds.maxDuplicates) {
        recommendations.push({
          type: 'duplicate-package',
          package,
          count,
          message: `Package "${package}" is included ${count} times`,
          suggestion: 'Consider using dynamic imports or moving to a shared chunk',
        });
      }
    });

    return recommendations;
  }

  checkBudgets(analysis) {
    const violations = [];

    if (analysis.totalSize > this.thresholds.maxBundleSize * 5) {
      violations.push('Total bundle size exceeds budget');
    }

    if (analysis.duplicates.length > this.thresholds.maxDuplicates) {
      violations.push('Too many duplicate packages');
    }

    if (violations.length > 0) {
      console.error('❌ Performance Budget Violations:');
      violations.forEach(violation => console.error(`  - ${violation}`));
      process.exit(1);
    } else {
      console.log('✅ All performance budgets met');
    }
  }
}

// Run analysis
if (require.main === module) {
  const optimizer = new BundleOptimizer();
  optimizer.analyzeBundles();
}

module.exports = BundleOptimizer;
```

### Dynamic Import Optimization

```typescript
// components/OptimizedComponentLoader.tsx
import { lazy, Suspense, ComponentType, useState, useEffect } from 'react';

interface LoaderOptions {
  delay?: number;
  timeout?: number;
  fallback?: ComponentType;
  preload?: boolean;
}

interface ComponentModule<T = {}> {
  default: ComponentType<T>;
}

// Higher-order component for optimized loading
export function withOptimizedLoading<T extends {}>(
  importFn: () => Promise<ComponentModule<T>>,
  options: LoaderOptions = {}
) {
  const {
    delay = 200,
    timeout = 10000,
    fallback: FallbackComponent,
    preload = false,
  } = options;

  // Create lazy component with timeout
  const LazyComponent = lazy(() => {
    return Promise.race([
      importFn(),
      new Promise<never>((_, reject) =>
        setTimeout(() => reject(new Error('Component load timeout')), timeout)
      ),
    ]);
  });

  // Preload component if needed
  if (preload && typeof window !== 'undefined') {
    importFn().catch(console.error);
  }

  return function OptimizedComponent(props: T) {
    const [shouldRender, setShouldRender] = useState(!delay);

    useEffect(() => {
      if (delay > 0) {
        const timer = setTimeout(() => setShouldRender(true), delay);
        return () => clearTimeout(timer);
      }
    }, []);

    if (!shouldRender) {
      return FallbackComponent ? <FallbackComponent {...props} /> : null;
    }

    return (
      <Suspense 
        fallback={
          FallbackComponent ? (
            <FallbackComponent {...props} />
          ) : (
            <div className="animate-pulse bg-gray-200 h-32 rounded" />
          )
        }
      >
        <LazyComponent {...props} />
      </Suspense>
    );
  };
}

// Smart component loader with intersection observer
export function useLazyComponent<T extends {}>(
  importFn: () => Promise<ComponentModule<T>>,
  options: LoaderOptions & { rootMargin?: string } = {}
) {
  const [Component, setComponent] = useState<ComponentType<T> | null>(null);
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState<Error | null>(null);

  const loadComponent = async () => {
    if (Component || isLoading) return;

    setIsLoading(true);
    setError(null);

    try {
      const module = await importFn();
      setComponent(() => module.default);
    } catch (err) {
      setError(err instanceof Error ? err : new Error('Failed to load component'));
    } finally {
      setIsLoading(false);
    }
  };

  // Create intersection observer ref
  const observerRef = useCallback((node: HTMLElement | null) => {
    if (!node) return;

    const observer = new IntersectionObserver(
      ([entry]) => {
        if (entry.isIntersecting) {
          loadComponent();
          observer.disconnect();
        }
      },
      { rootMargin: options.rootMargin || '100px' }
    );

    observer.observe(node);

    return () => observer.disconnect();
  }, []);

  return {
    Component,
    isLoading,
    error,
    observerRef,
    loadComponent,
  };
}

// Example usage
const OptimizedChart = withOptimizedLoading(
  () => import('../components/HeavyChart'),
  {
    delay: 300,
    preload: true,
    fallback: () => <div className="h-64 bg-gray-100 animate-pulse rounded" />,
  }
);

// Usage with intersection observer
function LazySection() {
  const { Component: HeavyComponent, isLoading, observerRef } = useLazyComponent(
    () => import('../components/HeavyDataTable')
  );

  return (
    <div ref={observerRef}>
      {isLoading && <div>Loading...</div>}
      {HeavyComponent && <HeavyComponent />}
    </div>
  );
}
```

## Caching Strategies {#caching-strategies}

### Advanced Caching Implementation

```typescript
// lib/advancedCache.ts
interface CacheEntry<T> {
  data: T;
  timestamp: number;
  expiresAt: number;
  tags: string[];
  version: string;
}

interface CacheOptions {
  ttl?: number; // Time to live in milliseconds
  tags?: string[];
  version?: string;
  maxSize?: number;
  staleWhileRevalidate?: boolean;
}

class AdvancedCache {
  private cache: Map<string, CacheEntry<any>> = new Map();
  private maxSize: number;
  private hitCount = 0;
  private missCount = 0;

  constructor(maxSize = 1000) {
    this.maxSize = maxSize;
    this.setupCleanup();
  }

  set<T>(key: string, data: T, options: CacheOptions = {}): void {
    const {
      ttl = 5 * 60 * 1000, // 5 minutes default
      tags = [],
      version = '1.0.0',
    } = options;

    const entry: CacheEntry<T> = {
      data,
      timestamp: Date.now(),
      expiresAt: Date.now() + ttl,
      tags,
      version,
    };

    // Check if cache is full
    if (this.cache.size >= this.maxSize && !this.cache.has(key)) {
      this.evictLRU();
    }

    this.cache.set(key, entry);
  }

  get<T>(key: string): T | null {
    const entry = this.cache.get(key);

    if (!entry) {
      this.missCount++;
      return null;
    }

    // Check expiration
    if (Date.now() > entry.expiresAt) {
      this.cache.delete(key);
      this.missCount++;
      return null;
    }

    this.hitCount++;
    return entry.data;
  }

  getStale<T>(key: string): T | null {
    const entry = this.cache.get(key);
    
    if (!entry) {
      this.missCount++;
      return null;
    }

    this.hitCount++;
    return entry.data;
  }

  invalidateByTag(tag: string): void {
    for (const [key, entry] of this.cache.entries()) {
      if (entry.tags.includes(tag)) {
        this.cache.delete(key);
      }
    }
  }

  invalidateByPattern(pattern: RegExp): void {
    for (const key of this.cache.keys()) {
      if (pattern.test(key)) {
        this.cache.delete(key);
      }
    }
  }

  clear(): void {
    this.cache.clear();
    this.hitCount = 0;
    this.missCount = 0;
  }

  getStats(): {
    size: number;
    hitRate: number;
    missRate: number;
    totalRequests: number;
  } {
    const totalRequests = this.hitCount + this.missCount;
    
    return {
      size: this.cache.size,
      hitRate: totalRequests > 0 ? this.hitCount / totalRequests : 0,
      missRate: totalRequests > 0 ? this.missCount / totalRequests : 0,
      totalRequests,
    };
  }

  private evictLRU(): void {
    let oldestKey: string | null = null;
    let oldestTime = Date.now();

    for (const [key, entry] of this.cache.entries()) {
      if (entry.timestamp < oldestTime) {
        oldestTime = entry.timestamp;
        oldestKey = key;
      }
    }

    if (oldestKey) {
      this.cache.delete(oldestKey);
    }
  }

  private setupCleanup(): void {
    // Clean expired entries every 5 minutes
    setInterval(() => {
      const now = Date.now();
      for (const [key, entry] of this.cache.entries()) {
        if (now > entry.expiresAt) {
          this.cache.delete(key);
        }
      }
    }, 5 * 60 * 1000);
  }
}

export const advancedCache = new AdvancedCache();

// React hook for advanced caching
export function useAdvancedCache<T>(
  key: string,
  fetcher: () => Promise<T>,
  options: CacheOptions & { enabled?: boolean } = {}
) {
  const [data, setData] = useState<T | null>(null);
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState<Error | null>(null);
  const [isStale, setIsStale] = useState(false);

  const { enabled = true, staleWhileRevalidate = false } = options;

  const fetchData = useCallback(async (useStale = false) => {
    if (!enabled) return;

    const cached = useStale ? advancedCache.getStale<T>(key) : advancedCache.get<T>(key);
    
    if (cached && !useStale) {
      setData(cached);
      setIsStale(false);
      return;
    }

    if (cached && useStale) {
      setData(cached);
      setIsStale(true);
    }

    setIsLoading(true);
    setError(null);

    try {
      const result = await fetcher();
      advancedCache.set(key, result, options);
      setData(result);
      setIsStale(false);
    } catch (err) {
      setError(err instanceof Error ? err : new Error('Fetch failed'));
    } finally {
      setIsLoading(false);
    }
  }, [key, fetcher, enabled, options]);

  useEffect(() => {
    if (staleWhileRevalidate) {
      fetchData(true);
    } else {
      fetchData(false);
    }
  }, [fetchData, staleWhileRevalidate]);

  const invalidate = useCallback(() => {
    advancedCache.invalidateByPattern(new RegExp(key));
    fetchData(false);
  }, [key, fetchData]);

  return {
    data,
    isLoading,
    error,
    isStale,
    invalidate,
    refetch: () => fetchData(false),
  };
}
```

### Service Worker Caching

```typescript
// public/sw.js
const CACHE_NAME = 'nextjs-cache-v1';
const STATIC_CACHE = 'static-cache-v1';
const DYNAMIC_CACHE = 'dynamic-cache-v1';

const CACHE_STRATEGIES = {
  CACHE_FIRST: 'cache-first',
  NETWORK_FIRST: 'network-first',
  STALE_WHILE_REVALIDATE: 'stale-while-revalidate',
  NETWORK_ONLY: 'network-only',
  CACHE_ONLY: 'cache-only',
};

const ROUTES_CONFIG = [
  {
    pattern: /\/_next\/static\//,
    strategy: CACHE_STRATEGIES.CACHE_FIRST,
    cache: STATIC_CACHE,
    maxAge: 365 * 24 * 60 * 60, // 1 year
  },
  {
    pattern: /\/api\//,
    strategy: CACHE_STRATEGIES.NETWORK_FIRST,
    cache: DYNAMIC_CACHE,
    maxAge: 5 * 60, // 5 minutes
  },
  {
    pattern: /\.(png|jpg|jpeg|webp|svg|gif)$/,
    strategy: CACHE_STRATEGIES.STALE_WHILE_REVALIDATE,
    cache: STATIC_CACHE,
    maxAge: 30 * 24 * 60 * 60, // 30 days
  },
  {
    pattern: /\//,
    strategy: CACHE_STRATEGIES.STALE_WHILE_REVALIDATE,
    cache: DYNAMIC_CACHE,
    maxAge: 24 * 60 * 60, // 1 day
  },
];

self.addEventListener('install', (event) => {
  console.log('Service Worker installing');
  
  event.waitUntil(
    caches.open(STATIC_CACHE).then((cache) => {
      return cache.addAll([
        '/',
        '/_next/static/css/',
        '/_next/static/js/',
        '/favicon.ico',
      ]);
    })
  );
  
  self.skipWaiting();
});

self.addEventListener('activate', (event) => {
  console.log('Service Worker activating');
  
  event.waitUntil(
    caches.keys().then((cacheNames) => {
      return Promise.all(
        cacheNames.map((cacheName) => {
          if (cacheName !== CACHE_NAME && cacheName !== STATIC_CACHE && cacheName !== DYNAMIC_CACHE) {
            console.log('Deleting old cache:', cacheName);
            return caches.delete(cacheName);
          }
        })
      );
    })
  );
  
  self.clients.claim();
});

self.addEventListener('fetch', (event) => {
  const { request } = event;
  const url = new URL(request.url);

  // Skip non-GET requests
  if (request.method !== 'GET') {
    return;
  }

  // Find matching route configuration
  const routeConfig = ROUTES_CONFIG.find(config => 
    config.pattern.test(url.pathname)
  );

  if (routeConfig) {
    event.respondWith(handleRequest(request, routeConfig));
  }
});

async function handleRequest(request, config) {
  const { strategy, cache: cacheName, maxAge } = config;

  switch (strategy) {
    case CACHE_STRATEGIES.CACHE_FIRST:
      return cacheFirst(request, cacheName, maxAge);
    
    case CACHE_STRATEGIES.NETWORK_FIRST:
      return networkFirst(request, cacheName, maxAge);
    
    case CACHE_STRATEGIES.STALE_WHILE_REVALIDATE:
      return staleWhileRevalidate(request, cacheName, maxAge);
    
    case CACHE_STRATEGIES.NETWORK_ONLY:
      return fetch(request);
    
    case CACHE_STRATEGIES.CACHE_ONLY:
      return caches.match(request);
    
    default:
      return fetch(request);
  }
}

async function cacheFirst(request, cacheName, maxAge) {
  const cache = await caches.open(cacheName);
  const cached = await cache.match(request);
  
  if (cached && !isExpired(cached, maxAge)) {
    return cached;
  }
  
  try {
    const response = await fetch(request);
    if (response.ok) {
      cache.put(request, response.clone());
    }
    return response;
  } catch {
    return cached || new Response('Network error', { status: 408 });
  }
}

async function networkFirst(request, cacheName, maxAge) {
  const cache = await caches.open(cacheName);
  
  try {
    const response = await fetch(request);
    if (response.ok) {
      cache.put(request, response.clone());
    }
    return response;
  } catch {
    const cached = await cache.match(request);
    return cached || new Response('Network error', { status: 408 });
  }
}

async function staleWhileRevalidate(request, cacheName, maxAge) {
  const cache = await caches.open(cacheName);
  const cached = await cache.match(request);
  
  const fetchPromise = fetch(request).then(response => {
    if (response.ok) {
      cache.put(request, response.clone());
    }
    return response;
  });
  
  if (cached && !isExpired(cached, maxAge)) {
    return cached;
  }
  
  return fetchPromise;
}

function isExpired(response, maxAge) {
  const dateHeader = response.headers.get('date');
  if (!dateHeader) return false;
  
  const responseTime = new Date(dateHeader).getTime();
  const now = Date.now();
  const age = (now - responseTime) / 1000; // Convert to seconds
  
  return age > maxAge;
}

// Background sync for offline actions
self.addEventListener('sync', (event) => {
  if (event.tag === 'background-sync') {
    event.waitUntil(doBackgroundSync());
  }
});

async function doBackgroundSync() {
  // Handle offline actions when back online
  console.log('Performing background sync');
}
```

## Runtime Performance {#runtime-performance}

### React Performance Optimization

```typescript
// hooks/usePerformanceOptimized.ts
import { useCallback, useMemo, memo, forwardRef } from 'react';
import { debounce } from 'lodash-es';

// Optimized component creator
export function createOptimizedComponent<P extends object>(
  Component: React.ComponentType<P>,
  options: {
    memo?: boolean;
    displayName?: string;
    propsAreEqual?: (prevProps: P, nextProps: P) => boolean;
  } = {}
) {
  const { memo: useMemo = true, displayName, propsAreEqual } = options;

  let OptimizedComponent = Component;

  if (useMemo) {
    OptimizedComponent = memo(Component, propsAreEqual);
  }

  if (displayName) {
    OptimizedComponent.displayName = displayName;
  }

  return OptimizedComponent;
}

// Performance-optimized list component
interface VirtualizedListProps<T> {
  items: T[];
  itemHeight: number;
  containerHeight: number;
  renderItem: (item: T, index: number) => React.ReactNode;
  keyExtractor: (item: T, index: number) => string;
  overscan?: number;
}

export function VirtualizedList<T>({
  items,
  itemHeight,
  containerHeight,
  renderItem,
  keyExtractor,
  overscan = 5,
}: VirtualizedListProps<T>) {
  const [scrollTop, setScrollTop] = useState(0);

  const visibleRange = useMemo(() => {
    const startIndex = Math.max(0, Math.floor(scrollTop / itemHeight) - overscan);
    const endIndex = Math.min(
      items.length - 1,
      Math.ceil((scrollTop + containerHeight) / itemHeight) + overscan
    );
    
    return { startIndex, endIndex };
  }, [scrollTop, itemHeight, containerHeight, items.length, overscan]);

  const visibleItems = useMemo(() => {
    return items.slice(visibleRange.startIndex, visibleRange.endIndex + 1);
  }, [items, visibleRange]);

  const totalHeight = items.length * itemHeight;
  const offsetY = visibleRange.startIndex * itemHeight;

  const handleScroll = useCallback(
    debounce((e: React.UIEvent<HTMLDivElement>) => {
      setScrollTop(e.currentTarget.scrollTop);
    }, 16), // ~60fps
    []
  );

  return (
    <div
      style={{ height: containerHeight, overflow: 'auto' }}
      onScroll={handleScroll}
    >
      <div style={{ height: totalHeight, position: 'relative' }}>
        <div style={{ transform: `translateY(${offsetY}px)` }}>
          {visibleItems.map((item, index) => (
            <div
              key={keyExtractor(item, visibleRange.startIndex + index)}
              style={{ height: itemHeight }}
            >
              {renderItem(item, visibleRange.startIndex + index)}
            </div>
          ))}
        </div>
      </div>
    </div>
  );
}

// Performance monitoring hook
export function usePerformanceMonitor(componentName: string) {
  const renderCount = useRef(0);
  const lastRenderTime = useRef(performance.now());

  useEffect(() => {
    renderCount.current++;
    const now = performance.now();
    const renderTime = now - lastRenderTime.current;
    
    if (renderTime > 16) { // Slower than 60fps
      console.warn(
        `${componentName} render took ${renderTime.toFixed(2)}ms (render #${renderCount.current})`
      );
    }
    
    lastRenderTime.current = now;
  });

  const logRenderInfo = useCallback(() => {
    console.log(`${componentName} has rendered ${renderCount.current} times`);
  }, [componentName]);

  return { renderCount: renderCount.current, logRenderInfo };
}

// Optimized event handlers
export function useOptimizedEventHandlers() {
  const createDebouncedHandler = useCallback(
    <T extends any[]>(handler: (...args: T) => void, delay: number) => {
      return debounce(handler, delay);
    },
    []
  );

  const createThrottledHandler = useCallback(
    <T extends any[]>(handler: (...args: T) => void, limit: number) => {
      let inThrottle = false;
      
      return (...args: T) => {
        if (!inThrottle) {
          handler(...args);
          inThrottle = true;
          setTimeout(() => (inThrottle = false), limit);
        }
      };
    },
    []
  );

  return {
    createDebouncedHandler,
    createThrottledHandler,
  };
}
```

### State Management Performance

```typescript
// store/performantStore.ts
import { create } from 'zustand';
import { subscribeWithSelector } from 'zustand/middleware';
import { immer } from 'zustand/middleware/immer';

interface PerformantState {
  // Data
  users: User[];
  posts: Post[];
  comments: Comment[];
  
  // UI State
  selectedUserId: string | null;
  searchQuery: string;
  filters: FilterState;
  
  // Performance tracking
  renderCount: number;
  lastUpdateTime: number;
}

interface PerformantActions {
  // Data actions
  setUsers: (users: User[]) => void;
  addUser: (user: User) => void;
  updateUser: (id: string, updates: Partial<User>) => void;
  removeUser: (id: string) => void;
  
  // UI actions
  setSelectedUserId: (id: string | null) => void;
  setSearchQuery: (query: string) => void;
  updateFilters: (filters: Partial<FilterState>) => void;
  
  // Performance actions
  incrementRenderCount: () => void;
  updateLastUpdateTime: () => void;
}

// Create performant store with middleware
export const usePerformantStore = create<PerformantState & PerformantActions>()(
  subscribeWithSelector(
    immer((set, get) => ({
      // Initial state
      users: [],
      posts: [],
      comments: [],
      selectedUserId: null,
      searchQuery: '',
      filters: {
        category: 'all',
        sortBy: 'name',
        order: 'asc',
      },
      renderCount: 0,
      lastUpdateTime: Date.now(),

      // Actions
      setUsers: (users) =>
        set((state) => {
          state.users = users;
          state.lastUpdateTime = Date.now();
        }),

      addUser: (user) =>
        set((state) => {
          state.users.push(user);
          state.lastUpdateTime = Date.now();
        }),

      updateUser: (id, updates) =>
        set((state) => {
          const index = state.users.findIndex((user) => user.id === id);
          if (index !== -1) {
            Object.assign(state.users[index], updates);
            state.lastUpdateTime = Date.now();
          }
        }),

      removeUser: (id) =>
        set((state) => {
          state.users = state.users.filter((user) => user.id !== id);
          state.lastUpdateTime = Date.now();
        }),

      setSelectedUserId: (id) =>
        set((state) => {
          state.selectedUserId = id;
        }),

      setSearchQuery: (query) =>
        set((state) => {
          state.searchQuery = query;
        }),

      updateFilters: (filters) =>
        set((state) => {
          Object.assign(state.filters, filters);
        }),

      incrementRenderCount: () =>
        set((state) => {
          state.renderCount++;
        }),

      updateLastUpdateTime: () =>
        set((state) => {
          state.lastUpdateTime = Date.now();
        }),
    }))
  )
);

// Selectors for performance
export const useUsers = () => usePerformantStore((state) => state.users);
export const useSelectedUser = () => {
  const users = useUsers();
  const selectedUserId = usePerformantStore((state) => state.selectedUserId);
  
  return useMemo(() => {
    return users.find((user) => user.id === selectedUserId) || null;
  }, [users, selectedUserId]);
};

export const useFilteredUsers = () => {
  const users = useUsers();
  const searchQuery = usePerformantStore((state) => state.searchQuery);
  const filters = usePerformantStore((state) => state.filters);
  
  return useMemo(() => {
    let filtered = users;
    
    // Apply search
    if (searchQuery) {
      filtered = filtered.filter((user) =>
        user.name.toLowerCase().includes(searchQuery.toLowerCase()) ||
        user.email.toLowerCase().includes(searchQuery.toLowerCase())
      );
    }
    
    // Apply filters
    if (filters.category !== 'all') {
      filtered = filtered.filter((user) => user.category === filters.category);
    }
    
    // Apply sorting
    filtered.sort((a, b) => {
      const aValue = a[filters.sortBy as keyof User];
      const bValue = b[filters.sortBy as keyof User];
      
      if (filters.order === 'asc') {
        return aValue > bValue ? 1 : -1;
      } else {
        return aValue < bValue ? 1 : -1;
      }
    });
    
    return filtered;
  }, [users, searchQuery, filters]);
};

// Performance monitoring
export const useStorePerformance = () => {
  const renderCount = usePerformantStore((state) => state.renderCount);
  const lastUpdateTime = usePerformantStore((state) => state.lastUpdateTime);
  
  useEffect(() => {
    usePerformantStore.getState().incrementRenderCount();
  });
  
  return {
    renderCount,
    lastUpdateTime,
    timeSinceLastUpdate: Date.now() - lastUpdateTime,
  };
};
```

## Best Practices {#best-practices}

### Performance Optimization Checklist

```typescript
// utils/performanceChecklist.ts
interface PerformanceCheck {
  category: 'Core Web Vitals' | 'Bundle' | 'Runtime' | 'Network' | 'Accessibility';
  name: string;
  description: string;
  check: () => Promise<boolean> | boolean;
  fix: string;
  priority: 'high' | 'medium' | 'low';
}

export const performanceChecks: PerformanceCheck[] = [
  // Core Web Vitals
  {
    category: 'Core Web Vitals',
    name: 'LCP < 2.5s',
    description: 'Largest Contentful Paint should be under 2.5 seconds',
    check: async () => {
      const lcp = await measureLCP();
      return lcp < 2500;
    },
    fix: 'Optimize images, use priority loading, reduce server response time',
    priority: 'high',
  },
  {
    category: 'Core Web Vitals',
    name: 'FID < 100ms',
    description: 'First Input Delay should be under 100 milliseconds',
    check: async () => {
      const fid = await measureFID();
      return fid < 100;
    },
    fix: 'Reduce JavaScript execution time, use code splitting',
    priority: 'high',
  },
  {
    category: 'Core Web Vitals',
    name: 'CLS < 0.1',
    description: 'Cumulative Layout Shift should be under 0.1',
    check: async () => {
      const cls = await measureCLS();
      return cls < 0.1;
    },
    fix: 'Add width/height to images, reserve space for dynamic content',
    priority: 'high',
  },

  // Bundle optimizations
  {
    category: 'Bundle',
    name: 'Bundle size < 300KB',
    description: 'Main bundle should be under 300KB',
    check: () => {
      return getBundleSize() < 300 * 1024;
    },
    fix: 'Use dynamic imports, tree shaking, remove unused dependencies',
    priority: 'high',
  },
  {
    category: 'Bundle',
    name: 'No duplicate dependencies',
    description: 'Should not have duplicate dependencies in bundle',
    check: () => {
      return getDuplicateDependencies().length === 0;
    },
    fix: 'Use webpack-bundle-analyzer to identify and remove duplicates',
    priority: 'medium',
  },

  // Runtime performance
  {
    category: 'Runtime',
    name: 'React DevTools profiling',
    description: 'Components should not have unnecessary re-renders',
    check: () => {
      return getUnnecessaryRerenders() < 5;
    },
    fix: 'Use React.memo, useMemo, useCallback appropriately',
    priority: 'medium',
  },
  {
    category: 'Runtime',
    name: 'Memory leaks',
    description: 'Should not have memory leaks',
    check: () => {
      return !hasMemoryLeaks();
    },
    fix: 'Clean up event listeners, cancel requests in useEffect cleanup',
    priority: 'high',
  },

  // Network optimization
  {
    category: 'Network',
    name: 'Image optimization',
    description: 'Images should be optimized and use modern formats',
    check: () => {
      return areImagesOptimized();
    },
    fix: 'Use Next.js Image component, WebP/AVIF formats',
    priority: 'high',
  },
  {
    category: 'Network',
    name: 'Gzip compression',
    description: 'Static assets should be compressed',
    check: () => {
      return isCompressionEnabled();
    },
    fix: 'Enable gzip/brotli compression in server configuration',
    priority: 'medium',
  },
];

// Run performance audit
export async function runPerformanceAudit(): Promise<{
  passed: PerformanceCheck[];
  failed: PerformanceCheck[];
  score: number;
}> {
  const results = await Promise.all(
    performanceChecks.map(async (check) => ({
      check,
      passed: await check.check(),
    }))
  );

  const passed = results.filter((r) => r.passed).map((r) => r.check);
  const failed = results.filter((r) => !r.passed).map((r) => r.check);
  
  const score = (passed.length / performanceChecks.length) * 100;

  return { passed, failed, score };
}

// Helper functions (implement based on your specific needs)
async function measureLCP(): Promise<number> {
  return new Promise((resolve) => {
    const observer = new PerformanceObserver((list) => {
      const entries = list.getEntries();
      const lastEntry = entries[entries.length - 1] as any;
      resolve(lastEntry.startTime);
    });
    observer.observe({ entryTypes: ['largest-contentful-paint'] });
  });
}

async function measureFID(): Promise<number> {
  return new Promise((resolve) => {
    const observer = new PerformanceObserver((list) => {
      const entries = list.getEntries();
      const lastEntry = entries[entries.length - 1] as any;
      resolve(lastEntry.processingStart - lastEntry.startTime);
    });
    observer.observe({ entryTypes: ['first-input'] });
  });
}

async function measureCLS(): Promise<number> {
  return new Promise((resolve) => {
    let clsValue = 0;
    const observer = new PerformanceObserver((list) => {
      for (const entry of list.getEntries()) {
        if (!(entry as any).hadRecentInput) {
          clsValue += (entry as any).value;
        }
      }
      resolve(clsValue);
    });
    observer.observe({ entryTypes: ['layout-shift'] });
    
    // Resolve after 5 seconds
    setTimeout(() => resolve(clsValue), 5000);
  });
}

function getBundleSize(): number {
  // Implementation depends on your build setup
  return 0;
}

function getDuplicateDependencies(): string[] {
  // Implementation depends on your build setup
  return [];
}

function getUnnecessaryRerenders(): number {
  // Implementation depends on React DevTools profiling
  return 0;
}

function hasMemoryLeaks(): boolean {
  // Implementation depends on memory profiling
  return false;
}

function areImagesOptimized(): boolean {
  const images = document.querySelectorAll('img');
  for (const img of images) {
    if (!img.src.includes('.webp') && !img.src.includes('.avif')) {
      return false;
    }
  }
  return true;
}

function isCompressionEnabled(): boolean {
  // Check if resources are compressed
  return true; // Implement actual check
}
```

## Exercises

### Exercise 1: Performance Dashboard
Create a comprehensive performance monitoring dashboard with:
- Real-time Core Web Vitals tracking
- Bundle size analysis and alerts
- Performance budget enforcement
- Historical performance data
- Automated performance testing

### Exercise 2: E-commerce Performance Optimization
Optimize an e-commerce application for performance:
- Product list virtualization
- Image lazy loading and optimization
- Search debouncing and caching
- Cart state optimization
- Checkout flow performance

### Exercise 3: Data-Heavy Application
Build a performant data visualization app with:
- Large dataset handling
- Virtual scrolling for tables
- Chart rendering optimization
- Real-time data updates
- Memory management

### Exercise 4: Mobile Performance Optimization
Optimize a Next.js app specifically for mobile:
- Touch interaction optimization
- Reduced bundle sizes for mobile
- Offline functionality
- Battery usage optimization
- Progressive Web App features

## Summary

Performance optimization in Next.js requires a holistic approach:

- **Core Web Vitals**: Focus on LCP, FID, and CLS metrics
- **Bundle optimization**: Code splitting, tree shaking, and analysis
- **Caching strategies**: Advanced caching and service workers
- **Runtime performance**: Component optimization and state management
- **Monitoring**: Continuous performance tracking and alerts

Key principles:
- Set and enforce performance budgets
- Measure everything that matters
- Optimize based on real user data
- Use automated testing and monitoring
- Balance performance with developer experience

Next, we'll explore **Authentication** patterns and security implementations in Next.js applications!
