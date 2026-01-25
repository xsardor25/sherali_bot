# Quick Start Guide - Docker Deployment

## 🚀 Fast Deployment (5 Minutes)

### Step 1: Clone Repository

```bash
git clone https://github.com/XushvaqtovSardor/sherali_bot2.git
cd sherali_bot2
```

### Step 2: Create Environment File

Create a `.env` file in the root directory:

```bash
# Copy the template
cp .env.example .env

# Edit with your values
nano .env
```

Add these variables:

```env
BOT_TOKEN=your_bot_token_from_@BotFather
BOT_USERNAME=your_bot_username
ADMIN_ID=your_telegram_user_id
WEB_PANEL_URL=http://localhost:4010
```

### Step 3: Start Docker Containers

```bash
# Build and start all services
docker-compose up -d

# This will:
# ✓ Build the application image
# ✓ Start PostgreSQL database
# ✓ Run database migrations
# ✓ Start the bot
```

### Step 4: Check Status

```bash
# View logs
docker-compose logs -f

# Check if services are running
docker-compose ps

# Should see both 'db' and 'app' as 'Up'
```

### Step 5: Test the Bot

1. Open Telegram
2. Search for your bot by username
3. Send `/start` command
4. Bot should respond!

---

## 🔍 Verify Everything Works

### Check Application

```bash
# Health check endpoint
curl http://localhost:4010/api/health

# Should return: {"status":"ok","timestamp":"..."}
```

### Check Database

```bash
# Connect to database
docker-compose exec db psql -U azizbot -d sherali_bot_db

# List tables
\dt

# Should see: User, Choice, Log, ScreenshotCache, etc.
```

### Check Browser (for screenshots)

```bash
# Access app container
docker-compose exec app sh

# Check if Chromium is installed
which chromium-browser
# Should output: /usr/bin/chromium-browser

# Test Chromium
chromium-browser --version
# Should show version number

# Exit container
exit
```

---

## 📋 Common Issues & Solutions

### Issue 1: Bot doesn't start

```bash
# Check logs
docker-compose logs app | grep ERROR

# Common causes:
# - Invalid BOT_TOKEN
# - Bot already running elsewhere
# - Database connection issues
```

**Solution:**
```bash
# Restart services
docker-compose down
docker-compose up -d
```

### Issue 2: Database connection error

```bash
# Check if database is running
docker-compose ps db

# Should show 'Up (healthy)'
```

**Solution:**
```bash
# Recreate database
docker-compose down -v
docker-compose up -d
```

### Issue 3: Screenshot errors

Check if Chromium is installed in container:

```bash
docker-compose exec app which chromium-browser
```

**Solution:**
If not found, rebuild the image:
```bash
docker-compose down
docker-compose build --no-cache
docker-compose up -d
```

### Issue 4: Port already in use

```bash
# Error: "port is already allocated"
```

**Solution:**
Change ports in `docker-compose.yml`:
```yaml
ports:
  - "5010:3000"  # Change 4010 to 5010
```

---

## 🛠 Useful Commands

### Start/Stop

```bash
# Start services
docker-compose up -d

# Stop services
docker-compose stop

# Stop and remove
docker-compose down

# Restart specific service
docker-compose restart app
```

### Logs

```bash
# All logs
docker-compose logs -f

# App logs only
docker-compose logs -f app

# Last 50 lines
docker-compose logs --tail=50 app
```

### Database

```bash
# Backup database
docker-compose exec db pg_dump -U azizbot sherali_bot_db > backup.sql

# Restore database
docker-compose exec -T db psql -U azizbot sherali_bot_db < backup.sql

# Access database shell
docker-compose exec db psql -U azizbot -d sherali_bot_db
```

### Updates

```bash
# Pull latest code
git pull origin main

# Rebuild and restart
docker-compose down
docker-compose up -d --build
```

---

## 🎯 Production Deployment

### Option 1: Manual Deployment

```bash
# On your server
ssh user@your-server-ip
git clone https://github.com/XushvaqtovSardor/sherali_bot2.git
cd sherali_bot2

# Setup environment
nano .env
# Add your production values

# Start services
docker-compose up -d

# Check status
docker-compose ps
docker-compose logs -f
```

### Option 2: Automatic Deployment with GitHub Actions

1. **Setup GitHub Secrets** (in repository settings):
   - `SSH_HOST`: Your server IP
   - `SSH_USERNAME`: SSH username
   - `SSH_KEY`: Private SSH key
   - `SSH_PORT`: SSH port (usually 22)

2. **Push to main branch**:
```bash
git add .
git commit -m "Deploy to production"
git push origin main
```

3. **GitHub Actions will automatically**:
   - SSH into your server
   - Pull latest changes
   - Rebuild Docker containers
   - Restart services

---

## 📊 Monitoring

### Real-time Stats

```bash
# Container resource usage
docker stats

# Service status
docker-compose ps

# Health check
curl http://localhost:4010/api/health
```

### Logs Location

```bash
# Application logs (inside container)
docker-compose exec app ls -la /app/logs

# View log files
docker-compose exec app cat /app/logs/combined.log
```

---

## 🔐 Security Checklist

Before deploying to production:

- [ ] Change PostgreSQL password in `docker-compose.yml`
- [ ] Set strong `BOT_TOKEN`
- [ ] Configure firewall (only allow needed ports)
- [ ] Use HTTPS for web panel if public
- [ ] Enable Docker resource limits
- [ ] Set up regular database backups
- [ ] Update `ADMIN_ID` to your Telegram ID
- [ ] Review and set appropriate file permissions

---

## 🎉 Success!

If you see this in logs:

```
✅ BOT STARTED: @YourBotUsername
✅ PostgreSQL connected successfully
✅ Browser initialized successfully: Chrome/...
✓ Application is running on: http://...
```

**Congratulations! Your bot is running! 🎊**

---

## 📞 Support

- **Issues**: Create an issue on GitHub
- **Telegram**: Contact @ksh247
- **Documentation**: Check `DOCKER_DEPLOYMENT.md` for detailed guide

---

## 📝 Quick Reference

| Command | Description |
|---------|-------------|
| `docker-compose up -d` | Start services |
| `docker-compose down` | Stop services |
| `docker-compose logs -f app` | View app logs |
| `docker-compose ps` | Check status |
| `docker-compose restart app` | Restart app |
| `docker-compose build --no-cache` | Rebuild from scratch |
| `docker-compose exec app sh` | Access container |

---

**Made with ❤️ by Sardor**
