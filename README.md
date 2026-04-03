# Snipe-IT Docker Deployment

Docker Compose setup for Snipe-IT with Nginx reverse proxy and SSL support.

## Features
- 🐳 Docker Compose setup
- 🔒 SSL/TLS with Cloudflare Origin Certificate
- 🚀 Nginx reverse proxy
- 📦 Redis caching
- 🔄 Auto-restart containers

## Quick Start

See [DEPLOYMENT.md](DEPLOYMENT.md) for complete deployment instructions.

## Structure
```
.
├── docker-compose.snipeit.yml  # Main docker compose file
├── nginx.conf                   # Nginx reverse proxy config
├── .env.example                 # Environment variables template
├── .env                         # Your actual config (not in git)
└── DEPLOYMENT.md                # Step-by-step deployment guide
```

## Access
Once deployed: **https://asset.pupitech.me**

## Documentation
- [Snipe-IT Documentation](https://snipe-it.readme.io/docs)
- [Docker Hub - Snipe-IT](https://hub.docker.com/r/snipe/snipe-it)
