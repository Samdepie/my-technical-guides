# Panduan Lengkap Install Java & Troubleshooting Aplikasi JAR di Windows

> **Untuk pemula.** Panduan ini bisa dibagikan di GitHub sebagai referensi belajar atau pegangan saat sebuah aplikasi Java tidak bisa dibuka. Contoh menggunakan **Windows 10/11 + Command Prompt (CMD)** dan distribusi **Eclipse Temurin JDK**. Semua nama aplikasi, folder, database, dan kredensial di bawah bersifat **contoh**; tidak berisi konfigurasi perusahaan atau data pribadi.

## Daftar isi

1. [Memahami Java sebelum install](#1-memahami-java-sebelum-install)
2. [Memilih versi Java yang benar](#2-memilih-versi-java-yang-benar)
3. [Cara install JDK di Windows](#3-cara-install-jdk-di-windows)
4. [Mengecek instalasi dan lokasi Java](#4-mengecek-instalasi-dan-lokasi-java)
5. [Cara menjalankan aplikasi JAR](#5-cara-menjalankan-aplikasi-jar)
6. [Panduan diagnosis berdasarkan pesan error](#6-panduan-diagnosis-berdasarkan-pesan-error)
7. [Mengatasi bentrok versi Java dan PATH](#7-mengatasi-bentrok-versi-java-dan-path)
8. [Membuat shortcut menjalankan aplikasi dengan BAT](#8-membuat-shortcut-menjalankan-aplikasi-dengan-bat)
9. [Masalah konfigurasi dan database setelah aplikasi berjalan](#9-masalah-konfigurasi-dan-database-setelah-aplikasi-berjalan)
10. [Mengumpulkan log dan mencari akar masalah](#10-mengumpulkan-log-dan-mencari-akar-masalah)
11. [Masalah lain yang mungkin terjadi](#11-masalah-lain-yang-mungkin-terjadi)
12. [Tips untuk developer dan distribusi ke pengguna](#12-tips-untuk-developer-dan-distribusi-ke-pengguna)
13. [Checklist dan ringkasan cepat](#13-checklist-dan-ringkasan-cepat)

---

## 1. Memahami Java sebelum install

### Apa itu Java?

Java adalah bahasa pemrograman dan platform eksekusi. Developer menulis source code, lalu *compiler* mengubahnya menjadi **bytecode** yang dapat dijalankan oleh Java Runtime yang sesuai.

Alur sederhana:

```text
Source Java (.java)
        |
        v
Compiler / javac
        |
        v
Bytecode (.class)
        |
        v
Dikemas menjadi JAR (.jar)
        |
        v
Dijalankan oleh Java Runtime / java.exe
```

**Catatan:** proses build Java modern dapat memakai Maven, Gradle, dan sebagainya. Sebuah JAR bisa berupa aplikasi lengkap, library, atau arsip yang masih membutuhkan dependensi eksternal.

### Istilah penting

| Istilah | Penjelasan sederhana | Contoh |
|---|---|---|
| Java Runtime | Bagian yang menjalankan bytecode Java | `java.exe` |
| JDK | Paket runtime **dan** tools development | `java`, `javac`, `jar`, `jpackage` |
| JRE | Runtime saja (istilah umum di instalasi lama) | Menjalankan aplikasi |
| `.java` | Source code developer | `Main.java` |
| `.class` | Bytecode hasil kompilasi | `Main.class` |
| `.jar` | Arsip yang berisi class, resource, dan metadata | `aplikasi.jar` |
| LTS | *Long-Term Support*; dukungan jangka panjang | Java 21, Java 25 |
| PATH | Daftar folder pencarian program oleh Windows | Menentukan `java` mana yang ditemukan |
| `JAVA_HOME` | Lokasi folder instalasi JDK untuk tool lain | `C:\Program Files\...\jdk-21...` |
| CMD | Terminal bawaan Windows untuk perintah berbasis teks | `java -version` |

**Perbedaan penting:** `JAVA_HOME` dan `PATH` adalah dua hal berbeda. Mengubah `JAVA_HOME` **tidak otomatis menjamin** CMD memakai versi Java itu kalau urutan PATH masih mengarah ke versi lama.

## 2. Memilih versi Java yang benar

**Jangan hanya melihat angka paling besar.** Pilih versi sesuai persyaratan aplikasi dan dukungan library.

Jika aplikasi Java dibuat untuk Java 21, Java 17 tidak bisa langsung menjalankan class yang membutuhkan Java 21. Rilis Java yang lebih baru umumnya bisa memuat class lama, tetapi kompatibilitas aplikasi secara keseluruhan perlu diuji.

### Contoh relasi versi Java dan bytecode

| Target kompilasi Java | Class file major version |
|---|---:|
| Java 8 | 52 |
| Java 11 | 55 |
| Java 17 | 61 |
| Java 21 | 65 |
| Java 25 | 69 |

Misalnya error:

```text
UnsupportedClassVersionError
... class file version 65.0
... only recognizes class file versions up to 61.0
```

Artinya:

- Aplikasi memuat class **target Java 21** (`65`).
- Runtime yang menjalankan aplikasi hanya mendukung sampai **Java 17** (`61`).
- Solusinya memakai runtime yang sesuai, biasanya **JDK/JRE Java 21** atau versi lebih baru yang kompatibel.

### Harus Java 21, 22, 23, atau 25?

- **Java 21 LTS**: pilihan wajar jika aplikasi dibangun untuk Java 21 dan belum ada tuntutan versi lebih tinggi.
- **Java 25 LTS**: pilihan lebih baru; bisa digunakan jika sudah teruji dengan aplikasi dan dependensinya.
- **Java 22/23**: rilis non-LTS, tidak ideal untuk instalasi baru yang mengutamakan dukungan panjang.

**Prinsip praktis:** gunakan versi yang diminta developer/vendor. Untuk software perusahaan, standar lingkungan pengguna idealnya ditentukan dan diuji, bukan berbeda-beda secara acak.

## 3. Cara install JDK di Windows

### Langkah 1 — Download dari sumber resmi

Buka:

**https://adoptium.net/temurin/releases/**

Pilih berdasarkan kebutuhan aplikasi:

| Pengaturan | Contoh pilihan |
|---|---|
| Version | `21 - LTS` |
| Operating System | `Windows` |
| Architecture | `x64` untuk PC Windows 64-bit x86 |
| Package Type | `JDK` |
| Download | `.msi` installer |

> Jika perangkat memakai Windows ARM64, pilih arsitektur yang sesuai. Jangan memilih x64 hanya karena contoh di sini menggunakan x64.

**Mengapa MSI, bukan ZIP?**

- **MSI**: installer Windows; lebih mudah untuk pemula dan dapat menyediakan integrasi PATH / file association / `JAVA_HOME`.
- **ZIP**: arsip yang diekstrak manual; cocok jika ingin instalasi portable atau mengatur environment sendiri.

### Langkah 2 — Jalankan installer MSI

Selama instalasi biasanya ada beberapa opsi:

| Opsi installer | Fungsinya | Rekomendasi pemula |
|---|---|---|
| Modify PATH variable | Menambahkan lokasi tools Java ke PATH | Aktifkan jika ingin memakai `java` dari CMD |
| Associate `.jar` | Mengaitkan double-click JAR dengan Java | Opsional; CMD tetap lebih baik untuk debug |
| Set or override `JAVA_HOME` | Menentukan folder JDK untuk tool lain | Berguna jika menggunakan Maven/Gradle; pertimbangkan dampak bila punya JDK lain |
| JavaSoft registry keys | Kompatibilitas registri untuk software tertentu | Biasanya tidak wajib kecuali aplikasi memintanya |

**Perbedaan “Will be installed” vs “Entire feature will be installed”:**

- **Will be installed on local hard drive**: memilih fitur tersebut.
- **Entire feature will be installed on local hard drive**: memilih fitur berikut semua subfiturnya.
- Jika fitur tunggal tidak mempunyai subfitur, perbedaan praktisnya biasanya tidak ada.

Lanjutkan instalasi sampai selesai, kemudian **tutup terminal lama dan buka CMD baru**.

### Langkah 3 — Konfirmasi instalasi

```cmd
java -version
```

Contoh hasil (angka patch berubah seiring pembaruan):

```text
openjdk version "21.x.x"
OpenJDK Runtime Environment Temurin-21
OpenJDK 64-Bit Server VM
```

Jika ternyata masih versi lama, **jangan langsung uninstall apa pun**. Lanjutkan ke bagian PATH dan pemanggilan `java.exe` secara langsung.

## 4. Mengecek instalasi dan lokasi Java

Buka **Command Prompt / CMD** lalu jalankan:

```cmd
java -version
where java
where javac
where jar
echo %JAVA_HOME%
```

Interpretasi:

- `java -version`: Java yang dipakai oleh perintah `java` pada CMD itu.
- `where java`: lokasi executable Java yang ditemukan; urutan membantu menganalisis konflik PATH.
- `where javac`: compiler Java yang ditemukan lewat PATH.
- `where jar`: alat arsip JAR yang ditemukan lewat PATH.
- `echo %JAVA_HOME%`: melihat lokasi JDK yang dikonfigurasi sebagai `JAVA_HOME`.

**Jangan salah simpulkan:** jika `jar` tidak dikenal, belum tentu aplikasi Java tidak bisa dijalankan. `java.exe` dan `jar.exe` adalah dua program yang berbeda. Begitu juga, output `where javac` saja belum cukup membuktikan sebuah JDK lengkap berfungsi dengan baik.

### Cara menemukan direktori JDK

Contoh lokasi instalasi Temurin:

```cmd
dir "C:\Program Files\Eclipse Adoptium"
```

Lalu sesuaikan path dengan **nama folder yang benar-benar ditemukan**, misalnya:

```cmd
"C:\Program Files\Eclipse Adoptium\jdk-21.0.8.9-hotspot\bin\java.exe" -version
```

> `jdk-21.0.8.9-hotspot` **hanya contoh**. Jangan copy path persis kalau foldermu bernama lain.

Kenapa pakai tanda petik? Karena path berisi spasi di `Program Files`. Tanpa tanda petik, CMD bisa membaca `C:\Program` sebagai perintah terpisah.

## 5. Cara menjalankan aplikasi JAR

Misalnya aplikasi berada di:

```text
C:\Apps\ContohJava\
    aplikasi.jar
    config.properties
```

Buka CMD, masuk ke foldernya:

```cmd
cd /d "C:\Apps\ContohJava"
```

Lihat JAR yang ada:

```cmd
dir /b *.jar
```

Jalankan:

```cmd
java -jar "aplikasi.jar"
```

Jika default Java salah, panggil Java dari JDK yang benar secara langsung. Contoh berikut **harus disesuaikan dengan folder instalasi**:

```cmd
"C:\Program Files\Eclipse Adoptium\jdk-21.0.8.9-hotspot\bin\java.exe" -jar "aplikasi.jar"
```

### Kenapa lebih baik CMD daripada double-click?

Pada aplikasi desktop, jendela GUI bisa gagal muncul ketika program startup melempar exception. Jika double-click, console error mungkin langsung hilang. CMD memungkinkan kita membaca pesan penyebabnya.

### Cara cek JAR valid

```cmd
dir "aplikasi.jar"
```

Kalau ukuran `0 bytes`, file kosong dan harus diganti. Kalau ukurannya ada, tes isi arsip:

```cmd
tar -tf "aplikasi.jar"
```

Atau, jika JDK tersedia di PATH:

```cmd
jar tf "aplikasi.jar"
```

Jika `jar` tidak dikenali tetapi JDK ter-install, jalankan `jar.exe` dengan path penuh, misalnya:

```cmd
"C:\Program Files\Eclipse Adoptium\jdk-21.0.8.9-hotspot\bin\jar.exe" tf "aplikasi.jar"
```

Arsip yang bisa dibaca biasanya berisi entri seperti:

```text
META-INF/
META-INF/MANIFEST.MF
com/example/Main.class
```

JAR yang valid **belum tentu** executable. Untuk `java -jar`, manifest juga perlu menunjuk kelas utama (*Main-Class*) dan dependensinya harus tersedia.

## 6. Panduan diagnosis berdasarkan pesan error

### Error A — `Invalid or corrupt jarfile`

```text
Error: Invalid or corrupt jarfile aplikasi.jar
```

**Arti:** Java tidak dapat membaca file sebagai JAR valid.

**Kemungkinan:** file kosong, download terpotong, arsip rusak, atau file yang dinamai `.jar` sebenarnya bukan JAR.

**Langkah:**

```cmd
dir "aplikasi.jar"
jar tf "aplikasi.jar"
```

Jika `jar` tidak ada, pakai `tar -tf`. Bila file `0 bytes`, download/copy ulang. Kalau memungkinkan, bandingkan checksum SHA-256 dengan file dari pembuat:

```cmd
certutil -hashfile "aplikasi.jar" SHA256
```

Dua file dengan hash SHA-256 sama umumnya identik secara byte; pastikan nilai pembanding diperoleh dari sumber yang dipercaya.

### Error B — `UnsupportedClassVersionError`

```text
... class file version 65.0
... only recognizes class file versions up to 61.0
```

**Arti:** versi runtime terlalu lama untuk class tersebut.

**Langkah:**

```cmd
java -version
where java
```

Kemudian install atau pilih Java yang mendukung target class. Contoh `65` membutuhkan Java **21 atau lebih baru yang kompatibel**.

### Error C — `'java' is not recognized`

```text
'java' is not recognized as an internal or external command
```

**Arti:** Windows tidak menemukan `java.exe` lewat PATH.

**Solusi:** periksa apakah JDK sudah di-install, cari lokasi `bin\java.exe`, lalu jalankan lewat path penuh atau atur PATH.

### Error D — `'jar' is not recognized`

Berbeda dari error C: bisa jadi `java.exe` tersedia, tetapi tool `jar.exe` tidak ditemukan lewat PATH. Gunakan JDK yang lengkap, `jar.exe` dari lokasi JDK, atau `tar -tf` untuk sekadar memeriksa JAR.

### Error E — `'C:\Program' is not recognized`

**Arti:** path dengan spasi ditulis tanpa tanda petik.

**Salah:**

```cmd
C:\Program Files\Eclipse Adoptium\jdk-...
```

**Benar** (contoh path yang sudah disesuaikan):

```cmd
"C:\Program Files\Eclipse Adoptium\jdk-21.0.8.9-hotspot\bin\java.exe" -version
```

### Error F — `The system cannot find the path specified`

**Arti:** path yang diketik tidak cocok dengan direktori/file aktual.

**Solusi:**

```cmd
dir "C:\Program Files\Eclipse Adoptium"
```

Jangan mengarang nama folder berdasarkan nomor versi yang terlihat di website: nama folder aktual bisa memiliki format build/revisi berbeda.

### Error G — `Unable to access jarfile`

**Arti:** path/nama JAR salah, file tidak ada, atau izin akses bermasalah.

**Solusi:** gunakan `dir /b *.jar`, periksa apakah nama file memiliki tambahan `(1)`/`(2)`, dan tulis path dalam tanda petik.

### Error H — `no main manifest attribute`

**Arti:** JAR tidak dikemas sebagai aplikasi executable dengan kelas utama yang bisa dipakai `java -jar`.

**Solusi:** minta atau build ulang JAR dengan konfigurasi `Main-Class`/plugin packaging yang sesuai.

### Error I — `Could not find or load main class` / `NoClassDefFoundError`

**Arti:** Java tidak menemukan class utama atau dependensi class saat runtime.

**Solusi:** verifikasi classpath, manifest, versi library, dan apakah developer mengirim semua dependensi atau menghasilkan *fat/uber JAR*.

### Error J — Warning restricted native access

```text
WARNING: java.lang.System::load has been called ...
WARNING: Use --enable-native-access=ALL-UNNAMED ...
```

Ini biasanya berasal dari library yang memuat native library. Pada versi Java baru, penggunaan API tertentu dapat memicu warning.

**Jangan langsung menyamakan warning dengan error fatal.** Periksa apakah aplikasi tetap terbuka dan fitur bekerja. Jika dokumentasi library/vendor meminta flag native access, aplikasi **tepercaya** bisa dijalankan seperti:

```cmd
java --enable-native-access=ALL-UNNAMED -jar "aplikasi.jar"
```

Flag tersebut memperluas izin native access bagi code di unnamed modules/classpath. Gunakan seperlunya; solusi jangka panjang adalah memperbarui library atau memakai runtime yang didukung vendor.

## 7. Mengatasi bentrok versi Java dan PATH

Komputer dapat memiliki lebih dari satu instalasi Java, misalnya Java 17 dan Java 21. Itu normal.

Ketika mengetik:

```cmd
java -version
```

CMD memakai `java.exe` yang ditemukan menurut lingkungan pencarian executable. Jadi Java yang baru di-install **belum tentu** otomatis menjadi Java default.

### Solusi paling aman: jalankan versi yang diinginkan langsung

```cmd
"C:\Program Files\Eclipse Adoptium\jdk-21.0.8.9-hotspot\bin\java.exe" -jar "aplikasi.jar"
```

Kelebihannya: tidak mengubah aplikasi lain yang mungkin butuh Java lama.

### Solusi sementara: ubah PATH hanya untuk satu CMD

```cmd
set "PATH=C:\Program Files\Eclipse Adoptium\jdk-21.0.8.9-hotspot\bin;%PATH%"
java -version
java -jar "aplikasi.jar"
```

Ingat: contoh folder harus diganti sesuai instalasi sebenarnya. Perubahan hanya berlaku pada sesi CMD tersebut dan proses turunannya.

### Solusi permanen: ubah Environment Variables Windows

1. Tekan `Win + R` → ketik `sysdm.cpl`.
2. Pilih **Advanced → Environment Variables**.
3. Cari `Path` di **User variables** dan/atau **System variables**.
4. Pastikan folder `...\jdk-21...\bin` ada dan diprioritaskan terhadap launcher Java lama dalam PATH efektif.
5. Jika tools build membutuhkan `JAVA_HOME`, set ke folder JDK-nya **tanpa `\bin`**.
6. Tutup CMD lama, buka lagi.
7. Verifikasi `where java`, `java -version`, dan `echo %JAVA_HOME%`.

**Perhatian:** perubahan Java default berpotensi memengaruhi aplikasi lain. Untuk mesin produksi, uji dampak terlebih dahulu.

## 8. Membuat shortcut menjalankan aplikasi dengan BAT

Jika pengguna teknis ingin double-click tanpa terus mengetik perintah panjang, buat file `run-app.bat` di folder yang sama dengan `aplikasi.jar`:

```bat
@echo off
setlocal
cd /d "%~dp0"

set "JAVA_EXE=C:\Program Files\Eclipse Adoptium\jdk-21.0.8.9-hotspot\bin\java.exe"
set "JAR=aplikasi.jar"

if not exist "%JAVA_EXE%" (
    echo ERROR: Java tidak ditemukan.
    echo Periksa instalasi JDK dan lokasi JAVA_EXE di file BAT ini.
    pause
    exit /b 1
)

if not exist "%JAR%" (
    echo ERROR: JAR tidak ditemukan: "%JAR%"
    pause
    exit /b 1
)

"%JAVA_EXE%" -jar "%JAR%"
echo.
echo Exit Code: %ERRORLEVEL%
pause
```

**Cara menggunakannya:**

1. Simpan sebagai `run-app.bat` (Save as type: **All Files**, bukan `.bat.txt`).
2. Letakkan sejajar dengan `aplikasi.jar`.
3. Sesuaikan nilai `JAVA_EXE` dengan folder JDK di PC tersebut.
4. Double-click `run-app.bat`.

`%~dp0` menunjuk folder file BAT. `pause` mempertahankan jendela CMD agar error tidak langsung hilang.

**Batasan:** path hardcoded tidak portable. Untuk distribusi luas, pertimbangkan installer yang membundel runtime sendiri.

## 9. Masalah konfigurasi dan database setelah aplikasi berjalan

Aplikasi Java bisa **berhasil diluncurkan** tetapi tetap **gagal menjalankan login atau fitur lain**. Ini adalah lapisan masalah yang berbeda.

Contoh file konfigurasi (*bukan kredensial nyata*):

```properties
db.host=127.0.0.1
db.port=3306
db.name=example_db
db.user=example_app_user
db.password=GANTI_DENGAN_PASSWORD_SENDIRI
```

**Arti parameter:**

- `db.host`: alamat database, misalnya `localhost` untuk komputer sendiri.
- `db.port`: port server database; MySQL sering menggunakan `3306`.
- `db.name`: nama skema/database.
- `db.user`, `db.password`: identitas database, **berbeda dari akun login aplikasi**.

### Hal yang sering salah

1. **Nama file konfigurasi**: program mungkin mencari `config.properties`, tetapi berkas sebenarnya `config.properties.txt`. Yang menentukan adalah **kode pembacanya**, bukan ekstensi yang “harus” berlaku universal.
2. **Working directory**: program membaca path relatif dari folder ketika proses dijalankan, bukan selalu folder tempat JAR berada.
3. **`localhost` di komputer pengguna**: berarti database **di komputer pengguna itu**, bukan di server pengembang.
4. **MySQL belum berjalan atau salah port**.
5. **Kredensial/izin database salah**.
6. **Versi struktur database tidak cocok dengan versi aplikasi**.

### Cara membedakan error database

| Pesan | Indikasi | Pengecekan |
|---|---|---|
| `Connection refused` | Database tidak terjangkau pada host/port itu | Service database, host, port, firewall |
| `Communications link failure` | Koneksi ke database gagal/terputus | Jaringan, server DB, timeout, TLS/driver |
| `Access denied for user` | Autentikasi atau grant DB ditolak | Username, password, host, privileges |
| `Unknown database` | Database tidak ditemukan di server itu | Nama database, lokasi server, daftar skema |
| `Unknown column` | Query merujuk kolom yang tidak tersedia | Cocokkan query dan skema tabel |
| `Table ... doesn't exist` | Query merujuk tabel yang tidak tersedia | Pastikan nama tabel/skema dan migrasi |

Periksa struktur kolom tanpa mengubah data (ganti nama database):

```sql
SELECT TABLE_NAME, COLUMN_NAME, DATA_TYPE
FROM information_schema.COLUMNS
WHERE TABLE_SCHEMA = 'example_db'
ORDER BY TABLE_NAME, ORDINAL_POSITION;
```

**Jangan langsung menambah atau menghapus kolom untuk “menghilangkan error”.** Cek versi database, migration script resmi, dan SQL aplikasi terlebih dahulu. Buat backup sebelum melakukan perubahan skema/data.

**Keamanan:** jangan menaruh password nyata di repo publik. Simpan file `config.properties.example` berisi placeholder dan masukkan file rahasia yang sebenarnya ke `.gitignore`. Untuk koneksi database produksi gunakan kredensial terbatas, TLS/jaringan privat sesuai arsitektur, dan jangan membagikan user `root` tanpa password.

## 10. Mengumpulkan log dan mencari akar masalah

### 10.1 Mulai dengan perintah minimal

```cmd
java -version
where java
cd /d "C:\Apps\ContohJava"
dir /b *.jar
java -jar "aplikasi.jar"
```

Tuliskan **pesan error pertama** yang bermakna. Jangan tergesa mengatasi warning jika setelahnya ada error fatal yang lebih relevan.

### 10.2 Simpan output console ke file

```cmd
java -jar "aplikasi.jar" > startup.log 2>&1
```

- `>` mengarahkan output standar ke `startup.log`.
- `2>&1` mengarahkan pesan error ke file yang sama.
- Tunggu aplikasi selesai atau tutup aplikasinya sebelum membaca log yang lengkap.

Cek kode keluar proses jika perintah sudah selesai:

```cmd
echo Exit Code: %ERRORLEVEL%
```

Kode keluar bukan pengganti log: beberapa aplikasi GUI tidak mencetak error ke console atau memakai file logging terpisah.

### 10.3 Cek apakah Java masih berjalan

```cmd
tasklist /FI "IMAGENAME eq java.exe"
tasklist /FI "IMAGENAME eq javaw.exe"
```

Jika proses masih ada, mungkin aplikasi sedang berjalan tanpa jendela yang terlihat atau sedang macet; periksa Task Manager dan file log, bukan langsung mengasumsikan JDK rusak.

### 10.4 Urutan investigasi yang lebih efektif

```text
JAR ada dan ukurannya masuk akal?
    |
    +-- Tidak --> Perbaiki unduhan/copy file
    |
    +-- Ya
         |
         v
JAR bisa dibaca? (jar tf / tar -tf)
    |
    +-- Tidak --> Arsip rusak/tidak valid
    |
    +-- Ya
         |
         v
Java yang dipakai kompatibel?
    |
    +-- Tidak --> Pakai JDK/runtime yang sesuai
    |
    +-- Ya
         |
         v
GUI bisa muncul?
    |
    +-- Tidak --> Baca stack trace, main class, library, native/UI
    |
    +-- Ya
         |
         v
Fitur/login bisa mengakses backend/database?
    |
    +-- Tidak --> Cek konfigurasi, network, kredensial, skema
    |
    +-- Ya --> Uji fungsi aplikasi dan lakukan regression testing
```

## 11. Masalah lain yang mungkin terjadi

### `NoClassDefFoundError`, `ClassNotFoundException`

Aplikasi tidak menemukan dependency. Bisa karena developer hanya mengirim JAR utama tanpa dependency lain. Solusi developer: packaging ulang dengan dependency yang sesuai atau mendistribusikan classpath/struktur folder yang benar.

### `UnsatisfiedLinkError` / DLL tidak ditemukan

Library Java mencoba memakai native library (DLL) yang tidak ditemukan atau tidak kompatibel dengan arsitektur sistem. Cek error `Caused by`, tipe CPU x64/ARM64, dan petunjuk vendor library.

### `ExceptionInInitializerError`

Ada masalah saat sebuah class diinisialisasi. Ini sering error “pembungkus”; baca bagian **`Caused by:`** di stack trace untuk penyebab sebenarnya.

### `OutOfMemoryError`

Aplikasi kehabisan memori Java atau native. Jangan otomatis menambah `-Xmx` sebelum mengecek penggunaan memori, spesifikasi sistem, dan kemungkinan memory leak. Jika direkomendasikan developer, contoh:

```cmd
java -Xmx2g -jar "aplikasi.jar"
```

Angka `2g` hanya contoh dan membutuhkan RAM yang memadai.

### Aplikasi tampil blank, hang, atau lambat

Masalahnya bisa pada GUI event thread, proses startup, database lambat, akses jaringan, atau driver grafis. Cari log/error dan ukur kapan macet; **tidak selalu** karena Java belum ter-install.

### `Access is denied` / aplikasi diblokir Windows

Periksa izin folder dan apakah file diblokir Windows SmartScreen/antivirus. Jangan mematikan perlindungan keamanan hanya agar JAR dapat dijalankan. Pastikan file berasal dari sumber resmi dan diverifikasi.

### JAR berjalan di laptop developer, gagal di laptop user

Bandingkan versi Java, sistem operasi, arsitektur, working directory, file konfigurasi, izin filesystem, dependency, jaringan, dan database. Aplikasi yang berjalan di satu mesin **belum tentu** siap didistribusikan.

### Port database terpakai / database tidak start

Cari proses/service database yang berjalan dan port yang dikonfigurasi; jika memakai MySQL lokal, klien database harus diarahkan ke host/port yang tepat. Perubahan port perlu konsisten antara server database dan aplikasi.

## 12. Tips untuk developer dan distribusi ke pengguna

### Jika aplikasi akan dibagikan ke banyak PC

- **Tetapkan Java minimum** (misalnya Java 21) sesuai target build dan library.
- **Uji pada lingkungan bersih**, bukan hanya laptop developer.
- **Gunakan versi JAR yang jelas**, misalnya `app-1.2.0.jar`, dan berikan checksum SHA-256 agar penerima dapat memverifikasi berkas.
- **Jelaskan kebutuhan database/jaringan**, bukan sekadar mengirim JAR tanpa konfigurasi.
- **Sediakan log startup dan pesan error yang jelas**.
- **Pertimbangkan paket native dengan runtime bundel**, misalnya `jpackage` (tersedia di JDK yang mendukungnya). Ini memungkinkan user membuka aplikasi seperti aplikasi Windows tanpa mengatur PATH/JDK secara manual, tetapi perlu proses packaging, signing, dan testing.
- **Jangan simpan kredensial nyata di kode/repo**, dan jangan membundel akun database admin untuk seluruh pengguna.

### Contoh struktur distribusi sederhana

```text
ContohJava/
├── aplikasi.jar
├── config.properties.example
├── run-app.bat
└── README.md
```

Hindari memasukkan file `.properties` yang berisi password asli ke GitHub. Dokumen publik sebaiknya memakai nama program, path, dan isi konfigurasi contoh yang tidak merujuk ke infrastruktur nyata.

## 13. Checklist dan ringkasan cepat

### Checklist instalasi

- [ ] Mengetahui versi Java minimum yang dibutuhkan aplikasi.
- [ ] Download JDK dari sumber resmi dan pilih arsitektur Windows yang sesuai.
- [ ] Selesaikan instalasi MSI.
- [ ] Buka CMD baru setelah instalasi.
- [ ] Periksa `java -version` dan `where java`.
- [ ] Bila banyak Java, uji dengan path `java.exe` lengkap dahulu.
- [ ] Pastikan file JAR tidak kosong dan arsipnya valid.
- [ ] Jalankan aplikasi lewat CMD, catat output lengkap.
- [ ] Pastikan konfigurasi ditemukan dan backend/database sesuai.
- [ ] Uji login, tampilan, dan fitur utama.

### Ringkasan satu tabel

| Kalau melihat... | Pertama lakukan... |
|---|---|
| `Invalid or corrupt jarfile` | `dir "aplikasi.jar"`, `jar tf`/`tar -tf`, unduh ulang jika rusak |
| `UnsupportedClassVersionError` | `java -version`, cocokkan major version class |
| `'java' is not recognized` | Cari instalasi JDK dan atur PATH |
| `'jar' is not recognized` | Pakai `jar.exe` dari JDK atau `tar -tf` |
| `The system cannot find the path specified` | `dir` untuk cari lokasi JDK/JAR yang sebenarnya |
| GUI tidak muncul | Jalankan `java -jar` dari CMD dan baca output |
| Peringatan native access | Pastikan aplikasi berjalan; cek versi library & flag bila perlu |
| `Access denied for user` | Cek kredensial dan izin database |
| `Unknown column` | Cocokkan skema database dengan query aplikasi |
| `NoClassDefFoundError` | Cek dependensi dan hasil packaging |

---

## Referensi resmi untuk belajar lebih lanjut

- [Eclipse Adoptium — Temurin Downloads](https://adoptium.net/temurin/releases/)
- [Oracle Java Tools — `java` command](https://docs.oracle.com/en/java/javase/21/docs/specs/man/java.html)
- [Oracle Java Tools — `jar` command](https://docs.oracle.com/en/java/javase/21/docs/specs/man/jar.html)
- [Oracle Java Tools — `jpackage`](https://docs.oracle.com/en/java/javase/21/docs/specs/man/jpackage.html)

**Intinya:** saat aplikasi Java tidak mau jalan, jangan buru-buru install ulang, ganti database, atau menghapus Java lama. Pisahkan diagnosis menjadi **validitas JAR → kompatibilitas Java → PATH/lokasi executable → startup GUI/dependency → konfigurasi → koneksi database → skema database → fitur**. Dengan urutan ini, sumber masalah biasanya lebih cepat ditemukan.
