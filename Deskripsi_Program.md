# SUSI SAKUCI — Deskripsi Program

## Sistem Pengaduan Sarana dan Prasarana Sekolah

---

## 1. Gambaran Umum Sistem

**SUSI SAKUCI** (Suara Siswa SMK Sangkuriang 1 Cimah) adalah sistem pengaduan sarana dan prasarana sekolah berbasis web yang memungkinkan siswa melaporkan kerusakan, keluhan, dan saran terkait fasilitas sekolah secara digital. Sistem ini menggunakan arsitektur **Single Page Application (SPA)** dengan backend API terpisah.

### Tujuan Sistem

- Menyediakan platform digital bagi siswa untuk melaporkan kerusakan sarana sekolah
- Mempermudah admin dalam mengelola dan menindaklanjuti pengaduan
- Memberikan transparansi status penanganan pengaduan kepada siswa
- Menyediakan data statistik untuk analisis kualitas fasilitas sekolah
- Memfasilitasi saran/kritik publik dari alumni, orang tua, dan masyarakat umum

### Pengguna Sistem

| Role             | Deskripsi                                |
| ---------------- | ---------------------------------------- |
| **Siswa**        | Melaporkan pengaduan dan memantau status |
| **Admin Sistem** | Mereview pengaduan masuk (terima/tolak)  |
| **Pelaksana**    | Menindaklanjuti pengaduan yang diterima  |

---

## 2. Technology Stack

### Backend

| Komponen  | Teknologi        | Versi |
| --------- | ---------------- | ----- |
| Framework | Laravel          | 11.x  |
| Bahasa    | PHP              | 8.2+  |
| Database  | MySQL            | 8.0+  |
| API       | RESTful JSON API | -     |

### Frontend

| Komponen         | Teknologi                | Versi |
| ---------------- | ------------------------ | ----- |
| Framework        | React                    | 18.x  |
| Bahasa           | TypeScript               | 5.x   |
| Build Tool       | Vite                     | 5.x   |
| Styling          | Tailwind CSS + shadcn/ui | 3.x   |
| State Management | React Query (TanStack)   | 5.x   |
| Charts           | Recharts                 | 2.x   |
| Routing          | React Router             | 6.x   |
| HTTP Client      | Axios                    | 1.x   |

---

## 3. Arsitektur Sistem

```
┌────────────────────┐   HTTP/JSON    ┌────────────────────┐
│   React Frontend   │ ◄────────────► │   Laravel Backend   │
│   (Vite + TS)      │   REST API     │   (PHP 8.2)         │
└────────────────────┘                └──────────┬─────────┘
                                                 │
                                      ┌──────────▼─────────┐
                                      │   MySQL Database    │
                                      │   (6 tabel utama)   │
                                      └────────────────────┘
```

### Data Flow

1. **Frontend** mengirim request ke API endpoint Laravel
2. **Controller** memproses request, validasi, dan query database
3. **Model** (Eloquent ORM) menangani interaksi database
4. **JSON Response** dikirim kembali ke frontend
5. **React Query** mengelola caching dan state management

---

## 4. Database Schema

### 4.1 Tabel `siswa` — Data Siswa

| Kolom             | Tipe         | Keterangan          |
| ----------------- | ------------ | ------------------- |
| `nis` (PK)        | BIGINT       | Nomor Induk Siswa   |
| `nama`            | VARCHAR(100) | Nama lengkap        |
| `kelas`           | VARCHAR(20)  | Kelas siswa         |
| `password`        | VARCHAR(255) | Password login      |
| `is_active`       | BOOLEAN      | Status aktif        |
| `dibuat_pada`     | TIMESTAMP    | Tanggal dibuat      |
| `terakhir_update` | TIMESTAMP    | Terakhir diperbarui |

### 4.2 Tabel `admin` — Data Admin/Pelaksana

| Kolom           | Tipe                             | Keterangan     |
| --------------- | -------------------------------- | -------------- |
| `id_admin` (PK) | BIGINT AUTO_INCREMENT            | ID Admin       |
| `nama_admin`    | VARCHAR(100)                     | Nama admin     |
| `username`      | VARCHAR(50) UNIQUE               | Username login |
| `password`      | VARCHAR(255)                     | Password login |
| `posisi`        | ENUM('admin_sistem','pelaksana') | Role admin     |

