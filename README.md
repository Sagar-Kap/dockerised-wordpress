# WordPress Docker with Caddy

Production-ready WordPress with automatic HTTPS using Docker and Caddy. Includes separate configurations for local development and production deployment.

## Features

- 🚀 One-command deployment
- 🔒 Automatic HTTPS with Let's Encrypt (production)
- 🐳 Fully containerized
- 🔧 Separate local and production configs
- 📦 Persistent data storage

## Prerequisites

- Docker (20.10.0+)
- Docker Compose (2.0.0+)
- A domain pointing to your server (production only)

## Quick Start

### Local Development

```bash
# 1. Clone and setup
git clone https://github.com/Sagar-Kap/dockerised-wordpress.git
cd dockerised-wordpress

# 2. Configure environment
cp .env.example .env
nano .env  # Use simple passwords for local

# 3. Start
docker compose -f docker-compose.local.yml up -d

# 4. Access at http://localhost:8080
```

### Production

```bash
# 1. Clone and setup
git clone https://github.com/Sagar-Kap/dockerised-wordpress.git
cd dockerised-wordpress

# 2. Configure environment
cp .env.example .env
nano .env  # Set DOMAIN, CADDY_EMAIL, and strong passwords

# 3. Verify DNS points to your server
nslookup yourdomain.com

# 4. Start
docker compose -f docker-compose.production.yml up -d

# 5. Access at https://yourdomain.com
```

## Project Structure

```
wordpress-docker-caddy/
├── docker-compose.local.yml        # Local development
├── docker-compose.production.yml   # Production
├── Caddyfile.local                 # HTTP on port 8080
├── Caddyfile.production            # HTTPS with auto SSL
└── .env.example                    # Environment template
```

## Local vs Production

| Feature          | Local                      | Production                      |
| ---------------- | -------------------------- | ------------------------------- |
| **Compose File** | `docker-compose.local.yml` | `docker-compose.production.yml` |
| **URL**          | http://localhost:8080      | https://yourdomain.com          |
| **SSL**          | ❌ No                      | ✅ Automatic                    |
| **Ports**        | 8080                       | 80, 443                         |

## Common Commands

### Local Development

```bash
# Start
docker compose -f docker-compose.local.yml up -d

# Stop
docker compose -f docker-compose.local.yml down

# View logs
docker compose -f docker-compose.local.yml logs -f

# Restart
docker compose -f docker-compose.local.yml restart
```

### Production

```bash
# Start
docker compose -f docker-compose.production.yml up -d

# Stop
docker compose -f docker-compose.production.yml down

# View logs
docker compose -f docker-compose.production.yml logs -f

# Update WordPress
docker compose -f docker-compose.production.yml pull
docker compose -f docker-compose.production.yml up -d
```

## Backup

### Database Backup

**Local:**

```bash
docker exec wordpress-mysql-local mysqldump -u root -p${MYSQL_ROOT_PASSWORD} \
  ${MYSQL_DATABASE} > backup-$(date +%Y%m%d).sql
```

**Production:**

```bash
docker exec wordpress-mysql mysqldump -u root -p${MYSQL_ROOT_PASSWORD} \
  ${MYSQL_DATABASE} > backup-$(date +%Y%m%d).sql
```

### Files Backup

**Local:**

```bash
docker cp wordpress-app-local:/var/www/html/wp-content ./wp-content-backup
```

**Production:**

```bash
docker cp wordpress-app:/var/www/html/wp-content ./wp-content-backup
```

## Troubleshooting

### SSL Certificate Issues (Production)

```bash
# Check Caddy logs
docker compose -f docker-compose.production.yml logs caddy

# Verify DNS
nslookup yourdomain.com

# Check ports are open
sudo netstat -tlnp | grep -E ':(80|443)'
```

### Can't Access Site

```bash
# Check services are running
docker compose -f docker-compose.local.yml ps  # or production.yml

# Restart all services
docker compose -f docker-compose.local.yml restart
```

### Database Connection Error

```bash
# Verify credentials in .env
cat .env

# Test connection
docker exec -it wordpress-mysql mysql -u ${MYSQL_USER} -p${MYSQL_PASSWORD}
```

### Permission Issues

```bash
# Fix WordPress permissions (use wordpress-app or wordpress-app-local)
docker exec wordpress-app chown -R www-data:www-data /var/www/html
```

## Environment Configuration

Example `.env` for **local development**:

```bash
DOMAIN=localhost
CADDY_EMAIL=dev@localhost
MYSQL_ROOT_PASSWORD=localroot
MYSQL_DATABASE=wordpress_dev
MYSQL_USER=wpuser
MYSQL_PASSWORD=wppass123
WORDPRESS_DB_HOST=mysql:3306
WORDPRESS_DB_USER=wpuser
WORDPRESS_DB_PASSWORD=wppass123
WORDPRESS_DB_NAME=wordpress_dev
```

Example `.env` for **production**:

```bash
DOMAIN=yourdomain.com
CADDY_EMAIL=admin@yourdomain.com
MYSQL_ROOT_PASSWORD=SuperSecure123!
MYSQL_DATABASE=wordpress_prod
MYSQL_USER=wordpress_user
MYSQL_PASSWORD=AnotherSecure456!
WORDPRESS_DB_HOST=mysql:3306
WORDPRESS_DB_USER=wordpress_user
WORDPRESS_DB_PASSWORD=AnotherSecure456!
WORDPRESS_DB_NAME=wordpress_prod
```

**Generate strong passwords:**

```bash
openssl rand -base64 32
```

## Migration from Existing WordPress

**1. Backup your current site:**

```bash
mysqldump -u user -p database > backup.sql
tar czf wp-content.tar.gz /path/to/wp-content
```

**2. Start Docker environment:**

```bash
cp .env.example .env
nano .env  # Configure
docker compose -f docker-compose.production.yml up -d
sleep 30
```

**3. Import database:**

```bash
docker cp backup.sql wordpress-mysql:/tmp/
docker exec -i wordpress-mysql mysql -u root -p${MYSQL_ROOT_PASSWORD} \
  ${MYSQL_DATABASE} < /tmp/backup.sql
```

**4. Restore files:**

```bash
docker compose -f docker-compose.production.yml stop wordpress
tar xzf wp-content.tar.gz
docker cp wp-content wordpress-app:/var/www/html/
docker exec wordpress-app chown -R www-data:www-data /var/www/html/wp-content
docker compose -f docker-compose.production.yml start wordpress
```

**5. Update URLs in database:**

```bash
docker exec -it wordpress-mysql mysql -u root -p${MYSQL_ROOT_PASSWORD} ${MYSQL_DATABASE}

UPDATE wp_options SET option_value = 'https://yourdomain.com' WHERE option_name = 'siteurl';
UPDATE wp_options SET option_value = 'https://yourdomain.com' WHERE option_name = 'home';
```

## Security Best Practices

1. Use strong passwords (20+ characters)
2. Keep `.env` file secure: `chmod 600 .env`
3. Update regularly: `docker compose -f docker-compose.production.yml pull`
4. Enable firewall:
   ```bash
   sudo ufw allow 80/tcp
   sudo ufw allow 443/tcp
   sudo ufw enable
   ```
5. Regular backups (automate with cron)

## Support

- **Issues:** [GitHub Issues](https://github.com/Sagar-Kap/dockerised-wordpress/issues)
- **Docker Docs:** https://docs.docker.com/
- **Caddy Docs:** https://caddyserver.com/docs/

---

**Made with ❤️ for easy WordPress deployment**
