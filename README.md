# CBT SMKN 14 EXAMBROWSER — 0.2.2 BETA

Aplikasi Android resmi SMK Negeri 14 Kabupaten Tangerang. Saat dibuka, aplikasi langsung menampilkan portal CBT sekolah di dalam WebView. Ini **bukan** browser umum: tidak ada address bar, tombol maju/mundur browser, menu browser, atau riwayat.

- **Application ID:** `id.sch.smkn14kabupatentangerang.cbtbeta` (sama dengan versi 0.1.0-beta)
- **Versi:** 0.2.2-beta (versionCode 22)
- **Android:** minSdk 23 (Android 6.0), compileSdk 35, targetSdk 34

## Status (baca dulu)

| Hal | Status |
|---|---|
| Source project Android Studio | Ada, lengkap |
| Pemeriksaan Kotlin | Dikompilasi dengan kompiler Kotlin 2.0.21 terhadap *stub* API Android buatan sendiri: tanpa error. Ini **bukan** build sungguhan |
| Build Gradle / APK | **Belum dijalankan.** Lingkungan penyusun tidak punya Android SDK dan akses internet, jadi APK belum ada. Build dilakukan lewat GitHub Actions (lihat di bawah) |
| Uji di HP | **Belum diuji** di perangkat mana pun |
| Server / backend PHP | **Belum diperiksa.** Lihat `PANDUAN_SERVER.md` |

## Fitur

- Langsung membuka portal CBT (`https://cbt.smkn14kabupatentangerang.sch.id/`) tanpa address bar.
- Header aplikasi berlogo SMKN 14 dengan tombol muat ulang; indikator loading di bawah header.
- Halaman pembuka saat portal pertama kali dimuat.
- Halaman error bila internet terputus atau server tidak terjangkau, dengan tombol **Coba Lagi**.
- JavaScript, DOM storage, cookies, dan sesi login aktif.
- Unggah file dari halaman web (pemilih file Android) dan unduh file (DownloadManager, membawa sesi login).
- Dialog `alert()` dan `confirm()` dari halaman web ditampilkan dengan judul aplikasi.
- Tombol Back Android normal: mundur di riwayat WebView; bila sudah habis, aplikasi keluar seperti biasa.
- Layar bisa diputar; keyboard tidak menutupi kolom isian (`adjustResize`).
- Koneksi tetap HTTPS. Sertifikat bermasalah ditolak, mixed content dan HTTP cleartext tidak diaktifkan.

## Yang sengaja tidak ada

Kiosk Mode, Lock Task, PIN proktor, `FLAG_SECURE` (screenshot boleh), penguncian tombol Back, screen pinning, pembatasan keluar aplikasi, dan sistem anti-exit. Fitur-fitur itu tidak ditambahkan kembali tanpa permintaan.

## Identitas aplikasi untuk server

Server sekolah menampilkan "Akses Browser Biasa Ditutup!" bila permintaan tidak dikenali sebagai aplikasi resmi. Aplikasi ini mengirim:

1. Token di akhir User-Agent pada semua permintaan: `CBT-SMKN14-Exambro/0.2.2`
2. Header `X-CBT-SMKN14-App: SMKN14-Verified` pada navigasi halaman utama (GET) **hanya ke host portal**, tidak ke situs lain.

Nilai bawaan itu sama dengan yang dipakai APK lama (`ujian.apk` v3.1). Apakah server menerimanya **belum terverifikasi**. Cara memeriksa dan mengubah sisi server ada di `PANDUAN_SERVER.md`. Nilainya bisa diubah di `gradle.properties` tanpa menyentuh kode Kotlin.

## Mengubah konfigurasi

Buka `gradle.properties`, bagian bawah:

```
cbt.portalUrl=https://cbt.smkn14kabupatentangerang.sch.id/
cbt.appHeaderName=X-CBT-SMKN14-App
cbt.appHeaderValue=SMKN14-Verified
cbt.appUaToken=CBT-SMKN14-Exambro/0.2.2
```

