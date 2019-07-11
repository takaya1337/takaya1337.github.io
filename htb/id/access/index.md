---
title: Access
layout: default
---

# Access

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/01/access.png">
</p>

#### OS    : Windows
#### IP    : 10.10.10.98
#### Maker : egre55
* * *
<br>
<br>
<br>
<br>

## Pendahuluan
<br>

Sebelum anda membaca hal-hal teknis, ada baiknya anda membaca sedikit kata pembuka dari saya terlebih dahulu. Seperti anda ketahui, saya membuat write-up dalam dua bahasa, Inggris dan Indonesia. Perlu saya ingatkan di awal bahwa write-up saya tidak akan sama persis antara dua bahasa tersebut karena saya tidak menerjemahkan mentah-mentah melainkan saya _edit_ ulang sesuai dengan pemikiran saya sebagai orang Indonesia.
* Mengapa saya mau repot-repot membuat write-up dalam dua bahasa?

Anggap saja sebagai bentuk pengabdian kepada bangsa. Saya ingin meningkatkan kemampuan _"heking"_ orang-orang yang ingin belajar tapi belum mampu mengerti write-up kelas atas (yang kebanyakan berbahasa Inggris) agar tidak seperti salah seorang pentester yang cukup dikenal karena "kehebatannya".
* Mengapa HTB?

