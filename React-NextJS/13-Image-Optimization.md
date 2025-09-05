# Image Optimization in Next.js

## Table of Contents
1. [Introduction to Image Optimization](#introduction)
2. [Next.js Image Component](#nextjs-image)
3. [Image Loading Strategies](#loading-strategies)
4. [Responsive Images](#responsive-images)
5. [Image Formats and Optimization](#formats)
6. [Advanced Image Techniques](#advanced-techniques)
7. [Custom Image Loaders](#custom-loaders)
8. [Image CDN Integration](#cdn-integration)
9. [Performance Monitoring](#performance-monitoring)
10. [Accessibility and SEO](#accessibility-seo)
11. [Best Practices](#best-practices)

## Introduction to Image Optimization {#introduction}

Images often account for the majority of bytes downloaded on web pages. Next.js provides powerful built-in image optimization features that can significantly improve your application's performance and user experience.

### Why Image Optimization Matters

```typescript
// Performance impact analysis
interface ImageMetrics {
  originalSize: number;
  optimizedSize: number;
  loadTime: number;
  coreWebVitals: {
    lcp: number; // Largest Contentful Paint
    cls: number; // Cumulative Layout Shift
    fid: number; // First Input Delay
  };
}

const imageOptimizationBenefits = {
  // File size reduction
  sizeSavings: '60-80%', // Typical savings with modern formats
  
  // Performance improvements
  loadTimeImprovement: '40-60%',
  
  // SEO benefits
  seoImpact: {
    pagespeedScore: '+20-30 points',
    searchRanking: 'Improved',
    userExperience: 'Enhanced',
  },
  
  // Business impact
  conversionRate: '+2-5%',
  bounceRate: '-10-15%',
};
```

### Next.js Image Optimization Features

```typescript
// Built-in optimization features
const nextjsImageFeatures = {
  automaticOptimization: {
    webp: 'Automatic WebP generation for supported browsers',
    avif: 'AVIF format for ultra-modern browsers',
    sizing: 'Automatic responsive sizing',
    quality: 'Intelligent quality adjustment',
  },
  
  performanceFeatures: {
    lazyLoading: 'Built-in lazy loading',
    priorityLoading: 'Above-the-fold optimization',
    placeholders: 'Blur and solid color placeholders',
    preloading: 'Intelligent preloading',
  },
  
  layoutStability: {
    aspectRatio: 'Maintains aspect ratio',
    layoutShift: 'Prevents Cumulative Layout Shift',
    responsive: 'Responsive design support',
  },
};
```

## Next.js Image Component {#nextjs-image}

### Basic Usage

```typescript
// components/OptimizedImage.tsx
import Image from 'next/image';
import { useState } from 'react';

interface OptimizedImageProps {
  src: string;
  alt: string;
  width?: number;
  height?: number;
  priority?: boolean;
  className?: string;
  objectFit?: 'contain' | 'cover' | 'fill' | 'none' | 'scale-down';
  quality?: number;
}

export default function OptimizedImage({
  src,
  alt,
  width,
  height,
  priority = false,
  className = '',
  objectFit = 'cover',
  quality = 75,
}: OptimizedImageProps) {
  const [isLoading, setIsLoading] = useState(true);
  const [hasError, setHasError] = useState(false);

  return (
    <div className={`relative overflow-hidden ${className}`}>
      {isLoading && (
        <div className="absolute inset-0 bg-gray-200 animate-pulse rounded-lg" />
      )}
      
      {hasError ? (
        <div className="absolute inset-0 bg-gray-100 flex items-center justify-center rounded-lg">
          <div className="text-center text-gray-500">
            <svg className="w-12 h-12 mx-auto mb-2" fill="currentColor" viewBox="0 0 20 20">
              <path fillRule="evenodd" d="M4 3a2 2 0 00-2 2v10a2 2 0 002 2h12a2 2 0 002-2V5a2 2 0 00-2-2H4zm12 12H4l4-8 3 6 2-4 3 6z" clipRule="evenodd" />
            </svg>
            <p className="text-sm">Image not available</p>
          </div>
        </div>
      ) : (
        <Image
          src={src}
          alt={alt}
          width={width}
          height={height}
          priority={priority}
          quality={quality}
          style={{
            objectFit,
            width: '100%',
            height: '100%',
          }}
          onLoad={() => setIsLoading(false)}
          onError={() => {
            setIsLoading(false);
            setHasError(true);
          }}
          placeholder="blur"
          blurDataURL="data:image/jpeg;base64,/9j/4AAQSkZJRgABAQAAAQABAAD/2wBDAAYEBQYFBAYGBQYHBwYIChAKCgkJChQODwwQFxQYGBcUFhYaHSUfGhsjHBYWICwgIyYnKSopGR8tMC0oMCUoKSj/2wBDAQcHBwoIChMKChMoGhYaKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCj/wAARCAAIAAoDASIAAhEBAxEB/8QAFQABAQAAAAAAAAAAAAAAAAAAAAv/xAAhEAACAQMDBQAAAAAAAAAAAAABAgMABAUGIWGRkqGx0f/EABUBAQEAAAAAAAAAAAAAAAAAAAMF/8QAGhEAAgIDAAAAAAAAAAAAAAAAAAECEgMRkf/aAAwDAQACEQMRAD8AltJagyeH0AthI5xdrLcNM91BF5pX2HaH9bcfaSXWGaRmknyJckliyjqTzSlT54b6bk+h0R//2Q=="
        />
      )}
    </div>
  );
}
```

### Advanced Image Component with Multiple Sources

```typescript
// components/ResponsiveImage.tsx
import Image from 'next/image';
import { useState, useEffect } from 'react';

interface ImageSource {
  src: string;
  width: number;
  height: number;
  media?: string; // Media query
}

interface ResponsiveImageProps {
  sources: ImageSource[];
  alt: string;
  priority?: boolean;
  className?: string;
  sizes?: string;
  quality?: number;
  placeholder?: 'blur' | 'empty';
  blurDataURL?: string;
  onLoad?: () => void;
  onError?: () => void;
}

export default function ResponsiveImage({
  sources,
  alt,
  priority = false,
  className = '',
  sizes = '100vw',
  quality = 75,
  placeholder = 'blur',
  blurDataURL,
  onLoad,
  onError,
}: ResponsiveImageProps) {
  const [currentSource, setCurrentSource] = useState(sources[0]);
  const [isLoading, setIsLoading] = useState(true);
  const [hasError, setHasError] = useState(false);

  // Update source based on media queries
  useEffect(() => {
    const updateSource = () => {
      for (const source of sources) {
        if (source.media) {
          if (window.matchMedia(source.media).matches) {
            setCurrentSource(source);
            return;
          }
        }
      }
      // Fallback to first source
      setCurrentSource(sources[0]);
    };

    updateSource();

    // Listen for viewport changes
    const mediaQueries = sources
      .filter(source => source.media)
      .map(source => window.matchMedia(source.media!));

    mediaQueries.forEach(mq => mq.addListener(updateSource));

    return () => {
      mediaQueries.forEach(mq => mq.removeListener(updateSource));
    };
  }, [sources]);

  const handleLoad = () => {
    setIsLoading(false);
    onLoad?.();
  };

  const handleError = () => {
    setIsLoading(false);
    setHasError(true);
    onError?.();
  };

  if (hasError) {
    return (
      <div className={`bg-gray-100 flex items-center justify-center ${className}`}>
        <div className="text-center text-gray-500">
          <svg className="w-12 h-12 mx-auto mb-2" fill="currentColor" viewBox="0 0 20 20">
            <path fillRule="evenodd" d="M4 3a2 2 0 00-2 2v10a2 2 0 002 2h12a2 2 0 002-2V5a2 2 0 00-2-2H4zm12 12H4l4-8 3 6 2-4 3 6z" clipRule="evenodd" />
          </svg>
          <p className="text-sm">Failed to load image</p>
        </div>
      </div>
    );
  }

  return (
    <div className={`relative ${className}`}>
      {isLoading && (
        <div className="absolute inset-0 bg-gray-200 animate-pulse" />
      )}
      
      <Image
        src={currentSource.src}
        alt={alt}
        width={currentSource.width}
        height={currentSource.height}
        priority={priority}
        quality={quality}
        sizes={sizes}
        placeholder={placeholder}
        blurDataURL={blurDataURL}
        onLoad={handleLoad}
        onError={handleError}
        style={{
          width: '100%',
          height: 'auto',
        }}
      />
    </div>
  );
}

// Usage example
const ExampleUsage = () => {
  const imageSources: ImageSource[] = [
    {
      src: '/images/hero-mobile.jpg',
      width: 480,
      height: 320,
      media: '(max-width: 768px)',
    },
    {
      src: '/images/hero-tablet.jpg',
      width: 1024,
      height: 576,
      media: '(max-width: 1024px)',
    },
    {
      src: '/images/hero-desktop.jpg',
      width: 1920,
      height: 1080,
      media: '(min-width: 1025px)',
    },
  ];

  return (
    <ResponsiveImage
      sources={imageSources}
      alt="Hero image"
      priority
      className="w-full h-64 md:h-96 lg:h-[500px]"
      sizes="(max-width: 768px) 100vw, (max-width: 1024px) 50vw, 33vw"
    />
  );
};
```

## Image Loading Strategies {#loading-strategies}

### Lazy Loading with Intersection Observer

```typescript
// hooks/useLazyLoading.ts
import { useState, useEffect, useRef, useCallback } from 'react';

interface UseLazyLoadingOptions {
  threshold?: number;
  rootMargin?: string;
  triggerOnce?: boolean;
}

export function useLazyLoading({
  threshold = 0.1,
  rootMargin = '50px',
  triggerOnce = true,
}: UseLazyLoadingOptions = {}) {
  const [isIntersecting, setIsIntersecting] = useState(false);
  const [hasIntersected, setHasIntersected] = useState(false);
  const elementRef = useRef<HTMLElement>(null);

  const setRef = useCallback((element: HTMLElement | null) => {
    elementRef.current = element;
  }, []);

  useEffect(() => {
    const element = elementRef.current;
    if (!element) return;

    const observer = new IntersectionObserver(
      ([entry]) => {
        const isElementIntersecting = entry.isIntersecting;
        setIsIntersecting(isElementIntersecting);

        if (isElementIntersecting && !hasIntersected) {
          setHasIntersected(true);
          
          if (triggerOnce) {
            observer.unobserve(element);
          }
        }
      },
      {
        threshold,
        rootMargin,
      }
    );

    observer.observe(element);

    return () => {
      observer.unobserve(element);
    };
  }, [threshold, rootMargin, triggerOnce, hasIntersected]);

  return {
    setRef,
    isIntersecting,
    hasIntersected,
    shouldLoad: triggerOnce ? hasIntersected : isIntersecting,
  };
}
```

### Progressive Image Loading

```typescript
// components/ProgressiveImage.tsx
import { useState, useRef, useEffect } from 'react';
import Image from 'next/image';

interface ProgressiveImageProps {
  src: string;
  lowQualitySrc?: string;
  alt: string;
  width: number;
  height: number;
  className?: string;
  priority?: boolean;
}

export default function ProgressiveImage({
  src,
  lowQualitySrc,
  alt,
  width,
  height,
  className = '',
  priority = false,
}: ProgressiveImageProps) {
  const [imageLoaded, setImageLoaded] = useState(false);
  const [imageError, setImageError] = useState(false);
  const [lowQualityLoaded, setLowQualityLoaded] = useState(false);
  const imgRef = useRef<HTMLImageElement>(null);

  // Preload low quality image
  useEffect(() => {
    if (lowQualitySrc) {
      const img = new window.Image();
      img.onload = () => setLowQualityLoaded(true);
      img.onerror = () => setLowQualityLoaded(false);
      img.src = lowQualitySrc;
    }
  }, [lowQualitySrc]);

  const handleLoad = () => {
    setImageLoaded(true);
  };

  const handleError = () => {
    setImageError(true);
  };

  return (
    <div className={`relative overflow-hidden ${className}`}>
      {/* Low quality placeholder */}
      {lowQualitySrc && lowQualityLoaded && !imageLoaded && (
        <div className="absolute inset-0">
          <Image
            src={lowQualitySrc}
            alt={alt}
            width={width}
            height={height}
            quality={10}
            style={{
              width: '100%',
              height: '100%',
              objectFit: 'cover',
              filter: 'blur(5px)',
              transform: 'scale(1.1)', // Prevent blur edge artifacts
            }}
          />
        </div>
      )}

      {/* Loading skeleton */}
      {!lowQualityLoaded && !imageLoaded && (
        <div className="absolute inset-0 bg-gray-200 animate-pulse" />
      )}

      {/* High quality image */}
      {!imageError && (
        <div
          className={`transition-opacity duration-300 ${
            imageLoaded ? 'opacity-100' : 'opacity-0'
          }`}
        >
          <Image
            ref={imgRef}
            src={src}
            alt={alt}
            width={width}
            height={height}
            priority={priority}
            quality={85}
            onLoad={handleLoad}
            onError={handleError}
            style={{
              width: '100%',
              height: '100%',
              objectFit: 'cover',
            }}
          />
        </div>
      )}

      {/* Error fallback */}
      {imageError && (
        <div className="absolute inset-0 bg-gray-100 flex items-center justify-center">
          <div className="text-center text-gray-500">
            <svg className="w-12 h-12 mx-auto mb-2" fill="currentColor" viewBox="0 0 20 20">
              <path fillRule="evenodd" d="M4 3a2 2 0 00-2 2v10a2 2 0 002 2h12a2 2 0 002-2V5a2 2 0 00-2-2H4zm12 12H4l4-8 3 6 2-4 3 6z" clipRule="evenodd" />
            </svg>
            <p className="text-sm">Image failed to load</p>
          </div>
        </div>
      )}

      {/* Loading indicator */}
      {!imageLoaded && !imageError && (
        <div className="absolute inset-0 flex items-center justify-center">
          <div className="w-8 h-8 border-2 border-blue-600 border-t-transparent rounded-full animate-spin" />
        </div>
      )}
    </div>
  );
}
```

### Image Gallery with Advanced Loading

```typescript
// components/ImageGallery.tsx
import { useState, useCallback, useMemo } from 'react';
import { useLazyLoading } from '../hooks/useLazyLoading';
import ProgressiveImage from './ProgressiveImage';

interface GalleryImage {
  id: string;
  src: string;
  thumbnail: string;
  lowQuality: string;
  alt: string;
  width: number;
  height: number;
  category?: string;
}

interface ImageGalleryProps {
  images: GalleryImage[];
  columns?: number;
  gap?: number;
  category?: string;
  onImageClick?: (image: GalleryImage) => void;
}

export default function ImageGallery({
  images,
  columns = 3,
  gap = 4,
  category,
  onImageClick,
}: ImageGalleryProps) {
  const [selectedCategory, setSelectedCategory] = useState(category || 'all');
  const [loadedImages, setLoadedImages] = useState<Set<string>>(new Set());

  // Filter images by category
  const filteredImages = useMemo(() => {
    if (selectedCategory === 'all') return images;
    return images.filter(img => img.category === selectedCategory);
  }, [images, selectedCategory]);

  // Get unique categories
  const categories = useMemo(() => {
    const cats = Array.from(new Set(images.map(img => img.category).filter(Boolean)));
    return ['all', ...cats];
  }, [images]);

  const handleImageLoad = useCallback((imageId: string) => {
    setLoadedImages(prev => new Set(prev).add(imageId));
  }, []);

  return (
    <div className="space-y-6">
      {/* Category Filter */}
      {categories.length > 1 && (
        <div className="flex flex-wrap gap-2">
          {categories.map(cat => (
            <button
              key={cat}
              onClick={() => setSelectedCategory(cat)}
              className={`px-4 py-2 rounded-lg text-sm font-medium transition-colors ${
                selectedCategory === cat
                  ? 'bg-blue-600 text-white'
                  : 'bg-gray-100 text-gray-700 hover:bg-gray-200'
              }`}
            >
              {cat.charAt(0).toUpperCase() + cat.slice(1)}
            </button>
          ))}
        </div>
      )}

      {/* Image Grid */}
      <div
        className={`grid gap-${gap}`}
        style={{
          gridTemplateColumns: `repeat(${columns}, minmax(0, 1fr))`,
        }}
      >
        {filteredImages.map((image, index) => (
          <GalleryImageItem
            key={image.id}
            image={image}
            index={index}
            onLoad={() => handleImageLoad(image.id)}
            onClick={() => onImageClick?.(image)}
          />
        ))}
      </div>

      {/* Loading stats */}
      <div className="text-center text-sm text-gray-500">
        Loaded {loadedImages.size} of {filteredImages.length} images
      </div>
    </div>
  );
}

interface GalleryImageItemProps {
  image: GalleryImage;
  index: number;
  onLoad: () => void;
  onClick: () => void;
}

function GalleryImageItem({ image, index, onLoad, onClick }: GalleryImageItemProps) {
  const { setRef, shouldLoad } = useLazyLoading({
    rootMargin: '200px', // Start loading 200px before the image enters viewport
    triggerOnce: true,
  });

  const handleLoad = () => {
    onLoad();
  };

  return (
    <div
      ref={setRef}
      className="relative aspect-square cursor-pointer group"
      onClick={onClick}
    >
      {shouldLoad ? (
        <ProgressiveImage
          src={image.src}
          lowQualitySrc={image.lowQuality}
          alt={image.alt}
          width={image.width}
          height={image.height}
          className="w-full h-full rounded-lg overflow-hidden"
          priority={index < 6} // Prioritize first 6 images
        />
      ) : (
        <div className="w-full h-full bg-gray-200 animate-pulse rounded-lg" />
      )}

      {/* Hover overlay */}
      <div className="absolute inset-0 bg-black bg-opacity-0 group-hover:bg-opacity-30 transition-all duration-200 rounded-lg flex items-center justify-center">
        <div className="opacity-0 group-hover:opacity-100 transition-opacity duration-200">
          <svg className="w-8 h-8 text-white" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z" />
          </svg>
        </div>
      </div>

      {/* Image info */}
      <div className="absolute bottom-0 left-0 right-0 bg-gradient-to-t from-black to-transparent p-3 rounded-b-lg opacity-0 group-hover:opacity-100 transition-opacity duration-200">
        <p className="text-white text-sm truncate">{image.alt}</p>
        {image.category && (
          <p className="text-gray-300 text-xs">{image.category}</p>
        )}
      </div>
    </div>
  );
}
```

## Responsive Images {#responsive-images}

### Art Direction with Picture Element

```typescript
// components/ArtDirectedImage.tsx
import { useState, useEffect } from 'react';

interface ArtDirectedSource {
  srcSet: string;
  media: string;
  type?: string;
}

interface ArtDirectedImageProps {
  sources: ArtDirectedSource[];
  fallbackSrc: string;
  alt: string;
  className?: string;
  loading?: 'lazy' | 'eager';
  onLoad?: () => void;
  onError?: () => void;
}

export default function ArtDirectedImage({
  sources,
  fallbackSrc,
  alt,
  className = '',
  loading = 'lazy',
  onLoad,
  onError,
}: ArtDirectedImageProps) {
  const [isLoaded, setIsLoaded] = useState(false);
  const [hasError, setHasError] = useState(false);

  const handleLoad = () => {
    setIsLoaded(true);
    onLoad?.();
  };

  const handleError = () => {
    setHasError(true);
    onError?.();
  };

  if (hasError) {
    return (
      <div className={`bg-gray-100 flex items-center justify-center ${className}`}>
        <div className="text-center text-gray-500">
          <svg className="w-12 h-12 mx-auto mb-2" fill="currentColor" viewBox="0 0 20 20">
            <path fillRule="evenodd" d="M4 3a2 2 0 00-2 2v10a2 2 0 002 2h12a2 2 0 002-2V5a2 2 0 00-2-2H4zm12 12H4l4-8 3 6 2-4 3 6z" clipRule="evenodd" />
          </svg>
          <p className="text-sm">Image not available</p>
        </div>
      </div>
    );
  }

  return (
    <div className={`relative ${className}`}>
      {!isLoaded && (
        <div className="absolute inset-0 bg-gray-200 animate-pulse" />
      )}
      
      <picture>
        {sources.map((source, index) => (
          <source
            key={index}
            srcSet={source.srcSet}
            media={source.media}
            type={source.type}
          />
        ))}
        <img
          src={fallbackSrc}
          alt={alt}
          loading={loading}
          onLoad={handleLoad}
          onError={handleError}
          className={`w-full h-full object-cover transition-opacity duration-300 ${
            isLoaded ? 'opacity-100' : 'opacity-0'
          }`}
        />
      </picture>
    </div>
  );
}

// Usage example with art direction
const HeroWithArtDirection = () => {
  const artDirectedSources: ArtDirectedSource[] = [
    {
      srcSet: '/images/hero-mobile.webp 1x, /images/hero-mobile@2x.webp 2x',
      media: '(max-width: 767px)',
      type: 'image/webp',
    },
    {
      srcSet: '/images/hero-mobile.jpg 1x, /images/hero-mobile@2x.jpg 2x',
      media: '(max-width: 767px)',
    },
    {
      srcSet: '/images/hero-tablet.webp 1x, /images/hero-tablet@2x.webp 2x',
      media: '(max-width: 1023px)',
      type: 'image/webp',
    },
    {
      srcSet: '/images/hero-tablet.jpg 1x, /images/hero-tablet@2x.jpg 2x',
      media: '(max-width: 1023px)',
    },
    {
      srcSet: '/images/hero-desktop.webp 1x, /images/hero-desktop@2x.webp 2x',
      media: '(min-width: 1024px)',
      type: 'image/webp',
    },
    {
      srcSet: '/images/hero-desktop.jpg 1x, /images/hero-desktop@2x.jpg 2x',
      media: '(min-width: 1024px)',
    },
  ];

  return (
    <ArtDirectedImage
      sources={artDirectedSources}
      fallbackSrc="/images/hero-desktop.jpg"
      alt="Hero banner showcasing our products"
      className="w-full h-96 md:h-[500px] lg:h-[600px]"
      loading="eager"
    />
  );
};
```

### Dynamic Image Sizing

```typescript
// components/DynamicImage.tsx
import Image from 'next/image';
import { useState, useEffect } from 'react';

interface DynamicImageProps {
  src: string;
  alt: string;
  aspectRatio?: number;
  maxWidth?: number;
  className?: string;
  priority?: boolean;
  quality?: number;
}

export default function DynamicImage({
  src,
  alt,
  aspectRatio = 16 / 9,
  maxWidth = 1200,
  className = '',
  priority = false,
  quality = 75,
}: DynamicImageProps) {
  const [dimensions, setDimensions] = useState({ width: 0, height: 0 });
  const [containerWidth, setContainerWidth] = useState(0);

  useEffect(() => {
    const updateContainerWidth = () => {
      const container = document.querySelector('[data-dynamic-image-container]');
      if (container) {
        setContainerWidth(container.clientWidth);
      }
    };

    updateContainerWidth();
    window.addEventListener('resize', updateContainerWidth);

    return () => window.removeEventListener('resize', updateContainerWidth);
  }, []);

  useEffect(() => {
    if (containerWidth > 0) {
      const width = Math.min(containerWidth, maxWidth);
      const height = width / aspectRatio;
      setDimensions({ width, height });
    }
  }, [containerWidth, maxWidth, aspectRatio]);

  // Generate responsive sizes attribute
  const sizes = `(max-width: 640px) 100vw, (max-width: 1024px) 75vw, ${maxWidth}px`;

  if (dimensions.width === 0) {
    return (
      <div 
        className={`bg-gray-200 animate-pulse ${className}`}
        style={{ aspectRatio }}
        data-dynamic-image-container
      />
    );
  }

  return (
    <div data-dynamic-image-container className={className}>
      <Image
        src={src}
        alt={alt}
        width={dimensions.width}
        height={dimensions.height}
        priority={priority}
        quality={quality}
        sizes={sizes}
        style={{
          width: '100%',
          height: 'auto',
        }}
      />
    </div>
  );
}
```

## Custom Image Loaders {#custom-loaders}

### Cloudinary Image Loader

```typescript
// lib/imageLoaders.ts
interface CloudinaryLoaderProps {
  src: string;
  width: number;
  quality?: number;
}

export function cloudinaryLoader({ src, width, quality }: CloudinaryLoaderProps): string {
  const params = ['f_auto', 'c_limit', `w_${width}`, `q_${quality || 'auto'}`];
  
  // Add responsive breakpoints
  if (width <= 640) {
    params.push('c_fill', 'ar_1:1'); // Square for mobile
  } else if (width <= 1024) {
    params.push('c_fill', 'ar_4:3'); // 4:3 for tablet
  } else {
    params.push('c_fill', 'ar_16:9'); // 16:9 for desktop
  }

  return `https://res.cloudinary.com/your-cloud-name/image/fetch/${params.join(',')}/${src}`;
}

// Custom hook for Cloudinary images
export function useCloudinaryImage(src: string, transformations?: string[]) {
  const getUrl = (width: number, quality?: number) => {
    let url = cloudinaryLoader({ src, width, quality });
    
    if (transformations?.length) {
      const baseUrl = url.split('/image/fetch/')[0];
      const existingParams = url.split('/image/fetch/')[1].split('/')[0];
      const imageSrc = url.split('/image/fetch/')[1].split('/').slice(1).join('/');
      
      const allParams = [existingParams, ...transformations].join(',');
      url = `${baseUrl}/image/fetch/${allParams}/${imageSrc}`;
    }
    
    return url;
  };

  return { getUrl };
}
```

### Multi-CDN Image Loader

```typescript
// lib/multiCdnLoader.ts
interface ImageCDN {
  name: string;
  baseUrl: string;
  transformParams: (width: number, quality: number) => string;
  isAvailable: () => Promise<boolean>;
}

const imageCDNs: ImageCDN[] = [
  {
    name: 'cloudinary',
    baseUrl: 'https://res.cloudinary.com/your-cloud-name/image/fetch',
    transformParams: (width, quality) => `f_auto,c_limit,w_${width},q_${quality}`,
    isAvailable: async () => {
      try {
        const response = await fetch('https://res.cloudinary.com/your-cloud-name/image/upload/sample.jpg', { method: 'HEAD' });
        return response.ok;
      } catch {
        return false;
      }
    },
  },
  {
    name: 'imagekit',
    baseUrl: 'https://ik.imagekit.io/your-imagekit-id',
    transformParams: (width, quality) => `tr=w-${width},q-${quality},f-auto`,
    isAvailable: async () => {
      try {
        const response = await fetch('https://ik.imagekit.io/your-imagekit-id', { method: 'HEAD' });
        return response.ok;
      } catch {
        return false;
      }
    },
  },
  {
    name: 'vercel',
    baseUrl: '',
    transformParams: (width, quality) => `w=${width}&q=${quality}`,
    isAvailable: async () => true, // Always available as fallback
  },
];

class MultiCDNImageLoader {
  private availableCDNs: ImageCDN[] = [];
  private initialized = false;

  async initialize() {
    if (this.initialized) return;

    const availabilityChecks = imageCDNs.map(async (cdn) => ({
      cdn,
      available: await cdn.isAvailable(),
    }));

    const results = await Promise.all(availabilityChecks);
    this.availableCDNs = results
      .filter(result => result.available)
      .map(result => result.cdn);

    this.initialized = true;
  }

  async getImageUrl(src: string, width: number, quality: number = 75): Promise<string> {
    if (!this.initialized) {
      await this.initialize();
    }

    const cdn = this.availableCDNs[0]; // Use first available CDN
    
    if (!cdn) {
      return src; // Fallback to original src
    }

    if (cdn.name === 'vercel') {
      // Use Next.js built-in loader
      return `/_next/image?url=${encodeURIComponent(src)}&${cdn.transformParams(width, quality)}`;
    }

    const params = cdn.transformParams(width, quality);
    return `${cdn.baseUrl}/${params}/${src}`;
  }

  // Preload images for better performance
  async preloadImage(src: string, width: number, quality?: number): Promise<void> {
    const url = await this.getImageUrl(src, width, quality);
    
    return new Promise((resolve, reject) => {
      const img = new window.Image();
      img.onload = () => resolve();
      img.onerror = reject;
      img.src = url;
    });
  }
}

export const multiCDNLoader = new MultiCDNImageLoader();

// Custom hook for multi-CDN images
export function useMultiCDNImage() {
  const [loaderReady, setLoaderReady] = useState(false);

  useEffect(() => {
    multiCDNLoader.initialize().then(() => setLoaderReady(true));
  }, []);

  const getImageUrl = async (src: string, width: number, quality?: number) => {
    if (!loaderReady) {
      await multiCDNLoader.initialize();
    }
    return multiCDNLoader.getImageUrl(src, width, quality);
  };

  const preloadImage = (src: string, width: number, quality?: number) => {
    return multiCDNLoader.preloadImage(src, width, quality);
  };

  return {
    loaderReady,
    getImageUrl,
    preloadImage,
  };
}
```

## Performance Monitoring {#performance-monitoring}

### Image Performance Analytics

```typescript
// lib/imagePerformanceAnalytics.ts
interface ImageLoadMetrics {
  src: string;
  loadTime: number;
  fileSize?: number;
  format: string;
  dimensions: { width: number; height: number };
  viewport: { width: number; height: number };
  timestamp: number;
  errors?: string[];
}

class ImagePerformanceAnalytics {
  private metrics: ImageLoadMetrics[] = [];
  private observers: Map<string, PerformanceObserver> = new Map();

  startTracking(src: string) {
    const startTime = performance.now();
    
    // Create performance observer for this image
    const observer = new PerformanceObserver((list) => {
      const entries = list.getEntries();
      entries.forEach((entry) => {
        if (entry.name.includes(src)) {
          this.recordMetric({
            src,
            loadTime: entry.duration,
            format: this.getImageFormat(src),
            dimensions: { width: 0, height: 0 }, // Will be updated
            viewport: {
              width: window.innerWidth,
              height: window.innerHeight,
            },
            timestamp: Date.now(),
          });
        }
      });
    });

    observer.observe({ entryTypes: ['resource'] });
    this.observers.set(src, observer);

    return startTime;
  }

  recordMetric(metric: ImageLoadMetrics) {
    this.metrics.push(metric);
    
    // Send to analytics service
    this.sendToAnalytics(metric);
    
    // Clean up old metrics (keep last 100)
    if (this.metrics.length > 100) {
      this.metrics = this.metrics.slice(-100);
    }
  }

  stopTracking(src: string) {
    const observer = this.observers.get(src);
    if (observer) {
      observer.disconnect();
      this.observers.delete(src);
    }
  }

  getAverageLoadTime(): number {
    if (this.metrics.length === 0) return 0;
    
    const total = this.metrics.reduce((sum, metric) => sum + metric.loadTime, 0);
    return total / this.metrics.length;
  }

  getSlowImages(threshold: number = 1000): ImageLoadMetrics[] {
    return this.metrics.filter(metric => metric.loadTime > threshold);
  }

  getFormatBreakdown(): Record<string, number> {
    const breakdown: Record<string, number> = {};
    
    this.metrics.forEach(metric => {
      breakdown[metric.format] = (breakdown[metric.format] || 0) + 1;
    });

    return breakdown;
  }

  private getImageFormat(src: string): string {
    const extension = src.split('.').pop()?.toLowerCase();
    return extension || 'unknown';
  }

  private sendToAnalytics(metric: ImageLoadMetrics) {
    // Send to your analytics service
    if (typeof window !== 'undefined' && window.gtag) {
      window.gtag('event', 'image_load', {
        custom_parameter_1: metric.src,
        custom_parameter_2: metric.loadTime,
        custom_parameter_3: metric.format,
      });
    }
  }

  generateReport(): {
    averageLoadTime: number;
    slowImages: ImageLoadMetrics[];
    formatBreakdown: Record<string, number>;
    totalImages: number;
  } {
    return {
      averageLoadTime: this.getAverageLoadTime(),
      slowImages: this.getSlowImages(),
      formatBreakdown: this.getFormatBreakdown(),
      totalImages: this.metrics.length,
    };
  }
}

export const imageAnalytics = new ImagePerformanceAnalytics();

// Hook for tracking image performance
export function useImagePerformance(src: string) {
  const [loadTime, setLoadTime] = useState<number | null>(null);
  const [startTime, setStartTime] = useState<number | null>(null);

  useEffect(() => {
    const start = imageAnalytics.startTracking(src);
    setStartTime(start);

    return () => {
      imageAnalytics.stopTracking(src);
    };
  }, [src]);

  const recordLoad = useCallback(() => {
    if (startTime !== null) {
      const endTime = performance.now();
      const duration = endTime - startTime;
      setLoadTime(duration);
      
      imageAnalytics.recordMetric({
        src,
        loadTime: duration,
        format: src.split('.').pop()?.toLowerCase() || 'unknown',
        dimensions: { width: 0, height: 0 },
        viewport: {
          width: window.innerWidth,
          height: window.innerHeight,
        },
        timestamp: Date.now(),
      });
    }
  }, [src, startTime]);

  return {
    loadTime,
    recordLoad,
  };
}
```

### Core Web Vitals for Images

```typescript
// components/ImageWithVitals.tsx
import Image from 'next/image';
import { useEffect, useRef, useState } from 'react';

interface ImageWithVitalsProps {
  src: string;
  alt: string;
  width: number;
  height: number;
  priority?: boolean;
  onVitalsUpdate?: (vitals: ImageVitals) => void;
}

interface ImageVitals {
  lcp?: number; // Largest Contentful Paint
  cls?: number; // Cumulative Layout Shift
  loadTime: number;
}

export default function ImageWithVitals({
  src,
  alt,
  width,
  height,
  priority = false,
  onVitalsUpdate,
}: ImageWithVitalsProps) {
  const [vitals, setVitals] = useState<ImageVitals>({ loadTime: 0 });
  const imageRef = useRef<HTMLImageElement>(null);
  const startTime = useRef<number>(0);

  useEffect(() => {
    startTime.current = performance.now();

    // Measure LCP if this is likely the largest contentful paint
    if (priority) {
      const observer = new PerformanceObserver((list) => {
        const entries = list.getEntries();
        const lcpEntry = entries[entries.length - 1] as any;
        
        if (lcpEntry && lcpEntry.element === imageRef.current) {
          setVitals(prev => ({
            ...prev,
            lcp: lcpEntry.startTime,
          }));
        }
      });

      observer.observe({ entryTypes: ['largest-contentful-paint'] });

      return () => observer.disconnect();
    }
  }, [priority]);

  useEffect(() => {
    // Measure CLS
    let clsValue = 0;
    const observer = new PerformanceObserver((list) => {
      for (const entry of list.getEntries()) {
        if (!(entry as any).hadRecentInput) {
          clsValue += (entry as any).value;
        }
      }
      
      setVitals(prev => ({
        ...prev,
        cls: clsValue,
      }));
    });

    observer.observe({ entryTypes: ['layout-shift'] });

    return () => observer.disconnect();
  }, []);

  const handleLoad = () => {
    const loadTime = performance.now() - startTime.current;
    
    const updatedVitals = {
      ...vitals,
      loadTime,
    };
    
    setVitals(updatedVitals);
    onVitalsUpdate?.(updatedVitals);
  };

  return (
    <Image
      ref={imageRef}
      src={src}
      alt={alt}
      width={width}
      height={height}
      priority={priority}
      onLoad={handleLoad}
      style={{
        width: '100%',
        height: 'auto',
      }}
    />
  );
}
```

## Best Practices {#best-practices}

### Image Optimization Checklist

```typescript
// utils/imageOptimizationChecklist.ts
interface ImageOptimizationCheck {
  name: string;
  description: string;
  check: (element: HTMLImageElement) => boolean;
  recommendation: string;
}

export const imageOptimizationChecks: ImageOptimizationCheck[] = [
  {
    name: 'Proper Alt Text',
    description: 'Images should have descriptive alt text',
    check: (img) => img.alt.length > 0 && img.alt !== img.src,
    recommendation: 'Add descriptive alt text that explains the image content and context',
  },
  {
    name: 'Lazy Loading',
    description: 'Images below the fold should use lazy loading',
    check: (img) => img.loading === 'lazy' || img.hasAttribute('data-lazy'),
    recommendation: 'Add loading="lazy" to images that are not immediately visible',
  },
  {
    name: 'Responsive Sizing',
    description: 'Images should have responsive sizing',
    check: (img) => img.hasAttribute('sizes') || img.style.width === '100%',
    recommendation: 'Use responsive sizing with the sizes attribute or CSS',
  },
  {
    name: 'Modern Format',
    description: 'Images should use modern formats like WebP or AVIF',
    check: (img) => {
      const src = img.src || img.currentSrc;
      return src.includes('.webp') || src.includes('.avif');
    },
    recommendation: 'Use WebP or AVIF formats for better compression',
  },
  {
    name: 'Proper Dimensions',
    description: 'Images should have width and height attributes',
    check: (img) => img.hasAttribute('width') && img.hasAttribute('height'),
    recommendation: 'Add width and height attributes to prevent layout shift',
  },
];

export function auditImages(): Array<{
  element: HTMLImageElement;
  results: Array<{ check: ImageOptimizationCheck; passed: boolean }>;
}> {
  const images = Array.from(document.querySelectorAll('img'));
  
  return images.map(img => ({
    element: img,
    results: imageOptimizationChecks.map(check => ({
      check,
      passed: check.check(img),
    })),
  }));
}

// Performance budget for images
export const imagePerformanceBudget = {
  maxImageSize: 500 * 1024, // 500KB
  maxLoadTime: 2000, // 2 seconds
  maxLCP: 2500, // 2.5 seconds
  maxCLS: 0.1, // 0.1 layout shift score
  recommendedFormats: ['webp', 'avif', 'jpg', 'png'],
  minQuality: 75,
  maxQuality: 90,
};
```

### Image SEO Optimization

```typescript
// components/SEOOptimizedImage.tsx
import Image from 'next/image';
import { useEffect, useState } from 'react';

interface SEOImageProps {
  src: string;
  alt: string;
  width: number;
  height: number;
  caption?: string;
  title?: string;
  loading?: 'lazy' | 'eager';
  priority?: boolean;
  schema?: {
    type: 'Product' | 'Person' | 'Place' | 'Thing';
    name: string;
    description?: string;
    url?: string;
  };
}

export default function SEOOptimizedImage({
  src,
  alt,
  width,
  height,
  caption,
  title,
  loading = 'lazy',
  priority = false,
  schema,
}: SEOImageProps) {
  const [imageData, setImageData] = useState<{
    naturalWidth: number;
    naturalHeight: number;
  } | null>(null);

  useEffect(() => {
    // Generate structured data for the image
    if (schema && typeof window !== 'undefined') {
      const structuredData = {
        '@context': 'https://schema.org',
        '@type': 'ImageObject',
        url: src,
        width: width,
        height: height,
        name: schema.name,
        description: schema.description || alt,
        ...(schema.url && { contentUrl: schema.url }),
      };

      // Add to page head
      const script = document.createElement('script');
      script.type = 'application/ld+json';
      script.text = JSON.stringify(structuredData);
      document.head.appendChild(script);

      return () => {
        document.head.removeChild(script);
      };
    }
  }, [src, alt, width, height, schema]);

  const handleLoad = (event: React.SyntheticEvent<HTMLImageElement>) => {
    const img = event.currentTarget;
    setImageData({
      naturalWidth: img.naturalWidth,
      naturalHeight: img.naturalHeight,
    });
  };

  return (
    <figure className="image-figure">
      <Image
        src={src}
        alt={alt}
        width={width}
        height={height}
        title={title}
        loading={loading}
        priority={priority}
        onLoad={handleLoad}
        style={{
          width: '100%',
          height: 'auto',
        }}
      />
      
      {caption && (
        <figcaption className="text-sm text-gray-600 mt-2 text-center">
          {caption}
        </figcaption>
      )}
      
      {/* Hidden metadata for search engines */}
      <div className="sr-only">
        {imageData && (
          <span>
            Image dimensions: {imageData.naturalWidth} x {imageData.naturalHeight} pixels
          </span>
        )}
        {schema && (
          <span>
            {schema.type}: {schema.name}
            {schema.description && ` - ${schema.description}`}
          </span>
        )}
      </div>
    </figure>
  );
}
```

## Exercises

### Exercise 1: Advanced Image Gallery
Create a comprehensive image gallery with:
- Infinite scroll with lazy loading
- Multiple view modes (grid, list, masonry)
- Advanced filtering and search
- Lightbox with keyboard navigation
- Performance monitoring and optimization

### Exercise 2: E-commerce Product Images
Build a product image system with:
- Multi-angle product views
- Zoom functionality
- Color variant switching
- Progressive loading with placeholders
- CDN integration with multiple fallbacks

### Exercise 3: Blog Image System
Create a blog image management system with:
- Automatic responsive image generation
- Art direction for different screen sizes
- SEO optimization with structured data
- Performance budgets and monitoring
- Accessibility features

### Exercise 4: Real-time Image Processing
Build an image processing interface with:
- Real-time filters and effects
- Multiple format downloads
- Batch processing capabilities
- Performance analytics
- Error handling and fallbacks

## Summary

Image optimization in Next.js involves:

- **Built-in optimization**: Automatic format conversion, sizing, and quality adjustment
- **Loading strategies**: Lazy loading, progressive loading, and priority loading
- **Responsive images**: Art direction and dynamic sizing
- **Performance monitoring**: Core Web Vitals and custom analytics
- **SEO optimization**: Structured data and accessibility features

Key principles:
- Use the Next.js Image component for automatic optimization
- Implement proper loading strategies based on image importance
- Monitor performance and set budgets
- Ensure accessibility and SEO compliance
- Use modern formats and CDNs for optimal delivery

Next, we'll explore **Performance Optimization** techniques to make your Next.js applications lightning fast!
