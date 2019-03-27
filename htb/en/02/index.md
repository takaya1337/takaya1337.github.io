---
title: Active
layout: default
---

# Access

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
By now you should be familiar with what we're going to do with the box. Hopefully the previous write-up gives you enough explanation to start on your own. 
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

This box has a lot more open ports than the previous one, but focus your attention to port 445 first. Why? Because if you searched for **microsoft port 445** or **microsoft-ds** you will stumble upon the infamous SMB service. 

The real reason you should put more thoughts into this service first is due to the reason that no other service in this machine allows enumeration without credentials (much like the previous anonymous FTP) except SMB.
<br>
<br>

### SMB
Maybe some of you have heard the name before. SMB was famous for a certain critical Windows exploit at some point. However, I've never actually used SMB up until now, so it's a new experience for me too.

SMB or Server Message Block is a client-server protocol used to share files (not limited to "files", may also include printers) between two or more machines (at least one server and one client) over network.

The most noticable difference between SMB and FTP is that SMB is a **network sharing protocol** which means its primary purpose is to **share** files over network while FTP, as the name implies, is a **file transfer protocol**, it's mainly designed to just **transfer** files and not interact with them. 

This is why SMB has more attack vectors to offer than FTP, because it interacts more with files (through IPC or Inter-Process Communication) and this is also how the [EternalBlue](https://research.checkpoint.com/eternalblue-everything-know/) is executed in Microsoft's implementation of SMB called CIFS or Common Internet File System (not to be confused with Linux/UNIX's implementation: Samba).

The good thing is, the exploit mentioned above has nothing to do with a 20-points box. I did mention something about being able to enumerate SMB without credentials or creating **Null Sessions**.

There are many tools to enumerate SMB shares (shared files or resources). One of the best is **[nullinux](https://github.com/m8r0wn/nullinux)**.
```
python3 nullinux.py -a 10.10.10.100
```
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/02/03-nullinux.png">
</p>

With this magic tool we got a lot of information on the SMB server, but especially, we know where to look next.
