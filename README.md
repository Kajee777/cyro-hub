# CYRO HUB — Website + Backend Starter

Paket ini menggabungkan frontend CYRO HUB dan backend Node.js untuk akun, downloader media web berbasis yt-dlp, Spotify search, AI generation, laporan online, serta membership melalui Midtrans. Integrasi eksternal tetap memerlukan akun provider dan secret yang kamu miliki sendiri.

## Deploy online dengan Render (disiapkan)

1. Upload project ini ke repository GitHub milikmu.
2. Di Render, pilih **New → Blueprint**, lalu pilih repository tersebut. Render akan membaca `render.yaml` dan membangun image dari `Dockerfile` yang sudah memasang Node.js, yt-dlp, dan FFmpeg.
3. Pastikan disk persisten `/var/data` aktif untuk database SQLite. Jangan pilih hosting yang meniadakan proses executable atau disk jika ingin downloader dan akun tetap berfungsi.
4. Setelah deploy, isi `SPOTIFY_CLIENT_ID`, `SPOTIFY_CLIENT_SECRET`, `AI_API_KEY`, dan `MIDTRANS_SERVER_KEY` pada Environment jika memakai fitur tersebut. Jangan masukkan secret ke HTML.
5. Atur URL notifikasi Midtrans ke `https://DOMAIN-RENDER-KAMU/api/payments/midtrans`; gunakan sandbox dahulu.
6. Buka alamat HTTPS dari Render, bukan file HTML yang tersimpan di HP. Cek `/api/health` dan `/api/downloader/status`.

> Deployment tidak bisa dilakukan otomatis dari file ZIP. Kamu tetap perlu akun hosting dan repository GitHub sendiri. `render.yaml` menyiapkan konfigurasi, bukan membuat layanan publik tanpa persetujuan akunmu.

## Persiapan

1. Install Node.js 20 atau lebih baru.
2. Ekstrak folder project.
3. Salin `.env.example` menjadi `.env`.
4. Isi `SESSION_SECRET` dengan string acak panjang (minimal 32 karakter).
5. Tambahkan kredensial provider pada `.env` jika fitur tersebut mau diaktifkan.
6. Pasang **yt-dlp** dan **FFmpeg** pada mesin server untuk fitur downloader (langkah di bawah).
7. Jalankan `npm install`, lalu `npm start`.
8. Buka `http://localhost:3000`.

## Memasang downloader (wajib untuk video/audio dari URL posting)

Backend menggunakan `yt-dlp` dan `FFmpeg`; ini bukan fitur yang bisa berjalan dari file HTML saja.

- **Linux/Debian/Ubuntu:** `python3 -m pip install -U yt-dlp` lalu `sudo apt-get update && sudo apt-get install -y ffmpeg`.
- **Windows:** pasang yt-dlp dari dokumentasi resminya dan FFmpeg, lalu pastikan keduanya ada di PATH.
- Jika executable berada di lokasi khusus, isi `YTDLP_PATH` dan `FFMPEG_PATH` pada `.env`.
- Endpoint memproses satu URL per permintaan, tidak mengambil playlist, dan membatasi ukuran unduhan default 500 MB serta waktu proses 4 menit. Atur `DOWNLOAD_MAX_FILESIZE` sesuai kapasitas server.
- Pada hosting yang tidak mengizinkan executable, proses panjang, atau penyimpanan sementara, downloader tidak akan berfungsi; gunakan VPS/container yang kamu kendalikan.

## Integrasi

- **Akun:** daftar/masuk melalui menu Akun; password di-hash dengan bcrypt.
- **Spotify:** isi `SPOTIFY_CLIENT_ID` dan `SPOTIFY_CLIENT_SECRET` dari Spotify Developer Dashboard. Secret hanya ada di server. Search endpoint: `GET /api/spotify/search?q=...`.
- **AI:** isi `AI_API_KEY`; opsional ubah `AI_BASE_URL` dan `AI_MODEL`. Endpoint: `POST /api/ai/generate`. AI Personal mensyaratkan membership aktif.
- **Pembayaran:** isi `MIDTRANS_SERVER_KEY`, pilih `MIDTRANS_ENV=sandbox` dulu, lalu atur URL notifikasi/webhook Midtrans ke `https://DOMAIN_KAMU/api/payments/midtrans`. Gunakan `MIDTRANS_VVIP_PRICE_IDR` dan `MIDTRANS_AM_PRICE_IDR` untuk harga IDR. Aktivasi membership terjadi hanya setelah notifikasi pembayaran valid.
- **GenMail:** tetap memakai API mail.tm dari browser; ketersediaan dan kebijakan layanan mail.tm berlaku.
- **Web Media Downloader:** endpoint `POST /api/downloader/download` menerima `{ "url": "https://...", "format": "video|audio|best", "quality": "480|720|1080|best" }` dan memanggil yt-dlp pada server. Mendukung situs yang dikenali yt-dlp, termasuk sejumlah halaman YouTube, TikTok, Instagram, X, Facebook, dan Reddit, bergantung pada perubahan situs serta akses publik. Audio MP3 membutuhkan FFmpeg. URL privat/lokal ditolak. Konten login/private/paywall/DRM tidak dibypass; gunakan hanya konten yang kamu miliki atau berizin untuk disimpan. File langsung tetap bisa diunduh dari frontend. Status binary: `GET /api/downloader/status`.
- **Maker, Vault, Motion Pack, komik lokal, profil proyek, timer, utilities:** fitur browser/local tetap berada di frontend.
- **Report:** endpoint server tersedia untuk laporan online; UI saat ini tetap menyimpan riwayat lokal.

## Sebelum produksi — wajib

- Deploy di HTTPS.
- Ganti session MemoryStore dengan session store persisten (misalnya Redis/Postgres-backed) dan pastikan cookie secure.
- Gunakan database persisten/terkelola, backup, monitoring, rate limits, dan log yang aman.
- Atur URL webhook Midtrans, verifikasi dari dashboard provider, tes transaksi sandbox dan uji idempotensi webhook sebelum live. Handler ini adalah starter; sebelum menerima uang sungguhan, tambahkan deduplikasi event/payment attempts, validasi nominal terhadap order, dan rekonsiliasi status transaksi melalui API provider.
- Set environment secrets di dashboard hosting, jangan commit `.env` dan jangan pernah memasukkan API keys ke HTML.
- Buat kebijakan privasi/ketentuan layanan dan jelaskan penggunaan data kepada pengguna.

## API health

Buka `/api/health` untuk status konfigurasi Spotify, AI, Midtrans, dan database. Status `false` berarti kredensial terkait belum diisi.

**Penting:** project ini belum live/ter-deploy. Fitur provider yang tidak memiliki credentials akan mengembalikan pesan konfigurasi, bukan berpura-pura berhasil.
