# 📚 BookStore — Sistem Manajemen Toko Buku Digital (Modular)

Proyek tugas praktikum **Studi Kasus Modular** — dibangun menggunakan **Java
Platform Module System (JPMS)** murni dari CLI (`javac` + `java
--module-path`), tanpa Maven/Gradle, sesuai ketentuan tugas.

## 1. Struktur Modul

Sesuai studi kasus, sistem dipecah menjadi **3 modul independen**:

| Modul | Tanggung Jawab | Mengekspor |
|---|---|---|
| `app.data` | Entitas data (`Buku`, `Kategori`) & simulasi database internal (`InMemoryDatabase`) | `com.bookstore.data.entity`, `com.bookstore.data.repository` |
| `app.logic` | Logika bisnis: diskon, hitung total, validasi stok | `com.bookstore.logic.service`, `com.bookstore.logic.dto` |
| `app.ui` | CLI interaktif, menerima input pengguna, **main entry point** | — |

```
app.data   ⟶  app.logic  ⟶  app.ui
(entitas)     (bisnis)      (CLI)
```

### Dependensi (`module-info.java`)

```
module app.data { exports com.bookstore.data.entity; exports com.bookstore.data.repository; }
module app.logic { requires transitive app.data; exports com.bookstore.logic.service; exports com.bookstore.logic.dto; }
module app.ui   { requires app.logic; }
```

## 2. Larangan Keras: app.ui ⛔ app.data (internal)

Paket `com.bookstore.data.internal` (tempat `InMemoryDatabase` berada) **tidak
pernah diekspor** oleh `app.data`. Karena `app.ui` hanya `requires app.logic`
dan tidak pernah mengimpor paket internal tersebut, Java Module System
menegakkan **strong encapsulation**: kompilasi akan **gagal** jika ada kode di
`app.ui` (atau modul lain mana pun) mencoba mengimpor
`com.bookstore.data.internal.*` secara langsung.

Ini sudah diverifikasi: mencoba mengimpor paket itu dari modul lain
menghasilkan:
```
error: package com.bookstore.data.internal is not visible
  (package com.bookstore.data.internal is declared in module app.data, which does not export it)
```

`app.ui` hanya berbicara dengan `BookService` (fasad di `app.logic`), yang di
baliknya memanggil `BukuRepository` (API publik `app.data`) → `InMemoryDatabase`
(internal, tersembunyi).

## 3. Struktur Folder

```
bookstore-modular/
├── app.data/
│   ├── module-info.java
│   └── com/bookstore/data/
│       ├── entity/        (Buku, Kategori)
│       ├── repository/    (BukuRepository — API publik)
│       └── internal/      (InMemoryDatabase — TIDAK diekspor)
├── app.logic/
│   ├── module-info.java
│   └── com/bookstore/logic/
│       ├── service/       (BookService, DiscountService)
│       └── dto/           (HasilTransaksi)
├── app.ui/
│   ├── module-info.java
│   └── com/bookstore/ui/
│       ├── Main.java       (entry point CLI)
│       └── Console.java    (helper tampilan ANSI)
├── build.sh
└── run.sh
```

## 4. Cara Menjalankan

Butuh **JDK 11+** (memakai fitur module system, tested di JDK 21).

```bash
chmod +x build.sh run.sh
./build.sh   # kompilasi 3 modul secara berurutan ke mods/
./run.sh     # jalankan CLI
```

Atau manual:

```bash
javac -d mods/app.data $(find app.data -name "*.java")
javac --module-path mods -d mods/app.logic $(find app.logic -name "*.java")
javac --module-path mods -d mods/app.ui $(find app.ui -name "*.java")
java  --module-path mods -m app.ui/com.bookstore.ui.Main
```

## 5. Fitur CLI

```
╔════════════════════════════════════════════════════════════════╗
║                         MENU UTAMA                              ║
╚════════════════════════════════════════════════════════════════╝
  [1]  Tampilkan semua buku
  [2]  Cari buku berdasarkan kategori
  [3]  Beli buku
  [4]  Tambah buku baru
  [0]  Keluar
```

- **Katalog** tabel rapi dengan warna (stok menipis → merah).
- **Beli buku** otomatis menghitung diskon bertingkat (5%/10%/15%) dan
  mencetak struk, sekaligus memvalidasi & mengurangi stok lewat `app.logic`.
- **Tambah buku** untuk menambah entri katalog baru secara runtime.

## 6. Ketentuan Tugas yang Dipenuhi

- ✅ Setiap modul punya `module-info.java` terkonfigurasi benar.
- ✅ `app.ui` tidak mengakses paket internal `app.data` secara langsung
  (terverifikasi gagal-kompilasi jika dicoba).
- ✅ Format pengumpulan: unggah repo ini ke GitHub, lalu tautannya ke LMS.
- ✅ Kelompok maksimal 3 mahasiswa — isi nama anggota di bawah ini.

| No | Nama | NIM |
|---|---|---|
| 1 | _isi nama_ | _isi NIM_ |
| 2 | _isi nama_ | _isi NIM_ |
| 3 | _isi nama_ | _isi NIM_ |
