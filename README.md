# MASAKU — React + Firebase + PWA + Pengingat Otomatis

MASAKU membantu pengguna mencatat produk dan memantau tanggal kedaluwarsa. Setiap akun hanya dapat mengakses produk miliknya sendiri.

## Fitur
- Daftar dan masuk menggunakan Firebase Authentication (email/password).
- Tambah, edit, hapus, cari, dan filter produk.
- Dashboard jumlah produk aman, mendekati kedaluwarsa, dan kedaluwarsa.
- PWA yang dapat dipasang pada perangkat yang mendukung.
- Push notification melalui Firebase Cloud Messaging (FCM).
- Pengingat otomatis pada H-7, H-3, H-1, dan hari-H.
- Penjadwalan harian melalui Vercel Cron (pukul sekitar 09.00 WIB; waktu cron Vercel berbasis UTC dan pada paket Hobby dapat bergeser dalam jam tersebut).

## 1. Persiapan Firebase
1. Buat project di https://console.firebase.google.com/.
2. Tambahkan aplikasi Web dan salin konfigurasi Firebase.
3. Authentication → Sign-in method → aktifkan Email/Password.
4. Firestore Database → buat database.
5. Project settings → Cloud Messaging → Web Push certificates → Generate key pair. Salin public VAPID key.
6. Pastikan Firebase Cloud Messaging API / FCM Registration API aktif pada Google Cloud project.
7. Project settings → Service accounts → Generate new private key. Simpan file JSON dengan aman; jangan masukkan ke GitHub.
8. Terapkan isi `firestore.rules` ke Firestore Rules.

## 2. Jalankan lokal
1. Ekstrak ZIP lalu buka folder `MASAKU-React-PWA` di VS Code.
2. Salin `.env.example` menjadi `.env`.
3. Isi semua variabel `VITE_FIREBASE_*` dari Firebase Web App dan VAPID public key.
4. Isi variabel server `FIREBASE_PROJECT_ID`, `FIREBASE_CLIENT_EMAIL`, `FIREBASE_PRIVATE_KEY`, dan `CRON_SECRET` dari service account / secret acak.
5. Buka terminal:
   ```bash
   npm install
   npm run dev
   ```
6. Untuk uji build: `npm run build`.

## 3. Deploy ke GitHub dan Vercel
1. Buat repository GitHub baru dan push semua file project (jangan upload `.env`).
2. Import repository tersebut ke Vercel. Framework preset: Vite; build command `npm run build`; output directory `dist`.
3. Tambahkan semua variabel dari `.env` di Vercel → Project → Settings → Environment Variables. Jangan beri awalan `VITE_` pada tiga kredensial server.
4. Deploy ulang setelah menambah environment variables.
5. Vercel membaca `vercel.json` untuk menjalankan `/api/send-reminders` setiap hari pukul 02:00 UTC (sekitar 09:00 WIB). Cron request diamankan dengan `CRON_SECRET`.

## 4. Mengaktifkan notifikasi pada HP
1. Buka URL HTTPS Vercel di browser yang mendukung Web Push (misalnya Chrome Android).
2. Login, tekan tombol **Aktifkan** pada bagian Pengingat Otomatis, lalu izinkan notifikasi.
3. Untuk PWA, gunakan menu browser **Tambahkan ke layar utama / Install app**.
4. Setiap perangkat perlu mengaktifkan notifikasi sendiri. Jika izin ditolak, ubah izin situs melalui pengaturan browser.

## 5. Cara kerja pengingat
- Backend memeriksa tanggal kedaluwarsa setiap hari berdasarkan tanggal Asia/Jakarta.
- Pengingat dikirim saat sisa hari tepat 7, 3, 1, atau 0.
- Log pengiriman disimpan per pengguna agar pengingat untuk tanggal yang sama tidak dikirim berulang.
- Produk tanpa perangkat yang mengaktifkan notifikasi tetap muncul di dashboard, tetapi tidak menerima push notification.
- Jika token perangkat tidak valid, token tersebut dibersihkan dari Firestore.

## 6. Struktur data Firestore
```text
users/{uid} (profil akun)
users/{uid}/products/{productId}
users/{uid}/devices/{hashedToken}
users/{uid}/reminderLogs/{productId_expiryDate_daysLeft}
```

## Catatan penting
- Push notification membutuhkan HTTPS, izin notifikasi, browser/perangkat yang mendukung Web Push, dan koneksi internet.
- Notifikasi otomatis terjadwal memerlukan Vercel deployment + environment variables server + Cron aktif. `npm run dev` saja tidak menjalankan penjadwalan harian.
- Vercel Cron pada paket Hobby berjalan sekali sehari dan waktunya dapat bergeser dalam jam jadwal. Untuk pengingat harian ini sudah memadai; tidak menjamin pengiriman tepat pada menit tertentu.
- PWA shell dapat dibuka dari cache, tetapi sinkronisasi Firestore dan pengiriman notifikasi membutuhkan koneksi internet.
- Jangan pernah commit `.env`, service account JSON, private key, atau CRON_SECRET ke GitHub.
