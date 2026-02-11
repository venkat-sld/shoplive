# Local Testing Guide - Docker Compose

This guide explains how to test the Live Sales Platform locally using Docker Compose with PostgreSQL.

## Prerequisites

- Docker and Docker Compose installed
- Git repository cloned locally
- Node.js 18+ (optional, for direct testing)

## Quick Start

### 1. Start All Services

```bash
cd /Users/srinivas/Documents/GitHub/shoplive
docker-compose -f docker-compose.railway.yml up --build
```

This will:
- Start PostgreSQL database on port 5432
- Build and start backend on port 3001
- Build and start frontend on port 80

### 2. Wait for Services to Be Ready

Watch the logs for:
```
live-sales-postgres | database system is ready to accept connections
live-sales-backend | Connected to PostgreSQL database
live-sales-backend | Database tables initialized successfully
live-sales-backend | Server running on port 3001
live-sales-frontend | nginx: master process started
```

### 3. Access the Application

- **Frontend**: http://localhost
- **Backend API**: http://localhost:3001
- **Health Check**: http://localhost:3001/health

## Testing Workflow

### 1. Register a New Account

```bash
curl -X POST http://localhost:3001/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "firstName": "John",
    "lastName": "Doe",
    "email": "john@example.com",
    "password": "password123",
    "companyName": "Test Company"
  }'
```

Response:
```json
{
  "message": "User registered successfully",
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

### 2. Login

```bash
curl -X POST http://localhost:3001/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "john@example.com",
    "password": "password123"
  }'
```

### 3. Create a Product

```bash
TOKEN="your-jwt-token-from-login"

curl -X POST http://localhost:3001/api/products \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{
    "name": "Test Product",
    "description": "A test product",
    "price": 99.99,
    "size": "M",
    "color": "Blue",
    "stock_quantity": 10
  }'
```

### 4. Get Products

```bash
curl -X GET http://localhost:3001/api/products \
  -H "Authorization: Bearer $TOKEN"
```

### 5. Test Frontend

1. Open http://localhost in your browser
2. Register a new account
3. Login with your credentials
4. Create a product
5. Generate QR code
6. Test order placement

## Troubleshooting

### PostgreSQL Connection Error

**Error**: `Error connecting to database: connect ECONNREFUSED`

**Solution**:
```bash
# Check if postgres service is running
docker-compose -f docker-compose.railway.yml ps

# Restart postgres
docker-compose -f docker-compose.railway.yml restart postgres

# Wait 10 seconds and restart backend
docker-compose -f docker-compose.railway.yml restart backend
```

### Backend Won't Start

**Error**: `Cannot find module 'pg'`

**Solution**:
```bash
# Rebuild without cache
docker-compose -f docker-compose.railway.yml up --build --no-cache

# Or manually install dependencies
cd backend
npm install
cd ..
```

### Frontend Can't Connect to Backend

**Error**: `Failed to fetch from http://localhost:3001`

**Solution**:
1. Check backend is running: `curl http://localhost:3001/health`
2. Check CORS is configured correctly
3. Verify `VITE_API_URL` is set to `http://localhost:3001`

### Port Already in Use

**Error**: `bind: address already in use`

**Solution**:
```bash
# Find process using port
lsof -i :3001  # Backend
lsof -i :80    # Frontend
lsof -i :5432  # PostgreSQL

# Kill the process
kill -9 <PID>

# Or change ports in docker-compose.railway.yml
```

### Database Tables Not Created

**Error**: `relation "users" does not exist`

**Solution**:
```bash
# Check backend logs
docker-compose -f docker-compose.railway.yml logs backend

# Restart backend to trigger table creation
docker-compose -f docker-compose.railway.yml restart backend

# Wait 30 seconds for tables to be created
```

## Useful Commands

### View Logs

```bash
# All services
docker-compose -f docker-compose.railway.yml logs -f

# Specific service
docker-compose -f docker-compose.railway.yml logs -f backend
docker-compose -f docker-compose.railway.yml logs -f postgres
docker-compose -f docker-compose.railway.yml logs -f frontend
```

### Stop Services

```bash
# Stop all services
docker-compose -f docker-compose.railway.yml down

# Stop and remove volumes (clean slate)
docker-compose -f docker-compose.railway.yml down -v
```

### Rebuild Services

```bash
# Rebuild without cache
docker-compose -f docker-compose.railway.yml up --build --no-cache

# Rebuild specific service
docker-compose -f docker-compose.railway.yml build --no-cache backend
```

### Access Database

```bash
# Connect to PostgreSQL
docker-compose -f docker-compose.railway.yml exec postgres psql -U postgres -d shoplive

# List tables
\dt

# Query users
SELECT * FROM users;

# Exit
\q
```

### Access Backend Container

```bash
# Open bash shell
docker-compose -f docker-compose.railway.yml exec backend sh

# View files
ls -la

# Check environment variables
env | grep DATABASE_URL

# Exit
exit
```

## Testing Checklist

- [ ] All services start without errors
- [ ] PostgreSQL is healthy
- [ ] Backend connects to database
- [ ] Frontend loads at http://localhost
- [ ] Can register a new account
- [ ] Can login with credentials
- [ ] Can create a product
- [ ] Can view products
- [ ] Can update a product
- [ ] Can delete a product
- [ ] Can place an order
- [ ] Can view orders
- [ ] Health check endpoint works
- [ ] Images can be uploaded
- [ ] QR codes can be generated

## Performance Testing

### Load Testing Backend

```bash
# Install Apache Bench (if not installed)
# macOS: brew install httpd

# Test health endpoint
ab -n 100 -c 10 http://localhost:3001/health

# Test product listing (requires auth token)
ab -n 100 -c 10 -H "Authorization: Bearer $TOKEN" \
  http://localhost:3001/api/products
```

### Database Performance

```bash
# Connect to database
docker-compose -f docker-compose.railway.yml exec postgres psql -U postgres -d shoplive

# Check table sizes
SELECT schemaname, tablename, pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) 
FROM pg_tables 
WHERE schemaname != 'pg_catalog' 
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC;

# Check slow queries
SELECT query, calls, mean_time FROM pg_stat_statements 
ORDER BY mean_time DESC LIMIT 10;
```

## Next Steps

After successful local testing:

1. **Push to GitHub**: `git push origin main`
2. **Deploy to Railway**: Follow QUICK_START_RAILWAY.md
3. **Test on Railway**: Verify all features work in production
4. **Monitor Logs**: Check Railway dashboard for errors
5. **Set up Backups**: Configure database backups

## Support

- **Docker Docs**: https://docs.docker.com
- **PostgreSQL Docs**: https://www.postgresql.org/docs
- **Railway Docs**: https://docs.railway.app

---

**Status**: ✅ Ready for local testing!

For deployment instructions, see `QUICK_START_RAILWAY.md`.
