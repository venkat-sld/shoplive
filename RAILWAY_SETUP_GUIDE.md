# Railway Setup Guide - Monorepo Deployment

This guide explains how to properly deploy the Live Sales Platform monorepo on Railway.

## 🎯 Overview

Your project is a **monorepo** with multiple services:
- **Backend**: Express.js API with PostgreSQL
- **Frontend**: React + Vite web application
- **Database**: PostgreSQL (managed by Railway)

Railway automatically detects services in monorepos by looking for `package.json` files in subdirectories.

## 📋 Prerequisites

1. **GitHub Account** - Repository must be pushed to GitHub
2. **Railway Account** - Sign up at [railway.app](https://railway.app)
3. **Git** - Installed on your local machine

## 🚀 Step-by-Step Deployment

### Step 1: Push Code to GitHub

```bash
cd /Users/srinivas/Documents/GitHub/shoplive
git add .
git commit -m "Ready for Railway deployment"
git push origin main
```

### Step 2: Create Railway Project

1. Go to [railway.app](https://railway.app)
2. Click **"New Project"**
3. Select **"Deploy from GitHub repo"**
4. Authorize Railway to access your GitHub account
5. Select the `shoplive` repository
6. Click **"Deploy"**

### Step 3: Configure Services

Railway will auto-detect your services. You'll see:
- ✅ `backend` service (detected from `backend/package.json`)
- ✅ `frontend` service (detected from `frontend/package.json`)

**If services are not auto-detected:**

1. Click **"New Service"** → **"GitHub Repo"**
2. Select `shoplive` repository
3. Set **"Root Directory"** to `backend`
4. Click **"Deploy"**

Repeat for frontend with root directory `frontend`.

### Step 4: Add PostgreSQL Database

1. Click **"New Service"** → **"Database"** → **"PostgreSQL"**
2. Railway automatically creates and configures PostgreSQL
3. The `DATABASE_URL` is automatically injected into the backend service

### Step 5: Configure Environment Variables

#### Backend Service Variables

1. Go to **Backend Service** → **Variables**
2. Add these variables:

```
NODE_ENV=production
JWT_SECRET=<generate-secure-key>
CORS_ORIGIN=https://<your-frontend-url>.railway.app
```

**To generate a secure JWT_SECRET:**
```bash
node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"
```

**Note**: `DATABASE_URL` is automatically provided by Railway's PostgreSQL service.

#### Frontend Service Variables

1. Go to **Frontend Service** → **Variables**
2. Add:

```
VITE_API_URL=https://<your-backend-url>.railway.app
```

### Step 6: Deploy

Railway automatically deploys when you push to GitHub. Monitor deployment in the **Deployments** tab.

## 📊 Service URLs

After deployment, Railway provides URLs for each service:

- **Backend API**: `https://<backend-service-name>.railway.app`
- **Frontend**: `https://<frontend-service-name>.railway.app`
- **Database**: Automatically connected via `DATABASE_URL`

## 🔧 Service Configuration Files

Each service has a `railway.json` configuration file:

### `backend/railway.json`
```json
{
  "$schema": "https://railway.app/railway.schema.json",
  "build": {
    "builder": "NIXPACKS"
  },
  "deploy": {
    "startCommand": "npm start",
    "restartPolicyType": "ON_FAILURE",
    "restartPolicyMaxRetries": 10
  }
}
```

### `frontend/railway.json`
```json
{
  "$schema": "https://railway.app/railway.schema.json",
  "build": {
    "builder": "NIXPACKS"
  },
  "deploy": {
    "startCommand": "npm run preview",
    "restartPolicyType": "ON_FAILURE",
    "restartPolicyMaxRetries": 10
  }
}
```

## 🔐 Security Configuration

### Backend CORS

The backend is configured to accept requests from your frontend domain:

```javascript
app.use(cors({
  origin: process.env.CORS_ORIGIN || '*',
  credentials: true
}));
```

Set `CORS_ORIGIN` to your frontend URL to restrict access.

### JWT Authentication

- Tokens expire in 7 days
- Passwords are hashed with bcryptjs
- Set a strong `JWT_SECRET` in environment variables

### Database Security

- PostgreSQL is managed by Railway
- Automatic backups are enabled
- Connection uses SSL/TLS

## 📁 File Storage

Images are stored in `/uploads` directory. For production:

**Option 1: Railway Volumes (Recommended for small deployments)**
1. Go to Backend Service → **Settings**
2. Add Volume: Mount path `/app/uploads`
3. Size: 1GB (or as needed)

**Option 2: Cloud Storage (Recommended for scalability)**
- Use AWS S3, Cloudinary, or similar
- Update `server.js` to upload to cloud storage
- More scalable and reliable

## 🧪 Testing Deployment

### Health Check

```bash
curl https://<your-backend-url>.railway.app/health
```

Expected response:
```json
{
  "status": "ok",
  "message": "Server is running"
}
```

### Test Registration

```bash
curl -X POST https://<your-backend-url>.railway.app/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "firstName": "Test",
    "lastName": "User",
    "email": "test@example.com",
    "password": "password123",
    "companyName": "Test Company"
  }'
```

### Test Frontend

1. Visit `https://<your-frontend-url>.railway.app`
2. Register a new account
3. Login with your credentials
4. Create a product
5. Generate QR code
6. Test order placement

## 🚨 Troubleshooting

### Services Not Detected

**Problem**: Railway doesn't show backend/frontend services

**Solution**:
1. Ensure `package.json` exists in `backend/` and `frontend/`
2. Manually create services with correct root directories
3. Check that `railway.json` files are present

### Backend Won't Start

**Problem**: Backend service fails to deploy

**Solution**:
1. Check logs: Backend Service → **Logs**
2. Verify `DATABASE_URL` is set
3. Verify `JWT_SECRET` is set
4. Check for syntax errors in `server.js`

### Frontend Can't Connect to Backend

**Problem**: Frontend shows API connection errors

**Solution**:
1. Verify `VITE_API_URL` is set correctly
2. Check backend `CORS_ORIGIN` includes frontend domain
3. Ensure both services are running
4. Check browser console for CORS errors

### Database Connection Errors

**Problem**: Backend can't connect to PostgreSQL

**Solution**:
1. Verify PostgreSQL service is running
2. Check `DATABASE_URL` is present in backend variables
3. Check backend logs for connection errors
4. Restart backend service

## 📈 Monitoring

### View Logs

1. Go to Service → **Logs**
2. Filter by date/time
3. Search for errors

### Monitor Metrics

1. Go to Service → **Metrics**
2. View CPU, Memory, Network usage
3. Set up alerts if needed

## 🔄 Continuous Deployment

Railway automatically deploys when you push to GitHub:

```bash
# Make changes locally
git add .
git commit -m "Update feature"
git push origin main

# Railway automatically detects and deploys
# Monitor in Railway Dashboard → Deployments
```

## 📝 Environment Variables Reference

### Backend (.env)

```
# Database (auto-provided by Railway)
DATABASE_URL=postgresql://user:password@host:5432/dbname

# Security
JWT_SECRET=your-secure-random-string
NODE_ENV=production

# Server
PORT=3001

# CORS
CORS_ORIGIN=https://your-frontend-domain.railway.app
```

### Frontend (.env.production)

```
VITE_API_URL=https://your-backend-domain.railway.app
```

## 🎯 Deployment Checklist

- [ ] Code pushed to GitHub
- [ ] Railway project created
- [ ] Backend service configured (root: `backend`)
- [ ] Frontend service configured (root: `frontend`)
- [ ] PostgreSQL database added
- [ ] Backend environment variables set
- [ ] Frontend environment variables set
- [ ] Deployment successful
- [ ] Health endpoint responds
- [ ] Frontend loads
- [ ] Can register account
- [ ] Can login
- [ ] Can create product
- [ ] Can place order

## 📞 Support

- **Railway Docs**: https://docs.railway.app
- **Railway Discord**: https://discord.gg/railway
- **Project Issues**: GitHub Issues

## 🎓 Next Steps

1. **Deploy to Railway** - Follow this guide
2. **Set up custom domain** - Railway → Settings → Domains
3. **Configure backups** - Railway → Database → Settings
4. **Monitor performance** - Railway → Metrics
5. **Set up alerts** - Railway → Alerts

---

**Status**: ✅ Ready for Railway deployment!

Your project is fully configured for monorepo deployment on Railway.
