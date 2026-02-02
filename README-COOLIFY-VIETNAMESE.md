# Tóm tắt / Summary: OpenProject 17.1 Deployment to Coolify

## 📌 Câu trả lời trực tiếp cho câu hỏi của bạn / Direct Answer to Your Question

### Tiếng Việt

**Câu hỏi:** "Mình thấy yêu cầu tạo khá nhiều domain cho backend, frontend, worker... Mình hỏi về sự cần thiết để cài đặt thành công"

**Câu trả lời:**

🎯 **Bạn CHỈ CẦN MỘT domain duy nhất!**

Không cần domain riêng cho:
- ❌ Backend
- ❌ Frontend  
- ❌ Worker
- ❌ Database
- ❌ Cache

**Lý do tại sao có sự nhầm lẫn:**

File `docker-compose.yml` ở thư mục gốc là dành cho **phát triển (development)** chứ không phải sản xuất (production). Nó có nhiều service riêng biệt cho mục đích phát triển:

```
Development (KHÔNG dùng cho Coolify):
├── backend:3000      ← Server phát triển backend
├── frontend:4200     ← Server phát triển frontend (Angular CLI)
├── worker            ← Background jobs
├── db                ← Database
└── cache             ← Cache

⚠️ Đây là lý do tại sao bạn thấy "nhiều domain"!
```

**Cài đặt thực tế cho Coolify (Production):**

```
Production (SỬ DỤNG cho Coolify):
└── web:8080          ← Ứng dụng hoàn chỉnh (backend + frontend đã build sẵn)
    ├── worker        ← Background jobs (nội bộ, không cần domain)
    ├── cron          ← Scheduled tasks (nội bộ, không cần domain)
    ├── db            ← Database (nội bộ, không cần domain)
    └── cache         ← Cache (nội bộ, không cần domain)

✅ CHỈ CẦN expose 1 port (8080) cho domain của bạn!
```

---

### English

**Question:** "I see it requires creating quite a few domains for backend, frontend, worker... I'm confused about what's necessary to install successfully"

**Answer:**

🎯 **You only need ONE domain!**

No separate domains needed for:
- ❌ Backend
- ❌ Frontend
- ❌ Worker
- ❌ Database
- ❌ Cache

**Why the confusion:**

The `docker-compose.yml` in the repository root is for **development** only, not production. It has multiple separate services for development purposes:

```
Development (DON'T use for Coolify):
├── backend:3000      ← Backend dev server
├── frontend:4200     ← Frontend dev server (Angular CLI)
├── worker            ← Background jobs
├── db                ← Database
└── cache             ← Cache

⚠️ This is why you see "multiple domains"!
```

**Actual production setup for Coolify:**

```
Production (USE for Coolify):
└── web:8080          ← Complete application (backend + pre-built frontend)
    ├── worker        ← Background jobs (internal, no domain)
    ├── cron          ← Scheduled tasks (internal, no domain)
    ├── db            ← Database (internal, no domain)
    └── cache         ← Cache (internal, no domain)

✅ Only expose 1 port (8080) to your domain!
```

---

## 📋 Hướng dẫn nhanh / Quick Guide

### Bước 1: Chuẩn bị / Step 1: Prepare

1. **File cần dùng / File to use:**
   - ✅ `docker-compose.coolify.yml` (production)
   - ❌ KHÔNG dùng `docker-compose.yml` (development only)

2. **Domain cần thiết / Domain needed:**
   - Chỉ 1 domain: `openproject.yourdomain.com`
   - Only 1 domain: `openproject.yourdomain.com`

3. **Tài nguyên / Resources:**
   - RAM: 4GB tối thiểu / 4GB minimum
   - CPU: 2 cores
   - Disk: 20GB

### Bước 2: Cấu hình biến môi trường / Step 2: Configure Environment Variables

Tạo các biến sau trong Coolify / Set these variables in Coolify:

```bash
# Version OpenProject
OPENPROJECT_VERSION=17.1

# Mật khẩu database (QUAN TRỌNG: Đổi thành mật khẩu mạnh!)
# Database password (IMPORTANT: Change to strong password!)
DATABASE_PASSWORD=your-secure-password-here

# Secret key (Tạo bằng: openssl rand -hex 64)
# Secret key (Generate with: openssl rand -hex 64)
SECRET_KEY_BASE=your-secret-key-here

# Domain của bạn / Your domain
OPENPROJECT_HOST__NAME=openproject.yourdomain.com

# HTTPS (Coolify tự động cấu hình Let's Encrypt)
# HTTPS (Coolify auto-configures Let's Encrypt)
OPENPROJECT_HTTPS=true

# Ngôn ngữ mặc định / Default language
OPENPROJECT_DEFAULT__LANGUAGE=vi  # hoặc/or: en, ja, ko, etc.
```

