# Fedora Linux Server Recovery & Security Troubleshooting Guide

> **Tujuan dokumen:** panduan praktis untuk memulihkan server Fedora/Linux yang gagal boot, mengalami error filesystem, dicurigai terkena malware, atau perlu dibangun ulang dengan aman.
>
> Panduan ini sengaja ditulis agar bisa diikuti oleh orang yang **belum terlalu familiar dengan Linux**. Jangan sekadar copy-paste command: baca bagian **Tujuan**, **Yang harus diperiksa**, dan **Peringatan** pada setiap langkah.

---

## Daftar Isi

1. [Kapan panduan ini digunakan?](#1-kapan-panduan-ini-digunakan)
2. [Gambaran besar proses recovery](#2-gambaran-besar-proses-recovery)
3. [Istilah dasar yang perlu dipahami](#3-istilah-dasar-yang-perlu-dipahami)
4. [Cara masuk ke lingkungan troubleshooting](#4-cara-masuk-ke-lingkungan-troubleshooting)
5. [Langkah awal ketika server gagal boot](#5-langkah-awal-ketika-server-gagal-boot)
6. [Memahami error symbolic link](#6-memahami-error-symbolic-link)
7. [Kapan harus mencurigai compromise/malware](#7-kapan-harus-mencurigai-compromisemalware)
8. [Menggunakan Fedora Live USB](#8-menggunakan-fedora-live-usb)
9. [Backup sebelum melakukan tindakan berisiko](#9-backup-sebelum-melakukan-tindakan-berisiko)
10. [Memverifikasi backup dengan checksum](#10-memverifikasi-backup-dengan-checksum)
11. [Memeriksa source code sebelum direstore](#11-memeriksa-source-code-sebelum-direstore)
12. [Kapan sebaiknya fresh reinstall](#12-kapan-sebaiknya-fresh-reinstall)
13. [Langkah setelah fresh install](#13-langkah-setelah-fresh-install)
14. [Recovery MariaDB dengan metode aman](#14-recovery-mariadb-dengan-metode-aman)
15. [Restore web application](#15-restore-web-application)
16. [Apache, PHP-FPM, dan aplikasi PHP/Yii](#16-apache-php-fpm-dan-aplikasi-phpyii)
17. [Permission dan SELinux](#17-permission-dan-selinux)
18. [Troubleshooting HTTP 403 dan 500](#18-troubleshooting-http-403-dan-500)
19. [SSH setelah reinstall](#19-ssh-setelah-reinstall)
20. [Firewall dan network](#20-firewall-dan-network)
21. [Mencegah server suspend](#21-mencegah-server-suspend)
22. [Checklist verifikasi akhir](#22-checklist-verifikasi-akhir)
23. [Hardening dan pencegahan](#23-hardening-dan-pencegahan)
24. [Decision tree cepat](#24-decision-tree-cepat)
25. [Command berbahaya yang harus dipahami](#25-command-berbahaya-yang-harus-dipahami)
26. [Cheat sheet](#26-cheat-sheet)

---

# 1. Kapan panduan ini digunakan?

Panduan ini cocok digunakan ketika server Fedora/Linux mengalami salah satu kondisi berikut:

- gagal masuk ke desktop atau shell normal;
- masuk **Emergency Mode**;
- service penting gagal saat boot;
- muncul error seperti:
  ```text
  Too many levels of symbolic links
  ```
- filesystem menjadi **read-only**;
- muncul file/system configuration yang mencurigakan;
- diduga terdapat malware/rootkit;
- server perlu di-install ulang lalu aplikasi dan database dikembalikan;
- website menghasilkan HTTP `403` atau `500`;
- aplikasi PHP/Yii bermasalah setelah dipindahkan ke server baru;
- SSH tidak bisa diakses setelah reinstall;
- server tiba-tiba tidur/suspend saat idle.

---

# 2. Gambaran besar proses recovery

Untuk kasus server yang gagal boot **dan** memiliki indikator compromise, pola yang aman adalah:

```text
Identifikasi gejala
        ↓
Cari layer yang bermasalah
        ↓
Jangan langsung menghapus atau memformat
        ↓
Boot dari Live USB bila perlu
        ↓
Mount filesystem lama secara read-only
        ↓
Backup data penting
        ↓
Verifikasi backup dengan checksum
        ↓
Periksa source code
        ↓
Fresh install jika OS tidak lagi dipercaya
        ↓
Bangun ulang service dari package resmi
        ↓
Pulihkan database secara terisolasi
        ↓
Restore aplikasi secara selektif
        ↓
Perbaiki permission + SELinux
        ↓
Tes localhost → LAN → internet
        ↓
Hardening
        ↓
Dokumentasikan hasil
```

Prinsip terpenting:

> **Jika sistem operasi lama sudah tidak dapat dipercaya, selamatkan datanya — jangan membawa seluruh sistem lamanya ke server baru.**

---

# 3. Istilah dasar yang perlu dipahami

## Terminal

Aplikasi tempat kita mengetik command.

Contoh prompt:

```bash
user@server:~$
```

## Shell

Program yang membaca command kita. Salah satu shell populer adalah **Bash**.

## Root

Administrator tertinggi di Linux.

Prompt user biasa biasanya:

```text
$
```

Prompt root biasanya:

```text
#
```

Lebih aman menjalankan command administratif dengan:

```bash
sudo command
```

daripada menjadi root terus-menerus.

## Filesystem

Cara data disusun pada disk.

## Partition

Bagian logis dari sebuah disk.

Contoh:

```text
nvme0n1
├─nvme0n1p1
├─nvme0n1p2
└─nvme0n1p3
```

## Mount

Menghubungkan filesystem ke sebuah folder.

Contoh:

```bash
sudo mount /dev/nvme0n1p3 /mnt/old-system
```

## Read-only

Filesystem hanya boleh dibaca.

Ditulis sebagai:

```text
ro
```

Sedangkan read-write:

```text
rw
```

Saat recovery, **read-only lebih aman** karena sistem lama tidak berubah saat kita melakukan pemeriksaan atau backup.

## Symbolic link

Shortcut di Linux.

Normal:

```text
A → B
```

Bermasalah:

```text
A → A
```

atau:

```text
A → B → A
```

Kondisi ini dapat menghasilkan:

```text
Too many levels of symbolic links
```

---

# 4. Cara masuk ke lingkungan troubleshooting

## 4.1 Jika desktop Fedora masih bisa dibuka

1. Tekan tombol **Super/Windows**.
2. Cari:
   ```text
   Terminal
   ```
3. Buka Terminal.

Cek posisi:

```bash
pwd
```

Cek user:

```bash
whoami
```

---

## 4.2 Jika desktop tidak muncul: gunakan TTY

Coba:

```text
Ctrl + Alt + F3
```

Jika tidak muncul, coba:

```text
Ctrl + Alt + F2
Ctrl + Alt + F4
Ctrl + Alt + F5
Ctrl + Alt + F6
```

Login dengan username dan password.

> Saat mengetik password Linux, karakter biasanya **tidak terlihat sama sekali**. Itu normal.

---

## 4.3 Jika masuk Emergency Mode

Mulai dari command yang hanya membaca informasi:

```bash
pwd
```

```bash
findmnt
```

```bash
systemctl --failed
```

```bash
journalctl -xb
```

Jika tampilan `journalctl` berhenti pada `(END)`, tekan:

```text
q
```

untuk keluar.

---

# 5. Langkah awal ketika server gagal boot

Jangan langsung reinstall.

Mulai dari:

```bash
systemctl --failed
```

Lalu:

```bash
journalctl -xb
```

Cek apakah root filesystem read-only:

```bash
findmnt -no TARGET,SOURCE,FSTYPE,OPTIONS /
```

Jika terlihat:

```text
rw
```

filesystem dapat ditulis.

Jika terlihat:

```text
ro
```

jangan langsung memaksa menjadi read-write.

Periksa error storage/filesystem:

```bash
sudo dmesg -T | grep -Ei 'i/o error|read-only|btrfs|nvme|corrupt|filesystem'
```

Untuk Btrfs:

```bash
sudo btrfs device stats /
```

---

# 6. Memahami error symbolic link

Jika ada error:

```text
Too many levels of symbolic links
```

cek file/link terkait:

```bash
ls -l /path/yang-bermasalah
```

Lihat target:

```bash
readlink /path/yang-bermasalah
```

Coba selesaikan target akhir:

```bash
readlink -f /path/yang-bermasalah
```

Jika file adalah milik package RPM:

```bash
rpm -qf /path/yang-bermasalah
```

Lalu verifikasi package:

```bash
rpm -V NAMA_PACKAGE
```

Contoh bila masalah berkaitan dengan initramfs dan sistem masih dipercaya:

```bash
sudo dracut --regenerate-all --force
```

### Apa itu initramfs?

Initramfs adalah filesystem kecil yang digunakan Linux pada tahap awal boot, sebelum root filesystem utama sepenuhnya digunakan.

Karena itu bisa terjadi:

```text
file pada root filesystem benar
tetapi
copy/konfigurasi di initramfs rusak
```

### Peringatan

`dracut --force` menulis ulang initramfs. Jika terdapat indikator malware/system compromise, jangan berhenti pada “rebuild initramfs”; lanjutkan investigasi keamanan.

---

# 7. Kapan harus mencurigai compromise/malware

Beberapa indikator berisiko tinggi:

- library asing pada `/lib` atau `/usr/lib`;
- `/etc/ld.so.preload` memuat library yang tidak dikenal;
- binary sistem tidak dimiliki package resmi;
- systemd service/timer asing;
- cron job tidak dikenal;
- SSH key asing;
- process root dari `/tmp`, `/dev/shm`, atau hidden folder;
- port asing listen tanpa alasan;
- file aplikasi berubah tanpa deployment yang sah.

## `/etc/ld.so.preload`

File ini dapat memaksa shared library dimuat oleh banyak program Linux.

Cek:

```bash
if [ -e /etc/ld.so.preload ]; then
    echo "WARNING: /etc/ld.so.preload exists"
    cat /etc/ld.so.preload
else
    echo "OK: /etc/ld.so.preload tidak ada"
fi
```

Jika isinya menunjuk library yang tidak dikenal, perlakukan sebagai **high-severity indicator** sampai terbukti sebaliknya.

## Cek apakah file dimiliki package RPM

```bash
rpm -qf /path/to/suspicious-file
```

Jika muncul:

```text
file ... is not owned by any package
```

itu meningkatkan kecurigaan, khususnya jika file berada di lokasi system-level.

> Jangan menyimpulkan malware hanya berdasarkan nama file. Gunakan kombinasi lokasi, owner package, preload, process, persistence, timestamp, dan konteks insiden.

---

# 8. Menggunakan Fedora Live USB

Live USB berguna ketika:

- Fedora ter-install tidak bisa boot;
- filesystem perlu diperiksa dari luar;
- sistem lama dicurigai compromised;
- kita ingin melakukan backup tanpa menjalankan OS lama.

## 8.1 Boot dari USB

1. Buat Fedora Live USB dengan tool resmi seperti **Fedora Media Writer**.
2. Pasang USB.
3. Restart komputer.
4. Tekan tombol Boot Menu, misalnya:
   ```text
   F12 / F11 / Esc / F9
   ```
   Tombol tergantung perangkat.
5. Pilih USB.
6. Pilih mode **Try Fedora / Live**, bukan langsung install.

## 8.2 Pastikan benar-benar berada di Live USB

Buka Terminal lalu:

```bash
cat /etc/os-release
```

```bash
findmnt /
```

```bash
whoami
```

User Live biasanya berbeda dari user server production.

---

# 9. Backup sebelum melakukan tindakan berisiko

## 9.1 Identifikasi disk

```bash
lsblk -f
```

Contoh:

```text
nvme0n1
├─nvme0n1p1   vfat
├─nvme0n1p2   ext4
└─nvme0n1p3   btrfs
```

> Jangan copy nama partition dari contoh. Gunakan nama yang benar-benar muncul di server Anda.

## 9.2 Mount filesystem lama read-only

Buat folder:

```bash
sudo mkdir -p /mnt/old-system
```

Mount:

```bash
sudo mount -o ro /dev/PARTISI_YANG_BENAR /mnt/old-system
```

Verifikasi:

```bash
ls -lah /mnt/old-system
```

Jika gagal:

```bash
lsblk -f
```

dan:

```bash
sudo dmesg -T | tail -n 100
```

## 9.3 Pastikan external backup drive tersedia

```bash
lsblk -f
```

```bash
df -h
```

Pastikan ruang kosong cukup.

## 9.4 Backup folder penting dengan `tar`

Misalnya data user:

```bash
sudo tar -cpf "/path/ke/backup/user_home.tar" \
  -C /mnt/old-system/home/USERNAME .
```

Arti:

- `-c` = create archive;
- `-p` = preserve permission;
- `-f` = file output;
- `-C` = pindah ke folder source sebelum mengarsipkan.

## 9.5 Backup raw MariaDB

Jika database lama berada pada:

```text
/var/lib/mysql
```

backup:

```bash
sudo tar -cpf "/path/ke/backup/mysql_data.tar" \
  -C /mnt/old-system/var/lib mysql
```

> Raw MariaDB backup **bukan berarti boleh langsung menimpa `/var/lib/mysql` server baru**.

## 9.6 Backup konfigurasi sebagai referensi

Contoh:

```bash
sudo tar -cpf "/path/ke/backup/mysql_config.tar" \
  -C /mnt/old-system/etc my.cnf my.cnf.d
```

Tujuannya adalah referensi. Jangan otomatis mengembalikan seluruh konfigurasi OS lama.

---

# 10. Memverifikasi backup dengan checksum

Masuk folder backup:

```bash
cd "/path/ke/backup"
```

Buat checksum:

```bash
sha256sum user_home.tar mysql_data.tar mysql_config.tar > SHA256SUMS.txt
```

Verifikasi:

```bash
sha256sum -c SHA256SUMS.txt
```

Hasil sehat:

```text
user_home.tar: OK
mysql_data.tar: OK
mysql_config.tar: OK
```

Jika ada:

```text
FAILED
```

jangan format sumber data. Ulangi backup atau cek media backup.

---

# 11. Memeriksa source code sebelum direstore

Sebaiknya source code diekstrak ke area quarantine, misalnya:

```text
/var/tmp/recovery-scan/
```

## 11.1 Scan antivirus

Update signature:

```bash
sudo freshclam
```

Scan:

```bash
clamscan -r -i /var/tmp/recovery-scan
```

Perhatikan:

```text
Infected files: 0
```

Tetapi:

> `0 infected` bukan jaminan 100% source bersih.

## 11.2 Cari pola PHP yang patut ditinjau

```bash
grep -RInE \
  --include='*.php' \
  --exclude-dir=vendor \
  --exclude-dir=node_modules \
  'eval\s*\(|assert\s*\(|shell_exec\s*\(|passthru\s*\(|proc_open\s*\(|popen\s*\(|system\s*\(|exec\s*\(|gzinflate\s*\(|base64_decode\s*\(' \
  /var/tmp/recovery-scan
```

Jangan otomatis menghapus file yang muncul. Fungsi seperti `base64_decode()` atau `exec()` dapat juga dipakai oleh aplikasi sah.

## 11.3 Cari file tersembunyi/ekstensi tidak biasa

```bash
find /var/tmp/recovery-scan \
  -type f \
  \( -name ".*.php" -o -name "*.php.*" -o -name "*.phtml" -o -name "*.phar" \) \
  -not -path "*/vendor/*" \
  -not -path "*/node_modules/*"
```

## 11.4 Cari binary ELF di webroot

```bash
find /var/tmp/recovery-scan \
  -type f \
  -not -path "*/vendor/*" \
  -not -path "*/node_modules/*" \
  -exec file {} + | grep -E 'ELF .* (executable|shared object)'
```

Pada aplikasi web PHP biasa, binary ELF custom di webroot patut diperiksa.

---

# 12. Kapan sebaiknya fresh reinstall

Fresh reinstall sangat disarankan jika ditemukan indikasi:

- root access tidak sah;
- `/etc/ld.so.preload` berisi library asing;
- rootkit;
- kernel module asing;
- binary sistem dimodifikasi;
- SSH key root asing;
- persistence di banyak tempat;
- scope compromise tidak dapat ditentukan.

Menghapus satu malware file belum membuktikan:

```text
seluruh persistence hilang
```

Prinsip:

> **Rebuild operating system, restore hanya data yang memang dibutuhkan.**

---

# 13. Langkah setelah fresh install

## 13.1 Pastikan benar-benar boot dari sistem baru

```bash
cat /etc/os-release
```

```bash
findmnt /
```

```bash
lsblk -f
```

## 13.2 Update package

```bash
sudo dnf upgrade --refresh
```

## 13.3 Periksa indikator lama tidak muncul

Contoh:

```bash
test -e /etc/ld.so.preload && cat /etc/ld.so.preload || echo "No ld.so.preload"
```

Periksa juga file mencurigakan yang sebelumnya ditemukan.

## 13.4 Jangan restore seluruh `/etc`, `/usr`, atau `/lib`

Yang umumnya aman direstore setelah diperiksa:

```text
source code aplikasi
file upload bisnis
logical database dump
konfigurasi aplikasi tertentu
```

Yang tidak boleh dibawa secara buta:

```text
/etc
/usr
/lib
systemd unit lama
cron lama
SSH key lama
binary lama
```

---

# 14. Recovery MariaDB dengan metode aman

Raw datadir lama sebaiknya dibuka **terisolasi**, lalu diekspor menjadi logical SQL dump.

## 14.1 Install versi MariaDB yang kompatibel

Cari versi dari catatan/package sistem lama, lalu install versi sedekat mungkin.

Contoh:

```bash
sudo dnf install mariadb-server mariadb
```

Pada Fedora tertentu, package versioned mungkin tersedia.

## 14.2 Ekstrak raw DB ke folder recovery

```bash
sudo mkdir -p /var/lib/mysql_recovery
```

```bash
sudo tar -xpf "/path/ke/mysql_data.tar" \
  -C /var/lib/mysql_recovery
```

Jangan menimpa:

```text
/var/lib/mysql
```

fresh instance.

## 14.3 SELinux context sementara

```bash
sudo chcon -R -t mysqld_db_t /var/lib/mysql_recovery/mysql
```

## 14.4 Jalankan MariaDB recovery tanpa networking

```bash
sudo -u mysql mariadbd \
  --datadir=/var/lib/mysql_recovery/mysql \
  --socket=/var/lib/mysql_recovery/mysql/recovery.sock \
  --pid-file=/var/lib/mysql_recovery/mysql/recovery.pid \
  --skip-networking \
  --skip-grant-tables \
  --log-error=/var/lib/mysql_recovery/mysql/recovery.log &
```

Kenapa aman relatif?

```text
--skip-networking
```

mencegah instance recovery listen melalui TCP.

Sedangkan:

```text
--skip-grant-tables
```

mempermudah membuka database lama tanpa mengandalkan credential/grant lama.

> Kombinasi ini hanya untuk **recovery sementara**, bukan production.

## 14.5 Lihat database

```bash
mariadb \
  --socket=/var/lib/mysql_recovery/mysql/recovery.sock \
  -e "SHOW DATABASES;"
```

## 14.6 Buat logical dump

```bash
mariadb-dump \
  --socket=/var/lib/mysql_recovery/mysql/recovery.sock \
  --single-transaction \
  --quick \
  --routines \
  --events \
  --triggers \
  --hex-blob \
  DATABASE_1 DATABASE_2 \
  > "/path/ke/backup/application_databases.sql"
```

Jangan sertakan database system lama tanpa alasan kuat.

## 14.7 Stop recovery instance

```bash
mariadb-admin \
  --socket=/var/lib/mysql_recovery/mysql/recovery.sock \
  shutdown
```

## 14.8 Start MariaDB fresh

```bash
sudo systemctl enable --now mariadb
```

## 14.9 Import logical dump

```bash
sudo mariadb < "/path/ke/backup/application_databases.sql"
```

Verifikasi:

```bash
sudo mariadb -e "SHOW DATABASES;"
```

---

# 15. Restore web application

Gunakan `rsync`:

```bash
sudo rsync -a \
  /var/tmp/recovery-scan/ \
  /home/APP_USER/public_html/
```

Jangan tambahkan:

```text
--delete
```

kecuali benar-benar memahami dampaknya.

Set baseline owner source:

```bash
sudo chown -R APP_USER:APP_USER /home/APP_USER/public_html
```

> Ganti `APP_USER` dengan user nyata pada server Anda.

---

# 16. Apache, PHP-FPM, dan aplikasi PHP/Yii

## 16.1 Install web stack

```bash
sudo dnf install \
  httpd \
  php php-cli php-fpm php-mysqlnd php-mbstring php-intl \
  php-xml php-gd php-process php-zip composer
```

## 16.2 Aktifkan service

```bash
sudo systemctl enable --now httpd php-fpm
```

Cek:

```bash
systemctl is-active httpd php-fpm
```

Expected:

```text
active
active
```

## 16.3 Tes lokal

```bash
curl -I http://127.0.0.1
```

Tes localhost terlebih dahulu sebelum mengurus DNS/NAT.

## 16.4 Yii CLI

Masuk project:

```bash
cd /home/APP_USER/public_html/PROJECT
```

Jalankan:

```bash
php yii
```

Jika berhasil, PHP CLI dan bootstrap Yii dasar biasanya sudah bekerja.

Namun web request masih bisa gagal karena:

- Apache;
- PHP-FPM;
- SELinux;
- ownership;
- writable folder;
- DB;
- routing.

## 16.5 Production mode

Pada Yii2:

```php
defined('YII_DEBUG') or define('YII_DEBUG', false);
defined('YII_ENV') or define('YII_ENV', 'prod');
```

Jangan biarkan debug aktif di production.

---

# 17. Permission dan SELinux

Linux web server biasanya memiliki dua lapisan:

```text
Unix ownership/permission
+
SELinux
```

Keduanya harus benar.

## 17.1 Arti permission

```text
r = read
w = write
x = execute/traverse
```

Angka:

```text
7 = rwx
6 = rw-
5 = r-x
4 = r--
```

Contoh:

```text
755 = owner rwx, lainnya r-x
775 = owner/group rwx, lainnya r-x
644 = owner rw, lainnya read-only
664 = owner/group rw, lainnya read-only
```

Hindari:

```text
777
```

sebagai solusi default.

## 17.2 Folder Yii yang biasanya perlu writable

Umumnya:

```text
runtime
web/assets
web/uploads   # jika aplikasi memang menerima upload
```

Folder source seperti ini biasanya **tidak perlu writable oleh web process**:

```text
controllers
models
config
vendor
```

## 17.3 Fix owner/group

Masuk project:

```bash
cd /home/APP_USER/public_html/PROJECT
```

Cek dulu:

```bash
ls -ldZ runtime web/assets
```

Set:

```bash
sudo chown -R APP_USER:apache runtime web/assets
```

## 17.4 Permission praktis

Directory:

```bash
sudo find runtime web/assets -type d -exec chmod 2775 {} \;
```

File:

```bash
sudo find runtime web/assets -type f -exec chmod 664 {} \;
```

## 17.5 SELinux writable context sementara

```bash
sudo chcon -R -t httpd_sys_rw_content_t runtime web/assets
```

## 17.6 SELinux permanen

Install tool bila perlu:

```bash
sudo dnf install policycoreutils-python-utils
```

Tambahkan aturan:

```bash
sudo semanage fcontext -a -t httpd_sys_rw_content_t \
  "/home/APP_USER/public_html/PROJECT/runtime(/.*)?"
```

```bash
sudo semanage fcontext -a -t httpd_sys_rw_content_t \
  "/home/APP_USER/public_html/PROJECT/web/assets(/.*)?"
```

Apply:

```bash
sudo restorecon -Rv \
  /home/APP_USER/public_html/PROJECT/runtime \
  /home/APP_USER/public_html/PROJECT/web/assets
```

---

# 18. Troubleshooting HTTP 403 dan 500

## 18.1 HTTP 403

Jangan langsung `chmod 777`.

Baca log Apache:

```bash
sudo tail -n 100 /var/log/httpd/error_log
```

atau custom log VirtualHost.

Contoh masalah umum:

```text
No matching DirectoryIndex ...
```

Jika aplikasi memakai `index.php`, pastikan config Apache mengenalinya:

```apache
DirectoryIndex index.php index.html
```

Setelah edit:

```bash
sudo apachectl configtest
```

Expected:

```text
Syntax OK
```

Lalu:

```bash
sudo systemctl reload httpd
```

## 18.2 HTTP 500

Periksa log Yii:

```bash
tail -n 100 runtime/logs/app.log
```

Periksa Apache:

```bash
sudo tail -n 100 /var/log/httpd/error_log
```

Periksa folder writable:

```bash
ls -ldZ runtime web/assets
```

Error seperti:

```text
The directory is not writable by the Web process
```

berarti fokus ke:

```text
owner
group
chmod
SELinux
```

bukan langsung framework/database.

---

# 19. SSH setelah reinstall

## 19.1 Connection refused

Jika:

```text
ssh: connect to host ... port 22: Connection refused
```

cek di console server:

```bash
systemctl is-active sshd
```

Jika belum ada:

```bash
sudo dnf install -y openssh-server
```

Aktifkan:

```bash
sudo systemctl enable --now sshd
```

Cek port:

```bash
sudo ss -lntp | grep ':22'
```

## 19.2 REMOTE HOST IDENTIFICATION HAS CHANGED

Ini normal setelah reinstall karena host key baru dibuat.

**Jangan langsung hapus known_hosts.**

Cek fingerprint server baru:

```bash
sudo ssh-keygen -lf /etc/ssh/ssh_host_ed25519_key.pub
```

Bandingkan dengan fingerprint di client.

Jika cocok, pada client:

```bash
ssh-keygen -R HOSTNAME_ATAU_IP
```

Lalu connect lagi.

---

# 20. Firewall dan network

## 20.1 Firewalld

Cek:

```bash
systemctl is-active firewalld
```

Zone:

```bash
sudo firewall-cmd --get-active-zones
```

Rules:

```bash
sudo firewall-cmd --list-all
```

## 20.2 Port umum

```text
22   SSH
80   HTTP
443  HTTPS
3306 MariaDB
```

Jangan expose `3306` ke internet kecuali ada arsitektur khusus dan kontrol keamanan yang jelas.

## 20.3 Tes berlapis

Urutan diagnosis web:

```text
localhost
↓
LAN IP
↓
domain
↓
internet/mobile data
```

### Test localhost

```bash
curl -I -H "Host: app.example.com" http://127.0.0.1/
```

### Test LAN dari komputer lain

```bash
curl -I -H "Host: app.example.com" http://192.168.1.100/
```

Jika localhost berhasil tapi LAN gagal:

```text
host firewall / interface / routing LAN
```

Jika LAN berhasil tapi internet gagal:

```text
DNS / NAT / firewall perimeter / port forwarding
```

---

# 21. Mencegah server suspend

Fedora Workstation dapat suspend saat idle.

Untuk mesin yang benar-benar digunakan sebagai server always-on:

```bash
sudo systemctl mask sleep.target suspend.target hibernate.target hybrid-sleep.target
```

Verifikasi:

```bash
systemctl is-enabled sleep.target suspend.target hibernate.target hybrid-sleep.target
```

Expected:

```text
masked
masked
masked
masked
```

Untuk mengembalikan:

```bash
sudo systemctl unmask sleep.target suspend.target hibernate.target hybrid-sleep.target
```

---

# 22. Checklist verifikasi akhir

## OS

```text
[ ] Fedora boot normal
[ ] systemctl --failed tidak menunjukkan service kritis
[ ] root filesystem rw
[ ] disk terlihat normal
[ ] tidak ada I/O error aktif
```

## Security

```text
[ ] indikator compromise lama tidak muncul kembali
[ ] /etc/ld.so.preload telah diperiksa
[ ] service/timer/cron asing tidak ditemukan
[ ] port listen direview
[ ] credential penting sudah dirotasi jika diperlukan
```

## Web

```text
[ ] httpd active
[ ] php-fpm active
[ ] apachectl configtest = Syntax OK
[ ] localhost test berhasil
[ ] LAN test berhasil
[ ] public test berhasil jika service memang public
[ ] Yii production mode aktif
```

## Permission

```text
[ ] source tidak world-writable
[ ] runtime writable
[ ] web/assets writable
[ ] upload folder writable hanya bila perlu
[ ] SELinux tetap aktif
```

## Database

```text
[ ] mariadb active
[ ] database aplikasi tersedia
[ ] dedicated DB user digunakan
[ ] aplikasi tidak memakai DB root
[ ] port 3306 tidak public
[ ] logical backup tersedia
```

## SSH

```text
[ ] sshd active
[ ] fingerprint server baru diverifikasi
[ ] root SSH login disabled jika memungkinkan
[ ] key-based auth diuji
```

## Backup

```text
[ ] backup berada pada media berbeda
[ ] checksum tersedia
[ ] logical DB dump tersedia
[ ] backup tidak disimpan di webroot
[ ] test restore pernah dilakukan
```

---

# 23. Hardening dan pencegahan

## Update rutin

```bash
sudo dnf upgrade --refresh
```

## Jangan gunakan DB root dari aplikasi

Gunakan user per aplikasi/database.

## Jangan gunakan `chmod 777`

Berikan write hanya pada folder yang benar-benar membutuhkan.

## Gunakan HTTPS

Login aplikasi sebaiknya menggunakan TLS/HTTPS.

## Backup di luar DocumentRoot

Contoh:

```text
/home/APP_USER/backups
```

bukan:

```text
/home/APP_USER/public_html/backups
```

## SSH

Setelah key login terbukti bekerja:

```text
PermitRootLogin no
```

dan bila sesuai kebijakan:

```text
PasswordAuthentication no
```

Validasi:

```bash
sudo sshd -t
```

Reload:

```bash
sudo systemctl reload sshd
```

## Cek enabled service

```bash
systemctl list-unit-files --state=enabled
```

## Cek timer

```bash
systemctl list-timers --all
```

## Cek cron

```bash
crontab -l
```

```bash
sudo crontab -l
```

## Cek port

```bash
sudo ss -lntup
```

## Cek process

```bash
ps aux --sort=-%cpu | head -n 20
```

```bash
ps aux --sort=-%mem | head -n 20
```

---

# 24. Decision tree cepat

## Server tidak boot

```text
Server gagal boot
   ↓
Catat error
   ↓
Coba TTY
   ↓
Ada shell?
 ├─ Ya → systemctl --failed + journalctl -xb
 └─ Tidak → Live USB
               ↓
           lsblk -f
               ↓
        mount read-only
               ↓
             backup
```

## Website error

```text
Website down
  ↓
httpd aktif?
  ↓
php-fpm aktif?
  ↓
localhost berhasil?
  ↓
Apache log
  ↓
permission + SELinux
  ↓
DB
  ↓
Yii/app log
  ↓
baru cek compatibility/code
```

## Permission denied

```text
Permission denied
  ↓
pwd
  ↓
ls -ld target
  ↓
ls -ldZ target
  ↓
owner/group?
  ↓
chmod?
  ↓
SELinux?
```

## MariaDB gagal start

```text
MariaDB gagal
  ↓
systemctl status mariadb
  ↓
journalctl -u mariadb
  ↓
disk penuh?
  ↓
permission/SELinux?
  ↓
datadir incompatible/corrupt?
  ↓
jangan overwrite backup
  ↓
isolated recovery
  ↓
logical dump
  ↓
fresh import
```

---

# 25. Command berbahaya yang harus dipahami

## `rm -rf`

Menghapus file/folder secara recursive tanpa recycle bin.

## `mkfs`

Memformat filesystem.

## `dd`

Dapat overwrite disk byte-per-byte.

## `chown -R`

Dapat mengubah ownership ribuan file jika path salah.

## `chmod -R`

Dapat membuka permission terlalu luas.

## `mount -o remount,rw`

Jangan memaksa filesystem menjadi writable jika kernel membuatnya read-only akibat error.

## `dracut --force`

Menulis ulang initramfs.

## SQL import

```bash
mariadb < backup.sql
```

dapat membuat/mengubah banyak object database.

Sebelum command berisiko:

```text
[ ] Saya tahu sedang berada di Live USB atau installed OS?
[ ] Saya tahu path/device yang benar?
[ ] Saya sudah menjalankan pwd / ls / lsblk?
[ ] Backup tersedia?
[ ] Saya tahu cara rollback?
```

---

# 26. Cheat sheet

## Saya berada di mana?

```bash
pwd
```

## Saya login sebagai siapa?

```bash
whoami
```

## Isi folder?

```bash
ls -lah
```

## Disk?

```bash
lsblk -f
```

## Root filesystem rw atau ro?

```bash
findmnt -no TARGET,SOURCE,FSTYPE,OPTIONS /
```

## Service gagal?

```bash
systemctl --failed
```

## Status service?

```bash
systemctl status NAMA_SERVICE
```

## Log service?

```bash
sudo journalctl -u NAMA_SERVICE -b --no-pager
```

## Port?

```bash
sudo ss -lntup
```

## Apache?

```bash
systemctl is-active httpd
sudo apachectl configtest
```

## PHP-FPM?

```bash
systemctl is-active php-fpm
```

## MariaDB?

```bash
systemctl is-active mariadb
sudo mariadb -e "SHOW DATABASES;"
```

## SSH?

```bash
systemctl is-active sshd
sudo ss -lntp | grep ':22'
```

## Firewall?

```bash
systemctl is-active firewalld
sudo firewall-cmd --list-all
```

## Yii permission?

```bash
ls -ldZ runtime web/assets
```

## Yii log?

```bash
tail -n 100 runtime/logs/app.log
```

## Apache log?

```bash
sudo tail -n 100 /var/log/httpd/error_log
```

## Preload?

```bash
test -e /etc/ld.so.preload && cat /etc/ld.so.preload || echo "No ld.so.preload"
```

---

# Penutup

Tujuan troubleshooting bukan menghafal command sebanyak mungkin.

Pola yang lebih penting adalah:

```text
Apa gejalanya?
↓
Masalah ada di layer mana?
↓
Apa kata log?
↓
Apa pemeriksaan paling sederhana?
↓
Apakah perlu backup sebelum mengubah sesuatu?
↓
Ubah satu hal
↓
Tes ulang
↓
Dokumentasikan
```

Dan untuk kasus security:

> **Jika operating system sudah tidak dapat dipercaya, jangan mencoba “membersihkan sampai kelihatan bersih”. Bangun ulang OS dari sumber terpercaya, lalu restore hanya data yang telah diperiksa.**
