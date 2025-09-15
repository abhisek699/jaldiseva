# 🚀 JALDISEVA Supabase + Vercel Deployment Guide (Option C)

This guide will help you deploy JALDISEVA using **Supabase** for the database and backend services, and **Vercel** for the frontend and serverless functions.

## 📋 Prerequisites

- Node.js 18+ installed
- Git installed
- Vercel CLI installed (`npm install -g vercel`)
- Supabase account
- Google Gemini API key: `AIzaSyCs-pxODTsSkxMWTW2GCtbzD2ichb4BFnU`

## 🗄️ Step 1: Set Up Supabase Database

### 1.1 Create Supabase Project
1. Go to [supabase.com](https://supabase.com)
2. Click "Start your project"
3. Create a new organization (if needed)
4. Click "New Project"
5. Choose a name: `jaldiseva-db`
6. Set a strong database password
7. Choose region closest to your users
8. Click "Create new project"

### 1.2 Set Up Database Schema
1. Wait for project to be ready (2-3 minutes)
2. Go to **SQL Editor** in Supabase dashboard
3. Copy the contents of `supabase-setup.sql` 
4. Paste and run the SQL script
5. Verify tables are created in **Table Editor**

### 1.3 Configure Authentication
1. Go to **Authentication** → **Settings**
2. Disable email confirmations for development:
   - Turn OFF "Enable email confirmations"
   - Turn OFF "Enable phone confirmations"
3. Go to **Authentication** → **URL Configuration**
4. Add your Vercel domain when ready

### 1.4 Get Supabase Credentials
1. Go to **Settings** → **API**
2. Copy these values:
   - `Project URL`
   - `anon public` key
   - `service_role` key (keep this secret!)

## 🌐 Step 2: Deploy Frontend to Vercel

### 2.1 Prepare Frontend
```bash
cd frontend
npm install
```

### 2.2 Set Environment Variables
Create `.env.local` in frontend folder:
```env
NEXT_PUBLIC_SUPABASE_URL=your_supabase_project_url
NEXT_PUBLIC_SUPABASE_ANON_KEY=your_supabase_anon_key
SUPABASE_SERVICE_ROLE_KEY=your_supabase_service_role_key
GEMINI_API_KEY=AIzaSyCs-pxODTsSkxMWTW2GCtbzD2ichb4BFnU
JWT_SECRET=your_super_secret_jwt_key_here
JWT_EXPIRES_IN=7d
```

### 2.3 Deploy to Vercel
```bash
# Login to Vercel
vercel login

# Deploy
vercel --prod

# Follow prompts:
# - Link to existing project? No
# - Project name: jaldiseva
# - Directory: ./
# - Override settings? No
```

### 2.4 Configure Vercel Environment Variables
1. Go to Vercel dashboard → Your project → Settings → Environment Variables
2. Add all the environment variables from `.env.local`
3. Make sure to set them for **Production**, **Preview**, and **Development**

## ⚙️ Step 3: Configure Serverless Functions

### 3.1 Verify API Routes
The following serverless functions are now available:
- `/api/auth/login` - User authentication
- `/api/auth/register` - User registration  
- `/api/ai/symptom-check` - AI symptom analysis
- `/api/queue/join` - Join doctor queue

### 3.2 Test API Endpoints
```bash
# Test login endpoint
curl -X POST https://your-vercel-app.vercel.app/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","password":"password123"}'
```

## 🔧 Step 4: Update Frontend Configuration

### 4.1 Update API Client
The frontend will now use relative paths for API calls:
- `/api/auth/login` instead of external backend URL
- All API routes are now serverless functions

### 4.2 Configure Real-time Features
For real-time features (Socket.IO), you have two options:

**Option A: Use Supabase Realtime**
```javascript
// In your components
import { supabase } from '@/lib/supabase'

// Subscribe to queue changes
supabase
  .channel('queue_changes')
  .on('postgres_changes', {
    event: '*',
    schema: 'public',
    table: 'queue'
  }, (payload) => {
    console.log('Queue updated:', payload)
  })
  .subscribe()
```

**Option B: Deploy Socket.IO server separately**
- Use Railway/Render for Socket.IO server
- Keep existing Socket.IO implementation

## 📊 Step 5: Database Management

### 5.1 Row Level Security (RLS)
Supabase has RLS enabled. Update policies as needed:
```sql
-- Example: Allow users to read their own data
CREATE POLICY "Users can read own profile" 
ON users FOR SELECT 
USING (auth.uid()::text = user_id::text);
```

### 5.2 Database Backups
- Supabase automatically backs up your database
- Pro plan includes point-in-time recovery
- Free tier includes daily backups

## 🚀 Step 6: Go Live!

### 6.1 Final Checklist
- [ ] Supabase database is set up and populated
- [ ] All environment variables are configured in Vercel
- [ ] Frontend is deployed and accessible
- [ ] API endpoints are working
- [ ] Authentication is functional
- [ ] AI features are working with Gemini API

### 6.2 Custom Domain (Optional)
1. In Vercel dashboard → Domains
2. Add your custom domain
3. Configure DNS records
4. Update Supabase auth settings with new domain

## 💰 Cost Breakdown

### Free Tier Limits:
- **Vercel**: 100GB bandwidth, unlimited static deployments
- **Supabase**: 500MB database, 2GB bandwidth, 50MB file storage
- **Google Gemini**: Generous free tier for API calls

### Paid Plans (when you scale):
- **Vercel Pro**: $20/month (team features, analytics)
- **Supabase Pro**: $25/month (8GB database, 250GB bandwidth)
- **Total**: ~$45/month for production-ready setup

## 🔍 Monitoring & Analytics

### 6.1 Vercel Analytics
- Built-in performance monitoring
- Real-time function logs
- Error tracking

### 6.2 Supabase Monitoring
- Database performance metrics
- API usage statistics
- Real-time dashboard

## 🛠️ Troubleshooting

### Common Issues:

**1. CORS Errors**
- Check Supabase RLS policies
- Verify environment variables
- Ensure proper headers in API routes

**2. Authentication Issues**
- Verify JWT_SECRET is the same everywhere
- Check Supabase auth settings
- Ensure user exists in database

**3. Database Connection Issues**
- Verify Supabase URL and keys
- Check if database is paused (free tier)
- Review connection limits

**4. Serverless Function Timeouts**
- Optimize database queries
- Add proper indexes
- Consider caching strategies

### Getting Help:
- Vercel Discord: [vercel.com/discord](https://vercel.com/discord)
- Supabase Discord: [discord.supabase.com](https://discord.supabase.com)
- GitHub Issues: Create issues in your repository

## 🎉 Success!

Your JALDISEVA telemedicine platform is now live with:
- ✅ Serverless architecture on Vercel
- ✅ Managed PostgreSQL database on Supabase
- ✅ AI-powered features with Google Gemini
- ✅ Scalable and cost-effective deployment
- ✅ Built-in security and authentication
- ✅ Real-time capabilities
- ✅ Global CDN and edge functions

**Live URLs:**
- Frontend: `https://your-app.vercel.app`
- API: `https://your-app.vercel.app/api/*`
- Database: Managed by Supabase
- Admin Panel: `https://app.supabase.com/project/your-project`

Your platform is ready to serve patients and doctors across India! 🇮🇳
