# Bounty Hacker Write-up

  An initial network scan gives us three possible ports.

  #### $ nmap -sC -sV -oN 10.10.147.114

  1. Port 21: ftp w/ anonymous login option
  3. Port 22: ssh
  4. Port 80: http

![image](https://github.com/user-attachments/assets/1c7fd2cb-1b5c-4351-b8c8-b62bd4920c3e)

Opening the http page gives us a list of possible usernames we could use to login to one of the other services.

![image](https://github.com/user-attachments/assets/b7e51f5a-e05e-499c-9e33-3273553f87ca)

Using gobuster to enumerate the server but found nothing of interest, including the possible usernames.

Logging into the ftp server using anonymous there is two files we can download.

![image](https://github.com/user-attachments/assets/a8d88b1a-bb08-4212-b41f-6d93a9022f91)

task.txt is a short todo list written by lin. Giving another possible username for SSH.
locks.txt appears to be a list of possible passwords.

Using the above information we can use Hydra to bruteforce the password to SSH with the username and the possible passwords

#### $ hydra -l lin -P locks.txt ssh://10.10.147.114

Hydra has given us the username and password to login to SSH with.

Logged in as lin we are immediatly given the first flag in user.txt

![image](https://github.com/user-attachments/assets/1eada02b-d510-48bb-9156-f3b24105c4bf)

Searching for root.txt provides nothing, elevation of privilege is the next step.

lin only has permissions to run tar as sudo so we can use gtfobins to obtain a root shell with tar.

#### $ sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh

Now with root access obtained we find and open root.txt giving us the final flag for this box.

![image](https://github.com/user-attachments/assets/23224e7d-6236-42c8-a847-726f70da16a9)







  

