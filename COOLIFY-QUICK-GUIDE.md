# OpenProject Coolify Deployment - Quick Visual Guide

## ❓ The Question / Câu hỏi

**Vietnamese:** "Tôi muốn cài đặt dự án này vào Coolify sử dụng /docker-compose.yml và muốn cài đặt phiên bản release 17.1. Bạn kiểm tra cho mình thực tế dự án cần tạo những domain nào vì mình thấy yêu cầu tạo khá nhiều domain cho backend, frontend, worker... Mình hỏi về sự cần thiết để cài đặt thành công."

**English:** "I want to install this project in Coolify using /docker-compose.yml and want to install version 17.1 release. Can you check for me what domains are actually needed for the project because I see it requires creating quite a few domains for backend, frontend, worker... I'm confused about what's necessary to install successfully."

---

## ✅ The Answer / Câu trả lời

### **You only need ONE domain!** / **Bạn chỉ cần MỘT domain!**

```
┌─────────────────────────────────────────────────────────────┐
│                         INTERNET                            │
│                            ↓                                │
│              https://openproject.yourdomain.com             │
│                    (YOUR ONE DOMAIN)                        │
└─────────────────────────────────────────────────────────────┘
                             ↓
┌─────────────────────────────────────────────────────────────┐
│                    Coolify Reverse Proxy                    │
│                    (Handles HTTPS/SSL)                      │
└─────────────────────────────────────────────────────────────┘
                             ↓
┌─────────────────────────────────────────────────────────────┐
│                      Web Container                          │
│          (OpenProject Application - Port 8080)              │
│         Complete App: Backend + Frontend Built-in           │
└─────────────────────────────────────────────────────────────┘
                             ↓
              Internal Docker Network
                (No external access)
                             ↓
        ┌────────────┬──────────────┬────────────┐
        ↓            ↓              ↓            ↓
   ┌────────┐  ┌─────────┐  ┌──────────┐  ┌─────────┐
   │ Worker │  │  Cron   │  │ Database │  │  Cache  │
   │Container  │Container │  │(Postgres)│  │(Memcache│
   └────────┘  └─────────┘  └──────────┘  └─────────┘
   Background   Scheduled      Data         Session
     Jobs        Tasks        Storage       Storage
```

---

## 🚫 What You DON'T Need / Những gì bạn KHÔNG cần

❌ **Backend domain** (backend.yourdomain.com) - NOT NEEDED  
❌ **Frontend domain** (frontend.yourdomain.com) - NOT NEEDED  
❌ **Worker domain** (worker.yourdomain.com) - NOT NEEDED  
❌ **Database domain** (db.yourdomain.com) - NOT NEEDED  
❌ **Cache domain** (cache.yourdomain.com) - NOT NEEDED

---

## 📋 What You DO Need / Những gì bạn CẦN

### 1. ONE Domain / MỘT Domain
```
openproject.yourdomain.com
```

### 2. Environment Variables / Biến môi trường
```bash
OPENPROJECT_VERSION=17.1
DATABASE_PASSWORD=your-secure-password
SECRET_KEY_BASE=<generate-this>
OPENPROJECT_HOST__NAME=openproject.yourdomain.com
OPENPROJECT_HTTPS=true
OPENPROJECT_DEFAULT__LANGUAGE=vi  # or en, ja, etc.
```

### 3. Resources / Tài nguyên
- 4GB RAM minimum
- 2 CPU cores
- 20GB disk space

---

## 🎯 Why the Confusion? / Tại sao có sự nhầm lẫn?

The `docker-compose.yml` in the repository root is for **DEVELOPMENT** only:

```
Development Setup (docker-compose.yml):
├── backend:4000      ← Dev server
├── frontend:4200     ← Dev server (Angular CLI)
├── worker            ← Background jobs
├── db                ← Database
└── cache             ← Cache

⚠️ This setup needs multiple ports exposed during development!
```

Production Setup (`docker-compose.coolify.yml`):

```
Production Setup (docker-compose.coolify.yml):
└── web:8080          ← Complete application (backend + pre-built frontend)
    ├── worker        ← Background jobs (internal)
    ├── cron          ← Scheduled tasks (internal)
    ├── db            ← Database (internal)
    └── cache         ← Cache (internal)

✅ Only ONE port (8080) exposed to your domain!
```

---

## 🚀 Quick Start / Bắt đầu nhanh

### Step 1: In Coolify
1. Create new Docker Compose service
2. Upload `docker-compose.coolify.yml`
3. Set environment variables (see `.env.coolify.example`)

### Step 2: Configure Domain
1. Add domain: `openproject.yourdomain.com`
2. Enable HTTPS (Let's Encrypt)
3. Set port: `8080`

### Step 3: Deploy
1. Click Deploy
2. Wait 2-5 minutes
3. Access: `https://openproject.yourdomain.com`
4. Login: admin/admin (change immediately!)

---

## 📚 Full Documentation / Tài liệu đầy đủ

- **Quick Guide:** `COOLIFY-DEPLOYMENT.md`
- **Full Guide (EN + VI):** `docs/installation-and-operations/installation/coolify/README.md`
- **Environment Variables:** `.env.coolify.example`

---

## 🆘 Common Issues / Vấn đề thường gặp

### "Container won't start"
- Check SECRET_KEY_BASE is set (at least 128 characters)
- Check DATABASE_PASSWORD is set
- Check OPENPROJECT_HOST__NAME matches your domain

### "Can't access the site"
- Verify DNS points to your Coolify server
- Check domain is configured in Coolify
- Ensure port 8080 is mapped correctly
- Check HTTPS is enabled

### "Email not working"
- Configure SMTP settings in environment variables
- See `.env.coolify.example` for examples

---

## 💡 Key Takeaway / Điểm chính

**The official OpenProject production images (`-slim` variant) contain EVERYTHING you need in a single container:**
- ✅ Backend (Rails API)
- ✅ Frontend (Pre-compiled Angular app)
- ✅ Web server (Puma)
- ✅ All assets and static files

**That's why you only need ONE domain!**

---

## 📞 Support / Hỗ trợ

- OpenProject Community: https://community.openproject.org/
- Coolify Discord: https://coolify.io/discord
- This Guide Issues: https://github.com/minhtuancn/openproject/issues
