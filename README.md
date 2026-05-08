# BidMart - Group A10

## 1. Current Architecture - Context, Container, and Deployment Diagram

### System Context Diagram

System Context Diagram ini memberikan gambaran ekosistem platform BidMart sebagai
sistem marketplace dan lelang real-time.

Diagram ini sengaja dibuat high-level. Aktor pengguna berinteraksi melalui
interface BidMart, administrator berinteraksi melalui admin interface, dan sistem
bergantung pada layanan eksternal untuk pengiriman email verifikasi, reset
password, dan MFA.

Diagram konteks tidak menampilkan database karena detail kepemilikan data berada
pada container dan deployment diagram.

![Current Context Diagram](docs/architecture/current-context.png)

**Aktor dan sistem eksternal yang terlibat:**

- **Buyer/User**: Pengguna yang melakukan autentikasi, mengakses marketplace,
  mengikuti lelang, membuat bid, dan mengelola transaksi.
- **Seller/User**: Pengguna yang melakukan autentikasi, membuat listing,
  mengelola barang, dan memproses pesanan.
- **Admin**: Pengguna internal yang mengakses admin interface untuk moderasi,
  laporan, dan operasi platform.
- **Email Provider**: Sistem eksternal yang digunakan oleh auth service untuk
  email verification, password reset, dan MFA email.

---

### Container Diagram

BidMart menggunakan arsitektur multi-container yang memisahkan frontend, backend,
dan kepemilikan database berdasarkan batas layanan.

Relasi antarkomponen dibuat eksplisit untuk mencegah coupling yang salah:

- `auth-fe` hanya memanggil `auth-be`.
- `core-fe` hanya memanggil `core-be`.
- `admin-fe` hanya memanggil `admin-be`.

![Current Container Diagram](docs/architecture/current-container.png)

**Container yang ada:**

- **auth-fe**: Frontend untuk autentikasi dan account/settings UI. Container ini
  hanya memanggil `auth-be`.
- **core-fe**: Frontend untuk marketplace, listing, auction, bid, order, wallet,
  dan fitur core lain. Container ini hanya memanggil `core-be`.
- **admin-fe**: Frontend untuk admin/moderation UI. Container ini hanya memanggil
  `admin-be`.
- **auth-be**: Backend autentikasi. Service ini memiliki `auth-db` dan menangani
  users, credentials, sessions, verification tokens, password reset tokens, MFA
  data, dan security settings.
- **core-be**: Backend marketplace. Service ini memiliki `core-db` dan menangani
  listings, auctions, bids, orders, wallets, notifications, dan marketplace data.
- **admin-be**: Backend admin. Service ini tidak memiliki akses database langsung.
  Untuk user/account data, `admin-be` memanggil `auth-be`. Untuk listing,
  auction, order, report, atau moderation data, `admin-be` memanggil `core-be`.
- **auth-db**: Database milik `auth-be`.
- **core-db**: Database milik `core-be`.
- **Email Provider**: Layanan eksternal untuk email verification, reset password,
  dan MFA email.

**Aturan kepemilikan data:**

- `auth-be -> auth-db` adalah satu-satunya akses langsung ke `auth-db`.
- `core-be -> core-db` adalah satu-satunya akses langsung ke `core-db`.
- `admin-be` tidak boleh mengakses `auth-db` atau `core-db` secara langsung.
- Tidak ada akses silang seperti `auth-be -> core-db` atau `core-be -> auth-db`.

---

### Deployment Diagram

Deployment Diagram menunjukkan bagaimana container dijalankan pada runtime
environment dan tetap mempertahankan aturan dependency yang sama dengan container
diagram.

Browser pengguna mengakses auth/core frontend, browser admin mengakses admin
frontend, dan setiap frontend memanggil backend pasangannya. `admin-be`
melakukan orchestration melalui API `auth-be` dan `core-be`, bukan melalui
database.

![Current Deployment Diagram](docs/architecture/current-deployment.png)

**Deployment utama:**

- **User Device**: Menjalankan user browser untuk auth/settings dan marketplace
  UI.
- **Admin Device**: Menjalankan admin browser untuk admin UI.
- **Frontend Hosting / Container Host**: Menjalankan `auth-fe`, `core-fe`, dan
  `admin-fe`.
- **Backend Runtime / Container Host**: Menjalankan `auth-be`, `core-be`, dan
  `admin-be`.
- **Database Runtime**: Menyediakan dua database, yaitu `auth-db` dan `core-db`.
- **GitHub Actions + GHCR**: CI/CD, Docker build, image scan, dan image registry.
- **Email Provider**: Dipanggil oleh `auth-be` untuk kebutuhan auth/security
  email.