### 4.3 Tabel `kategori` — Kategori Pengaduan

| Kolom              | Tipe                  | Keterangan    |
| ------------------ | --------------------- | ------------- |
| `id_kategori` (PK) | BIGINT AUTO_INCREMENT | ID Kategori   |
| `ket_kategori`     | VARCHAR(100)          | Nama kategori |

### 4.4 Tabel `input_aspirasi` — Pengaduan Siswa

| Kolom               | Tipe                                 | Keterangan                        |
| ------------------- | ------------------------------------ | --------------------------------- |
| `id_pelaporan` (PK) | BIGINT AUTO_INCREMENT                | ID Pengaduan                      |
| `nis` (FK)          | BIGINT                               | NIS pelapor → `siswa.nis`         |
| `id_kategori` (FK)  | BIGINT                               | Kategori → `kategori.id_kategori` |
| `lokasi`            | VARCHAR(255)                         | Lokasi kerusakan                  |
| `foto_dokumentasi`  | VARCHAR(255)                         | Path foto bukti                   |
| `keterangan`        | TEXT                                 | Deskripsi lengkap                 |
| `status_review`     | ENUM('pending','diterima','ditolak') | Status review admin sistem        |
| `created_at`        | TIMESTAMP                            | Tanggal dibuat                    |

### 4.5 Tabel `aspirasi` — Tanggapan/Tindak Lanjut

| Kolom               | Tipe                                | Keterangan                                |
| ------------------- | ----------------------------------- | ----------------------------------------- |
| `id_aspirasi` (PK)  | BIGINT AUTO_INCREMENT               | ID Tanggapan                              |
| `id_pelaporan` (FK) | BIGINT                              | Pengaduan → `input_aspirasi.id_pelaporan` |
| `id_kategori` (FK)  | BIGINT                              | Kategori → `kategori.id_kategori`         |
| `id_admin` (FK)     | BIGINT                              | Admin pelaksana → `admin.id_admin`        |
| `status`            | ENUM('Menunggu','Proses','Selesai') | Status penanganan                         |
| `feedback`          | TEXT                                | Catatan singkat                           |
| `detail_tanggapan`  | TEXT                                | Detail tanggapan lengkap                  |

### 4.6 Tabel `saran_publiks` — Saran dari Publik

| Kolom               | Tipe                                                   | Keterangan                        |
| ------------------- | ------------------------------------------------------ | --------------------------------- |
| `id` (PK)           | BIGINT AUTO_INCREMENT                                  | ID Saran                          |
| `nama_pengirim`     | VARCHAR(100)                                           | Nama pengirim                     |
| `email`             | VARCHAR(100) NULL                                      | Email (opsional)                  |
| `no_telepon`        | VARCHAR(20) NULL                                       | Telepon (opsional)                |
| `kategori_pengirim` | ENUM('Alumni','Orang Tua','Masyarakat Umum','Lainnya') | Jenis pengirim                    |
| `id_kategori` (FK)  | BIGINT NULL                                            | Kategori → `kategori.id_kategori` |
| `isi_saran`         | TEXT                                                   | Isi saran/kritik                  |
| `status`            | ENUM('Baru','Dibaca','Ditindaklanjuti')                | Status saran                      |

### Relasi Antar Tabel

```
siswa (1) ──────── (N) input_aspirasi
kategori (1) ──────── (N) input_aspirasi
kategori (1) ──────── (N) aspirasi
input_aspirasi (1) ── (N) aspirasi
admin (1) ──────── (N) aspirasi
kategori (1) ──────── (N) saran_publiks
```

### Database Indexes (Performance)

| Tabel            | Kolom                       | Tipe Index      |
| ---------------- | --------------------------- | --------------- |
| `input_aspirasi` | `nis`                       | INDEX           |
| `input_aspirasi` | `id_kategori`               | INDEX           |
| `input_aspirasi` | `status_review`             | INDEX           |
| `input_aspirasi` | `created_at`                | INDEX           |
| `input_aspirasi` | `status_review, created_at` | COMPOSITE INDEX |
| `aspirasi`       | `id_pelaporan`              | INDEX           |
| `aspirasi`       | `id_kategori`               | INDEX           |
| `aspirasi`       | `id_admin`                  | INDEX           |
| `aspirasi`       | `status`                    | INDEX           |

