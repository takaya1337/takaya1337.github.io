---
title: Access
layout: default
---

# Access
<br>
<br>
<br>
<br>

<p align="center"> 
<img src="https://takaya1337.github.io/htb/en/01/assets/access.png">
</p>
<br>
<br>

#### OS    : Windows
#### IP    : 10.10.10.98
#### Maker : egre55
<br>
<br>
<br>
<br>

## User
<br>

### NMAP
This is the first box that I worked on, ever. We all gotta start somewhere. Should I make future HTB write-ups, you can compare them to this one to see how much I improved (and you as well I hope).
<br>

The first thing to do when given an IP is to do `nmap` on the machine, to see what kind of attack might be used. The magic syntax is:

```
$ nmap -p 1-65535 -T4 -A -v 10.10.10.98
```
to scan all open ports and services. Here is the output.
<br>
