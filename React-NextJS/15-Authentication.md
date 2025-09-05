# Authentication in Next.js

## Table of Contents
1. [Introduction to Authentication](#introduction)
2. [NextAuth.js Setup and Configuration](#nextauth-setup)
3. [Custom Authentication System](#custom-auth)
4. [JWT and Session Management](#jwt-sessions)
5. [OAuth Integration](#oauth-integration)
6. [Role-Based Access Control (RBAC)](#rbac)
7. [Multi-Factor Authentication](#mfa)
8. [Security Best Practices](#security-practices)
9. [Middleware for Authentication](#auth-middleware)
10. [Frontend Authentication Patterns](#frontend-patterns)
11. [Testing Authentication](#testing-auth)
12. [Best Practices](#best-practices)

## Introduction to Authentication {#introduction}

Authentication in Next.js can be implemented using various approaches, from third-party solutions like NextAuth.js to custom implementations. Understanding security principles, session management, and user experience is crucial for building secure applications.

### Authentication Strategies Overview

```typescript
// types/auth.ts
interface AuthStrategy {
  name: string;
  complexity: 'low' | 'medium' | 'high';
  security: 'basic' | 'standard' | 'enterprise';
  features: string[];
  useCases: string[];
}

export const authStrategies: AuthStrategy[] = [
  {
    name: 'NextAuth.js',
    complexity: 'low',
    security: 'standard',
    features: ['OAuth', 'JWT', 'Database sessions', 'Built-in providers'],
    useCases: ['Rapid development', 'Standard web apps', 'OAuth integration'],
  },
  {
    name: 'Custom JWT',
    complexity: 'medium',
    security: 'standard',
    features: ['Full control', 'Stateless', 'Custom claims', 'API-friendly'],
    useCases: ['API-first apps', 'Microservices', 'Custom requirements'],
  },
  {
    name: 'Session-based',
    complexity: 'medium',
    security: 'standard',
    features: ['Server sessions', 'CSRF protection', 'Session storage'],
    useCases: ['Traditional web apps', 'High security needs'],
  },
  {
    name: 'Enterprise SSO',
    complexity: 'high',
    security: 'enterprise',
    features: ['SAML', 'OIDC', 'AD integration', 'MFA'],
    useCases: ['Enterprise apps', 'Corporate environments'],
  },
];

interface User {
  id: string;
  email: string;
  name: string;
  role: string;
  permissions: string[];
  lastLogin?: Date;
  emailVerified: boolean;
  twoFactorEnabled: boolean;
  profile?: {
    avatar?: string;
    bio?: string;
    preferences: Record<string, any>;
  };
}

interface AuthSession {
  user: User;
  accessToken: string;
  refreshToken?: string;
  expiresAt: number;
  csrfToken?: string;
}
```

## NextAuth.js Setup and Configuration {#nextauth-setup}

### Installation and Basic Setup

```bash
npm install next-auth
npm install @next-auth/prisma-adapter prisma @prisma/client
npm install @types/next-auth
```

```typescript
// pages/api/auth/[...nextauth].ts
import NextAuth, { NextAuthOptions } from 'next-auth';
import GoogleProvider from 'next-auth/providers/google';
import GitHubProvider from 'next-auth/providers/github';
import CredentialsProvider from 'next-auth/providers/credentials';
import { PrismaAdapter } from '@next-auth/prisma-adapter';
import { prisma } from '../../../lib/prisma';
import bcrypt from 'bcryptjs';

export const authOptions: NextAuthOptions = {
  adapter: PrismaAdapter(prisma),
  
  providers: [
    // OAuth Providers
    GoogleProvider({
      clientId: process.env.GOOGLE_CLIENT_ID!,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET!,
    }),
    
    GitHubProvider({
      clientId: process.env.GITHUB_ID!,
      clientSecret: process.env.GITHUB_SECRET!,
    }),
    
    // Custom credentials provider
    CredentialsProvider({
      name: 'credentials',
      credentials: {
        email: { label: 'Email', type: 'email' },
        password: { label: 'Password', type: 'password' },
      },
      async authorize(credentials) {
        if (!credentials?.email || !credentials?.password) {
          throw new Error('Missing credentials');
        }

        const user = await prisma.user.findUnique({
          where: { email: credentials.email },
          include: {
            accounts: true,
            sessions: true,
          },
        });

        if (!user || !user.password) {
          throw new Error('User not found');
        }

        const isPasswordValid = await bcrypt.compare(
          credentials.password,
          user.password
        );

        if (!isPasswordValid) {
          throw new Error('Invalid password');
        }

        if (!user.emailVerified) {
          throw new Error('Please verify your email');
        }

        return {
          id: user.id,
          email: user.email,
          name: user.name,
          role: user.role,
          emailVerified: user.emailVerified,
          image: user.image,
        };
      },
    }),
  ],

  pages: {
    signIn: '/auth/signin',
    signUp: '/auth/signup',
    error: '/auth/error',
    verifyRequest: '/auth/verify-request',
    newUser: '/auth/new-user',
  },

  callbacks: {
    async jwt({ token, user, account }) {
      // Persist the OAuth access_token and refresh_token to the token right after signin
      if (account) {
        token.accessToken = account.access_token;
        token.refreshToken = account.refresh_token;
        token.accessTokenExpires = account.expires_at! * 1000;
      }

      // Add user info to token
      if (user) {
        token.role = user.role;
        token.id = user.id;
        token.emailVerified = user.emailVerified;
      }

      // Return previous token if the access token has not expired
      if (Date.now() < (token.accessTokenExpires as number)) {
        return token;
      }

      // Access token has expired, try to update it
      return await refreshAccessToken(token);
    },

    async session({ session, token }) {
      // Send properties to the client
      session.user.id = token.id as string;
      session.user.role = token.role as string;
      session.user.emailVerified = token.emailVerified as boolean;
      session.accessToken = token.accessToken as string;
      session.error = token.error as string;

      return session;
    },

    async signIn({ user, account, profile, email, credentials }) {
      // Allow OAuth signin
      if (account?.provider !== 'credentials') {
        return true;
      }

      // For credentials, additional checks
      if (user.emailVerified) {
        // Log signin attempt
        await prisma.loginAttempt.create({
          data: {
            userId: user.id,
            success: true,
            ip: '', // Get from request
            userAgent: '', // Get from request
          },
        });

        return true;
      }

      return false;
    },

    async redirect({ url, baseUrl }) {
      // Allows relative callback URLs
      if (url.startsWith('/')) return `${baseUrl}${url}`;
      
      // Allows callback URLs on the same origin
      if (new URL(url).origin === baseUrl) return url;
      
      return baseUrl;
    },
  },

  events: {
    async signIn({ user, account, profile, isNewUser }) {
      console.log(`User ${user.email} signed in`);
      
      // Update last login
      await prisma.user.update({
        where: { id: user.id },
        data: { lastLogin: new Date() },
      });
    },
    
    async signOut({ session, token }) {
      console.log(`User signed out`);
    },
    
    async createUser({ user }) {
      console.log(`New user created: ${user.email}`);
      
      // Send welcome email
      // await sendWelcomeEmail(user.email);
    },
  },

  session: {
    strategy: 'jwt',
    maxAge: 30 * 24 * 60 * 60, // 30 days
    updateAge: 24 * 60 * 60, // 24 hours
  },

  jwt: {
    maxAge: 60 * 60 * 24 * 30, // 30 days
  },

  debug: process.env.NODE_ENV === 'development',
};

async function refreshAccessToken(token: any) {
  try {
    const url = `https://oauth2.googleapis.com/token?` +
      new URLSearchParams({
        client_id: process.env.GOOGLE_CLIENT_ID!,
        client_secret: process.env.GOOGLE_CLIENT_SECRET!,
        grant_type: 'refresh_token',
        refresh_token: token.refreshToken,
      });

    const response = await fetch(url, {
      headers: {
        'Content-Type': 'application/x-www-form-urlencoded',
      },
      method: 'POST',
    });

    const refreshedTokens = await response.json();

    if (!response.ok) {
      throw refreshedTokens;
    }

    return {
      ...token,
      accessToken: refreshedTokens.access_token,
      accessTokenExpires: Date.now() + refreshedTokens.expires_in * 1000,
      refreshToken: refreshedTokens.refresh_token ?? token.refreshToken,
    };
  } catch (error) {
    console.log(error);

    return {
      ...token,
      error: 'RefreshAccessTokenError',
    };
  }
}

export default NextAuth(authOptions);
```

### Advanced NextAuth Configuration

```typescript
// lib/auth.ts
import { getServerSession } from 'next-auth/next';
import { authOptions } from '../pages/api/auth/[...nextauth]';
import { redirect } from 'next/navigation';

export async function getAuthSession() {
  return await getServerSession(authOptions);
}

export async function requireAuth() {
  const session = await getAuthSession();
  
  if (!session?.user) {
    redirect('/auth/signin');
  }
  
  return session;
}

export async function requireRole(allowedRoles: string[]) {
  const session = await requireAuth();
  
  if (!allowedRoles.includes(session.user.role)) {
    redirect('/unauthorized');
  }
  
  return session;
}

// Custom hooks for client-side
export function useAuthSession() {
  const { data: session, status } = useSession();
  
  return {
    user: session?.user,
    isLoading: status === 'loading',
    isAuthenticated: !!session?.user,
    role: session?.user?.role,
    hasRole: (role: string) => session?.user?.role === role,
    hasAnyRole: (roles: string[]) => roles.includes(session?.user?.role || ''),
  };
}

// Enhanced error handling
export class AuthError extends Error {
  constructor(
    message: string,
    public code: string,
    public statusCode: number = 401
  ) {
    super(message);
    this.name = 'AuthError';
  }
}

export const authErrors = {
  INVALID_CREDENTIALS: new AuthError('Invalid credentials', 'INVALID_CREDENTIALS', 401),
  EMAIL_NOT_VERIFIED: new AuthError('Email not verified', 'EMAIL_NOT_VERIFIED', 401),
  ACCOUNT_DISABLED: new AuthError('Account disabled', 'ACCOUNT_DISABLED', 403),
  INSUFFICIENT_PERMISSIONS: new AuthError('Insufficient permissions', 'INSUFFICIENT_PERMISSIONS', 403),
  TOKEN_EXPIRED: new AuthError('Token expired', 'TOKEN_EXPIRED', 401),
  RATE_LIMIT_EXCEEDED: new AuthError('Too many attempts', 'RATE_LIMIT_EXCEEDED', 429),
};
```

## Custom Authentication System {#custom-auth}

### JWT-Based Authentication

```typescript
// lib/customAuth.ts
import jwt from 'jsonwebtoken';
import bcrypt from 'bcryptjs';
import { prisma } from './prisma';
import { User } from '@prisma/client';

interface JWTPayload {
  userId: string;
  email: string;
  role: string;
  sessionId: string;
  iat?: number;
  exp?: number;
}

interface AuthTokens {
  accessToken: string;
  refreshToken: string;
  expiresAt: number;
}

export class CustomAuthService {
  private readonly JWT_SECRET = process.env.JWT_SECRET!;
  private readonly JWT_REFRESH_SECRET = process.env.JWT_REFRESH_SECRET!;
  private readonly ACCESS_TOKEN_EXPIRY = '15m';
  private readonly REFRESH_TOKEN_EXPIRY = '7d';

  async register(userData: {
    email: string;
    password: string;
    name: string;
  }): Promise<{ user: User; tokens: AuthTokens }> {
    const { email, password, name } = userData;

    // Check if user exists
    const existingUser = await prisma.user.findUnique({
      where: { email },
    });

    if (existingUser) {
      throw new AuthError('User already exists', 'USER_EXISTS', 409);
    }

    // Hash password
    const hashedPassword = await bcrypt.hash(password, 12);

    // Create user
    const user = await prisma.user.create({
      data: {
        email,
        password: hashedPassword,
        name,
        role: 'USER',
        emailVerified: false,
      },
    });

    // Generate verification token
    const verificationToken = this.generateVerificationToken(user.id);
    
    // Store verification token
    await prisma.verificationToken.create({
      data: {
        identifier: user.email,
        token: verificationToken,
        expires: new Date(Date.now() + 24 * 60 * 60 * 1000), // 24 hours
      },
    });

    // Send verification email
    await this.sendVerificationEmail(user.email, verificationToken);

    // Generate auth tokens
    const tokens = await this.generateTokens(user);

    return { user, tokens };
  }

  async login(credentials: {
    email: string;
    password: string;
    rememberMe?: boolean;
  }): Promise<{ user: User; tokens: AuthTokens }> {
    const { email, password, rememberMe } = credentials;

    // Rate limiting check
    await this.checkRateLimit(email);

    // Find user
    const user = await prisma.user.findUnique({
      where: { email },
    });

    if (!user || !user.password) {
      await this.logFailedAttempt(email);
      throw authErrors.INVALID_CREDENTIALS;
    }

    // Verify password
    const isPasswordValid = await bcrypt.compare(password, user.password);

    if (!isPasswordValid) {
      await this.logFailedAttempt(email, user.id);
      throw authErrors.INVALID_CREDENTIALS;
    }

    // Check if email is verified
    if (!user.emailVerified) {
      throw authErrors.EMAIL_NOT_VERIFIED;
    }

    // Check if account is active
    if (user.status === 'DISABLED') {
      throw authErrors.ACCOUNT_DISABLED;
    }

    // Update last login
    await prisma.user.update({
      where: { id: user.id },
      data: { lastLogin: new Date() },
    });

    // Generate tokens
    const tokens = await this.generateTokens(user, rememberMe);

    // Log successful login
    await this.logSuccessfulLogin(user.id);

    return { user, tokens };
  }

  async refreshToken(refreshToken: string): Promise<AuthTokens> {
    try {
      const payload = jwt.verify(refreshToken, this.JWT_REFRESH_SECRET) as JWTPayload;

      // Check if session exists and is valid
      const session = await prisma.session.findUnique({
        where: { id: payload.sessionId },
        include: { user: true },
      });

      if (!session || session.expires < new Date()) {
        throw new Error('Invalid session');
      }

      // Generate new tokens
      return await this.generateTokens(session.user);
    } catch (error) {
      throw authErrors.TOKEN_EXPIRED;
    }
  }

  async logout(sessionId: string): Promise<void> {
    await prisma.session.delete({
      where: { id: sessionId },
    });
  }

  async verifyEmail(token: string): Promise<boolean> {
    const verificationToken = await prisma.verificationToken.findFirst({
      where: {
        token,
        expires: { gt: new Date() },
      },
    });

    if (!verificationToken) {
      return false;
    }

    // Update user as verified
    await prisma.user.update({
      where: { email: verificationToken.identifier },
      data: { emailVerified: true },
    });

    // Delete verification token
    await prisma.verificationToken.delete({
      where: { token },
    });

    return true;
  }

  async requestPasswordReset(email: string): Promise<void> {
    const user = await prisma.user.findUnique({
      where: { email },
    });

    if (!user) {
      // Don't reveal if user exists
      return;
    }

    // Generate reset token
    const resetToken = this.generateResetToken();
    
    // Store reset token
    await prisma.passwordResetToken.create({
      data: {
        userId: user.id,
        token: resetToken,
        expires: new Date(Date.now() + 60 * 60 * 1000), // 1 hour
      },
    });

    // Send reset email
    await this.sendPasswordResetEmail(user.email, resetToken);
  }

  async resetPassword(token: string, newPassword: string): Promise<boolean> {
    const resetToken = await prisma.passwordResetToken.findFirst({
      where: {
        token,
        expires: { gt: new Date() },
        used: false,
      },
      include: { user: true },
    });

    if (!resetToken) {
      return false;
    }

    // Hash new password
    const hashedPassword = await bcrypt.hash(newPassword, 12);

    // Update password
    await prisma.user.update({
      where: { id: resetToken.userId },
      data: { password: hashedPassword },
    });

    // Mark token as used
    await prisma.passwordResetToken.update({
      where: { id: resetToken.id },
      data: { used: true },
    });

    // Invalidate all sessions for this user
    await prisma.session.deleteMany({
      where: { userId: resetToken.userId },
    });

    return true;
  }

  private async generateTokens(user: User, rememberMe = false): Promise<AuthTokens> {
    // Create session
    const session = await prisma.session.create({
      data: {
        userId: user.id,
        expires: new Date(Date.now() + (rememberMe ? 30 : 7) * 24 * 60 * 60 * 1000),
      },
    });

    const payload: JWTPayload = {
      userId: user.id,
      email: user.email,
      role: user.role,
      sessionId: session.id,
    };

    const accessToken = jwt.sign(payload, this.JWT_SECRET, {
      expiresIn: this.ACCESS_TOKEN_EXPIRY,
    });

    const refreshToken = jwt.sign(payload, this.JWT_REFRESH_SECRET, {
      expiresIn: this.REFRESH_TOKEN_EXPIRY,
    });

    return {
      accessToken,
      refreshToken,
      expiresAt: Date.now() + 15 * 60 * 1000, // 15 minutes
    };
  }

  async verifyToken(token: string): Promise<JWTPayload> {
    try {
      const payload = jwt.verify(token, this.JWT_SECRET) as JWTPayload;
      
      // Verify session still exists
      const session = await prisma.session.findUnique({
        where: { id: payload.sessionId },
        include: { user: true },
      });

      if (!session || session.expires < new Date()) {
        throw new Error('Session expired');
      }

      return payload;
    } catch (error) {
      throw authErrors.TOKEN_EXPIRED;
    }
  }

  private async checkRateLimit(email: string): Promise<void> {
    const attempts = await prisma.loginAttempt.count({
      where: {
        email,
        createdAt: {
          gt: new Date(Date.now() - 15 * 60 * 1000), // Last 15 minutes
        },
        success: false,
      },
    });

    if (attempts >= 5) {
      throw authErrors.RATE_LIMIT_EXCEEDED;
    }
  }

  private async logFailedAttempt(email: string, userId?: string): Promise<void> {
    await prisma.loginAttempt.create({
      data: {
        email,
        userId,
        success: false,
        ip: '', // Get from request context
        userAgent: '', // Get from request context
      },
    });
  }

  private async logSuccessfulLogin(userId: string): Promise<void> {
    await prisma.loginAttempt.create({
      data: {
        userId,
        success: true,
        ip: '', // Get from request context
        userAgent: '', // Get from request context
      },
    });
  }

  private generateVerificationToken(userId: string): string {
    return jwt.sign({ userId, type: 'verification' }, this.JWT_SECRET, {
      expiresIn: '24h',
    });
  }

  private generateResetToken(): string {
    return require('crypto').randomBytes(32).toString('hex');
  }

  private async sendVerificationEmail(email: string, token: string): Promise<void> {
    // Implement email sending logic
    console.log(`Verification email sent to ${email} with token ${token}`);
  }

  private async sendPasswordResetEmail(email: string, token: string): Promise<void> {
    // Implement email sending logic
    console.log(`Password reset email sent to ${email} with token ${token}`);
  }
}

export const customAuth = new CustomAuthService();
```

### Authentication API Routes

```typescript
// pages/api/auth/register.ts
import { NextApiRequest, NextApiResponse } from 'next';
import { customAuth } from '../../../lib/customAuth';
import { z } from 'zod';

const registerSchema = z.object({
  email: z.string().email(),
  password: z.string().min(8).max(100),
  name: z.string().min(1).max(100),
  acceptTerms: z.boolean().refine(val => val === true, {
    message: 'You must accept the terms and conditions',
  }),
});

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  try {
    const validatedData = registerSchema.parse(req.body);

    const { user, tokens } = await customAuth.register({
      email: validatedData.email,
      password: validatedData.password,
      name: validatedData.name,
    });

    // Set httpOnly cookie for refresh token
    res.setHeader('Set-Cookie', [
      `refreshToken=${tokens.refreshToken}; HttpOnly; Secure; SameSite=Strict; Path=/; Max-Age=${7 * 24 * 60 * 60}`,
    ]);

    // Return user data and access token
    res.status(201).json({
      user: {
        id: user.id,
        email: user.email,
        name: user.name,
        role: user.role,
        emailVerified: user.emailVerified,
      },
      accessToken: tokens.accessToken,
      expiresAt: tokens.expiresAt,
    });
  } catch (error) {
    console.error('Registration error:', error);

    if (error instanceof AuthError) {
      return res.status(error.statusCode).json({
        error: error.message,
        code: error.code,
      });
    }

    if (error instanceof z.ZodError) {
      return res.status(400).json({
        error: 'Validation error',
        details: error.errors,
      });
    }

    res.status(500).json({ error: 'Internal server error' });
  }
}

// pages/api/auth/login.ts
import { NextApiRequest, NextApiResponse } from 'next';
import { customAuth } from '../../../lib/customAuth';
import { z } from 'zod';

const loginSchema = z.object({
  email: z.string().email(),
  password: z.string().min(1),
  rememberMe: z.boolean().optional(),
});

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  try {
    const validatedData = loginSchema.parse(req.body);

    const { user, tokens } = await customAuth.login(validatedData);

    // Set httpOnly cookie for refresh token
    const maxAge = validatedData.rememberMe ? 30 * 24 * 60 * 60 : 7 * 24 * 60 * 60;
    
    res.setHeader('Set-Cookie', [
      `refreshToken=${tokens.refreshToken}; HttpOnly; Secure; SameSite=Strict; Path=/; Max-Age=${maxAge}`,
    ]);

    res.status(200).json({
      user: {
        id: user.id,
        email: user.email,
        name: user.name,
        role: user.role,
        emailVerified: user.emailVerified,
        lastLogin: user.lastLogin,
      },
      accessToken: tokens.accessToken,
      expiresAt: tokens.expiresAt,
    });
  } catch (error) {
    console.error('Login error:', error);

    if (error instanceof AuthError) {
      return res.status(error.statusCode).json({
        error: error.message,
        code: error.code,
      });
    }

    if (error instanceof z.ZodError) {
      return res.status(400).json({
        error: 'Validation error',
        details: error.errors,
      });
    }

    res.status(500).json({ error: 'Internal server error' });
  }
}

// pages/api/auth/refresh.ts
export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  try {
    const refreshToken = req.cookies.refreshToken;

    if (!refreshToken) {
      return res.status(401).json({ error: 'No refresh token provided' });
    }

    const tokens = await customAuth.refreshToken(refreshToken);

    // Update refresh token cookie
    res.setHeader('Set-Cookie', [
      `refreshToken=${tokens.refreshToken}; HttpOnly; Secure; SameSite=Strict; Path=/; Max-Age=${7 * 24 * 60 * 60}`,
    ]);

    res.status(200).json({
      accessToken: tokens.accessToken,
      expiresAt: tokens.expiresAt,
    });
  } catch (error) {
    console.error('Token refresh error:', error);

    // Clear invalid refresh token
    res.setHeader('Set-Cookie', [
      'refreshToken=; HttpOnly; Secure; SameSite=Strict; Path=/; Max-Age=0',
    ]);

    if (error instanceof AuthError) {
      return res.status(error.statusCode).json({
        error: error.message,
        code: error.code,
      });
    }

    res.status(500).json({ error: 'Internal server error' });
  }
}
```

## Role-Based Access Control (RBAC) {#rbac}

### RBAC Implementation

```typescript
// lib/rbac.ts
interface Permission {
  id: string;
  name: string;
  description: string;
  resource: string;
  action: string;
}

interface Role {
  id: string;
  name: string;
  description: string;
  permissions: Permission[];
  isDefault?: boolean;
  priority: number;
}

interface RBACUser extends User {
  roles: Role[];
  permissions: Permission[];
}

export class RBACService {
  private static instance: RBACService;
  private roles: Map<string, Role> = new Map();
  private permissions: Map<string, Permission> = new Map();

  static getInstance(): RBACService {
    if (!RBACService.instance) {
      RBACService.instance = new RBACService();
    }
    return RBACService.instance;
  }

  constructor() {
    this.initializeDefaultRoles();
  }

  private initializeDefaultRoles() {
    // Define permissions
    const permissions: Permission[] = [
      // User permissions
      { id: 'user:read', name: 'Read User', description: 'View user information', resource: 'user', action: 'read' },
      { id: 'user:update', name: 'Update User', description: 'Update user information', resource: 'user', action: 'update' },
      { id: 'user:delete', name: 'Delete User', description: 'Delete user account', resource: 'user', action: 'delete' },
      { id: 'user:create', name: 'Create User', description: 'Create new user', resource: 'user', action: 'create' },
      
      // Post permissions
      { id: 'post:read', name: 'Read Post', description: 'View posts', resource: 'post', action: 'read' },
      { id: 'post:create', name: 'Create Post', description: 'Create new posts', resource: 'post', action: 'create' },
      { id: 'post:update', name: 'Update Post', description: 'Update posts', resource: 'post', action: 'update' },
      { id: 'post:delete', name: 'Delete Post', description: 'Delete posts', resource: 'post', action: 'delete' },
      { id: 'post:publish', name: 'Publish Post', description: 'Publish posts', resource: 'post', action: 'publish' },
      
      // Admin permissions
      { id: 'admin:dashboard', name: 'Admin Dashboard', description: 'Access admin dashboard', resource: 'admin', action: 'dashboard' },
      { id: 'admin:users', name: 'Manage Users', description: 'Manage all users', resource: 'admin', action: 'users' },
      { id: 'admin:roles', name: 'Manage Roles', description: 'Manage roles and permissions', resource: 'admin', action: 'roles' },
      { id: 'admin:settings', name: 'System Settings', description: 'Manage system settings', resource: 'admin', action: 'settings' },
    ];

    permissions.forEach(permission => {
      this.permissions.set(permission.id, permission);
    });

    // Define roles
    const roles: Role[] = [
      {
        id: 'guest',
        name: 'Guest',
        description: 'Unauthenticated user',
        permissions: [this.permissions.get('post:read')!],
        priority: 0,
      },
      {
        id: 'user',
        name: 'User',
        description: 'Regular authenticated user',
        permissions: [
          this.permissions.get('user:read')!,
          this.permissions.get('user:update')!,
          this.permissions.get('post:read')!,
          this.permissions.get('post:create')!,
        ],
        isDefault: true,
        priority: 1,
      },
      {
        id: 'moderator',
        name: 'Moderator',
        description: 'Content moderator',
        permissions: [
          this.permissions.get('user:read')!,
          this.permissions.get('user:update')!,
          this.permissions.get('post:read')!,
          this.permissions.get('post:create')!,
          this.permissions.get('post:update')!,
          this.permissions.get('post:delete')!,
        ],
        priority: 2,
      },
      {
        id: 'admin',
        name: 'Administrator',
        description: 'System administrator',
        permissions: Array.from(this.permissions.values()),
        priority: 3,
      },
    ];

    roles.forEach(role => {
      this.roles.set(role.id, role);
    });
  }

  hasPermission(user: RBACUser | null, permissionId: string): boolean {
    if (!user) {
      // Check guest permissions
      const guestRole = this.roles.get('guest');
      return guestRole?.permissions.some(p => p.id === permissionId) || false;
    }

    // Check direct permissions
    if (user.permissions?.some(p => p.id === permissionId)) {
      return true;
    }

    // Check role permissions
    return user.roles?.some(role => 
      role.permissions.some(p => p.id === permissionId)
    ) || false;
  }

  hasRole(user: RBACUser | null, roleId: string): boolean {
    if (!user) return roleId === 'guest';
    return user.roles?.some(role => role.id === roleId) || false;
  }

  hasAnyRole(user: RBACUser | null, roleIds: string[]): boolean {
    return roleIds.some(roleId => this.hasRole(user, roleId));
  }

  canAccess(user: RBACUser | null, resource: string, action: string): boolean {
    const permissionId = `${resource}:${action}`;
    return this.hasPermission(user, permissionId);
  }

  getHighestRole(user: RBACUser | null): Role | null {
    if (!user || !user.roles?.length) {
      return this.roles.get('guest') || null;
    }

    return user.roles.reduce((highest, current) => 
      current.priority > (highest?.priority || 0) ? current : highest
    , null as Role | null);
  }

  async getUserWithRoles(userId: string): Promise<RBACUser | null> {
    const user = await prisma.user.findUnique({
      where: { id: userId },
      include: {
        roles: {
          include: {
            permissions: true,
          },
        },
        permissions: true,
      },
    });

    return user as RBACUser;
  }

  async assignRole(userId: string, roleId: string): Promise<void> {
    const role = this.roles.get(roleId);
    if (!role) {
      throw new Error(`Role ${roleId} not found`);
    }

    await prisma.userRole.create({
      data: {
        userId,
        roleId,
      },
    });
  }

  async removeRole(userId: string, roleId: string): Promise<void> {
    await prisma.userRole.deleteMany({
      where: {
        userId,
        roleId,
      },
    });
  }

  async grantPermission(userId: string, permissionId: string): Promise<void> {
    const permission = this.permissions.get(permissionId);
    if (!permission) {
      throw new Error(`Permission ${permissionId} not found`);
    }

    await prisma.userPermission.create({
      data: {
        userId,
        permissionId,
      },
    });
  }

  async revokePermission(userId: string, permissionId: string): Promise<void> {
    await prisma.userPermission.deleteMany({
      where: {
        userId,
        permissionId,
      },
    });
  }

  getAllRoles(): Role[] {
    return Array.from(this.roles.values());
  }

  getAllPermissions(): Permission[] {
    return Array.from(this.permissions.values());
  }

  getRole(roleId: string): Role | undefined {
    return this.roles.get(roleId);
  }

  getPermission(permissionId: string): Permission | undefined {
    return this.permissions.get(permissionId);
  }
}

export const rbac = RBACService.getInstance();

// React hooks for RBAC
export function useRBAC() {
  const { user } = useAuthSession();
  const rbacUser = user as RBACUser;

  const hasPermission = useCallback(
    (permissionId: string) => rbac.hasPermission(rbacUser, permissionId),
    [rbacUser]
  );

  const hasRole = useCallback(
    (roleId: string) => rbac.hasRole(rbacUser, roleId),
    [rbacUser]
  );

  const hasAnyRole = useCallback(
    (roleIds: string[]) => rbac.hasAnyRole(rbacUser, roleIds),
    [rbacUser]
  );

  const canAccess = useCallback(
    (resource: string, action: string) => rbac.canAccess(rbacUser, resource, action),
    [rbacUser]
  );

  return {
    user: rbacUser,
    hasPermission,
    hasRole,
    hasAnyRole,
    canAccess,
    highestRole: rbac.getHighestRole(rbacUser),
  };
}

// HOC for role-based component rendering
export function withRoleGuard<P extends object>(
  Component: React.ComponentType<P>,
  requiredRoles: string[],
  fallback?: React.ComponentType<P>
) {
  return function RoleGuardedComponent(props: P) {
    const { hasAnyRole } = useRBAC();

    if (!hasAnyRole(requiredRoles)) {
      if (fallback) {
        const FallbackComponent = fallback;
        return <FallbackComponent {...props} />;
      }
      
      return (
        <div className="text-center py-12">
          <h3 className="text-lg font-medium text-gray-900 mb-2">
            Access Denied
          </h3>
          <p className="text-gray-600">
            You don't have permission to view this content.
          </p>
        </div>
      );
    }

    return <Component {...props} />;
  };
}

// Permission-based component
export function PermissionGuard({
  children,
  permission,
  resource,
  action,
  fallback,
}: {
  children: React.ReactNode;
  permission?: string;
  resource?: string;
  action?: string;
  fallback?: React.ReactNode;
}) {
  const { hasPermission, canAccess } = useRBAC();

  const hasAccess = permission 
    ? hasPermission(permission)
    : resource && action 
    ? canAccess(resource, action)
    : false;

  if (!hasAccess) {
    return <>{fallback}</> || null;
  }

  return <>{children}</>;
}
```

## Frontend Authentication Patterns {#frontend-patterns}

### Authentication Context and Hooks

```typescript
// contexts/AuthContext.tsx
import React, { createContext, useContext, useEffect, useState, useCallback } from 'react';
import { useRouter } from 'next/router';

interface AuthContextType {
  user: User | null;
  isLoading: boolean;
  isAuthenticated: boolean;
  login: (credentials: LoginCredentials) => Promise<void>;
  logout: () => Promise<void>;
  register: (userData: RegisterData) => Promise<void>;
  refreshToken: () => Promise<void>;
  updateUser: (updates: Partial<User>) => Promise<void>;
  error: string | null;
  clearError: () => void;
}

const AuthContext = createContext<AuthContextType | undefined>(undefined);

export function AuthProvider({ children }: { children: React.ReactNode }) {
  const [user, setUser] = useState<User | null>(null);
  const [isLoading, setIsLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);
  const router = useRouter();

  const clearError = useCallback(() => setError(null), []);

  const setAuthData = useCallback((userData: User, accessToken: string) => {
    setUser(userData);
    localStorage.setItem('accessToken', accessToken);
    setIsLoading(false);
  }, []);

  const clearAuthData = useCallback(() => {
    setUser(null);
    localStorage.removeItem('accessToken');
    setIsLoading(false);
  }, []);

  const login = useCallback(async (credentials: LoginCredentials) => {
    try {
      setIsLoading(true);
      setError(null);

      const response = await fetch('/api/auth/login', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(credentials),
      });

      const data = await response.json();

      if (!response.ok) {
        throw new Error(data.error || 'Login failed');
      }

      setAuthData(data.user, data.accessToken);
      
      // Redirect to intended page or dashboard
      const redirectTo = router.query.redirectTo as string || '/dashboard';
      router.push(redirectTo);
    } catch (err) {
      setError(err instanceof Error ? err.message : 'Login failed');
      setIsLoading(false);
    }
  }, [router, setAuthData]);

  const register = useCallback(async (userData: RegisterData) => {
    try {
      setIsLoading(true);
      setError(null);

      const response = await fetch('/api/auth/register', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(userData),
      });

      const data = await response.json();

      if (!response.ok) {
        throw new Error(data.error || 'Registration failed');
      }

      setAuthData(data.user, data.accessToken);
      router.push('/dashboard');
    } catch (err) {
      setError(err instanceof Error ? err.message : 'Registration failed');
      setIsLoading(false);
    }
  }, [router, setAuthData]);

  const logout = useCallback(async () => {
    try {
      // Call logout API to invalidate session
      await fetch('/api/auth/logout', {
        method: 'POST',
        headers: {
          'Authorization': `Bearer ${localStorage.getItem('accessToken')}`,
        },
      });
    } catch (error) {
      console.error('Logout error:', error);
    } finally {
      clearAuthData();
      router.push('/');
    }
  }, [clearAuthData, router]);

  const refreshToken = useCallback(async () => {
    try {
      const response = await fetch('/api/auth/refresh', {
        method: 'POST',
      });

      if (!response.ok) {
        throw new Error('Token refresh failed');
      }

      const data = await response.json();
      localStorage.setItem('accessToken', data.accessToken);
    } catch (error) {
      console.error('Token refresh error:', error);
      clearAuthData();
      router.push('/auth/signin');
    }
  }, [clearAuthData, router]);

  const updateUser = useCallback(async (updates: Partial<User>) => {
    try {
      const response = await fetch('/api/user/profile', {
        method: 'PATCH',
        headers: {
          'Content-Type': 'application/json',
          'Authorization': `Bearer ${localStorage.getItem('accessToken')}`,
        },
        body: JSON.stringify(updates),
      });

      const data = await response.json();

      if (!response.ok) {
        throw new Error(data.error || 'Update failed');
      }

      setUser(data.user);
    } catch (err) {
      setError(err instanceof Error ? err.message : 'Update failed');
    }
  }, []);

  // Initialize auth state
  useEffect(() => {
    const initializeAuth = async () => {
      const accessToken = localStorage.getItem('accessToken');
      
      if (!accessToken) {
        setIsLoading(false);
        return;
      }

      try {
        const response = await fetch('/api/auth/me', {
          headers: {
            'Authorization': `Bearer ${accessToken}`,
          },
        });

        if (response.ok) {
          const userData = await response.json();
          setUser(userData.user);
        } else {
          // Try to refresh token
          await refreshToken();
        }
      } catch (error) {
        console.error('Auth initialization error:', error);
        clearAuthData();
      } finally {
        setIsLoading(false);
      }
    };

    initializeAuth();
  }, [refreshToken, clearAuthData]);

  // Auto-refresh token
  useEffect(() => {
    if (!user) return;

    const interval = setInterval(() => {
      refreshToken();
    }, 14 * 60 * 1000); // Refresh every 14 minutes

    return () => clearInterval(interval);
  }, [user, refreshToken]);

  const value: AuthContextType = {
    user,
    isLoading,
    isAuthenticated: !!user,
    login,
    logout,
    register,
    refreshToken,
    updateUser,
    error,
    clearError,
  };

  return (
    <AuthContext.Provider value={value}>
      {children}
    </AuthContext.Provider>
  );
}

export function useAuth() {
  const context = useContext(AuthContext);
  if (context === undefined) {
    throw new Error('useAuth must be used within an AuthProvider');
  }
  return context;
}

// Route protection HOCs
export function withAuthRequired<P extends object>(
  Component: React.ComponentType<P>
) {
  return function AuthRequiredComponent(props: P) {
    const { isAuthenticated, isLoading } = useAuth();
    const router = useRouter();

    useEffect(() => {
      if (!isLoading && !isAuthenticated) {
        router.push(`/auth/signin?redirectTo=${encodeURIComponent(router.asPath)}`);
      }
    }, [isAuthenticated, isLoading, router]);

    if (isLoading) {
      return (
        <div className="flex items-center justify-center h-screen">
          <div className="animate-spin rounded-full h-12 w-12 border-b-2 border-blue-600"></div>
        </div>
      );
    }

    if (!isAuthenticated) {
      return null;
    }

    return <Component {...props} />;
  };
}

export function withAuthOptional<P extends object>(
  Component: React.ComponentType<P>
) {
  return function AuthOptionalComponent(props: P) {
    const { isLoading } = useAuth();

    if (isLoading) {
      return (
        <div className="flex items-center justify-center h-screen">
          <div className="animate-spin rounded-full h-12 w-12 border-b-2 border-blue-600"></div>
        </div>
      );
    }

    return <Component {...props} />;
  };
}
```

## Best Practices {#best-practices}

### Security Checklist

```typescript
// utils/authSecurityChecklist.ts
interface SecurityCheck {
  name: string;
  description: string;
  check: () => boolean | Promise<boolean>;
  severity: 'critical' | 'high' | 'medium' | 'low';
  fix: string;
}

export const authSecurityChecks: SecurityCheck[] = [
  {
    name: 'HTTPS Enforcement',
    description: 'All authentication endpoints must use HTTPS',
    check: () => process.env.NODE_ENV === 'production' ? 
      window.location.protocol === 'https:' : true,
    severity: 'critical',
    fix: 'Configure HTTPS in production environment',
  },
  {
    name: 'Secure Headers',
    description: 'Security headers should be set',
    check: async () => {
      const response = await fetch('/api/auth/status');
      const hasCSP = response.headers.get('content-security-policy');
      const hasHSTS = response.headers.get('strict-transport-security');
      return !!(hasCSP && hasHSTS);
    },
    severity: 'high',
    fix: 'Configure security headers in next.config.js',
  },
  {
    name: 'Password Strength',
    description: 'Password should meet strength requirements',
    check: () => true, // Implement based on your requirements
    severity: 'high',
    fix: 'Implement password strength validation',
  },
  {
    name: 'Rate Limiting',
    description: 'Login endpoints should have rate limiting',
    check: () => true, // Check if rate limiting is configured
    severity: 'high',
    fix: 'Implement rate limiting for authentication endpoints',
  },
  {
    name: 'Session Security',
    description: 'Sessions should be properly configured',
    check: () => {
      const hasSecureCookies = document.cookie.includes('Secure');
      const hasHttpOnly = !document.cookie.includes('refreshToken');
      return hasSecureCookies && hasHttpOnly;
    },
    severity: 'high',
    fix: 'Configure secure session cookies',
  },
];

// Authentication best practices guide
export const authBestPractices = {
  passwords: {
    minLength: 8,
    requireUppercase: true,
    requireLowercase: true,
    requireNumbers: true,
    requireSpecialChars: true,
    preventCommonPasswords: true,
    preventPasswordReuse: 5, // Last 5 passwords
  },
  
  sessions: {
    maxAge: 24 * 60 * 60, // 24 hours
    renewThreshold: 15 * 60, // 15 minutes before expiry
    secureTransport: true,
    httpOnly: true,
    sameSite: 'strict',
  },
  
  tokens: {
    accessTokenExpiry: 15 * 60, // 15 minutes
    refreshTokenExpiry: 7 * 24 * 60 * 60, // 7 days
    jwtAlgorithm: 'HS256',
    tokenRotation: true,
  },
  
  rateLimit: {
    loginAttempts: 5,
    windowMs: 15 * 60 * 1000, // 15 minutes
    blockDuration: 60 * 60 * 1000, // 1 hour
  },
  
  monitoring: {
    logFailedAttempts: true,
    logSuccessfulLogins: true,
    alertOnSuspiciousActivity: true,
    trackSessionActivity: true,
  },
};
```

## Exercises

### Exercise 1: Complete Authentication System
Build a comprehensive authentication system with:
- User registration and email verification
- Login with multiple providers
- Password reset functionality
- Role-based access control
- Session management and security

### Exercise 2: Enterprise SSO Integration
Implement enterprise authentication with:
- SAML 2.0 integration
- Active Directory connection
- Multi-factor authentication
- Audit logging and compliance
- User provisioning and deprovisioning

### Exercise 3: Social Authentication
Create a social login system with:
- Multiple OAuth providers
- Account linking and unlinking
- Profile synchronization
- Privacy controls
- Social data integration

### Exercise 4: Mobile Authentication
Build mobile-optimized authentication with:
- JWT token management
- Biometric authentication
- Offline authentication
- Push notification verification
- Device management

## Summary

Authentication in Next.js involves multiple layers of security:

- **NextAuth.js**: Rapid development with built-in providers and security
- **Custom Authentication**: Full control over authentication flow and data
- **JWT Management**: Stateless authentication with proper token handling
- **Role-Based Access Control**: Granular permissions and access management
- **Security Best Practices**: Comprehensive security measures and monitoring

Key principles:
- Always use HTTPS in production
- Implement proper session management
- Use strong password policies
- Enable rate limiting and monitoring
- Follow the principle of least privilege
- Regularly audit and update security measures

Next, we'll explore **Database Integration** patterns and advanced data management techniques in Next.js applications!
