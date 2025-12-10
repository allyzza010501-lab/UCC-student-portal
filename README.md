# UCC Student Portal

**University of Caloocan Campus Student Portal** - A modern, elegant student community platform built with cutting-edge web technologies.

## 🎓 About

The UCC Student Portal is a Facebook-like platform exclusively for University of Caloocan Campus students. It enables students to:
- **Share Posts**: Express your thoughts and experiences
- **Join Discussions**: Participate in topics that matter to your campus
- **React & Comment**: Engage with your community
- **Connect**: Build relationships with fellow students

## 🚀 Tech Stack

- **Frontend Framework**: Next.js 16 with TypeScript
- **Styling**: Tailwind CSS
- **Hosting**: Cloudflare Pages
- **Database**: Cloudflare D1
- **File Storage**: Cloudflare R2
- **Edge Compute**: Cloudflare Workers

## 📋 Project Structure

```
src/
├── app/          # Next.js App Router
├── components/   # Reusable React components
├── styles/       # Global styles and Tailwind config
└── lib/          # Utility functions
```

## 🛠️ Getting Started

### Prerequisites
- Node.js 18+ 
- npm or yarn

### Installation

```bash
# Clone the repository
git clone https://github.com/allyzza010501-lab/UCC-student-portal.git
cd UCC-Student-Portal

# Install dependencies
npm install

# Run development server
npm run dev
```

Open [http://localhost:3000](http://localhost:3000) to view in browser.

### Build for Production

```bash
npm run build
npm start
```

## 📝 MVP Scope

**Phase 1: Landing Page**
- Hero section with CTA
- Features showcase
- Community stats
- Student testimonials
- Navigation & Footer
- Mobile responsive design

## 🎨 Design Philosophy

- **Elegant & Branded**: Reflect UCC's identity
- **Modern UI**: Clean, contemporary design
- **Accessible**: WCAG compliant
- **Performance-First**: Fast load times via Cloudflare

## 📦 Deployment

Hosted on **Cloudflare Pages** for fast, global distribution:

```bash
# Deploy to Cloudflare Pages
npm run build
npx wrangler pages deploy
```

## 📄 License

Private - University of Caloocan Campus

## 👤 Author

**Allyzza** - Project Lead & Designer

---

**Let's build something amazing for the UCC community! 🎉**
