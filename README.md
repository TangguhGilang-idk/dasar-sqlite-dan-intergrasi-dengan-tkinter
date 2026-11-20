Praktikum **Dasar SQLite dan Integrasi dengan Tkinter**. Aplikasi desktop berbasis Python untuk menyimpan data mahasiswa ke database SQLite, dengan antarmuka Tkinter.

- Nama: Tangguh Gilang Saputra
- NIM: 2595114026
- Kelas: Teknik Informatika

## Fitur

- **Tambah** data mahasiswa (`INSERT`)
- **Tampil** data pada tabel Treeview (`SELECT`)
- **Ubah** data (`UPDATE`)
- **Hapus** data dengan konfirmasi (`DELETE`)
- **Validasi masukan:** nama, NIM, dan program studi wajib diisi; NIM tidak boleh duplikat; angkatan harus tahun 4 digit
- **Pencarian** berdasarkan nama atau NIM (tugas pengembangan)
- **Kolom angkatan** dengan migrasi otomatis untuk database lama (tugas pengembangan)
- **Persistensi:** data tersimpan di file `mahasiswa.db` dan tetap ada setelah aplikasi ditutup
- Semua query memakai *placeholder* `?` sehingga aman dari SQL injection

## Struktur Proyek

```
.
├── aplikasi_mahasiswa.py   # kode aplikasi
├── mahasiswa.db            # database SQLite
├── screenshot/             # folder tangkapan layar
│   ├── formulir.png        # formulir aplikasi
│   ├── treeview.png        # data pada Treeview
│   └── isi_database.png    # isi database di DB Browser
└── README.md
```

## Struktur Database

Tabel `mahasiswa`:

| Kolom | Tipe | Keterangan |
| --- | --- | --- |
| id | INTEGER | Primary key, autoincrement |
| nama | TEXT | NOT NULL |
| nim | TEXT | NOT NULL, UNIQUE (TEXT agar angka nol di depan tidak hilang) |
| prodi | TEXT | NOT NULL |
| alamat | TEXT | Opsional |
| angkatan | INTEGER | Opsional |

## Cara Menjalankan

Persyaratan: Python 3 dengan Tkinter (sudah termasuk pada instalasi Python standar di Windows). Modul `sqlite3` bawaan Python, tidak perlu instalasi tambahan.
```
python aplikasi_mahasiswa.py
```
Jika `mahasiswa.db` belum ada, file dan tabelnya dibuat otomatis di folder yang sama dengan `aplikasi_mahasiswa.py`.
---

# Berkas yang Dikumpulkan

## 1. File `aplikasi_mahasiswa.py`

Kode aplikasi lengkap: koneksi database, pembuatan tabel, antarmuka Tkinter, dan fungsi simpan, tampil, ubah, hapus, serta cari. Lihat [aplikasi_mahasiswa.py](aplikasi_mahasiswa.py).

## 2. File database `mahasiswa.db`

Database SQLite hasil praktikum yang berisi tabel `mahasiswa` beserta data yang disimpan melalui aplikasi. Lihat [mahasiswa.db](mahasiswa.db).

## 3. Tangkapan layar formulir aplikasi dan data pada Treeview

**Formulir aplikasi**

![Formulir aplikasi](screenshot/formulir.png)

**Data pada Treeview**

![Treeview](screenshot/treeview.png)

## 4. Tangkapan layar isi database pada DB Browser for SQLite

![Isi database](screenshot/isi_database.png)

## 5. Tabel hasil pengujian

| No. | Skenario | Langkah uji | Hasil yang diharapkan | Hasil aktual | Status |
| --- | --- | --- | --- | --- | --- |
| 1 | Data valid | Isi nama, NIM, prodi, lalu klik Simpan Data | Data muncul pada tabel | Muncul pesan "Data berhasil disimpan" dan data tampil di Treeview | Berhasil |
| 2 | Masukan kosong | Kosongkan salah satu kolom wajib | Peringatan muncul, data tidak disimpan | Muncul peringatan "Nama, NIM, dan Program Studi wajib diisi" | Berhasil |
| 3 | NIM duplikat | Simpan dua data dengan NIM sama | Data kedua ditolak dengan pesan galat | Muncul pesan "NIM sudah terdaftar", data kedua tidak masuk | Berhasil |
| 4 | Refresh | Klik tombol Refresh | Daftar dimuat ulang dari database | Daftar dimuat ulang sesuai isi database | Berhasil |
| 5 | Hapus | Pilih satu baris, klik Hapus, konfirmasi | Baris hilang dari tabel dan database | Baris hilang dari Treeview dan dari file database | Berhasil |
| 6 | Ubah | Pilih satu baris, ubah nama, klik Ubah Data | Nama baru tampil pada tabel | Nama berubah di Treeview dan di database | Berhasil |
| 7 | Persistensi | Tutup lalu buka kembali aplikasi | Data sebelumnya tetap tersedia | Data tetap tampil setelah aplikasi dibuka ulang | Berhasil |
| 8 | Pemeriksaan DB | Buka file melalui DB Browser | Tabel dan record dapat diperiksa | Tabel `mahasiswa` dan datanya terbaca (lihat `screenshot/isi_database.png`) | Berhasil |

Pengujian tambahan (tugas pengembangan):

