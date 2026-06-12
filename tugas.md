Anda diminta untuk membuat sebuah aplikasi website sederhana untuk pemesanan Bahan Ajar di Universitas Terbuka. Pada Tugas Praktik Kedua ini, mahasiswa berfokus untuk membuat aplikasi website berbasis Vue.js sebagai framework yang digunakan sepanjang pengerjaan Tugas Praktik 3 Sehingga, data-data yang dibutuhkan tidak dibaca melalui database, tapi melalui json file sebagai dummy data yang akan dilampirkan bersamaan dengan Tugas Praktik ini.

Pada Tugas Praktikum 2, kita sudah membuat beberapa fitur pada sistem Bahan Ajar Universitas Terbuka menggunakan Vue.js yang masih sederhana. Sekarang, tugas teman-teman adalah merefaktor halaman web yang sudah dibuat menjadi custom element dalam bentuk Vue Component dan Template. Namun, tentunya ada beberapa hal yang perlu ditambahkan untuk mencapai indikator untuk penilaian hasil belajar teman-teman.

Catatan:
- Nama file untuk nama komponen menggunakan gaya kebab-case untuk naming convention style yang digunakan dalam tugas ini.
- Contoh: File component stock-table.js diregistrasikan menjadi Vue Component dengan perintah sebagai berikut: Vue.component('ba-stock-table', ...)
- Setelah itu, id template juga perlu konsisten. Sebagai contoh: tpl-stock, tpl-tracking, dsb., agar mudah dicari.
- Routing ke halaman lain cukup tab state di root (index.html)
- Misal: tab: 'stok' | 'tracking' | 'order'

1. Halaman Stok Bahan Ajar
    - Halaman ini menampilkan daftar stok bahan ajar untuk seluruh UT daerah (silakan bebas menggunakan table/list/bentuk lain yang menampilkan list data dalam jumlah banyak dan memiliki data-data field). Teman-teman mahasiswa perlu menampilkan daftar stok bahan ajar yang terdiri dari kolom data field sebagai berikut: 
        - Kode Mata Kuliah/Nama Mata Kuliah → kode, judul
        - Kategori Mata Kuliah → Kategori
        - UT-Daerah → upbjj
        - Lokasi Rak → lokasiRak
        - Harga → harga
            - Buatlah formatting teks harga dengan menambahkan currency (Rp)
        - Jumlah Stok Bahan Ajar → qty
            - Buatlah formatting teks Jumlah Stok Bahan Ajar dengan menambahkan satuan (buah)
        - Jumlah Stok Safety Bahan Ajar → safety
            - Buatlah formatting teks Jumlah Stok Bahan Ajar dengan menambahkan satuan (buah)
        - Status
        
        Untuk data field Catatan → catatanHTML, ditampilkan saat mentrigger
        event mouse hover pada kolom ‘Status’dan menampilkan tooltip/preview
        
    - Terdapat Fitur untuk mengedit stok data bahan ajar
    - Terdapat fitur untuk melakukan filter dan sort data stock (silakan bebas menggunakan elemen select/radio/checkbox)
        - Terdapat filter stok berdasarkan:
            - ‘UT-Daerah’: terdiri pilihan dari seluruh UT-daerah, dan daftar setiap UT-daerah (upbjjList)
            - ‘Kategori Mata Kuliah’ (upbjjList)
        - Terdapat filter yang hanya menampilkan jumlah stock (qty) < dari jumlah safety stock (safety) dan jumlah stock = 0 untuk menjadi daftar yang dapat mengingatkan untuk melakukan re-order bahan ajar.
        - Terdapat fitur sort yang mengurutkan berdasarkan judul, stock, dan harga
        - Fitur filter dapat di-set ulang (reset)
        - Terapkan fitur di vue.js di mana di semua filter tidak perlu recompute kembali.
        - Implementasikan dependent options seperti misalkan ketika memilih filter stok berdasarkan ‘UT-daerah’, baru akan memunculkan filter untuk ‘Kategori Mata Kuliah’
    - Menampilkan status dari stok bahan ajar
        - Memberikan status ‘Aman dengan menggunakan simbol yang memberikan tanda yang memberikan kesan ‘aman’ ataupun teks berwarna hijau apabila stok >= safety stock
        - Memberikan status ‘Menipis’ dengan menggunakan simbol yang memberikan kesan adanya ‘warning’ ataupun teks berwarna orange apabila stok < safety stock.
        - Memberikan status ‘Kosong’ dengan menggunakan simbol yang memberikan kesan adanya ‘bahaya’ ataupun teks berwarna merah apabila stok = 0
    - Terdapat Fitur untuk menambahkan data bahan ajar baru (create) melalui formulir kecil dan melakukan validasi sederhana untuk setiap data field.
        - Ketika mentrigger event keyboard ‘Enter’, maka data bahan ajar baru langsung
        disimpan dan diperbaharui pada tampilan daftar
    - Terdapat Fitur untuk memperbaharui (update) data bahan ajar baru melalui formulir
    kecil dan melakukan validasi sederhana untuk setiap data field.
        - Ketika mentrigger event keyboard ‘Enter’, maka data bahan ajar akan
        langsung ter-update dengan nilai terbaru
    - Terdapat Fitur untuk menghapus data bahan ajar melalui event click tombol/icon “delete”
        - Sebelum menghapus data bahan ajar, terdapat pop up yang mengkonfirmasi penghapusan data kepada pengguna
2. Tracking Delivery Order (DO)
    - Halaman ini menampilkan halaman untuk tracking status pengiriman DO bahan ajar seperti pada Tugas Praktik 1, tetapi di sini perlu menerapkan penggunaan framework Vue.js.
        - Menampilkan fitur pencarian berdasarkan:
            - Nomor DO, atau
            - NIM
        - Pengguna dapat melakukan submit pencarian dengan cara menekan tombol keyboard ‘Enter’
        - Pengguna dapat melakukan clear/reset pencarian dengan cara menekan tombol keyboard ‘Esc’
    - Terdapat fitur untuk menambahkan input delivery order yang baru. Berikut adalah data field yang perlu diisi.
        - Nomor DO, di mana tergenerate otomatis. Penomorannya terdiri dari:
            - DO
            - Tahun Berjalan
            - Sequence Number
            Misal:
            DO2025-001 untuk data awal
            Selanjutnya jika sudah ada data tracking terbaru, menjadi DO2025-002 dst.
        - NIM, diisi secara manual oleh pengguna
        - Nama, diisi secara manual oleh pengguna
        - Ekspedisi (JNE Regular/JNE Express), dipilih oleh pengguna berdasarkan data pada upbjjList
        - Paket Bahan Ajar, dipilih oleh pengguna berdasarkan data pada paket
            - Gunakan elemen dan menampilkan di halaman web informasi kode paket dan nama paket
            - Setelah memilih paket, akan muncul detail isi paket bahan ajar tersebut di bawah elemen untuk memberikan informasi
        - Tanggal Kirim, dapat diisi manual ataupun menggunakan fungsi Date untuk mengambil local time. Pada tanggal kirim, perlu diformat sebagai berikut: 
        tanggal bulan tahun. Contoh: 25 Agustus 2025
        - Total Harga, diambil dari data pada Array of Objects paket → harga. Pada tanggal kirim, perlu diformat dengan menambahkan currency (Rp) Buatlah juga validasi input yang sederhana saat memasukkan data delivery order baru.
    - Terdapat fitur untuk menambah status progress perjalanan pengiriman
        - Data “waktu” diambil berdasarkan local time menggunakan Date.
        - Data “keterangan” diambil berdasarkan input pengguna