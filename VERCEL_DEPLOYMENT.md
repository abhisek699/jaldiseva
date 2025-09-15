# JALDISEVA Vercel Deployment Guide

This guide provides step-by-step instructions for deploying the JALDISEVA telemedicine platform to Vercel with optimal configuration.

## 🚀 Deployment Architecture

### Frontend (Vercel)
- **Next.js Frontend**: Deployed on Vercel with automatic builds
- **Static Assets**: CDN-optimized delivery
- **Environment Variables**: Secure configuration management

### Backend Options
- **Option 1**: Deploy backend to Railway/Render (Recommended)
- **Option 2**: Use Vercel Serverless Functions (Limited features)
- **Option 3**: External VPS with Vercel frontend

## 📋 Prerequisites

- [ ] GitHub repository with JALDISEVA code
- [ ] Vercel account (free tier available)
- [ ] Database hosting (Supabase, Neon, or Railway PostgreSQL)
- [ ] Google Gemini API key
- [ ] Domain name (optional)

## 🎯 Quick Deployment (Recommended)

### Step 1: Prepare Repository

1. **Push to GitHub**
   ```bash
   git add .
   git commit -m "Prepare for Vercel deployment"
   git push origin main
   ```

2. **Create Vercel Configuration**
   ```bash
   # In project root
   touch vercel.json
   ```

### Step 2: Frontend Deployment to Vercel

