# Docker Deployment Guide

## Prerequisites

Before deploying, ensure you have:
- Docker installed (version 20.10+)
- Docker Compose installed (version 2.0+)
- `.env` file configured with required variables

## Environment Variables

Create a `.env` file in the root directory with the following variables:

```env
# Bot Configuration
BOT_TOKEN=your_telegram_bot_token
BOT_USERNAME=your_bot_username
ADMIN_ID=your_telegram_user_id

# Web Panel (optional)
WEB_PANEL_URL=http://your-domain.com

# Database (these are already configured in docker-compose.yml)
DATABASE_URL=postgresql://azizbot:your_strong_password@db:5432/sherali_bot_db?schema=public
```

## Docker Configuration Features

### Dockerfile Optimizations

1. **Multi-stage Build**: Reduces final image size
2. **Alpine Linux**: Lightweight base image (~5MB)
3. **Chromium**: Pre-installed for screenshot functionality
4. **Non-root User**: Enhanced security
5. **Health Checks**: Automatic service monitoring
6. **dumb-init**: Proper signal handling

### Installed Packages in Container

- **chromium**: Browser for screenshots
- **chromium-chromedriver**: WebDriver support
- **nss, freetype, harfbuzz**: Rendering dependencies
- **ttf-freefont, font-noto-emoji**: Font support
- **wqy-zenhei**: Chinese character support
- **dumb-init**: Process manager

## Deployment Commands

### 1. Build and Start Services

```bash
# Build containers
docker-compose build

# Start services in background
docker-compose up -d

# View logs
docker-compose logs -f

# View specific service logs
docker-compose logs -f app
docker-compose logs -f db
```

### 2. Database Migrations

Migrations are automatically run when the container starts. To manually run:

```bash
# Run migrations
docker-compose exec app npx prisma migrate deploy

# Generate Prisma client
docker-compose exec app npx prisma generate
```

### 3. Monitor Services

```bash
# Check running containers
docker-compose ps

# Check container health
docker ps --format "table {{.Names}}\t{{.Status}}"

# View resource usage
docker stats

# Access container shell
docker-compose exec app sh
```

### 4. Stop and Remove Services

```bash
# Stop services
docker-compose stop

# Stop and remove containers
docker-compose down

# Stop, remove containers and volumes (⚠️ deletes database data)
docker-compose down -v
```

## Production Deployment

### 1. On Remote Server (e.g., DigitalOcean, AWS)

```bash
# SSH into your server
ssh user@your-server-ip

# Clone repository
git clone https://github.com/YourUsername/sherali_bot2.git
cd sherali_bot2

# Create .env file
nano .env
# Add your environment variables

# Build and start
docker-compose up -d

# Check logs
docker-compose logs -f
```

### 2. GitHub Actions Deployment (Already Configured)

Your repository already has GitHub Actions workflow. To deploy:

```bash
# Push to main branch
git add .
git commit -m "Deploy changes"
git push origin main
```

The workflow will:
1. SSH into your server
2. Pull latest changes
3. Rebuild containers
4. Restart services

### 3. Manual Update on Server

```bash
# SSH into server
ssh user@your-server-ip
cd /path/to/sherali_bot2

# Pull latest changes
git pull origin main

# Rebuild and restart
docker-compose down
docker-compose build --no-cache
docker-compose up -d

# Check logs
docker-compose logs -f
```

## Troubleshooting

### Browser/Screenshot Issues

If you see browser initialization errors:

```bash
# Check if Chromium is installed in container
docker-compose exec app which chromium-browser

# Should output: /usr/bin/chromium-browser

# Test Chromium manually
docker-compose exec app chromium-browser --version
```

### Database Connection Issues

```bash
# Check database status
docker-compose exec db pg_isready -U azizbot -d sherali_bot_db

# Connect to database
docker-compose exec db psql -U azizbot -d sherali_bot_db

# View tables
\dt

# Exit
\q
```

### Container Not Starting

```bash
# View detailed logs
docker-compose logs --tail=100 app

# Check container health
docker inspect sherali_bot_app | grep -A 10 Health

# Remove and recreate container
docker-compose down
docker-compose up -d
```

### Out of Memory

```bash
# Check memory usage
docker stats

# Increase Docker memory limit in Docker settings
# Or add to docker-compose.yml under app service:
# deploy:
#   resources:
#     limits:
#       memory: 1G
```

## Health Check Endpoints

The application exposes health check endpoints:

- **Application Health**: `http://localhost:4010/api/health`
- **Database Connection**: Automatically checked by health endpoint

## Container Ports

- **Application**: 4010 (host) → 3000 (container)
- **PostgreSQL**: 5435 (host) → 5432 (container)

## Backup and Restore

### Backup Database

```bash
# Create backup
docker-compose exec db pg_dump -U azizbot sherali_bot_db > backup.sql

# With timestamp
docker-compose exec db pg_dump -U azizbot sherali_bot_db > backup_$(date +%Y%m%d_%H%M%S).sql
```

### Restore Database

```bash
# Restore from backup
docker-compose exec -T db psql -U azizbot sherali_bot_db < backup.sql
```

## Security Recommendations

1. **Change Default Passwords**: Update PostgreSQL password in docker-compose.yml
2. **Use Secrets**: Store sensitive data in Docker secrets or environment variables
3. **Enable Firewall**: Only expose necessary ports
4. **Regular Updates**: Keep Docker images updated
5. **SSL/TLS**: Use HTTPS for web panel if exposed publicly
6. **Non-root User**: Already configured in Dockerfile
7. **Resource Limits**: Set memory and CPU limits in production

## Performance Optimization

1. **Resource Limits**: Adjust based on your server capacity
2. **Connection Pooling**: Already configured in DATABASE_URL
3. **Screenshot Cache**: Reduces Chromium usage
4. **Log Rotation**: Logs are stored in mounted volume

## Monitoring

### View Real-time Logs

```bash
# All services
docker-compose logs -f

# Only app
docker-compose logs -f app

# Last 100 lines
docker-compose logs --tail=100
```

### Container Stats

```bash
# Real-time stats
docker stats

# One-time snapshot
docker stats --no-stream
```

## Common Commands Cheatsheet

```bash
# Start services
docker-compose up -d

# Stop services
docker-compose down

# Rebuild after code changes
docker-compose up -d --build

# View logs
docker-compose logs -f app

# Access app container
docker-compose exec app sh

# Access database
docker-compose exec db psql -U azizbot -d sherali_bot_db

# Restart app only
docker-compose restart app

# Check service status
docker-compose ps

# Remove everything including volumes
docker-compose down -v

# Prune unused Docker resources
docker system prune -a
```

## Updating Application

```bash
# Method 1: Pull and rebuild
git pull origin main
docker-compose down
docker-compose up -d --build

# Method 2: Without downtime (rolling update)
docker-compose build app
docker-compose up -d --no-deps app
```

## Support

If you encounter issues:
1. Check logs: `docker-compose logs -f`
2. Verify environment variables in `.env`
3. Ensure ports 4010 and 5435 are available
4. Check Docker and Docker Compose versions
5. Verify server has enough resources (minimum 1GB RAM)

## Production Checklist

- [ ] Updated `.env` with production values
- [ ] Changed default PostgreSQL password
- [ ] Configured firewall rules
- [ ] Set up SSL/TLS if using web panel
- [ ] Configured backup strategy
- [ ] Set up monitoring/alerting
- [ ] Tested health check endpoints
- [ ] Verified screenshot functionality
- [ ] Set up log rotation
- [ ] Documented deployment process for team
