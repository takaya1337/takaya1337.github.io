---
title: Friendzone
layout: default
---

# Friendzone
<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/04/friendzone.png">
</p>

#### OS            : Linux
#### IP            : 10.10.10.123
#### Maker         : askar
* * *
<br>
<br>
<br>
<br>

## User
<br>

### NMAP
At the time of writing, all easy to medium difficulty Windows boxes are either owned or too hard, so I decided to work on a Linux box (also because a friend was stuck, so so stuck).
```
$ nmap -p 1-65535 -T4 -A -v 10.10.10.123
```
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/04/01-fz-nmap1.png">
</p>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/04/02-fz-nmap2.png">
</p>

There are some services worth noting on, **FTP**, **SSH**, **DNS**, and both **HTTP and HTTPS**. As always, start with services that doesn't require credentials.
<br>
<br>
<br>

### SMB
After checking that the FTP doesn't allow anonymous login, I tried to see if SMB does.
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/04/04-fz-smb.png">
</p>

There are some shares on the SMB and two of them are accessible by null session, **general** and **Development**. It's worth mentioning that the **Development** share is also writable. Let's take the file from **general**.
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/04/05-fz-smbcreds.png">
</p>

It looks like a credential indeed, but somehow everything that requires credentials so far hasn't been accessible by the one we found (FTP, SSH, and the other shares).

That calls for another enumeration.
<br>
<br>
<br>

### Web Server
There isn't much we can take from the website, except an "e-mail address" which will be useful later on.
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/04/03-fz-web.png">
</p>

You can try to use `gobuster` or `disrbuster`, but they won't give anything of importance in this box.

So far I have enumerated SMB, FTP, and HTTP/HTTPS. As usual, SSH is out of the question. That leaves only port 53, or DNS, to enumerate.
<br>
<br>
<br>

### DNS
If you simply Google "port 53 exploit", you will stumble upon some article that explains juicy stuff concerning "Basic DNS Hacking".

I happen to get my information from this **[article](https://resources.infosecinstitute.com/dns-hacking/#gref)**. Although the website is badly formatted and there is a very annoying ad each time you load the page, the article provided what I needed, a crash course on the subject.

Among the types of enumeration listed on the article, one that caught my eye was **DNS Zone Transfer**. The name somehow corresponds with the current box, so I took it as a big hint.
<br>
<br>
<br>

### DNS in a Glance
The article above is enough to give you the ability to proceed on the box, but if you want to know more, keep reading this section.

In a simple term, DNS or _Domain Name System_ is like a phone book which will point the "name of your contacts" to their respective "phone numbers". In this analogy, consider the "name of contacts" as the **domain name** and the "phone numbers" as **IP addresses**.

Everytime you type a domain name, for example www.google.com, the browser will first contact the DNS server (usually provided by your ISP or can also be overridden through your computer's settings), which will then try to "resolve" the domain name into its IP address(es).

The ISP's DNS will try to check in its cache to see if it still have the "answer", if it doesn't, it will query the root name server which will then return the correct TLD or _Top Level Domain_ for the domain name, and so on until the host is reached.

For example: `mail.google.com.`

Will get queried from `.` -> `com` -> `google` -> `mail`
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/assets/04/06-fz-dns-hierarchy.png">
</p>


<br>
<br>
<br>

### To the Transfer Zone