1. **Visit Vercel Dashboard**
   - Go to [vercel.com](https://vercel.com)
   - Sign in with GitHub account
   - Click "New Project"

2. **Import Repository**
   - Select your JALDISEVA repository
   - Choose "Frontend" folder as root directory
   - Framework: Next.js (auto-detected)

3. **Configure Build Settings**
   ```json
   {
     "buildCommand": "npm run build",
     "outputDirectory": ".next",
     "installCommand": "npm install"
   }
   ```

4. **Environment Variables**
   Add these in Vercel dashboard:
   ```
   NEXT_PUBLIC_API_URL=https://your-backend-url.com/api
   NEXT_PUBLIC_SOCKET_URL=https://your-backend-url.com
   NEXT_PUBLIC_APP_NAME=JALDISEVA
   ```

5. **Deploy**
   - Click "Deploy"
   - Wait for build completion
   - Your frontend will be live at `https://your-project.vercel.app`

### Step 3: Backend Deployment Options

#### Option A: Railway (Recommended)

1. **Create Railway Account**
   - Visit [railway.app](https://railway.app)
   - Connect GitHub account

2. **Deploy Backend**
   - Click "New Project"
   - Select "Deploy from GitHub repo"
   - Choose your repository
   - Select "backend" folder

3. **Configure Environment Variables**
   ```env
   NODE_ENV=production
   DATABASE_URL=postgresql://user:pass@host:port/db
   JWT_SECRET=your-jwt-secret
   GEMINI_API_KEY=AIzaSyCs-pxODTsSkxMWTW2GCtbzD2ichb4BFnU
   FRONTEND_URL=https://your-vercel-app.vercel.app
   PORT=3001
   ```

4. **Configure Build**
   ```json
   {
     "build": "npm install",
     "start": "node server.js"
   }
   ```

#### Option B: Render

1. **Create Render Account**
   - Visit [render.com](https://render.com)
   - Connect GitHub

2. **Create Web Service**
   - New → Web Service
   - Connect repository
   - Root Directory: `backend`
   - Build Command: `npm install`
   - Start Command: `node server.js`

3. **Environment Variables**
   Same as Railway configuration above

#### Option C: Vercel Serverless Functions

1. **Create API Routes**
   ```bash
   mkdir -p frontend/pages/api
   ```

2. **Move Backend Logic**
   ```javascript
   // frontend/pages/api/auth/login.js
   export default async function handler(req, res) {
     // Your auth logic here
   }
   ```

   **Note**: This approach has limitations for real-time features and file uploads.

## 🔧 Configuration Files

### Frontend Vercel Configuration

Create `frontend/vercel.json`:
```json
{
  "framework": "nextjs",
  "buildCommand": "npm run build",
  "devCommand": "npm run dev",
  "installCommand": "npm install",
  "functions": {
    "pages/api/**/*.js": {
      "maxDuration": 30
    }
  },
  "rewrites": [
    {
      "source": "/api/:path*",
      "destination": "https://your-backend-url.com/api/:path*"
    }
  ],
  "headers": [
    {
      "source": "/(.*)",
      "headers": [
        {
          "key": "X-Frame-Options",
          "value": "DENY"
        },
        {
          "key": "X-Content-Type-Options",
          "value": "nosniff"
        }
      ]
    }
  ]
}
```

### Next.js Configuration

Update `frontend/next.config.js`:
```javascript
/** @type {import('next').NextConfig} */
const nextConfig = {
  output: 'standalone',
  images: {
    domains: ['your-backend-domain.com'],
    unoptimized: true
  },
  async rewrites() {
    return [
      {
        source: '/api/:path*',
        destination: `${process.env.NEXT_PUBLIC_API_URL}/:path*`
      }
    ]
  },
  async headers() {
    return [
      {
        source: '/(.*)',
        headers: [
          {
            key: 'X-Frame-Options',
            value: 'DENY'
          },
          {
            key: 'X-Content-Type-Options',
            value: 'nosniff'
          },
          {
            key: 'Referrer-Policy',
            value: 'origin-when-cross-origin'
          }
        ]
      }
    ]
  }
}

module.exports = nextConfig
```

## 🗄️ Database Setup

### Option 1: Supabase (Recommended)

1. **Create Supabase Project**
   - Visit [supabase.com](https://supabase.com)
   - Create new project
   - Note connection details

2. **Import Schema**
   ```sql
   -- Run your schema.sql in Supabase SQL editor
   ```

3. **Get Connection String**
   ```
   postgresql://postgres:password@host:5432/postgres
   ```

### Option 2: Neon

1. **Create Neon Account**
   - Visit [neon.tech](https://neon.tech)
   - Create database

2. **Configure Connection**
   ```env
   DATABASE_URL=postgresql://user:pass@host/dbname?sslmode=require
   ```

### Option 3: Railway PostgreSQL

1. **Add PostgreSQL Plugin**
   - In Railway dashboard
   - Add PostgreSQL plugin to your project

2. **Use Generated URL**
   ```env
   DATABASE_URL=${{Postgres.DATABASE_URL}}
   ```

## 🔐 Environment Variables Setup

### Vercel Frontend Environment Variables

In Vercel Dashboard → Project → Settings → Environment Variables:

```env
# API Configuration
NEXT_PUBLIC_API_URL=https://your-backend.railway.app/api
NEXT_PUBLIC_SOCKET_URL=https://your-backend.railway.app
NEXT_PUBLIC_APP_NAME=JALDISEVA

# Optional: Analytics
NEXT_PUBLIC_GOOGLE_ANALYTICS=G-XXXXXXXXXX
```

### Backend Environment Variables

In Railway/Render Dashboard:

```env
# Production Configuration
NODE_ENV=production

# Database
DATABASE_URL=postgresql://user:pass@host:port/db
DB_HOST=host
DB_PORT=5432
DB_NAME=jaldiseva
DB_USER=user
DB_PASSWORD=password

# JWT
JWT_SECRET=your-super-secret-jwt-key-here
JWT_EXPIRES_IN=7d

# AI Configuration
GEMINI_API_KEY=AIzaSyCs-pxODTsSkxMWTW2GCtbzD2ichb4BFnU

# Server
PORT=3001
FRONTEND_URL=https://your-vercel-app.vercel.app

# File Upload
MAX_FILE_SIZE=10485760
UPLOAD_PATH=./uploads

# Email (Optional)
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=your-email@gmail.com
SMTP_PASS=your-app-password
```

## 🚀 Deployment Steps

### Complete Deployment Process

1. **Prepare Code**
   ```bash
   # Ensure all changes are committed
   git add .
   git commit -m "Ready for production deployment"
   git push origin main
   ```

2. **Deploy Backend First**
   - Choose Railway, Render, or VPS
   - Configure environment variables
   - Deploy and note the URL

3. **Deploy Frontend**
   - Import to Vercel
   - Set environment variables with backend URL
   - Deploy

4. **Configure Domain (Optional)**
   - Add custom domain in Vercel
   - Update DNS records
   - Enable SSL (automatic)

5. **Test Deployment**
   - Test all features
   - Check API connectivity
   - Verify real-time features

## 🔧 Post-Deployment Configuration

### Custom Domain Setup

1. **Add Domain to Vercel**
   - Project Settings → Domains
   - Add your domain
   - Follow DNS configuration

2. **Update Environment Variables**
   ```env
   FRONTEND_URL=https://yourdomain.com
   ```

### SSL and Security

1. **Automatic SSL**
   - Vercel provides automatic SSL
   - No additional configuration needed

2. **Security Headers**
   - Already configured in `next.config.js`
   - Additional security via Vercel dashboard

### Performance Optimization

1. **Enable Analytics**
   ```bash
   npm install @vercel/analytics
   ```

2. **Configure Caching**
   ```javascript
   // In next.config.js
   async headers() {
     return [
       {
         source: '/static/(.*)',
         headers: [
           {
             key: 'Cache-Control',
             value: 'public, max-age=31536000, immutable'
           }
         ]
       }
     ]
   }
   ```

## 🐛 Troubleshooting

### Common Issues

#### Build Failures
```bash
# Check build logs in Vercel dashboard
# Common fixes:
npm install --legacy-peer-deps
npm run build --verbose
```

#### API Connection Issues
```javascript
// Check CORS configuration in backend
app.use(cors({
  origin: process.env.FRONTEND_URL,
  credentials: true
}));
```

#### Environment Variable Issues
```bash
# Ensure all required variables are set
# Check variable names match exactly
# Restart deployments after changes
```

### Performance Issues

1. **Optimize Images**
   ```javascript
   // Use Next.js Image component
   import Image from 'next/image'
   ```

2. **Code Splitting**
   ```javascript
   // Use dynamic imports
   const Component = dynamic(() => import('./Component'))
   ```

## 📊 Monitoring and Analytics

### Vercel Analytics

1. **Enable Analytics**
   ```bash
   npm install @vercel/analytics
   ```

2. **Add to App**
   ```javascript
   // pages/_app.js
   import { Analytics } from '@vercel/analytics/react'
   
   export default function App({ Component, pageProps }) {
     return (
       <>
         <Component {...pageProps} />
         <Analytics />
       </>
     )
   }
   ```

### Error Monitoring

1. **Vercel Speed Insights**
   ```bash
   npm install @vercel/speed-insights
   ```

2. **Custom Error Tracking**
   ```javascript
   // Add error boundary components
   // Log errors to external service
   ```

## 💰 Cost Optimization

### Vercel Pricing
- **Hobby Plan**: Free for personal projects
- **Pro Plan**: $20/month for teams
- **Enterprise**: Custom pricing

### Backend Hosting Costs
- **Railway**: $5/month for 512MB RAM
- **Render**: $7/month for 512MB RAM
- **VPS**: $5-20/month depending on specs

### Database Costs
- **Supabase**: Free tier + $25/month for pro
- **Neon**: Free tier + usage-based pricing
- **Railway PostgreSQL**: $5/month

## 🔄 CI/CD Pipeline

### Automatic Deployments

1. **Vercel GitHub Integration**
   - Automatic deployments on push
   - Preview deployments for PRs
   - Production deployment on main branch

2. **Environment Branches**
   ```bash
   # Development
   git push origin develop → preview deployment
   
   # Production
   git push origin main → production deployment
   ```

### Build Optimization

1. **Vercel Build Cache**
   - Automatic dependency caching
   - Incremental builds
   - Fast rebuilds

2. **Custom Build Commands**
   ```json
   {
     "scripts": {
       "build": "next build",
       "start": "next start",
       "vercel-build": "npm run build"
     }
   }
   ```

## 📱 Mobile Optimization

### PWA Configuration

1. **Service Worker**
   ```javascript
   // Already configured in Next.js
   // Automatic PWA features
   ```

2. **App Manifest**
   ```json
   // public/manifest.json
   {
     "name": "JALDISEVA",
     "short_name": "JALDISEVA",
     "description": "AI-Powered Telemedicine Platform",
     "start_url": "/",
     "display": "standalone",
     "theme_color": "#0070f3",
     "background_color": "#ffffff"
   }
   ```

## 🎉 Go Live Checklist

### Pre-Launch
- [ ] All environment variables configured
- [ ] Database schema imported
- [ ] SSL certificates active
- [ ] Custom domain configured (if applicable)
- [ ] All features tested in production

### Launch
- [ ] Deploy backend to production
- [ ] Deploy frontend to Vercel
- [ ] Update DNS records
- [ ] Test all user flows
- [ ] Monitor error logs

### Post-Launch
- [ ] Set up monitoring and alerts
- [ ] Configure backup strategies
- [ ] Document deployment process
- [ ] Train team on deployment workflow

---

## 🚀 Quick Start Commands

```bash
# 1. Deploy Backend (Railway)
railway login
railway new
railway add postgresql
railway deploy

# 2. Deploy Frontend (Vercel)
npm install -g vercel
vercel login
vercel --prod

# 3. Configure Environment Variables
vercel env add NEXT_PUBLIC_API_URL
vercel env add NEXT_PUBLIC_SOCKET_URL
vercel env add NEXT_PUBLIC_APP_NAME

# 4. Redeploy with Environment Variables
vercel --prod
```

Your JALDISEVA platform is now live on Vercel! 🎉

**Frontend URL**: `https://your-project.vercel.app`  
**Backend URL**: `https://your-backend.railway.app`

For support and updates, refer to the [main documentation](./README.md) and [deployment guide](./DEPLOYMENT.md).