### Bước 3: Triển khai trong Coolify / Step 3: Deploy in Coolify

1. Tạo service mới / Create new service → Docker Compose
2. Upload file `docker-compose.coolify.yml`
3. Thiết lập biến môi trường / Set environment variables (bước 2)
4. Cấu hình domain / Configure domain:
   - Domain: `openproject.yourdomain.com`
   - Port: `8080`
   - HTTPS: Enabled
5. Click "Deploy" / Click "Deploy"
6. Đợi 2-5 phút / Wait 2-5 minutes
7. Truy cập / Access: `https://openproject.yourdomain.com`
8. Đăng nhập / Login: `admin` / `admin` (đổi ngay!)

---

## 📚 Tài liệu chi tiết / Detailed Documentation

### Hướng dẫn ngắn / Quick Guides
1. **COOLIFY-DEPLOYMENT.md** - Hướng dẫn khởi động nhanh / Quick start guide
2. **COOLIFY-QUICK-GUIDE.md** - Sơ đồ trực quan / Visual diagrams
3. **DOCKER-COMPOSE-COMPARISON.md** - So sánh Development vs Production

### Hướng dẫn đầy đủ / Full Guide
4. **docs/installation-and-operations/installation/coolify/README.md** - Hướng dẫn song ngữ đầy đủ / Complete bilingual guide

### Cấu hình / Configuration
5. **docker-compose.coolify.yml** - File docker-compose cho production
6. **.env.coolify.example** - Mẫu biến môi trường / Environment variables template

---

## 🏗️ Kiến trúc / Architecture

```
                    Internet
                       ↓
        https://openproject.yourdomain.com
                 (YOUR DOMAIN)
                       ↓
              Coolify Reverse Proxy
                 (HTTPS/SSL)
                       ↓
                 Web Container
            (OpenProject:17.1-slim)
                   Port 8080
         Backend + Frontend đã build sẵn
         Backend + Pre-built Frontend
                       ↓
            Internal Docker Network
                       ↓
        ┌──────┬──────┬──────┬──────┐
        ↓      ↓      ↓      ↓      ↓
      Worker  Cron   DB   Cache
    (internal)(internal)(internal)(internal)
```

---

## ✅ Checklist cài đặt / Installation Checklist

### Trước khi bắt đầu / Before Starting
- [ ] Có domain và DNS đã trỏ đến server / Have domain and DNS pointing to server
- [ ] Có Coolify đang chạy / Have Coolify running
- [ ] Có ít nhất 4GB RAM / Have at least 4GB RAM
- [ ] Có ít nhất 20GB disk / Have at least 20GB disk space

### Chuẩn bị / Preparation
- [ ] Tạo mật khẩu database mạnh / Generate strong database password
- [ ] Tạo SECRET_KEY_BASE: `openssl rand -hex 64`
- [ ] Quyết định domain: `openproject.yourdomain.com`

### Trong Coolify / In Coolify
- [ ] Tạo Docker Compose service mới / Create new Docker Compose service
- [ ] Upload `docker-compose.coolify.yml`
- [ ] Thiết lập tất cả biến môi trường / Set all environment variables
- [ ] Cấu hình domain / Configure domain
- [ ] Enable HTTPS
- [ ] Deploy

### Sau khi deploy / After Deployment
- [ ] Truy cập domain / Access domain
- [ ] Đăng nhập admin/admin / Login with admin/admin
- [ ] Đổi mật khẩu admin ngay! / Change admin password immediately!
- [ ] Cấu hình SMTP (tuỳ chọn) / Configure SMTP (optional)
- [ ] Tạo backup / Setup backups

---

## 🔧 Cấu hình nâng cao / Advanced Configuration

### Email (Tuỳ chọn / Optional)

Thêm vào biến môi trường / Add to environment variables:

```bash
OPENPROJECT_EMAIL__DELIVERY__METHOD=smtp
OPENPROJECT_SMTP__ADDRESS=smtp.gmail.com
OPENPROJECT_SMTP__PORT=587
OPENPROJECT_SMTP__DOMAIN=yourdomain.com
OPENPROJECT_SMTP__USER__NAME=your-email@gmail.com
OPENPROJECT_SMTP__PASSWORD=your-app-password
OPENPROJECT_SMTP__ENABLE__STARTTLS__AUTO=true
```

