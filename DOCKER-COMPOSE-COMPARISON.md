# Development vs Production Docker Compose Comparison

## File Purpose Comparison

| File | Purpose | Use Case | Domains Needed |
|------|---------|----------|----------------|
| `docker-compose.yml` | **Development** | Local development, requires LOCAL_DEV_CHECK env var | Multiple ports for dev servers |
| `docker-compose.coolify.yml` | **Production** | Coolify, Portainer, production deployments | **ONE domain only** |

## Architecture Comparison

### Development Setup (`docker-compose.yml`)

```yaml
services:
  backend:           # Rails server (port 3000)
  frontend:          # Angular dev server (port 4200) - HOT RELOAD
  worker:            # Background jobs
  db:                # PostgreSQL
  cache:             # Memcached
  # + test services
```

**Characteristics:**
- ❌ Uses development images
- ❌ Frontend dev server with hot reload
- ❌ Larger image sizes
- ❌ Slower performance
- ❌ Requires building from source
- ✅ Good for development

**Port Exposure:**
```
localhost:3000  → Backend API
localhost:4200  → Frontend Dev Server (Angular CLI)
```

---

### Production Setup (`docker-compose.coolify.yml`)

```yaml
services:
  web:               # Complete app (port 8080) - PRODUCTION
  worker:            # Background jobs (no port)
  cron:              # Scheduled tasks (no port)
  db:                # PostgreSQL (internal)
  cache:             # Memcached (internal)
```

**Characteristics:**
- ✅ Uses production-optimized slim images
- ✅ Pre-compiled frontend assets included
- ✅ Smaller image sizes
- ✅ Better performance
- ✅ Ready to deploy
- ✅ Production-ready

**Port Exposure:**
```
your-domain.com → Web Container (port 8080)
                  └─ Contains: Backend API + Pre-built Frontend
```

---

## Image Comparison

### Development Image
```dockerfile
FROM custom development build
- Source code mounted as volume
- Live code reloading
- Development dependencies included
- ~2-3GB size
```

### Production Image (`openproject/openproject:17.1-slim`)
```dockerfile
FROM official production image
- Pre-compiled application
- Production dependencies only
- Optimized for performance
- ~500MB-1GB size
```

---

## Environment Variables Comparison

### Development (`docker-compose.yml`)
```bash
# Required
LOCAL_DEV_CHECK=true          # Must be set!
RAILS_ENV=development
DEV_UID=1000
DEV_GID=1000

# Database
DB_USERNAME=postgres
DB_PASSWORD=postgres
DB_HOST=db
```

### Production (`docker-compose.coolify.yml`)
```bash
# Required
OPENPROJECT_VERSION=17.1
DATABASE_PASSWORD=secure-password
SECRET_KEY_BASE=long-random-string
OPENPROJECT_HOST__NAME=your-domain.com
OPENPROJECT_HTTPS=true
```

---

## Service Details

### Web Service

#### Development
```yaml
backend:
  ports:
    - "3000:3000"    # Rails server
frontend:
  ports:
    - "4200:4200"    # Angular dev server
```
**Purpose:** Separate backend and frontend for development with hot reload

#### Production
```yaml
web:
  ports:
    - "8080:8080"    # Complete application
```
**Purpose:** Single unified application server

---

### Worker Service

#### Development
```yaml
worker:
  command: bundle exec good_job start
  # No port exposure
```

#### Production
```yaml
worker:
  image: openproject/openproject:17.1-slim
  command: "./docker/prod/worker"
  # No port exposure
```
**Same purpose, optimized image**

---

### Database

#### Development
```yaml
db:
  image: postgres:17
  environment:
    POSTGRES_USER: postgres
    POSTGRES_PASSWORD: postgres
```

#### Production
```yaml
db:
  image: postgres:17
  environment:
    POSTGRES_USER: openproject
    POSTGRES_PASSWORD: ${DATABASE_PASSWORD}
```
**Same, but with secure password management**

---

## Network Configuration

### Development
```yaml
networks:
  network:          # Internal network
  testing:          # Additional network for tests
```
**Multiple networks for development and testing**

### Production
```yaml
networks:
  openproject:      # Single internal network
```
**Simple, secure internal network**

---

## Volume Configuration

### Development
```yaml
volumes:
  pgdata:           # Database
  opdata:           # Assets
  bundle:           # Ruby gems
  npm:              # Node modules
  tmp:              # Temporary files
  # + test volumes
```
**Many volumes for development needs**

### Production
```yaml
volumes:
  pgdata:           # Database
  opdata:           # Assets only
```
**Minimal volumes for production data**

---

## Health Checks

### Development
```yaml
# Usually minimal or no health checks
```

### Production
```yaml
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost:8080/health_checks/default"]
  interval: 30s
  timeout: 10s
  retries: 3
  start_period: 90s
```
**Comprehensive health monitoring**

---

## Deployment Differences

### Development Deployment
1. Clone repository
2. Set LOCAL_DEV_CHECK=true
3. Run `docker-compose up`
4. Access backend and frontend separately
5. Code changes reflect immediately

### Production Deployment (Coolify)
1. Use docker-compose.coolify.yml
2. Set production environment variables
3. Configure ONE domain in Coolify
4. Deploy
5. Access via single domain

---

## When to Use Which

### Use `docker-compose.yml` (Development)
- ✅ Local development
- ✅ Testing changes with hot reload
- ✅ Debugging
- ✅ Contributing to OpenProject

### Use `docker-compose.coolify.yml` (Production)
- ✅ Coolify deployment
- ✅ Portainer deployment
- ✅ Production hosting
- ✅ Staging environments
- ✅ When you need ONE domain

---

## Migration Path

If you tried to use `docker-compose.yml` for production:

### Problems you'll encounter:
1. ❌ Need to set LOCAL_DEV_CHECK (error message)
2. ❌ Development images are larger
3. ❌ Slower performance
4. ❌ Multiple services to manage
5. ❌ Not optimized for production

### Solution:
Switch to `docker-compose.coolify.yml`:
1. ✅ Production-ready images
2. ✅ ONE domain configuration
3. ✅ Optimized performance
4. ✅ Simpler management
5. ✅ Better security

---

## Summary Table

| Aspect | Development | Production |
|--------|-------------|------------|
| **File** | docker-compose.yml | docker-compose.coolify.yml |
| **Image** | Custom dev build | openproject/openproject:17.1-slim |
| **Domains** | localhost:3000, localhost:4200 | ONE domain |
| **Size** | ~2-3GB | ~500MB-1GB |
| **Services** | 5+ services | 5 services (1 exposed) |
| **Ports** | Multiple (3000, 4200) | One (8080) |
| **Performance** | Development | Optimized |
| **Hot Reload** | Yes | No (not needed) |
| **Use Case** | Local dev | Production hosting |
| **Complexity** | High (for development) | Low (for deployment) |

---

## Conclusion

**For Coolify deployment:**
- ✅ Use `docker-compose.coolify.yml`
- ✅ Configure ONE domain
- ✅ Set environment variables
- ✅ Deploy and enjoy!

**Don't use `docker-compose.yml` for production!**
