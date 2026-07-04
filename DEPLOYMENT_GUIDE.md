# 🚀 Polymarket Trading Bot - One-Click Deployment Guide

## Quick Start (5 Minutes)

### Step 1: Prerequisites
```bash
# Check Docker installation
docker --version
docker-compose --version

# If not installed:
# macOS: brew install docker docker-compose
# Linux: sudo apt-get install docker.io docker-compose
# Windows: Download Docker Desktop
```

### Step 2: Configuration
```bash
# Clone repository
git clone <repo-url>
cd polymarket-bot

# Copy environment template
cp .env.example .env

# Edit with your values
# nano .env  (or vi, code, etc.)

# Required values:
# - POLYGON_RPC_URL: https://polygon-rpc.com
# - WALLET_ADDRESS: Your Polygon wallet (0x...)
# - PRIVATE_KEY: Your private key (0x... WITHOUT this, many features won't work)
# - POLYMARKET_API_KEY: Request from Polymarket API docs
```

### Step 3: Launch (One Command!)
```bash
# Start everything with Docker Compose
docker-compose up -d

# Wait 30 seconds for services to initialize
sleep 30

# Open browser
open http://localhost:8501
# or navigate to http://localhost:8501
```

### Step 4: Access Dashboard
- **Streamlit App**: http://localhost:8501
- **Grafana Dashboards**: http://localhost:3000 (admin/admin)
- **Prometheus Metrics**: http://localhost:9090

That's it! 🎉

---

## Troubleshooting

### Services won't start
```bash
# Check logs
docker-compose logs app

# Restart everything
docker-compose restart

# Full reset
docker-compose down -v
docker-compose up -d
```

### Connection errors
```bash
# Verify services are running
docker-compose ps

# Check network connectivity
docker-compose exec app curl -I https://api.polymarket.com

# Verify database connection
docker-compose exec app python -c "from data.db import get_db; print('DB OK')"
```

### High memory usage
```bash
# Check resource limits
docker stats

# Reduce cache TTL in .env
REDIS_CACHE_TTL=1800  # 30 minutes instead of 1 hour

# Restart with resource limits
docker-compose down
docker-compose up -d
```

---

## Production Deployment

### Cloud Deployment (AWS)

```bash
# Using AWS ECS
aws ecs create-cluster --cluster-name polymarket-bot

# Create task definition
aws ecs register-task-definition --cli-input-json file://task-definition.json

# Run service
aws ecs create-service --cluster polymarket-bot \
  --service-name polymarket-bot \
  --task-definition polymarket-bot:1 \
  --desired-count 1
```

### Kubernetes Deployment

```bash
# Build and push image
docker build -t your-registry/polymarket-bot:latest .
docker push your-registry/polymarket-bot:latest

# Deploy to Kubernetes
kubectl create namespace polymarket
kubectl apply -f k8s/deployment.yaml -n polymarket
kubectl apply -f k8s/service.yaml -n polymarket
```

### Manual VPS Deployment

```bash
# SSH into VPS
ssh user@your-vps.com

# Install Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Clone repo and start
git clone <repo-url>
cd polymarket-bot
cp .env.example .env
# Edit .env with your credentials
docker-compose up -d

# Enable auto-start
sudo systemctl enable docker
```

---

## Monitoring & Maintenance

### View Logs
```bash
# Real-time logs
docker-compose logs -f app

# Last 100 lines
docker-compose logs --tail=100 app

# Export logs
docker-compose logs app > logs_backup.txt
```

### Health Checks
```bash
# Check all services
docker-compose ps

# Check specific service
docker-compose exec app curl http://localhost:8501/_stcore/health

# Database health
docker-compose exec postgres pg_isready -U postgres

# Redis health
docker-compose exec redis redis-cli ping
```

### Backup Data
```bash
# Backup PostgreSQL
docker-compose exec postgres pg_dump -U postgres polymarket_bot > backup.sql

# Backup Redis
docker-compose exec redis redis-cli BGSAVE

# Restore backup
docker-compose exec postgres psql -U postgres polymarket_bot < backup.sql
```

