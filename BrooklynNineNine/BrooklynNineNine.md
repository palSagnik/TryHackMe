## Description

This room is aimed for beginner level hackers but anyone can try to hack this box. There are two main intended ways to root the box. 
If you find more dm me (the creator of this box, not me lol) in discord at Fsociety2006.

## 1. Reconnaissance

```
nmap -sV 10.10.102.110
```


```
Host is up (1.2s latency).
Not shown: 970 filtered tcp ports (no-response), 30 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
...
```




After finishing this box, I referred to a few writeups, where masscan is used as an alternative to nmap, because nmap takes up quite a bit of time. This might not
be noticeable in easy boxes, but will further change for harder ones.
I will also try to follow the same methods.


Running gobuster now


```
gobuster dir -url http://10.10.102.110 -w usr/share/wordlists/dirb/common.txt -t 100
```



-t is for number of threads to be checked together here it is 100




The result is just /index.html. After checking the source code there is a mention of "Have you heard of Steganography?" so not so subtle hint towards
Steganography.



I have missed taking screenshots of the initial steps, honestly I was fighting sleep at this point. But will try to avoid this. 




Before we proceed with steg, check the next service we found on the nmap scan that is ftp.



## 2. This time it is Steg




As we know ftp allows us to download files over the network.

```
ftp 10.10.102.110
```


You can log in with anonymous as usual and for the password prompt just press enter. There is a file called note_to_jake.txt and download it with get command.

```
get note_to_jake.txt

 ```

If you face some kind of error regarding ENTERING EXTENDED PASSIVE MODE just type passive as a command and press enter and then type the previous get command 
again. This should work.


Contents of the .txt file

```
will put it no worries
```


Sounds like some kind of weak password, and it is a Steg challenge maybe? So a password cracker? Could be, we will get to the point while we move forward.

Now the actual steg part, 
We first download the image from the website by the wget command


```
wget http://10.10.102.110/brooklyn99.jpg
```

First time I always proceed with steghide

```
steghide --extract -sf brooklyn99.jpg
```

It prompted me for a password, and I run ran the stegcracker command on it. If there is no wordlist supplied to stegcracker it would use the default kali
rockyou.txt wordlist in /usr/share/wordlists/rockyou.txt
 


<img width="1260" alt="Screenshot 2023-08-16 at 6 31 14 PM" src="https://github.com/palSagnik/TryHackMe/assets/112897062/620d799c-ac6b-472d-8bf8-72ad9aca1efa">


And reading the .out file,  we have our password



<img width="1243" alt="Screenshot 2023-08-16 at 6 31 23 PM" src="https://github.com/palSagnik/TryHackMe/assets/112897062/4043c796-ac87-4e7c-8985-ac51fade9dc0">




## 3. Bro, just SSH and root



<img width="1260" alt="Screenshot 2023-08-16 at 6 03 47 PM" src="https://github.com/palSagnik/TryHackMe/assets/112897062/a7dea860-072d-4918-ad70-d1e817d6f716">



Now that I am in, read the user.txt file first and then move on to the root part.



<img width="1254" alt="Screenshot 2023-08-16 at 6 31 39 PM" src="https://github.com/palSagnik/TryHackMe/assets/112897062/182cb677-85c5-4f13-b65c-78e9eed0d2c9">



Checking for the privileges holt has on this host 


```
sudo -l
```

This command lists the privileges the particular user is allowed to run.
These privileges are stored in the etc/sudoers file
This file helps to give certain privileged access to only a certain number of hosts.


<img width="1254" alt="Screenshot 2023-08-16 at 6 25 46 PM" src="https://github.com/palSagnik/TryHackMe/assets/112897062/66fc3de9-376e-4839-98af-e6674d07dc6d">




Right now the most you can do is fall at the feet of GTFO and I did just that which lands me with the code

```
sudo nano -s /bin/sh
```

After this just press Ctrl + T and you would have root access.



The root.txt file was obviously in /root/root.txt.

And this box is over. It was really just ssh and root :)


Nine-Nine!
