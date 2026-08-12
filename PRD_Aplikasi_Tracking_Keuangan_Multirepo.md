# PRD trackU

## 1. Informasi Dokumen

| Item | Detail |
|---|---|
| Nama Produk | trackU |
| Jenis Produk | Web application |
| Versi PRD | 2.0 |
| Status | Revisi arsitektur multi-repo |
| Target Pengguna | Individu, pasangan, keluarga kecil, dan kelompok yang ingin mencatat serta memantau keuangan bersama |
| Platform | Web responsive untuk desktop, tablet, dan mobile browser |

## 2. Ringkasan Produk

trackU adalah aplikasi web untuk membantu pengguna mencatat pemasukan, pengeluaran, saldo dompet, kategori transaksi, anggaran, target tabungan, dan ringkasan kondisi keuangan.

Aplikasi ini juga mendukung penggunaan berbasis grup, sehingga beberapa pengguna dapat mengelola catatan keuangan bersama dengan hak akses yang berbeda.

Pada revisi ini, arsitektur aplikasi ditetapkan sebagai sistem multi-repo:

- Frontend dan backend berada di repository terpisah.
- Frontend dibangun sebagai Vue 3 SPA dan dideploy ke GitHub Pages.
- Backend dibangun menggunakan Node.js, Express.js, dan TypeScript, lalu dideploy ke Vercel sebagai serverless/functions.
- Database menggunakan PostgreSQL dari Neon.
- Frontend dan backend berkomunikasi melalui REST API HTTPS.

## 3. Tujuan Produk

Tujuan utama aplikasi:

1. Membantu pengguna mencatat transaksi keuangan harian secara rapi.
2. Menyediakan dashboard untuk melihat kondisi keuangan secara cepat.
3. Memisahkan data berdasarkan grup agar cocok untuk kebutuhan personal atau bersama.
4. Menyediakan fitur kategori, wallet/dompet, budget, dan target tabungan.
5. Menyediakan AI Finance Assistant untuk membantu pengguna bertanya tentang data keuangannya.
6. Menjadi aplikasi web modern yang dapat dideploy menggunakan layanan gratis atau free-tier.

## 4. Masalah Yang Diselesaikan

Banyak pengguna mencatat keuangan secara manual di catatan, spreadsheet, atau aplikasi yang tidak fleksibel untuk kebutuhan grup. Akibatnya:

- Riwayat transaksi sulit dicari.
- Pengeluaran per kategori tidak terlihat jelas.
- Saldo per dompet tidak terpantau.
- Target tabungan sulit dipantau progresnya.
- Keuangan bersama sulit dikelola karena tidak ada akses berbasis grup.
- Pengguna sulit mendapat insight cepat dari data keuangan.

Aplikasi ini menyelesaikan masalah tersebut dengan pencatatan transaksi yang terstruktur, dashboard visual, sistem grup, dan asisten AI berbasis data keuangan pengguna.

## 5. Target Pengguna

### 5.1 Pengguna Individu

Pengguna yang ingin mencatat pemasukan dan pengeluaran pribadi, memantau saldo, serta melihat ringkasan keuangan bulanan.

### 5.2 Pasangan atau Keluarga Kecil

Pengguna yang ingin mencatat keuangan bersama, seperti pengeluaran rumah, tagihan, tabungan bersama, dan budget keluarga.

### 5.3 Kelompok Kecil

Kelompok yang membutuhkan pencatatan pemasukan dan pengeluaran sederhana, misalnya komunitas kecil, kepanitiaan, atau tim informal.

## 6. Scope MVP

### 6.1 Termasuk Dalam MVP

Fitur yang masuk MVP:

1. Authentication dan user account.
2. Group management sederhana.
3. Group-based RBAC.
4. Wallet/dompet.
5. Kategori transaksi.
6. Transaksi pemasukan dan pengeluaran.
7. Dashboard ringkasan keuangan.
8. Budget bulanan.
9. Target tabungan.
10. AI Finance Assistant berbasis request-response.
11. REST API backend.
12. Deploy frontend ke GitHub Pages.
13. Deploy backend ke Vercel.
14. Database PostgreSQL Neon.

### 6.2 Tidak Termasuk Dalam MVP

Fitur yang tidak masuk MVP:

1. Mobile app native Android/iOS.
2. WebSocket realtime.
3. Background worker permanen.
4. Sistem notifikasi push realtime.
5. Integrasi bank otomatis.
6. Pembayaran online.
7. Multi-currency kompleks.
8. Audit log enterprise.
9. Role permission editor yang sangat granular.

## 7. Modul Produk

## 7.1 Authentication

Modul authentication digunakan untuk login, registrasi, dan validasi akses pengguna.

Fitur:

- Register user.
- Login user.
- Logout di sisi frontend.
- Password hashing di backend.
- JWT Bearer token untuk MVP.
- Proteksi route frontend.
- Proteksi endpoint backend.

Catatan MVP:

