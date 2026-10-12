# Tentang Kita — Pembelian & Persediaan Bahan Baku

Aplikasi sederhana untuk mencatat bahan baku, pemasok, pembelian, dan pemakaian bahan Cafe Tentang Kita. Database menggunakan Supabase/PostgreSQL. Di Windows, server lokal dapat dijalankan memakai PowerShell yang sudah tersedia di Windows, tanpa mengunduh Node.js atau dependensi npm.

## Struktur

- `frontend/` — halaman HTML, CSS, dan JavaScript.
- `backend/app.js` — server HTTP dan API yang menghubungkan aplikasi ke Supabase.
- `backend/run.ps1` — server lokal untuk Windows; memakai konfigurasi Supabase di `backend/app.js`.
- `backend/schema.sql` — tiga tabel, ringkasan stok, dan fungsi pencatatan transaksi.
- `.github/workflows/deploy-pages.yml` — menerbitkan frontend ke GitHub Pages saat ada push ke `main` atau `master`.
- `Jalankan Tentang Kita.bat` — cara praktis untuk menjalankan server dan membuka halaman di browser.

## Navigasi aplikasi

Sidebar memisahkan aplikasi menjadi **Ringkasan**, **Daftar persediaan**, **Data pemasok**, dan **Transaksi**. Pada layar ponsel, menu sidebar dapat dibuka melalui tombol menu di header.

Halaman transaksi menampilkan statistik cafe, formulir pembelian/pemakaian, dan ringkasan nilai transaksi yang diperbarui langsung saat formulir diisi.

## Menyiapkan Supabase

1. Buat project Supabase, buka **SQL Editor**, lalu jalankan isi `backend/schema.sql`.
2. Untuk membuka `frontend/index.html` langsung, isi `SUPABASE_PUBLIC_KEY` di `frontend/app.js` dengan **anon** atau **publishable** key dari Supabase → Project Settings → API Keys. Jangan pernah menaruh service_role/Secret key di frontend.
3. Di Supabase SQL Editor, jalankan isi `backend/izin-file-html.sql` untuk memberikan hak akses tabel dan RLS yang dibutuhkan saat file dibuka langsung. Jalankan berkas ini juga bila skema database sudah pernah dipasang sebelumnya.

4. Buka `frontend/index.html` dengan browser. Mode file mengakses Supabase langsung. Jika masih ada tab aplikasi terbuka, muat ulang tab setelah SQL selesai dijalankan.

   Alternatifnya, jika ingin menggunakan mode backend:

   ```powershell
   .\backend\run.ps1
   ```

   Lalu buka `http://127.0.0.1:3000`. Saat pertama berjalan, masukkan service-role/Secret key pada prompt tersembunyi atau atur environment variable `SUPABASE_SERVICE_ROLE_KEY`. Jangan commit atau membagikan key tersebut.

> Key anon/publishable memang terlihat oleh browser. Izin file-HTML mengizinkan publik membaca data cafe dan menambahkan bahan/pemasok serta transaksi lewat fungsi yang memvalidasi stok. Gunakan mode ini hanya jika data cafe boleh diakses publik; untuk data privat, jalankan aplikasi melalui backend atau tambahkan autentikasi Supabase.

## Menerbitkan ke GitHub Pages

1. Push folder proyek ke branch `main` atau `master`.
2. Di repository GitHub buka **Settings → Pages**, lalu atur **Build and deployment → Source** ke **GitHub Actions**.
3. Buka tab **Actions** dan tunggu workflow **Deploy cafe app to GitHub Pages** selesai.
4. Buka URL Pages yang ditampilkan pada workflow (biasanya `https://USERNAME.github.io/NAMA-REPOSITORY/`). Jangan buka tampilan kode `github.com/.../blob/...` karena GitHub menampilkan file HTML sebagai teks pada halaman tersebut.

Workflow hanya menerbitkan isi folder `frontend`, bukan backend. Pada GitHub Pages aplikasi memakai publishable/anon key dan kebijakan RLS Supabase; backend server tidak dijalankan oleh Pages.

**Penting:** backend hanya membaca `SUPABASE_SERVICE_ROLE_KEY` dari environment atau prompt tersembunyi. Jangan simpan server key di file yang di-commit. Jika service-role key lama pernah di-push ke GitHub, segera rotate/revoke key itu di Supabase; menghapusnya dari versi terbaru tidak menghapusnya dari riwayat Git.

## ERD (3 entitas)

```text
bahan_baku (1) ─── (N) transaksi_persediaan (N) ─── (0..1) pemasok
```

- `bahan_baku`: nama, satuan, stok minimum.
- `pemasok`: nama dan kontak.
- `transaksi_persediaan`: satu baris per pembelian atau pemakaian; menyimpan jumlah dan biaya per satuan.
- Stok dan nilai persediaan dihitung dari seluruh mutasi. Biaya pemakaian mengikuti rata-rata tertimbang berjalan dan stok negatif ditolak secara atomik oleh fungsi database.

Backend hanya membuka akses API melalui server lokal dan menggunakan key server di sisi backend. Jangan bagikan URL aplikasi ini ke jaringan publik tanpa menambahkan autentikasi dan pengamanan deployment.
