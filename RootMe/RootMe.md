## Description
A ctf for beginners, can you root me?

## 1. Reconnaissance

After deploying the machine in TryHackMe, the given IP was 10.10.192.170

Running an *nmap* scan for open ports along with their service and version numbers.

```
nmap -sV 10.10.192.170
```

````
Nmap scan report for 10.10.192.170
Host is up (0.48s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
````

An open port 80, signifies the given IP is a website run by an Apache web server.

The next logical step would be finding the, hidden directories of the website. This is done by *gobuster*.

```
gobuster dir -u http://10.10.192.170 -w /usr/share/dirb/wordlists/common.txt
```

````
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.32.139
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/small.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Timeout:                 10s
===============================================================
2023/08/05 10:54:29 Starting gobuster in directory enumeration mode
===============================================================


/css                  (Status: 301) [Size: 310] [--> http://10.10.32.139/css/]
/js                   (Status: 301) [Size: 309] [--> http://10.10.32.139/js/]
/panel                (Status: 301) [Size: 312] [--> http://10.10.32.139/panel/]
/uploads              (Status: 301) [Size: 314] [--> http://10.10.32.139/uploads/]
Progress: 959 / 960 (99.90%)
````

We find that this website has hidden directories like, /panel and /uploads. Upon visiting then we get that /panel is a file uploader and /uploads is used to open the uploaded file.


## 2. Getting a Shell

The idea behind file upload exploit is to upload a script file, more often a php file.
But in most cases we find that, the uploader has some kind of restrictions to not directly upload .php files.

Here the extensions to try were
*.Php, .PHp, .php2, .php3, .php4, .php5, .pHp, .phtm, .phtml*

The script for the reverse shell used was, from github of pentest-monkey
[here](https://github.com/pentestmonkey/php-reverse-shell)

Writing this in php tags and uploading it as file with *.phtml* extension.
After the upload is shown as successful, go to /uploads and before clicking on the uploaded file, we open a terminal and start our listener. This is important because the above script tries to establish a connection with listener, in the form of socket hence the listener end of the socket must be open for a good connection.

NOTE:
This also worked with a *.php5* extension.

```
nc -lnvp 6969
```

<img width="1267" alt="Screenshot1" src="https://github.com/palSagnik/TryHackMe/assets/112897062/5deac99e-76bf-487e-a109-ad679e35dac1">

We have our reverse shell, and before moving on we will have to stabilise our connection.
This is achieved by 

```
python -c 'import pty; pty.spawn("/bin/bash")'
```

Finding the file user.txt was a bit tricky, because I got the reverse shell access at the / directory.

<img width="925" alt="Screenshot3" src="https://github.com/palSagnik/TryHackMe/assets/112897062/27a46a29-7da0-4ab6-8192-fc42c0c03aaf">


After some thought this was kind of obvious, as most websites follow this path only.


## 3. Being Root?


After getting reverse shell and stabilising the connection, are next step is getting the root access. For this we can take a look at unusual SUID bits set in binaries which shouldn't be so.
The command for finding is 

```
find / -perm /4000 -type f  2>/dev/null
```

<img width="612" alt="Screenshot2" src="https://github.com/palSagnik/TryHackMe/assets/112897062/fbf667a9-f233-4c69-9454-69c74c5eb9b6">


/usr/bin/python never has a SUID bit set, if this is the case this actually means that any command ran using /usr/bin/python interpreter will run with root privileges. This can be understood by seeing the permissions of the /usr/bin/python file.

<img width="1066" alt="Screenshot4" src="https://github.com/palSagnik/TryHackMe/assets/112897062/7bee1dbb-ef97-49e4-940f-7295713315cb">

The command for SUID access using python was found in GTFOBins.

<img width="1066" alt="Screenshot4" src="https://github.com/palSagnik/TryHackMe/assets/112897062/ce029c42-009b-40a3-8505-0a55d626cd20">

And hence we have root access, where any command run within this terminal is run with root privileges.

<img width="1237" alt="Screenshot5" src="https://github.com/palSagnik/TryHackMe/assets/112897062/8294cda3-2b00-4b1a-bdf8-a58f6da183dd">

Yup, this box is now done.
