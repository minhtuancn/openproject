---
sidebar_navigation:
  title: Coolify Deployment
  priority: 350
---

# OpenProject Deployment to Coolify

This guide explains how to deploy OpenProject to [Coolify](https://coolify.io) using Docker Compose.

## Important: Domain Requirements (Câu trả lời cho câu hỏi của bạn)

### English Version

**You only need ONE domain for OpenProject deployment!**

The confusion about multiple domains (backend, frontend, worker) comes from the **development docker-compose.yml** file in this repository, which is NOT for production use.

#### What you actually need:

1. **Single Domain**: e.g., `openproject.yourdomain.com`
   - This is the only domain users will access
   - All services (web, worker, cron) run behind this single domain
   - Services communicate internally via Docker network

2. **No separate domains needed for**:
   - ❌ Backend - runs internally
   - ❌ Frontend - built and served by the web container
   - ❌ Worker - runs as background service (no HTTP exposure)
   - ❌ Cron - runs scheduled tasks (no HTTP exposure)
   - ❌ Database - runs internally
   - ❌ Cache - runs internally

#### Architecture Overview:

```
Internet → Your Domain (openproject.yourdomain.com) → Web Container
                                                        ↓
                                                Internal Network
                                                        ↓
                                    ┌──────────────────┼──────────────────┐
                                    ↓                  ↓                  ↓
                                  Worker            Database            Cache
                              (background jobs)    (PostgreSQL)      (Memcached)
```

---

### Phiên bản Tiếng Việt

**Bạn CHỈ CẦN MỘT domain duy nhất để cài đặt OpenProject!**

Sự nhầm lẫn về nhiều domain (backend, frontend, worker) xuất phát từ file **docker-compose.yml dành cho development** trong repository này, file đó KHÔNG dùng cho production.

#### Những gì bạn thực sự cần:

1. **Một Domain duy nhất**: ví dụ `openproject.yourdomain.com`
   - Đây là domain duy nhất mà người dùng sẽ truy cập
   - Tất cả các dịch vụ (web, worker, cron) chạy phía sau domain này
   - Các dịch vụ giao tiếp nội bộ qua mạng Docker

2. **KHÔNG cần domain riêng cho**:
   - ❌ Backend - chạy nội bộ
   - ❌ Frontend - được build và serve bởi web container
   - ❌ Worker - chạy như dịch vụ nền (không expose HTTP)
   - ❌ Cron - chạy các tác vụ định kỳ (không expose HTTP)
   - ❌ Database - chạy nội bộ
   - ❌ Cache - chạy nội bộ

#### Sơ đồ kiến trúc:

```
Internet → Domain của bạn (openproject.yourdomain.com) → Web Container
                                                            ↓
                                                    Mạng nội bộ
                                                            ↓
                                        ┌──────────────────┼──────────────────┐
                                        ↓                  ↓                  ↓
                                    Worker            Database            Cache
                                (background jobs)    (PostgreSQL)      (Memcached)
```

---

## Prerequisites

- Coolify instance running
- Domain name configured and pointing to your Coolify server
- At least 4GB RAM available
- 20GB disk space

## Deployment Steps

### Step 1: Create a New Service in Coolify

1. Log in to your Coolify dashboard
2. Create a new **Project** or select an existing one
3. Click **+ Add** → **Docker Compose**
4. Name your service (e.g., "OpenProject")

### Step 2: Configure the Docker Compose

Use the production-ready docker-compose file provided in this repository:

**File**: `docker-compose.coolify.yml`

This file includes:
- Web service (main application)
- Worker service (background jobs)
- Cron service (scheduled tasks)
- PostgreSQL database
- Memcached cache

### Step 3: Configure Environment Variables in Coolify

Set the following environment variables in Coolify:

#### Required Variables:

```bash
# OpenProject Version
OPENPROJECT_VERSION=17.1

# Database Password (IMPORTANT: Change this!)
DATABASE_PASSWORD=your-secure-password-here

# Secret Key Base (Generate with: openssl rand -hex 64)
SECRET_KEY_BASE=your-secret-key-base-here

# Domain Configuration
OPENPROJECT_HOST__NAME=openproject.yourdomain.com
OPENPROJECT_HTTPS=true

# Default Language (en, de, fr, es, pt, ru, zh, ja, ko, vi, etc.)
OPENPROJECT_DEFAULT__LANGUAGE=en
```

#### Optional Variables:

```bash
# Port (if you need custom port, default is 8080)
PORT=8080

# Web Workers (adjust based on available RAM)
OPENPROJECT_WEB_WORKERS=2

# Email Configuration (if you want to send emails)
OPENPROJECT_EMAIL__DELIVERY__METHOD=smtp
OPENPROJECT_SMTP__ADDRESS=smtp.example.com
OPENPROJECT_SMTP__PORT=587
OPENPROJECT_SMTP__AUTHENTICATION=plain
OPENPROJECT_SMTP__DOMAIN=example.com
OPENPROJECT_SMTP__USER__NAME=your-smtp-username
OPENPROJECT_SMTP__PASSWORD=your-smtp-password
OPENPROJECT_SMTP__ENABLE__STARTTLS__AUTO=true
```

### Step 4: Generate Required Secrets

Generate a secure `SECRET_KEY_BASE`:

```bash
openssl rand -hex 64
```

Or use any strong random string generator.

### Step 5: Configure Domain in Coolify

1. In Coolify service settings, go to **Domains**
2. Add your domain: `openproject.yourdomain.com`
3. Enable **HTTPS** (Let's Encrypt)
4. Set port to `8080` (the web container exposes port 8080)

### Step 6: Deploy

1. Click **Deploy** in Coolify
2. Wait for the deployment to complete (first deployment takes 2-5 minutes)
3. Check logs if there are any issues

### Step 7: Access OpenProject

1. Visit your domain: `https://openproject.yourdomain.com`
2. Default credentials:
   - Username: `admin`
   - Password: `admin`
3. **IMPORTANT**: Change the admin password immediately!

## Configuration

### Resource Requirements

Recommended minimum resources:
- **RAM**: 4GB (2GB for application, 1GB for database, 1GB for system)
- **CPU**: 2 cores
- **Disk**: 20GB minimum (grows with attachments and data)

### Scaling

You can scale the worker service in production:

```yaml
worker:
  deploy:
    replicas: 2  # Add more workers for heavy workloads
```

### Backup

Backup the following volumes:
- `pgdata` - PostgreSQL database
- `opdata` - OpenProject attachments and assets

In Coolify, you can use the built-in backup feature or manual Docker commands:

```bash
# Backup database
docker exec openproject-db-1 pg_dump -U openproject openproject > backup.sql

# Backup assets
docker run --rm -v openproject_opdata:/data -v $(pwd):/backup ubuntu tar czf /backup/opdata-backup.tar.gz /data
```

### Restore

```bash
# Restore database
docker exec -i openproject-db-1 psql -U openproject openproject < backup.sql

# Restore assets
docker run --rm -v openproject_opdata:/data -v $(pwd):/backup ubuntu tar xzf /backup/opdata-backup.tar.gz -C /
```

## Upgrading

To upgrade OpenProject:

1. Update the `OPENPROJECT_VERSION` environment variable to the new version
2. Click **Deploy** in Coolify
3. The upgrade will run automatically
4. Check logs for any migration messages

**Note**: Always backup before upgrading!

## Troubleshooting

### Container keeps restarting

Check logs in Coolify:
```bash
docker compose logs web
```

Common issues:
- `SECRET_KEY_BASE` not set or too short
- Database not ready (wait for health check)
- Domain mismatch in `OPENPROJECT_HOST__NAME`

### Cannot access the application

- Verify domain is correctly configured in Coolify
- Check DNS records point to your server
- Ensure HTTPS is enabled
- Check if port 80/443 are open in firewall

### Email not working

Configure SMTP settings in environment variables (see Optional Variables above).

### Performance issues

- Increase `OPENPROJECT_WEB_WORKERS` (each worker uses ~512MB RAM)
- Ensure you have enough RAM available
- Consider using external PostgreSQL for better performance

## Differences from Development Setup

The `docker-compose.yml` in the repository root is for **development only** and includes:
- Separate frontend dev server (port 4200) - NOT needed for production
- Test services - NOT needed for production
- Development-specific settings - NOT needed for production

The production setup (`docker-compose.coolify.yml`) uses the official OpenProject image which:
- Contains pre-built frontend assets
- Runs production-optimized web server
- Uses proper background job processing
- Includes all necessary services in one efficient setup

## Additional Resources

- [OpenProject Official Documentation](https://www.openproject.org/docs/)
- [OpenProject Docker Documentation](../docker/)
- [OpenProject Configuration Guide](https://www.openproject.org/docs/installation-and-operations/configuration/)
- [Coolify Documentation](https://coolify.io/docs)

## Support

For issues specific to:
- **OpenProject**: [OpenProject Community](https://community.openproject.org/)
- **Coolify**: [Coolify Discord](https://coolify.io/discord)
- **This Deployment Guide**: Open an issue in this repository

## License

This deployment guide is part of OpenProject and follows the same GPL-3.0 license.
