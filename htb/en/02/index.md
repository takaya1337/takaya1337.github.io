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

This box has a lot more open ports than the previous one, but you should focus your attention to port 445.
<br>
<br>

