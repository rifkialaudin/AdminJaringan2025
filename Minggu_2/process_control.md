<h1 align=center>
    Chapter 4: Process Control
</h1>

terdiri dari dua bagian utama:

- **Address Space**: Sekumpulan halaman memori yang digunakan untuk menyimpan kode, data, dan stack dari proses.
- **Kernel Data Structures**: Struktur data dalam kernel yang menyimpan informasi tentang status proses, prioritas, sumber daya yang digunakan, dan lainnya

## Komponen dalam Proses

- **Thread**: bagian dari proses yang berbagi ruang alamat dan sumber daya yang sama. Contohnya, server web dapat memiliki banyak thread untuk menangani banyak permintaan secara bersamaan.
- **PID (Process ID)**: ID unik untuk setiap proses yang digunakan dalam berbagai sistem operasi.
- **PPID (Parent Process ID)**: ID dari proses induk yang membuat proses baru.
- **UID (User ID) & EUID (Effective User ID)**: UID menunjukkan siapa pemilik proses, sedangkan EUID menentukan hak akses proses terhadap sumber daya sistem.

## Siklus Hidup Proses 

Proses baru dibuat menggunakan sistem call fork(), yang membuat salinan dari proses induknya. Dalam sistem Linux modern, fork() dipanggil melalui clone(), yang mendukung fitur tambahan seperti thread.

Proses pertama yang dibuat oleh kernel saat sistem menyala adalah init atau systemd (PID 1), yang bertanggung jawab untuk menjalankan skrip startup dan mengelola proses lainnya.

### Sinyal

Digunakan untuk komunikasi antar proses, mengontrol proses (seperti menghentikan atau menjeda), dan memberi tahu proses tentang peristiwa tertentu. Ada sekitar 30 jenis signal, digunakan untuk berbagai tujuan

