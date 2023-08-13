****S****

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
