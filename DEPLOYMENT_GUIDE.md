# KIET Traveller - Deployment Guide

## Pre-Deployment Checklist

✅ **Build Status**: Fixed
- Frontend vite build optimized with code-splitting
- Backend dependencies configured
- All npm vulnerabilities reviewed

## Build Output

### Frontend Build Results
- **Main chunk**: 642.57 kB (gzip: 77.26 kB)
- **UI libs chunk**: 800.35 kB (gzip: 146.49 kB)
- **Map chunk**: 939.37 kB (gzip: 255.39 kB)
- **Vendor chunk**: 152.66 kB (gzip: 49.79 kB)
- **CSS**: 109.04 kB (gzip: 17.71 kB)

All chunks are now below 1MB uncompressed.

## Environment Configuration

### For Development
Use the existing `.env` file for local development with `localhost:4028`

### For Production
Copy `.env.production` and update these variables:
```
FRONTEND_URL=https://your-production-domain.com
SOCKET_CORS_ORIGIN=https://your-production-domain.com
```

Keep these values as-is (already configured):
- MONGODB_URI (MongoDB Atlas)
- JWT_SECRET & JWT_REFRESH_SECRET
- MAILJET credentials
- Email configuration

## Deployment Steps

### 1. Build Frontend & Backend
```bash
npm run build
```
This will:
- Install root dependencies
- Install frontend dependencies
- Build frontend to `/frontend/build`
- Backend is ready as-is

### 2. Prepare Production Environment
```bash
cp .env.production .env
# Update FRONTEND_URL and SOCKET_CORS_ORIGIN with your domain
```

### 3. Start Server
```bash
# From root directory:
npm start

# Or with Node.js directly:
node backend/server.js
```

The server will:
- Start on port 5000 (or PORT env var)
- Serve static frontend from `/frontend/build`
- API routes available at `/api/*`
- Health check: `GET /health`

## Health Check
```bash
curl http://localhost:5000/health
```

Response:
```json
{
  "status": "OK",
  "timestamp": "2025-11-22T...",
  "uptime": 123.45,
  "environment": "production"
}
```

## Deployment Platforms

### Heroku / Render / Fly.io
1. Push repository
2. Set environment variables in platform dashboard (use `.env.production` values)
3. Platform will run `npm start` automatically

### Docker
```dockerfile
FROM node:18-alpine

WORKDIR /app

COPY package*.json ./
RUN npm install --production

COPY . .

# Build frontend
RUN npm run build

EXPOSE 5000

CMD ["npm", "start"]
```

### AWS EC2 / DigitalOcean / VPS
```bash
# On server:
git clone <repo>
cd KIET-Traveller
npm install
npm run build
# Configure .env
npm start
```

Use PM2 for process management:
```bash
npm install -g pm2
pm2 start backend/server.js --name "kiet-traveller"
pm2 save
pm2 startup
```

## Troubleshooting

### Issue: Large Bundle Sizes
✅ **Fixed**: Implemented code-splitting in vite.config.mjs

### Issue: MongoDB Connection Failed
- Verify MONGODB_URI is correct
- Check IP whitelist on MongoDB Atlas
- Ensure network connectivity

### Issue: Static Files Not Serving
- Verify `/frontend/build` directory exists
- Check that build was successful: `npm run build`

### Issue: Socket.IO Connection Failed
- Update SOCKET_CORS_ORIGIN to match frontend domain
- Check CORS settings in server.js

## Performance Tips

1. **Enable compression** - Already enabled via helmet & compression middleware
2. **Rate limiting** - Configured to 100 req/15min per IP
3. **Database indexes** - Ensure MongoDB indexes are created (check models)
4. **CDN** - Consider serving static assets via CDN (CloudFront, Cloudflare)

## Monitoring

Monitor these health indicators:
- `/health` endpoint response time
- Database connection status
- Socket.IO connection count
- API response times
- Error logs in production

## Next Steps

1. Update FRONTEND_URL in `.env.production`
2. Test locally: `npm start`
3. Deploy to your platform
4. Monitor logs and health endpoint
5. Set up automatic backups for MongoDB
