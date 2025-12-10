# UCC Student Portal

**University of Caloocan Campus Student Portal** - A modern, elegant student community platform built with cutting-edge web technologies.

## 🎓 About

The UCC Student Portal is a Facebook-like platform exclusively for University of Caloocan Campus students. It enables students to:
- **Share Posts**: Express your thoughts and experiences
- **Join Discussions**: Participate in topics that matter to your campus
- **React & Comment**: Engage with your community
- **Connect**: Build relationships with fellow students

## 🚀 Tech Stack

### **Frontend** (SPA - Single Page Application)
- **Framework**: Vite 5 + React 18
- **Language**: TypeScript 5
- **Styling**: Tailwind CSS 4
- **Routing**: React Router v6
- **State Management**: Zustand
- **Data Fetching**: TanStack Query v5
- **Build**: Vite (instant HMR, optimized builds)

### **Backend** (Serverless API)
- **Edge Compute**: Cloudflare Workers
- **Database**: Cloudflare D1 (SQLite)
- **File Storage**: Cloudflare R2 (S3-compatible)
- **Authentication**: JWT (custom implementation)

### **Deployment**
- **Frontend Hosting**: Cloudflare Pages (static)
- **API Hosting**: Cloudflare Workers
- **CDN**: Cloudflare global edge network

## 📁 Project Structure

```
UCC-Student-Portal/
├── frontend/                      # Vite + React SPA
│   ├── src/
│   │   ├── components/           # Reusable React components
│   │   ├── pages/                # Page components (route views)
│   │   ├── hooks/                # Custom React hooks (useAuth, usePost, etc.)
│   │   ├── types/                # TypeScript interfaces & types
│   │   ├── lib/                  # Utilities, helpers, constants
│   │   ├── store/                # Zustand state management
│   │   ├── styles/               # Global CSS & Tailwind config
│   │   ├── App.tsx               # Root component with routing
│   │   └── main.tsx              # Vite entry point
│   ├── public/                   # Static assets
│   ├── package.json
│   ├── vite.config.ts
│   ├── tsconfig.json
│   ├── tailwind.config.ts
│   └── index.html
├── backend/                       # Cloudflare Workers API
│   ├── src/
│   │   ├── api/                  # Route handlers (/auth, /users, /posts)
│   │   ├── middleware/           # Auth middleware, error handling
│   │   ├── types/                # TypeScript interfaces
│   │   └── worker.ts             # Worker entry point
│   ├── wrangler.toml             # Cloudflare Workers config
│   ├── package.json
│   └── migrations/               # D1 database migrations
└── [documentation files]          # API specs, architecture, setup guides
```

## 🛠️ Getting Started

### Prerequisites
- Node.js 18+
- npm or yarn
- Cloudflare account (for backend/deployment)

### Installation

This is a monorepo with separate frontend and backend packages.

**Terminal 1 - Frontend (Vite + React)**
```bash
# Clone the repository
git clone https://github.com/allyzza010501-lab/UCC-student-portal.git
cd UCC-Student-Portal/frontend

# Install dependencies
npm install

# Run development server
npm run dev
```
Frontend runs on [http://localhost:5173](http://localhost:5173)

**Terminal 2 - Backend (Cloudflare Workers)**
```bash
cd UCC-Student-Portal/backend

# Install dependencies
npm install

# Run Cloudflare Workers development server
npm run dev
```
Backend API runs on [http://localhost:8787](http://localhost:8787)

### Build for Production

**Frontend**
```bash
cd frontend
npm run build
# Creates dist/ folder for deployment to Cloudflare Pages
```

**Backend**
```bash
cd backend
npm run build
# Bundles Worker code for Cloudflare deployment
```

## 📝 MVP Scope & Timeline

**Week 1-2**: Design System & Components
- Establish Tailwind design tokens
- Build reusable component library
- Accessibility testing

**Week 3-4**: Landing Page
- Hero section with CTA
- Features showcase
- Community stats
- Mobile responsive design

**Week 5-6**: Core Features
- User authentication (registration, login)
- Create/read/delete posts
- Comments and reactions
- User profiles

**Week 7-8**: Polish & Launch
- Performance optimization
- Security hardening
- Beta testing
- Production deployment

## 🎨 Design Philosophy

- **Elegant & Branded**: Reflect UCC's identity
- **Modern UI**: Clean, contemporary design
- **Accessible**: WCAG compliant
- **Performance-First**: Fast load times via Cloudflare

## 📦 Deployment

### Frontend Deployment (Cloudflare Pages)
```bash
cd frontend
npm run build
npx wrangler pages deploy dist/
```

### Backend Deployment (Cloudflare Workers)
```bash
cd backend
npm run deploy
# Authenticates with Cloudflare and deploys worker
```

### Environment Configuration
See [FRONTEND_SETUP.md](./FRONTEND_SETUP.md) and [BACKEND_SETUP.md](./BACKEND_SETUP.md) for detailed setup instructions and configuration.

## 📄 License

Private - University of Caloocan Campus

## 👤 Author

**Allyzza** - Project Lead & Designer

---

**Let's build something amazing for the UCC community! 🎉**
