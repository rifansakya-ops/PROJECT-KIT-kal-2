# Sistem Manajemen Material - PLN Nusa Daya

## 🎯 Project Overview

**Nama Aplikasi**: Sistem Manajemen Material PLN Nusa Daya  
**Tujuan**: Sistem manajemen inventori material untuk PLN Nusa Daya dengan fitur tracking material, perhitungan stok, monitoring umur material, dan pembuatan Berita Acara (BA) otomatis.

**Tech Stack**:
- **Backend**: Hono Framework (TypeScript)
- **Frontend**: Vanilla JavaScript + TailwindCSS
- **Database**: Cloudflare D1 (SQLite)
- **Deployment**: Cloudflare Pages
- **Process Manager**: PM2 (for development)

## 🌐 URLs

- **Development (Sandbox)**: https://3000-ix5mv84fvglwgtnfpo8yg-0e616f0a.sandbox.novita.ai
- **Local Development**: http://localhost:3000
- **GitHub**: (To be configured)
- **Production**: (To be deployed)

## ✨ Fitur Utama yang Sudah Diimplementasi

### 1. Form Input Material ✅
- **Informasi Umum**:
  - Tanggal transaksi
  - Jenis transaksi (Keluar/Masuk)
  - Lokasi asal dan tujuan
- **Penanggung Jawab**:
  - Dropdown Pemeriksa
  - Dropdown Penerima
- **Detail Material (Dynamic)**:
  - Autocomplete Part Number dengan pencarian real-time
  - Auto-fill: Jenis Barang, Material, dan Mesin setelah Part Number dipilih
  - Input manual: S/N Mesin dan Jumlah
  - Mendukung multiple material dalam satu transaksi

### 2. Dashboard Stok Material ✅
- Menampilkan stok berdasarkan Part Number
- Perhitungan: Total Masuk - Total Keluar = Stok
- Filter berdasarkan Tipe Mesin
- Color coding untuk status stok:
  - 🔴 Merah: Stok < 5 (Kritis)
  - 🟡 Kuning: Stok 5-9 (Perlu Perhatian)
  - 🟢 Hijau: Stok ≥ 10 (Aman)

### 3. Monitoring Umur Material ✅
- Tracking material berdasarkan S/N Mesin dan Part Number
- Perhitungan umur dalam hari sejak material dipasang
- Filter berdasarkan:
  - Part Number
  - S/N Mesin
  - Lokasi pemasangan
- Color coding untuk umur material:
  - 🔴 Merah: > 365 hari (Perlu Penggantian)
  - 🟡 Kuning: 180-365 hari (Monitoring)
  - 🟢 Hijau: < 180 hari (Normal)

### 4. Berita Acara (BA) ✅
- Pembuatan BA otomatis untuk setiap transaksi
- Nomor BA dengan format: `XXX/KIT/PLNND.DFKAL2/YYYY`
- Grouping: Material dengan transaksi bersamaan mendapat nomor BA yang sama
- Detail BA mencakup:
  - Nomor BA dan tanggal
  - Lokasi asal dan tujuan
  - Unit/Gudang
  - Daftar material lengkap
  - Pemeriksa dan Penerima
  - Template tanda tangan digital (siap untuk implementasi)

### 5. Riwayat Transaksi ✅
- Monitoring semua transaksi masuk dan keluar
- Filter berdasarkan tanggal
- Informasi lengkap per transaksi:
  - Nomor transaksi
  - Jenis transaksi (Keluar/Masuk)
  - Part Number dan Material
  - S/N Mesin
  - Jumlah
  - Nomor BA terkait

## 📊 Data Architecture

### Database Schema

**Tabel Utama**:

1. **materials** - Master data material
   - id, jenis_barang, part_number (unique), material, mesin, created_at
   - Total: 83 material dari Google Sheets

2. **locations** - Lokasi/Unit/Gudang
   - id, name (unique), type (UNIT/GUDANG), created_at
   - Total: 23 lokasi (BABAI, GUDANG BUNTOK, dll)

3. **users** - Pemeriksa dan Penerima
   - id, name (unique), role (PEMERIKSA/PENERIMA), created_at
   - Total: 14 users

4. **transactions** - Transaksi material
   - id, transaction_number, transaction_date, transaction_type
   - part_number, material_name, mesin, sn_mesin, quantity
   - location_from_id, location_to_id, pemeriksa_id, penerima_id
   - ba_number, notes, created_at

