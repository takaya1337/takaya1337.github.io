---
title: Irked
layout: default
---

# Irked
<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/03/irked.png">
</p>

#### OS            : Linux
#### IP            : 10.10.10.117
#### Maker         : MrAgent
* * *
<br>
<br>
<br>
<br>

## User
<br>

### NMAP
Kebetulan box ini adalah box Linux pertama saya, awalnya saya ingin fokus bermain di box Windows, namun banyak teman saya yang mencoba box Linux dan karena banyak yang bertanya-tanya, saya akhirnya mencoba juga.

Seperti biasa, langsung `nmap` saja:
```
$ nmap -p 1-65535 -T4 -A -v 10.10.10.117
```
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/03/irked1-nmap.png">
</p>

Ada beberapa _service_ yang terdapat dalam box ini, yaitu **SSH**, **Apache 2.4.10**, dan **UnrealIRCd**. Pada umumnya, _web server_ dienumerasi terlebih dahulu karena biasanya terdapat banyak informasi yang dapat digali tentang sebuah sistem.
<br>
<br>
<br>

### The Website
<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/03/irked1.2-web.png">
</p>

Di dalam situs web hanya terdapat sebuah gambar dan tulisan yang menyatakan bahwa service IRC hampir berjalan.

Mari kita lihat IRC tersebut.
<br>
<br>
<br>

### IRC
Sebagai pembuka, IRC atau Internet Relay Chat adalah sebuah protokol yang memperbolehkan banyak client berkomunikasi secara _real-time_ di dalam suatu server.
IRC sangat populer di tahun 90-an hingga awal 2000, namun sekarang penggunanya sudah berkurang jauh akibat maraknya platform modern yang lebih banyak fitur.
Saya sendiri lebih menyukai platform yang ringan dan tidak memiliki terlalu banyak fitur yang tidak penting (favorit saya sekarang adalah Telegram).

Kembali ke box, anda dapat langsung memeriksa IRC tersebut dengan terhubung kedalamnya.
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/03/irked2-irc.png">
</p>

Dalam box ini, IRC tidak berfungsi sebagaimana mestinya, seperti yang dijelaskan oleh situs di port 80.

Akan tetapi, apabila anda mencari informasi tentang jenis IRC yang digunakan, anda akan menemukan sebuah artikel menarik.
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/03/irked2.3-infocve.png">
</p>

Terdapat sebuah CVE atau _Common Vulnerability and Exposure_ mengenai **UnrealIRCd**, service IRC yang digunakan oleh box ini.

CVE tersebut menjelaskan bahwa ada **versi UnrealIRCd yang dimodifikasi sehingga mengandung kode yang dapat memberikan akses (umumnya berupa _shell_) atau yang lebih dikenal _backdoor_ dan disebarluaskan kembali secara online**.

Yang lebih nikmat lagi adalah, ada sebuah modul **metasploit** yang siap dipakai oleh kalangan hacker menengah kebawah :)
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/03/irked2.2-irccve.png">
</p>

Mari kita buka tool tersebut.
<br>
<br>
<br>

### Metasploit My Life
Menggunakan Metasploit sangat mudah, anda hanya perlu mengatur segala yang diminta, seperti exploit apa yang ingin anda gunakan, port target yang ingin anda serang, dan seterusnya.

Untuk box ini saya menggunakan modul **unreal_ircd_3281_backdoor**.
```
msf > use exploit/unix/irc/unreal_ircd_3281_backdoor
msf > set RHOST 10.10.10.117
msf > set RPORT 6697
msf > exploit
```
> `use` menyatakan modul yang ingin kita gunakan, yaitu **unreal_ircd_3281_backdoor**
> 
> `set RHOST` dan `set RPORT` adalah **RemoteHost** dan **RemotePort**, diisi dengan alamat target yang ingin diserang
>
> `exploit` menjalankan modul dengan pengaturan yang telah ditetapkan sebelumnya.

<br>
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/03/irked3-msf.png">
</p>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/03/irked4-ircshell.png">
</p>

Setelah anda mendapatkan shell, silahkan anda enumerasi agar dapat mengetahui lebih dalam tentang box ini.

Dalam beberapa box, terkadang anda harus mendapatkan akses sebagai user lagi setelah mendapat _initial foothold_.
<br>
<br>
<br>

### Another User
Bila anda sudah mengenumerasi box tersebut, anda pasti mengetahui lokasi flag user tetapi tidak dapat mengakses file tersebut (kecuali ada yang merubah _permission_ karena iseng, tidak jarang terjadi di box Linux yang _easy_).