---

## 5. Alur Kerja Sistem

### Alur Pengaduan Siswa

```
Siswa Login → Buat Pengaduan → Upload Foto
       │
       ▼
Admin Sistem Review → Terima ──→ Pelaksana Proses → Selesai
                    → Tolak ──→ Arsip
                    → Auto-arsip (>2 minggu pending)
```

### Detail Alur

1. **Siswa** login dan membuat pengaduan dengan lokasi, kategori, foto, dan keterangan
2. **Admin Sistem** mereview pengaduan baru (status: `pending`)
3. Jika **diterima** (status: `diterima`): pengaduan muncul di halaman Pelaksana
4. Jika **ditolak** (status: `ditolak`): pengaduan masuk ke arsip
5. **Pelaksana** membuat tanggapan → status berubah (`Menunggu` → `Proses` → `Selesai`)
6. **Auto-arsip**: pengaduan `pending` > 2 minggu otomatis diarsipkan

---

## 6. API Endpoints

### Auth (Login)

| Method | Endpoint           | Keterangan  |
| ------ | ------------------ | ----------- |
| POST   | `/api/siswa/login` | Login siswa |
| POST   | `/api/admin/login` | Login admin |

### Siswa

| Method | Endpoint           | Keterangan       |
| ------ | ------------------ | ---------------- |
| GET    | `/api/siswa`       | List semua siswa |
| GET    | `/api/siswa/{nis}` | Detail siswa     |
| POST   | `/api/siswa`       | Tambah siswa     |
| PUT    | `/api/siswa/{nis}` | Update siswa     |
| DELETE | `/api/siswa/{nis}` | Hapus siswa      |

### Admin

| Method | Endpoint          | Keterangan       |
| ------ | ----------------- | ---------------- |
| GET    | `/api/admin`      | List semua admin |
| GET    | `/api/admin/{id}` | Detail admin     |
| POST   | `/api/admin`      | Tambah admin     |
| PUT    | `/api/admin/{id}` | Update admin     |
| DELETE | `/api/admin/{id}` | Hapus admin      |

### Kategori

| Method | Endpoint             | Keterangan      |
| ------ | -------------------- | --------------- |
| GET    | `/api/kategori`      | List kategori   |
| GET    | `/api/kategori/{id}` | Detail kategori |
| POST   | `/api/kategori`      | Tambah kategori |
| PUT    | `/api/kategori/{id}` | Update kategori |
| DELETE | `/api/kategori/{id}` | Hapus kategori  |

### Pengaduan (Input Aspirasi)

| Method | Endpoint                          | Keterangan                           |
| ------ | --------------------------------- | ------------------------------------ |
| GET    | `/api/input-aspirasi`             | List semua pengaduan                 |
| GET    | `/api/input-aspirasi/recent`      | **[NEW]** Pengaduan terbaru (ringan) |
| GET    | `/api/input-aspirasi/{id}`        | Detail pengaduan                     |
| GET    | `/api/input-aspirasi/siswa/{nis}` | Pengaduan per siswa                  |
| POST   | `/api/input-aspirasi`             | Buat pengaduan                       |
| PUT    | `/api/input-aspirasi/{id}`        | Update pengaduan                     |
| DELETE | `/api/input-aspirasi/{id}`        | Hapus pengaduan                      |

### Aspirasi (Tanggapan)

| Method | Endpoint                        | Keterangan          |
| ------ | ------------------------------- | ------------------- |
| GET    | `/api/aspirasi`                 | List tanggapan      |
| GET    | `/api/aspirasi/{id}`            | Detail tanggapan    |
| GET    | `/api/aspirasi/admin/{id}`      | Tanggapan per admin |
| GET    | `/api/aspirasi/status/{status}` | Filter status       |
| POST   | `/api/aspirasi`                 | Buat tanggapan      |
| PUT    | `/api/aspirasi/{id}`            | Update tanggapan    |
| DELETE | `/api/aspirasi/{id}`            | Hapus tanggapan     |
| PATCH  | `/api/aspirasi/{id}/status`     | Update status       |
| PATCH  | `/api/aspirasi/{id}/feedback`   | Update feedback     |

### Review (Admin Sistem)

