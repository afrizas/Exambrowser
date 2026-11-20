# Daftar perubahan

## 0.2.2-beta (dari 0.1.0-beta)

Catatan: ZIP yang diterima berlabel 0.2.1, tetapi isinya (README dan Gradle) adalah 0.1.0-beta. Perubahan di bawah dibandingkan dengan isi ZIP tersebut.

### Diubah
- Aplikasi tidak lagi berupa browser. Dihapus: kolom alamat, tombol BUKA, pencarian Google, tombol Kembali/Maju/Beranda, halaman beranda dengan pilihan portal.
- Saat dibuka, aplikasi langsung memuat portal CBT (`cbt.smkn14kabupatentangerang.sch.id`).
- Nama aplikasi menjadi **CBT SMKN 14 EXAMBROWSER**, versi **0.2.2-beta** (versionCode 22). Application ID tetap `id.sch.smkn14kabupatentangerang.cbtbeta`.
- Tampilan dirancang ulang: biru navy dan biru terang, header berlogo, layout XML (sebelumnya dibangun lewat kode).
- Ikon aplikasi memakai logo SMKN 14, termasuk ikon adaptif Android 8+ (sebelumnya vektor "14" buatan sendiri).
- Tombol Back kini mundur di riwayat WebView dan keluar bila riwayat habis. Tombol muat ulang pindah ke header.
- `allowBackup` dimatikan agar cookie sesi tidak ikut tercadangkan.

### Ditambahkan
- Identitas aplikasi untuk server: token di User-Agent dan header `X-CBT-SMKN14-App` (hanya ke host portal), dapat diatur lewat `gradle.properties`. Penyebab dan cara memeriksa server ada di `PANDUAN_SERVER.md`.
- Halaman error saat internet terputus, server tidak terjangkau, atau sertifikat tidak valid, dengan tombol Coba Lagi.
- Halaman pembuka saat portal pertama dimuat.
- Dialog `alert()`/`confirm()` bermerek aplikasi, seperti pada APK lama.
- Izin penyimpanan untuk unduhan di Android 9 ke bawah.
- Gradle wrapper 8.9, workflow GitHub Actions, README, dan `PANDUAN_SERVER.md`.
- Konfigurasi release bertanda tangan (opsional, lewat GitHub Secrets).

### Diperbaiki
- Unduhan sebelumnya tidak membawa cookie sesi dan header identitas, sehingga file yang butuh login bisa gagal.
- Unduhan di Android 9 ke bawah gagal karena izin penyimpanan tidak diminta.
- Tautan dengan skema selain http/https sebelumnya dibuka lewat aplikasi lain tanpa filter. Sekarang hanya `mailto`, `tel`, `sms`, `smsto`, `geo`; skema lain (misalnya `file:`, `intent:`) ditolak.
- Perubahan mode gelap, ukuran huruf, atau mode jendela tidak lagi membuat aktivitas dibuat ulang (yang memuat ulang halaman).
- Keyboard tidak lagi menutupi kolom isian di halaman web.
- `WebView` dilepas dari layar sebelum dihancurkan.

### Dihapus
- Pencarian Google dari kolom alamat.
- Halaman "Beranda" dan pintasan Portal Siswa/Guru di dalam aplikasi; aplikasi kini memuat halaman utama portal.

### Tidak berubah (sengaja)
- Tanpa Kiosk Mode, Lock Task, PIN proktor, FLAG_SECURE, penguncian Back, screen pinning, dan anti-exit.
- Alamat server, HTTPS, dukungan JavaScript, cookies, unggah dan unduh file.