5. **berita_acara** - Berita Acara
   - id, ba_number (unique), transaction_number, ba_date
   - location_from, location_to, unit_gudang
   - pemeriksa_name, penerima_name
   - pdf_url, signature_pemeriksa, signature_penerima, created_at

6. **material_age** - Tracking umur material
   - id, part_number, sn_mesin, install_date, replacement_date
   - days_used, location_id, created_at

**Storage Service**: Cloudflare D1 (SQLite-based globally distributed database)

**Data Flow**:
1. User input material → Autocomplete from materials table
2. Submit transaksi → Insert to transactions + berita_acara tables
3. Dashboard Stok → Aggregate transactions by part_number
4. Umur Material → Calculate from transaction_date where sn_mesin matches

## 🚀 API Endpoints

### Materials
- `GET /api/materials` - Get all materials
- `GET /api/materials/search?q={query}` - Search materials (autocomplete)
- `GET /api/materials/:partNumber` - Get material by part number
- `GET /api/mesin-types` - Get unique mesin types

### Locations & Users
- `GET /api/locations` - Get all locations
- `GET /api/users` - Get all users
- `GET /api/users?role={PEMERIKSA|PENERIMA}` - Get users by role

### Transactions
- `POST /api/transactions` - Create new transaction
- `GET /api/transactions` - Get all transactions with filters
  - Query params: `mesin`, `location`, `start_date`, `end_date`

### Stock & Analytics
- `GET /api/stock` - Get stock summary by part number
  - Query param: `mesin` (optional filter)
- `GET /api/material-age` - Get material age tracking
  - Query params: `part_number`, `sn_mesin`, `location`

### Berita Acara
- `GET /api/berita-acara` - Get all Berita Acara
- `GET /api/berita-acara/:baNumber` - Get BA details with transactions

## 📖 User Guide

### 1. Input Material Baru

1. Buka tab **"Input Material"**
2. Isi **Informasi Umum**:
   - Pilih tanggal transaksi
   - Pilih jenis transaksi (Keluar/Masuk)
   - Pilih lokasi asal dan tujuan
3. Isi **Penanggung Jawab**:
   - Pilih Pemeriksa dari dropdown
   - Pilih Penerima dari dropdown
4. Isi **Detail Material**:
   - Ketik Part Number (akan muncul autocomplete)
   - Pilih material dari suggestion (auto-fill otomatis)
   - Isi S/N Mesin
   - Isi Jumlah
   - Klik "Tambah Baris Material" untuk material tambahan
5. Klik **"Simpan Transaksi"**
6. Sistem akan generate Nomor Transaksi dan Nomor BA otomatis

### 2. Cek Stok Material

1. Buka tab **"Stok Material"**
2. (Opsional) Filter berdasarkan Tipe Mesin
3. Lihat stok real-time dengan color coding:
   - Merah: Stok kritis (< 5)
   - Kuning: Stok rendah (5-9)
   - Hijau: Stok aman (≥ 10)

### 3. Monitor Umur Material

1. Buka tab **"Umur Material"**
2. Filter berdasarkan:
   - Part Number (opsional)
   - S/N Mesin (opsional)
   - Lokasi (opsional)
3. Klik "Cari" untuk melihat hasil
4. Sistem menampilkan umur material dalam hari dengan color coding

### 4. Lihat Berita Acara

1. Buka tab **"Berita Acara"**
2. Klik tombol **"Lihat"** pada BA yang ingin dilihat
3. Modal akan menampilkan:
   - Detail BA (nomor, tanggal, lokasi)
   - Daftar material lengkap
   - Penanggung jawab (Pemeriksa & Penerima)

### 5. Cek Riwayat Transaksi

1. Buka tab **"Riwayat"**
2. (Opsional) Filter berdasarkan rentang tanggal
3. Klik "Cari" untuk melihat riwayat transaksi

## 💻 Development Setup

### Prerequisites
- Node.js 18+ installed
- npm or yarn package manager
- Wrangler CLI (installed automatically)

### Local Development

```bash
# Navigate to project
cd /home/user/webapp

# Install dependencies (already installed)
npm install

# Build project
npm run build

# Migrate database (local SQLite)
npm run db:migrate:local

# Seed initial data
npm run db:seed

# Start development server
npm run dev:d1
# or with PM2 (recommended)
pm2 start ecosystem.config.cjs

# Access application
# Local: http://localhost:3000
# Sandbox: https://3000-ix5mv84fvglwgtnfpo8yg-0e616f0a.sandbox.novita.ai
```

