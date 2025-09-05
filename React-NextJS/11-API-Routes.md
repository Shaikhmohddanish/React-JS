# API Routes in Next.js

## Table of Contents
1. [Introduction to API Routes](#introduction)
2. [Basic API Route Structure](#basic-structure)
3. [HTTP Methods and RESTful APIs](#http-methods)
4. [Dynamic API Routes](#dynamic-routes)
5. [Middleware and Authentication](#middleware)
6. [Database Integration](#database)
7. [File Upload and Processing](#file-upload)
8. [External API Integration](#external-apis)
9. [Error Handling and Validation](#error-handling)
10. [Performance and Caching](#performance)
11. [Testing API Routes](#testing)
12. [Best Practices](#best-practices)

## Introduction to API Routes {#introduction}

API Routes in Next.js allow you to build a complete backend API alongside your frontend React application. They run on the server-side and can handle database operations, authentication, external API calls, and more.

### Key Features of Next.js API Routes

```typescript
interface APIRouteFeatures {
  serverless: boolean;
  hotReloading: boolean;
  typeScript: boolean;
  middleware: boolean;
  fileBasedRouting: boolean;
}

const nextjsAPIFeatures: APIRouteFeatures = {
  serverless: true,      // Deployed as serverless functions
  hotReloading: true,    // Instant updates during development
  typeScript: true,      // Full TypeScript support
  middleware: true,      // Custom middleware support
  fileBasedRouting: true // Routes based on file structure
};
```

### When to Use API Routes

```typescript
const apiRouteUseCases = [
  'User authentication and authorization',
  'Database CRUD operations', 
  'File upload and processing',
  'Third-party API integration',
  'Webhook endpoints',
  'Data validation and sanitization',
  'Email sending and notifications',
  'Payment processing',
  'Real-time features with WebSockets',
  'Microservice endpoints'
];

const notIdealFor = [
  'Heavy computational tasks (use dedicated services)',
  'Long-running processes (use background jobs)',
  'Static file serving (use CDN)',
  'Complex business logic (consider separate backend)'
];
```

## Basic API Route Structure {#basic-structure}

### Simple API Route Example

```typescript
// pages/api/hello.ts
import { NextApiRequest, NextApiResponse } from 'next';

interface HelloResponse {
  message: string;
  timestamp: string;
  userAgent?: string;
}

export default function handler(
  req: NextApiRequest,
  res: NextApiResponse<HelloResponse>
) {
  // Set CORS headers
  res.setHeader('Access-Control-Allow-Origin', '*');
  res.setHeader('Access-Control-Allow-Methods', 'GET, POST, OPTIONS');
  res.setHeader('Access-Control-Allow-Headers', 'Content-Type');

  // Handle preflight requests
  if (req.method === 'OPTIONS') {
    res.status(200).end();
    return;
  }

  // Only allow GET requests
  if (req.method !== 'GET') {
    res.status(405).json({
      message: 'Method not allowed',
      timestamp: new Date().toISOString(),
    });
    return;
  }

  // Return success response
  res.status(200).json({
    message: 'Hello from Next.js API!',
    timestamp: new Date().toISOString(),
    userAgent: req.headers['user-agent'],
  });
}
```

### API Route with Request Validation

```typescript
// pages/api/contact.ts
import { NextApiRequest, NextApiResponse } from 'next';

interface ContactRequest {
  name: string;
  email: string;
  subject: string;
  message: string;
}

interface ContactResponse {
  success: boolean;
  message: string;
  id?: string;
  errors?: Record<string, string>;
}

// Validation schema
function validateContactForm(data: any): { isValid: boolean; errors: Record<string, string> } {
  const errors: Record<string, string> = {};

  if (!data.name || data.name.trim().length < 2) {
    errors.name = 'Name must be at least 2 characters long';
  }

  if (!data.email || !/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(data.email)) {
    errors.email = 'Valid email address is required';
  }

  if (!data.subject || data.subject.trim().length < 5) {
    errors.subject = 'Subject must be at least 5 characters long';
  }

  if (!data.message || data.message.trim().length < 10) {
    errors.message = 'Message must be at least 10 characters long';
  }

  return {
    isValid: Object.keys(errors).length === 0,
    errors
  };
}

// Simulate saving to database
async function saveContactForm(data: ContactRequest): Promise<string> {
  // In a real app, you'd save to a database
  // For demo, we'll just return a mock ID
  await new Promise(resolve => setTimeout(resolve, 500)); // Simulate DB delay
  return `contact_${Date.now()}`;
}

// Simulate sending email notification
async function sendNotificationEmail(data: ContactRequest): Promise<void> {
  // In a real app, you'd use a service like SendGrid, Nodemailer, etc.
  console.log('Email notification sent for contact form:', data.subject);
}

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse<ContactResponse>
) {
  // Only allow POST requests
  if (req.method !== 'POST') {
    return res.status(405).json({
      success: false,
      message: 'Method not allowed. Use POST.',
    });
  }

  try {
    // Validate request data
    const { isValid, errors } = validateContactForm(req.body);
    
    if (!isValid) {
      return res.status(400).json({
        success: false,
        message: 'Validation failed',
        errors,
      });
    }

    const contactData: ContactRequest = {
      name: req.body.name.trim(),
      email: req.body.email.trim().toLowerCase(),
      subject: req.body.subject.trim(),
      message: req.body.message.trim(),
    };

    // Save to database and send notification in parallel
    const [contactId] = await Promise.all([
      saveContactForm(contactData),
      sendNotificationEmail(contactData)
    ]);

    return res.status(201).json({
      success: true,
      message: 'Contact form submitted successfully',
      id: contactId,
    });

  } catch (error) {
    console.error('Contact form error:', error);
    
    return res.status(500).json({
      success: false,
      message: 'Internal server error. Please try again later.',
    });
  }
}
```

## HTTP Methods and RESTful APIs {#http-methods}

### Complete CRUD API for Blog Posts

```typescript
// pages/api/posts/index.ts
import { NextApiRequest, NextApiResponse } from 'next';

interface BlogPost {
  id: string;
  title: string;
  content: string;
  excerpt: string;
  author: {
    id: string;
    name: string;
    email: string;
  };
  tags: string[];
  publishedAt: string | null;
  createdAt: string;
  updatedAt: string;
  status: 'draft' | 'published' | 'archived';
}

interface PostsResponse {
  posts: BlogPost[];
  total: number;
  page: number;
  limit: number;
  hasMore: boolean;
}

interface CreatePostRequest {
  title: string;
  content: string;
  excerpt?: string;
  tags?: string[];
  status?: 'draft' | 'published';
}

// Mock database operations
class PostsDB {
  private static posts: BlogPost[] = [];
  private static idCounter = 1;

  static async getAll(options: {
    page?: number;
    limit?: number;
    status?: string;
    author?: string;
  } = {}): Promise<PostsResponse> {
    const { page = 1, limit = 10, status, author } = options;
    
    let filteredPosts = [...this.posts];
    
    // Filter by status
    if (status) {
      filteredPosts = filteredPosts.filter(post => post.status === status);
    }
    
    // Filter by author
    if (author) {
      filteredPosts = filteredPosts.filter(post => 
        post.author.id === author || post.author.name.toLowerCase().includes(author.toLowerCase())
      );
    }
    
    // Sort by creation date (newest first)
    filteredPosts.sort((a, b) => new Date(b.createdAt).getTime() - new Date(a.createdAt).getTime());
    
    // Pagination
    const startIndex = (page - 1) * limit;
    const endIndex = startIndex + limit;
    const paginatedPosts = filteredPosts.slice(startIndex, endIndex);
    
    return {
      posts: paginatedPosts,
      total: filteredPosts.length,
      page,
      limit,
      hasMore: endIndex < filteredPosts.length,
    };
  }

  static async create(data: CreatePostRequest, authorId: string): Promise<BlogPost> {
    const now = new Date().toISOString();
    const id = `post_${this.idCounter++}`;
    
    const post: BlogPost = {
      id,
      title: data.title,
      content: data.content,
      excerpt: data.excerpt || data.content.substring(0, 200) + '...',
      author: {
        id: authorId,
        name: 'John Doe', // In real app, fetch from user database
        email: 'john@example.com',
      },
      tags: data.tags || [],
      publishedAt: data.status === 'published' ? now : null,
      createdAt: now,
      updatedAt: now,
      status: data.status || 'draft',
    };
    
    this.posts.push(post);
    return post;
  }

  static async findById(id: string): Promise<BlogPost | null> {
    return this.posts.find(post => post.id === id) || null;
  }

  static async update(id: string, data: Partial<CreatePostRequest>): Promise<BlogPost | null> {
    const index = this.posts.findIndex(post => post.id === id);
    if (index === -1) return null;
    
    const post = this.posts[index];
    const updatedPost: BlogPost = {
      ...post,
      ...data,
      updatedAt: new Date().toISOString(),
      publishedAt: data.status === 'published' && !post.publishedAt 
        ? new Date().toISOString() 
        : post.publishedAt,
    };
    
    this.posts[index] = updatedPost;
    return updatedPost;
  }

  static async delete(id: string): Promise<boolean> {
    const index = this.posts.findIndex(post => post.id === id);
    if (index === -1) return false;
    
    this.posts.splice(index, 1);
    return true;
  }
}

export default async function handler(req: NextApiRequest, res: NextApiResponse) {
  const { method } = req;

  try {
    switch (method) {
      case 'GET':
        return await handleGetPosts(req, res);
      case 'POST':
        return await handleCreatePost(req, res);
      default:
        res.setHeader('Allow', ['GET', 'POST']);
        return res.status(405).json({
          error: 'Method not allowed',
          allowedMethods: ['GET', 'POST'],
        });
    }
  } catch (error) {
    console.error('API Error:', error);
    return res.status(500).json({
      error: 'Internal server error',
      message: 'Something went wrong. Please try again later.',
    });
  }
}

async function handleGetPosts(req: NextApiRequest, res: NextApiResponse) {
  const {
    page = '1',
    limit = '10',
    status,
    author,
  } = req.query;

  // Validate query parameters
  const pageNum = parseInt(page as string);
  const limitNum = parseInt(limit as string);

  if (isNaN(pageNum) || pageNum < 1) {
    return res.status(400).json({
      error: 'Invalid page parameter',
      message: 'Page must be a positive integer',
    });
  }

  if (isNaN(limitNum) || limitNum < 1 || limitNum > 100) {
    return res.status(400).json({
      error: 'Invalid limit parameter',
      message: 'Limit must be between 1 and 100',
    });
  }

  const result = await PostsDB.getAll({
    page: pageNum,
    limit: limitNum,
    status: status as string,
    author: author as string,
  });

  return res.status(200).json(result);
}

async function handleCreatePost(req: NextApiRequest, res: NextApiResponse) {
  // Validate required fields
  const { title, content } = req.body;

  if (!title || title.trim().length < 3) {
    return res.status(400).json({
      error: 'Validation failed',
      message: 'Title is required and must be at least 3 characters long',
    });
  }

  if (!content || content.trim().length < 10) {
    return res.status(400).json({
      error: 'Validation failed',
      message: 'Content is required and must be at least 10 characters long',
    });
  }

  // In a real app, you'd get the user ID from authentication
  const authorId = req.headers.authorization || 'user_1';

  const postData: CreatePostRequest = {
    title: title.trim(),
    content: content.trim(),
    excerpt: req.body.excerpt?.trim(),
    tags: Array.isArray(req.body.tags) ? req.body.tags : [],
    status: req.body.status === 'published' ? 'published' : 'draft',
  };

  const newPost = await PostsDB.create(postData, authorId);

  return res.status(201).json({
    message: 'Post created successfully',
    post: newPost,
  });
}
```

### Individual Post API Route

```typescript
// pages/api/posts/[id].ts
import { NextApiRequest, NextApiResponse } from 'next';

export default async function handler(req: NextApiRequest, res: NextApiResponse) {
  const {
    query: { id },
    method,
  } = req;

  // Validate post ID
  if (!id || typeof id !== 'string') {
    return res.status(400).json({
      error: 'Invalid post ID',
      message: 'Post ID is required and must be a string',
    });
  }

  try {
    switch (method) {
      case 'GET':
        return await handleGetPost(id, req, res);
      case 'PUT':
        return await handleUpdatePost(id, req, res);
      case 'DELETE':
        return await handleDeletePost(id, req, res);
      default:
        res.setHeader('Allow', ['GET', 'PUT', 'DELETE']);
        return res.status(405).json({
          error: 'Method not allowed',
          allowedMethods: ['GET', 'PUT', 'DELETE'],
        });
    }
  } catch (error) {
    console.error('API Error:', error);
    return res.status(500).json({
      error: 'Internal server error',
      message: 'Something went wrong. Please try again later.',
    });
  }
}

async function handleGetPost(id: string, req: NextApiRequest, res: NextApiResponse) {
  const post = await PostsDB.findById(id);

  if (!post) {
    return res.status(404).json({
      error: 'Post not found',
      message: `No post found with ID: ${id}`,
    });
  }

  // Increment view count (in a real app)
  // await PostsDB.incrementViews(id);

  return res.status(200).json({ post });
}

async function handleUpdatePost(id: string, req: NextApiRequest, res: NextApiResponse) {
  // Check if post exists
  const existingPost = await PostsDB.findById(id);
  if (!existingPost) {
    return res.status(404).json({
      error: 'Post not found',
      message: `No post found with ID: ${id}`,
    });
  }

  // Validate update data
  const updateData: Partial<CreatePostRequest> = {};

  if (req.body.title !== undefined) {
    if (!req.body.title.trim() || req.body.title.trim().length < 3) {
      return res.status(400).json({
        error: 'Validation failed',
        message: 'Title must be at least 3 characters long',
      });
    }
    updateData.title = req.body.title.trim();
  }

  if (req.body.content !== undefined) {
    if (!req.body.content.trim() || req.body.content.trim().length < 10) {
      return res.status(400).json({
        error: 'Validation failed',
        message: 'Content must be at least 10 characters long',
      });
    }
    updateData.content = req.body.content.trim();
  }

  if (req.body.excerpt !== undefined) {
    updateData.excerpt = req.body.excerpt.trim();
  }

  if (req.body.tags !== undefined) {
    updateData.tags = Array.isArray(req.body.tags) ? req.body.tags : [];
  }

  if (req.body.status !== undefined) {
    if (!['draft', 'published', 'archived'].includes(req.body.status)) {
      return res.status(400).json({
        error: 'Validation failed',
        message: 'Status must be one of: draft, published, archived',
      });
    }
    updateData.status = req.body.status;
  }

  const updatedPost = await PostsDB.update(id, updateData);

  return res.status(200).json({
    message: 'Post updated successfully',
    post: updatedPost,
  });
}

async function handleDeletePost(id: string, req: NextApiRequest, res: NextApiResponse) {
  const deleted = await PostsDB.delete(id);

  if (!deleted) {
    return res.status(404).json({
      error: 'Post not found',
      message: `No post found with ID: ${id}`,
    });
  }

  return res.status(200).json({
    message: 'Post deleted successfully',
    deletedId: id,
  });
}
```

## Dynamic API Routes {#dynamic-routes}

### Nested Dynamic Routes

```typescript
// pages/api/users/[userId]/posts/[postId].ts
import { NextApiRequest, NextApiResponse } from 'next';

interface UserPostParams {
  userId: string;
  postId: string;
}

function parseParams(query: any): UserPostParams | null {
  const { userId, postId } = query;
  
  if (!userId || !postId || typeof userId !== 'string' || typeof postId !== 'string') {
    return null;
  }
  
  return { userId, postId };
}

export default async function handler(req: NextApiRequest, res: NextApiResponse) {
  const params = parseParams(req.query);
  
  if (!params) {
    return res.status(400).json({
      error: 'Invalid parameters',
      message: 'Both userId and postId are required',
    });
  }

  const { userId, postId } = params;

  try {
    switch (req.method) {
      case 'GET':
        return await getUserPost(userId, postId, res);
      case 'PUT':
        return await updateUserPost(userId, postId, req.body, res);
      case 'DELETE':
        return await deleteUserPost(userId, postId, res);
      default:
        res.setHeader('Allow', ['GET', 'PUT', 'DELETE']);
        return res.status(405).json({
          error: 'Method not allowed',
          allowedMethods: ['GET', 'PUT', 'DELETE'],
        });
    }
  } catch (error) {
    console.error('User post API error:', error);
    return res.status(500).json({
      error: 'Internal server error',
      message: 'Something went wrong',
    });
  }
}

async function getUserPost(userId: string, postId: string, res: NextApiResponse) {
  // Verify user exists and owns the post
  const post = await PostsDB.findById(postId);
  
  if (!post) {
    return res.status(404).json({
      error: 'Post not found',
      message: `Post ${postId} not found`,
    });
  }

  if (post.author.id !== userId) {
    return res.status(403).json({
      error: 'Access denied',
      message: 'You can only access your own posts',
    });
  }

  return res.status(200).json({ post });
}

async function updateUserPost(userId: string, postId: string, updateData: any, res: NextApiResponse) {
  // Verify ownership and update
  const post = await PostsDB.findById(postId);
  
  if (!post) {
    return res.status(404).json({
      error: 'Post not found',
      message: `Post ${postId} not found`,
    });
  }

  if (post.author.id !== userId) {
    return res.status(403).json({
      error: 'Access denied',
      message: 'You can only edit your own posts',
    });
  }

  const updatedPost = await PostsDB.update(postId, updateData);
  
  return res.status(200).json({
    message: 'Post updated successfully',
    post: updatedPost,
  });
}

async function deleteUserPost(userId: string, postId: string, res: NextApiResponse) {
  // Verify ownership and delete
  const post = await PostsDB.findById(postId);
  
  if (!post) {
    return res.status(404).json({
      error: 'Post not found',
      message: `Post ${postId} not found`,
    });
  }

  if (post.author.id !== userId) {
    return res.status(403).json({
      error: 'Access denied',
      message: 'You can only delete your own posts',
    });
  }

  await PostsDB.delete(postId);
  
  return res.status(200).json({
    message: 'Post deleted successfully',
    deletedId: postId,
  });
}
```

### Catch-All API Routes

```typescript
// pages/api/proxy/[...path].ts
import { NextApiRequest, NextApiResponse } from 'next';

interface ProxyConfig {
  allowedDomains: string[];
  rateLimit: number;
  timeout: number;
}

const config: ProxyConfig = {
  allowedDomains: [
    'jsonplaceholder.typicode.com',
    'api.github.com',
    'httpbin.org',
  ],
  rateLimit: 100, // requests per minute
  timeout: 10000, // 10 seconds
};

// Simple in-memory rate limiting
const rateLimitMap = new Map<string, { count: number; resetTime: number }>();

function checkRateLimit(ip: string): boolean {
  const now = Date.now();
  const windowMs = 60 * 1000; // 1 minute
  
  const current = rateLimitMap.get(ip);
  
  if (!current || now > current.resetTime) {
    rateLimitMap.set(ip, { count: 1, resetTime: now + windowMs });
    return true;
  }
  
  if (current.count >= config.rateLimit) {
    return false;
  }
  
  current.count++;
  return true;
}

function getClientIP(req: NextApiRequest): string {
  const forwarded = req.headers['x-forwarded-for'] as string;
  const ip = forwarded ? forwarded.split(',')[0] : req.connection.remoteAddress;
  return ip || 'unknown';
}

export default async function handler(req: NextApiRequest, res: NextApiResponse) {
  const { path } = req.query;
  
  // Validate path
  if (!path || !Array.isArray(path)) {
    return res.status(400).json({
      error: 'Invalid path',
      message: 'Path is required',
    });
  }

  // Rate limiting
  const clientIP = getClientIP(req);
  if (!checkRateLimit(clientIP)) {
    return res.status(429).json({
      error: 'Rate limit exceeded',
      message: `Maximum ${config.rateLimit} requests per minute`,
    });
  }

  // Reconstruct the full path
  const fullPath = path.join('/');
  
  // Extract domain from path (first segment)
  const [domain, ...restPath] = path;
  
  if (!config.allowedDomains.includes(domain)) {
    return res.status(403).json({
      error: 'Domain not allowed',
      message: `Domain ${domain} is not in the allowlist`,
      allowedDomains: config.allowedDomains,
    });
  }

  try {
    // Build the target URL
    const targetUrl = `https://${domain}/${restPath.join('/')}`;
    
    // Forward query parameters
    const urlObj = new URL(targetUrl);
    Object.entries(req.query).forEach(([key, value]) => {
      if (key !== 'path' && value) {
        urlObj.searchParams.append(key, value as string);
      }
    });

    console.log(`[Proxy] ${req.method} ${urlObj.toString()}`);

    // Make the request with timeout
    const controller = new AbortController();
    const timeoutId = setTimeout(() => controller.abort(), config.timeout);

    const response = await fetch(urlObj.toString(), {
      method: req.method,
      headers: {
        ...req.headers,
        // Remove host header to avoid conflicts
        host: undefined,
        // Add user agent
        'User-Agent': 'Next.js Proxy Server',
      },
      body: req.method !== 'GET' && req.method !== 'HEAD' ? JSON.stringify(req.body) : undefined,
      signal: controller.signal,
    });

    clearTimeout(timeoutId);

    // Forward response headers (selectively)
    const allowedHeaders = [
      'content-type',
      'content-length',
      'cache-control',
      'etag',
      'last-modified',
    ];

    allowedHeaders.forEach(header => {
      const value = response.headers.get(header);
      if (value) {
        res.setHeader(header, value);
      }
    });

    // Set CORS headers
    res.setHeader('Access-Control-Allow-Origin', '*');
    res.setHeader('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE, OPTIONS');
    res.setHeader('Access-Control-Allow-Headers', 'Content-Type, Authorization');

    // Forward the response
    const data = await response.text();
    
    return res.status(response.status).send(data);

  } catch (error) {
    console.error('[Proxy] Error:', error);
    
    if (error.name === 'AbortError') {
      return res.status(408).json({
        error: 'Request timeout',
        message: `Request timed out after ${config.timeout}ms`,
      });
    }

    return res.status(500).json({
      error: 'Proxy error',
      message: 'Failed to fetch from target server',
    });
  }
}
```

## Middleware and Authentication {#middleware}

### JWT Authentication Middleware

```typescript
// lib/auth.ts
import jwt from 'jsonwebtoken';
import { NextApiRequest, NextApiResponse } from 'next';

export interface User {
  id: string;
  email: string;
  name: string;
  role: 'user' | 'admin' | 'moderator';
}

export interface AuthenticatedRequest extends NextApiRequest {
  user: User;
}

const JWT_SECRET = process.env.JWT_SECRET || 'your-super-secret-key';

export function generateToken(user: User): string {
  return jwt.sign(
    { 
      userId: user.id, 
      email: user.email, 
      role: user.role 
    },
    JWT_SECRET,
    { expiresIn: '7d' }
  );
}

export function verifyToken(token: string): User | null {
  try {
    const decoded = jwt.verify(token, JWT_SECRET) as any;
    return {
      id: decoded.userId,
      email: decoded.email,
      name: decoded.name || '',
      role: decoded.role || 'user',
    };
  } catch (error) {
    return null;
  }
}

// Authentication middleware
export function withAuth(
  handler: (req: AuthenticatedRequest, res: NextApiResponse) => Promise<void> | void,
  options: { requiredRole?: User['role'] } = {}
) {
  return async (req: NextApiRequest, res: NextApiResponse) => {
    try {
      // Extract token from Authorization header or cookie
      const authHeader = req.headers.authorization;
      const token = authHeader?.startsWith('Bearer ') 
        ? authHeader.substring(7)
        : req.cookies.authToken;

      if (!token) {
        return res.status(401).json({
          error: 'Authentication required',
          message: 'No authentication token provided',
        });
      }

      // Verify token
      const user = verifyToken(token);
      if (!user) {
        return res.status(401).json({
          error: 'Invalid token',
          message: 'Authentication token is invalid or expired',
        });
      }

      // Check role if required
      if (options.requiredRole && user.role !== options.requiredRole) {
        return res.status(403).json({
          error: 'Insufficient permissions',
          message: `This endpoint requires ${options.requiredRole} role`,
        });
      }

      // Add user to request object
      (req as AuthenticatedRequest).user = user;

      // Call the original handler
      return handler(req as AuthenticatedRequest, res);

    } catch (error) {
      console.error('Authentication middleware error:', error);
      return res.status(500).json({
        error: 'Authentication error',
        message: 'Internal authentication error',
      });
    }
  };
}

// Rate limiting middleware
const rateLimitStore = new Map<string, { count: number; resetTime: number }>();

export function withRateLimit(
  handler: (req: NextApiRequest, res: NextApiResponse) => Promise<void> | void,
  options: { maxRequests?: number; windowMs?: number } = {}
) {
  const { maxRequests = 100, windowMs = 60 * 1000 } = options;

  return async (req: NextApiRequest, res: NextApiResponse) => {
    const ip = req.headers['x-forwarded-for'] || req.connection.remoteAddress || 'unknown';
    const key = Array.isArray(ip) ? ip[0] : ip;
    const now = Date.now();

    const current = rateLimitStore.get(key);

    if (!current || now > current.resetTime) {
      rateLimitStore.set(key, { count: 1, resetTime: now + windowMs });
    } else if (current.count >= maxRequests) {
      return res.status(429).json({
        error: 'Rate limit exceeded',
        message: `Too many requests. Maximum ${maxRequests} requests per ${windowMs / 1000} seconds`,
        retryAfter: Math.ceil((current.resetTime - now) / 1000),
      });
    } else {
      current.count++;
    }

    return handler(req, res);
  };
}

// CORS middleware
export function withCORS(
  handler: (req: NextApiRequest, res: NextApiResponse) => Promise<void> | void,
  options: {
    origin?: string | string[];
    methods?: string[];
    allowedHeaders?: string[];
  } = {}
) {
  const {
    origin = '*',
    methods = ['GET', 'POST', 'PUT', 'DELETE', 'OPTIONS'],
    allowedHeaders = ['Content-Type', 'Authorization'],
  } = options;

  return async (req: NextApiRequest, res: NextApiResponse) => {
    // Set CORS headers
    if (Array.isArray(origin)) {
      const requestOrigin = req.headers.origin;
      if (requestOrigin && origin.includes(requestOrigin)) {
        res.setHeader('Access-Control-Allow-Origin', requestOrigin);
      }
    } else {
      res.setHeader('Access-Control-Allow-Origin', origin);
    }

    res.setHeader('Access-Control-Allow-Methods', methods.join(', '));
    res.setHeader('Access-Control-Allow-Headers', allowedHeaders.join(', '));
    res.setHeader('Access-Control-Allow-Credentials', 'true');

    // Handle preflight requests
    if (req.method === 'OPTIONS') {
      return res.status(200).end();
    }

    return handler(req, res);
  };
}
```

### Protected API Route Example

```typescript
// pages/api/admin/users.ts
import { NextApiResponse } from 'next';
import { withAuth, withRateLimit, withCORS, AuthenticatedRequest } from '../../../lib/auth';

interface AdminUsersResponse {
  users: Array<{
    id: string;
    name: string;
    email: string;
    role: string;
    createdAt: string;
    lastLogin: string;
    status: 'active' | 'suspended' | 'pending';
  }>;
  total: number;
  page: number;
}

async function getUsersHandler(req: AuthenticatedRequest, res: NextApiResponse) {
  if (req.method !== 'GET') {
    return res.status(405).json({
      error: 'Method not allowed',
      allowedMethods: ['GET'],
    });
  }

  try {
    const { page = '1', limit = '20', search = '', status = '' } = req.query;
    
    // Mock user data (in real app, fetch from database)
    const mockUsers = [
      {
        id: '1',
        name: 'John Doe',
        email: 'john@example.com',
        role: 'user',
        createdAt: '2023-01-15T10:00:00Z',
        lastLogin: '2023-08-30T09:30:00Z',
        status: 'active' as const,
      },
      {
        id: '2',
        name: 'Jane Smith',
        email: 'jane@example.com',
        role: 'moderator',
        createdAt: '2023-02-20T14:30:00Z',
        lastLogin: '2023-08-29T16:45:00Z',
        status: 'active' as const,
      },
      // Add more mock users...
    ];

    // Filter users based on search and status
    let filteredUsers = mockUsers;
    
    if (search) {
      const searchLower = (search as string).toLowerCase();
      filteredUsers = filteredUsers.filter(user =>
        user.name.toLowerCase().includes(searchLower) ||
        user.email.toLowerCase().includes(searchLower)
      );
    }

    if (status) {
      filteredUsers = filteredUsers.filter(user => user.status === status);
    }

    // Pagination
    const pageNum = parseInt(page as string);
    const limitNum = parseInt(limit as string);
    const startIndex = (pageNum - 1) * limitNum;
    const paginatedUsers = filteredUsers.slice(startIndex, startIndex + limitNum);

    const response: AdminUsersResponse = {
      users: paginatedUsers,
      total: filteredUsers.length,
      page: pageNum,
    };

    return res.status(200).json(response);

  } catch (error) {
    console.error('Admin users API error:', error);
    return res.status(500).json({
      error: 'Internal server error',
      message: 'Failed to fetch users',
    });
  }
}

// Apply middleware chain
export default withCORS(
  withRateLimit(
    withAuth(getUsersHandler, { requiredRole: 'admin' }),
    { maxRequests: 50, windowMs: 60 * 1000 } // 50 requests per minute
  ),
  {
    origin: ['http://localhost:3000', 'https://yourdomain.com'],
    methods: ['GET'],
  }
);
```

## Database Integration {#database}

### MongoDB Integration with Mongoose

```typescript
// lib/mongodb.ts
import mongoose from 'mongoose';

const MONGODB_URI = process.env.MONGODB_URI!;

if (!MONGODB_URI) {
  throw new Error('Please define the MONGODB_URI environment variable');
}

interface MongooseCache {
  conn: typeof mongoose | null;
  promise: Promise<typeof mongoose> | null;
}

// Global is used here to maintain a cached connection across hot reloads in development
declare global {
  var mongooseCache: MongooseCache | undefined;
}

let cached: MongooseCache = global.mongooseCache || { conn: null, promise: null };

if (!global.mongooseCache) {
  global.mongooseCache = cached;
}

async function connectDB(): Promise<typeof mongoose> {
  if (cached.conn) {
    return cached.conn;
  }

  if (!cached.promise) {
    const opts = {
      bufferCommands: false,
    };

    cached.promise = mongoose.connect(MONGODB_URI, opts);
  }

  try {
    cached.conn = await cached.promise;
  } catch (e) {
    cached.promise = null;
    throw e;
  }

  return cached.conn;
}

export default connectDB;

// User model
import { Schema, model, models, Document } from 'mongoose';

export interface IUser extends Document {
  name: string;
  email: string;
  password: string;
  role: 'user' | 'admin' | 'moderator';
  avatar?: string;
  emailVerified: boolean;
  createdAt: Date;
  updatedAt: Date;
}

const UserSchema = new Schema<IUser>({
  name: {
    type: String,
    required: [true, 'Name is required'],
    trim: true,
    maxlength: [50, 'Name cannot exceed 50 characters'],
  },
  email: {
    type: String,
    required: [true, 'Email is required'],
    unique: true,
    lowercase: true,
    trim: true,
    match: [/^\S+@\S+\.\S+$/, 'Please enter a valid email'],
  },
  password: {
    type: String,
    required: [true, 'Password is required'],
    minlength: [6, 'Password must be at least 6 characters'],
    select: false, // Don't include password in queries by default
  },
  role: {
    type: String,
    enum: ['user', 'admin', 'moderator'],
    default: 'user',
  },
  avatar: {
    type: String,
    default: '',
  },
  emailVerified: {
    type: Boolean,
    default: false,
  },
}, {
  timestamps: true,
});

// Create indexes
UserSchema.index({ email: 1 });
UserSchema.index({ role: 1 });

export const User = models.User || model<IUser>('User', UserSchema);
```

### Database API Routes

```typescript
// pages/api/auth/register.ts
import bcrypt from 'bcryptjs';
import { NextApiRequest, NextApiResponse } from 'next';
import connectDB from '../../../lib/mongodb';
import { User } from '../../../lib/mongodb';
import { generateToken } from '../../../lib/auth';
import { withCORS, withRateLimit } from '../../../lib/auth';

interface RegisterRequest {
  name: string;
  email: string;
  password: string;
}

interface RegisterResponse {
  success: boolean;
  message: string;
  token?: string;
  user?: {
    id: string;
    name: string;
    email: string;
    role: string;
  };
  errors?: Record<string, string>;
}

async function registerHandler(req: NextApiRequest, res: NextApiResponse<RegisterResponse>) {
  if (req.method !== 'POST') {
    return res.status(405).json({
      success: false,
      message: 'Method not allowed',
    });
  }

  try {
    await connectDB();

    const { name, email, password }: RegisterRequest = req.body;

    // Validation
    const errors: Record<string, string> = {};

    if (!name || name.trim().length < 2) {
      errors.name = 'Name must be at least 2 characters long';
    }

    if (!email || !/^\S+@\S+\.\S+$/.test(email)) {
      errors.email = 'Valid email address is required';
    }

    if (!password || password.length < 6) {
      errors.password = 'Password must be at least 6 characters long';
    }

    if (Object.keys(errors).length > 0) {
      return res.status(400).json({
        success: false,
        message: 'Validation failed',
        errors,
      });
    }

    // Check if user already exists
    const existingUser = await User.findOne({ email: email.toLowerCase() });
    if (existingUser) {
      return res.status(409).json({
        success: false,
        message: 'User already exists with this email',
        errors: { email: 'Email already registered' },
      });
    }

    // Hash password
    const saltRounds = 12;
    const hashedPassword = await bcrypt.hash(password, saltRounds);

    // Create user
    const newUser = await User.create({
      name: name.trim(),
      email: email.toLowerCase().trim(),
      password: hashedPassword,
      role: 'user',
    });

    // Generate JWT token
    const token = generateToken({
      id: newUser._id.toString(),
      email: newUser.email,
      name: newUser.name,
      role: newUser.role,
    });

    // Return success response (don't include password)
    return res.status(201).json({
      success: true,
      message: 'User registered successfully',
      token,
      user: {
        id: newUser._id.toString(),
        name: newUser.name,
        email: newUser.email,
        role: newUser.role,
      },
    });

  } catch (error) {
    console.error('Registration error:', error);

    // Handle mongoose validation errors
    if (error.name === 'ValidationError') {
      const mongooseErrors: Record<string, string> = {};
      Object.keys(error.errors).forEach(key => {
        mongooseErrors[key] = error.errors[key].message;
      });

      return res.status(400).json({
        success: false,
        message: 'Validation failed',
        errors: mongooseErrors,
      });
    }

    return res.status(500).json({
      success: false,
      message: 'Internal server error during registration',
    });
  }
}

// Apply middleware
export default withCORS(
  withRateLimit(registerHandler, { maxRequests: 5, windowMs: 60 * 1000 }), // 5 registrations per minute
  {
    origin: process.env.NODE_ENV === 'production' 
      ? ['https://yourdomain.com'] 
      : ['http://localhost:3000'],
    methods: ['POST'],
  }
);
```

### Login API Route

```typescript
// pages/api/auth/login.ts
import bcrypt from 'bcryptjs';
import { NextApiRequest, NextApiResponse } from 'next';
import connectDB from '../../../lib/mongodb';
import { User } from '../../../lib/mongodb';
import { generateToken } from '../../../lib/auth';
import { withCORS, withRateLimit } from '../../../lib/auth';

interface LoginRequest {
  email: string;
  password: string;
  rememberMe?: boolean;
}

interface LoginResponse {
  success: boolean;
  message: string;
  token?: string;
  user?: {
    id: string;
    name: string;
    email: string;
    role: string;
    avatar?: string;
  };
}

async function loginHandler(req: NextApiRequest, res: NextApiResponse<LoginResponse>) {
  if (req.method !== 'POST') {
    return res.status(405).json({
      success: false,
      message: 'Method not allowed',
    });
  }

  try {
    await connectDB();

    const { email, password, rememberMe = false }: LoginRequest = req.body;

    // Validation
    if (!email || !password) {
      return res.status(400).json({
        success: false,
        message: 'Email and password are required',
      });
    }

    // Find user and include password for verification
    const user = await User.findOne({ email: email.toLowerCase() }).select('+password');
    
    if (!user) {
      return res.status(401).json({
        success: false,
        message: 'Invalid email or password',
      });
    }

    // Verify password
    const isPasswordValid = await bcrypt.compare(password, user.password);
    
    if (!isPasswordValid) {
      return res.status(401).json({
        success: false,
        message: 'Invalid email or password',
      });
    }

    // Check if email is verified (optional)
    if (!user.emailVerified) {
      return res.status(403).json({
        success: false,
        message: 'Please verify your email before logging in',
      });
    }

    // Generate JWT token
    const token = generateToken({
      id: user._id.toString(),
      email: user.email,
      name: user.name,
      role: user.role,
    });

    // Set HTTP-only cookie for security (optional)
    if (rememberMe) {
      res.setHeader('Set-Cookie', [
        `authToken=${token}; HttpOnly; Secure; SameSite=Strict; Max-Age=${7 * 24 * 60 * 60}; Path=/`, // 7 days
      ]);
    }

    // Update last login time (optional)
    await User.findByIdAndUpdate(user._id, { lastLogin: new Date() });

    return res.status(200).json({
      success: true,
      message: 'Login successful',
      token,
      user: {
        id: user._id.toString(),
        name: user.name,
        email: user.email,
        role: user.role,
        avatar: user.avatar,
      },
    });

  } catch (error) {
    console.error('Login error:', error);
    return res.status(500).json({
      success: false,
      message: 'Internal server error during login',
    });
  }
}

// Apply middleware with stricter rate limiting for login
export default withCORS(
  withRateLimit(loginHandler, { maxRequests: 10, windowMs: 60 * 1000 }), // 10 login attempts per minute
  {
    origin: process.env.NODE_ENV === 'production' 
      ? ['https://yourdomain.com'] 
      : ['http://localhost:3000'],
    methods: ['POST'],
  }
);
```

## Best Practices {#best-practices}

### 1. API Response Standardization

```typescript
// lib/apiResponse.ts
export interface APIResponse<T = any> {
  success: boolean;
  message: string;
  data?: T;
  errors?: Record<string, string> | string[];
  meta?: {
    page?: number;
    limit?: number;
    total?: number;
    hasMore?: boolean;
  };
  timestamp: string;
}

export class APIResponseBuilder<T = any> {
  private response: APIResponse<T>;

  constructor() {
    this.response = {
      success: false,
      message: '',
      timestamp: new Date().toISOString(),
    };
  }

  success(message: string, data?: T): this {
    this.response.success = true;
    this.response.message = message;
    if (data !== undefined) {
      this.response.data = data;
    }
    return this;
  }

  error(message: string, errors?: Record<string, string> | string[]): this {
    this.response.success = false;
    this.response.message = message;
    if (errors) {
      this.response.errors = errors;
    }
    return this;
  }

  withMeta(meta: APIResponse<T>['meta']): this {
    this.response.meta = meta;
    return this;
  }

  build(): APIResponse<T> {
    return this.response;
  }
}

// Usage helper functions
export function successResponse<T>(message: string, data?: T): APIResponse<T> {
  return new APIResponseBuilder<T>().success(message, data).build();
}

export function errorResponse(message: string, errors?: Record<string, string> | string[]): APIResponse {
  return new APIResponseBuilder().error(message, errors).build();
}

export function paginatedResponse<T>(
  message: string,
  data: T[],
  pagination: { page: number; limit: number; total: number }
): APIResponse<T[]> {
  return new APIResponseBuilder<T[]>()
    .success(message, data)
    .withMeta({
      ...pagination,
      hasMore: pagination.page * pagination.limit < pagination.total,
    })
    .build();
}
```

### 2. Request Validation

```typescript
// lib/validation.ts
import { z } from 'zod';
import { NextApiRequest, NextApiResponse } from 'next';

// Common validation schemas
export const emailSchema = z.string().email('Invalid email format');
export const passwordSchema = z.string().min(6, 'Password must be at least 6 characters');
export const nameSchema = z.string().min(2, 'Name must be at least 2 characters').max(50, 'Name cannot exceed 50 characters');

// Pagination schema
export const paginationSchema = z.object({
  page: z.string().optional().transform(val => val ? parseInt(val) : 1),
  limit: z.string().optional().transform(val => val ? Math.min(parseInt(val) || 10, 100) : 10),
});

// Generic validation middleware
export function withValidation<T>(
  schema: z.ZodSchema<T>,
  handler: (req: NextApiRequest & { validatedData: T }, res: NextApiResponse) => Promise<void> | void
) {
  return async (req: NextApiRequest, res: NextApiResponse) => {
    try {
      const validatedData = schema.parse(req.body);
      (req as any).validatedData = validatedData;
      return handler(req as NextApiRequest & { validatedData: T }, res);
    } catch (error) {
      if (error instanceof z.ZodError) {
        const errors: Record<string, string> = {};
        error.errors.forEach(err => {
          const path = err.path.join('.');
          errors[path] = err.message;
        });

        return res.status(400).json(errorResponse('Validation failed', errors));
      }
      throw error;
    }
  };
}

// Query validation middleware
export function withQueryValidation<T>(
  schema: z.ZodSchema<T>,
  handler: (req: NextApiRequest & { validatedQuery: T }, res: NextApiResponse) => Promise<void> | void
) {
  return async (req: NextApiRequest, res: NextApiResponse) => {
    try {
      const validatedQuery = schema.parse(req.query);
      (req as any).validatedQuery = validatedQuery;
      return handler(req as NextApiRequest & { validatedQuery: T }, res);
    } catch (error) {
      if (error instanceof z.ZodError) {
        const errors: Record<string, string> = {};
        error.errors.forEach(err => {
          const path = err.path.join('.');
          errors[path] = err.message;
        });

        return res.status(400).json(errorResponse('Invalid query parameters', errors));
      }
      throw error;
    }
  };
}
```

### 3. Error Handling

```typescript
// lib/errorHandler.ts
import { NextApiRequest, NextApiResponse } from 'next';
import { errorResponse } from './apiResponse';

export class APIError extends Error {
  public statusCode: number;
  public errors?: Record<string, string> | string[];

  constructor(message: string, statusCode: number = 500, errors?: Record<string, string> | string[]) {
    super(message);
    this.name = 'APIError';
    this.statusCode = statusCode;
    this.errors = errors;
  }
}

export function withErrorHandler(
  handler: (req: NextApiRequest, res: NextApiResponse) => Promise<void> | void
) {
  return async (req: NextApiRequest, res: NextApiResponse) => {
    try {
      await handler(req, res);
    } catch (error) {
      console.error('API Error:', error);

      if (error instanceof APIError) {
        return res.status(error.statusCode).json(
          errorResponse(error.message, error.errors)
        );
      }

      // Handle specific error types
      if (error.name === 'ValidationError') {
        return res.status(400).json(
          errorResponse('Validation failed', error.message)
        );
      }

      if (error.name === 'MongoError' && error.code === 11000) {
        return res.status(409).json(
          errorResponse('Duplicate entry', 'Resource already exists')
        );
      }

      // Generic server error
      return res.status(500).json(
        errorResponse('Internal server error', 'Something went wrong')
      );
    }
  };
}

// Convenience error creators
export const badRequest = (message: string, errors?: Record<string, string> | string[]) => 
  new APIError(message, 400, errors);

export const unauthorized = (message: string = 'Unauthorized') => 
  new APIError(message, 401);

export const forbidden = (message: string = 'Forbidden') => 
  new APIError(message, 403);

export const notFound = (message: string = 'Not found') => 
  new APIError(message, 404);

export const conflict = (message: string, errors?: Record<string, string> | string[]) => 
  new APIError(message, 409, errors);

export const tooManyRequests = (message: string = 'Too many requests') => 
  new APIError(message, 429);

export const internalError = (message: string = 'Internal server error') => 
  new APIError(message, 500);
```

## Exercises

### Exercise 1: Complete Blog API
Create a complete blog API with:
- CRUD operations for posts
- User authentication and authorization
- Comments system with nested replies
- Tags and categories management
- Search and filtering capabilities
- File upload for images

### Exercise 2: E-commerce API
Build an e-commerce API with:
- Product catalog management
- Shopping cart functionality
- Order processing and tracking
- Payment integration (Stripe/PayPal)
- Inventory management
- User reviews and ratings

### Exercise 3: Social Media API
Create a social media platform API with:
- User profiles and authentication
- Posts, likes, and comments
- Follow/unfollow functionality
- Real-time notifications
- Image and video uploads
- Content moderation

### Exercise 4: Task Management API
Build a project management API with:
- Projects and task organization
- Team collaboration features
- File attachments and comments
- Time tracking and reporting
- Role-based permissions
- Webhook notifications

## Summary

API Routes in Next.js provide:

- **Full-stack Capability**: Build complete applications with frontend and backend
- **Serverless Architecture**: Automatic scaling and deployment
- **Type Safety**: Full TypeScript support throughout
- **Developer Experience**: Hot reloading and integrated development

Key features covered:
- RESTful API design patterns
- Authentication and authorization
- Database integration with MongoDB
- Request validation and error handling
- Middleware for common functionality
- File uploads and external API integration
- Performance optimization and caching

Next, we'll explore **Data Fetching** patterns and learn advanced techniques for loading data in Next.js applications!