- Setelah login berhasil, backend mengembalikan access token JWT.
- Frontend mengirim token pada setiap request API menggunakan header:

```http
Authorization: Bearer <token>
```

- Token hanya boleh digunakan untuk autentikasi API.
- Untuk versi lanjutan, sistem dapat dievaluasi ulang menggunakan HttpOnly Secure Cookie.

## 7.2 Group Management

Modul group digunakan agar satu pengguna dapat memiliki atau bergabung dengan beberapa grup keuangan.

Fitur:

- Membuat grup.
- Melihat daftar grup pengguna.
- Mengubah active group.
- Menambahkan anggota grup.
- Menghapus anggota grup.
- Melihat anggota grup.

Contoh penggunaan:

- Grup "Keuangan Pribadi".
- Grup "Rumah Tangga".
- Grup "Komunitas".

## 7.3 Group-Based RBAC

RBAC digunakan untuk membatasi hak akses pengguna dalam sebuah grup.

Role MVP:

| Role | Hak Akses |
|---|---|
| Owner | Akses penuh, kelola grup, kelola anggota, kelola semua data |
| Admin | Kelola transaksi, wallet, kategori, budget, dan target tabungan |
| Member | Melihat data grup; membuat transaksi; mengubah dan menghapus transaksi yang dibuat sendiri |
| Viewer | Hanya melihat data dan dashboard |

Aturan dasar:

- Semua data transaksi, wallet, kategori, budget, dan target tabungan wajib terikat ke `group_id`.
- Backend wajib memvalidasi bahwa user adalah anggota grup sebelum mengakses data grup.
- Owner dan Admin dapat membuat, mengubah, dan menghapus seluruh transaksi dalam grup.
- Member dapat membuat transaksi serta mengubah atau menghapus transaksi dengan `created_by` miliknya sendiri. Member tetap dapat melihat wallet dan kategori grup untuk memilihnya saat membuat transaksi, tetapi tidak dapat mengelola wallet, kategori, budget, target tabungan, atau anggota grup.
- Viewer hanya dapat melihat data dan dashboard; Viewer tidak dapat membuat, mengubah, atau menghapus data.
- Frontend boleh menyembunyikan menu berdasarkan role, tetapi validasi utama tetap wajib di backend.

## 7.4 Wallet / Dompet

Wallet digunakan untuk mencatat sumber atau tempat penyimpanan uang.

Contoh wallet:

- Cash.
- Bank BCA.
- Bank Mandiri.
- E-Wallet.
- Tabungan.

Fitur:

- Membuat wallet.
- Mengubah wallet.
- Menghapus wallet jika belum digunakan atau sesuai aturan.
- Melihat saldo wallet.
- Mengelompokkan transaksi berdasarkan wallet.

## 7.5 Kategori Transaksi

Kategori digunakan untuk mengelompokkan pemasukan dan pengeluaran.

Contoh kategori pengeluaran:

- Makan.
- Transportasi.
- Belanja.
- Tagihan.
- Hiburan.

Contoh kategori pemasukan:

- Gaji.
- Bonus.
- Freelance.
- Hadiah.

Fitur:

- Membuat kategori.
- Mengubah kategori.
- Menghapus kategori sesuai aturan.
- Menentukan tipe kategori: income atau expense.
- Menampilkan ringkasan pengeluaran berdasarkan kategori.

## 7.6 Transaksi

Transaksi adalah modul utama aplikasi.

Fitur:

- Mencatat pemasukan.
- Mencatat pengeluaran.
- Mengubah transaksi.
- Menghapus transaksi.
- Filter transaksi berdasarkan tanggal, kategori, wallet, tipe, dan keyword.
- Melihat daftar transaksi.
- Menambahkan catatan transaksi.

Field utama transaksi:

| Field | Keterangan |
|---|---|
| group_id | Grup pemilik transaksi |
| wallet_id | Wallet sumber/tujuan |
| category_id | Kategori transaksi |
| type | income atau expense |
| amount | Nominal transaksi |
| transaction_date | Tanggal transaksi |
| note | Catatan opsional |
| created_by | User pembuat transaksi |

## 7.7 Dashboard

Dashboard menampilkan ringkasan kondisi keuangan pengguna atau grup.

Komponen dashboard MVP:

- Total pemasukan bulan ini.
- Total pengeluaran bulan ini.
- Selisih cashflow.
- Saldo total wallet.
- Grafik pemasukan vs pengeluaran.
- Grafik pengeluaran berdasarkan kategori.
- Daftar transaksi terbaru.
- Progress budget.
- Progress target tabungan.

Library chart yang dapat digunakan:

- ECharts.
- ApexCharts.

Pilihan final dapat menggunakan salah satu. Jika ingin chart yang fleksibel dan kuat untuk dashboard, ECharts direkomendasikan.

## 7.8 Budget

Budget digunakan untuk membatasi atau memantau pengeluaran dalam kategori tertentu.

Fitur:

