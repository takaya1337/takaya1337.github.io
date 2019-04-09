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

This is why SMB has more attack vectors to offer than FTP, because it interacts more with files (through IPC or Inter-Process Communication) and this is also how the [EternalBlue](https://research.checkpoint.com/eternalblue-everything-know/) is executed in Microsoft's implementation of SMB called CIFS or Common Internet File System (not to be confused with Linux/UNIX's SMB implementation: Samba).

The good thing is, the exploit mentioned above has nothing to do with this box. But I did mention something about being able to enumerate SMB without credentials.

There are many tools to enumerate SMB shares (shared files or resources). One of the best is **[nullinux](https://github.com/m8r0wn/nullinux)**.
```
python3 nullinux.py -a 10.10.10.100
```
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/02/03-nullinux.png">
</p>

From this tool, we can see that there is a share that allows a null session (similar to anonymous FTP login), **Replication**, even though it's just a read-only access.

You can connect to the share with `smbclient`.
```
smbclient //10.10.10.100/Replication/ -I 10.10.10.100
```
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/02/04-smb.png">
</p>

Now that you gained access to a share, you can take your time in exploring the insides and enumerate everything. However, to minimize the scope of your search, you can use this article as a reference: **[Attack Methods for Gaining Domain Admin Rights in Active Directory](https://adsecurity.org/?p=2362)**.

The article mentioned some XML files which normally would contain credentials. The box happens to have one.
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/02/04-smbexplore.png">
</p>

Because the **Replication** share allows read-only access, you can download the file as usual with the `get` command.

Let's check the **Groups.xml** file.
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/02/05-groupxml.png">
</p>

The highlighted part is the juiciest bit of this XML file. If you read the article a little bit more, you will find that the **cpassword** tag is actually the user's password (you can even find the username in the file too), encrypted with AES-256 bit.

By itself, the encryption should be enough, but somehow, with reasons unkown, Microsoft released the **[encryption key](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-gppref/2c15cbf0-f086-4c74-8b70-1f2fa45dd4be)**. In other words, the GPP password is ultimately hackable.

If you're using Kali Linux, `gpp-decrypt` is already installed by default.
```
gpp-decrypt [insert the encrypted password here]
```
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/02/06-gpp.png">
</p>

Username    : **active.htb\SVC_TGS**

Password    : **GPPstillStandingStrong2k18**
<br>

Good enough, there goes the user's share. Let's connect.
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/02/07-smbuser.png">
</p>

The flag is where it should be.
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/02/08-userflag.png">
</p>

That's the flag.
