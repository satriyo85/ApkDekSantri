# SIM RS Pembelajaran

Aplikasi laboratorium pembelajaran **Sistem Informasi Manajemen Rumah Sakit (SIM RS)** dengan modul **Rekam Medis Elektronik (RME)**. Dibuat dengan **Laravel 12, PHP 8.2+, MySQL, dan Bootstrap 5.3**; semua berkas Bootstrap dan Bootstrap Icons dibundel lokal sehingga aplikasi tidak meminta CDN ketika dijalankan.

> **Khusus pendidikan dan data sintetis.** Aplikasi ini belum tervalidasi sebagai sistem klinis/rumah sakit dan tidak boleh digunakan untuk merawat pasien, membuat keputusan klinis, atau menyimpan data pasien nyata. Sebelum penggunaan selain latihan, institusi perlu melakukan penilaian keamanan, privasi, tata kelola, uji klinis/operasional, dan validasi regulasi yang berlaku.

## Modul yang tersedia

- Login, logout, pembatasan percobaan login, akun aktif/nonaktif, dan role.
- Dashboard operasional: pasien aktif, kunjungan, encounter terbuka, serta RME ditandatangani.
- Registrasi dan pencarian pasien; nomor RM demo dihasilkan otomatis; data dapat di-soft-delete melalui pengembangan lanjutan.
- Registrasi encounter/kunjungan dan antrean dengan pencarian serta filter status.
- RME format SOAP: subjektif, objektif, asesmen, rencana, serta teks/kode diagnosis untuk latihan.
- Tanda vital: tekanan darah, suhu, nadi, respirasi, SpO₂, berat, dan tinggi dengan validasi rentang.
- Item terapi/resep ilustratif; tidak ada katalog atau rekomendasi obat klinis.
- Dokter dapat menandatangani encounter simulasi; catatan yang ditandatangani dikunci oleh alur aplikasi dan nama penandatangan dicatat terpisah dari penulis.
- Pengelolaan akun role oleh admin dan audit trail untuk perubahan penting. Audit tidak mencatat isi SOAP.
- Antarmuka Bootstrap responsif untuk desktop, tablet, dan ponsel.

## Hak akses

| Peran | Akses |
| --- | --- |
| Administrator | Dashboard, direktori pasien, registrasi encounter, pengelolaan akun, audit |
| Petugas rekam medis | Dashboard, registrasi dan pencarian pasien, registrasi/daftar encounter |
| Dokter | Dashboard, membuka encounter, menulis SOAP/vital, menambahkan item terapi simulasi, menandatangani RME |
| Perawat | Dashboard, membuka encounter, menulis SOAP/vital; tidak menandatangani atau menambah item terapi |

Hak akses ini merupakan baseline edukasi. Ruang lingkup akses yang tepat perlu ditetapkan lagi oleh pengelola server kampus. Implementasi ini belum mengatur organisasi/unit/fasilitas secara granular.

## Persyaratan server

- PHP 8.2 atau lebih baru (Laravel 12); ekstensi Laravel umum termasuk `pdo_mysql`, `mbstring`, `openssl`, `fileinfo`, `xml`, `curl`, dan `ctype`.
- Composer 2.
- MySQL 8 atau MariaDB 10.5+.
- Apache dengan `mod_rewrite` atau Nginx; **document root wajib menunjuk ke folder `public/`**.
- HTTPS dan backup terenkripsi pada deployment kampus.

## Pengembangan lokal

1. Ekstrak proyek dan dari folder root jalankan:

   ```bash
   composer install
   cp .env.example .env
   php artisan key:generate
   ```

2. Buat database dan pengguna MySQL khusus aplikasi, lalu isi `DB_HOST`, `DB_PORT`, `DB_DATABASE`, `DB_USERNAME`, dan `DB_PASSWORD` di `.env`. Berikan hak minimum atas satu database aplikasi saja.
3. Jalankan migration:

   ```bash
   php artisan migrate
   ```

4. **Hanya untuk mesin lab terisolasi**, tambahkan data fiktif dan akun demo:

   ```bash
   php artisan db:seed --class=DemoSeeder
   ```

   Seeder ini membuat 18 pasien dan 30 encounter sintetis, serta akun:

   | Peran | Email | Kata sandi demo |
   | --- | --- | --- |
   | Admin | `admin@simrs.local` | `DemoAdmin!2026` |
   | Dokter | `dokter@simrs.local` | `DemoDokter!2026` |
   | Perawat | `perawat@simrs.local` | `DemoPerawat!2026` |
   | Rekam medis | `rm@simrs.local` | `DemoRM!2026` |

   **Jangan jalankan seeder demo pada server terbuka/produksi.** Kredensial tersebut publik dan hanya untuk evaluasi lokal.