### Database Management

```bash
# Reset database (drop all data and re-migrate)
npm run db:reset

# Execute SQL query (local)
npm run db:console:local

# Execute SQL query (production)
npm run db:console:prod

# Apply migrations to production
npm run db:migrate:prod
```

### PM2 Management

```bash
# List running services
pm2 list

# View logs (non-blocking)
pm2 logs --nostream

# Restart service
pm2 restart webapp

# Stop service
pm2 stop webapp

# Delete from PM2
pm2 delete webapp
```

## 📋 Fitur yang Belum Diimplementasi

1. **Export BA ke PDF** 📄
   - Library: jsPDF atau Puppeteer
   - Generate PDF dari template BA
   - Download/print functionality

2. **Digital Signature** ✍️
   - Canvas-based signature pad
   - Save signature sebagai base64/image
   - Store di database dan tampilkan di BA

3. **Notifikasi Email** 📧
   - Email notifikasi saat BA dibuat
   - Email reminder untuk material kritis
   - Integrasi dengan SendGrid/Mailgun

4. **Advanced Reporting** 📊
   - Chart.js untuk visualisasi stok
   - Report bulanan/tahunan
   - Export ke Excel

5. **User Authentication** 🔐
   - Login system dengan role-based access
   - Admin, Pemeriksa, Penerima roles
   - Session management

6. **Barcode/QR Code** 📲
   - Generate QR Code untuk Part Number
   - Scan untuk input cepat
   - Print label material

## 🎯 Next Development Steps (Rekomendasi)

### Priority 1 - Critical Features
1. ✅ **Material Age Calculation Enhancement**
   - Tambah fitur replacement tracking
   - History penggantian material per S/N Mesin

2. ✅ **BA PDF Generation**
   - Implement jsPDF untuk generate PDF
   - Template sesuai format PLN Nusa Daya
   - Download dan auto-save ke storage

3. ✅ **Digital Signature**
   - Canvas signature pad
   - Save dan display di BA

### Priority 2 - Enhancement
4. **Stock Alert System**
   - Browser notification untuk stok kritis
   - Dashboard alert panel

5. **Advanced Filters**
   - Date range picker yang lebih baik
   - Multi-select filters
   - Save filter preferences

6. **Data Export**
   - Export stok ke Excel/CSV
   - Export riwayat transaksi
   - Export BA batch

### Priority 3 - Nice to Have
7. **Mobile Responsive Optimization**
   - Touch-friendly interface
   - Mobile-first design improvements

8. **Multi-language Support**
   - Bahasa Indonesia (default)
   - English option

9. **Dashboard Analytics**
   - Chart visualizations
   - Trend analysis
   - Predictive analytics untuk kebutuhan material

## 🚀 Deployment Status

- **Development**: ✅ Active (Sandbox)
- **Staging**: ❌ Not Deployed
- **Production**: ❌ Not Deployed

### Deploy to Cloudflare Pages (Production)

```bash
# Setup Cloudflare API Key first
# Call setup_cloudflare_api_key tool

# Verify authentication
npx wrangler whoami

# Create D1 database (production)
npx wrangler d1 create webapp-production
# Copy database_id to wrangler.jsonc

# Create Cloudflare Pages project
npx wrangler pages project create webapp \\
  --production-branch main \\
  --compatibility-date 2024-01-01

# Build and deploy
npm run deploy:prod

# Apply migrations to production database
npm run db:migrate:prod

# Seed production data
npx wrangler d1 execute webapp-production --file=./seed.sql

# Access production URL
# https://webapp.pages.dev
```

## 📝 Notes

- Database menggunakan local SQLite untuk development (`--local` flag)
- Production menggunakan Cloudflare D1 (globally distributed)
- Semua fitur CRUD sudah berfungsi dengan baik
- Frontend menggunakan Vanilla JS untuk performa optimal
- No Node.js runtime dependencies (Cloudflare Workers compatible)

## 📞 Support & Contact

Untuk pertanyaan atau masalah, hubungi tim development PLN Nusa Daya.

---

**Last Updated**: 2025-12-07  
**Version**: 1.0.0  
**Status**: ✅ Development Active
