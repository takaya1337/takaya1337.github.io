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

### A Special Note
To tell you the truth, I didn't finish the rooting process in time, but because the box is making me crazy, I decided to do something I never should've done, I checked another **[write-up](https://medium.com/bugbountywriteup/active-a-kerberos-and-active-directory-hackthebox-walkthrough-fed9bf755d15)**.

Of course I'm still trying to absorb the matter completely, but still the experience is never the same if you're not doing it yourself. Even to me, learning a little bit about the Active Directory and Kerberos is very hard even with my learning pace because I didn't experience everything first-hand.

So my advice is this, unless you're really really really stuck, keep trying on your own. Clues are fine, but a straight answer never is. 

With that being said, enjoy my writeup.
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
$ python3 nullinux.py -a 10.10.10.100
```
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/02/03-nullinux.png">
</p>

From this tool, we can see that there is a share that allows a null session (similar to anonymous FTP login), **Replication**, even though it's just a read-only access.

You can connect to the share with `smbclient`.
```
$ smbclient //10.10.10.100/Replication/ -I 10.10.10.100
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

Because the **Replication** share allows you to read files, you can download the file as usual with the `get` command.

Let's check the **Groups.xml** file.
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/02/05-groupxml.png">
</p>

The highlighted part is the juiciest bit of this XML file. If you read the article a little bit more, you will find that the **cpassword** tag is actually the user's password (you can even find the username in the file too), encrypted with AES-256 bit.

By itself, the encryption should be enough, but somehow, with reasons unkown, Microsoft released the **[encryption key](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-gppref/2c15cbf0-f086-4c74-8b70-1f2fa45dd4be)**. In other words, the GPP password is ultimately hackable.

If you're using Kali Linux, `gpp-decrypt` is already installed by default.
```
$ gpp-decrypt [insert the encrypted password here]
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

That's it for the user.
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
The main star of this machine is **Kerberos**. There are websites on the Internet that could explain this topic better than me. You should take a look, especially **[this one](https://www.roguelynn.com/words/explain-like-im-5-kerberos/)**.

In a sentence, **Kerberos is a ticket-based authentication protocol that involves a third party**.

If you read the explanation from the link above, you should already made a connection between the word "TGS" and the username where we got the user flag before.
<br>
<br>

### About Kerberoast
From **[a website](http://www.harmj0y.net/blog/powershell/kerberoasting-without-mimikatz/)** I stumbled upon while searching for **SVC_TGS**, I found that there are different types of attacks on a Kerberos service and the most famous is of course **Kerberoast**.

According to a very valid **[source](https://files.sans.org/summit/hackfest2014/PDFs/Kicking%20the%20Guard%20Dog%20of%20Hades%20-%20Attacking%20Microsoft%20Kerberos%20%20-%20Tim%20Medin%281%29.pdf)** from Derbycon 2014, Kerberoast is an attack on the ticketing service used in Windows' implementation of Kerberos. However, to use this method one must already have initial access to the system, that is a (normal) user account registered on the Active Directory or AD complete with the password, which we coincidentally aqcuired in the previous step.

Explaining how this works is very tricky because the rabbit hole goes very deep when it comes to "Windows' implementation of [anything]" so I'm going to oversimplify things a bit.

The whole point of this exploit is to crack another user's password and enable further enumeration which could lead to the Golden Ticket exploit with the ultimate goal to gain access as the Enterprise Admin, which means you get to control the whole AD forest.

Again, the goal this time is only to get the Administrator credentials, so...
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/02/09-attacklist.png">
</p>

On the article I referenced before, there are some attacks that should be performed as parts of the Kerberoasting session. Fortunately for us, with the right tool, everything could be as easy as a few commands away.

I used **[impacket](https://github.com/SecureAuthCorp/impacket)** for this.
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/02/10-impacket.png">
</p>

It even has its own feature to request a TGS! Let's use it then.
```
$ impacket/examples/GetUserSPNs.py -request -dc-ip 10.10.10.100 ACTIVE.HTB/SVC_TGS:GPPstillStandingStrong2k18
```
> This will request a TGS (Ticket Granting Service) or The Silver Ticket for the user **SVC_TGS** complete with the password on **ACTIVE.HTB** domain.

<br>
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/02/11-getuserspn.png">
</p>

Alright, we got the hash for Administrator. The password is only a `hashcat` away.
<br>
<br>

### Crack!
Before even starting to crack the hash, first we need to know what kind of hash it is. You can try to check **[hashcat's own page](https://hashcat.net/wiki/doku.php?id=example_hashes)** for a reference.
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/02/13-hashtype.png">
</p>

You have to be careful because there are five entries with "Kerberos" in it, but what we actually wanted is **Kerberos 5 TGS-REP etype 23** because the DC or Domain Controller returns TGS-REP (reply) via the KDC or Key Distribution Center to us. The **etype 23** means that it's using **encryption type 23** which is the code for **rc4-hmac (Rivest Cipher 4)**, which, I quote from this **[website](https://web.mit.edu/kerberos/kfw-4.1/kfw-4.1/kfw-4.1-help/html/encryption_types.htm)**, "is a symmetric stream cipher that can use multiple key sizes".

In other words, just use the one with the ID of 13100.

With **[rockyou.txt](https://github.com/praetorian-inc/Hob0Rules/blob/master/wordlists/rockyou.txt.gz)** we're set to crack this hash.
```
$ hashcat -m 13100 -a 0 tgs /usr/share/wordlist/rockyou.txt
```
> Basically, we want to crack a hash with the type ID of 13100 with the attack mode of zero, which is **straight** (a simple pick-one-and-try-next-if-fails from a wordlist), we're cracking the file called **tgs** with the wordlist **rockyou.txt**.
> I hope that will clarify the syntax.

<br>
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/02/14-hashcat.png">
</p>

And there goes the Administrator's password!
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/02/15-rootflag.png">
</p>

Active Directory is quite fun, right? Oh, and a little bonus if you want to get shell instead of SMB.
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/02/16-trueshell.png">
</p>

That should end this write-up. Hope you're having fun reading it as much as I'm having a hard time researching it :D
<br>
<br>