5. Jalankan web server development:

   ```bash
   php artisan serve --host=127.0.0.1
   ```

   Buka `http://127.0.0.1:8000`. Untuk login demo, gunakan akun admin di tabel di atas.

`DatabaseSeeder` standar sengaja tidak membuat akun maupun data; ia aman dipanggil saat instalasi produksi. Akun administrator awal dibuat memakai perintah interaktif:

```bash
php artisan simrs:create-admin
```

Perintah tersebut meminta nama, email, serta kata sandi minimal 12 karakter dan memeriksa agar email belum digunakan.

Pengguna yang sudah masuk dapat mengganti kata sandi lewat menu identitas akun pada bar atas, **Akun Saya**.

## Deployment server kampus

1. Buat database dan user MySQL khusus aplikasi dengan hak hanya pada database SIM RS.
2. Unggah source release ke server; atur virtual host agar document root mengarah ke `<folder-proyek>/public` (misalnya `simrs-laravel12/public` setelah ekstraksi), bukan root proyek. Folder `.env`, `storage`, `vendor`, dan source aplikasi tidak boleh dapat diakses sebagai dokumen publik.
3. Instal dependency produksi (bila release tidak menyertakan folder `vendor`):

   ```bash
   composer install --no-dev --optimize-autoloader --no-interaction
   ```

4. Siapkan konfigurasi produksi:

   ```bash
   cp .env.example .env
   ```

   Set `APP_ENV=production`, `APP_DEBUG=false`, `APP_URL` ke alamat HTTPS resmi kampus, rincian database, dan pastikan `SESSION_SECURE_COOKIE=true`. Lindungi file `.env` agar hanya user proses aplikasi dan administrator berwenang yang dapat membacanya.
5. Jalankan setup sekali:

   ```bash
   php artisan key:generate
   php artisan migrate --force
   php artisan simrs:create-admin
   php artisan optimize
   ```

6. Pastikan user PHP-FPM dapat menulis ke `storage/` dan `bootstrap/cache/`; jangan memberi hak tulis pada source kode yang tidak perlu. Atur HTTPS/TLS di reverse proxy/web server kampus.
7. Jalankan backup basis data berkala, terenkripsi, terbatas aksesnya, dan uji restore. Tetapkan prosedur patch security Laravel/PHP, retensi, respons insiden, serta pengelolaan kredensial.

Untuk memperbarui source, lakukan backup terlebih dahulu, pasang dependency versi lock file, tinjau migration, kemudian jalankan `php artisan migrate --force`. Jangan menjalankan `migrate:fresh` pada data kampus.

## Pemeriksaan yang disarankan

```bash
php artisan about
php artisan route:list
php artisan migrate:status
php artisan route:cache
php artisan view:cache
```

Bersihkan cache setelah perubahan konfigurasi bila diperlukan: `php artisan optimize:clear`.

## Struktur basis data

- `users`: akun dan role
- `patients`: demografi minimum, alergi, kontak darurat, soft delete
- `encounters`: registrasi kunjungan dan status
- `medical_records`: SOAP, diagnosis, penulis, penandatangan, waktu tanda tangan
- `vital_signs`: pengukuran tanda vital per encounter
- `prescriptions`: item terapi simulasi
- `audit_logs`: aktor, aksi, objek, waktu, IP, dan user-agent

## Catatan keamanan dan privasi

Aplikasi memakai autentikasi bawaan Laravel, hash kata sandi, CSRF untuk formulir, validasi server, rate limiter login, middleware role, dan audit operasi penting. Ini **bukan** jaminan siap produksi: belum ada uji penetrasi, disaster recovery exercise, persetujuan privasi, autentikasi multi-faktor, tanda tangan elektronik tersertifikasi, integrasi standar interoperabilitas klinis, atau kontrol akses per unit/fasilitas. Gunakan hanya data fiktif sampai seluruh aspek tersebut ditinjau dan disetujui institusi.

## Aset frontend dan lisensi

- Bootstrap 5.3.8 — MIT, salinan lisensi ada di `public/vendor/bootstrap/LICENSE`.
- Bootstrap Icons 1.13.1 — MIT, lisensi ada di `public/vendor/bootstrap-icons/LICENSE`.
- Laravel framework — MIT; lihat dokumentasi resmi Laravel.
