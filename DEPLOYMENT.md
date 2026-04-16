# Pupitech Unified Deployment (Snipe-IT & WordPress)

## 📋 Prerequisites
- VPS with SSH access
- Domains:
  - `asset.pupitech.me` (Snipe-IT)
  - `site.pupitech.me` or `pupitech.me` (WordPress)
- Cloudflare Origin Certificates in `/etc/ssl/cloudflare/`

---

## Deployment Steps

### Step 1: Prepare Environment
1. Copy `.env.example` to `.env`:
   ```bash
   cp .env.example .env
   ```
2. Generate a key for Snipe-IT:
   ```bash
   docker run --rm snipe/snipe-it php artisan key:generate --show
   ```
3. Update `.env` with the generated key and secure passwords.

### Step 2: Configure Cloudflare
Ensure your DNS records (A records) for `asset`, `site`, and `@` point to your VPS IP and are "Proxied" (Orange cloud).

### Step 3: Deploy
```bash
# Stop old services if running
docker-compose -f docker-compose.snipeit.yml down

# Start new unified setup
docker-compose up -d
```

### Step 4: Verify
- **Snipe-IT**: https://asset.pupitech.me
- **WordPress**: https://site.pupitech.me or https://pupitech.me

---

## 🔧 Maintenance
```bash
# View logs
docker-compose logs -f nginx
docker-compose logs -f wordpress

# Update images
docker-compose pull
docker-compose up -d
```

