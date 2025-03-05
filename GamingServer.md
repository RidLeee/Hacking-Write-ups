# GamingServer Write-Up

## Initial Enumeration

The only initial information provided is an IP address. We begin by conducting an Nmap scan to identify open ports and services:

```bash
nmap -sC -sV -v {ip_address}
```

### Scan Results:
1. **Port 22** – SSH service detected
2. **Port 80** – HTTP service detected

## Web Enumeration

Navigating to the webpage hosted on port 80, we discover a gaming-themed website with multiple links. One of the links is labeled **Uploads**, which might present an opportunity for a reverse shell.

While inspecting the homepage source code, we find a comment containing a potential username for SSH access.

### Further Enumeration with Gobuster

To discover hidden directories, we run Gobuster:

```bash
gobuster dir -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -u {ip_address}
```

This reveals two key directories:
1. **Uploads**
2. **Secret**

## Exploring the Uploads Directory

The **Uploads** directory contains three files:
- `meme.jpg` – A standard image file
- `manifesto` – A text file containing a manifesto (What on earth?)
- `dict.lst` – A potential password list

## Exploring the Secret Directory

The **Secret** directory contains a private RSA key file, which could be useful for SSH authentication.

## Cracking the RSA Key

We download `secretKey` and `dict.lst` for further analysis. Using John the Ripper, we attempt to crack the private RSA key:

1. Convert the private key to a format usable by John:
   ```bash
   sudo /usr/share/john/ssh2john.py secretKey > hashedkey
   ```

2. Use `dict.lst` to brute-force password for the hashed key:
   ```bash
   sudo john hashedkey --wordlist=dict.lst
   ```

3. Retrieve the cracked password:
   ```bash
   john hashedkey --show
   ```

## SSH Access

Since SSH requires appropriate file permissions for private keys, we adjust the key's permissions:

```bash
chmod 600 secretKey
```

Using the username `john`, found in the website's HTML comments, we attempt to log in via SSH:

```bash
ssh john@{ip_address} -i secretKey
```

After entering the recovered password, we successfully gain access to the system.

## Next Steps: Privilege Escalation

At this stage, privilege escalation is required to gain further access. This is an area I'm currently trying to improve, root access was gained but not without an explanation.

---
### TODO:
- Improve Linux privilege escalation techniques
- Learn linpeas and how it's transfer onto a machine






