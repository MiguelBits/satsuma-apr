# Heroku Deployment Guide

## What I Fixed

1. **Created `requirements.txt`** - Heroku requires this file for Python projects (your project was using Poetry)
2. **Fixed `manage.py` location** - Moved it to the root directory where Heroku expects it
3. **Updated `Procfile`** - Fixed the paths to work with the new structure
4. **Fixed `STATIC_ROOT`** - Changed from Docker path to Heroku-compatible path
5. **Added missing dependencies** - Added `requests`, `eth-typing`, and `kombu`

## Deployment Steps

### 1. Connect to Heroku
```bash
heroku login
```

### 2. Create Heroku App
```bash
heroku create your-app-name
```

### 3. Add PostgreSQL Database
```bash
heroku addons:create heroku-postgresql:mini
```

### 4. Add Redis for Celery
```bash
heroku addons:create heroku-redis:mini
```

### 5. Set Environment Variables
You need to set these environment variables in Heroku. Go to your Heroku dashboard → Settings → Config Vars, or use the CLI:

```bash
# Django Settings
heroku config:set BACKEND_SECRET_KEY="your-secret-key-here"
heroku config:set BACKEND_DEBUG_MODE=0
heroku config:set BACKEND_SETTINGS_MODE=production
heroku config:set BACKEND_SETTINGS_TYPE=production

# Database (Heroku will set these automatically, but you can override)
heroku config:set DATABASE_NAME="your-db-name"
heroku config:set DATABASE_ORIGIN_USER="your-db-user"
heroku config:set DATABASE_ORIGIN_PASS="your-db-password"
heroku config:set DATABASE_HOST="your-db-host"
heroku config:set DATABASE_PORT="5432"

# Allowed Hosts
heroku config:set BACKEND_ALLOWED_HOSTS="your-app-name.herokuapp.com"
heroku config:set BACKEND_CSRF_TRUSTED_ORIGINS="https://your-app-name.herokuapp.com"

# CORS Settings
heroku config:set CORS_ALLOWED_ORIGINS="https://your-app-name.herokuapp.com"
heroku config:set CORS_ALLOW_CREDENTIALS=1
heroku config:set CORS_ORIGIN_ALLOW_ALL=0
heroku config:set CORS_ALLOW_METHODS="GET POST PUT DELETE OPTIONS"

# Redis/Celery Settings
heroku config:set BROKER_SERVICE_NAME="your-redis-service-name"
heroku config:set BROKER_HOST="your-redis-host"
heroku config:set BROKER_PORT="6379"
heroku config:set CELERY_CONCURRENCY=1

# Email Settings (if needed)
heroku config:set EMAIL_HOST="smtp.gmail.com"
heroku config:set EMAIL_PORT="587"
heroku config:set EMAIL_HOST_USER="your-email@gmail.com"
heroku config:set EMAIL_HOST_PASSWORD="your-email-password"
heroku config:set EMAIL_TO="recipient@example.com"

# Blockchain Settings
heroku config:set BLOCK_DELTA=100
heroku config:set APR_DELTA=3600
heroku config:set DEFAULT_TXN_TIMEOUT=300
heroku config:set DEFAULT_POLL_LATENCY=1
```

### 6. Deploy to Heroku
```bash
git add .
git commit -m "Prepare for Heroku deployment"
git push heroku main
```

### 7. Run Database Migrations
```bash
heroku run python manage.py migrate
```

### 8. Create Superuser (Optional)
```bash
heroku run python manage.py createsuperuser
```

### 9. Collect Static Files
```bash
heroku run python manage.py collectstatic --noinput
```

### 10. Scale Workers (Optional)
If you want to run Celery workers:
```bash
heroku ps:scale worker=1
```

## Important Notes

1. **Database**: Heroku automatically provides PostgreSQL. The connection details are available in `DATABASE_URL` environment variable.

2. **Redis**: Heroku Redis addon provides Redis for Celery. The connection details are in `REDIS_URL`.

3. **Static Files**: For production, consider using a CDN like AWS S3 or Cloudinary for static files.

4. **Environment Variables**: Make sure to replace placeholder values with your actual configuration.

5. **SSL**: Heroku automatically provides SSL certificates for your domain.

## Troubleshooting

If you encounter issues:

1. Check logs: `heroku logs --tail`
2. Verify environment variables: `heroku config`
3. Test locally: `heroku local web`

## File Structure After Changes

```
backend/
├── manage.py (NEW - moved from best_apr_backend/)
├── requirements.txt (NEW - for Heroku)
├── Procfile (UPDATED - fixed paths)
├── runtime.txt (EXISTING - Python version)
├── best_apr_backend/
│   ├── manage.py (EXISTING - original)
│   ├── requirements.txt (NEW - backup)
│   └── backend/settings/base.py (UPDATED - STATIC_ROOT)
└── ...
```

Your project should now deploy successfully to Heroku! 