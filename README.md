# Laporan Praktikum: Rekayasa Sistem Basis Data: Persistensi MySQL, Connection Pooling, Prepared Statements, dan Arsitektur CRUD 

Laporan praktikum ini mengimplementasi aplikasi web berbasis **Express.js**, **MySQL (`mysql2`)**, dan **EJS (Embedded JavaScript)**, mencakup operasi CRUD dasar, penerapan pola *Post-Redirect-Get* (PRG), pengujian mitigasi *SQL Injection*, serta laboratorium investigasi masalah (*troubleshooting*).

---

## Identitas Praktikan
* **Nama:** Made Adelia Febriana
* **NIM:** 42530007
* **Mata Kuliah:** Pemrograman Web (Semester 3)
* **Dosen Pengampu:** Ir. Gede Humaswara Prathama, S.T., M.T.
* **Institusi:** Program Studi Teknologi Informasi, Fakultas Teknik dan Informatika, Universitas Pendidikan Nasional (2026)

---

## Bagian 1: Setup Lingkungan Kerja & Instalasi Dependensi
Tahap awal konfigurasi lingkungan pengembangan (development environment) menggunakan Command Prompt (CMD) serta inisialisasi manajemen dependensi Express, EJS, Driver MySQL2, dan Dotenv.


---

## Bagian 2: Pengalaman Laboratorium Terpandu (Progressive Hands-On Labs)

Praktikum ini dibagi menjadi 6 tahap eksperimen:

1. **Pengalaman 1: Persiapan  Skema  Basis  Data  &  Konfigurasi  Connection  Pool Terisolasi**
   * Tujuan: Mengonfigurasi basis data MySQL lokal, mengamankan kredensial server via variabel lingkungan (.env), dan menginisialisasi connection pool.
   * MySQL Workbench
    <img width="900" height="712" alt="image" src="https://github.com/user-attachments/assets/4e8c14cb-cce6-4e56-82b3-8ffb5c3ae601" />
    
   * **Inspeksi Terminal :** Pastikan  terminal mencetak  string  pesan [DATABASE]  Terhubung  ke  MySQL Pool  secara  aman.
   * 01-db-pool-connection.png.
    <img width="1917" height="1005" alt="image" src="https://github.com/user-attachments/assets/177e6c36-1964-4457-9369-fb9b7bc72193" />

  
2. **Pengalaman 2: Abstraksi Layer Model & Operasi READ Data ke T abel EJS**
   * Membangun kueri SELECT berparameter pada layer Model dan menyajikannya secara dinamis ke dalam tabel EJS.
     
   *  **Inspeksi DevTools:** Amati  bahwa  dokumen  HTML  memiliki  ukuran  (transfer size)  yang  membawa  seluruh  tabel  data  dari MySQL dengan Status Code: 200 OK
   * 02-read-select-render.png.
     <img width="1916" height="1015" alt="image" src="https://github.com/user-attachments/assets/dd26d75c-8d86-4ddb-aa18-d956e19c77a1" />


3. **Pengalaman 3: Transaksi CREATE Data via Prepared Statements & Pola PRG**
   * Mengirim  data  input  dari  formulir  HTML,  mengeksekusi  query  INSERT  berparameter secara aman, dan menerapkan pola Post-Redirect-Get.
  
   * **Inspeksi DevTools:** Amati  urutan  request  di  tab  Network:  Request  pertama  adalah  /produk/tambah dengan  status 302  Found  yang  memiliki  respons  header  Location:  /produk. Request berikutnya adalah /produk dengan metode GET dan status 200 OK.
   * 03-create-insert-prg.png.
     <img width="1917" height="1026" alt="image" src="https://github.com/user-attachments/assets/22831af5-0688-4461-a286-8a1bb6749f2d" />


4. **Pengalaman 4: Parameter Rute Dinamis (req.params.id) & Operasi UPDATE**
   * Mengambil  data  spesifik  berdasarkan nilai parameter URL untuk mengisi formulir edit (pre-filling), lalu memperbarui data ke MySQL.
     
   * **Inspeksi DevTools:** formulir edit menampilkan data lama (pre-filled) dan setelah perubahan berhasil tersimpan di tabel utama
   * 04-update-param-redirect.png.
     <img width="1917" height="1017" alt="image" src="https://github.com/user-attachments/assets/96056baa-2b67-4e54-804c-360063a81952" />


5. **Pengalaman 5: Operasi DELETE Berparameter & Validasi Integritas**
   * Menghapus baris entitas pada basis data menggunakan parameter id dan memastikan data benar-benar terhapus secara persisten.
     
   * **Inspeksi DevTools & Database:** Pada terminal VS Code yang mencetak log penghapusan Record
ID X berhasil dihapus dengan metode POST dan status 302.

   * 05-delete-feedback.png.
     <img width="1600" height="548" alt="image" src="https://github.com/user-attachments/assets/d0f8286e-40a4-437b-a676-22f117b2bc15" />

6. **Pengalaman 6: Eksperimen  Keamanan:  Uji  Serangan  SQL  Injection  vs  Prepared Statements**
   * Membuktikan  secara  empiris  kelemahan  kueri konkatenasi (string concatenation) 
     
   * **Simulasi Serangan SQL Injection:**
     *Pengujian  1  (Rentan):  Aktifkan  getByKategoriInsecure.  Amati  terminal:  Kueri
yang  terbentuk  menjadi SELECT  *  FROM  produk WHERE kategori = 'Periferal'
OR  '1'='1'.  Akibatnya,  seluruh  isi  tabel  bocor  ke  publik  karena  klausul  '1'='1'
selalu bernilai benar (truthy).
     *Pengujian  2  (Aman):  Aktifkan  getByKategoriSecure.  Muat  ulang  URL  yang