Kosongkan `cbt.appHeaderName` bila server tidak memakai header. Kosongkan `cbt.appUaToken` bila tidak memakai token UA. Jangan memakai tanda kutip (`"`) atau garis miring terbalik (`\`) pada nilai.

## Membangun APK dari HP (GitHub Actions)

Cara paling mudah tanpa komputer, memakai satu file ZIP:

1. Di repo GitHub kamu, pilih **Add file → Upload files**, lalu unggah file ZIP proyek ini (satu file). Hapus ZIP atau source lama bila ada.
2. Pilih **Add file → Create new file**. Pada kolom nama ketik `.github/workflows/build-apk.yml` (garis miring otomatis membuat folder). Tempel isi file `build-apk.yml` (ada di folder utama ZIP, dan dikirim terpisah di chat), lalu **Commit changes**.
3. Buka tab **Actions → Build APK → Run workflow**.
4. Tunggu hingga hijau (beberapa menit). Buka run tersebut, lalu di bagian **Artifacts** unduh `CBT-SMKN14-Exambrowser-APK`. Hasilnya berupa ZIP; ekstrak di aplikasi File, lalu pasang `...-debug.apk`.

Workflow juga bisa membangun dari source yang diunggah apa adanya (tanpa ZIP) di root repo atau subfolder.

Catatan pemasangan:
- APK debug memakai kunci tanda tangan yang dibuat ulang pada setiap build GitHub. Jika pemasangan di atas versi lama gagal ("Aplikasi tidak terpasang"), hapus (uninstall) aplikasi lama lebih dulu. Untuk kunci tetap, lihat bagian APK release.
- Izinkan "Pasang dari sumber ini" untuk aplikasi File/Chrome bila diminta.

## APK release (opsional)

APK release hanya dibuat bila keystore tersedia. Simpan di **Settings → Secrets and variables → Actions** pada repo:

- `KEYSTORE_BASE64` (isi file keystore yang dikodekan base64)
- `KEYSTORE_PASSWORD`, `KEY_ALIAS`, `KEY_PASSWORD`

Bila `KEYSTORE_BASE64` ada, workflow otomatis ikut membuat `...-release.apk` yang bertanda tangan tetap. Tanpa itu, hanya APK debug yang dibuat. Keystore tidak termasuk dalam proyek ini dan tidak boleh diunggah ke repo.

## Membuka di Android Studio

1. Ekstrak ZIP, lalu **File → Open** dan pilih folder `CBT_SMKN14_Exambrowser`.
2. Pastikan JDK 17 dan Android SDK Platform 35 terpasang. Gradle wrapper (8.9) sudah disertakan.
3. Tunggu sinkronisasi Gradle, lalu **Run** ke perangkat atau **Build → Build APK(s)**.
4. Alternatif perintah: `./gradlew assembleDebug`. Hasil: `app/build/outputs/apk/debug/app-debug.apk`.

Versi: AGP 8.7.3, Kotlin 2.0.21, Gradle 8.9.

## Mengganti logo

Logo ada di `app/src/main/res/drawable-nodpi/logo_smkn14.png` (header dan halaman pembuka), `app/src/main/res/drawable-nodpi/ic_launcher_foreground.png` (ikon adaptif Android 8+), dan `app/src/main/res/mipmap-*/ic_launcher*.png` (ikon Android 7 ke bawah).

Logo yang terpasang diambil dari APK lama (`ujian.apk`), resolusi maksimal 192×192 piksel, karena file logo resmi tidak terlampir. Bila punya logo beresolusi lebih tinggi, ganti file-file di atas dengan nama yang sama.

## Struktur proyek

```
app/build.gradle.kts                 versi, konfigurasi build, BuildConfig
gradle.properties                    konfigurasi admin (alamat portal, identitas aplikasi)
app/src/main/AndroidManifest.xml     deklarasi aplikasi
app/src/main/java/.../MainActivity.kt  logika WebView
app/src/main/res/layout/activity_main.xml  tampilan (header, WebView, halaman pembuka, halaman error)
app/src/main/res/values/             warna, teks, tema
.github/workflows/build-apk.yml      build APK otomatis
PANDUAN_SERVER.md                    penyelidikan "Akses Browser Biasa Ditutup" dan panduan sisi server
CHANGELOG.md                         daftar perubahan
```

## Keterbatasan yang diketahui

- targetSdk 34. Menaikkan ke 35 membuat Android 15 menerapkan tampilan edge-to-edge, sehingga header dan konten perlu penanganan *window insets*.
- Setelah proses aplikasi dimatikan sistem, aplikasi membuka portal dari awal. Sesi login tetap tersimpan lewat cookie.
- Tautan ke situs lain tetap dibuka di dalam aplikasi (tidak ada daftar domain yang diizinkan, sesuai versi sebelumnya).
- Unduhan `blob:` dan `data:` belum didukung.
