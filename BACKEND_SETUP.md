# Backend Setup Guide: Cloudflare Workers + D1 + R2

Complete guide for setting up and developing the UCC Student Portal backend (Cloudflare Workers API).

## 📋 Table of Contents

1. [Quick Start](#quick-start)
2. [Prerequisites & Installation](#prerequisites--installation)
3. [Wrangler Configuration](#wrangler-configuration)
4. [Project Structure](#project-structure)
5. [Database Setup (D1)](#database-setup-d1)
6. [File Storage Setup (R2)](#file-storage-setup-r2)
7. [Environment Variables](#environment-variables)
8. [Development Workflow](#development-workflow)
9. [API Route Handlers](#api-route-handlers)
10. [Middleware & Authentication](#middleware--authentication)
11. [Deployment](#deployment)

---

## 🚀 Quick Start

### Prerequisites
- **Node.js**: 18.0.0 or higher
- **npm**: 9.0.0 or higher
- **Cloudflare Account**: Free tier sufficient for development
- **Git**: For version control

### Installation & Development

```bash
# Navigate to backend directory
cd UCC-Student-Portal/backend

# Install dependencies
npm install

# Start development server
npm run dev
```

**Development Server runs on**: `http://localhost:8787`

---

## 📦 Prerequisites & Installation

### Step 1: Install Wrangler CLI

```bash
npm install -g wrangler@latest
# or use local installation (already in package.json)
npm install
```

### Step 2: Authenticate with Cloudflare

```bash
wrangler login
# Opens browser to authenticate
# Grants local CLI access to your Cloudflare account
```

### Step 3: Create D1 Database

```bash
# Create SQLite database for production
wrangler d1 create ucc-student-portal

# Returns database ID (save this for wrangler.toml)
# Example output:
# ✅ Successfully created DB "ucc-student-portal"
# 📝 Please add the following to your wrangler.toml:
# [[d1_databases]]
# binding = "DB"
# database_name = "ucc-student-portal"
# database_id = "12345678-1234-1234-1234-123456789012"
```

### Step 4: Create R2 Bucket

```bash
# Create R2 bucket for file storage
wrangler r2 bucket create ucc-student-portal

# Returns bucket name and stores locally for development
```

### Step 5: Initial Project Setup

```bash
# Install npm dependencies
npm install

# Run database migrations
npm run migrate

# Start development server
npm run dev
```

---

## ⚙️ Wrangler Configuration

Create `wrangler.toml` in the `backend/` directory:

```toml
#:schema node_modules/wrangler/assets/config-schema.json
name = "ucc-student-portal"
main = "src/worker.ts"
type = "service"
compatibility_date = "2024-01-01"

# D1 Database Binding
[[d1_databases]]
binding = "DB"
database_name = "ucc-student-portal"
database_id = "YOUR_DATABASE_ID"

# R2 Bucket Binding
[[r2_buckets]]
bucket_name = "ucc-student-portal"
binding = "BUCKET"

# Environment Variables
[env.production]
routes = [
  { pattern = "api.ucc-student-portal.com/*", zone_name = "ucc-student-portal.com" }
]

[env.development]
vars = { ENVIRONMENT = "development" }

[env.staging]
vars = { ENVIRONMENT = "staging" }

# Custom Domain (after purchase)
[custom_domain]
name = "api.ucc-student-portal.com"

# Triggers for scheduled tasks
[[triggers.crons]]
cron = "0 0 * * *"
job = "cleanup-old-notifications"

[[triggers.crons]]
cron = "0 */6 * * *"
job = "sync-external-data"
```

---

## 📁 Project Structure

```
backend/
├── src/
│   ├── worker.ts                     # Worker entry point
│   │
│   ├── api/                          # Route handlers
│   │   ├── auth.ts                   # Auth endpoints (/login, /register, /refresh)
│   │   ├── users.ts                  # User endpoints (/users, /users/:id)
│   │   ├── posts.ts                  # Post endpoints (/posts, /posts/:id)
│   │   ├── comments.ts               # Comment endpoints
│   │   ├── reactions.ts              # Reaction endpoints
│   │   ├── notifications.ts          # Notification endpoints
│   │   ├── search.ts                 # Search endpoints
│   │   ├── blocks.ts                 # Block/mute endpoints
│   │   └── reports.ts                # Report endpoints
│   │
│   ├── middleware/                   # Request middleware
│   │   ├── auth.ts                   # JWT verification
│   │   ├── errorHandler.ts           # Global error handling
│   │   ├── cors.ts                   # CORS configuration
│   │   ├── logging.ts                # Request logging
│   │   └── validation.ts             # Input validation
│   │
│   ├── db/                           # Database utilities
│   │   ├── schema.ts                 # Database schema types
│   │   ├── queries.ts                # Prepared statements
│   │   └── init.ts                   # Database initialization
│   │
│   ├── services/                     # Business logic
│   │   ├── authService.ts            # Authentication logic
│   │   ├── userService.ts            # User operations
│   │   ├── postService.ts            # Post operations
│   │   ├── tokenService.ts           # JWT handling
│   │   └── emailService.ts           # Email notifications (future)
│   │
│   ├── types/                        # TypeScript interfaces
│   │   ├── auth.types.ts
│   │   ├── user.types.ts
│   │   ├── post.types.ts
│   │   ├── api.types.ts
│   │   └── env.types.ts              # Environment variables type
│   │
│   ├── lib/                          # Utilities
│   │   ├── utils.ts                  # General utilities
│   │   ├── validators.ts             # Input validation
│   │   ├── hash.ts                   # Password hashing (bcrypt)
│   │   ├── jwt.ts                    # JWT token handling
│   │   └── constants.ts              # Constants
│   │
│   └── env.d.ts                      # Environment types
│
├── migrations/                       # D1 database migrations
│   ├── 0001_init_schema.sql          # Initial schema
│   ├── 0002_add_notifications.sql
│   ├── 0003_add_indexes.sql
│   └── 0004_add_user_preferences.sql
│
├── tests/                            # Test files
│   ├── auth.test.ts
│   ├── posts.test.ts
│   └── integration.test.ts
│
├── wrangler.toml                     # Cloudflare Workers config
├── wrangler.toml.local               # Local development config (gitignored)
├── tsconfig.json                     # TypeScript configuration
├── package.json
├── package-lock.json
└── README.md
```

---

## 🗄️ Database Setup (D1)

### Initial Schema Migration

Create `migrations/0001_init_schema.sql`:

```sql
-- Create users table
CREATE TABLE IF NOT EXISTS users (
  id TEXT PRIMARY KEY DEFAULT (gen_random_uuid()),
  email TEXT UNIQUE NOT NULL,
  password_hash TEXT NOT NULL,
  first_name TEXT NOT NULL,
  last_name TEXT NOT NULL,
  username TEXT UNIQUE NOT NULL,
  profile_picture_url TEXT,
  bio TEXT,
  campus_id TEXT NOT NULL,
  status TEXT DEFAULT 'active',
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  updated_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  deleted_at DATETIME
);

-- Create posts table
CREATE TABLE IF NOT EXISTS posts (
  id TEXT PRIMARY KEY DEFAULT (gen_random_uuid()),
  user_id TEXT NOT NULL,
  content TEXT NOT NULL,
  image_url TEXT,
  privacy TEXT DEFAULT 'public',
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  updated_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  deleted_at DATETIME,
  FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);

-- Create comments table
CREATE TABLE IF NOT EXISTS comments (
  id TEXT PRIMARY KEY DEFAULT (gen_random_uuid()),
  post_id TEXT NOT NULL,
  user_id TEXT NOT NULL,
  content TEXT NOT NULL,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  updated_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  deleted_at DATETIME,
  FOREIGN KEY (post_id) REFERENCES posts(id) ON DELETE CASCADE,
  FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);

-- Create reactions table
CREATE TABLE IF NOT EXISTS reactions (
  id TEXT PRIMARY KEY DEFAULT (gen_random_uuid()),
  post_id TEXT NOT NULL,
  comment_id TEXT,
  user_id TEXT NOT NULL,
  reaction_type TEXT NOT NULL,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (post_id) REFERENCES posts(id) ON DELETE CASCADE,
  FOREIGN KEY (comment_id) REFERENCES comments(id) ON DELETE CASCADE,
  FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
  UNIQUE(post_id, comment_id, user_id)
);

-- Create indexes for performance
CREATE INDEX idx_posts_user_id ON posts(user_id);
CREATE INDEX idx_posts_created_at ON posts(created_at DESC);
CREATE INDEX idx_comments_post_id ON comments(post_id);
CREATE INDEX idx_reactions_post_id ON reactions(post_id);
```

### Run Migrations

```bash
# Apply migrations locally for development
wrangler d1 execute ucc-student-portal --local < migrations/0001_init_schema.sql

# Apply to production database
wrangler d1 execute ucc-student-portal --remote < migrations/0001_init_schema.sql
```

---

## 📦 File Storage Setup (R2)

### Create R2 Bucket

```bash
# Create bucket
wrangler r2 bucket create ucc-student-portal

# List buckets
wrangler r2 bucket list
```

### Upload Handling in Worker

```typescript
// src/api/uploads.ts
import { Router } from 'itty-router'

export const uploadsRouter = Router()

uploadsRouter.post('/upload', async (request, env) => {
  const formData = await request.formData()
  const file = formData.get('file') as File

  if (!file) {
    return new Response(JSON.stringify({ error: 'No file provided' }), {
      status: 400,
    })
  }

  const filename = `${Date.now()}-${file.name}`
  
  try {
    // Upload to R2
    await env.BUCKET.put(filename, await file.arrayBuffer(), {
      httpMetadata: {
        contentType: file.type,
      },
    })

    const url = `https://cdn.ucc-student-portal.com/${filename}`
    
    return new Response(JSON.stringify({ url, filename }), {
      status: 200,
    })
  } catch (error) {
    return new Response(JSON.stringify({ error: 'Upload failed' }), {
      status: 500,
    })
  }
})
```

---

## 🔐 Environment Variables

### Development (`wrangler.toml.local`)

```toml
name = "ucc-student-portal"

[env.development]
vars = {
  ENVIRONMENT = "development",
  JWT_SECRET = "dev-secret-key-change-in-production",
  JWT_EXPIRY = "7d",
  CORS_ORIGIN = "http://localhost:5173",
  API_URL = "http://localhost:8787"
}
```

### Production (Set in Cloudflare Dashboard)

```
ENVIRONMENT=production
JWT_SECRET=<strong-random-key>
JWT_EXPIRY=7d
CORS_ORIGIN=https://ucc-student-portal.com
API_URL=https://api.ucc-student-portal.com
```

### Access in Worker

```typescript
// src/types/env.types.ts
export interface Env {
  DB: D1Database
  BUCKET: R2Bucket
  ENVIRONMENT: 'development' | 'staging' | 'production'
  JWT_SECRET: string
  JWT_EXPIRY: string
  CORS_ORIGIN: string
  API_URL: string
}
```

---

## 🔧 Development Workflow

### Start Development Server

```bash
npm run dev
# Runs: wrangler dev
# Serves on http://localhost:8787
# Hot reload enabled - changes auto-refresh
```

### Test Endpoints Locally

```bash
# Test login endpoint
curl -X POST http://localhost:8787/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"user@example.com","password":"password123"}'

# Test with token
curl -X GET http://localhost:8787/api/users/me \
  -H "Authorization: Bearer YOUR_TOKEN"
```

### Database Access

```bash
# Open local database shell
wrangler d1 execute ucc-student-portal --local

# Run queries
sqlite> SELECT COUNT(*) FROM users;
sqlite> .exit
```

---

## 🔌 API Route Handlers

### Example: Authentication Routes (`src/api/auth.ts`)

```typescript
import { Router } from 'itty-router'
import { verify, sign } from '@tsndr/cloudflare-worker-jwt'
import { hash, compare } from 'bcryptjs'
import type { Env } from '@/types/env.types'

export const authRouter = Router()

// POST /auth/register
authRouter.post('/auth/register', async (request, env: Env) => {
  try {
    const { email, password, firstName, lastName, username } = await request.json()

    // Validate input
    if (!email || !password || !username) {
      return new Response(
        JSON.stringify({ error: 'Missing required fields' }),
        { status: 400 }
      )
    }

    // Hash password
    const passwordHash = await hash(password, 10)

    // Insert into database
    const result = await env.DB.prepare(
      `INSERT INTO users (email, password_hash, first_name, last_name, username)
       VALUES (?, ?, ?, ?, ?)`
    ).bind(email, passwordHash, firstName, lastName, username).run()

    return new Response(JSON.stringify({
      success: true,
      message: 'User registered successfully',
      userId: result.meta.last_row_id,
    }), { status: 201 })
  } catch (error) {
    return new Response(
      JSON.stringify({ error: 'Registration failed' }),
      { status: 500 }
    )
  }
})

// POST /auth/login
authRouter.post('/auth/login', async (request, env: Env) => {
  try {
    const { email, password } = await request.json()

    // Get user from database
    const user = await env.DB.prepare(
      'SELECT * FROM users WHERE email = ?'
    ).bind(email).first()

    if (!user) {
      return new Response(
        JSON.stringify({ error: 'Invalid credentials' }),
        { status: 401 }
      )
    }

    // Verify password
    const isValidPassword = await compare(password, user.password_hash)

    if (!isValidPassword) {
      return new Response(
        JSON.stringify({ error: 'Invalid credentials' }),
        { status: 401 }
      )
    }

    // Generate JWT token
    const token = await sign({ userId: user.id }, env.JWT_SECRET)

    return new Response(JSON.stringify({
      token,
      user: {
        id: user.id,
        email: user.email,
        username: user.username,
      },
    }), { status: 200 })
  } catch (error) {
    return new Response(
      JSON.stringify({ error: 'Login failed' }),
      { status: 500 }
    )
  }
})
```

### Example: Post Routes (`src/api/posts.ts`)

```typescript
import { Router } from 'itty-router'
import { requireAuth } from '@/middleware/auth'

export const postsRouter = Router()

// GET /posts (public feed)
postsRouter.get('/posts', async (request, env: Env) => {
  try {
    const posts = await env.DB.prepare(`
      SELECT p.*, u.username, u.profile_picture_url
      FROM posts p
      JOIN users u ON p.user_id = u.id
      WHERE p.deleted_at IS NULL
      ORDER BY p.created_at DESC
      LIMIT 20
    `).all()

    return new Response(JSON.stringify(posts), { status: 200 })
  } catch (error) {
    return new Response(
      JSON.stringify({ error: 'Failed to fetch posts' }),
      { status: 500 }
    )
  }
})

// POST /posts (create new post)
postsRouter.post('/posts', requireAuth, async (request, env: Env) => {
  try {
    const { content, imageUrl } = await request.json()
    const userId = request.userId // from auth middleware

    if (!content?.trim()) {
      return new Response(
        JSON.stringify({ error: 'Content is required' }),
        { status: 400 }
      )
    }

    const result = await env.DB.prepare(
      `INSERT INTO posts (user_id, content, image_url)
       VALUES (?, ?, ?)`
    ).bind(userId, content, imageUrl || null).run()

    return new Response(JSON.stringify({
      success: true,
      postId: result.meta.last_row_id,
    }), { status: 201 })
  } catch (error) {
    return new Response(
      JSON.stringify({ error: 'Failed to create post' }),
      { status: 500 }
    )
  }
})
```

---

## 🔒 Middleware & Authentication

### JWT Authentication Middleware (`src/middleware/auth.ts`)

```typescript
import { verify } from '@tsndr/cloudflare-worker-jwt'
import type { Env } from '@/types/env.types'

export async function requireAuth(request: Request, env: Env) {
  const authHeader = request.headers.get('Authorization')

  if (!authHeader?.startsWith('Bearer ')) {
    return new Response(
      JSON.stringify({ error: 'Missing or invalid authorization header' }),
      { status: 401 }
    )
  }

  const token = authHeader.slice(7)

  try {
    const payload = await verify(token, env.JWT_SECRET)
    
    // Attach user ID to request for downstream handlers
    ;(request as any).userId = payload.userId

    return request
  } catch (error) {
    return new Response(
      JSON.stringify({ error: 'Invalid or expired token' }),
      { status: 401 }
    )
  }
}

// Usage in routes:
// router.post('/posts', requireAuth, async (request) => { ... })
```

### Error Handler Middleware

```typescript
export function errorHandler(error: any) {
  console.error('API Error:', error)

  if (error instanceof ValidationError) {
    return new Response(
      JSON.stringify({ error: error.message }),
      { status: 400 }
    )
  }

  if (error instanceof AuthError) {
    return new Response(
      JSON.stringify({ error: error.message }),
      { status: 401 }
    )
  }

  return new Response(
    JSON.stringify({ error: 'Internal server error' }),
    { status: 500 }
  )
}
```

---

## 🚀 Deployment

### Deploy to Cloudflare Workers

```bash
# Deploy production version
npm run deploy

# Or manually:
wrangler deploy

# Deploy to specific environment
wrangler deploy --env production
```

### Verify Deployment

```bash
# Check deployment status
wrangler deployments list

# Test production API
curl https://api.ucc-student-portal.com/api/health
```

### Custom Domain Setup

1. Buy domain (Namecheap, GoDaddy, etc.)
2. Add to Cloudflare in Dashboard
3. Update `wrangler.toml`:
   ```toml
   [custom_domain]
   name = "api.ucc-student-portal.com"
   ```
4. Deploy: `wrangler deploy --env production`

### Monitoring & Logs

```bash
# View recent logs
wrangler tail

# View specific environment logs
wrangler tail --env production

# Export logs
wrangler tail > logs.txt
```

---

## 📚 Additional Resources

- [Cloudflare Workers Documentation](https://developers.cloudflare.com/workers)
- [D1 Database Docs](https://developers.cloudflare.com/d1)
- [R2 Object Storage Docs](https://developers.cloudflare.com/r2)
- [Wrangler CLI Reference](https://developers.cloudflare.com/workers/wrangler)
- [JWT Best Practices](https://tools.ietf.org/html/rfc8725)

---

**Questions?** Refer to [API_SPECIFICATION.md](./API_SPECIFICATION.md) for endpoint details or [AUTHENTICATION_GUIDE.md](./AUTHENTICATION_GUIDE.md) for JWT implementation.
