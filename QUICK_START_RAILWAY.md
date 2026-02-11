# Quick Start: Deploy to Railway in 5 Minutes

This is a quick reference guide for deploying the Live Sales Platform to Railway.

## Prerequisites

- GitHub account with the repository pushed
- Railway account (free tier available at [railway.app](https://railway.app))

## Step-by-Step Deployment

### 1. Push Your Code to GitHub

```bash
cd /Users/srinivas/Documents/GitHub/shoplive
git add .
git commit -m "Ready for Railway deployment"
git push origin main
```

### 2. Create Railway Project

1. Go to [railway.app](https://railway.app)
2. Click **"New Project"**
3. Select **"Deploy from GitHub repo"**
4. Select your `shoplive` repository
5. Click **"Deploy"**

### 3. Add Services

Railway will auto-detect your project. Add these services:

#### Backend Service
- **Type**: GitHub Repo
- **Root Directory**: `backend`
- **Build Command**: `npm install`
- **Start Command**: `npm start`

#### Frontend Service
- **Type**: GitHub Repo
- **Root Directory**: `frontend`
- **Build Command**: `npm run build`
- **Start Command**: `npm run preview`

#### Database Service
- **Type**: PostgreSQL
- Railway auto-creates and connects it

### 4. Set Environment Variables

#### Backend Variables
```
NODE_ENV=production
JWT_SECRET=<run: node -e "console.log(require('crypto').randomBytes(32).toString('hex'))">
CORS_ORIGIN=https://<your-frontend-url>.railway.app
```

#### Frontend Variables
```
VITE_API_URL=https://<your-backend-url>.railway.app
```

### 5. Deploy

Railway automatically deploys when you push to GitHub. Monitor in the **Deployments** tab.

## Verify Deployment

1. **Backend Health**: Visit `https://<backend-url>.railway.app/health`
2. **Frontend**: Visit `https://<frontend-url>.railway.app`
3. **Test**: Register, login, and create a product

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Backend won't start | Check logs for `DATABASE_URL` and `JWT_SECRET` |
| Frontend can't connect | Verify `VITE_API_URL` and backend `CORS_ORIGIN` |
| Database errors | Ensure PostgreSQL service is running |
| Image uploads fail | Add Railway Volume at `/app/uploads` |

## Useful Commands

```bash
# Generate secure JWT_SECRET
node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"

# View logs locally
docker-compose -f docker-compose.railway.yml logs -f

# Test locally before deploying
docker-compose -f docker-compose.railway.yml up --build
```

## Next Steps

- [ ] Set up custom domain (Railway → Settings → Domains)
- [ ] Enable auto-deploy on push (Railway → Settings)
- [ ] Configure backups (Railway → Database → Settings)
- [ ] Set up monitoring alerts (Railway → Alerts)
- [ ] Review security settings (Railway → Settings → Security)

## Support

- **Railway Docs**: https://docs.railway.app
- **Full Guide**: See `RAILWAY_DEPLOYMENT.md`
- **Issues**: Check GitHub Issues or Railway Discord

---

**Status**: ✅ Ready to deploy!

For detailed instructions, see `RAILWAY_DEPLOYMENT.md`.
