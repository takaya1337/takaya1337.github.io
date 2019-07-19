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

### A Special Note
Karena beberapa hal, saya tidak dapat menyelesaikan _privilege escalation_ sebelum boxnya dinonaktifkan. Karena saya sangat penasaran, saya mencoba melihat **[write-up orang lain](https://medium.com/bugbountywriteup/active-a-kerberos-and-active-directory-hackthebox-walkthrough-fed9bf755d15)** dan mencoba mengerti tahap-tahapnya.

Saya tidak suka mengambil properti intelektual orang lain tanpa memberikan sumber yang jelas, dan saya harap anda dapat melakukan hal yang sama di kemudian hari.

Selamat menikmati.
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

Box kali ini memiliki banyak _port_ yang terbuka sehingga berpotensi membingungkan pemain yang belum terbiasa. Akan tetapi, dari seluruh _service_ yang terlihat dari `nmap`, ada sebuah service yang dapat dienumerasi tanpa memiliki _credential_ apapun, yaitu **port 445** atau yang tertulis sebagai **microsoft-ds**.
<br>
<br>
<br>

### SMB
SMB atau _Server Message Block_ adalah sebuah protokol berbasis _client-server_ yang berfungsi untuk membagi akses terhadap sebuah file (atau service, misalnya _printer_) antara dua box atau lebih di dalam suatu jaringan.

Perbedaan utama SMB dengan FTP terdapat dalam desainnya. SMB adalah **network sharing protocol** dimana tujuan utamanya ialah untuk **membagi akses** suatu file melalui sebuah jaringan sedangkan FTP, sesuai dengan namanya **file transfer protocol** dibuat untuk **mentransfer** file tanpa berinteraksi lebih lanjut dengan file tersebut.

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

Sebagai referensi sedikit, anda dapat membaca artikel berikut bertajuk **[Attack Methods for Gaining Domain Admin Rights in Active Directory](https://adsecurity.org/?p=2362)**.

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

Apabila anda membaca artikel tersebut, anda akan mengerti betapa vitalnya **cpassword** dalam box ini. Tag tersebut adalah credential yang dienkripsi oleh AES-256 bit yang seharusnya "relatif" aman, namun menjadi mudah untuk dipecahkan karena Microsoft membagikan **[kunci enkripsi](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-gppref/2c15cbf0-f086-4c74-8b70-1f2fa45dd4be)** dalam situs dokumentasinya.

Jika anda menggunakan Kali Linux seperti _heker-heker_ normal pada umumnya, `gpp-decrypt` telah tersedia dari awal.
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

Dengan credential diatas, flag user sudah di tangan.
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
Tema utama dalam box ini adalah Active Directory, sesuai dengan namanya. Active Directory menggunakan **Kerberos**, sebuah protokol autentikasi yang melibatkan pihak ketiga berbasis tiket (apabila anda pernah bekerja di dunia IT, anda pasti familiar dengan hal seperti "membuka tiket" ketika ada kendala dan nantinya tim IT akan memeriksa rincian yang tertera dalam tiket anda) sebagai metode autentikasinya.

Untuk informasi lebih lanjut, saya sangat anjurkan untuk membaca artikel berikut tentang **[Kerberos](https://www.roguelynn.com/words/explain-like-im-5-kerberos/)**.

Petunjuk untuk tahap ini ada di dalam nama User.
<br>
<br>
<br>

### About Kerberoast
Ketika saya mencari informasi tentang **SVC_TGS**, saya mendapat sebuah **[artikel menarik](http://www.harmj0y.net/blog/powershell/kerberoasting-without-mimikatz/)** (saat ini situs tersebut tidak dapat diakses, anda dapat membaca artikel **[berikut](https://www.blackhillsinfosec.com/a-toast-to-kerberoast/)** sebagai referensi tambahan) tentang **Kerberoast**.

Menurut sebuah **[dokumen](https://www.sans.org/cyber-security-summit/archives/file/summit-archive-1493862736.pdf)** dari Derbycon 2014 yang saya temui, **Kerberoast** adalah sebuah serangan terhadap sistem tiket Kerberos. Service yang terdaftar dalam domain Active Directory tersebut memiliki SPN atau _Service Principal Name_, sebuah ID unik yang nantinya akan diasosiasikan dengan sebuah "akun" untuk service tersebut, sehingga ketika user ingin mengakses service tersebut, ia hanya diberikan tiket (perlu diingat bahwa Kerberos berbasis tiket) yang mengandung credential penyedia service (dalam bentuk **NTLM hash**).

Untuk melancarkan serangan tersebut, anda perlu memiliki sebuah akun yang valid agar bisa melakukan _request_ tiket karena service hanya dapat diakses oleh user yang terdaftar. Disini akun user berperan.

Dari referensi write-up yang saya letakkan diatas, saya juga menggunakan **[impacket](https://github.com/SecureAuthCorp/impacket)**, serangkaian _networking tool_ yang memiliki banyak fitur, salah satunya adalah fitur untuk melakukan **request TGS**.
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/02/10-impacket.png">
</p>

```
$ impacket/examples/GetUserSPNs.py -request -dc-ip 10.10.10.100 ACTIVE.HTB/SVC_TGS:GPPstillStandingStrong2k18
```
> command tersebut akan meminta _Silver Ticket_ atau _Ticket Granting Service_ (sebuah tiket yang diberikan ketika user ingin mengakses suatu service) ke _Domain Controller_ 10.10.10.100 dengan credential user **SVC_TGS**.

<br>
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/02/11-getuserspn.png">
</p>

Langkah terakhir adalah mendapatkan password Administrator dari hash tersebut.
<br>
<br>
<br>

### Crack!
Untuk memecahkan (_crack_) hash tersebut, anda dapat menggunakan `hashcat`. Tetapi sebelum itu anda harus mengetahui tipe hash yang ingin dicrack.
Informasi tersebut bisa didapat dari situs **[hashcat](https://hashcat.net/wiki/doku.php?id=example_hashes)** dengan mencari `krb5tgs$23`, sesuai dengan _response_ yang diberikan request TGS tadi.
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/02/13-hashtype.png">
</p>

Biasanya, _wordlist_ yang digunakan untuk _bruteforce_ atau crack adalah **[rockyou.txt](https://github.com/praetorian-inc/Hob0Rules/blob/master/wordlists/rockyou.txt.gz)**.

Setelah semuanya siap, anda dapat menyalakan `hashcat`.
```
$ hashcat -m 13100 -a 0 tgs /usr/share/wordlist/rockyou.txt
```
> `-m 13100` adalah tipe hash yang didapat dari situs diatas
>
> `-a 0` adalah _attack mode_ yang digunakan, yaitu **straight** (mencoba _entry_ dari wordlist satu-satu)
>
> `tgs` adalah nama file yang berisi response
>
> dan `/usr/share/wordlist/rockyou.txt` adalah wordlist yang digunakan.

<br>
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/02/14-hashcat.png">
</p>

Anda dapat melihat password Administrator :)
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/02/15-rootflag.png">
</p>

Jika anda ingin mendapatkan _shell_ ketimbang menggunakan SMB:
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/02/16-trueshell.png">
</p>


Saya sempat absen beberapa bulan dikarenakan pekerjaan (_underpaid_ a.k.a _intern_) dan baru bisa main HTB lagi karena akhir-akhir ini sedang sedikit senggang.

Saya masih menyimpan beberapa write-up yang telah selesai dan tinggal dipindahkan ke _markdown_, tapi butuh waktu karena saya ingin membuat konten yang "relatif" berkualitas.

Akhir kata, mohon bersabar jika anda menunggu write-up berikutnya :)