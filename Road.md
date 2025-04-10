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
<br/><br/>
![image](https://github.com/user-attachments/assets/ede4905b-decc-4c73-a6ce-d51263151496)
<br/><br/>
4. Reset Password feature.
<br/><br/>
![image](https://github.com/user-attachments/assets/66cb4b96-e919-4db7-ad66-8d8b3109071f)
<br/><br/>

The reset password feature autofills the relevant content. A request capture could give information on how to exploit this process.
### Webpage Password Reset

![image](https://github.com/user-attachments/assets/98adb69b-ed3b-477d-bf93-87894018be66)

Using burpsuite we may be able to capture a password reset request for another user, the admin account is a perfect candidate.

#### Reset Request
- The form allows for manipulation by submitting a username (uname) to reset, a new password (npass), and a confirmation of the new password (cpass).

#### Server Response:
The server responds with confirmation of the password change!

### Admin Exploitation
> **The ability to reset a password for other accounts is a severe **authentication bypass** issue. The server didn't verify that the owner of the account being reset, meaning any user can reset a password by providing an email address.**

With the admin@sky.thm account compromised, there is some additional functionioalty to continue exploiting the server with. Using the **profile upload** feature only available to admin could provide a means of obtaining a **reverse shell**.

Uploading a php reverse shell, the location can be found in /v2/profileimages/shell.file which is in a comment on the webpage.
<br/><br/>
1. Creating the payload for the reverse shell can be found at ![link](https://github.com/pentestmonkey/php-reverse-shell).
2. Starting the listener to catch the shell
<br/><br/>
```bash
nc -lvnp 8080
```
3. Open /v2/profileimages/shell.file in a browser to send the shell.

And with that our reverse shell is up and running, we've gained access as www-data user.
<br/><br/>
## Shell Stabilization
<br/><br/>
```bash
python3 -c 'import pty;pty.spawn("/bin/bash")'
CTRL+Z
stty raw -echo; fg
export TERM=xterm
```
### First Flag
The first flag requires no further privilege escalation!
![image](https://github.com/user-attachments/assets/e72c2721-d40c-4231-ada1-0d5383f6f59c)

## Privilege Escalation

### System Enumeration

Working through some basic information gathering, the /etc/passwd file shows the system is has MongoDB.

![image](https://github.com/user-attachments/assets/e7844299-64b8-4c51-a312-fe604d74e915)

### Mongodb exploitation

```bash
ss -tulnp
```
ss stand for socket statistics and gives information about network sockets.
Options Breakdown: -tulnp

-t : Show TCP sockets
-u : Show UDP sockets
-l : Show only listening sockets (services waiting for incoming connections)
-n : Show numeric addresses (don’t try to resolve hostnames or service names)
-p : Show the process using the socket (requires root privileges to see all)

![image](https://github.com/user-attachments/assets/22ef5b29-8fd0-4885-82df-94e2458920d5)

```bash
show databases
```
1. admin
2. backup
3. config
4. local

```bash
use backup
```
```bash
show collections
```
Looking at the backup database, there is a collection containing user info for www-developer

After switching users, we can begin gaining root privileges using LD_PRELOAD enviroment.

![image](https://github.com/user-attachments/assets/908cced0-71c7-457b-88ce-987059ec74a0)

![image](https://github.com/user-attachments/assets/ad7a6de0-9fc2-45c9-935b-53cae8682a25)


### Notes:
- Priv Esc techniques still rusty
- Mongdb is entirely new