- Membuat budget bulanan.
- Menentukan kategori budget.
- Menentukan limit nominal.
- Melihat pemakaian budget.
- Menampilkan progress penggunaan budget.
- Memberi status aman, mendekati limit, atau melewati limit.

Contoh:

```text
Kategori: Makan
Limit: Rp2.000.000
Terpakai: Rp1.450.000
Sisa: Rp550.000
```

## 7.9 Target Tabungan

Target tabungan digunakan untuk membantu pengguna mencapai tujuan finansial tertentu.

Fitur:

- Membuat target tabungan.
- Menentukan nominal target.
- Menentukan tanggal target.
- Menambahkan kontribusi tabungan.
- Melihat daftar kontribusi pada setiap target.
- Menghapus kontribusi yang keliru sesuai hak akses role.
- Melihat progress target.

Ketentuan data:

- Setiap kontribusi disimpan sebagai record terpisah pada `savings_contributions`.
- Progress target dihitung dari total seluruh kontribusi pada target tersebut; aplikasi tidak memakai nilai progress yang diedit manual.

Contoh target:

- Dana darurat.
- Beli laptop.
- Liburan.
- DP rumah.

## 7.10 AI Finance Assistant

AI Finance Assistant membantu pengguna bertanya tentang data keuangan menggunakan bahasa natural.

Contoh pertanyaan:

- "Pengeluaran makan bulan ini berapa?"
- "Bulan ini saya paling boros di kategori apa?"
- "Apakah budget transportasi saya sudah hampir habis?"
- "Berapa saldo semua wallet saya?"

Prinsip keamanan:

- AI tidak boleh diberi akses SQL mentah.
- AI hanya boleh menggunakan function/tool yang disediakan backend.
- Backend tetap memvalidasi user, group, dan role.
- Data yang dikirim ke LLM harus dibatasi sesuai kebutuhan jawaban.

Contoh function internal:

```text
getMonthlySummary(groupId, month)
getExpenseByCategory(groupId, month)
getRecentTransactions(groupId, limit)
getBudgetStatus(groupId, month)
getSavingsGoalProgress(groupId)
```

AI chat pada MVP berjalan dengan pola request-response:

```text
User bertanya -> Frontend mengirim request -> Backend memproses -> LLM dipanggil bila perlu -> Backend mengembalikan jawaban
```

Pola ini cocok untuk Vercel serverless karena tidak membutuhkan koneksi realtime atau proses background permanen.

## 8. Arsitektur Sistem

## 8.1 Keputusan Arsitektur Final

Arsitektur final menggunakan multi-repo:

```text
GitHub Account
|
|-- tracku-frontend
|
`-- tracku-backend
```

Frontend dan backend tidak berada dalam satu aplikasi monolith. Keduanya berjalan dan dideploy secara terpisah.

Keputusan penting:

- Frontend adalah Vue 3 SPA.
- Backend adalah REST API.
- Komunikasi frontend-backend menggunakan HTTPS.
- Database hanya diakses oleh backend.
- Frontend tidak boleh menyimpan atau mengetahui `DATABASE_URL`.
- Inertia.js tidak digunakan pada arsitektur ini.

## 8.2 Diagram Arsitektur

```text
+-----------------------------+
|         User Browser         |
+--------------+--------------+
               |
               | HTTPS
               v
+-----------------------------+
| Frontend                    |
| Vue 3 SPA + TypeScript      |
| Vite                        |
| Tailwind CSS                |
| shadcn-vue                  |
| Pinia + Vue Router          |
| Axios                       |
| ECharts / ApexCharts        |
|                             |
| Source: GitHub Repo         |
| Deploy: GitHub Pages        |
+--------------+--------------+
               |
               | REST API HTTPS
               | VITE_API_URL
               v
+-----------------------------+
| Backend                     |
| Node.js + Express.js        |
| TypeScript                  |
| Zod                         |
| Drizzle ORM                 |
| JWT Bearer Auth             |
| Group-based RBAC            |
| CORS                        |
|                             |
| Source: Separate GitHub Repo|
| Deploy: Vercel Functions    |
+--------------+--------------+
               |
               | DATABASE_URL
               | Neon pooled connection
               v
