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