### Tăng hiệu suất / Performance Tuning

Tùy thuộc vào RAM có sẵn / Based on available RAM:

```bash
# Mỗi worker dùng ~512MB RAM / Each worker uses ~512MB RAM
OPENPROJECT_WEB_WORKERS=2  # 4GB RAM: 2 workers
OPENPROJECT_WEB_WORKERS=4  # 8GB RAM: 4 workers
OPENPROJECT_WEB_WORKERS=6  # 12GB+ RAM: 6 workers
```

---

## 🆘 Xử lý sự cố / Troubleshooting

### Container không khởi động / Container won't start

Kiểm tra logs / Check logs:
```bash
docker compose logs web
```

Vấn đề thường gặp / Common issues:
- SECRET_KEY_BASE chưa được đặt hoặc quá ngắn / Not set or too short
- DATABASE_PASSWORD chưa được đặt / Not set
- Database chưa ready (đợi health check / wait for health check)
- Domain mismatch trong OPENPROJECT_HOST__NAME

### Không truy cập được / Can't access

- [ ] Kiểm tra DNS đã trỏ đúng / Verify DNS points correctly
- [ ] Kiểm tra domain trong Coolify / Check domain in Coolify
- [ ] Kiểm tra HTTPS đã bật / Ensure HTTPS enabled
- [ ] Kiểm tra port 80/443 mở / Check ports 80/443 open
- [ ] Kiểm tra logs của web container / Check web container logs

### Email không hoạt động / Email not working

- Cấu hình SMTP trong biến môi trường / Configure SMTP in environment variables
- Kiểm tra SMTP credentials / Verify SMTP credentials
- Xem ví dụ trong `.env.coolify.example`

---

## 💾 Backup và Restore

### Backup

```bash
# Database
docker exec openproject-db-1 pg_dump -U openproject openproject > backup.sql

# Files & assets
docker run --rm -v openproject_opdata:/data -v $(pwd):/backup ubuntu tar czf /backup/opdata-backup.tar.gz /data
```

### Restore

```bash
# Database
docker exec -i openproject-db-1 psql -U openproject openproject < backup.sql

# Files & assets
docker run --rm -v openproject_opdata:/data -v $(pwd):/backup ubuntu tar xzf /backup/opdata-backup.tar.gz -C /
```

---

## 📊 So sánh Development vs Production / Development vs Production Comparison

| Tiêu chí / Aspect | Development | Production |
|-------------------|-------------|------------|
| File | docker-compose.yml | docker-compose.coolify.yml |
| Domain cần / Domains needed | localhost:3000, :4200 | 1 domain only |
| Image | Custom dev build | openproject:17.1-slim |
| Size | ~2-3GB | ~500MB-1GB |
| Frontend | Dev server riêng / Separate | Đã build sẵn / Pre-built |
| Backend | Dev server riêng / Separate | Tích hợp / Integrated |
| Hiệu suất / Performance | Development | Optimized |
| Sử dụng / Use for | Local dev | Coolify, Production |

---

## 🎯 Kết luận / Conclusion

### Tiếng Việt

✅ **Để cài đặt OpenProject 17.1 trên Coolify:**
1. Sử dụng file `docker-compose.coolify.yml` (KHÔNG phải `docker-compose.yml`)
2. Chỉ cần MỘT domain duy nhất
3. Thiết lập biến môi trường theo hướng dẫn
4. Deploy và hoàn thành!

❌ **KHÔNG sử dụng `docker-compose.yml` cho production!**
- File đó dành cho development
- Sẽ gây nhầm lẫn về nhiều domain
- Không được tối ưu cho production

### English

✅ **To install OpenProject 17.1 on Coolify:**
1. Use `docker-compose.coolify.yml` (NOT `docker-compose.yml`)
2. Only need ONE domain
3. Set environment variables as instructed
4. Deploy and done!

❌ **DON'T use `docker-compose.yml` for production!**
- It's for development only
- Will confuse about multiple domains
- Not optimized for production

---

## 📞 Hỗ trợ / Support

- **OpenProject Community**: https://community.openproject.org/
- **Coolify Discord**: https://coolify.io/discord
- **Issues**: https://github.com/minhtuancn/openproject/issues

---

**Chúc bạn triển khai thành công! / Good luck with your deployment!** 🚀
