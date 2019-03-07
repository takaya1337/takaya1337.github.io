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

## Preface
If you've played this kind of game before, you can skip this chapter. If you're totally new (like me when I first started working on this box), consider this the tutorial.
* So what is HTB? Or rather, what should you do with it?

To put it simply, we're given a vulnerable machine's IP address. Since the machine has vulnerabilities, it is our duty to exploit them and retrieve a hash (something like "7f3ff9f556f5f470e41508ff970c794e") and submit it to HTB for scores.

For each machine there are two hashes (or flags), one for **user** and one for **root**. Usually we'll get the user flag when we first gain a shell access to the machine and the root flag when we gain the highest-privileged access to the machine (or "rooted" the machine).

Thus, my write-ups will contain two chapters in general: User and Root and each will have its own subchapters depending on what kind of vulnerabilities on the machine.

That's it. That's what it's all about.
* * *
<br>
<br>
<br>
<br>

## User
<br>

### NMAP
The first thing to do when given an IP is to do `nmap` on the machine, to see what kind of attack might be used. The magic syntax is:
```
$ nmap -p 1-65535 -T4 -A -v 10.10.10.98
```
to scan all open ports and services. Here is the output.
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/01/01-nmap.png">
</p>

We can see that the box has **FTP server, Telnet, and HTTP server (Microsoft IIS 7.5)** made accessible to the internet. The most methodical approach is to try each of them starting from the first, and since an FTP server sometimes allows anonymous login, we can at least check that out.
<br>
<br>

### FTP
Simply fire-up the FTP, you can use command-line or GUI to achieve this.
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/01/02-ftp.png">
</p>

If you take some time in exploring the FTP, you will find two folders: "Backup" and "Engineer". Download the files to proceed.
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/01/03-ftperror.png">
</p>

You can use the ftp-command `get` to download files from an FTP server, but it won't work flawlessly everytime. If you're using the Linux FTP command-line client out of the box, you should get the same error as me, though I haven't tried the GUI or the Windows version of the client.

A quick search shows that the file is "not transferred correctly" because the way FTP transfer files, **ASCII mode** and **Binary mode**.

**ASCII mode** will modify the files transferred (the line-feed) if the communication is between two different systems with different line-feed convention (like Linux and Windows).

**Binary mode**, on the other hand will transfer the files without any modification, and this is what we want at the moment (and ideally this is what we should use on any FTP transfer).
```
ftp> bin
ftp> get [whatever you want to get]
```
<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/01/06-ftpbin.png">
</p>

Or if you want to use `wget`, of course you can.
```
$ wget --no-passive-ftp -r ftp://anonymous:anonymous@10.10.10.98//
```
> the `--no-passive-ftp` argument disables the use of passive FTP transfer mode (it has something to do about firewall and stuff because sometimes the transfer doesn't work using the passive mode) and the `-r` argument means that we want to download the files recursively. 
> 
> Note that the default depth of `-r` is 5, so you have to change this as needed if you want to use it later in real life.
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/01/07-wgetway.png">
</p>

To ensure that both ways work correctly, we can compare the files.
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/01/08-compare.png">
</p>

Everything's good, let's proceed by checking what kind of files they are.
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/01/09-filecheck.png">
</p>

Look's like we need to go Windows to open these files. But before that, why don't we check the Telnet service?
<br>
<br>

### Telnet
You can use the `telnet` command out of the box.
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/01/10-telnetcheck.png">
</p>

It seems we don't have enough credentials to continue this way. Alright, to Windows then.
<br>
<br>

### The files
We can try to open them with the help of online tools (as for the .zip, it can even be done manually on Linux systems) but for the best experience, it would be more practical to access them in their natural environment, Windows.

The **Access Control.zip** is password protected, and we simply don’t have the power and time to crack it manually, leaving us with the other file. 

The **backup.mdb** is a Microsoft Access file, as been reported by the `file` command. We can open it with MS Access and see if there's anything of importance.
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/01/11-access.png">
</p>

This looks nice. Remember the folder "Engineer"? It was where we got the password protected zip.
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/01/12-zipopen.png">
</p>

The file contains a MS Outlook Data File, Microsoft's e-mail service. Opening it in Outlook gives us another information.
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/01/14-outlookpassword.png">
</p>

Right on! There's the credential for Telnet.
<br>

Username	: **security**

Password	: **4Cc3ssC0ntr0ller**
<br>
<br>

### Telnet