+-----------------------------+
| Database                    |
| PostgreSQL                  |
| Neon                        |
+-----------------------------+
```

## 8.3 Frontend Architecture

Frontend menggunakan Vue 3 SPA.

Stack frontend:

| Area | Teknologi |
|---|---|
| Framework | Vue.js 3 |
| Language | TypeScript |
| Build Tool | Vite |
| Styling | Tailwind CSS |
| UI Component | shadcn-vue |
| State Management | Pinia |
| Routing | Vue Router |
| Chart | ECharts atau ApexCharts |
| HTTP Client | Axios |
| Hosting | GitHub Pages |

Struktur frontend yang direkomendasikan:

```text
tracku-frontend/
|
|-- public/
|-- src/
|   |-- api/
|   |   |-- axios.ts
|   |   |-- auth.api.ts
|   |   |-- transaction.api.ts
|   |   |-- wallet.api.ts
|   |   `-- dashboard.api.ts
|   |
|   |-- components/
|   |   |-- ui/
|   |   |-- layout/
|   |   |-- dashboard/
|   |   `-- transaction/
|   |
|   |-- views/
|   |   |-- LoginView.vue
|   |   |-- DashboardView.vue
|   |   |-- TransactionView.vue
|   |   |-- WalletView.vue
|   |   |-- CategoryView.vue
|   |   |-- BudgetView.vue
|   |   |-- SavingsView.vue
|   |   |-- AiAssistantView.vue
|   |   `-- SettingsView.vue
|   |
|   |-- router/
|   |-- stores/
|   |   |-- auth.store.ts
|   |   `-- group.store.ts
|   |
|   |-- types/
|   |-- App.vue
|   `-- main.ts
|
|-- index.html
|-- package.json
|-- vite.config.ts
`-- tailwind.config.ts
```

Frontend environment variable:

```env
VITE_API_URL=https://tracku-backend.vercel.app
```

Catatan:

- Semua request API menggunakan base URL dari `VITE_API_URL`.
- Frontend tidak boleh memiliki `DATABASE_URL`.
- Token JWT dikirim lewat header `Authorization: Bearer <token>`.
- Untuk MVP GitHub Pages, Vue Router wajib menggunakan `createWebHashHistory()`. Contoh URL halaman: `https://username.github.io/tracku-frontend/#/dashboard`.
- `base` pada Vite tetap diatur sesuai nama repository agar aset statis dapat dimuat dengan benar.

## 8.4 Backend Architecture

Backend menggunakan Node.js, Express.js, dan TypeScript.

Stack backend:

| Area | Teknologi |
|---|---|
| Runtime | Node.js |
| Framework | Express.js |
| Language | TypeScript |
| Validation | Zod |
| ORM | Drizzle ORM |
| Auth | JWT Bearer |
| Authorization | Group-based RBAC |
| Database | Neon PostgreSQL |
| Hosting | Vercel serverless/functions |

Struktur backend yang direkomendasikan:

```text
tracku-backend/
|
|-- api/
|   `-- index.ts
|
|-- src/
|   |-- app.ts
|   |
|   |-- config/
|   |   `-- env.ts
|   |
|   |-- db/
|   |   |-- index.ts
|   |   `-- schema/
|   |       |-- users.ts
|   |       |-- groups.ts
|   |       |-- groupMembers.ts
|   |       |-- wallets.ts
|   |       |-- categories.ts
|   |       |-- transactions.ts
|   |       |-- budgets.ts
|   |       |-- savingsGoals.ts
|   |       |-- savingsContributions.ts
|   |       `-- aiChatLogs.ts
|   |
|   |-- middleware/
|   |   |-- auth.middleware.ts
|   |   |-- rbac.middleware.ts
|   |   |-- error.middleware.ts
|   |   `-- cors.middleware.ts
|   |
|   |-- modules/
|   |   |-- auth/
|   |   |-- groups/
|   |   |-- wallets/
|   |   |-- categories/
|   |   |-- transactions/
|   |   |-- budgets/
|   |   |-- savings/
|   |   |-- dashboard/
|   |   `-- ai/
|   |
|   |-- types/
|   `-- utils/
|
|-- drizzle/
|-- drizzle.config.ts
|-- package.json
|-- tsconfig.json
`-- vercel.json
```

Backend environment variable:

```env
DATABASE_URL=postgresql://user:password@ep-xxxx-pooler.region.aws.neon.tech/dbname?sslmode=require
JWT_SECRET=change-this-secret
CORS_ORIGIN=https://username.github.io
```

Catatan Neon:

- Backend menggunakan `DATABASE_URL`.
- Untuk Vercel serverless, gunakan pooled connection Neon untuk mengurangi risiko terlalu banyak koneksi database.
- Hostname pooled connection Neon biasanya mengandung `-pooler`.
- Database credential hanya boleh berada di backend/Vercel environment variables.

## 8.5 REST API Communication

Frontend memanggil backend melalui REST API HTTPS.

Contoh base URL:

```text
https://tracku-backend.vercel.app
```

Contoh endpoint:

```text
POST   /api/auth/register
POST   /api/auth/login
GET    /api/groups
POST   /api/groups
GET    /api/groups/:groupId/members
POST   /api/groups/:groupId/members
GET    /api/wallets?groupId=:groupId
POST   /api/wallets
GET    /api/categories?groupId=:groupId
POST   /api/categories
GET    /api/transactions?groupId=:groupId
POST   /api/transactions
PATCH  /api/transactions/:id
DELETE /api/transactions/:id
GET    /api/dashboard/summary?groupId=:groupId
GET    /api/dashboard/cashflow?groupId=:groupId
GET    /api/dashboard/expenses-by-category?groupId=:groupId
GET    /api/budgets?groupId=:groupId
POST   /api/budgets
GET    /api/savings-goals?groupId=:groupId
POST   /api/savings-goals
GET    /api/savings-goals/:goalId/contributions
POST   /api/savings-goals/:goalId/contributions
DELETE /api/savings-goals/:goalId/contributions/:contributionId
POST   /api/ai/chat
```

