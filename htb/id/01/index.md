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

Sebelum anda membaca hal-hal teknis, ada baiknya anda membaca sedikit kata pembuka dari saya terlebih dahulu. Seperti anda ketahui, saya membuat write-up dalam dua bahasa, Inggris dan Indonesia. Saya perlu ingatkan di awal bahwa write-up saya tidak akan sama persis antara dua bahasa tersebut karena saya tidak menerjemahkan mentah-mentah melainkan saya __edit__ ulang sesuai dengan pemikiran saya sebagai orang Indonesia.
* Mengapa saya mau repot-repot membuat write-up dalam dua bahasa?

Anggap saja sebagai bentuk pengabdian kepada bangsa. Saya ingin meningkatkan kemampuan **"heking"** orang-orang yang ingin belajar tapi belum mampu mengerti write-up kelas atas (yang kebanyakan berbahasa Inggris) agar tidak seperti salah seorang pentester yang cukup dikenal karena "kehebatannya".
* Mengapa HTB?

Karena saya bosan main CTF terus dan saya juga berpikir bahwa sudah banyak write-up CTF berbahasa Indonesia yang cukup bagus (baca: [petircysec.com](https://petircysec.com)), jadi saya ingin membuat sesuatu yang relatif baru.

* Bagaimana cara mainnya?

HTB atau __Hack the Box__ adalah sebuah permainan dimana anda bisa memilih beberapa list IP address yang merupakan sebuah mesin yang pasti memiliki kelemahan tertentu.

Dari kelemahan tersebut, anda harus mendapatkan dua buah hash atau flag seperti "7f3ff9f556f5f470e41508ff970c794e". Ada dua flag dalam tiap mesin, yaitu **user** dan **root**.

Flag user anda dapatkan ketika anda mendapat akses pertama ke mesin dengan __privilege__ seadanya (user biasa). Flag root akan anda dapatkan ketika anda berhasil melakukan __privilege escalation__ (atau EOP dalam bahasa korporat) dimana anda mendapatkan akses sebagai user tertinggi (root atau Administrator).

Secara garis besar, tiap write-up akan saya bagi menjadi dua bab besar: User dan Root, dimana masing-masing akan memiliki penjelasan sendiri yang berbeda sesuai dengan kelemahan yang ada dalam mesin tersebut.

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

Kita dapat melihat bahwa mesin tersebut memiliki beberapa service yang terbuka, yaitu: **FTP server, Telnet, dan HTTP server (Microsoft IIS 7.5)**. Mari kita cek yang paling pertama, yaitu **FTP**.