Anda juga sudah mengetahui bahwa user yang anda gunakan saat ini adalah **ircd**, sedangkan flag tersebut milik **djmardov**. Untuk itu, anda harus mencari cara agar anda bisa mendapatkan _credential_ user tersebut.

Umumnya di dalam sebuah sistem Linux, ada sebuah file yang mengandung _history_ perintah-perintah yang pernah digunakan sebelumnya, yaitu **.bash_history**.
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/03/irked5-enum.png">
</p>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/03/irked6-history.png">
</p>

Dengan memeriksa **.bash_history** milik user yang anda gunakan sekarang, anda dapat melihat bahwa ada interaksi antara file **.backup** yang diletakkan dalam directory Document milik **djmardov**.

Tidak seperti flag user, anda dapat mengakses file tersebut.
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/03/irked7-hiddenfile.png">
</p>
<br>
<br>

### Steg is for Steganography
File tersebut berisi password, namun jika anda mencoba password tersebut untuk SSH, anda tidak akan bisa masuk.

Jika anda membaca deskripsi yang diberikan dengan seksama, ada sebuah kata yang terlihat seperti _typo_ padahal bukan.

Apabila anda bingung, anda dapat melihat-lihat **[forum](https://forum.hackthebox.eu/discussion/1278/irked/p4)** agar mendapat beberapa petunjuk. Anda tidak perlu takut akan di-spoiler, karena komentar-komentar yang terindikasi bersifat membocorkan akan dihapus oleh moderator.

Dengan petunjuk yang sudah anda dapatkan dan dengan membaca judul bab ini, anda akan semakin yakin bahwa anda menghadapi sebuah **steganography challenge**.

Password yang anda dapatkan dari file **.backup** adalah password yang digunakan untuk mengunci file ke dalam sebuah media. Media yang digunakan dalam soal-soal stegano umumnya berbentuk gambar atau lagu (karena video biasanya berukuran besar sehingga tidak praktis untuk di-download).

Dalam box ini, hanya ada satu media yang sejauh ini anda temui.
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/03/irked1.2-web.png">
</p>

Jaman saya masih suka main CTF, saya pernah mendapatkan soal serupa dimana peserta harus mendukuni password yang digunakan untuk ekstraksi file di dalam media.

Dibanding pengalaman buruk masa lalu, soal ini sangat mudah karena tidak ada unsur [[[dukun]]] dalam pengerjaannya.

Anda cukup perlu mencoba beberapa tool untuk soal stegano yang cukup populer, sebelum akhirnya menemukan bahwa **steghide** adalah tool yang tepat.
```
$ steghide extract -sf irked.jpg -p UpupDOWNdownLRlrBAbaSSss
```
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/03/irked10-djpass.png">
</p>

Dari gambar tersebut anda akan mendapatkan credential user yang sebenarnya.
<br>

Username    : **djmardov**

Password    : **Kab6h+m+bbp2J:HG**
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/03/irked11-user.png">
</p>

Selanjutnya akan jauh lebih menyenangkan :)
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
Seperti box-box sebelumnya dan yang akan datang, tahap ini merupakan tahap terpenting dalam _privilege escalation_. Lewat enumerasi anda dapat mengetahui service apa yang sedang berjalan, file apa yang dapat anda akses dan seperti apa permissionnya, dan sebagainya.

Anda dapat menggunakan script dari Internet untuk memudahkan enumerasi, tetapi anda perlu mengerti terlebih dahulu apa yang harus diperhatikan sebelum script tersebut dapat berguna untuk anda.