Format response umum:

```json
{
  "success": true,
  "data": {},
  "message": "OK"
}
```

Format error umum:

```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Input tidak valid",
    "details": {}
  }
}
```

## 9. Sequence Diagram

## 9.1 Login

```mermaid
sequenceDiagram
    participant U as User
    participant FE as Vue Frontend - GitHub Pages
    participant BE as Express Backend - Vercel Function
    participant DB as Neon PostgreSQL

    U->>FE: Input email dan password
    FE->>BE: POST /api/auth/login
    BE->>DB: Cari user berdasarkan email
    DB-->>BE: Data user
    BE->>BE: Validasi password dan buat JWT
    BE-->>FE: Access token + user profile
    FE->>FE: Simpan auth state
    FE-->>U: Masuk ke dashboard
```

## 9.2 Membuat Transaksi

```mermaid
sequenceDiagram
    participant U as User
    participant FE as Vue Frontend - GitHub Pages
    participant BE as Express Backend - Vercel Function
    participant DB as Neon PostgreSQL

    U->>FE: Submit form transaksi
    FE->>BE: POST /api/transactions dengan Bearer token
    BE->>BE: Validasi JWT
    BE->>BE: Validasi role dan akses group
    BE->>BE: Validasi input dengan Zod
    BE->>DB: Insert transaksi
    DB-->>BE: Transaksi tersimpan
    BE-->>FE: Response success
    FE->>FE: Refresh list dan dashboard state
    FE-->>U: Transaksi tampil
```

## 9.3 Dashboard

```mermaid
sequenceDiagram
    participant U as User
    participant FE as Vue Frontend - GitHub Pages
    participant BE as Express Backend - Vercel Function
    participant DB as Neon PostgreSQL

    U->>FE: Buka dashboard
    FE->>BE: GET /api/dashboard/summary?groupId=...
    BE->>BE: Validasi JWT dan akses group
    BE->>DB: Query ringkasan income, expense, saldo, budget
    DB-->>BE: Data agregasi
    BE-->>FE: JSON dashboard
    FE->>FE: Render card dan chart
    FE-->>U: Dashboard tampil
```

## 9.4 AI Finance Assistant

```mermaid
sequenceDiagram
    participant U as User
    participant FE as Vue Frontend - GitHub Pages
    participant BE as Express Backend - Vercel Function
    participant DB as Neon PostgreSQL
    participant LLM as LLM API

    U->>FE: Bertanya tentang keuangan
    FE->>BE: POST /api/ai/chat dengan Bearer token
    BE->>BE: Validasi JWT dan akses group
    BE->>LLM: Kirim prompt dan daftar function yang aman
    LLM-->>BE: Pilih function jika butuh data
    BE->>DB: Query data melalui service internal
    DB-->>BE: Data terbatas sesuai kebutuhan
    BE->>LLM: Kirim hasil function
    LLM-->>BE: Jawaban natural language
    BE-->>FE: Jawaban AI
    FE-->>U: Tampilkan jawaban
```

## 10. Data Model Awal

Entitas utama:

```text
users
groups
group_members
wallets
categories
transactions
budgets
savings_goals
savings_contributions
ai_chat_logs
```

## 10.1 users

| Field | Tipe | Keterangan |
|---|---|---|
| id | uuid | Primary key |
| name | varchar | Nama user |
| email | varchar | Email unik |
| password_hash | text | Password hash |
| created_at | timestamp | Waktu dibuat |
| updated_at | timestamp | Waktu diubah |

## 10.2 groups

| Field | Tipe | Keterangan |
|---|---|---|
| id | uuid | Primary key |
| name | varchar | Nama grup |
| owner_id | uuid | User pembuat/pemilik |
| created_at | timestamp | Waktu dibuat |
| updated_at | timestamp | Waktu diubah |

## 10.3 group_members

| Field | Tipe | Keterangan |
|---|---|---|
| id | uuid | Primary key |
| group_id | uuid | Relasi ke groups |
| user_id | uuid | Relasi ke users |
| role | varchar | owner/admin/member/viewer |
| created_at | timestamp | Waktu dibuat |

## 10.4 wallets

| Field | Tipe | Keterangan |
|---|---|---|
| id | uuid | Primary key |
| group_id | uuid | Relasi ke groups |
| name | varchar | Nama wallet |
| type | varchar | cash/bank/e-wallet/saving/other |
| initial_balance | numeric | Saldo awal |
| created_at | timestamp | Waktu dibuat |
| updated_at | timestamp | Waktu diubah |

## 10.5 categories

