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
This is my first Linux box, and I only played it because my friend got stuck a little bit so I decided to try it out.
```
$ nmap -p 1-65535 -T4 -A -v 10.10.10.117
```
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/03/irked1-nmap.png">
</p>

The box has **SSH**, **Apache 2.4.10**, and **UnrealIRCd**. Of all the services, only one service can be enumerated without any credentials.
<br>
<br>
<br>

### The Website
<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/03/irked1.2-web.png">
</p>

The website doesn't have anything except this image, and a very important information about an **IRC** service that's "almost working".

Let's see what we can find from it.
<br>
<br>
<br>

### IRC
IRC is a protocol that allows multiple client to chat in real-time. It was created by Jarkko (also known as WiZ) somewhere in the 90's. I once saw a thread dedicated for him in an image board, it was quite nice. IRC may not be as popular now as it was back then, but it was significant back in the day, or so I've heard.

You can check the IRC by connecting to it, but as the website says, it's just "almost" working.
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/03/irked2-irc.png">
</p>

Aside from connecting and probing some channels, I think the other features weren't working.

So I searched for a vulnerability specific to the service.
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/03/irked2.3-infocve.png">
</p>

The CVE is about a **modified version of UnrealIRCd that has been tampered with a backdoor and redistributed back to the wild**.

Somehow the backdoor'ed version got more popular than the official around the time.

And the best thing is, there is a **Metasploit** module for this exploit.
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/03/irked2.2-irccve.png">
</p>

To be honest, `msf` made it too easy.
<br>
<br>
<br>

### Metasploit My Life
Probably the hardest part about using Metasploit is the installation part, but if you use Kali Linux, everything will be as smooth as butter.
```
msf > use exploit/unix/irc/unreal_ircd_3281_backdoor
msf > set RHOST 10.10.10.117
msf > set RPORT 6697
msf > exploit
```
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/03/irked3-msf.png">
</p>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/03/irked4-ircshell.png">
</p>

Now that you've got the initial foothold, as usual, enumerate. I've heard some people even got root before user. But personally, I think it's more fun to do it as the problemsetter intended so you can learn a little bit more.
<br>
<br>
<br>

### Another User
If you explored thoroughly, you should've found the user flag in **djmardov**'s home directory.

Unfortunately, your shell belongs to **ircd** so you can't read the flag. So there is no other option except to get the credentials for **djmardov**.

In a Linux box, one way of enumerating is to search the command history located in the **.bash_history** file.
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/03/irked5-enum.png">
</p>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/03/irked6-history.png">
</p>

By checking **ircd**'s **.bash_history** you can see the previous commands performed by this user, and you can see that it interacts with **djmardov**'s Document directory. It also read a file called **.backup**.
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/03/irked8-steg.png">
</p>
<br>
<br>

### Steg is for Steganography
The password above **is not** the password for SSH.

If you read the sentence very very very carefully, there is a word that seems out of place. What does "elite steg" have to do with anything? Not to mention the password looks like a modified Konami Code. Everything seems misleading.

At times like these, it's alright to check the **[forums](https://forum.hackthebox.eu/discussion/1278/irked/p4)** for a hint or two :)

