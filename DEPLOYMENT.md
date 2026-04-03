# Snipe-IT VPS Deployment Guide

## 📋 Prerequisites
- GitHub account
- VPS with SSH access
- Domain (asset.pupitech.me) pointed to VPS IP
- Cloudflare Origin Certificate generated

---

## PART 1: Local Machine (Where you are now)

### Step 1: Initialize Git Repository
```bash
cd /home/jomar/snipeit
git init
```

### Step 2: Create .gitignore
```bash
cat > .gitignore << 'EOF'
.env
*.log
.DS_Store
EOF
```

### Step 3: Commit Files
```bash
git add .
git commit -m "Initial commit: Snipe-IT docker setup with Nginx SSL"
```

### Step 4: Push to GitHub
```bash
# Create a new repository on GitHub first (e.g., snipeit-deployment)
# Then run:
git remote add origin https://github.com/YOUR_USERNAME/snipeit-deployment.git
git branch -M main
git push -u origin main
```

---

## PART 2: VPS Setup

### Step 5: SSH into VPS
```bash
ssh your_user@your_vps_ip
```

### Step 6: Install Docker & Docker Compose (if not installed)
```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Add user to docker group
sudo usermod -aG docker $USER

# Install Docker Compose
sudo apt install docker-compose -y

# Verify installation
docker --version
docker-compose --version

# Log out and back in for group changes to take effect
exit
# SSH back in
ssh your_user@your_vps_ip
```

### Step 7: Create SSL Directory & Upload Certificates
```bash
# Create directory for SSL certificates
sudo mkdir -p /etc/ssl/cloudflare
sudo chown $USER:$USER /etc/ssl/cloudflare
```

**Upload certificates from local to VPS:**
```bash
# On your LOCAL machine (open new terminal):
scp /path/to/cloudflare-cert.pem your_user@your_vps_ip:/etc/ssl/cloudflare/cert.pem
scp /path/to/cloudflare-key.pem your_user@your_vps_ip:/etc/ssl/cloudflare/key.pem

# Back on VPS, secure the files:
sudo chmod 600 /etc/ssl/cloudflare/key.pem
sudo chmod 644 /etc/ssl/cloudflare/cert.pem
```

**OR manually create them on VPS:**
```bash
# On VPS:
sudo nano /etc/ssl/cloudflare/cert.pem
# Paste certificate content, save (Ctrl+X, Y, Enter)

sudo nano /etc/ssl/cloudflare/key.pem
# Paste private key content, save (Ctrl+X, Y, Enter)

# Secure permissions
sudo chmod 600 /etc/ssl/cloudflare/key.pem
sudo chmod 644 /etc/ssl/cloudflare/cert.pem
```

### Step 8: Clone Repository on VPS
```bash
cd ~
git clone https://github.com/YOUR_USERNAME/snipeit-deployment.git
cd snipeit-deployment
```

### Step 9: Generate APP_KEY
```bash
docker run --rm snipe/snipe-it php artisan key:generate --show
# Copy the output (e.g., base64:xxxxxxxxxxxx)
```

### Step 10: Create .env File
```bash
nano .env
```

**Paste this and update values:**
```env
# Application
APP_ENV=production
APP_DEBUG=false
APP_KEY=base64:PASTE_YOUR_GENERATED_KEY_HERE

# Database
SNIPEIT_DB_PASSWORD=YOUR_SECURE_DB_PASSWORD_HERE

# Redis
REDIS_PASSWORD=YOUR_SECURE_REDIS_PASSWORD_HERE

# Mail (Optional - configure later)
MAIL_DRIVER=smtp
MAIL_HOST=
MAIL_PORT=587
MAIL_USERNAME=
MAIL_PASSWORD=
MAIL_ENCRYPTION=tls
MAIL_FROM_ADDR=
MAIL_FROM_NAME=Snipe-IT

# Session & Cache
SESSION_DRIVER=redis
CACHE_DRIVER=redis
QUEUE_CONNECTION=redis
```

Save: `Ctrl+X`, `Y`, `Enter`

### Step 11: Set up Database Network
```bash
# Check if facultyportfolio_default network exists
docker network ls | grep facultyportfolio_default

# If it doesn't exist, either:
# A) Create it manually:
docker network create facultyportfolio_default

# OR B) Remove it from docker-compose.snipeit.yml
# (If you're not using an external database)
```

### Step 12: Deploy Snipe-IT
```bash
# Start all services
docker-compose -f docker-compose.snipeit.yml up -d

# Check if containers are running
docker ps

# View logs
docker-compose -f docker-compose.snipeit.yml logs -f
# Press Ctrl+C to exit logs
```

### Step 13: Configure Firewall (if enabled)
```bash
# Allow HTTP and HTTPS
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw status
```

---

## PART 3: Cloudflare Configuration

### Step 14: Configure Cloudflare DNS
1. Go to Cloudflare Dashboard → DNS
2. Add/Update A Record:
   - **Type**: A
   - **Name**: asset
   - **IPv4 address**: YOUR_VPS_IP
   - **Proxy status**: Proxied (Orange cloud) ✅
   - **TTL**: Auto

### Step 15: Configure Cloudflare SSL
1. Go to SSL/TLS → Overview
2. Set mode to: **Full (strict)** ✅

---

## PART 4: Verify & Access

### Step 16: Test Access
```bash
# On VPS, test internal connection
curl -k https://localhost

# Check Nginx is working
docker logs snipeit-nginx

# Check Snipe-IT is working
docker logs snipeit
```

### Step 17: Access Snipe-IT
- Open browser: **https://asset.pupitech.me**
- Complete Snipe-IT setup wizard

---

## 🔧 Useful Commands

```bash
# View logs
docker-compose -f docker-compose.snipeit.yml logs -f snipeit
docker-compose -f docker-compose.snipeit.yml logs -f nginx

# Restart services
docker-compose -f docker-compose.snipeit.yml restart

# Stop services
docker-compose -f docker-compose.snipeit.yml down

# Update & redeploy
git pull origin main
docker-compose -f docker-compose.snipeit.yml down
docker-compose -f docker-compose.snipeit.yml pull
docker-compose -f docker-compose.snipeit.yml up -d

# Access container shell
docker exec -it snipeit bash

# Check disk space
df -h
docker system df
```

---

## ⚠️ Troubleshooting

**If asset.pupitech.me doesn't load:**
1. Check DNS propagation: `nslookup asset.pupitech.me`
2. Check containers: `docker ps`
3. Check Nginx logs: `docker logs snipeit-nginx`
4. Verify certificates: `ls -la /etc/ssl/cloudflare/`
5. Check firewall: `sudo ufw status`

**If you see SSL errors:**
- Verify Cloudflare SSL mode is "Full (strict)"
- Check certificate paths in docker-compose.yml
- Verify certificate files exist and have correct permissions

**Database connection issues:**
- Verify `facultyportfolio-db` container is running
- Check database credentials in .env
- Ensure both containers are on the same network

---

## 📚 Next Steps After Deployment
1. Complete Snipe-IT initial setup wizard
2. Configure email settings (optional)
3. Set up regular backups
4. Configure automatic updates (optional)
5. Review security settings

---

## 🔐 Security Recommendations
- Keep .env file secure (never commit to git)
- Use strong passwords for DB and Redis
- Regularly update Docker images
- Enable UFW firewall
- Monitor logs regularly
- Set up automated backups