sama.  Amati  terminal:  Driver  MySQL  secara  otomatis  melakukan  escaping
karakter kutip dan memperlakukan seluruh string Periferal' OR '1'='1 sebagai nilai
literal, bukan perintah SQL. Hasil query menghasilkan 0 data atau hanya mencari
teks persis tersebut.

   * 06-sql-injection-mitigation.png.
     <img width="1917" height="1017" alt="image" src="https://github.com/user-attachments/assets/882d9ea7-d216-4e3a-9b75-e7052838f1d3" />

---

## Bagian 3: Laboratorium Investigasi Masalah (Troubleshooting & Error Analysis)
Setiap  praktikan  wajib  memicu,  menganalisis,  dan  memperbaiki  tiga  skenario  kegagalan  fatal
(fatal backend crashes) berikut


### 1. Kasus Investigasi 1: "ER_ACCESS_DENIED_ERROR" pada Variabel Lingkungan
* **Pemicu Galat:** Buka berkas .env Anda, ubah DB_PASSWORD menjadi sandi acak yang salah (misal: salah_password_123). Simpan  berkas, lalu matikan & jalankan ulang server (npm run dev). 

* **Analisis & Temuan:** Terminal akan menampilkan unhandled exception 
* 07-error-access-denied.png.
  <img width="1915" height="1013" alt="image" src="https://github.com/user-attachments/assets/907fb6d3-b218-4367-b9bd-3fbc094c6dcf" />


### 2. Kasus Investigasi 2: "Bind parameters must not contain undefined"
* **Pemicu Galat:** Buka  views/pages/produk-form-tambah.ejs. Ubah atribut name="harga" menjadi  salah ketik: name="harga_produk".Simpan, muat form di browser, lalu kirim formulir.
  
* **Analisis & Temuan:** Server Express akan crash dan terminal memuntahkan log: Error: Bind parameters must not contain undefined.
* 08-error-undefined-bind.png.
  <img width="1915" height="1016" alt="image" src="https://github.com/user-attachments/assets/56d62d7b-9e9d-4710-8bbb-b9386016b41a" />


### 3. Kasus Investigasi 3: "Connection Pool Exhaustion / Timeout"
* **Pemicu Galat:** Pada config/database.js, set connectionLimit:1 dan hapus penanganan promise. Kemudian lakukan pemanggilan kueri di controller tanpa melepaskan (release).
  
* **Analisis & Temuan:** Saat banyak request masuk simultan, server akan berhenti meresponshanging / infinite loading) karena pool kehabisan koneksi bebas.


---

#### Logbook Pengujian & Integritas Kode 
## Logbook Pengujian & Integritas Kode (Traceability Matrix)

Matriks penelusuran pengujian (*Traceability Matrix*) ini memuat rangkaian uji coba, skenario, serta bukti tangkapan layar dari seluruh modul praktikum:

| ID Uji | Fitur / Operasi | Target Endpoint URL | HTTP Method | Skenario Pengujian | Hasil yang Diharapkan | Status | Bukti Tangkapan Layar |
| :--- | :--- | :--- | :---: | :--- | :--- | :---: | :--- |
| **TC-01** | Koneksi Pool | `App Startup` | - | Server dijalankan pertama kali | Terhubung ke MySQL Pool secara aman tanpa eror | Pass | [docs/screenshots/01-db-pool-connection.png](https://drive.google.com/file/d/1o5LYZ0oBg4pfR4mkhhmnbH2FiReE1VjR/view?usp=drive_link) |
| **TC-02** | Read (All) | `/produk` | GET | Akses halaman daftar produk dan inspeksi via DevTools Network | Menampilkan seluruh record tabel MySQL ke antarmuka EJS (Status 200 OK) | Pass | [docs/screenshots/02-read-select-render.png](https://drive.google.com/file/d/1yNcpPbQWs6cKOWIFUwXPPJUqm3baHtcV/view?usp=drive_link) |
| **TC-03** | Create | `/produk/tambah` | POST | Submit formulir produk baru dengan pola *Post-Redirect-Get* | Data tersimpan, DevTools mencatat alur 302 Found $\rightarrow$ 200 OK | Pass | [docs/screenshots/03-create-insert-prg.png](https://drive.google.com/file/d/1RIcRIADOwcXbobbewTRBpgIFzFde0tGx/view?usp=drive_link) |
| **TC-04** | Update | `/produk/edit/:id` | POST | Menggunakan rute dinamis untuk *pre-filling* form edit dan mengubah data | Nilai kolom pada baris MySQL berubah sesuai input baru | Pass | [docs/screenshots/04-update-param-redirect.png](https://drive.google.com/file/d/1K5zypyH0tSE6juy7XWGkwy4T9Sg9eliq/view?usp=drive_link) |
| **TC-05** | Delete | `/produk/hapus/:id` | POST | Eksekusi tombol hapus dengan parameter ID dan validasi basis data | Baris record terhapus dari MySQL dan UI ter-refresh | Pass | [docs/screenshots/05-delete-feedback.png](https://drive.google.com/file/d/1Pz2B-ZUdKEu2jjfdKmJCAEWTgOWLdA17/view?usp=drive_link) |
| **TC-06** | Keamanan | `/uji-keamanan` | GET | Input payload `' OR '1'='1` via URL untuk uji kerentanan | Prepared Statement memperlakukan input sebagai literal string (No Injection) | Pass | [docs/screenshots/06-sql-injection-mitigation.png](https://drive.google.com/file/d/14hciZ40c-xsZH6P7VoJvQZgR4MpS4Pyj/view?usp=drive_link) |