| Method | Endpoint                   | Keterangan        |
| ------ | -------------------------- | ----------------- |
| GET    | `/api/review/pending`      | Pengaduan pending |
| PATCH  | `/api/review/{id}/accept`  | Terima pengaduan  |
| PATCH  | `/api/review/{id}/reject`  | Tolak pengaduan   |
| GET    | `/api/review/arsip`        | Arsip pengaduan   |
| POST   | `/api/review/auto-archive` | Auto-arsip manual |

### Statistik

| Method | Endpoint                   | Keterangan             |
| ------ | -------------------------- | ---------------------- |
| GET    | `/api/statistik/dashboard` | Dashboard stats        |
| GET    | `/api/statistik/trend`     | Tren pengaduan (chart) |
| GET    | `/api/statistik/category`  | Distribusi kategori    |
| GET    | `/api/statistik/status`    | Distribusi status      |
| GET    | `/api/statistik/summary`   | Ringkasan statistik    |

### Saran Publik

| Method | Endpoint                        | Keterangan           |
| ------ | ------------------------------- | -------------------- |
| POST   | `/api/saran-publik`             | Kirim saran (publik) |
| GET    | `/api/saran-publik`             | List saran (admin)   |
| GET    | `/api/saran-publik/statistics`  | Statistik saran      |
| GET    | `/api/saran-publik/{id}`        | Detail saran         |
| PATCH  | `/api/saran-publik/{id}/status` | Update status saran  |
| DELETE | `/api/saran-publik/{id}`        | Hapus saran          |

---

## 7. Halaman Frontend

### Halaman Publik

| URL      | Halaman                      |
| -------- | ---------------------------- |
| `/`      | Halaman utama / landing page |
| `/login` | Login (siswa dan admin)      |
| `/saran` | Form saran publik            |

### Halaman Siswa

| URL                       | Halaman               |
| ------------------------- | --------------------- |
| `/student/dashboard`      | Dashboard siswa       |
| `/student/complaint/new`  | Buat pengaduan baru   |
| `/student/complaints`     | Daftar pengaduan saya |
| `/student/complaints/:id` | Detail pengaduan      |

### Halaman Admin

| URL                     | Halaman                                         |
| ----------------------- | ----------------------------------------------- |
| `/admin/dashboard`      | Dashboard admin (statistik + pengaduan terbaru) |
| `/admin/complaints`     | Semua pengaduan                                 |
| `/admin/complaints/:id` | Detail pengaduan                                |
| `/admin/statistics`     | Statistik dan analisis                          |
| `/admin/review`         | Review pengaduan (admin sistem)                 |
| `/admin/review/arsip`   | Arsip pengaduan                                 |
| `/admin/tanggapi/:id`   | Buat/edit tanggapan (pelaksana)                 |
| `/admin/siswa`          | Kelola data siswa                               |
| `/admin/kategori`       | Kelola kategori                                 |
| `/admin/admin`          | Kelola admin                                    |
| `/admin/saran`          | Kelola saran publik                             |

---

## 8. Fitur Optimasi Performa

### Backend

- **SQL-level Aggregation**: Endpoint statistik menggunakan SQL `GROUP BY`, `COUNT`, `JOIN` langsung di database — tidak memproses data di PHP
- **Column Selection**: Semua endpoint list menggunakan `select()` untuk membatasi kolom yang ditransmisikan
- **Column-limited Eager Loading**: Relasi di-load hanya dengan kolom yang diperlukan (contoh: `with('siswa:nis,nama,kelas')`)
- **Ordering**: Semua list endpoint disorting di database (`orderByDesc`) bukan di frontend
- **Lightweight Endpoint**: Endpoint `/input-aspirasi/recent` hanya mengembalikan 5 data terbaru untuk dashboard

### Database

- **9 Performance Indexes**: Index pada foreign keys dan kolom filter untuk mempercepat JOIN dan WHERE

### Frontend

- **Targeted Data Fetching**: Dashboard hanya fetch 3 pengaduan terbaru — bukan seluruh data
- **No Redundant Hooks**: Halaman statistik tidak lagi memuat seluruh data pengaduan/siswa/kategori untuk fallback
- **React Query Caching**: `staleTime: 30s` mencegah re-fetch data yang masih fresh