| No. | Skenario | Hasil aktual | Status |
| --- | --- | --- | --- |
| 9 | Pencarian dengan kata kunci yang ada | Hanya data yang namanya atau NIM-nya cocok yang tampil | Berhasil |
| 10 | Angkatan bukan angka 4 digit (misalnya `20x3`) | Ditolak dengan peringatan validasi | Berhasil |
| 11 | Ubah NIM menjadi NIM milik data lain | Ditolak dengan pesan "NIM sudah digunakan data lain" | Berhasil |
| 12 | Database lama tanpa kolom angkatan | Kolom angkatan ditambahkan otomatis, data lama tetap utuh | Berhasil |
| 13 | NIM berawalan nol (`00123`) | Tetap tersimpan dan tampil sebagai `00123` | Berhasil |

## 6. Hasil tugas pengembangan (Bagian 2.7)

### 6.1 Fitur pencarian

Kotak **Cari (nama / NIM)** ditambahkan di atas Treeview. Pencarian memakai query berparameter:

```sql
SELECT id, nama, nim, prodi, alamat, angkatan
FROM mahasiswa
WHERE nama LIKE ? OR nim LIKE ?
ORDER BY id DESC
```

Parameternya berbentuk `%kata_kunci%`, sehingga kata kunci dicari di bagian mana pun dari nama atau NIM. Hasil tampil langsung saat mengetik, dan tombol **Reset** mengembalikan semua data. Karena memakai placeholder `?`, masukan pengguna diperlakukan sebagai data, bukan bagian dari perintah SQL.

### 6.2 Kolom angkatan dan penanganan database lama

Kolom `angkatan INTEGER` ditambahkan pada tabel, formulir, dan Treeview. `CREATE TABLE IF NOT EXISTS` tidak mengubah tabel yang sudah ada, sehingga file `mahasiswa.db` lama tidak otomatis mendapat kolom baru. Langkah yang ditempuh:

1. Memeriksa struktur tabel dengan `PRAGMA table_info(mahasiswa)`.
2. Jika kolom `angkatan` belum ada, menjalankan `ALTER TABLE mahasiswa ADD COLUMN angkatan INTEGER`.
3. Data lama tetap utuh, dan kolom angkatan pada data lama bernilai NULL (tampil kosong).

Cara ini lebih aman daripada menghapus file database karena data yang sudah tersimpan tidak hilang.

### 6.3 Pengujian persistensi

Data disimpan di file `mahasiswa.db`, bukan di memori program. Buktinya, data yang disimpan tetap muncul setelah aplikasi ditutup dan dibuka lagi, serta dapat dilihat dari luar aplikasi melalui DB Browser (lihat `screenshot/isi_database.png`).

## 7. Jawaban pertanyaan refleksi

### 7.1 Peran `connect()`, `execute()`, dan `commit()`

`connect()` membuka hubungan antara program dan file database `mahasiswa.db`, dan membuat file itu bila belum ada. `execute()` mengirim perintah SQL ke database, misalnya `INSERT` untuk menambah data atau `SELECT` untuk membaca data, dengan nilai yang dikirim terpisah lewat placeholder agar aman. Perubahan dari `execute()` baru berupa transaksi sementara. `commit()` yang menyimpannya secara permanen ke file. Tanpa `commit()`, data tampak tersimpan selama koneksi terbuka tetapi hilang ketika koneksi ditutup.

Pada aplikasi ini ketiganya dibungkus fungsi `koneksi()`. Fungsi itu membuka koneksi, melakukan `commit()` jika blok `with` berhasil, melakukan `rollback()` jika terjadi galat, dan selalu menutup koneksi di bagian `finally`. Dengan begitu data tidak setengah tersimpan dan file database tidak terkunci.

### 7.2 Bagian paling menantang dan cara mengatasinya

Bagian yang paling menantang adalah menghubungkan tombol di Tkinter dengan fungsi database, terutama menjaga urutan kode. Fungsi harus dibuat sebelum tombol yang memakainya, dan widget harus dibuat sebelum fungsi memanggilnya, jika tidak muncul `NameError`. Saya mengatasinya dengan membaca pesan galat baris demi baris, menyusun kode sesuai urutan di modul (impor, koneksi, tabel, antarmuka, fungsi, tombol, `mainloop`), dan mencoba setiap fungsi satu per satu. Masalah lain yang saya temui adalah tabel kosong di DB Browser. Penyebabnya, file `mahasiswa.db` yang dibuka berbeda dengan file yang dibuat aplikasi karena lokasi database bergantung pada folder tempat program dijalankan. Saya mengatasinya dengan menetapkan lokasi database di folder yang sama dengan file `.py`.

### 7.3 Fitur dan perlindungan data untuk bagian administrasi program studi

Jika aplikasi dipakai oleh bagian administrasi, beberapa hal perlu ditambahkan:

- **Login dan peran pengguna**, agar hanya petugas berwenang yang boleh mengubah atau menghapus data.
- **Cadangan (backup) berkala** file database, karena satu file yang rusak atau terhapus berarti seluruh data hilang.
- **Catatan riwayat perubahan (audit log)** untuk mengetahui siapa mengubah atau menghapus data dan kapan.
- **Soft delete**, yaitu menandai data tidak aktif tanpa menghapusnya permanen, agar salah klik tidak menghilangkan data.
- **Perlindungan data pribadi**, misalnya membatasi akses ke alamat mahasiswa dan mengenkripsi file database.
- **Database server** seperti MySQL atau PostgreSQL bila pengguna bertambah banyak dan bekerja bersamaan, karena SQLite kurang cocok untuk akses serentak.