Karena saya bosan main CTF terus dan saya juga berpikir bahwa sudah banyak write-up CTF berbahasa Indonesia yang cukup bagus (baca: [petircysec.com](https://petircysec.com)), jadi saya ingin membuat sesuatu yang relatif baru.

* Bagaimana cara mainnya?

HTB atau _Hack the Box_ adalah sebuah permainan dimana anda bisa memilih beberapa list _IP address_ "mesin-mesin" yang pasti memiliki kelemahan tertentu.

Dari kelemahan tersebut, anda harus mendapatkan dua buah hash atau flag seperti "7f3ff9f556f5f470e41508ff970c794e". Ada dua flag dalam tiap mesin, yaitu **user** dan **root**.

Flag user anda dapatkan ketika anda mendapat akses pertama ke mesin dengan _privilege_ seadanya (user biasa). Flag root akan anda dapatkan ketika anda berhasil melakukan _privilege escalation_ (atau EOP dalam bahasa korporat) dimana anda mendapatkan akses sebagai user tertinggi (root atau Administrator).

Secara garis besar, tiap write-up akan saya bagi menjadi dua bab utama: User dan Root, dimana masing-masing bab akan memiliki penjelasan sendiri sesuai dengan kelemahan yang ada dalam mesin tersebut.

Sekian untuk pendahuluan.

## User
<br>

### NMAP
Hal pertama yang perlu dilakukan ketika menemukan sebuah alamat IP adalah meng-`nmap` IP tersebut. Hal ini diperlukan agar kita dapat melihat gambaran mesin secara keseluruhan sehingga dapat menentukan serangan apa yang cocok. Sintaks favorit saya adalah:

```
$ nmap -p 1-65535 -T4 -A -v 10.10.10.98
```
> `-p 1-65535` akan memeriksa semua _port_
>
> `-T 4` akan melakukan _scan_ dengan _Timing-level_ 4 (**Aggresive** -- artinya mempercepat scan tetapi tidak terlalu cepat hingga hasil scan tidak akurat)
>
> `-A` akan memeriksa OS, versi, melakukan scan menggunakan _script_ dan _traceroute_
>
> `-v` atau _verbosity-level_ 1 membuat `nmap` akan sedikit lebih detail dalam memberikan hasil.

<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/01/01-nmap.png">
</p>

Dapat kita lihat bahwa mesin tersebut memiliki beberapa _service_ yang terbuka, yaitu: **FTP server, Telnet, dan HTTP server (Microsoft IIS 7.5)**. Mari kita cek yang paling pertama.
<br>
<br>
<br>

### FTP
FTP atau _File Transfer Protocol_ adalah sebuah protokol atau **cara** untuk memindahkan file secara online. Anda bisa mencari info lebih lanjut tentang FTP di Internet (seperti perbedaan FTP dan FTPS serta mengapa anda tidak disarankan untuk menggunakan FTP untuk keperluan sehari-hari) karena saya hanya akan membahas yang berhubungan dengan mesin ini saja.

Untuk mengakses FTP anda dapat menggunakan FTP client (baik CLI maupun GUI).
<br>

<p align="center">                                                        
<img src="https://takaya1337.github.io/htb/assets/01/02-ftp.png">         
</p>

Setelah anda berhasil masuk, anda akan menemukan dua folder yang mencurigakan: "Backup" dan "Engineer". Anda bisa men-download kedua file tersebut.
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/01/03-ftperror.png">
</p>

Anda bisa menggunakan _command-command_ FTP untuk berinteraksi dengan file yang ada di FTP server. Untuk men-download misalnya, anda dapat menggunakan `get`.

Bila anda mendapatkan error seperti gambar diatas dan anda melakukan sedikit _research_, anda akan tahu bahwa error tersebut disebabkan oleh tipe **file transfer** yang digunakan oleh FTP, yaitu **ASCII mode** dan **Binary mode**.

**ASCII mode** akan mengubah _Line Feed_ yang dimiliki file bila transfer dilakukan ke mesin Windows dari Linux atau sebaliknya. Line Feed atau LF adalah sebuah ASCII karakter yang biasa direpresentasikan dengan bentuk "\n" atau yang lebih sering disebut "enter" oleh orang Indonesia. 

Secara konvensi, Windows dan Linux memiliki Line Feed yang berbeda, dimana Windows menerapkan CRLF atau _Carriage Return Line Feed_: Carriage Return akan mengembalikan posisi cursor paling kiri secara horizontal (bayangkan mesin tik jaman dulu) setelah itu di"enter" oleh Line Feed; dan Linux hanya memakai LF saja.

ASCII mode akan mengganti CRLF dengan LF dan sebaliknya ketika transfer terjadi antara dua sistem tersebut.

**Binary mode** tidak mengubah file sama sekali dan melakukan transfer apa adanya.

Secara default, yang digunakan oleh FTP adalah ASCII mode. Jadi kita harus mengganti tipe transfer menjadi binary untuk mencegah error tadi.
```
ftp> bin
ftp> get [file tadi]
```
<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/01/06-ftpbin.png">
</p>

Ada cara yang lebih efisien bila anda lebih menyukai _one-liner_ dengan `wget`.
```
$ wget --no-passive-ftp -r ftp://anonymous:anonymous@10.10.10.98//
```
> `--no-passive-ftp` akan men-disable Passive mode FTP (singkatnya dalam Passive mode FTP, server tidak membuat koneksi ke client dan file transfer hanya satu arah dari client ke server). Biasanya argumen ini dipakai bila **tidak ada** _client-side firewall_.
>
> `-r` mengindikasikan bahwa kita ingin mengambil file secara _recursive_, artinya bila ada folder di dalam folder, kita akan mengambil file sampai ke dalam folder terakhir. Secara default, `-r` hanya mengambil sampai maksimum 5 file ke dalam. Jika anda ingin men-download file yang memiliki banyak subfolder anda harus mengganti angkanya sesuai kebutuhan.

<br>
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/01/07-wgetway.png">
</p>

Agar lebih yakin bahwa cara FTP dan `wget` berjalan lancar, kita dapat membandingkan file-nya.
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/01/08-compare.png">
</p>

Mantap, file-nya oke. Mari kita cek file tersebut lebih lanjut.
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/01/09-filecheck.png">
</p>

Sepertinya file-file tersebut akan sangat bergantung pada Windows. Untungnya, tidak sulit bagi orang Indonesia untuk mendapatkan akses ke mesin Windows :)

Tapi sebelumnya mari kita lihat Telnet terlebih dahulu.
<br>
<br>
<br>

### Telnet
Telnet atau _Teletype Network_ adalah sebuah protokol atau **cara** untuk mengakses sebuah _remote machine_. Bila anda familiar dengan SSH, Telnet adalah nenek moyangnya yang sedikit lebih berbahaya. Fungsinya sama seperti SSH, yaitu untuk memberikan akses interaktif dengan sebuah mesin lewat internet.

Untuk _connect_ ke Telnet mesin ini, anda dapat menggunakan `telnet` command di Linux.
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/01/10-telnetcheck.png">
</p>

Kita belum mendapatkan _credential_ yang diperlukan untuk mengakses Telnet. Anda bisa menyiapkan mesin Windows anda untuk langkah selanjutnya.
<br>
<br>
<br>

### The files
Sejauh ini kita memiliki dua file dari dua folder berbeda:

- #### Backups
    - backup.mdb
<br>
<br>

- #### Engineer
    - Access Control.zip
<br>

File pertama merupakan sebuah database untuk **Microsoft Access** dan file kedua adalah sebuah zip yang dilindungi _password_.

Akan sangat sulit untuk mencoba _brute-force_ pada password tersebut karena kita tidak memiliki petunjuk sama sekali tentang kriteria passwordnya. Oleh karena itu, lebih logis jika kita melihat isi dari file pertama terlebih dahulu.
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/01/11-access.png">
</p>

Jika anda teliti maka anda akan mendapatkan sebuah tabel yang cukup mencurigakan. Dalam kolom **username** terdapat nama **engineer** dan di kolom **password** terlihat, ya, password: **access4u@security**.

Mari kita buka zip milik **engineer** tadi.
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/01/12-zipopen.png">
</p>

Asik. Selanjutnya ada sebuah e-mail Microsoft Outlook yang memberikan informasi untuk tahap berikutnya.
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/01/14-outlookpassword.png">
</p>

Dalam e-mail tersebut kita mendapatkan sebuah credential. Mungkin anda bisa menebak untuk apa?
<br>

Username       : **security**

Password       : **4Cc3ssC0ntr0ller**
<br>
<br>
<br>

### Telnet Revisited
Kali ini kita bisa masuk :)
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/01/15-telnetlogin.png">
</p>

