# Rencana Perbaikan Tampilan (Responsive) dan Kalkulasi Saldo

Tugas ini mencakup dua bagian utama:
1. Memastikan aplikasi dapat menyesuaikan tampilan secara otomatis pada mode Portrait maupun Landscape (Responsive).
2. Memperbaiki logika kalkulasi saldo yang menyebabkan selisih (discrepancy) semakin besar seiring waktu.

## Analisis Masalah

### 1. Tampilan Portrait/Landscape
Aplikasi saat ini memiliki `manifest.json` yang mengunci orientasi ke `portrait`. Selain itu, terdapat logika simulator preview di `index.html` yang mungkin membatasi pengalaman pengguna di perangkat asli.

### 2. Selisih Saldo yang Membesar
Berdasarkan analisis kode pada `keuangan-app.js`, ditemukan masalah **Double Counting** (Penghitungan Ganda) pada fungsi `getFinancialData` dan `renderPrintBundle`.
- Sistem memuat `saldo_awal` tahun berjalan.
- Sistem kemudian menjumlahkan **seluruh** riwayat transaksi dari koleksi `jurnal` tanpa memfilter tahun.
- **Akibatnya:** Jika terdapat transaksi dari tahun-tahun sebelumnya, nilai tersebut dihitung dua kali: pertama sebagai bagian dari `saldo_awal` tahun ini, dan kedua sebagai entri jurnal individu. Hal ini menyebabkan selisih saldo terus membengkak setiap kali ada perpindahan tahun atau penambahan saldo awal.

## Perubahan yang Diusulkan

### [Config]

#### [MODIFY] [manifest.json](file:///C:/Users/Lenovo/StudioProjects/https-laporankeuanganijef.app-/manifest.json)
- Mengubah `orientation` dari `portrait` menjadi `any` agar perangkat diizinkan untuk berotasi secara otomatis.

### [UI/UX]

#### [MODIFY] [index.html](file:///C:/Users/Lenovo/StudioProjects/https-laporankeuanganijef.app-/index.html)
- Memastikan meta viewport dan CSS mendukung transisi responsive yang mulus antara portrait dan landscape.
- Menyesuaikan CSS Sidebar agar tetap mudah digunakan pada layar landscape yang memiliki tinggi (height) terbatas.

### [Core Logic]

#### [MODIFY] [keuangan-app.js](file:///C:/Users/Lenovo/StudioProjects/https-laporankeuanganijef.app-/keuangan-app.js)
- **Fungsi `getFinancialData`:** Menambahkan filter tahun pada iterasi jurnal. Jika `saldo_awal` untuk tahun tertentu tersedia, maka hanya transaksi pada tahun tersebut yang akan dijumlahkan.
- **Fungsi `renderPrintBundle`:** Memperbaiki logika pengambilan saldo awal agar sesuai dengan periode yang difilter, bukan hanya berdasarkan tahun kalender saat ini.
- **Sinkronisasi Petty Cash:** Memastikan logika Petty Cash konsisten dengan akun bank lainnya dalam hal penanganan tahun fiskal.

## Rencana Verifikasi

### Verifikasi Manual
1. **Rotasi Layar:** Membuka aplikasi di browser (simulasi mobile) dan merotasi layar untuk memastikan layout (sidebar, tabel, kartu saldo) menyesuaikan dengan benar tanpa elemen yang terpotong.
2. **Pengujian Saldo:**
   - Mencatat saldo saat ini.
   - Menambahkan transaksi di tahun sebelumnya (misal 2025) dan memastikan saldo tahun berjalan (2026) tidak ikut berubah secara salah.
   - Melakukan "Koreksi Saldo Awal" dan memverifikasi bahwa `getFinancialData` menghasilkan nilai yang tepat tanpa double counting.
3. **Print Report:** Membuka Laporan Print Bundle dan memverifikasi saldo akhir di Neraca sesuai dengan Dashboard.