Anda dapat membaca artikel **[berikut](https://blog.g0tmi1k.com/2011/08/basic-linux-privilege-escalation/)** untuk mengerti sekilas tentang enumerasi.

Yang menjadi perhatian untuk box ini adalah **Advanced File Permission di dalam Linux**.
<br>
<br>
<br>

### Basic Linux File Permission
Sebelum terjun ke bagian  _"advanced"_, ada baiknya kita mengulang yang _basic_ terlebih dahulu.

Di dalam sistem Linux, semua file diatur dengan permission sebagai bentuk _access control_, dimana permission menentukan siapa saja yang bisa mengakses file tersebut dan apa saja akses yang diberikan.

Contoh permission dapat berbentuk seperti ini:
```
drwxrwxrwx 2 john john    0 Jul 14 20:00 test-dir
-rw-r--r-- 1 root root   32 Aug 10 00:00 root.txt
```
> atau lebih nikmat jika disajikan berwarna:

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/03/perm.png">
</p>

Semua file di Linux (termasuk directory/folder) memiliki tiga pasang atribut `rwx` yang menentukan apakah file tersebut: 
> bisa dibaca (`readable`)
>
> bisa ditulis (`writable`)
>
> dan bisa dijalankan (`executable`)

oleh:
> `owner` (user yang digunakan saat pembuatan file)
>
> `group` (group utama/_primary group_ milik user)
>
> `others` (user lain yang bukan pemilik dan tidak memiliki group yang sama dengan file).

<br>

Misalnya dalam kasus flag user milik **djmardov** tadi, anda tidak dapat "membaca" file tersebut karena semua permission untuk `group` dan `others` tidak diaktifkan, sehingga hanya sang pemilik, **djmardov** sendiri yang bisa membaca dan menulis file tersebut.

Itulah sebabnya anda wajib mendapatkan akses sebagai **djmardov** untuk membaca flag user, karena siapapun selain **djmardov** dan **root** tidak diperbolehkan membaca, menulis, maupun mengeksekusi file tersebut.

Contoh lebih lanjut misalnya, ketika anda melakukan:
```
$ chmod 644 root.txt
```

Inilah yang terjadi:
<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/03/perm2.png">
</p>

Bilangan octal yang anda masukkan sebagai argumen untuk `chmod` akan diterjemahkan ke dalam bentuk biner, untuk menyatakan apakah "bit" yang dimiliki atribut tersebut "dinyalakan" atau "dimatikan" (apabila anda ingat pelajaran waktu SD, guru sering menerangkan bahwa komputer hanya mengerti 0 dan 1).

Tiga bit diatas terletak di dalam kolom yang mewakili para pihak yang terlibat dalam sistem:
> kolom pertama (hijau) milik `owner/user`
> 
> kolom kedua (biru) milik `group`
>
> kolom ketiga (oranye) milik `others`.

Posisi bit dalam kolom tersebut mewakili tiga atribut yang akan menentukan jenis akses yang diberikan:
> 2<sup>2</sup> = 4, untuk `r`
> 
> 2<sup>1</sup> = 2, untuk `w`
>
> 2<sup>0</sup> = 1, untuk `x`.

Karena tiap kolom memiliki tiga atribut (`rwx`), jumlah maksimal yang dapat ditampung oleh ketiga bit tersebut adalah jumlah bilangan biner sampai angka ketiga dari kanan, yaitu 2<sup>2</sup> + 2<sup>1</sup> + 2<sup>0</sup> = 7. 

Itulah kenapa bilangan octal yang digunakan, bukan decimal. Perlu anda ingat bahwa angka tertinggi dalam bilangan decimal adalah 9, sedangkan dalam bilangan octal adalah 7, sehingga paling sesuai untuk kondisi ini.

Gabungan dari tiga kolom dan tiga atribut diatas menyusun yang disebut **Basic Linux File Permission**.
<br>
<br>
<br>

### On to SUID, SGID, and Sticky Bit
Dari penjelasan diatas, harusnya anda sudah mengetahui bahaya sebuah file yang memiliki permission 777, dimana siapapun dapat melakukan apa saja terhadap file tersebut.

Terlebih lagi, apabila pemilik file tersebut adalah user lain, dan ada sesuatu yang aneh dari permissionnya.
```
-rwxrwxrwx 2 john john    0 Jul 14 20:00 script.sh
```

```
-rwsrwxrwx 2 john john    0 Jul 14 21:00 shell.sh
```

Terlihat ada yang berbeda dari kedua permission diatas?

Yang kedua akan menjalankan (mengeksekusi) `shell.sh` sebagai **john**, terlepas dari siapapun yang menjalankan file tersebut.

Mengapa demikian?

Contoh diatas merupakan sebuah bentuk dari tiga **Advanced Linux File Permission**.

Ada tiga jenis permission lanjutan yang diterapkan dalam sistem Linux,

> 1. **[SUID](https://www.linux.com/blog/what-suid-and-how-set-suid-linuxunix)** atau Set owner User ID, jika dinyalakan dalam suatu file akan membuat file tersebut dijalankan atas nama user yang tertera dalam permission.
>
> 2. **[SGID](https://www.linux.com/blog/what-sgid-and-how-set-sgid-linux)**, sama seperti SUID, hanya bedanya terletak pada group, bukan user.
>
> 3. **[Sticky Bit](https://www.linux.com/blog/what-sticky-bit-and-how-set-it-linux)**, permission yang umumnya terletak dalam directory dan membuat hanya si pemilik file yang bisa menghapus file yang dimilikinya.

```
$ SUID:
-rwsr-xr-x 2  john john    0 Jul 14 20:00 user-script.sh

$ SGID:
-rwxr-sr-x 2  john smith   0 Jul 14 20:00 group-script.sh

$ Sticky Bit
drwxrwxrwt 13 root root    0 Jul 14 20:00 /tmp
```

Perhatikan perbedaan ketiga file diatas:
> file pertama akan membuat `user-script.sh` berjalan dengan permission yang dimiliki **john**
>
> file kedua akan mengizinkan siapapun yang memiliki group **smith** dalam akunnya untuk menjalankan `group-script.sh` sebagai usernya sendiri namun dengan permission yang dimiliki group **smith**
>
> file ketiga tidak mengizinkan user **bruce** untuk menghapus file yang dimiliki user **john** maupun sebaliknya.

Penjelasan diatas hanyalah penjelasan saya yang dibuat seringkas mungkin agar dapat dipahami dengan mudah, namun jika anda tertarik lebih lanjut anda dapat membaca artikel **[berikut](https://www.thegeekdiary.com/what-is-suid-sgid-and-sticky-bit/)**.
<br>
<br>
<br>

### Just a Common Binary, with a Special Permission
Setelah mengetahui sedikit tentang cara kerja file permission di Linux, anda dapat membayangkan apa yang akan terjadi jika ada sebuah file yang dibuat oleh **root** dan memiliki SUID-bit.

Biasanya, memang ada beberapa program yang memiliki SUID-bit agar dapat menjalankan fungsinya, contohnya: `sudo`. Hal ini membuktikan bahwa tidak semua file yang memiliki SUID-bit berbahaya, akan tetapi hal tersebut akan menjadi bahaya jika terjadi kesalahan implementasi program yang kemudian memiliki SUID-bit.

Karena kebanyakan programmer/sysadmin sering menghalalkan segala cara agar pekerjaannya selesai dengan cepat, yang biasanya terjadi adalah hal-hal yang tidak diinginkan seperti yang dibicarakan diatas.

Oleh sebab itu, pencarian file dengan _special permission_ menjadi suatu keharusan dalam tahap enumerasi.

Untuk mencari file yang memiliki SUID-bit, anda dapat menggunakan `find`:
```
$ find / -perm -u=s -type f 2>/dev/null
```
> anda ingin mencari file mulai dari `/`
>
> dimana file tersebut memiliki `special permission dalam kolom user/owner`
>
> tipenya adalah `file` **bukan** `directory`
>
> dan anda tidak ingin melihat _error message_.

<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/03/irked12-suid.png">
</p>

Dari semua program yang keluar dengan command diatas, carilah yang tidak semestinya ada di sistem. Seburuk-buruknya apabila anda harus mencoba satu-satu pun tidak memakan waktu lama. Akan tetapi jika anda ingin lebih efektif, anda dapat membandingkan file-file tersebut dengan file yang ada di komputer anda.
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/03/irked13-anomaly.png">
</p>

Program ini saya temukan setelah beberapa kali _trial & error_. Jika anda jalankan, program ini akan menjalankan command `who` dan mencoba menjalankan **tmp/listusers** dengan `sh`.

Jika benar, maka program ini dapat memberikan kita shell sebagai **root**. Agar lebih yakin, saya mengambil programnya lewat `scp` agar dapat dianalisa lebih lanjut.

Perlu diketahui bahwa anda bisa berinteraksi langsung dengan box melalui alamat IP **tun0** yang diberikan dalam **.ovpn** anda, tetapi (menurut pihak HTB) anda tidak dapat berinteraksi dengan pemain lainnya menggunakan cara yang sama.

Anda perlu menyalakan `openssh-server` agar bisa melakukan `scp` dengan box ini. Ada beberapa kasus dimana `scp` tidak bisa digunakan, namun `wget` bisa, ada juga sebaliknya. Hal seperti ini tidak dapat ditentukan adanya, karena murni tergantung dari box yang anda mainkan.
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/03/exfiltrate-program.png">
</p>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/03/file.png">
</p>

Program tersebut adalah ELF 32-bit, atau dapat disebut "exe-nya Linux". Anda bisa menggunakan _disassembler_ jika ingin melihat isinya.
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/03/program.png">
</p>

Isi dari fungsi _main()_ program ini sangat sederhana, yaitu menjalankan `who`, menyalakan SUID-bit sebagai **root** (karena User ID **root** selalu '0'), dan mengeksekusi apapun yang ada di dalam **/tmp/listusers**.

Dugaan awal saya benar, saya hanya perlu menaruh shell di file tersebut dan mengeksekusi program ini sehingga **root** akan membuka shell saya.
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/03/irked15-privesc.png">
</p>

Asik, akhirnya dapat **#**!