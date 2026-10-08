# Sesuatu DariKota Malang

> Platform web + mobile untuk toko oleh-oleh artisan di kawasan **Kayutangan Heritage, Malang**.
> Pickup on-site, storytelling seniman lokal, dan portal konsinyasi — bukan marketplace pengiriman.

<p>
  <strong>Versi 0.9.0</strong> — Masih dalam tahap pengembangan & finalisasi.<br>
  <em>Semua fitur utama sudah diimplementasi pada frontend; integrasi backend produksi sedang berjalan.</em>
</p>

[![Version](https://img.shields.io/badge/version-0.9.0--dev-orange?style=flat-square)](#status)
[![Next.js](https://img.shields.io/badge/Next.js-16-black?style=flat-square&logo=next.js)](https://nextjs.org/)
[![Flutter](https://img.shields.io/badge/Flutter-3.22%2B-blue?style=flat-square&logo=flutter)](https://flutter.dev/)
[![TypeScript](https://img.shields.io/badge/TypeScript-5-3178c6?style=flat-square&logo=typescript)](https://www.typescriptlang.org/)
[![Tailwind CSS](https://img.shields.io/badge/Tailwind-4-38bdf8?style=flat-square&logo=tailwindcss)](https://tailwindcss.com/)
[![Prisma](https://img.shields.io/badge/Prisma-6-2d3748?style=flat-square&logo=prisma)](https://www.prisma.io/)
[![License](https://img.shields.io/badge/license-MIT-green?style=flat-square)](#license)

---

## Daftar Isi

- [Tentang Project](#tentang-project)
- [Arsitektur](#arsitektur)
- [Fitur Utama](#fitur-utama)
- [Screenshots](#screenshots)
- [Daftar Library](#daftar-library)
- [Tech Stack](#tech-stack)
- [Struktur Repo](#struktur-repo)
- [Quick Start](#quick-start)
- [Environment Variables](#environment-variables)
- [Akun Demo](#akun-demo)
- [Design System](#design-system)
- [Aturan Bisnis Kunci](#aturan-bisnis-kunci)
- [Integrasi AI — Bantu Tulis Behind the Design](#integrasi-ai--bantu-tulis-behind-the-design)
- [Skema Database](#skema-database)
- [Figma Export Utilities](#figma-export-utilities)
- [Roadmap](#roadmap)
- [Tim](#tim)
- [Status](#status)
- [License](#license)

---

## Tentang Project

**Sesuatu DariKota Malang** adalah platform digital untuk toko oleh-oleh artisan di Jl. Jenderal Basuki Rahmat No. 45, Kawasan Kayutangan Heritage, Malang. Toko menjual karya seniman lokal (postcard, pin enamel, apparel, kriya kayu, tote bag, dll) dengan sistem **konsinyasi** (titip jual).

Project ini adalah **tugas kuliah semester 3** (tim 3 orang, satu semester) yang mencakup desain UI/UX (Figma), web (admin panel), dan mobile app (customer & creator).

### Kenapa Butuh Platform Sendiri?

- **Pembagian peran:** Shopee/Tokopedia dipakai khusus pengiriman luar kota. Platform sendiri fokus ke **pickup on-site**, **storytelling seni**, dan **portal kemitraan konsinyasi**.
- **Storytelling:** Narasi "Behind the Design" + profil kreator tidak terlayani di marketplace generik.
- **Kemitraan terstruktur:** Pendaftaran kreator baru lewat form, bukan chat WhatsApp berantakan.

---

## Arsitektur

```
                ┌──────────────────────────────────────┐
                │       Single Source of Truth          │
                │     (Dokumen Master Project)          │
                └────────────────┬─────────────────────┘
                                 │
                ┌────────────────┴────────────────┐
                ▼                                  ▼
   ┌────────────────────────┐         ┌────────────────────────┐
   │   Web (Next.js 16)     │         │   Mobile (Flutter)     │
   │   ── Panel Admin ──    │         │ ── Customer (buyer +   │
   │                        │         │     creator) ──        │
   │  • Dashboard + Charts  │         │  • Home & Katalog      │
   │  • CRUD Produk & Varian│         │  • Detail Produk + BTD │
   │  • Kelola Pesanan      │         │  • Reservasi + QR      │
   │  • Scan QR Pickup      │         │  • Meet The Artisans   │
   │  • Kurasi Konsinyasi   │         │  • Dashboard Kreator   │
   │  • Moderasi Ulasan     │         │  • Akun & Notifikasi   │
   │  • Pengaturan Sistem   │         │  • Bantu Tulis AI      │
   └────────────┬───────────┘         └────────────┬───────────┘
                │                                  │
                └────────────────┬─────────────────┘
                                 │
                                 ▼
                ┌──────────────────────────────────────┐
                │     Backend (Express.js + Prisma)    │
                │  • REST API                          │
                │  • Gemini API integration           │
                │  • Midtrans Snap payment             │
                │  • Resend email OTP                  │
                │  • Google OAuth verification         │
                └────────────────┬─────────────────────┘
                                 │
                                 ▼
                ┌──────────────────────────────────────┐
                │      Database (MySQL + Prisma)       │
                │  14 tabel — users, products, orders, │
                │  variants, reviews, notifications…   │
                └──────────────────────────────────────┘
```

**Pembagian platform (final):**

| Platform | Cakupan | Pengguna |
|---|---|---|
| **Web (Next.js)** | Panel admin saja | Admin toko |
| **Mobile (Flutter)** | Seluruh pengalaman customer — buyer + dashboard kreator | Buyer & Creator |

---

## Fitur Utama

### Customer (Mobile Flutter)

- **Guest browse bebas** — katalog, detail produk (termasuk "Behind the Design" & review), profil kreator, semua bisa diakses tanpa login.
- **AuthGate modal** — guest yang klik aksi butuh-login (pesan, ajukan konsinyasi, ulasan) dapat dialog ajakan login, bukan redirect diam-diam.
- **Katalog + Rekomendasi** — filter kategori, sort (terbaru/termurah/termahal/terlaris), rekomendasi rule-based di landing & "Produk Serupa" di detail.
- **Varian 2D dinamis** — multi-sumbu (Motif x Ukuran), label bebas, **stok per kombinasi** (bukan per produk).
- **Reservasi pickup 3-step** — pilih varian + jadwal + metode bayar, lalu terbit kode `SDK-XXXX` + QR.
- **Check-at-submit** — stok tidak di-lock saat browsing; cek dilakukan saat klik "Pesan Sekarang".
- **Dual payment cycle:**
  - **Midtrans (online)** → langsung Lunas, stok dikurangi permanen, tidak ada auto-cancel.
  - **Cash/QRIS di toko** → status `Menunggu Bayar`, stok di-hold 2 jam, auto-cancel bisa diaktifkan admin.
- **QR pickup** — kasir scan QR lewat kamera browser di HP (bukan app native terpisah).
- **Hub Akun** — profil + menu ke Riwayat Pesanan, Ulasan Saya, Status Pengajuan Kreator, Dashboard Kreator, Pengaturan Akun, Keluar.
- **Notifikasi in-app** — hasil kurasi konsinyasi & event relevan (tidak lewat WhatsApp/email).
- **Search 3 lapis:** Global (navbar) → Lokal Katalog → Lokal Daftar Kreator.

### Creator (Mobile Flutter — setelah pengajuan disetujui)

- **Dashboard Kreator** — stats card + menu kelola produk/cerita/status pengajuan.
- **Kelola Produk** — tambah/edit produk + variant builder matrix + Limited Edition switch.
- **Kelola Cerita "Behind the Design"** — judul + isi + foto proses pembuatan per produk.
- **Bantu Tulis dengan AI** — integrasi Gemini API untuk membantu kreator menyusun draft cerita dari poin singkat (lihat section [Integrasi AI](#integrasi-ai--bantu-tulis-behind-the-design)).
- **Tetap bisa beli** produk kreator lain seperti buyer biasa (tapi tidak bisa beli produk sendiri — divalidasi saat checkout).

### Admin (Web Next.js)

- **Dashboard** — stat cards + grafik (donut + bar) + aktivitas terbaru + upcoming pickups + restock alerts.
- **CRUD Produk** — termasuk variant builder + stok per kombinasi + Limited Edition toggle.
- **Kelola Pesanan** — daftar, ubah status, **extend waktu hold** (khusus belum lunas), batalkan.
- **Scan QR Pickup** — kamera browser, 4-state flow (idle → manual input → result → confirmed).
- **Kurasi Konsinyasi** — terima/tolak pengajuan, aktif/nonaktifkan kreator.
- **Moderasi Ulasan** — sembunyikan/hapus review tidak pantas.
- **Pengaturan Sistem** — durasi hold, auto-cancel toggle, batas pengambilan (7/14/30 hari), nomor WA admin.
- **Login terpisah** di `/admin/login` tapi tetap validasi ke tabel `users` yang sama (`role === "admin"`).

---

## Screenshots

Berikut tangkapan layar hasil akhir aplikasi (klik untuk melihat ukuran penuh).

### Mobile (Flutter → dirender via TSX export)

<details open>
<summary><b>Home, Katalog, Detail Produk & Kreator</b></summary>

![Mobile Home + Katalog + Detail](docs/screenshots/mobile-home.png)

_Halaman Beranda (hero + kategori + rekomendasi + preview kreator + ringkasan ulasan + CTA), Katalog (filter + sort + grid), Detail Produk (galeri + varian + Behind the Design + review), Daftar Kreator._
</details>

<details>
<summary><b>Profil Kreator, Reservasi & Konfirmasi QR</b></summary>

![Mobile Reservasi](docs/screenshots/mobile-reservasi.png)

_Profil kreator, alur reservasi 3-step (jadwal → bayar → tiket QR), dan halaman konfirmasi dengan QR ticket._
</details>

<details>
<summary><b>Daftar Kreator, Dashboard Kreator & Form Produk</b></summary>

![Mobile Dashboard Kreator](docs/screenshots/mobile-dashboard-kreator.png)

_Form daftar konsinyasi, dashboard kreator (stat cards + menu), daftar produk saya, dan form tambah produk dengan variant builder._
</details>

<details>
<summary><b>Cerita Saya, Form Cerita (Bantu Tulis AI), Notifikasi & Hub Akun</b></summary>

![Mobile Story Form + Account](docs/screenshots/mobile-story-form.png)

_Daftar cerita per produk, form "Behind the Design" (tempat tombol "Bantu Tulis dengan AI" berada), halaman notifikasi, dan Hub Akun._
</details>

<details>
<summary><b>Login, Register & Lupa Password</b></summary>

![Mobile Auth](docs/screenshots/mobile-auth.png)

_Halaman autentikasi dengan brand panel kiri (ornamen heritage) + form kanan._
</details>

<details>
<summary><b>Semua Halaman Akun (Pesanan, Detail, Ulasan, Pengajuan, Pengaturan)</b></summary>

![Mobile Account](docs/screenshots/mobile-account.png)

_Riwayat pesanan, detail tiket QR, ulasan saya, tulis ulasan, status pengajuan kreator, pengaturan akun._
</details>

### Web Admin (Next.js)

<details open>
<summary><b>Login & Dashboard Admin</b></summary>

![Admin Dashboard](docs/screenshots/admin-dashboard.png)

_Halaman login admin + dashboard dengan stat cards, grafik distribusi pesanan, top produk, aktivitas terbaru, upcoming pickups, dan restock alerts._
</details>

<details>
<summary><b>Kelola Produk (List, Tambah & Edit)</b></summary>

![Admin Produk](docs/screenshots/admin-produk.png)

_Daftar produk dengan filter + search, form tambah produk (foto + varian + stok), dan form edit._
</details>

<details>
<summary><b>Kelola Pesanan & Detail Pesanan</b></summary>

![Admin Pesanan](docs/screenshots/admin-pesanan.png)

_Daftar pesanan dengan filter status + tab, detail pesanan dengan info buyer + produk + timeline + aksi (extend hold, batalkan, cetak tiket)._
</details>

<details>
<summary><b>Scan QR Pickup, Kelola Kreator & Detail Pengajuan</b></summary>

![Admin Scan QR + Kreator](docs/screenshots/admin-scanqr-kreator.png)

_Scan QR (4-state flow: idle → manual input → hasil scan → konfirmasi), daftar kreator dengan tab aktif/pengajuan, dan detail pengajuan konsinyasi dengan aksi terima/tolak._
</details>

---

## Daftar Library

### Web (Next.js 16) — `admin-web/package.json`

#### Core Framework & Bahasa

| Library | Versi | Fungsi |
|---|---|---|
| `next` | ^16.1.1 | Next.js 16 (App Router, Turbopack) |
| `react` / `react-dom` | ^19.0.0 | React 19 |
| `typescript` | ^5 | Type system |
| `eslint` / `eslint-config-next` | ^9 / ^16.1.1 | Linting |

#### Styling & UI

| Library | Versi | Fungsi |
|---|---|---|
| `tailwindcss` | ^4 | Utility-first CSS framework |
| `@tailwindcss/postcss` | ^4 | Tailwind PostCSS plugin |
| `tailwindcss-animate` | ^1.0.7 | Animasi utilities |
| `tw-animate-css` | ^1.3.5 | Animate utilities (Tailwind 4 compat) |
| `class-variance-authority` | ^0.7.1 | Variant management (shadcn/ui) |
| `clsx` | ^2.1.1 | Conditional className helper |
| `tailwind-merge` | ^3.3.1 | Tailwind class dedup |
| `lucide-react` | ^0.525.0 | Icon library |
| `@radix-ui/react-*` (30+ packages) | various | Headless UI primitives (basis shadcn/ui) |
| `cmdk` | ^1.1.1 | Command palette |
| `embla-carousel-react` | ^8.6.0 | Carousel (galeri produk) |
| `react-day-picker` | ^9.8.0 | Calendar / date picker |
| `vaul` | ^1.1.2 | Bottom sheet (AuthGate mobile) |
| `react-resizable-panels` | ^3.0.3 | Resizable panels |
| `input-otp` | ^1.4.2 | OTP input (lupa password) |
| `@radix-ui/react-aspect-ratio` | ^1.1.7 | Aspect ratio container |

#### State & Data

| Library | Versi | Fungsi |
|---|---|---|
| `zustand` | ^5.0.6 | Client state management (persist) |
| `@tanstack/react-query` | ^5.82.0 | Server state |
| `@tanstack/react-table` | ^8.21.3 | Table component (admin) |

#### Database & Backend

| Library | Versi | Fungsi |
|---|---|---|
| `prisma` | ^6.11.1 | Prisma ORM CLI + generator |
| `@prisma/client` | ^6.11.1 | Prisma Client (SQLite dev / MySQL prod) |
| `next-auth` | ^4.24.11 | Authentication (NextAuth.js v4) |
| `resend` | ^6.28.1 | Email OTP service |

#### Forms & Validation

| Library | Versi | Fungsi |
|---|---|---|
| `react-hook-form` | ^7.60.0 | Form state management |
| `@hookform/resolvers` | ^5.1.1 | Schema resolvers |
| `zod` | ^4.0.2 | Schema validation |

#### Visualization & Content

| Library | Versi | Fungsi |
|---|---|---|
| `recharts` | ^2.15.4 | Charts (donut, bar, line) untuk dashboard admin |
| `react-markdown` | ^10.1.0 | Markdown rendering |
| `@mdxeditor/editor` | ^3.39.1 | MDX rich text editor |
| `react-syntax-highlighter` | ^15.6.1 | Code syntax highlighting |

#### Utilities

| Library | Versi | Fungsi |
|---|---|---|
| `date-fns` | ^4.1.0 | Date formatting (id-ID) |
| `sonner` | ^2.0.6 | Toast notifications |
| `next-themes` | ^0.4.6 | Light/dark mode |
| `next-intl` | ^4.3.4 | Internationalization |
| `sharp` | ^0.34.3 | Image processing (Next.js image optimization) |
| `uuid` | ^11.1.0 | UUID generator |
| `framer-motion` | ^12.23.2 | Animations (page transitions, hover) |
| `@dnd-kit/core` / `sortable` / `utilities` | ^6.3.1 / ^10.0.0 / ^3.2.2 | Drag & drop (sortable variant builder) |
| `@reactuses/core` | ^6.0.5 | Custom React hooks |

### Backend (Express.js) — `backend/package.json`

| Library | Versi | Fungsi |
|---|---|---|
| `express` | ^4.21.0 | Web framework |
| `@prisma/client` | ^5.20.0 | Prisma Client (MySQL) |
| `prisma` (dev) | ^5.20.0 | Prisma ORM CLI |
| `bcrypt` | ^5.1.1 | Password hashing |
| `jsonwebtoken` | ^9.0.2 | JWT auth tokens |
| `cors` | ^2.8.5 | CORS middleware |
| `helmet` | ^8.3.0 | Security headers |
| `dotenv` | ^16.4.5 | Environment variables |
| `express-rate-limit` | ^8.7.0 | Rate limiting |
| `google-auth-library` | ^11.1.0 | Google OAuth verification |
| `midtrans-client` | ^1.4.3 | Midtrans payment gateway |
| `multer` | ^1.4.5-lts.1 | File upload (multipart) |
| `node-cron` | ^4.6.0 | Scheduled jobs (auto-cancel expired holds) |
| `resend` | ^6.28.1 | Email OTP service |
| `nodemon` (dev) | ^3.1.7 | Auto-restart dev server |

### Mobile (Flutter) — `user_mobile/pubspec.yaml`

| Library | Versi | Fungsi |
|---|---|---|
| `flutter` | sdk >=3.22.0 | Flutter framework |
| `provider` | ^6.1.2 | State management (pola MVVM) |
| `shared_preferences` | ^2.2.3 | Local persistence (auth state) |
| `go_router` | ^14.2.0 | Declarative routing |
| `google_fonts` | ^6.2.1 | Baloo 2 (display) + Poppins (body) |
| `qr_flutter` | ^4.1.0 | QR code generation (tiket reservasi) |
| `url_launcher` | ^6.3.0 | Open WhatsApp & Shopee external links |
| `intl` | ^0.19.0 | Currency & date formatting (id_ID) |
| `image_picker` | ^1.1.2 | Photo upload (form konsinyasi + produk) |
| `flutter_lints` | ^4.0.0 (dev) | Lint rules |

### Dev Tools

| Tool | Versi | Fungsi |
|---|---|---|
| `npm` | v10+ | Package manager (admin-web & backend) |
| `node` | v20+ | JavaScript runtime |
| `flutter` | >=3.22.0 | Mobile SDK |
| `prisma` CLI | ^5.20.0 | Database migration & generation |
| `nodemon` | ^3.1.7 | Auto-restart backend dev server |
| `eslint` | ^9 | Linting (web) |

---

## Tech Stack

| Layer | Teknologi | Catatan |
|---|---|---|
| **Frontend Web** | Next.js 16 + TypeScript + Tailwind CSS 4 + shadcn/ui | Panel admin only |
| **Frontend Mobile** | Flutter 3.22+ (Dart) | Customer + creator |
| **Backend** | Express.js + Prisma ORM | REST API, JWT auth |
| **Database** | MySQL (via Laragon) | 14 tabel, schema di Prisma |
| **Payment** | Midtrans Snap | Online payment gateway |
| **Email** | Resend | OTP & transactional email |
| **AI** | Google Gemini API | Bantu Tulis Behind the Design |
| **Auth** | JWT + bcrypt + Google OAuth | Multi-provider |
| **File Upload** | Multer (lokal sementara) | Target: cloud storage |
| **Scheduled Jobs** | node-cron | Auto-cancel expired holds |

---

## Struktur Repo

```
.
├── admin-web/                    # Next.js 16 (web — admin panel)
│   ├── src/
│   │   ├── app/                  # App Router (Dashboard, Produk, Pesanan, Scan QR, Kreator, Ulasan, Pengaturan, Login)
│   │   ├── components/           # UI Components (admin shell, stat cards, dialogs, tables)
│   │   ├── data/                 # Dummy/seed data
│   │   ├── hooks/                # Custom hooks (use-mobile, use-toast)
│   │   └── lib/                  # Fetch API helpers, formatters, & auth stores
│   ├── public/                   # Static assets
│   ├── package.json              # Dependencies (managed via npm)
│   └── tsconfig.json
│
├── backend/                      # Express.js + Prisma + MySQL (REST API)
│   ├── prisma/
│   │   └── schema.prisma         # Database schema (14 tabel)
│   ├── src/
│   │   ├── controllers/          # Business logic per resource
│   │   │   ├── admin.controller.js
│   │   │   ├── artisans.controller.js
│   │   │   ├── auth.controller.js
│   │   │   ├── notifications.controller.js
│   │   │   ├── orders.controller.js
│   │   │   ├── payments.controller.js
│   │   │   ├── products.controller.js
│   │   │   ├── reviews.controller.js
│   │   │   ├── search.controller.js
│   │   │   └── upload.controller.js
│   │   ├── routes/               # Express route definitions
│   │   │   ├── admin.routes.js
│   │   │   ├── artisans.routes.js
│   │   │   ├── auth.routes.js
│   │   │   ├── notifications.routes.js
│   │   │   ├── orders.routes.js
│   │   │   ├── payments.routes.js
│   │   │   ├── products.routes.js
│   │   │   ├── reviews.routes.js
│   │   │   ├── search.routes.js
│   │   │   └── upload.routes.js
│   │   ├── middleware/           # Auth, rate limiter, role guard
│   │   │   ├── auth.js
│   │   │   ├── rateLimiter.js
│   │   │   └── requireRole.js
│   │   ├── lib/                  # Shared utilities
│   │   │   ├── mailer.js         # Resend email integration
│   │   │   ├── midtrans.js       # Midtrans client config
│   │   │   ├── prisma.js         # Prisma client singleton
│   │   │   ├── stock.js          # Effective stock calculation
│   │   │   ├── upload.js         # Multer config
│   │   │   ├── validatePassword.js
│   │   │   └── variants.js       # Cartesian product for variant combos
│   │   ├── jobs/                 # Scheduled tasks
│   │   │   └── orderStatusJob.js # Auto-cancel & expired hold checker
│   │   └── index.js              # Express app entry point
│   ├── uploads/                  # File uploads (lokal, git-ignored)
│   ├── .env.example
│   └── package.json
│
├── user_mobile/                  # Flutter 3.22+ (customer & creator mobile app)
│   ├── lib/
│   │   ├── core/                 # Theme, routing, constants, & utils
│   │   ├── data/                 # Models & data sources
│   │   ├── features/             # Feature modules (auth, home, catalog, product_detail, reservation, creator, account, dll)
│   │   └── main.dart             # Entry point
│   ├── android/                  # Android platform files
│   ├── pubspec.yaml              # Flutter dependencies
│   └── analysis_options.yaml
│
├── docs/                         # Dokumentasi project
│   ├── dokumen-master-sesuatu-darikota-malang.md   # Single source of truth (aturan bisnis, alur, spesifikasi)
│   ├── setup-project.md          # Panduan setup development
│   └── catatanterbaru.md         # Catatan perubahan terkini
│
├── .gitignore
└── README.md
```

---

## Quick Start

### Prasyarat

- **Node.js** 20+ dan **npm** (v10+)
- **Flutter** 3.22+ dengan Dart SDK 3.3+ (untuk mobile)
- **MySQL** (via Laragon / XAMPP / MySQL Server lokal)
- **Git**

### 1. Clone Repo

```bash
git clone https://github.com/4abduu/sesuatudarikotamalang.git
cd sesuatu-darikota-malang
```

### 2. Backend (Express.js + Prisma + MySQL)

```bash
# Masuk ke folder backend
cd backend

# Install dependencies
npm install

# Copy dan isi environment variables
cp .env.example .env
# Edit .env → isi DATABASE_URL, JWT_SECRET, dll (lihat section Environment Variables)

# Push schema Prisma ke MySQL lokal
npx prisma db push

# Generate Prisma Client
npx prisma generate

# Jalankan dev server
npm run dev
```

Server API berjalan di `http://localhost:4000`.

**Scripts tersedia (backend):**

| Command | Fungsi |
|---|---|
| `npm run dev` | Jalankan dev server dengan nodemon (port 4000) |
| `npm start` | Jalankan server produksi |
| `npm run prisma:generate` | Generate Prisma Client |
| `npm run prisma:push` | Push Prisma schema ke database |
| `npm run prisma:studio` | Buka Prisma Studio (GUI database) |

### 3. Web (Next.js — Admin Panel)

```bash
# Masuk ke folder admin-web
cd admin-web

# Install dependencies
npm install

# Jalankan dev server
npm run dev
```

Buka `http://localhost:3000`. Server akan otomatis reload saat ada perubahan.

### 4. Mobile (Flutter — Customer & Creator)

```bash
# Masuk ke folder user_mobile
cd user_mobile

# Install dependencies
flutter pub get

# Jalankan di emulator/device
flutter run

# Build APK (debug)
flutter build apk --debug

# Build APK (release)
flutter build apk --release
```

> **Tip:** Untuk Google Sign-In, perlu daftar OAuth Client ID di Google Cloud Console (tipe Web, Android dengan SHA-1 fingerprint via `cd android && ./gradlew signingReport`, dan iOS jika build ke iOS).

---

## Environment Variables

### Backend (`backend/.env`)

```env
# Database (MySQL via Laragon)
DATABASE_URL="mysql://root:@localhost:3306/sesuatu_darikota_malang"

# JWT Secret (wajib, gunakan random string panjang)
JWT_SECRET=""

# Port server
PORT=4000

# Google OAuth (untuk Google Sign-In di mobile)
GOOGLE_CLIENT_ID=""

# Email OTP (Resend) — kosongkan = mode simulasi
RESEND_API_KEY=""
RESEND_FROM_EMAIL=""

# Midtrans Payment Gateway
MIDTRANS_MERCHANT_ID=""
MIDTRANS_SERVER_KEY=""
MIDTRANS_CLIENT_KEY=""
MIDTRANS_IS_PRODUCTION="false"
MIDTRANS_IS_SANITIZED="true"
MIDTRANS_IS_3DS="true"

# CORS (origins yang diizinkan, pisahkan dengan koma)
CORS_ALLOWED_ORIGINS=""

# AI Integration — Gemini API (untuk "Bantu Tulis Behind the Design")
GEMINI_API_KEY=""
```

---

## Akun Demo

Aplikasi sudah punya 3 akun demo siap pakai (di kedua platform — web & mobile):

| Role | Email | Password | Akses |
|---|---|---|---|
| **Pembeli** | `maya@email.com` | `demo1234` | Browse, beli, ulasan, ajukan jadi kreator |
| **Kreator** | `dini.draws@gmail.com` | `demo1234` | Semua akses buyer + Dashboard Kreator |
| **Admin** | `admin@darikotamalang.id` | `admin1234` | Panel admin (`/admin/login`) |

Nama-nama ini sengaja disamakan dengan **User Persona** di dokumentasi UX untuk konsistensi.

---

## Design System

Project ini punya identitas brand yang kuat, konsisten di web & mobile.

### Palet Warna

| Token | Hex | Penggunaan |
|---|---|---|
| `--primary` | `#A8452B` | Terracotta — header, tombol utama, kode OTP |
| `--primary-foreground` | `#F7EFDD` | Cream gading di atas terracotta |
| `--background` | `#F2E8D5` | Krem hangat (page background) |
| `--card` | `#FBF6EA` | Cream lebih terang (card background) |
| `--foreground` | `#2B211A` | Coklat tua hampir hitam (body text) |
| `--secondary` | `#E8D9BC` | Deeper cream/sand (OTP badge) |
| `--accent` | `#6B7A3D` | Moss green — badges, eyebrow text |
| `--border` | `#D9C9AC` | Sand border |
| `--destructive` | `#9B3320` | Error / delete actions |
| `--muted-foreground` | `#6B5D4F` | Muted brown (secondary text) |

### Tipografi

- **Display:** [Baloo 2](https://fonts.google.com/specimen/Baloo+2) — rounded, playful, untuk judul & brand wordmark
- **Body:** [Poppins](https://fonts.google.com/specimen/Poppins) — clean sans-serif untuk body text

### Ornamen & Detail

- **Line-art ornaments:** `LeafSprig`, `SwirlOrnament`, `HeritageWindow`, `HandPrint`, `SunMotif` — ilustrasi garis sederhana terinspirasi Kayutangan Heritage.
- **Sticker shadow:** custom `box-shadow` preset untuk kesan depth (bukan flat design).
- **Blob frames:** border-radius organik untuk product images.
- **Paper texture:** subtle SVG noise overlay untuk kesan vintage paper.

### Komponen Khas

- **MobilePhoneFrame** — 402px fixed-width frame untuk export mobile pages ke Figma.
- **AuthGate bottom sheet** — modal ajakan login (bukan redirect diam-diam).
- **HoldCountdownWidget** — timer hitung mundur untuk hold stok.
- **Craft cards** — `StatChipCard`, `MenuCard`, `SectionCard` pola kartu konsisten di mobile.

---

## Aturan Bisnis Kunci

### 1. Single `users` Table with Role

Semua peran (Guest, Buyer, Creator, Admin) disimpan dalam **satu tabel `users`** dengan kolom `role`. Bukan sistem akun terpisah per peran.

### 2. Varian 2D dengan Stok per Kombinasi

Produk bisa punya multiple variant axes (mis. Motif x Ukuran). **Stok dihitung per kombinasi**, bukan per produk. Produk tanpa varian tetap punya 1 baris di `product_variants` dengan `combination: []`.

### 3. Check-at-Submit (Bukan Lock-on-Browse)

Stok **tidak di-lock** saat user memilih varian atau mengisi form checkout. Pengecekan dilakukan tepat saat tombol "Pesan Sekarang" diklik. Jika kombinasi sudah diambil pembeli lain, tampilkan error jelas tanpa mengunci apapun sebelumnya.

### 4. Dual Payment Cycle

| Metode | Status Awal | Stok | Auto-cancel? |
|---|---|---|---|
| **Midtrans (online)** | `Lunas` | Dikurangi permanen saat itu juga | Tidak — barang sudah terjual final |
| **Cash/QRIS di toko** | `Menunggu Bayar` | Di-hold sementara (default 2 jam) | Ya, bisa diaktifkan admin |

Untuk Midtrans: batas pengambilan default **14 hari** (konfigurabel 7/14/30), lewat batas → status `Lewat Batas Pengambilan` (bukan `Dibatalkan`).

Untuk Cash/QRIS: admin bisa **extend waktu hold** secara manual (maksimal **total 2 hari** sejak order dibuat).

### 5. Larangan Beli Produk Sendiri

Saat checkout, jika `product.creatorId` terhubung ke `user_id` pembeli yang login → transaksi diblok dengan pesan "kamu tidak bisa membeli produkmu sendiri".

### 6. AuthGate (Bukan Redirect Diam-Diam)

Guest yang klik aksi butuh-login → **dialog ajakan login** ("Yuk masuk dulu") dengan tombol ke halaman login + opsi "Nanti dulu". Hanya halaman yang sepenuhnya personal (`/akun`, `/dashboard-kreator`, `/notifikasi`) yang redirect penuh ke login.

### 7. Notifikasi In-App Saja (Buyer & Creator)

Notifikasi ditampilkan **di dalam aplikasi**, bukan via WhatsApp/email. **Admin sengaja tidak punya halaman notifikasi terpisah** — semua info sudah otomatis tersaji di Dashboard admin.

### 8. Rekomendasi Rule-Based (Bukan AI Generatif)

Rekomendasi produk memakai pendekatan **rule-based** dengan urutan prioritas fallback:
1. Produk dengan `soldCount` tertinggi
2. Jika belum ada penjualan → produk dengan views tertinggi
3. Jika belum ada data sama sekali → random (jaring pengaman)

> AI generatif (LLM) **tidak dipakai untuk rekomendasi** karena kurang cocok untuk ranking/sorting data terstruktur (mahal, lambat, tidak deterministik). LLM dipakai untuk kebutuhan lain yang lebih natural — lihat section [Integrasi AI](#integrasi-ai--bantu-tulis-behind-the-design).

### 9. Redirect Setelah Login/Logout

- **Login:** ke tujuan dari `next` parameter (jika ada), atau ke landing page `/`.
- **Logout:** selalu ke landing page `/`.

### 10. Google Sign-In (Mobile Only)

- Hanya untuk **mobile/buyer**. Admin-web tetap email+password.
- Akun Google murni (`authProvider = "google"`) tidak punya password (`passwordHash` nullable).
- Jika user coba login manual dengan email Google → error `GOOGLE_ACCOUNT`, aplikasi minta pakai tombol Google.
- Setelah reset password via OTP → `authProvider` berubah jadi `"both"`.

---

## Integrasi AI — Bantu Tulis Behind the Design

Project ini mengintegrasikan **Google Gemini API** untuk satu fitur spesifik: membantu kreator menyusun draft cerita "Behind the Design" dari poin-poin singkat. Ini adalah pemenuhan syarat wajib integrasi AI dari dosen.

### Cara Kerja

```
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│  Kreator input  │───▶│  Backend Express │───▶│  Gemini API     │
│  poin singkat   │    │  (prompt engineer)│   │  (Generative AI)│
│  - motif        │    │  - system prompt │    │                 │
│  - proses       │    │  - user prompt   │    │                 │
│  - inspirasi    │    │  - JSON schema   │    │                 │
└─────────────────┘    └──────────────────┘    └────────┬────────┘
                                                        │
                                                        ▼
                                              ┌─────────────────┐
                                              │  Draft          │
                                              │  { judul, isi } │
                                              └────────┬────────┘
                                                       │
                                                       ▼
                                              ┌─────────────────┐
                                              │  Kreator review │
                                              │  • Gunakan Draft│
                                              │  • Buat Ulang   │
                                              │  • Tulis Sendiri│
                                              └─────────────────┘
```

### Detail Implementasi

1. **Lokasi fitur:** Form "Behind the Design" di Dashboard Kreator (mobile Flutter).
2. **Input kreator:** Beberapa poin singkat (bukan kalimat lengkap) — misal: motif/objek yang digambar, proses pembuatan, inspirasi di baliknya.
3. **Pemanggilan API:** Backend memanggil **Gemini API** dengan prompt yang sudah direkayasa ketat, lalu mengembalikan draft `{ judul, isi }` terstruktur.
4. **Wajib peninjauan kreator:** Kreator **wajib meninjau** hasilnya sebelum disimpan. Ada 3 pilihan:
   - **"Gunakan Draft Ini"** — isi otomatis ke form, tetap bisa diedit manual
   - **"Buat Ulang"** — generate ulang dengan parameter yang sama
   - **"Tulis Sendiri"** — batal, isi manual seperti biasa
5. **AI tidak pernah menyimpan otomatis** tanpa peninjauan kreator.
6. **Opsional:** Kreator yang nyaman menulis sendiri tetap bisa skip tombol ini sepenuhnya.
7. **Lingkup dibatasi ke konten storytelling saja** (bukan deskripsi produk, bukan rekomendasi produk) — ini area yang paling natural untuk AI generatif (menulis teks kreatif dari poin singkat).

### Kenapa Bukan untuk Rekomendasi?

AI generatif (LLM) kurang cocok untuk kasus ranking/sorting data terstruktur karena:
- Mahal per request
- Lambat (dibanding query database)
- Tidak deterministik (sulit didemo konsisten)
- Sulit di-debug

Rekomendasi produk tetap pakai **rule-based** (lihat [Aturan Bisnis #8](#8-rekomendasi-rule-based-bukan-ai-generatif)). LLM lebih cocok untuk kebutuhan generatif teks seperti draft cerita ini.

### Environment Variable

```env
GEMINI_API_KEY=your_gemini_api_key_here
```

---

## Skema Database

Skema lengkap ada di `backend/prisma/schema.prisma`. Ringkasan tabel:

| Tabel | Fungsi |
|---|---|
| `users` | Satu tabel untuk buyer/creator/admin (kolom `role`) |
| `otp_codes` | Kode OTP untuk ganti email & lupa password |
| `artisans` | Profil kreator (1-1 ke `users`, nullable) |
| `artisan_applications` | Pengajuan konsinyasi (pending/diterima/ditolak) |
| `categories` | Kategori produk |
| `products` | Produk konsinyasi (`isActive` untuk nonaktifkan tanpa hapus) |
| `product_variant_options` | Sumbu varian dinamis (JSON array values) |
| `product_variants` | Kombinasi final + stok per kombinasi |
| `product_stories` | Cerita "Behind the Design" per produk (1-0..1) |
| `orders` | Header transaksi reservasi (tidak nempel langsung ke produk) |
| `order_items` | Baris produk+varian dalam order (disiapkan multi-produk) |
| `payments` | Detail transaksi Midtrans per order (1-0..1) |
| `reviews` | Ulasan, terhubung ke `order_items` (bukan `orders`) |
| `notifications` | Notifikasi in-app per user |
| `app_settings` | Satu baris pengaturan sistem (bukan key-value) |

**Relasi kunci:** `artisans 1—N products`, `products 1—N product_variants`, `products 1—0..1 product_stories`, `users 1—N orders`, `orders 1—N order_items`, `orders 1—0..1 payments`, `order_items 1—0..1 review`, `users 1—0..1 artisans`.

---

## Figma Export Utilities

Project ini punya infrastruktur khusus untuk export desain ke Figma via plugin [html.to.design](https://html.to.design/). Plugin ini punya kuota terbatas per akun, jadi halaman-halaman digabung jadi satu URL per combo.

### Akses

- **High-fidelity:** `/export-figma` — styling penuh untuk mockup
- **Low-fidelity wireframe:** `/export-wireframe` — kotak-kotak grayscale untuk deliverable wireframe terpisah

### Cara Pakai

1. Buka `/export-figma` (atau `/export-wireframe`) di browser
2. Pilih salah satu combo (mis. "Admin: Login & Dashboard")
3. Tunggu semua halaman render (ada section divider antar halaman)
4. Di Figma, jalankan plugin html.to.design → paste URL combo → import
5. Semua halaman akan masuk sebagai frames terpisah dalam satu Figma file

### Combo yang Tersedia

- **Admin web:** 5 combo (login+dashboard, produk+tambah+edit, pesanan+detail, scan+kreator+pengajuan, ulasan+pengaturan)
- **Mobile (Flutter → TSX):** 7 combo (home+katalog+detail+kreator, profil+reservasi+konfirmasi, auth screens, daftar+dashboard kreator, cerita+notif+hub, semua halaman akun, pencarian)
- **Modals:** "Semua Modal" — 10 modals inline (6 admin web + 4 mobile)
- **Revisions:** combo fokus untuk perubahan terbaru

---

## Roadmap

### Sudah Diimplementasi (Versi 0.9.0)

- [x] Web admin panel lengkap (dashboard, produk, pesanan, scan QR, kreator, ulasan, pengaturan)
- [x] Mobile app customer lengkap (25+ halaman Flutter)
- [x] Sistem varian 2D dengan stok per kombinasi
- [x] Reservasi pickup 3-step + QR ticket
- [x] Dual payment cycle (Midtrans mock + Cash/QRIS hold)
- [x] AuthGate modal (guest → ajakan login)
- [x] Notifikasi in-app
- [x] 3-layer search
- [x] Dashboard kreator (produk + cerita + variant builder)
- [x] Figma export utilities (hi-fi + wireframe)
- [x] Design system (terracotta + cream + moss + Baloo 2/Poppins)
- [x] Backend Express.js + Prisma + MySQL (REST API lengkap)
- [x] JWT authentication + Google OAuth
- [x] File upload (lokal via Multer)
- [x] Scheduled jobs (auto-cancel expired holds)

### Sedang Berjalan / Target Produksi

- [ ] Midtrans Snap integration (real payment)
- [ ] Gemini API integration untuk "Bantu Tulis Behind the Design"
- [ ] Cloud storage untuk media (Cloudinary/Firebase Storage/S3)
- [ ] Resend email OTP (real send, bukan simulation)
- [ ] Integrasi API admin-web ↔ backend
- [ ] Integrasi API mobile ↔ backend

### Out-of-Scope (Sengaja Tidak Dikerjakan)

- **Pengiriman dari platform sendiri** — diarahkan ke Shopee/Tokopedia.
- **Chat in-app** — cukup redirect WhatsApp.
- **Sistem report/ban formal ala marketplace P2P** — kreator sudah dikurasi, cukup kontrol admin.
- **Sinkron stok real-time dengan Shopee** — produk di e-commerce adalah barang toko sendiri, bukan konsinyasi.
- **Notifikasi approve/reject via WhatsApp/email** — cukup in-app.
- **Dashboard kreator self-service penuh** — dibatasi untuk menyeimbangkan fitur dengan keterbatasan waktu tim.

---

## Tim

Project tugas kuliah oleh 3 orang (semester 3):

| Nama | Peran |
|---|---|
| **Abdurrahman Ichwan** | Initial Frontend UI/UX Development + Back-End with AI + PM + QA/QC |
| **Teuku Atha Athaya Nafi** | Frontend User Mobile + Integration API Mobile + Integration Midtrans Mobile + QA/QC |
| **Muhammad Ikhsan Putra Abiansyah** | Frontend Admin Web + Integration API Admin + QA/QC |

### Narasumber Wawancara

- **Mas Zainal** — staf toko (wawancara awal untuk validasi kebutuhan)

---

## Status

| Aspek | Status |
|---|---|
| **Versi** | 0.9.0 — Masih dalam tahap pengembangan & finalisasi |
| **Dokumen master** | Draft komprehensif (single source of truth), masih direvisi untuk integrasi AI |
| **Frontend Web** | Berfungsi penuh (panel admin) |
| **Frontend Mobile** | Berfungsi penuh (25+ halaman Flutter) |
| **Backend** | REST API lengkap (Express.js + MySQL + Prisma), integrasi frontend sedang berjalan |
| **Design system** | Stabil (terracotta + cream + moss + Baloo 2/Poppins) |
| **Figma export** | Aktif (14+ combo hi-fi + 14+ combo wireframe) |
| **Integrasi AI** | Designed (Gemini API untuk Behind the Design), implementation in progress |

---

## Kontribusi

Karena ini tugas kuliah, kontribusi eksternal tidak diterima. Tapi jika kamu menemukan bug atau punya saran, dipersilakan untuk open issue di GitHub.

### Konvensi Kode

- **TypeScript** strict mode (web admin)
- **CommonJS** (`require`) untuk backend Express.js
- **ES6+ imports** untuk frontend Next.js
- **shadcn/ui components** lebih diutamakan daripada custom implementations (admin-web)
- **`'use client'` / `'use server'`** directive untuk client/server boundary (Next.js)
- **Prisma schema** di `backend/prisma/schema.prisma`
- **Flutter:** pola Provider (mirip MVVM), bukan MVC klasik
- **Tidak ada emoji di kode produksi**

### Konvensi Commit

```
<type>: <description>

feat: menambahkan halaman scan QR dengan input manual
fix: memperbaiki layout footer di mobile
style: menyesuaikan warna primary ke terracotta
refactor: memecah AdminScanQrContent menjadi komponen terpisah
docs: update README dengan bagian Figma export
chore: upgrade dependencies
```

---

## License

MIT License — bebas dipakai untuk tujuan edukasi. Untuk penggunaan komersial, hubungi tim.

---

<p align="center">
  <sub>Dibuat dengan <strong>terracotta + cream + kopi</strong> di Malang.</sub><br>
  <sub>&copy; 2025 Sesuatu DariKota Malang · Kayutangan Heritage, Klojen, Malang</sub>
</p>
