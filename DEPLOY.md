# Deployment Guide for Vercel

## Prerequisites
- Vercel account (sign up at https://vercel.com)
- Git repository (GitHub, GitLab, or Bitbucket)

## Database Setup Options

### Option 1: Vercel Postgres (Recommended)
1. Install Vercel Postgres package:
   ```bash
   npm install @vercel/postgres
   ```

2. Update `prisma/schema.prisma`:
   ```prisma
   datasource db {
     provider = "postgresql"
     url      = env("POSTGRES_PRISMA_URL")
   }
   ```

3. In Vercel Dashboard:
   - Go to Storage → Create Database → Postgres
   - Copy the connection strings to your environment variables

### Option 2: Turso (Serverless SQLite)
1. Install Turso:
   ```bash
   npm install @libsql/client
   ```

2. Update `prisma/schema.prisma`:
   ```prisma
   datasource db {
     provider = "sqlite"
     url      = env("DATABASE_URL")
   }
   ```

3. Sign up at https://turso.tech and create a database
4. Get your database URL and auth token

### Option 3: External PostgreSQL (Supabase, Neon, etc.)
1. Create a database at your preferred provider
2. Update `prisma/schema.prisma` to use PostgreSQL
3. Copy the connection URL

## Deployment Steps

### Method 1: Deploy via Vercel CLI (Current Directory)

1. **Login to Vercel**
   ```bash
   vercel login
   ```

2. **Deploy to Preview**
   ```bash
   vercel
   ```
   - Follow the prompts
   - Select your scope/team
   - Link to existing project or create new
   - Accept default settings

3. **Set Environment Variables**
   ```bash
   vercel env add DATABASE_URL production
   vercel env add NEXTAUTH_URL production
   vercel env add NEXTAUTH_SECRET production
   ```

4. **Deploy to Production**
   ```bash
   vercel --prod
   ```

### Method 2: Deploy via Vercel Dashboard (Git Integration)

1. **Push to Git**
   ```bash
   git add .
   git commit -m "Prepare for Vercel deployment"
   git push origin main
   ```

2. **Import to Vercel**
   - Go to https://vercel.com/new
   - Import your Git repository
   - Configure project:
     - Framework Preset: Next.js
     - Build Command: `npm run build`
     - Output Directory: `.next`
     - Install Command: `npm install`

3. **Add Environment Variables** in Vercel Dashboard:
   ```
   DATABASE_URL=your-database-url
   NEXTAUTH_URL=https://your-app.vercel.app
   NEXTAUTH_SECRET=your-generated-secret
   NEXT_PUBLIC_APP_URL=https://your-app.vercel.app
   ```

4. **Deploy**
   - Click "Deploy"
   - Wait for build to complete

## Generate NEXTAUTH_SECRET

Run this command to generate a secure secret:
```bash
openssl rand -base64 32
```

Or use:
```bash
node -e "console.log(require('crypto').randomBytes(32).toString('base64'))"
```

## Post-Deployment

### Run Database Migrations
After deployment, run migrations:
```bash
vercel env pull .env.production.local
npx prisma migrate deploy
```

Or set up in your build command in `package.json`:
```json
"scripts": {
  "build": "prisma generate && prisma migrate deploy && next build"
}
```

### Verify Deployment
1. Check build logs in Vercel Dashboard
2. Test your app at the provided URL
3. Check environment variables are set correctly

## Troubleshooting

### Build Failures
- Check build logs in Vercel Dashboard
- Ensure all dependencies are in `dependencies` (not `devDependencies`)
- Verify environment variables are set

### Database Connection Issues
- Verify DATABASE_URL is correct
- Check database is accessible from Vercel's servers
- For Postgres, use connection pooling URL

### NextAuth Issues
- Ensure NEXTAUTH_URL matches your deployment URL
- NEXTAUTH_SECRET must be set and secure
- Check callback URLs in OAuth providers

## Custom Domain

1. Go to Vercel Dashboard → Your Project → Settings → Domains
2. Add your custom domain
3. Follow DNS configuration instructions
4. Update NEXTAUTH_URL to use custom domain

## Continuous Deployment

Once connected to Git, Vercel automatically:
- Deploys every push to `main` as production
- Creates preview deployments for pull requests
- Runs builds and tests before deploying