![Signals](https://liujunming.top/images/2018/12/71.png)

Beberapa sinyal penting:
- KILL: Menghentikan proses secara paksa.
- INT: Dikirim saat pengguna menekan <Control-C> untuk menginterupsi proses.
- TERM: Meminta proses untuk menghentikan eksekusi.
- HUP: Sering digunakan untuk meminta proses (seperti daemon) untuk restart.
- QUIT: Mirip dengan TERM, tetapi menghasilkan core dump jika tidak ditangkap.

**kill**: mengirim sinyal
Digunakan untuk mengirim sinyal ke proses berdasarkan PID (Process ID).

```bash
kill [-signal] pid
```
Contoh:
```bash
kill 1234  # Mengirim sinyal TERM (default) ke proses dengan PID 1234
kill -9 5678  # Mengirim sinyal KILL ke proses dengan PID 5678 (tidak bisa dicegah)
```
Penjelasan:
- kill 1234 → Mengirim sinyal TERM ke proses dengan PID 1234, meminta proses untuk berhenti dengan cara yang bersih. Namun, proses bisa menangkap dan mengabaikannya.
- kill -9 5678 → Mengirim sinyal KILL ke proses dengan PID 5678. Sinyal ini tidak bisa ditangkap atau diabaikan, sehingga proses pasti akan dihentikan secara paksa.

**killall**: Menghentikan semua proses yang memiliki nama yang sama.

Contoh:
```bash
killall firefox  # Menghentikan semua proses dengan nama "firefox"
```

**pkill**: Mirip dengan killall, tetapi lebih fleksibel karena mendukung pemfilteran berdasarkan pola atau pengguna.

```bash
pkill -u abdoufermat  # Menghentikan semua proses yang dijalankan oleh user 'abdoufermat'
```

## Memantau Monitoring dengan PS

Perintah **ps** digunakan untuk memantau proses. Opsi aux memberikan informasi detail tentang semua proses.

```bash
ps aux | grep firefox  # Mencari proses Firefox
pgrep firefox  # Menampilkan PID proses Firefox
pidof /usr/bin/firefox  # Menampilkan PID berdasarkan path executable
```

**ps lax** menampilkan informasi teknis tentang proses yang sedang berjalan di sistem. Mode **lax** lebih cepat dibandingkan **aux** karena tidak perlu menyelesaikan nama pengguna dan grup.  

Untuk mencari proses tertentu, bisa menggunakan **grep** pada output dari **ps aux**, misalnya untuk mencari proses **firefox**:  

```bash
$ ps aux | grep -v grep | grep firefox
```

Untuk mengetahui **PID** dari suatu proses, dapat digunakan perintah **pgrep** atau **pidof**:  

```bash
$ pgrep firefox
$ pidof /usr/bin/firefox
```  

## Pemantauan Interaktif menggunakan Interactive monitoring with top

Perintah **top** digunakan untuk melihat proses yang sedang berjalan di sistem Linux secara real-time. Perintah ini menampilkan informasi seperti penggunaan CPU, memori, dan daftar proses yang aktif. Tampilan **top** diperbarui otomatis setiap 1-2 detik.  

Selain itu, ada **htop**, yang fungsinya mirip dengan **top**, tetapi lebih mudah digunakan karena memiliki tampilan yang lebih interaktif. Dengan **htop**, pengguna bisa menggulir ke atas, bawah, kiri, dan kanan untuk melihat semua proses serta perintah lengkapnya.

## Nice and renice: Mengubah Prioritas Proses

Niceness adalah angka yang digunakan oleh kernel untuk menentukan prioritas suatu proses dibandingkan dengan proses lain yang bersaing untuk CPU.

Nilai niceness tinggi (+19) = Prioritas rendah (proses kurang penting).
Nilai niceness rendah (-20) = Prioritas tinggi (proses lebih penting).
Di Linux, rentang nilai niceness adalah -20 hingga +19.

**nice** → Menjalankan proses dengan prioritas tertentu
```bash
nice -n 10 sleep 10 &
```

**renice** → Mengubah prioritas proses yang sedang berjalan.
```bash
renice -n -5 -p 215
```

Hubungan antara niceness dan priority:
priority_value = 20 + nice_value

Secara default, nice value adalah 0. Semakin kecil angkanya, semakin tinggi prioritas proses tersebut.

## The /proc filesystem

perintah **ps** dan **top** mendapatkan informasi status proses dari direktori /proc. Direktori ini merupakan pseudo-filesystem yang digunakan oleh kernel untuk menyajikan berbagai informasi tentang keadaan sistem.

tidak hanya berisi informasi tentang proses yang berjalan, tetapi juga berbagai statistik sistem.

### Struktur Direktori /proc
Setiap proses yang berjalan memiliki direktori di /proc yang dinamai sesuai dengan PID (Process ID) dari proses tersebut. Di dalam direktori ini, terdapat berbagai file yang menyimpan informasi tentang proses tersebut, seperti:
- cmdline → Menyimpan perintah yang digunakan untuk menjalankan proses.
- environ → Berisi variabel lingkungan dari proses.
- fd/ → Menyimpan daftar file descriptor yang digunakan oleh proses.

Direktori /proc sangat berguna untuk memantau dan menganalisis proses serta performa sistem secara langsung dari kernel.

## Strace and truss

Untuk mengetahui apa yang dilakukan sebuah proses, kita dapat menggunakan perintah strace di Linux atau truss di FreeBSD. Perintah ini digunakan untuk melacak system calls dan sinyal yang dilakukan oleh suatu proses.

### Fungsi Strace dan Truss
- Debugging → Membantu menemukan kesalahan dalam program.
- Analisis Perilaku Program → Memahami bagaimana suatu program berinteraksi dengan sistem.

### Contoh:
Dengan menjalankan,
```bash
strace -p [PID]
```
kita bisa melihat aktivitas sistem dari proses yang sedang berjalan. Misalnya, ketika menjalankan strace terhadap proses top, hasil yang diperoleh menunjukkan bahwa:

1. top memeriksa waktu saat ini.
2. Membuka dan membaca isi direktori /proc.
3. Mengakses file /proc/1/stat untuk mendapatkan informasi tentang proses init.

Dengan strace atau truss, kita bisa melihat bagaimana suatu program bekerja dan berinteraksi dengan sistem secara lebih mendetail.

## Proses Runaway

Terkadang, sebuah proses bisa berhenti merespons dan terus berjalan tanpa terkendali. Proses ini disebut runaway process dan dapat menyebabkan CPU bekerja 100%, sehingga sistem menjadi lambat.

Untuk menghentikan proses yang tidak merespons, kita dapat menggunakan perintah kill dengan sinyal yang sesuai:

SIGTERM (15) → Sinyal standar untuk meminta proses berhenti dengan baik.
SIGKILL (9) → Digunakan jika proses tidak merespons SIGTERM.

```bash
kill -9 pid

or

kill -KILL pid
```
### Menganalisis Penyebab Proses Runaway
1. Menggunakan strace atau truss → Untuk melihat sistem panggilan (syscalls) yang dilakukan oleh proses dan memahami mengapa proses tersebut berjalan tanpa henti.

2. Memeriksa Penggunaan Filesystem
- Jalankan df -h untuk mengecek apakah sistem penyimpanan penuh.
- Jika penuh, gunakan du untuk menemukan file atau direktori terbesar.

3. Melihat File yang Dibuka oleh Proses
- Jalankan lsof -p PID untuk mengetahui file mana yang sedang digunakan oleh proses runaway.

Dengan teknik ini, kita bisa menghentikan dan menganalisis penyebab runaway process, serta mencegah dampak buruk terhadap sistem.

## Proses Berkala

### cron: Menjadwalkan Perintah

cron adalah daemon yang digunakan untuk menjalankan perintah secara terjadwal. Program ini mulai berjalan saat sistem dinyalakan dan terus berjalan selama sistem aktif.

### Cara Kerja cron
1. Membaca File Konfigurasi
- cron membaca daftar perintah dan waktu eksekusi yang sudah ditentukan dalam file konfigurasi.
- Perintah yang dijalankan oleh cron dieksekusi menggunakan sh, sehingga semua perintah yang bisa dilakukan di terminal juga bisa dijalankan melalui cron.
2. File Konfigurasi (Crontab)
- Crontab adalah file tempat menyimpan daftar tugas yang akan dijalankan oleh cron.
- Lokasi penyimpanan crontab berbeda tergantung pada sistem operasi:
  - Linux: /var/spool/cron
  - FreeBSD: /var/cron/tabs

Dengan menggunakan cron, kita bisa mengotomatisasi berbagai tugas seperti backup data, pembaruan sistem, atau menjalankan skrip tertentu pada waktu yang ditentukan.

### Format Crontab

Crontab adalah file konfigurasi yang digunakan oleh cron untuk menjadwalkan eksekusi perintah secara otomatis pada waktu tertentu. Format crontab terdiri dari lima kolom utama yang menentukan waktu eksekusi, diikuti dengan perintah yang akan dijalankan.

**crontab management**

Crontab dikelola menggunakan perintah crontab, yang memungkinkan pengguna untuk membuat, mengedit, menampilkan, dan menghapus jadwal tugas.

- Mengedit crontab
```bash
crontab -e
```
Membuka editor untuk mengedit file crontab pengguna.

- Menampilkan crontab yang sedang aktif
```bash
crontab -l
```
Menampilkan daftar tugas yang telah dijadwalkan.

- Menghapus crontab pengguna
```bash
crontab -r
```
Menghapus semua jadwal yang tersimpan di crontab.

### Systemd Timer

Systemd Timer adalah unit konfigurasi dalam systemd yang digunakan untuk menjadwalkan eksekusi tugas secara otomatis. File konfigurasi untuk timer ini memiliki ekstensi .timer dan dapat digunakan sebagai alternatif cron jobs, dengan fitur yang lebih fleksibel dan lebih kuat.

### Cara Kerja Systemd Timer

Systemd timer bekerja dengan memicu service unit pada waktu tertentu yang telah ditentukan dalam file konfigurasi timer. Timer ini dapat dijalankan berdasarkan:
- Jadwal waktu tertentu
- Saat sistem melakukan booting
- Ketika terjadi suatu event dalam sistem

Untuk melihat daftar timer yang sedang aktif, gunakan perintah berikut:

```bash
systemctl list-timers
```
Perintah ini akan menampilkan daftar timer yang telah dikonfigurasi dalam sistem.

### Contoh Penggunaan Systemd Timer

1. Mengirim Email Secara Otomatis
Systemd timer atau cron dapat digunakan untuk mengirim email secara otomatis, misalnya untuk mengirim laporan bulanan kepada admin setiap tanggal 20 pukul 05:30 pagi.
```bash
30 5 20 * * /usr/bin/mail -s "Laporan Bulanan" admin@example.com < /path/to/report.txt
```
Dengan perintah ini, email akan dikirim secara otomatis setiap bulan pada tanggal 25 pukul 04:30 pagi.

2. Membersihkan File Sampah (Cleanup Filesystem)
 
Untuk menghemat ruang penyimpanan, sistem dapat secara otomatis menghapus file sampah yang sudah lebih dari 30 hari di direktori Trash setiap tengah malam.
```bash
0 0 * * * /usr/bin/find /home/user/.local/share/Trash/files -mtime +30 -exec /bin/rm -f {} \;
```
Dengan script ini, file yang lebih dari 30 hari dalam folder Trash akan dihapus secara otomatis.

3. Rotasi File Log (Log Rotation)

Rotasi log adalah proses membagi file log menjadi segmen berdasarkan ukuran atau tanggal, serta menyimpan beberapa versi lama untuk referensi. Hal ini membantu menghindari akumulasi file log yang terlalu besar.

Systemd timer dapat digunakan untuk memutar log secara otomatis setiap hari dengan konfigurasi seperti:
```bash
[Timer]
OnCalendar=daily
```
Dengan pengaturan ini, sistem akan memastikan bahwa file log tidak tumbuh tanpa batas dan tetap terkelola dengan baik.

4. Menjalankan Batch Job Secara Otomatis

Beberapa tugas seperti memproses antrian pesan atau memindahkan data antar sistem lebih baik dijalankan sebagai batch job. Misalnya, pesan yang menumpuk dalam antrian dapat diproses sekaligus menggunakan tugas terjadwal sebagai bagian dari proses ETL (Extract, Transform, Load).

5. Backup dan Mirroring Secara Berkala

Backup adalah proses menyimpan salinan data ke sistem lain untuk keamanan, sedangkan mirroring adalah metode menduplikasi file sistem secara otomatis ke server lain.

Dengan menggunakan rsync, kita bisa menjadwalkan sinkronisasi file untuk menjaga mirror tetap up-to-date:
```bash
0 2 * * * rsync -avz /data/ user@backupserver:/backup/
```
Perintah ini akan menjalankan backup setiap hari pukul 02:00 pagi ke server lain.
