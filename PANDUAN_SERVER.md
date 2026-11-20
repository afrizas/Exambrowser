# Penyelidikan "Akses Browser Biasa Ditutup!" dan panduan sisi server

Status: **sisi server belum diperiksa**. Dokumen ini memisahkan apa yang terbukti dari kode aplikasi dan apa yang masih perlu dicek administrator di server. Jangan menganggap masalah server sudah selesai sebelum diuji.

## Yang terbukti

1. Source di ZIP (0.1.0-beta) adalah browser biasa. Ia memuat portal dengan User-Agent bawaan WebView Chrome dan **tidak mengirim identitas aplikasi** apa pun (tidak ada User-Agent khusus, tidak ada header tambahan). Karena itu server menganggapnya "browser biasa".
2. Pesan "Akses Browser Biasa Ditutup! ... Pengguna HP Android wajib menginstal dan membuka aplikasi resmi CBT SMKN 14" tampil di dalam halaman portal. Jadi pemeriksaannya dilakukan oleh website itu sendiri (kode PHP atau JavaScript di server), bukan oleh Android atau WebView.
3. APK lama (`ujian.apk`, v3.1) mengenalkan diri dengan dua hal:
   - Header `X-CBT-SMKN14-App: SMKN14-Verified` pada setiap navigasi halaman utama ke domain sekolah.
   - Token di User-Agent: ` CBT-SMKN14-Exambro/3.0 (Android; KioskLock)`.

## Yang belum diketahui (perlu dicek di server)

- Pemeriksaan server memakai header, User-Agent, atau keduanya.
- Pola yang dicocokkan: nilai persis, bagian dari teks, atau nomor versi tertentu. Bila server mencari kata `KioskLock` atau versi `3.0`, aplikasi 0.2.2 tidak akan cocok dengan nilai bawaannya.

## Yang dikirim aplikasi 0.2.2

| Apa | Nilai bawaan | Dikirim ke |
|---|---|---|
| Akhir User-Agent | `CBT-SMKN14-Exambro/0.2.2` | semua permintaan WebView |
| Header | `X-CBT-SMKN14-App: SMKN14-Verified` | navigasi halaman utama (GET) ke host portal saja |

Nilai bawaan sengaja disamakan dengan APK lama agar kemungkinan besar dikenali server yang sama. Itu **dugaan**, bukan hasil pengujian.

## Langkah pemeriksaan di server

Jalankan di folder website CBT (sesuaikan lokasi):

```
grep -rn "Browser Biasa" .
grep -rniE "Exambro|X-CBT|HTTP_X_CBT|HTTP_USER_AGENT|HTTP_X_REQUESTED" .
```

Biasanya file yang ditemukan adalah berkas yang di-*include* di awal halaman (misalnya `config.php`, `header.php`, `auth.php`, atau `.htaccess`). Baca kondisi yang menentukan kapan pesan itu tampil.

Untuk memastikan token User-Agent sampai ke server, lihat log akses setelah membuka aplikasi:

```
grep "Exambro" /var/log/nginx/access.log | tail
# atau /var/log/apache2/access.log
```

Jika baris dengan `CBT-SMKN14-Exambro/0.2.2` muncul tetapi pesan masih tampil, server memeriksa hal lain (misalnya header, atau versi tertentu).

## Contoh logika sisi server (contoh, belum diuji di server sekolah)

Header `X-CBT-SMKN14-App` di PHP terbaca sebagai `$_SERVER['HTTP_X_CBT_SMKN14_APP']`.

```php
function permintaan_dari_aplikasi_resmi(): bool {
    $ua     = $_SERVER['HTTP_USER_AGENT'] ?? '';
    $header = $_SERVER['HTTP_X_CBT_SMKN14_APP'] ?? '';
    return stripos($ua, 'CBT-SMKN14-Exambro') !== false
        || hash_equals('SMKN14-Verified', $header);
}
```

Jika kode lama di server mencocokkan nilai yang lebih ketat (misalnya `KioskLock` atau versi `3.0`), ubah pencocokan itu agar menerima `CBT-SMKN14-Exambro`, atau ubah nilai di `gradle.properties` agar sama dengan yang diharapkan server, lalu bangun ulang APK.

## Catatan keamanan

- Token User-Agent dan header bernilai tetap itu hanya penyaring "bukan browser biasa". Siapa pun yang membongkar APK atau meniru permintaan bisa menemukannya, jadi jangan dipakai sebagai satu-satunya pengaman ujian.
- Bila ingin lebih kuat, ganti `cbt.appHeaderValue` dengan nilai rahasia khusus sekolah (acak, panjang), simpan nilai yang sama di konfigurasi server, dan ganti secara berkala. Tetap bukan jaminan, tetapi lebih sulit ditebak.
- Header hanya dipasang pada navigasi halaman utama (GET). Ia tidak ikut pada POST dan XHR/fetch, dan tidak dijamin saat tombol Back/Forward; token User-Agent ikut di semua permintaan. Karena itu pemeriksaan server yang paling stabil memakai User-Agent, dan header hanya sebagai tambahan.
- Aplikasi 0.2.2 tidak lagi mengunci ujian, jadi server tidak bisa mengandalkan aplikasi untuk mencegah perpindahan aplikasi atau screenshot. Bila itu dibutuhkan, harus dideteksi di sisi halaman web atau diminta ulang sebagai fitur.

## Cara menguji setelah perubahan

1. Pasang APK 0.2.2, buka aplikasi.
2. Portal harus tampil normal tanpa pesan "Akses Browser Biasa Ditutup".
3. Buka portal yang sama di Chrome biasa: pesan penolakan harus tetap tampil.
4. Cek log akses seperti di atas.