### Update Bot
```bash
# Pull latest changes
git pull origin main

# Rebuild images
docker-compose build

# Restart services
docker-compose restart

# Or full restart
docker-compose down
docker-compose up -d
```

---

## Performance Tuning

### Optimize for Low Latency
```yaml
# In docker-compose.yml
cap_add:
  - NET_ADMIN
environment:
  - TCP_BUFFER_SIZE=4194304,6291456,8388608
```

### Increase Connection Pool
```bash
# In .env
DB_POOL_SIZE=50
DB_MAX_OVERFLOW=100
```

### Enable Caching
```bash
# In .env
REDIS_CACHE_TTL=7200  # 2 hours
CACHE_COMPRESSION=true
```

### Scale Horizontally
```bash
# Add load balancer
docker-compose up -d --scale app=3

# Use Nginx for load balancing
# See nginx.conf for configuration
```

---

## Security Hardening

### 1. Secure Environment Variables
```bash
# Use AWS Secrets Manager
aws secretsmanager create-secret --name polymarket-bot --secret-string file://secrets.json

# Use HashiCorp Vault
vault write secret/polymarket-bot @secrets.json
```

### 2. Network Security
```bash
# Add firewall rules
sudo ufw allow 8501/tcp  # Streamlit
sudo ufw allow 5432/tcp  # PostgreSQL (internal only)
sudo ufw allow 6379/tcp  # Redis (internal only)

# Restrict access
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

### 3. API Key Rotation
```bash
# Automate key rotation
./scripts/rotate-keys.sh

# Store keys encrypted
gpg -c .env

# Use key management service
export AWS_KMS_KEY_ID=arn:aws:kms:region:account:key/id
```

### 4. SSL/TLS
```bash
# Generate self-signed cert
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 365

# Use Let's Encrypt
sudo apt-get install certbot
sudo certbot certonly --standalone -d yourdomain.com
```

---

## Cost Optimization

### Reduce Compute
```bash
# Use spot instances (30-70% cheaper)
docker-compose -f docker-compose.spot.yml up -d

# Schedule downtime
# Use cron to stop bot during low liquidity hours
0 0 * * * docker-compose stop  # 00:00 UTC
0 8 * * * docker-compose start  # 08:00 UTC
```

### Database Optimization
```bash
# Enable connection pooling
PGBOUNCER_ENABLED=true

# Archive old trades
./scripts/archive-trades.sh

# Optimize indexes
docker-compose exec postgres psql -U postgres -d polymarket_bot -f optimize.sql
```

### CDN & Caching
```bash
# Use CloudFlare for DDoS protection
# Cache API responses
CACHE_TTL=3600

# Compress responses
COMPRESSION_ENABLED=true
COMPRESSION_LEVEL=6
```

---

## Rollback Procedure

### If Something Goes Wrong
```bash
# Stop current version
docker-compose stop

# Checkout previous version
git checkout HEAD~1

# Rebuild and start
docker-compose build
docker-compose up -d

# Monitor logs
docker-compose logs -f app

# If still broken, full rollback
docker system prune -a
git reset --hard HEAD~1
docker-compose up -d
```

---

## Support & Help

- 📖 Full Documentation: See README.md
- 🐛 Report Issues: GitHub Issues
- 💬 Community: Discord Server
- 📧 Email Support: support@polymarketbot.dev

---

## Cheat Sheet

```bash
# One-liners
docker-compose up -d                    # Start everything
docker-compose down                     # Stop everything
docker-compose logs -f app              # Watch logs
docker-compose ps                       # See status
docker-compose restart                  # Restart services
docker-compose exec app bash            # Shell access
docker-compose pull                     # Update images
docker system prune -a                  # Clean up
make docker-up                          # Using Makefile
make help                               # See all commands
```

Good luck! 🚀
