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

Sebelum anda membaca hal-hal teknis, ada baiknya anda membaca sedikit kata pembuka dari saya terlebih dahulu. Seperti anda ketahui, saya membuat write-up dalam dua bahasa, Inggris dan Indonesia. Saya perlu ingatkan di awal bahwa write-up saya tidak akan sama persis antara dua bahasa tersebut karena saya tidak menerjemahkan mentah-mentah melainkan saya _edit_ ulang sesuai dengan pemikiran saya sebagai orang Indonesia.
* Mengapa saya mau repot-repot membuat write-up dalam dua bahasa?

Anggap saja sebagai bentuk pengabdian kepada bangsa. Saya ingin meningkatkan kemampuan _"heking"_ orang-orang yang ingin belajar tapi belum mampu mengerti write-up kelas atas (yang kebanyakan berbahasa Inggris) agar tidak seperti salah seorang pentester yang cukup dikenal karena "kehebatannya".
* Mengapa HTB?

Karena saya bosan main CTF terus dan saya juga berpikir bahwa sudah banyak write-up CTF berbahasa Indonesia yang cukup bagus (baca: [petircysec.com](https://petircysec.com)), jadi saya ingin membuat sesuatu yang relatif baru.

* Bagaimana cara mainnya?

HTB atau _Hack the Box_ adalah sebuah permainan dimana anda bisa memilih beberapa list IP address yang merupakan sebuah mesin yang pasti memiliki kelemahan tertentu.

Dari kelemahan tersebut, anda harus mendapatkan dua buah hash atau flag seperti "7f3ff9f556f5f470e41508ff970c794e". Ada dua flag dalam tiap mesin, yaitu **user** dan **root**.

Flag user anda dapatkan ketika anda mendapat akses pertama ke mesin dengan _privilege_ seadanya (user biasa). Flag root akan anda dapatkan ketika anda berhasil melakukan _privilege escalation_ (atau EOP dalam bahasa korporat) dimana anda mendapatkan akses sebagai user tertinggi (root atau Administrator).

Secara garis besar, tiap write-up akan saya bagi menjadi dua bab utama: User dan Root, dimana masing-masing akan memiliki penjelasan sendiri yang berbeda sesuai dengan kelemahan yang ada dalam mesin tersebut.

Sekian untuk pendahuluan.

## User
<br>

### NMAP
Hal pertama yang perlu dilakukan ketika menemukan sebuah IP adalah meng-`nmap` IP tersebut. Hal ini diperlukan agar kita dapat melihat gambaran mesin secara keseluruhan sehingga dapat menentukan serangan apa yang cocok. Sintaks favorit saya adalah:

```
$ nmap -p 1-65535 -T4 -A -v 10.10.10.98
```
Sintaks tersebut akan mengecek semua _port_, melakukan scan dengan _Timing-level_ 4 (**Aggresive** -- artinya mempercepat scan tetapi tidak terlalu cepat sampai hasil scan tidak akurat), dan menggunakan _verbosity-level_ 1 (artinya `nmap` akan sedikit lebih detail dalam memberikan hasil).
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/01/01-nmap.png">
</p>

Kita dapat melihat bahwa mesin tersebut memiliki beberapa _service_ yang terbuka, yaitu: **FTP server, Telnet, dan HTTP server (Microsoft IIS 7.5)**. Mari kita cek yang paling pertama.
<br>
<br>

### FTP
FTP atau _File Transfer Protocol_ adalah sebuah protokol atau **cara** untuk memindahkan file secara online. Anda bisa mencari info lebih lanjut tentang FTP di Internet (seperti perbedaan FTP dan FTPS serta mengapa anda tidak disarankan untuk menggunakan FTP untuk keperluan sehari-hari) karena saya hanya akan membahas yang berhubungan dengan mesin ini saja.

Untuk mengakses FTP anda bisa menggunakan FTP client (bisa CLI maupun GUI).
<br>

<p align="center">                                                        
<img src="https://takaya1337.github.io/htb/assets/01/02-ftp.png">         
</p>

Setelah anda berhasil masuk, anda harusnya menemukan dua folder yang mencurigakan: "Backup" dan "Engineer". Anda bisa men-download filenya.
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
> argumen `--no-passive-ftp` akan men-disable Passive mode FTP (singkatnya dalam Passive mode FTP, server tidak membuat koneksi ke client dan file transfer hanya satu arah dari client ke server). Biasanya argumen ini dipakai bila **tidak ada** _client-side firewall_.
>
> argumen `-r` mengindikasikan bahwa kita ingin mengambil file secara _recursive_, artinya bila ada folder kita ingin mengambil file sampai ke folder terakhir. Secara default, `-r` hanya mengambil sampai maksimum 5 file kedalam. Jika anda ingin men-download file yang memiliki banyak subfolder anda harus mengganti angkanya sesuai kebutuhan.
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/01/07-wgetway.png">
</p>

Agar lebih yakin kalau cara FTP dan `wget` berjalan lancar, kita bisa bandingkan file-nya.
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/01/08-compare.png">
</p>

Mantap, file-nya oke. Mari kita cek file tersebut lebih lanjut.
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/01/09-filecheck.png">
</p>

Sepertinya file-file tersebut akan sangat bergantung pada Windows. Untungnya, tidak sulit bagi orang Indonesia untuk mendapat akses ke mesin Windows :)

Tapi sebelumnya mari kita lihat Telnet terlebih dahulu.
<br>
<br>

### Telnet
Telnet atau _Teletype Network_ adalah sebuah protokol atau **cara** untuk mengakses sebuah _remote machine_. Bila anda familiar dengan SSH, Telnet adalah nenek moyangnya yang sedikit lebih berbahaya. Fungsinya sama seperti SSH, untuk memberikan akses interaktif dengan sebuah mesin lewat internet.

Untuk _connect_ ke Telnet mesin ini, anda dapat menggunakan `telnet` _command_ di Linux.
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/01/10-telnetcheck.png">
</p>

Kita belum mendapatkan _credential_ yang diperlukan untuk mengakses Telnet. Anda bisa menyiapkan mesin Windows anda untuk langkah selanjutnya.
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


