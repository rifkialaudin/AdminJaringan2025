<h1 align=center>
  Chapter 5: Filesystem
</h1>

![filesystem-icon](https://miro.medium.com/v2/resize:fit:752/1*quw0WvsLLCxad3WC6fjQ1Q.png)  

Filesystem adalah sistem yang digunakan untuk merepresentasikan dan mengorganisir sumber daya penyimpanan dalam sebuah sistem.  

Filesystem memiliki **empat komponen utama**, yaitu:  
1. **Namespace** – Menyediakan cara untuk memberi nama dan mengorganisir data dalam bentuk hierarki.  
2. **API** – Kumpulan sistem panggilan (system calls) yang digunakan untuk menavigasi dan memanipulasi data.  
3. **Model Keamanan** – Mekanisme untuk melindungi, menyembunyikan, dan membagikan data sesuai dengan aturan tertentu.  
4. **Implementasi** – Perangkat lunak yang menghubungkan model logis filesystem dengan perangkat keras.  

Filesystem yang paling umum digunakan untuk penyimpanan berbasis disk meliputi **ext4, XFS, dan UFS**, serta **ZFS dan Btrfs** dari Oracle. Selain itu, ada juga filesystem lainnya seperti **VxFS dari Veritas dan JFS dari IBM**.  

Selain filesystem utama, ada **filesystem asing** seperti **FAT dan NTFS** yang digunakan oleh Windows, serta **ISO 9660** yang digunakan untuk CD dan DVD.  

Filesystem modern umumnya berusaha meningkatkan kecepatan dan keandalan dibandingkan filesystem tradisional, atau menambahkan fitur tambahan di atas fungsi standar filesystem.

## Pathnames

Dalam konteks teknis, istilah "folder" sebenarnya berasal dari Windows dan macOS, tetapi maknanya sama dengan "directory". Dalam dunia teknis, lebih baik menggunakan istilah directory karena lebih umum dan tepat digunakan dalam lingkungan berbasis UNIX/Linux.

### Pathname (Nama Jalur) dalam Filesystem
Pathname adalah daftar direktori yang menunjukkan lokasi sebuah file dalam hierarki filesystem.

Pathname terbagi menjadi dua jenis:
1. Absolute Pathname – Menunjukkan lokasi file atau direktori mulai dari root (/), sehingga selalu memberikan jalur yang pasti. <br>
Contoh:
```bash
/home/username/file.txt
```
2. Relative Pathname – Menunjukkan lokasi file atau direktori relatif terhadap posisi saat ini (current directory). <br>
Contoh:
```bash
./file.txt
```
Tanda . menunjukkan direktori saat ini, sedangkan .. digunakan untuk merujuk ke direktori induk (parent directory).

## Filesystem, Mount, dan Unmount

### Filesystem dan File Tree
Filesystem terdiri dari bagian-bagian kecil yang masing-masing memiliki satu direktori utama beserta subdirektori dan file di dalamnya.
- File tree adalah struktur keseluruhan dari filesystem.
- Filesystem mengacu pada cabang-cabang yang melekat pada file tree.

### Mount dan Unmount Filesystem
Filesystem dapat dihubungkan ke file tree menggunakan perintah mount, dengan direktori tujuan disebut mount point.

Contoh:
```bash
mount /dev/sda4 /users
```
Perintah di atas memasang filesystem dari /dev/sda4 ke direktori /users.

Untuk melepaskan filesystem, gunakan perintah umount:
- **umount -l** (lazy unmount) – Melepas filesystem dari hierarki nama tetapi tetap aktif sampai tidak digunakan lagi.
- **umount -f** (forceful unmount) – Memaksa unmount meskipun filesystem sedang digunakan.

### Mengetahui Proses yang Menggunakan Filesystem
Daripada menggunakan **umount -f**, lebih baik mencari tahu proses mana yang masih menggunakan filesystem dengan perintah berikut:
1. Menggunakan lsof untuk melihat proses yang menggunakan filesystem:
```bash
lsof /home/abdou
```

2. Menggunakan ps untuk mendapatkan informasi detail tentang proses tersebut:
```bash
ps up "1234 5678 91011"
```
Dengan mengetahui proses yang masih aktif, kita bisa menutupnya terlebih dahulu sebelum melakukan unmount untuk menghindari error atau potensi kehilangan data.

## Organisasi File Tree dalam Sistem UNIX

Sistem UNIX memiliki struktur file tree yang kurang terorganisir dengan baik karena adanya berbagai konvensi penamaan yang tidak konsisten. Hal ini membuat proses upgrade sistem operasi menjadi sulit.

### Struktur Root Filesystem
Root filesystem mencakup direktori root (/) beserta beberapa file dan subdirektori penting, seperti:
- /boot → Menyimpan file kernel OS (lokasi dan nama dapat bervariasi).
- /etc → Berisi file konfigurasi dan sistem yang krusial.
- /sbin dan /bin → Menyimpan utilitas penting untuk sistem.
- /tmp → Direktori untuk menyimpan file sementara.
- /dev → Dahulu merupakan bagian dari root filesystem, tetapi sekarang merupakan filesystem virtual yang dimount secara terpisah.

### Struktur Direktori Tambahan
- /lib atau /lib64 → Berisi shared library dan beberapa komponen lainnya. Pada beberapa sistem, file-file ini dipindahkan ke /usr/lib, sedangkan /lib hanya menjadi symbolic link.
- /usr → Menyimpan sebagian besar program non-kritis untuk sistem, manual online, dan pustaka tambahan. FreeBSD juga menyimpan konfigurasi lokal di /usr/local.
- /var → Menyimpan log sistem, informasi akuntansi, spool direktori, dan file lain yang sering berubah atau bertambah seiring waktu.

Baik /usr maupun /var harus tersedia agar sistem bisa berjalan hingga mode multiuser.

## File types

Most filesystem implementations define seven types of files:

Sistem UNIX mendefinisikan tujuh jenis file dalam filesystem:

1. Regular files → Berisi kumpulan byte tanpa struktur tertentu, termasuk file teks, data, program eksekusi, dan pustaka bersama.
2. Directories → Referensi yang menunjuk ke file lain.
3. Character device files → Digunakan untuk komunikasi dengan perangkat keras berbasis karakter (misalnya keyboard, terminal).
4. Block device files → Digunakan untuk perangkat berbasis blok (misalnya hard disk, USB).
5. Local domain sockets → Digunakan untuk komunikasi antarproses dalam satu host, mirip dengan network sockets tetapi terbatas pada sistem lokal.
6. Named pipes (FIFO) → Seperti local domain sockets, tetapi lebih sederhana dan digunakan untuk komunikasi antarproses dalam satu sistem.
7. Symbolic links (Soft links) → Referensi ke file lain berdasarkan nama, lebih fleksibel daripada hard link karena dapat menunjuk ke file di filesystem yang berbeda dan ke direktori.

**Menentukan tipe file**
Gunakan perintah file untuk mengetahui tipe file:

```bash
$ file /bin/bash
```
Atau gunakan ls -ld untuk menampilkan informasi direktori tanpa menampilkan isinya

**Hard Links**
Hard link memungkinkan satu file memiliki beberapa nama. Hard link dibuat dengan perintah:
```bash
$ ln /etc/passwd /tmp/passwd
```
Gunakan ls -i untuk melihat jumlah hard link dalam suatu file.

**Character dan Block Device Files**

File perangkat digunakan untuk komunikasi antara program dan perangkat keras melalui driver sistem operasi.
- Character device files menangani data satu karakter dalam satu waktu.
- Block device files menangani data dalam blok.

Setiap perangkat memiliki major number (menunjukkan driver yang mengontrolnya) dan minor number (menunjukkan unit spesifik). Misalnya, /dev/tty0 adalah terminal pertama dengan major device number 4 dan minor device number 0.

**Local Domain Sockets dan Named Pipes**

- Local domain sockets digunakan untuk komunikasi antarproses dalam satu sistem, misalnya oleh Syslog dan X Window System.
- Named pipes (FIFO) juga digunakan untuk komunikasi antarproses dalam satu host tetapi lebih sederhana dibandingkan local domain sockets.

**Symbolic Links**

Symbolic link adalah referensi ke file berdasarkan nama, lebih fleksibel dibandingkan hard link karena dapat menunjuk ke file di filesystem lain atau ke direktori.

Contoh symbolic link:
```bash
$ ln -s /bin /usr/bin
$ ls -l /usr/bin
lrwxrwxrwx 1 root root 4 Mar  1  2020 /usr/bin -> /bin
```
Symbolic link sering digunakan untuk menyederhanakan struktur direktori dan mempermudah manajemen sistem.

## File attributes

Di sistem Unix dan Linux, setiap file memiliki 12 bit mode yang menentukan izin akses (read, write, execute) untuk pemilik, grup, dan pengguna lain. Selain itu, terdapat 4 bit tambahan untuk menentukan jenis file. Izin ini dapat dimodifikasi menggunakan perintah chmod oleh pemilik file atau superuser.

![file-attributes](https://cdn.storyasset.link/nlFtWFR5rySdmletT0jhDUQ0tXl2/ms-yxhcfoletf.jpg)

### 1. Permission Bits (Bit Izin)
Izin file dibagi menjadi tiga kelompok:
- u (owner/pemilik) → Pemilik file-
- g (group/grup) → Grup yang memiliki file
- o (others/lainnya) → Pengguna lain

Masing-masing memiliki tiga izin:
- r (read) → Bisa membaca file-
- w (write) → Bisa mengedit atau menghapus isi file
- x (execute) → Bisa mengeksekusi file jika berbentuk program

Izin ini juga bisa dinyatakan dalam format oktal:
- 4 (r) → Read
- 2 (w) → Write
- 1 (x) → Execute

### 2. Setuid dan Setgid
- Setuid (4000) → Jika diaktifkan pada file eksekusi, file tersebut akan berjalan dengan hak akses pemiliknya.
- Setgid (2000) → Jika diaktifkan pada file eksekusi, file tersebut akan berjalan dengan hak akses grupnya.
Jika diterapkan pada direktori, Setgid memastikan semua file yang dibuat di dalamnya mewarisi grup dari direktori.

### 3. Sticky Bit (1000)
- Diterapkan pada direktori untuk mencegah pengguna lain menghapus atau mengganti nama file yang bukan miliknya.
- Umum digunakan di direktori /tmp untuk melindungi file pengguna lain.

### 4. ls: Melihat Atribut File
- Perintah ls -l menampilkan informasi lengkap tentang file, termasuk izin akses, pemilik, ukuran, dan waktu modifikasi.

### 5. chmod: Mengubah Izin File
- Menggunakan format oktal atau simbolik.
```bash
chmod u+w file.txt       # Menambahkan izin write untuk pemilik
chmod ug=rw,o=r file.txt # Pemilik dan grup bisa baca/tulis, lainnya hanya baca
chmod a-x file.sh        # Menghapus izin eksekusi untuk semua pengguna
```

### 6. chown: Mengubah Kepemilikan File
- Mengganti pemilik dan grup file
```bash
chown -R user:group /home/user
```
### 7. chgrp: Mengubah Grup File
- Sama seperti chown, tapi hanya mengubah grup file
```bash
chgrp -R users /home/user
```

### 8. umask: Mengatur Izin Default
- Menentukan izin default saat file/direktori baru dibuat.
- Nilai umask dikurangi dari izin default sistem:
```bash
umask 022
```

## Access Control Lists
Access Control Lists (ACLs) adalah fitur yang memperluas model izin tradisional Unix. ACL memungkinkan sebuah file memiliki banyak pemilik dan memberikan hak akses berbeda ke berbagai pengguna atau grup.

Setiap aturan dalam ACL disebut Access Control Entry (ACE), yang terdiri dari:
- User atau grup (bisa berupa nama pengguna, nama grup, atau kata kunci khusus seperti owner dan other).
- Permission mask (hak akses seperti read, write, execute).
- Tipe (allow atau deny).

### Perintah Dasar ACL

- Melihat ACL suatu file:
```bash
getfacl /etc/passwd
```
- Menetapkan ACL suatu file:
```bash
setfacl -m u:abdou:rw /etc/passwd
```

### Jenis ACL
Terdapat dua jenis ACL utama:
- POSIX ACLs – Model ACL tradisional yang digunakan pada Unix/Linux.
- NFSv4 ACLs – Model ACL yang lebih fleksibel dan mendukung fitur tambahan.

### POSIX ACLs
POSIX ACLs merupakan model tradisional yang digunakan dalam sistem operasi berbasis Unix, seperti Linux, FreeBSD, dan Solaris.


| Format	                | Contoh	        | Menetapkan izin untuk     |
|-------------------------|-----------------|---------------------------|
| uer::perms		          |user:rw-         | Pemilik file              |
| user:username:perms	    | user:abdou:rw-	| Pengguna tertentu         |
| group::perms		        | group:r-x       | Grup pemilik file         |
| group:groupname:perms	  | group:users:r-x	| Grup tertentu             |
| mask::perms		          | mask::rwx       | Izin maksimal yang berlaku|
| other::perms	          | other::r--	    | Pengguna lain             |

### NFSv4 ACLs

NFSv4 ACLs adalah versi yang lebih canggih dibandingkan POSIX ACLs, dengan fitur tambahan seperti default ACL, yang memungkinkan pengaturan hak akses otomatis untuk file dan direktori yang baru dibuat. NFSv4 ACLs banyak digunakan dalam sistem berbasis jaringan seperti NFS (Network File System).

Keunggulan NFSv4 ACLs
- Memiliki lebih banyak tipe izin yang lebih fleksibel.
- Mendukung daftar izin bawaan (default ACL).
- Cocok untuk lingkungan jaringan yang kompleks.