| Field | Tipe | Keterangan |
|---|---|---|
| id | uuid | Primary key |
| group_id | uuid | Relasi ke groups |
| name | varchar | Nama kategori |
| type | varchar | income/expense |
| color | varchar | Warna kategori |
| icon | varchar | Icon kategori opsional |
| created_at | timestamp | Waktu dibuat |
| updated_at | timestamp | Waktu diubah |

## 10.6 transactions

| Field | Tipe | Keterangan |
|---|---|---|
| id | uuid | Primary key |
| group_id | uuid | Relasi ke groups |
| wallet_id | uuid | Relasi ke wallets |
| category_id | uuid | Relasi ke categories |
| type | varchar | income/expense |
| amount | numeric | Nominal |
| transaction_date | date | Tanggal transaksi |
| note | text | Catatan opsional |
| created_by | uuid | User pembuat |
| created_at | timestamp | Waktu dibuat |
| updated_at | timestamp | Waktu diubah |

## 10.7 budgets

| Field | Tipe | Keterangan |
|---|---|---|
| id | uuid | Primary key |
| group_id | uuid | Relasi ke groups |
| category_id | uuid | Relasi ke categories |
| month | varchar | Format YYYY-MM |
| limit_amount | numeric | Limit budget |
| created_at | timestamp | Waktu dibuat |
| updated_at | timestamp | Waktu diubah |

## 10.8 savings_goals

| Field | Tipe | Keterangan |
|---|---|---|
| id | uuid | Primary key |
| group_id | uuid | Relasi ke groups |
| name | varchar | Nama target |
| target_amount | numeric | Nominal target |
| target_date | date | Tanggal target |
| created_at | timestamp | Waktu dibuat |
| updated_at | timestamp | Waktu diubah |

Progress target dihitung dari total nilai `amount` pada `savings_contributions` yang terkait dengan target tersebut.

## 10.9 savings_contributions

| Field | Tipe | Keterangan |
|---|---|---|
| id | uuid | Primary key |
| group_id | uuid | Relasi ke groups untuk validasi akses |
| savings_goal_id | uuid | Relasi ke savings_goals |
| amount | numeric | Nominal kontribusi; harus lebih dari 0 |
| contribution_date | date | Tanggal kontribusi |
| note | text | Catatan opsional |
| created_by | uuid | User yang membuat kontribusi |
| created_at | timestamp | Waktu dibuat |
| updated_at | timestamp | Waktu diubah |

## 10.10 ai_chat_logs

| Field | Tipe | Keterangan |
|---|---|---|
| id | uuid | Primary key |
| group_id | uuid | Relasi ke groups |
| user_id | uuid | Relasi ke users |
| question | text | Pertanyaan user |
| answer | text | Jawaban AI |
| created_at | timestamp | Waktu dibuat |

## 11. UI/UX Requirement

## 11.1 Layout Umum

Aplikasi menggunakan layout dashboard modern dan responsive.

Komponen utama:

- Sidebar navigation untuk desktop.
- Bottom navigation atau collapsible menu untuk mobile.
- Header dengan active group selector.
- Main content area.
- Card ringkasan keuangan.
- Chart area.
- Table transaksi.
- Modal atau drawer untuk form.

## 11.2 Halaman MVP

Halaman yang dibutuhkan:

1. Login.
2. Register.
3. Dashboard.
4. Transactions.
5. Wallets.
6. Categories.
7. Budgets.
8. Savings Goals.
9. AI Assistant.
10. Group Settings.
11. Profile/Settings.

## 11.3 Prinsip Desain

Prinsip desain:

- Clean dan modern.
- Mudah dibaca.
- Fokus ke data dan workflow.
- Responsive untuk desktop, tablet, dan mobile.
- Form transaksi harus cepat digunakan.
- Dashboard harus mudah dipahami dalam beberapa detik.

## 12. Technical Constraints

## 12.1 Multi-Repo

Aplikasi wajib menggunakan dua repository:

```text
tracku-frontend
tracku-backend
```

Alasan:

- Deployment frontend dan backend terpisah.
- Frontend dapat dideploy ke GitHub Pages.
- Backend dapat dideploy ke Vercel.
- Lifecycle pengembangan lebih jelas.

## 12.2 Frontend Static Hosting

Frontend dideploy sebagai static site ke GitHub Pages.

Konsekuensi:

- Frontend tidak menjalankan server sendiri.
- Semua logic backend harus dipanggil melalui REST API.
- Tidak ada akses langsung ke database.
- Build output berada di folder `dist`.
- Untuk MVP, Vue Router wajib menggunakan hash history (`createWebHashHistory()`), sehingga refresh halaman tidak meminta path aplikasi secara langsung ke GitHub Pages.
- `base` Vite wajib disesuaikan dengan nama repository untuk path aset statis.
- Environment variable frontend harus menggunakan prefix `VITE_`.

Env frontend:

```env
VITE_API_URL=https://tracku-backend.vercel.app
```

## 12.3 Backend Serverless di Vercel

Backend dideploy ke Vercel sebagai serverless/functions.

