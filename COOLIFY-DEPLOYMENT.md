# OpenProject Coolify/Production Deployment

## Quick Start for Coolify

**Bạn chỉ cần MỘT domain duy nhất!** / **You only need ONE domain!**

This docker-compose file is ready for production deployment to Coolify, Portainer, or any Docker orchestration platform.

### What You Need

1. **ONE domain**: `openproject.yourdomain.com`
2. **Environment variables** (see below)
3. **At least 4GB RAM**

### Confusion About Multiple Domains?

The `docker-compose.yml` file in the root is for **development only**. It has separate services for frontend dev server, backend, worker, etc. **This is NOT what you need for production!**

For production deployment (Coolify), use `docker-compose.coolify.yml` which:
- Uses the official OpenProject production image
- Only exposes ONE port (80) for the web service
- All other services run internally
- **You only need ONE domain!**

### Quick Deploy to Coolify

1. In Coolify, create a new Docker Compose service
2. Use this file: `docker-compose.coolify.yml`
3. Set these environment variables:

```bash
OPENPROJECT_VERSION=17.1
DATABASE_PASSWORD=your-secure-password
SECRET_KEY_BASE=<generate with: openssl rand -hex 64>
OPENPROJECT_HOST__NAME=openproject.yourdomain.com
OPENPROJECT_HTTPS=true
OPENPROJECT_DEFAULT__LANGUAGE=vi  # or en, ja, etc.
```

4. Configure ONE domain in Coolify: `openproject.yourdomain.com`
5. Deploy!

### Full Documentation

See comprehensive guide with Vietnamese translation:
📖 [docs/installation-and-operations/installation/coolify/README.md](docs/installation-and-operations/installation/coolify/README.md)

### Services in Production Setup

The production compose file includes:
- **web** - Main application (port 80) - THIS is what you expose to your domain
- **worker** - Background jobs (no port exposed)
- **cron** - Scheduled tasks (no port exposed)
- **db** - PostgreSQL database (internal only)
- **cache** - Memcached (internal only)

All services communicate via internal Docker network. Only the **web** service needs your domain.

### Architecture

```
Internet
   ↓
Your ONE Domain (openproject.yourdomain.com)
   ↓
Web Container (OpenProject)
   ↓
Internal Docker Network
   ↓
┌─────────┬──────────┬────────┐
Worker    Database   Cache
(internal) (internal) (internal)
```

### Why Not Use Root docker-compose.yml?

The `docker-compose.yml` in the root directory is specifically for **local development**:
- Has separate frontend dev server on port 4200 (not needed for production)
- Has test services (not needed for production)
- Uses development builds (slower, larger)
- **Requires environment variable LOCAL_DEV_CHECK to be set**

The `docker-compose.coolify.yml` is for **production**:
- Uses official production images
- Pre-built frontend included
- Optimized for production use
- Smaller, faster, secure

### Support

- **English/Vietnamese Guide**: See [docs/installation-and-operations/installation/coolify/README.md](docs/installation-and-operations/installation/coolify/README.md)
- **OpenProject Community**: https://community.openproject.org/
- **Coolify Support**: https://coolify.io/discord

### License

GPL-3.0 - Same as OpenProject
