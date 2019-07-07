---
title: Active
layout: default
---

# Active

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/02/active.png">
</p>

#### OS            : Windows
#### IP            : 10.10.10.100
#### Maker         : eks & mrb3n
* * *
<br>
<br>
<br>
<br>

## User
<br>

### NMAP
Seperti sebelumnya, anda dapat memulai dengan melakukan `nmap` terlebih dahulu.
```
$ nmap -p 1-65535 -T4 -A -v 10.10.10.100
```
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/02/01-nmap.png">
</p>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/02/02-nmap.png">
</p>

Mesin kali ini memiliki banyak _port_ yang terbuka sehingga berpotensi membingungkan pemain yang belum terbiasa. Akan tetapi, dari seluruh _service_ yang terlihat dari `nmap`, ada sebuah service yang dapat dienumerasi tanpa memiliki _credential_ apapun, yaitu **port 445** atau yang tertulis sebagai **microsoft-ds**.
<br>
<br>

##SMB
SMB atau _Server Message Block_ adalah sebuah protokol berbasis _client-server_ yang berfungsi untuk membagi akses terhadap sebuah file (atau service, misalnya _printer_) antara dua mesin atau lebih di dalam suatu jaringan.

Perbedaan utama SMB dengan FTP ialah SMB adalah **network sharing protocol** dimana tujuan utamanya adalah **membagi akses** suatu file melalui sebuah jaringan sedangkan FTP sesuai dengan namanya **file transfer protocol** dibuat untuk **mentransfer** file tanpa berinteraksi lebih lanjut dengan file tersebut.

Untuk mengenumerasi SMB, anda dapat menggunakan `smbclient` atau `nullinux`. Anda dapat mencari informasi lebih lanjut tentang kedua _tool_ tersebut di Internet.
```
$ python3 nullinux.py -a 10.10.10.100
```
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/02/03-nullinux.png">
</p>

Dengan `nullinux`, kita dapat melihat ada sebuah _share_ yang dapat diakses dengan _null session_ (koneksi tanpa credential terhadap service di Windows), yaitu **Replication**.

Anda dapat menginisiasi koneksi dengan menggunakan tool pertama yang saya sebut diatas.
```
$ smbclient //10.10.10.100/Replication/ -I 10.10.10.100
```
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/02/04-smb.png">
</p>

Ada baiknya jika anda melakukan sedikit eksplorasi setelah mendapat akses pertama (atau yang lebih sering disebut sebagai _initial foothold_ di komunitas HTB) sebelum membaca lebih lanjut.

Sebagai referensi sedikit, anda dapat membaca artikel berikut bertajuk: **[Attack Methods for Gaining Domain Admin Rights in Active Directory](https://adsecurity.org/?p=2362)**.

Artikel tersebut menyebutkan bahwa dalam Active Directory, terdapat beberapa file XML berisi credential yang dienkripsi oleh AES-256 bit.
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/02/04-smbexplore.png">
</p>

Anda dapat mengambil file tersebut dengan menggunakan _command_ `get` sama seperti FTP.
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/02/05-groupxml.png">
</p>

Apabila anda membaca lebih lanjut, anda akan mengerti betapa vitalnya **cpassword** dalam mesin ini. Tag tersebut adalah credential yang dienkripsi oleh AES-256 bit yang seharusnya "relatif" aman, akan tetapi Microsoft membagikan **[kunci enkripsi](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-gppref/2c15cbf0-f086-4c74-8b70-1f2fa45dd4be)** dalam halaman dokumentasinya.

Jika anda menggunakan Kali Linux seperti _heker-heker_ normal pada umumnya, `gpp-decrypt` telah tersedia ketika anda selesai menginstall.
```
$ gpp-decrypt [password yang dienkripsi]
```
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/02/06-gpp.png">
</p>

Username    : **active.htb\SVC_TGS**

Password    : **GPPstillStandingStrong2k18**
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/02/07-smbuser.png">
</p>

Tinggal ambil flagnya :)
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/02/08-userflag.png">
</p>

Berikutnya anda hanya perlu melakukan _privilege escalation_ untuk mendapatkan **Administrator**.
<br>
<br>
<br>
<br>

* * *
<br>
<br>
<br>
<br>

## Root
<br>

### Kerberos
Tema utama dalam mesin ini adalah Active Directory, sesuai dengan nama mesinnya. Active Directory menggunakan **Kerberos**, sebuah protokol autentikasi yang melibatkan pihak ketiga berbasis tiket (apabila anda pernah bekerja di dunia IT, anda pasti familiar dengan hal seperti "membuka tiket" ketika ada kendala dan nantinya tim IT akan memeriksa rincian yang tertera dalam tiket anda) sebagai metode autentikasinya.

Untuk informasi lebih lanjut, saya sangat anjurkan untuk membaca artikel berikut tentang **[Kerberos](https://www.roguelynn.com/words/explain-like-im-5-kerberos/)**.

Petunjuk untuk tahap ini ada di dalam nama User.
<br>
<br>

### About Kerberoast