Ini adalah wajah dari _shell_ Windows (atau `cmd.exe`) yang terkoneksi melalui Telnet. Untuk saat ini, kita masuk sebagai user **security** yang memiliki privilege terbatas. Mari kita ambil flag user di **Desktop**.
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/01/16-user.png">
</p>

Mantap, sudah setengah jalan.
<br>
<br>
<br>
<br>

* * *
<br>
<br>
<br>
<br>

# Root
<br>

### Enumeration
Dalam setiap _pentest_/_heking_/mesin HTB, ada tahap dimana kita harus mencari informasi sebanyak-banyaknya agar lebih memudahkan kita dalam mengambil alih target. Tahap tersebut dapat dilakukan baik sebelum kita memulai (dengan tujuan _information gathering_) atau setelah kita mendapat akses terbatas seperti sekarang ini (dengan tujuan _privilege escalation_). Tahap tersebut disebut **enumeration**.

Singkatnya kita menggali lebih dalam tentang apa saja yang dijalankan komputer dan apa saja yang bisa diakses. Sebagai panduan awal, anda bisa melihat apa yang kira-kira dapat dilakukan saat pertama mendapatkan shell dari link **[berikut](https://exploitedbunker.com/articles/pentest-cheatsheet/)**.

Setelah mencoba-coba beberapa command dari link diatas, saya menemukan sesuatu yang menarik.
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/01/17-cmdkey.png">
</p>

Langkah berikutnya akan semakin menyenangkan.
<br>
<br>
<br>

### Run-as Who?
Apabila anda memiliki semangat nasionalis pantang menyerah, anda pasti sudah menemukan [artikel](https://www.maketecheasier.com/standard-users-run-program-admin-rights/) yang menjelaskan bahwa credential yang disimpan dan terlihat melalui `cmdkey` bisa dieksploit dengan sebuah command bernama `runas`.

Pada dasarnya, `runas` akan menjalankan command sebagai user yang ditentukan (mirip seperti `su` di Linux). Biasanya, untuk memudahkan seseorang agar tidak perlu mengetik password tiap kali `runas` dijalankan, seseorang akan menyimpan credential mereka layaknya anda mengisi kotak _remember me_ di situs-situs online saat login.

Efeknya adalah `runas` bisa dipakai siapa saja untuk menjalankan command sebagai user tersebut. Dalam kasus ini, user tersebut kebetulan adalah **Administrator**. Langsung saja kita eksekusi.

Saya lupa mengingatkan anda. Jika anda ingin mengerjakan box yang sama, alangkah lebih baik bila tahap ini dikerjakan di Windows. Saya mengerjakan semua di Linux dan ternyata _input/output_ dari shell yang digunakan tidak sejelas Windows dalam memberikan informasi (seperti _error message_ dan sejenisnya).

Pada kali pertama saya mengerjakan, saya tidak peduli dengan akses yang jauh lebih stabil ketimbang hanya mendapatkan flag, misalnya: root shell; dan hanya mementingkan flag root. Jadi saya tidak berpikir banyak dan langsung melakukan apa yang terpikir di benak saya saat itu (haha).
```
C:\Users\security> runas /user:ACCESS\Administrator /savecred "cmd.exe /c type c:\users\administrator\desktop\root.txt > c:\users\security\something"
```
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/01/18-runas.png">
</p>

Command tersebut akan menulis isi dari **root.txt** (`type` mirip seperti `cat` di Linux) ke tempat dimana user **security** bisa membaca.
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/01/19-root.png">
</p>

Dan akhirnya selesai :D

Box ini merupakan box pertama saya dalam HTB. Meskipun tingkat kesulitannya rendah (_easy_), box ini memberikan banyak pelajaran untuk saya. 

Meskipun anda banyak mendengar ini-itu tentang eksploit yang ada, sebelum anda mengerjakan sendiri anda tidak akan benar-benar mengerti.

Sekian untuk **Access**, silahkan ditunggu write-up selanjutnya.