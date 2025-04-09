# Road Write-up

## Initial Enumeration

The only initial information provided is an IP address. We begin by conducting an Nmap scan to identify open ports and services:

```bash
nmap -sC -sV -v {target_ip}
```
### Scan Results:
![image](https://github.com/user-attachments/assets/8a900757-5405-458e-ba6b-bf2435ee8803)

No anonymous login anywhere venturing to the webpage brings us 

## Webpage Enumeration

Looking around and clicking on things eventually brings us {ip_address}/v2/admin/login.html

Registering for an account gives us the ability to login! Now we have access to a dashboard for ordering.

Moving around the website three important things can be found.

1. Upload profile picture (Can't use unless we are admin!)
2. The admin email address: admin@sky.thm
![image](https://github.com/user-attachments/assets/ede4905b-decc-4c73-a6ce-d51263151496)
3. Reset Password feature.
![image](https://github.com/user-attachments/assets/66cb4b96-e919-4db7-ad66-8d8b3109071f)

### Webpage Password Reset

Using burpsuite we may be able to capture a password reset request for another user, the admin account is a perfect candidate.

![image](https://github.com/user-attachments/assets/98adb69b-ed3b-477d-bf93-87894018be66)

And with that, the admin@sky.thm account has been compromised! Using the profile upload feature only available to admin may be a way to get a shell running.

Uploading a php reverse shell, the location can be found in /v2/profileimages/shell.file

Starting a nc listener, the system has been compromised as user www-data! Checking for other users reveals the system is using mongodb and mysql.

![image](https://github.com/user-attachments/assets/e7844299-64b8-4c51-a312-fe604d74e915)

### Shell Stabilization

```bash
python3 -c 'import pty;pty.spawn("/bin/bash")'
CTRL+Z
stty raw -echo; fg
export TERM=xterm
```
### First Flag

![image](https://github.com/user-attachments/assets/e72c2721-d40c-4231-ada1-0d5383f6f59c)

## Privilege Escalation

### Mongodb exploitation

```bash
ss -tulnp
```

![image](https://github.com/user-attachments/assets/22ef5b29-8fd0-4885-82df-94e2458920d5)

After switching users, we can begin gaining root privileges using LD_PRELOAD enviroment.

![image](https://github.com/user-attachments/assets/908cced0-71c7-457b-88ce-987059ec74a0)

![image](https://github.com/user-attachments/assets/ad7a6de0-9fc2-45c9-935b-53cae8682a25)


### Notes:
- Priv Esc techniques still rusty
- Mongdb is entirely new
