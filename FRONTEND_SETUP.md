# Frontend Setup Guide: Vite 5 + React 18 + TypeScript

Complete guide for setting up and developing the UCC Student Portal frontend SPA.

## 📋 Table of Contents

1. [Quick Start](#quick-start)
2. [Project Dependencies](#project-dependencies)
3. [Project Structure](#project-structure)
4. [Configuration Files](#configuration-files)
5. [Environment Variables](#environment-variables)
6. [npm Scripts](#npm-scripts)
7. [API Integration](#api-integration)
8. [Testing Setup](#testing-setup)
9. [Mobile-First Design](#mobile-first-design)
10. [Accessibility Guidelines](#accessibility-guidelines)
11. [Deployment](#deployment)

---

## 🚀 Quick Start

### Prerequisites
- **Node.js**: 18.0.0 or higher
- **npm**: 9.0.0 or higher (or yarn 4.0+)
- **Git**: For cloning and version control

### Installation & Development

```bash
# Clone repository
git clone https://github.com/allyzza010501-lab/UCC-student-portal.git
cd UCC-Student-Portal/frontend

# Install dependencies
npm install

# Start development server
npm run dev
```

**Development Server runs on**: `http://localhost:5173`

Hot Module Replacement (HMR) enabled - changes reflect instantly (< 100ms).

---

## 📦 Project Dependencies

### Core Dependencies

```json
{
  "react": "^18.2.0",
  "react-dom": "^18.2.0",
  "react-router-dom": "^6.20.0",
  "zustand": "^4.4.0",
  "@tanstack/react-query": "^5.32.0",
  "axios": "^1.6.0",
  "typescript": "^5.3.0"
}
```

### Development Dependencies

```json
{
  "vite": "^5.0.0",
  "@vitejs/plugin-react": "^4.2.0",
  "tailwindcss": "^3.3.0",
  "postcss": "^8.4.0",
  "autoprefixer": "^10.4.0",
  "typescript": "^5.3.0",
  "@types/react": "^18.2.0",
  "@types/react-dom": "^18.2.0",
  "@types/node": "^20.10.0",
  "vitest": "^1.0.0",
  "@testing-library/react": "^14.1.0",
  "@testing-library/jest-dom": "^6.1.0",
  "eslint": "^8.55.0",
  "@typescript-eslint/eslint-plugin": "^6.13.0",
  "@typescript-eslint/parser": "^6.13.0"
}
```

---

## 📁 Project Structure

```
frontend/
├── src/
│   ├── components/
│   │   ├── auth/                     # Authentication-related components
│   │   │   ├── LoginForm.tsx
│   │   │   ├── RegisterForm.tsx
│   │   │   └── LogoutButton.tsx
│   │   ├── common/                   # Shared UI components
│   │   │   ├── Header.tsx
│   │   │   ├── Footer.tsx
│   │   │   ├── Navigation.tsx
│   │   │   ├── Card.tsx
│   │   │   └── Button.tsx
│   │   ├── posts/                    # Post-related components
│   │   │   ├── PostCard.tsx
│   │   │   ├── CreatePostForm.tsx
│   │   │   ├── PostComments.tsx
│   │   │   └── PostReactions.tsx
│   │   ├── users/                    # User-related components
│   │   │   ├── UserProfile.tsx
│   │   │   ├── UserCard.tsx
│   │   │   └── UserSettings.tsx
│   │   └── layout/                   # Layout components
│   │       ├── MainLayout.tsx
│   │       └── AuthLayout.tsx
│   │
│   ├── pages/                        # Page-level components (route views)
│   │   ├── Home.tsx
│   │   ├── Login.tsx
│   │   ├── Register.tsx
│   │   ├── Profile.tsx
│   │   ├── Settings.tsx
│   │   ├── NotFound.tsx
│   │   └── ErrorBoundary.tsx
│   │
│   ├── hooks/                        # Custom React hooks
│   │   ├── useAuth.ts                # Authentication context hook
│   │   ├── usePost.ts                # Post data fetching
│   │   ├── useUser.ts                # User data fetching
│   │   ├── useApi.ts                 # Generic API hook
│   │   └── useFetch.ts               # HTTP request hook
│   │
│   ├── types/                        # TypeScript interfaces & types
│   │   ├── auth.types.ts
│   │   ├── post.types.ts
│   │   ├── user.types.ts
│   │   ├── api.types.ts
│   │   └── index.ts
│   │
│   ├── lib/                          # Utilities & helpers
│   │   ├── api/
│   │   │   ├── client.ts             # Axios instance with interceptors
│   │   │   ├── endpoints.ts          # API endpoint constants
│   │   │   └── auth.ts               # Auth-related API calls
│   │   ├── constants.ts              # App-wide constants
│   │   ├── utils.ts                  # General utility functions
│   │   └── validators.ts             # Form validation functions
│   │
│   ├── store/                        # Zustand state management
│   │   ├── authStore.ts              # Auth state (user, token)
│   │   ├── postStore.ts              # Posts state
│   │   ├── userStore.ts              # User profile state
│   │   └── uiStore.ts                # UI state (modals, notifications)
│   │
│   ├── styles/                       # Global styles
│   │   ├── globals.css               # Global CSS reset & utilities
│   │   ├── variables.css             # CSS custom properties
│   │   └── animations.css            # Keyframe animations
│   │
│   ├── App.tsx                       # Root component with routing
│   ├── main.tsx                      # Vite entry point
│   └── vite-env.d.ts                 # Vite type definitions
│
├── public/                           # Static assets (copied to dist/)
│   ├── favicon.ico
│   ├── logo.png
│   └── hero-bg.jpg
│
├── tests/                            # Test files (mirrors src/ structure)
│   ├── components/
│   ├── hooks/
│   └── pages/
│
├── .env.local                        # Local environment variables (gitignored)
├── .env.example                      # Environment variable template
├── index.html                        # HTML entry point
├── vite.config.ts                    # Vite configuration
├── tsconfig.json                     # TypeScript configuration
├── tailwind.config.ts                # Tailwind CSS configuration
├── postcss.config.mjs                # PostCSS configuration
├── package.json
├── package-lock.json
└── README.md
```

---

## ⚙️ Configuration Files

### `vite.config.ts`

```typescript
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'
import path from 'path'

export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
  server: {
    port: 5173,
    proxy: {
      '/api': {
        target: 'http://localhost:8787',
        changeOrigin: true,
        rewrite: (path) => path.replace(/^\/api/, ''),
      },
    },
  },
  build: {
    outDir: 'dist',
    sourcemap: false,
    minify: 'terser',
  },
})
```

### `tsconfig.json`

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "useDefineForClassFields": true,
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "skipLibCheck": true,
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx",
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"]
    }
  },
  "include": ["src"],
  "references": [{ "path": "./tsconfig.node.json" }]
}
```

### `tailwind.config.ts`

```typescript
import type { Config } from 'tailwindcss'

const config: Config = {
  content: [
    './index.html',
    './src/**/*.{js,ts,jsx,tsx}',
  ],
  theme: {
    extend: {
      colors: {
        primary: {
          50: '#f0f7ff',
          500: '#0066cc',
          600: '#0052a3',
          700: '#003d7a',
          900: '#001f3f',
        },
        secondary: {
          500: '#ff6b35',
          600: '#e55a24',
          700: '#cc4913',
        },
        accent: {
          500: '#2ecc71',
          600: '#27ae60',
        },
        warning: {
          500: '#ffc107',
          600: '#e0a800',
        },
      },
      fontFamily: {
        sans: ['Inter', 'sans-serif'],
        display: ['Poppins', 'sans-serif'],
      },
    },
  },
  plugins: [],
}

export default config
```

### `postcss.config.mjs`

```javascript
export default {
  plugins: {
    tailwindcss: {},
    autoprefixer: {},
  },
}
```

---

## 🔐 Environment Variables

Create a `.env.local` file in the `frontend/` directory (copy from `.env.example`):

```env
# API Configuration
VITE_API_BASE_URL=http://localhost:8787
VITE_API_TIMEOUT=10000

# Authentication
VITE_JWT_STORAGE_KEY=ucc_token
VITE_REFRESH_TOKEN_KEY=ucc_refresh_token

# Features
VITE_ENABLE_ANALYTICS=false
VITE_ENABLE_DEBUG_MODE=true

# Cloudflare (for production)
VITE_CF_ACCOUNT_ID=your_account_id
VITE_CF_PAGES_PROJECT=ucc-student-portal
```

**Important**: Never commit `.env.local` - it's gitignored.

---

## 📝 npm Scripts

```bash
# Development
npm run dev              # Start Vite dev server (port 5173)
npm run dev:debug       # Dev server with debug mode enabled
npm run preview         # Preview production build locally

# Building
npm run build           # Build for production
npm run build:report    # Build with size analysis

# Testing
npm run test            # Run Vitest tests
npm run test:ui         # Vitest UI dashboard
npm run test:coverage   # Coverage report

# Code Quality
npm run lint            # Run ESLint
npm run lint:fix        # Fix linting issues
npm run type-check      # TypeScript type checking
npm run format          # Format code with Prettier

# Deployment
npm run deploy:preview  # Deploy to Cloudflare Pages (preview)
npm run deploy:prod     # Deploy to Cloudflare Pages (production)
```

---

## 🔌 API Integration

### API Client Setup (`lib/api/client.ts`)

```typescript
import axios from 'axios'
import { useAuthStore } from '@/store/authStore'

const API_BASE_URL = import.meta.env.VITE_API_BASE_URL

export const apiClient = axios.create({
  baseURL: API_BASE_URL,
  timeout: import.meta.env.VITE_API_TIMEOUT,
  headers: {
    'Content-Type': 'application/json',
  },
})

// Request interceptor - add auth token
apiClient.interceptors.request.use((config) => {
  const { token } = useAuthStore.getState()
  if (token) {
    config.headers.Authorization = `Bearer ${token}`
  }
  return config
})

// Response interceptor - handle token refresh
apiClient.interceptors.response.use(
  (response) => response,
  async (error) => {
    if (error.response?.status === 401) {
      // Token expired - refresh or redirect to login
      const { logout } = useAuthStore.getState()
      logout()
    }
    return Promise.reject(error)
  }
)

export default apiClient
```

### Custom Hook (`hooks/useAuth.ts`)

```typescript
import { useCallback } from 'react'
import { useQuery, useMutation } from '@tanstack/react-query'
import { apiClient } from '@/lib/api/client'
import { useAuthStore } from '@/store/authStore'
import type { LoginCredentials, RegisterData, User } from '@/types'

export function useAuth() {
  const { token, setToken, setUser, logout } = useAuthStore()

  // Login mutation
  const loginMutation = useMutation({
    mutationFn: async (credentials: LoginCredentials) => {
      const response = await apiClient.post('/auth/login', credentials)
      return response.data
    },
    onSuccess: (data) => {
      setToken(data.token)
      setUser(data.user)
    },
  })

  // Register mutation
  const registerMutation = useMutation({
    mutationFn: async (data: RegisterData) => {
      const response = await apiClient.post('/auth/register', data)
      return response.data
    },
  })

  // Get current user
  const { data: user, isLoading } = useQuery({
    queryKey: ['currentUser'],
    queryFn: async () => {
      const response = await apiClient.get('/users/me')
      return response.data
    },
    enabled: !!token,
  })

  return {
    user,
    token,
    isLoading,
    login: loginMutation.mutate,
    register: registerMutation.mutate,
    logout,
    isAuthenticated: !!token,
  }
}
```

### Login Form Component (`components/auth/LoginForm.tsx`)

```typescript
import { useState } from 'react'
import { useAuth } from '@/hooks/useAuth'
import { useNavigate } from 'react-router-dom'

export function LoginForm() {
  const [email, setEmail] = useState('')
  const [password, setPassword] = useState('')
  const [error, setError] = useState('')
  const { login, isLoading } = useAuth()
  const navigate = useNavigate()

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault()
    setError('')

    login(
      { email, password },
      {
        onSuccess: () => navigate('/home'),
        onError: (err: any) => {
          setError(err.response?.data?.message || 'Login failed')
        },
      }
    )
  }

  return (
    <form onSubmit={handleSubmit} className="max-w-md mx-auto">
      <div className="mb-4">
        <label className="block text-sm font-medium text-gray-700">
          Email
        </label>
        <input
          type="email"
          value={email}
          onChange={(e) => setEmail(e.target.value)}
          className="mt-1 block w-full px-3 py-2 border border-gray-300 rounded-md"
          required
        />
      </div>

      <div className="mb-4">
        <label className="block text-sm font-medium text-gray-700">
          Password
        </label>
        <input
          type="password"
          value={password}
          onChange={(e) => setPassword(e.target.value)}
          className="mt-1 block w-full px-3 py-2 border border-gray-300 rounded-md"
          required
        />
      </div>

      {error && <div className="text-red-600 text-sm mb-4">{error}</div>}

      <button
        type="submit"
        disabled={isLoading}
        className="w-full bg-primary-500 text-white py-2 px-4 rounded-md disabled:opacity-50"
      >
        {isLoading ? 'Logging in...' : 'Login'}
      </button>
    </form>
  )
}
```

---

## 🧪 Testing Setup

### Vitest Configuration (`vitest.config.ts`)

```typescript
import { defineConfig } from 'vitest/config'
import react from '@vitejs/plugin-react'
import path from 'path'

export default defineConfig({
  plugins: [react()],
  test: {
    globals: true,
    environment: 'jsdom',
    setupFiles: ['./tests/setup.ts'],
  },
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
})
```

### Test Example (`tests/components/LoginForm.test.tsx`)

```typescript
import { render, screen, fireEvent, waitFor } from '@testing-library/react'
import { LoginForm } from '@/components/auth/LoginForm'
import { vi } from 'vitest'

describe('LoginForm', () => {
  it('submits form with valid credentials', async () => {
    const mockLogin = vi.fn()
    
    render(<LoginForm />)

    fireEvent.change(screen.getByLabelText(/email/i), {
      target: { value: 'user@example.com' },
    })
    fireEvent.change(screen.getByLabelText(/password/i), {
      target: { value: 'password123' },
    })

    fireEvent.click(screen.getByRole('button', { name: /login/i }))

    await waitFor(() => {
      expect(mockLogin).toHaveBeenCalled()
    })
  })
})
```

---

## 📱 Mobile-First Design

All components use Tailwind's responsive prefixes for mobile-first design:

```typescript
// Mobile-first responsive example
export function Hero() {
  return (
    <section className="px-4 py-12 sm:px-6 md:px-8 lg:px-12">
      <h1 className="text-2xl sm:text-3xl md:text-4xl lg:text-5xl font-bold">
        Welcome to UCC Student Portal
      </h1>
      <p className="mt-4 text-sm sm:text-base md:text-lg text-gray-600">
        Connect, share, and grow with your campus community
      </p>
    </section>
  )
}
```

### Responsive Breakpoints (Tailwind)

- **sm**: 640px (tablets)
- **md**: 768px (small laptops)
- **lg**: 1024px (laptops)
- **xl**: 1280px (desktops)
- **2xl**: 1536px (large displays)

---

## ♿ Accessibility Guidelines

### WCAG 2.1 Level AA Compliance

**Color Contrast**: Minimum 4.5:1 for text
```typescript
// ✅ Good - sufficient contrast
<button className="bg-primary-500 text-white">Submit</button>

// ❌ Bad - insufficient contrast
<button className="bg-gray-100 text-gray-200">Submit</button>
```

**ARIA Labels**: Provide context for screen readers
```typescript
export function SearchInput() {
  return (
    <input
      type="search"
      placeholder="Search posts"
      aria-label="Search posts by keywords or authors"
      aria-describedby="search-help"
    />
  )
}
```

**Keyboard Navigation**: All interactive elements must be keyboard accessible
```typescript
// ✅ Native button (built-in keyboard support)
<button onClick={handleClick}>Submit</button>

// ✅ Custom element with keyboard support
<div
  role="button"
  onClick={handleClick}
  onKeyDown={(e) => {
    if (e.key === 'Enter' || e.key === ' ') handleClick()
  }}
  tabIndex={0}
>
  Submit
</div>
```

**Semantic HTML**: Use semantic elements instead of divs

```typescript
// ✅ Good
<header>...</header>
<nav>...</nav>
<main>...</main>
<footer>...</footer>

// ❌ Bad
<div className="header">...</div>
<div className="nav">...</div>
```

---

## 🚀 Deployment

### Deploy to Cloudflare Pages

```bash
# Build production bundle
npm run build

# Deploy to Cloudflare Pages
npm run deploy:prod
```

### Build Output
- **Location**: `dist/` folder
- **Size**: Typically < 150KB gzipped
- **Compatibility**: All modern browsers (ES2020)

### Environment Setup for Production

Create `.env.production.local`:
```env
VITE_API_BASE_URL=https://api.ucc-student-portal.com
VITE_ENABLE_DEBUG_MODE=false
VITE_CF_PAGES_PROJECT=ucc-student-portal
```

### Performance Optimization

- **Code Splitting**: Vite automatic route-based splitting
- **Lazy Loading**: React.lazy() for route components
- **Image Optimization**: Compress all images < 100KB
- **Cache Strategy**: Cloudflare edge caching with revalidation

---

## 📚 Additional Resources

- [Vite Documentation](https://vitejs.dev)
- [React 18 Documentation](https://react.dev)
- [React Router Guide](https://reactrouter.com)
- [Zustand Documentation](https://github.com/pmndrs/zustand)
- [TanStack Query](https://tanstack.com/query)
- [Tailwind CSS](https://tailwindcss.com)
- [TypeScript Handbook](https://www.typescriptlang.org/docs)

---

**Questions?** Refer to [API_SPECIFICATION.md](./API_SPECIFICATION.md) for backend endpoints or [DOCUMENTATION_INDEX.md](./DOCUMENTATION_INDEX.md) for other guides.
