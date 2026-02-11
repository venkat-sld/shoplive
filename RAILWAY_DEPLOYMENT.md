# Railway Deployment Guide for Live Sales Platform

This guide provides step-by-step instructions to deploy the Live Sales Platform on Railway.

## 📋 Prerequisites

1. **Railway Account**: Sign up at [railway.app](https://railway.app)
2. **GitHub Account**: Repository must be pushed to GitHub
3. **Git**: Installed on your local machine
4. **Node.js**: Version 18+ (for local testing)

## 🚀 Quick Start Deployment

### Step 1: Push Code to GitHub

```bash
cd /Users/srinivas/Documents/GitHub/shoplive
git add .
git commit -m "Prepare for Railway deployment"
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

Railway will automatically detect your project structure. You need to create two services:

#### Service 1: Backend (Express.js)

1. In Railway dashboard, click **"New Service"** → **"GitHub Repo"**
2. Select your `shoplive` repository
3. Configure the service:
   - **Name**: `backend`
   - **Root Directory**: `backend`
   - **Build Command**: `npm install`
   - **Start Command**: `npm start`

#### Service 2: Frontend (React)

1. Click **"New Service"** → **"GitHub Repo"**
2. Select your `shoplive` repository
3. Configure the service:
   - **Name**: `frontend`
   - **Root Directory**: `frontend`
   - **Build Command**: `npm run build`
   - **Start Command**: `npm run preview` (or use Railway's default)

#### Service 3: PostgreSQL Database

1. Click **"New Service"** → **"Database"** → **"PostgreSQL"**
2. Railway will automatically create a PostgreSQL instance
3. The `DATABASE_URL` will be automatically available to your backend service

### Step 4: Set Environment Variables

#### For Backend Service:

1. Go to **Backend Service** → **Variables**
2. Add the following variables:

```
NODE_ENV=production
JWT_SECRET=<generate-a-secure-random-string>
CORS_ORIGIN=https://<your-frontend-domain>.railway.app
```

**To generate a secure JWT_SECRET:**
```bash
node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"
```

#### For Frontend Service:

1. Go to **Frontend Service** → **Variables**
2. Add:

```
VITE_API_URL=https://<your-backend-domain>.railway.app
```

### Step 5: Connect Services

1. **Backend to Database**:
   - Go to Backend Service → **Variables**
   - Railway automatically injects `DATABASE_URL` from PostgreSQL service
   - Verify it's present in the variables list

2. **Frontend to Backend**:
   - Update `frontend/.env.production` with the backend URL
   - Or set `VITE_API_URL` environment variable in Railway

### Step 6: Deploy

1. Railway automatically deploys when you push to GitHub
2. Monitor deployment in the **Deployments** tab
3. Check logs for any errors

## 📊 Service URLs

After deployment, Railway provides URLs for each service:

- **Backend API**: `https://<backend-service-name>.railway.app`
- **Frontend**: `https://<frontend-service-name>.railway.app`
- **Database**: Automatically connected via `DATABASE_URL`

## 🔧 Configuration Details

### Backend Configuration

The backend is configured to:
- Use PostgreSQL via `DATABASE_URL` environment variable
- Support CORS with configurable `CORS_ORIGIN`
- Use bcryptjs for password hashing
- Generate JWT tokens with configurable `JWT_SECRET`
- Store uploaded images in `/uploads` directory (Railway Volume)

### Frontend Configuration

The frontend is configured to:
- Use `VITE_API_URL` to connect to backend
- Build with Vite for optimized production bundle
- Support environment-based API URLs

### Database

PostgreSQL is automatically provisioned with:
- Users table for merchant accounts
- Products table for product listings
- Orders table for customer orders
- Automatic backups (Railway feature)

## 📁 File Storage

Images are stored in the `/uploads` directory. For production:

**Option 1: Railway Volumes (Recommended for small deployments)**
- Persistent storage within Railway
- Automatic backups
- Limited to service instance

**Option 2: Cloud Storage (Recommended for scalability)**
- Use AWS S3, Cloudinary, or similar
- Update `server.js` to upload to cloud storage
- More scalable and reliable

To enable Railway Volumes:
1. Go to Backend Service → **Settings**
2. Add Volume: Mount path `/app/uploads`
3. Size: 1GB (or as needed)

## 🔐 Security Checklist

- [ ] Set strong `JWT_SECRET` (use generated value)
- [ ] Set `NODE_ENV=production`
- [ ] Configure `CORS_ORIGIN` to your frontend domain
- [ ] Enable Railway's built-in SSL/TLS (automatic)
- [ ] Use strong database password (Railway generates this)
- [ ] Regularly update dependencies
- [ ] Monitor logs for errors and suspicious activity

## 🚨 Troubleshooting

### Backend won't start

**Check logs:**
```
Railway Dashboard → Backend Service → Logs
```

**Common issues:**
- Missing `DATABASE_URL`: Ensure PostgreSQL service is connected
- Missing `JWT_SECRET`: Add to environment variables
- Port binding: Railway automatically assigns PORT

### Frontend can't connect to backend

**Check:**
1. `VITE_API_URL` is set correctly in frontend environment
2. Backend `CORS_ORIGIN` includes frontend domain
3. Both services are running (check Deployments tab)

### Database connection errors

**Check:**
1. PostgreSQL service is running
2. `DATABASE_URL` is present in backend variables
3. Database tables are created (check backend logs)

### Image uploads not working

**Check:**
1. Volume is mounted at `/app/uploads`
2. File permissions are correct
3. Disk space is available

## 📈 Monitoring & Logs

### View Logs

1. Go to Service → **Logs**
2. Filter by date/time
3. Search for errors

### Monitor Performance

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

## 🆘 Support & Resources

- **Railway Docs**: https://docs.railway.app
- **Railway Community**: https://discord.gg/railway
- **Project Issues**: Check GitHub Issues

## 📝 Environment Variables Reference

### Backend (.env)

```
# Database
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

## 🎯 Next Steps

1. **Test the deployment**: Visit your frontend URL
2. **Create a test account**: Register and login
3. **Add test products**: Create some products
4. **Test QR codes**: Generate and scan QR codes
5. **Monitor logs**: Watch for any errors
6. **Set up backups**: Configure database backups
7. **Enable monitoring**: Set up alerts for errors

## 📞 Getting Help

If you encounter issues:

1. Check Railway logs for error messages
2. Verify all environment variables are set
3. Ensure database is connected
4. Check CORS configuration
5. Review backend and frontend logs
6. Contact Railway support if infrastructure issue

---

**Deployment Status**: Ready for production deployment on Railway ✅

For questions or issues, refer to the Railway documentation or contact support.