Konsekuensi:

- Backend berjalan saat ada request.
- Jangan mengandalkan proses Node.js yang hidup terus-menerus.
- Jangan menyimpan state penting di memory global.
- Gunakan database atau storage eksternal untuk data permanen.
- Cocok untuk REST API request-response.

Tidak cocok untuk:

- Worker yang harus hidup 24 jam.
- WebSocket server permanen.
- Background process persisten.
- Queue consumer yang berjalan terus-menerus.
- Cron/job berat yang durasinya panjang.
- Proses yang bergantung pada `setInterval` permanen.

Cocok untuk aplikasi ini karena fitur MVP adalah:

- Login.
- CRUD transaksi.
- CRUD wallet.
- CRUD kategori.
- Dashboard query.
- Budget.
- Target tabungan.
- AI chat berbasis request-response.

## 12.4 Database Neon PostgreSQL

Database menggunakan PostgreSQL dari Neon.

Ketentuan:

- Backend menggunakan `DATABASE_URL`.
- Gunakan pooled connection untuk runtime Vercel.
- Migration dikelola dengan Drizzle.
- Database tidak boleh diakses langsung dari frontend.
- Credential database hanya disimpan di Vercel environment variables dan local `.env` backend.

Env backend:

```env
DATABASE_URL=postgresql://user:password@ep-xxxx-pooler.region.aws.neon.tech/dbname?sslmode=require
```

## 12.5 CORS

Karena frontend dan backend berada pada domain berbeda, backend wajib mengaktifkan CORS.

Contoh origin:

```text
Local frontend: http://localhost:5173
Production frontend: https://username.github.io
Production backend: https://tracku-backend.vercel.app
```

Aturan:

- Production origin harus dibatasi ke domain frontend resmi.
- Hindari `origin: "*"` untuk endpoint yang menggunakan authentication.
- Backend harus menangani preflight request.

## 12.6 Authentication dan Authorization

MVP menggunakan JWT Bearer auth.

Ketentuan:

- Access token dibuat saat login.
- Frontend mengirim token lewat header `Authorization`.
- Backend memvalidasi JWT di endpoint yang protected.
- Backend mengambil `user_id` dari token.
- Backend memvalidasi membership dan role terhadap `group_id`.

RBAC wajib diterapkan di backend, bukan hanya di frontend.

## 12.7 Inertia.js

Inertia.js tidak digunakan.

Alasan:

- Frontend dan backend dipisah ke repository berbeda.
- Frontend adalah SPA static di GitHub Pages.
- Backend adalah REST API di Vercel.
- Inertia.js lebih cocok untuk pendekatan server-driven atau monolith-style, sedangkan arsitektur final memakai REST API terpisah.

## 13. Deployment

## 13.1 Frontend Deployment ke GitHub Pages

Flow:

```text
Developer push ke GitHub
        |
        v
GitHub Actions menjalankan build
        |
        v
npm install
npm run build
        |
        v
Output dist/
        |
        v
Deploy ke GitHub Pages
```

Checklist:

- Repository frontend sudah dibuat.
- GitHub Pages aktif.
- GitHub Actions workflow tersedia.
- `VITE_API_URL` production sudah diset.
- `vite.config.ts` mengatur base path sesuai nama repo jika memakai URL GitHub Pages default.

Contoh URL:

```text
https://username.github.io/tracku-frontend/
```

## 13.2 Backend Deployment ke Vercel

Flow:

```text
Developer push ke GitHub repo backend
        |
        v
Vercel menarik source backend
        |
        v
Install dependencies
        |
        v
Build TypeScript jika diperlukan
        |
        v
Deploy sebagai Vercel Functions
```

Checklist:

- Repository backend terpisah sudah dibuat.
- Project backend sudah diimport ke Vercel.
- Environment variables sudah diset di Vercel.
- `DATABASE_URL` menggunakan Neon pooled connection.
- `JWT_SECRET` sudah diset.
- `CORS_ORIGIN` mengarah ke URL frontend GitHub Pages.
- Endpoint API bisa diakses lewat HTTPS.

Contoh URL:

```text
https://tracku-backend.vercel.app
```

## 13.3 Database Deployment di Neon

Flow:

```text
Buat project Neon
        |
        v
Buat database PostgreSQL
        |
        v
Ambil pooled connection string
        |
        v
Set sebagai DATABASE_URL di Vercel
        |
        v
Jalankan Drizzle migration
```

Checklist:

- Neon project dibuat.
- Database dibuat.
- Pooled connection string tersedia.
- SSL mode aktif.
- Migration berhasil dijalankan.

## 14. Environment Variables

## 14.1 Frontend

| Variable | Contoh | Keterangan |
|---|---|---|
| VITE_API_URL | https://tracku-backend.vercel.app | Base URL backend API |

## 14.2 Backend

