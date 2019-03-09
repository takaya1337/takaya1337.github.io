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



## User
<br>

### NMAP
Mesin ini merupakan mesin pertama saya yang berhasil saya root. Sebelum saya menjelaskan lebih jauh, perlu diingat bahwa terjemahan Bahasa Indonesia dalam website saya tidak akan selalu sama persis dengan versi bahasa Inggrisnya. Menurut saya, saya lebih mudah menjelaskan dalam bahasa Inggris, tetapi saya akan mencoba sebisa mungkin untuk menjelaskan sejelas-jelasnya.
<br>

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
