# 🚀 PANDUAN DEPLOY - Vercel (React) + Railway (Laravel)
## Tanggal: 26 Februari 2026

---

## 📁 Struktur Repository

```
hokojons/UMKM-DIGITAL-Railway-Verchel-
├── react/              ← Frontend React (Deploy ke Vercel)
│   ├── src/
│   ├── public/
│   ├── package.json
│   ├── vite.config.ts
│   └── ...
└── laravel/            ← Backend Laravel API (Deploy ke Railway)
    ├── app/
    ├── config/
    ├── database/
    ├── routes/
    ├── public/
    ├── storage/
    ├── composer.json
    └── ...
```

**GitHub Repository:** `https://github.com/hokojons/UMKM-DIGITAL-Railway-Verchel-`

---

## 🟢 BAGIAN 1: Deploy Laravel ke Railway

### Langkah 1: Buat Project di Railway

1. Buka [https://railway.app](https://railway.app)
2. Login dengan akun GitHub
3. Klik **"New Project"** → **"Deploy from GitHub Repo"**
4. Pilih repo: `hokojons/UMKM-DIGITAL-Railway-Verchel-`
5. Railway akan otomatis detect project

### Langkah 2: Set Root Directory

1. Di Railway dashboard, klik service yang baru dibuat
2. Buka tab **Settings**
3. Set **Root Directory** ke: `laravel`
4. Set **Start Command** ke:
   ```
   php artisan serve --host=0.0.0.0 --port=$PORT
   ```

### Langkah 3: Tambah Database MySQL

1. Di Railway project, klik **"+ New"** → **"Database"** → **"MySQL"**
2. Railway akan otomatis membuat database MySQL
3. Klik database MySQL → tab **"Connect"** → catat credentials:
   - `MYSQL_HOST`
   - `MYSQL_PORT`
   - `MYSQL_DATABASE`
   - `MYSQL_USER`
   - `MYSQL_PASSWORD`

### Langkah 4: Set Environment Variables

Di Railway, klik service Laravel → tab **"Variables"** → tambahkan:

```env
APP_NAME="UMKM Digital"
APP_ENV=production
APP_KEY=base64:f2lwoi7cCOb9srctJtpVrBsTf3lmbL+zu0qlJceOaao=
APP_DEBUG=false
APP_URL=https://[NAMA-SERVICE].up.railway.app

DB_CONNECTION=mysql
DB_HOST=${{MySQL.MYSQL_HOST}}
DB_PORT=${{MySQL.MYSQL_PORT}}
DB_DATABASE=${{MySQL.MYSQL_DATABASE}}
DB_USERNAME=${{MySQL.MYSQL_USER}}
DB_PASSWORD=${{MySQL.MYSQL_PASSWORD}}

SESSION_DRIVER=file
CACHE_STORE=file
QUEUE_CONNECTION=sync

NIXPACKS_PHP_VERSION=8.2
```

> ⚠️ Ganti `[NAMA-SERVICE]` dengan URL Railway yang diberikan setelah deploy.
> 💡 Gunakan format `${{MySQL.VARIABLE}}` untuk referensi otomatis ke database Railway.

### Langkah 5: Import Database

**Opsi A - Via Railway CLI:**
```bash
# Install Railway CLI
npm install -g @railway/cli

# Login
railway login

# Link project
railway link

# Jalankan migration
railway run php artisan migrate --seed
```

**Opsi B - Via phpMyAdmin (jika ada):**
1. Gunakan credentials MySQL dari Railway
2. Connect via MySQL client (DBeaver, TablePlus, dll)
3. Import file SQL

### Langkah 6: Jalankan Artisan Commands

Bisa via Railway CLI atau Railway shell:

```bash
railway run php artisan config:clear
railway run php artisan cache:clear
railway run php artisan route:clear
railway run php artisan storage:link
```

### Langkah 7: Verify Laravel

Buka URL Railway (contoh: `https://xxx.up.railway.app/api/`) → harus ada response JSON.

---

## 🔺 BAGIAN 2: Deploy React ke Vercel

### Langkah 1: Buat Project di Vercel

1. Buka [https://vercel.com](https://vercel.com)
2. Login dengan akun GitHub
3. Klik **"Add New..."** → **"Project"**
4. Import repo: `hokojons/UMKM-DIGITAL-Railway-Verchel-`

### Langkah 2: Configure Build Settings

Di halaman setup project:

| Setting | Value |
|---------|-------|
| **Framework Preset** | Vite |
| **Root Directory** | `react` |
| **Build Command** | `npm run build` |
| **Output Directory** | `dist` |
| **Install Command** | `npm install` |

### Langkah 3: Set Environment Variables

Tambahkan environment variables berikut di Vercel:

```env
VITE_API_BASE_URL=https://[RAILWAY-URL]/api
VITE_BASE_HOST=https://[RAILWAY-URL]
```

> ⚠️ Ganti `[RAILWAY-URL]` dengan URL Railway Laravel kamu (contoh: `umkm-production.up.railway.app`)

### Langkah 4: Deploy

1. Klik **"Deploy"**
2. Vercel akan build dan deploy otomatis
3. Setiap push ke `main` branch akan auto-deploy

### Langkah 5: Verify React

Buka URL Vercel (contoh: `https://xxx.vercel.app`) → harus muncul React app.

---

## 🔄 Update / Re-deploy

### Auto Deploy (Recommended)
Setiap kali push ke branch `main`, Vercel dan Railway akan **otomatis re-deploy**:

```bash
# Dari folder project lokal
git add -A
git commit -m "update: deskripsi perubahan"
git push railway-vercel main
```

### Manual Redeploy
- **Vercel:** Dashboard → Project → Deployments → Redeploy
- **Railway:** Dashboard → Service → Settings → Redeploy

---

## 🔗 Konfigurasi API URLs

File konfigurasi API ada di: `react/src/config/api.ts`

```typescript
export const API_BASE_URL = import.meta.env.VITE_API_BASE_URL || "http://localhost:8000/api";
export const BASE_HOST = import.meta.env.VITE_BASE_HOST || "http://localhost:8000";
```

Pastikan environment variables di Vercel sudah benar mengarah ke Railway URL.

---

## ⚠️ TROUBLESHOOTING

### Error 500 di API (Railway)
- Cek **Variables** di Railway dashboard sudah benar
- Buka Railway → Service → **Logs** untuk lihat error
- Jalankan: `railway run php artisan config:clear`
- Pastikan `APP_KEY` sudah di-set

### CORS Error di Browser
- Cek file `laravel/config/cors.php` → pastikan domain Vercel ada di allowed origins
- Atau set: `'allowed_origins' => ['*']` untuk development
- Jalankan: `railway run php artisan config:clear`

### Gambar Tidak Muncul
- Jalankan: `railway run php artisan storage:link`
- Pastikan folder `storage/app/public/uploads/` writable
- Cek apakah `BASE_HOST` di Vercel env mengarah ke Railway URL

### React Routing 404 (Vercel)
- Pastikan file `react/vercel.json` ada dengan isi:
  ```json
  {
    "rewrites": [
      { "source": "/(.*)", "destination": "/index.html" }
    ]
  }
  ```

### Database Connection Error (Railway)
- Pastikan MySQL service sudah running di Railway
- Cek apakah environment variables menggunakan format reference: `${{MySQL.MYSQL_HOST}}`
- Pastikan `DB_PORT` sesuai (Railway MySQL biasanya bukan 3306)

### Build Error di Vercel
- Pastikan **Root Directory** di-set ke `react`
- Pastikan `package.json` ada di folder `react/`
- Cek Vercel build logs untuk detail error

---

## 📞 Checklist Final

### Railway (Laravel)
- [ ] Project dibuat di Railway
- [ ] Root directory di-set ke `laravel`
- [ ] Start command: `php artisan serve --host=0.0.0.0 --port=$PORT`
- [ ] MySQL database ditambahkan
- [ ] Environment variables di-set (DB, APP_KEY, APP_URL)
- [ ] Database di-import / migration dijalankan
- [ ] `php artisan storage:link` dijalankan
- [ ] API bisa diakses via URL Railway ✅

### Vercel (React)
- [ ] Project dibuat di Vercel
- [ ] Root directory di-set ke `react`
- [ ] Build command: `npm run build`
- [ ] Output directory: `dist`
- [ ] Environment variables di-set (VITE_API_BASE_URL, VITE_BASE_HOST)
- [ ] `vercel.json` ada untuk SPA routing
- [ ] Website bisa diakses via URL Vercel ✅

### Koneksi
- [ ] React (Vercel) bisa call API ke Laravel (Railway)
- [ ] Gambar dari Laravel bisa dimuat di React
- [ ] Login/Register berfungsi
- [ ] CORS tidak error

---

## 📌 URLs Penting

| Service | URL |
|---------|-----|
| **GitHub Repo** | `https://github.com/hokojons/UMKM-DIGITAL-Railway-Verchel-` |
| **Vercel (React)** | *(isi setelah deploy)* |
| **Railway (Laravel)** | *(isi setelah deploy)* |
| **Railway MySQL** | *(internal, via environment variables)* |