I never asked a question myself, because the information I need most likely has been asked and provided by someone else, and asking a previously answered question will make me look like a retard (at least that's how I see people who did). So I just read.

Anyway, you will notice that lots of people say something about the box being more CTF oriented than real-life case, and it's kinda true.

The **.backup** provides a password for **steganography**, roughly the art of hiding something in a media. And the only media so far is...
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/03/irked1.2-web.png">
</p>

I'm not even remotely good at this kind of technique (due to its more tool-oriented nature instead of concept-oriented), but I've had one or two CTF challenges about it. One of the tools that requires a password for extracting files from an image is **steghide**, and it's also the first tool that came to my mind.
```
$ steghide extract -sf irked.jpg -p UpupDOWNdownLRlrBAbaSSss
```
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/03/irked10-djpass.png">
</p>

Now that, is the credential we're looking for.
<br>

Username	: **djmardov**

Password	: **Kab6h+m+bbp2J:HG**
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/03/irked11-user.png">
</p>

There goes the user flag.
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
There are many ways of enumerating a Linux machine, and there are also many kinds of scripts provided online.

Scripts may help in a real pentesting job, but one thing that will definitely help is understanding the concept.

If you search for "Linux enumeration", chances are you have already read this **[blog](https://blog.g0tmi1k.com/2011/08/basic-linux-privilege-escalation/)**.

That blog gives a very nice enumeration cheat sheet and a little bit of information as well. However, what I want to highlight for this machine is the **Advanced File Permission in Linux**.
<br>
<br>
<br>

### Basic Linux File Permission
There is a famous saying about Linux systems that says,

"Everything is a file".

Even though this is a bit of an oversimplification of the matter, the saying is true to some extent. Everything does appear as files in Linux, and with it comes a handy little attribute called **permission**.

This permission manages who can access the file and how far can they access it. It is common in Linux to limit everything to everyone, thus implementing the **[the Principle of Least Privilege](https://en.wikipedia.org/wiki/Principle_of_least_privilege)** in a way.

The permission works like this:
```
drwxrwxrwx 2 john john	  0 Jul 14 20:00 test-dir
-rw-r--r-- 1 root root	 32 Aug 10 00:00 root.txt
```
> I think it will be easier to explain it with colors.

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/03/perm.png">
</p>

Each file (yes, directories are files too) has three `rwx` attributes that determines whether the file is `readable`, `writable`, and `executable` by the file's **owner** (the user under whom the file was created), **group** (by default will refer to the user's primary group), and **others** (everyone else).

When you do:
```
$ chmod 644 root.txt
```

here's what happens:
<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/03/perm2.png">
</p>

The number that `chmod` takes as the first argument is called an octal number, or base-eight (which means the "ten" is eight, instead of our usual base-ten/decimal where the "ten" is, in fact, ten). Octal number is used because it denotes the maximum sum for three binary digits (4 + 2 + 1 = 7 is the maximum number).

The octal number will then be split, and each will be converted to binary digits where the three attributes `r`, `w`, and `x` will be checked if their "switch" is turned on or not. If yes, then the permission for the respective attribute and position is granted.

Linux is very strict when it gets to security attributes and not even **root** can execute a file unless the permission is changed to allow it to be executable, but of course **root** can easily fix that. It will be a different matter, however when a normal user is faced with the same situation. The user can only change the permission if he owns the file, but allowing it to be writable, readable, and executable by everyone defeats the very purpose of security.

You should understand by now that most people's favorite permission from `chmod 777` is **very dangerous** and should be avoided whenever possible.
<br>
<br>
<br>

### On to SUID, SGID, and Sticky Bit
There are more to files than just the basic permissions above. Let's say that you're a lead sysadmin working with a few sysadmins on a project. Due to some circumstances, you are the only one allowed to have a `sudo` access. Now what if, your fellow sysadmin needed to execute a program as root, manually, consistently but in random interval, but the user itself cannot be granted root access? Not to mention you have to go to meetings so you're not always available, and lending your credential is not even an option.

At a time like this, it would be a [good] idea to use **SUID** on your program.

**[SUID](https://www.linux.com/blog/what-suid-and-how-set-suid-linuxunix)** or Set owner User ID is one of the three special permissions that if set on a file will enable anyone to execute the file as the file's owner, the dark green one on my picture above.

**[SGID](https://www.linux.com/blog/what-sgid-and-how-set-sgid-linux)** is just about the same, but switch the user with group instead.

**[Sticky Bit](https://www.linux.com/blog/what-sticky-bit-and-how-set-it-linux)** is a permission that will only let a user delete its own files in the directory where the Sticky Bit is set (most of the time, Sticky Bit is set on a directory).

You can find more information about the special permissions **[here](https://www.thegeekdiary.com/what-is-suid-sgid-and-sticky-bit/)**.

Getting back to the first paragraph, right now your fellow would be able to execute that program as root anytime he needs without you giving up the credential. Everyone's happy. Until he found out how to modify the program to give him a shell everytime he executes it. Guess under what user the shell is run with? **root**.
<br>
<br>
<br>

### Just a Common Binary, with a Special Permission
It is common to search for SUID-enabled program when someone is looking for a privilege escalation attack surface. But that doesn't mean every program with SUID-enabled is dangerous. Some of the basic utilities in Linux are SUID-enabled because they need to be set that way to function as intended. Only when the implementation is wrong (which is to say, most of the time) the program becomes dangerous.

To search for SUID-enabled program you can use `find`:
```
$ find / -perm -u=s -type f 2>/dev/null
```
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/03/irked12-suid.png">
</p>

Of all the programs listed above, try to find one that doesn't belong. I got it quite fast because I compared the result with my computer's.
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/03/irked13-anomaly.png">
</p>

After some trial and error, I found that this is the weird program. It shows the result of `who` command and it tries to execute something with `sh` in **/tmp/listusers**.

To make sure, I exfiltrated the binary to my computer. You need `openssh-server` to run so you can `scp` the binary straight to your **tun0** IP address from your **.ovpn** file.
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/03/exfiltrate-program.png">
</p>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/03/file.png">
</p>

Indeed, it's a 32-bit executable. You can consult your disassembler to find out how it works.
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/03/program.png">
</p>

See how simple it is? It executes `who`, set the SUID-bit as **root**, and executes anything on **/tmp/listusers**.

We only have to put a shell command on the location.
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/03/irked15-privesc.png">
</p>

Really loved the **#**!