| Variable | Contoh | Keterangan |
|---|---|---|
| DATABASE_URL | postgresql://...-pooler...neon.tech/... | Connection string Neon pooled |
| JWT_SECRET | random-secret | Secret untuk sign JWT |
| CORS_ORIGIN | https://username.github.io | Origin frontend production |
| NODE_ENV | production | Environment runtime |
| LLM_API_KEY | sk-... | API key untuk AI assistant jika digunakan |

## 15. Security Requirement

Security requirement:

1. Password harus di-hash sebelum disimpan.
2. Database credential tidak boleh masuk frontend.
3. JWT secret tidak boleh masuk repository.
4. Endpoint protected wajib memvalidasi JWT.
5. Akses data grup wajib memvalidasi membership.
6. Role user wajib dicek di backend.
7. Input wajib divalidasi dengan Zod.
8. Error response tidak boleh membocorkan detail sensitif.
9. CORS production harus dibatasi.
10. AI tidak boleh mengakses SQL mentah.

## 16. Non-Functional Requirement

## 16.1 Performance

- Dashboard harus terasa cepat untuk data MVP.
- Query agregasi dashboard perlu dioptimalkan dengan index.
- Pagination digunakan untuk daftar transaksi.
- Chart hanya mengambil data sesuai periode yang dibutuhkan.

## 16.2 Scalability

- Backend stateless agar cocok di serverless.
- Database menggunakan Neon pooled connection.
- File upload tidak masuk MVP.
- Data besar harus memakai pagination dan filter.

## 16.3 Maintainability

- Frontend dipisah berdasarkan view, component, store, dan API client.
- Backend dipisah berdasarkan module.
- Validasi request menggunakan Zod schema.
- Query database dikelola melalui Drizzle ORM.
- Migration database disimpan di repo backend.

## 16.4 Reliability

- Backend harus memiliki error handling middleware.
- Frontend harus menangani loading, empty state, dan error state.
- API response memiliki format konsisten.
- Token expired harus ditangani dengan redirect login atau mekanisme re-login.

## 17. Prioritas Implementasi

Urutan implementasi yang disarankan:

1. Setup repository frontend dan backend.
2. Setup database Neon dan Drizzle migration.
3. Implement auth dan JWT.
4. Implement group dan RBAC dasar.
5. Implement wallet.
6. Implement kategori.
7. Implement transaksi.
8. Implement dashboard.
9. Implement budget.
10. Implement target tabungan.
11. Implement AI assistant.
12. Setup deployment GitHub Pages.
13. Setup deployment Vercel.
14. Testing end-to-end.

## 18. Risiko dan Mitigasi

| Risiko | Dampak | Mitigasi |
|---|---|---|
| CORS salah konfigurasi | Frontend gagal memanggil API | Set `CORS_ORIGIN` sesuai domain frontend |
| DATABASE_URL bocor ke frontend | Credential database terekspos | Simpan hanya di backend/Vercel |
| Serverless membuka terlalu banyak koneksi DB | Error koneksi database | Gunakan Neon pooled connection |
| Vue Router bermasalah di GitHub Pages | Refresh halaman menghasilkan 404 | Gunakan `createWebHashHistory()` untuk MVP dan atur `base` Vite untuk aset statis |
| JWT disimpan kurang aman | Risiko token dicuri lewat XSS | Perketat sanitasi, hindari inject HTML, evaluasi HttpOnly cookie di versi lanjut |
| AI mengakses data berlebihan | Risiko privasi | Batasi function dan scope data berdasarkan group/user |

## 19. Acceptance Criteria MVP

MVP dianggap selesai jika:

1. User dapat register dan login.
2. User dapat membuat dan memilih group.
3. Role user dalam group dapat membatasi akses.
4. User dapat membuat wallet.
5. User dapat membuat kategori.
6. User dapat mencatat pemasukan dan pengeluaran.
7. Dashboard menampilkan ringkasan keuangan.
8. User dapat membuat budget.
9. User dapat membuat target tabungan.
10. User dapat bertanya ke AI assistant dengan data terbatas dan aman.
11. Frontend berhasil dideploy ke GitHub Pages.
12. Backend berhasil dideploy ke Vercel.
13. Backend berhasil terhubung ke Neon PostgreSQL melalui `DATABASE_URL`.
14. Frontend berhasil memanggil backend melalui `VITE_API_URL`.
15. CORS production berjalan dengan domain frontend resmi.

## 20. Kesimpulan

Arsitektur final aplikasi adalah:

```text
Vue 3 SPA di GitHub Pages
        |
        | REST API HTTPS
        v
Express.js Backend di Vercel Serverless/Functions
        |
        | Drizzle ORM + DATABASE_URL pooled connection
        v
Neon PostgreSQL
```

Keputusan ini cocok untuk kebutuhan MVP karena aplikasi didominasi oleh workflow request-response seperti CRUD transaksi, dashboard, budget, savings goal, dan AI chat.

Inertia.js dihapus dari stack karena frontend dan backend kini dipisahkan menjadi dua repository dan berkomunikasi melalui REST API HTTPS.
