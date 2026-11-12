# Maem Bolo - Sistem Review Kantin (Tugas 4)

Aplikasi full-stack untuk menjelajahi warung kantin, membaca dan menulis review, serta mengelola warung berdasarkan peran pengguna. Frontend React memanggil REST API Express melalui **Axios**, dan backend menyimpan data di **SQL Server**.

Dikerjakan oleh: [isi nama] - [isi NIM] - [isi kelas]

## Teknologi

| Bagian | Teknologi |
|---|---|
| Frontend | React 19, TypeScript, Vite, Tailwind CSS v4, React Router, Axios, Lucide |
| Backend | Express 5, TypeScript, Zod, `mssql`, `jsonwebtoken`, `bcryptjs`, Swagger UI |
| Database | SQL Server (database `review_kantin`) |
| Autentikasi | JWT di cookie HttpOnly `kantin_session` (tanpa localStorage/sessionStorage) |

## Fitur Utama

- **Publik:** daftar warung dengan pencarian nama dan filter kategori, detail warung beserta menu dan review, registrasi, dan login.
- **Customer:** menulis, mengubah, dan menghapus review sendiri; like; melaporkan review; mengelola profil.
- **Owner:** membuat warung, serta mengubah, menghapus, dan mengelola menu pada warung miliknya.
- **Admin:** mengelola semua warung dan pengguna, memproses laporan review, dan melihat audit log.

Hak akses dijaga di dua lapisan: `ProtectedRoute` di frontend dan pengecekan role serta kepemilikan di backend. Respons `401` mengembalikan pengguna ke halaman login, sedangkan `403` tidak menghapus sesi.

## Prasyarat

- Node.js 20 atau lebih baru dan npm
- SQL Server yang sudah berisi database `review_kantin` (skema dan data dari Tugas 2). Tidak ada perubahan skema pada tugas ini.

## Instalasi

```bash
git clone <URL-repository>
cd <nama-folder-repository>

cd backend && npm install
cd ../frontend && npm install
```

## Konfigurasi Environment

Salin file contoh lalu isi nilainya. **Jangan commit file `.env`.**

```bash
# dari folder root repository
cp backend/.env.example backend/.env
cp frontend/.env.example frontend/.env
# Windows (cmd): copy backend\.env.example backend\.env
```

**`backend/.env`**

```dotenv
NODE_ENV=development
PORT=3000
CORS_ORIGIN=http://localhost:5173
AUTH_COOKIE_NAME=kantin_session

DB_SERVER=localhost
DB_PORT=1433
DB_USER=<user-sql-server>
DB_PASSWORD=<password-sql-server>
DB_NAME=review_kantin
DB_ENCRYPT=false
DB_TRUST_SERVER_CERTIFICATE=true

JWT_SECRET=<string-acak-panjang>
JWT_EXPIRES_IN=2h
BCRYPT_SALT_ROUNDS=10
```

**`frontend/.env`**

```dotenv
VITE_API_BASE_URL=http://localhost:3000/api
```

`CORS_ORIGIN` harus sama persis dengan alamat frontend (host dan port). Jika frontend dibuka lewat `127.0.0.1:5173`, isi `CORS_ORIGIN` dengan alamat tersebut.

## Menjalankan Aplikasi

Buka dua terminal.

```bash
# Terminal 1 - backend (http://localhost:3000)
cd backend
npm run typecheck
npm run dev
```

```bash
# Terminal 2 - frontend (http://localhost:5173)
cd frontend
npm run lint
npm run build
npm run dev
```

| Alamat | Keterangan |
|---|---|
| `http://localhost:5173` | Aplikasi web |
| `http://localhost:3000/api-docs` | Dokumentasi Swagger (login dulu agar cookie tersimpan) |
| `http://localhost:3000/api/health` | Pengecekan server dan koneksi database |

## Endpoint Warung dan Autentikasi

Base URL: `http://localhost:3000/api`

| Method | Path | Akses |
|---|---|---|
| GET | `/stalls` | Publik (query `search`, `category`) |
| GET | `/stalls/:id` | Publik |
| POST | `/stalls` | Admin, Owner (Owner otomatis menjadi pemilik) |
| PUT | `/stalls/:id` | Admin, Owner pemilik |
| DELETE | `/stalls/:id` | Admin, Owner pemilik |
| POST | `/auth/register` | Publik |
| POST | `/auth/login` | Publik |
| GET | `/auth/me` | Login |
| POST | `/auth/logout` | Publik (menghapus cookie) |

Daftar endpoint lengkap (menu, review, like, flag, user, audit log) tersedia di Swagger.

## Pengujian Manual Singkat

1. Buka daftar dan detail warung tanpa login.
2. Registrasi customer, login, lalu muat ulang halaman; sesi harus tetap aktif.
3. Customer membuka `/admin` atau `/owner` sehingga diarahkan ke halaman Akses Ditolak, sesi tetap aktif.
4. Owner membuat dan mengubah warung miliknya; mengubah atau menghapus warung owner lain harus `403`.
5. Admin menambah, mengubah, dan menghapus warung; cek perubahan pada tabel `stalls` di SQL Server.
6. Hapus cookie `kantin_session`, lalu buka halaman terproteksi; frontend mengarahkan ke login.

## Catatan

- `GET /stalls` belum mendukung pagination, sehingga frontend memuat seluruh data.
- Belum ada automated test; verifikasi dilakukan manual.