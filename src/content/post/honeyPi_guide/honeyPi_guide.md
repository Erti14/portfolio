---
title: Deploying a Samba Honeypot on a Raspberry Pi Guide
description: Catch Attackers by Tricking them. 
publishDate: "2025-04-26T16:00:00Z"
---

![dashboard.png](/portfolio/email_alert.png)
*Figure 1: Email alert notification showing detected intrusion attempt*

## Introduction

Cybersecurity threats are constantly evolving, and organizations need proactive ways to detect potential attacks. One effective method is deploying a **honeypot**, a system designed to attract and log unauthorized access attempts.

In this guide, we'll set up a **Samba-based honeypot** on a Raspberry Pi that mimics a Windows file share. Attackers trying to access it will trigger automated **email alerts** containing their IP addresses, helping you monitor suspicious activity in real time.

---

## Why Use a Samba Honeypot?

A Samba honeypot allows you to:

✅ **Simulate a Windows File Share** — Attract unauthorized users by mimicking a real network resource  
✅ **Monitor Suspicious Activity** — Log interactions and identify potential threats  
✅ **Automate Email Alerts** — Get real-time notifications when someone accesses the honeypot

By the end of this tutorial, you'll have a fully functional honeypot running on your Raspberry Pi, capable of sending alerts whenever an unauthorized user tries to access shared files.

---

## What You'll Need

- Raspberry Pi (Model 3B+ or later recommended)  
- Raspberry Pi OS (64-bit preferred)  
- Internet connection & SSH access  
- Installed tools: `samba`, `incron`, `msmtp` for email alerts

---

## Step 1: Update and Install Required Packages

Before installing any software, update your system:

```bash
sudo apt update && sudo apt upgrade -y
```

Now install the required dependencies:

```bash
sudo apt install python3 python3-pip python3-venv -y
sudo apt install samba -y
sudo apt install rsyslog -y
sudo apt install incron -y
sudo apt install msmtp -y
```

Or install everything at once:

```bash
sudo apt install python3 python3-pip python3-venv samba rsyslog incron msmtp -y
```

**What are these packages for?**

- `python3`, `pip`, `venv` → Required for OpenCanary  
- `samba` → Creates the shared network folder (honeypot)  
- `rsyslog` → Handles logging  
- `incron` → Monitors the log file and triggers alerts  
- `msmtp` → Sends email notifications

---

## Step 2: Install and Configure OpenCanary

1. Create a directory (optional):

```bash
mkdir opencanary && cd opencanary
```

2. Set up a virtual environment:

```bash
python3 -m venv venv
source venv/bin/activate
```

3. Install OpenCanary:

```bash
pip install opencanary
```

4. Generate default config:

```bash
opencanaryd --copyconfig
```

> Config file path: `/etc/opencanaryd/opencanary.conf`

5. Enable SMB logging in the config file:

```bash
sudo nano /etc/opencanaryd/opencanary.conf
```

Search for `smb` settings and ensure these lines are present:

```json
"smb.auditfile": "/var/log/samba-audit.log",
"smb.enabled": true,
```

---

## Step 3: Configure Samba

1. Create a shared folder:

```bash
mkdir /files
chmod 755 /files
```

2. Add a decoy file:

```bash
cd /files
touch backup.sql
```

3. Edit Samba config:

```bash
sudo nano /etc/samba/smb.conf
```

Replace contents with:

```ini
[global]
 workgroup = WORKGROUP
 server string = NBDocs
 netbios name = WIN-DC01
 dns proxy = no
 log file = /var/log/samba/log.all
 log level = 0
 max log size = 100
 panic action = /usr/share/samba/panic-action %d
 server role = standalone
 passdb backend = tdbsam
 obey pam restrictions = yes
 unix password sync = no
 map to guest = bad user
 usershare allow guests = yes
 load printers = no
 vfs object = full_audit
 full_audit:prefix = %U|%I|%i|%m|%S|%L|%R|%a|%T|%D
 full_audit:success = flistxattr
 full_audit:failure = none
 full_audit:facility = local7
 full_audit:priority = notice

[myshare]
 comment = Sensitive Data
 path = /files
 guest ok = yes
 read only = yes
 browseable = yes
```

---

## Step 4: Configure Logging (rsyslog & incron)

1. Set up rsyslog:

```bash
sudo nano /etc/rsyslog.conf
```

Add this line:

```
local7.*  /var/log/samba-audit.log
```

Then:

```bash
sudo touch /var/log/samba-audit.log
sudo chown root:adm /var/log/samba-audit.log
sudo systemctl restart rsyslog
```

2. Set up `incron`:

Edit incrontab:

```bash
incrontab -e
```

Add this line:

```
/var/log/samba-audit.log IN_MODIFY /usr/local/bin/send-email-notification.sh
```

---

## Step 5: Set Up Email Alerts

Create the script:

```bash
sudo nano /usr/local/bin/send-email-notification.sh
```

Paste:

```bash
#!/bin/bash
LOG_ENTRY=$(tail -n 1 /var/log/samba-audit.log)
ATTACKER_IP=$(echo "$LOG_ENTRY" | awk -F '|' '{print $2}')
FORMATTED_LOG=$(echo "$LOG_ENTRY" | awk -F '|')

echo -e "Subject: Samba Honeypot Alert\n\nSuspicious access detected from IP: $ATTACKER_IP\n\nLog Entry:\n$LOG_ENTRY" | msmtp recipient@example.com
```

Make it executable:

```bash
sudo chmod +x /usr/local/bin/send-email-notification.sh
```

---

## Step 6: Configure MSMTP (Email Sending Service)

Find the MSMTP config location:

```bash
msmtp --version
```

Edit the config file:

```bash
nano /root/.msmtprc
```
Paste the following:

```bash
defaults
auth on
tls on
tls_trust_file /etc/ssl/certs/ca-certificates.crt
logfile ~/.msmtp.log
account gmail
host smtp.gmail.com
port 465
tls_starttls off
from your_email@gmail.com
user your_email@gmail.com
password *************
account default: gmail
```

Note: Gmail requires an App Password, which you can generate in Google Account settings under App Passwords.

## Step 7: Start OpenCanary
Run the following command to start OpenCanary:

```bash
opencanaryd --start --uid=nobody --gid=nogroup
```

## How to Test?
1. On a Windows machine, open Explorer, go to Network and enter:
\\[RraspberryPi_IpAddress]

In my example it is: \\192.168.100.12

![access_folder.png](/portfolio/access_folder.png)
*Figure 2: Accessing the shared folder*

Access the Shared Folder.

2. Try to open or copy files.
3. You should receive an email alert with the attack details.

![email_alert.png](/portfolio/email_alert.png)
*Figure 3: Real-time email alerts for detected security threats*

## Step 8: Block Traffic from the Attacker
Edit the notification script:

```bash
nano /usr/local/bin/send-email-notification.sh
```

Add the following line at the end:

```bash
# Block the attacker's IP
iptables -A INPUT -s "$ATTACKER_IP" -j DROP
```

After this, access the shared folder again and you shoul receive the following message:

![block.png](/portfolio/block_attacker.png)
*Figure 4: Automatic blocking of detected malicious IP addresses*

## Conclusion

You now have a fully operational **Samba honeypot** with automated email alerts running on your Raspberry Pi. This setup can help detect unauthorized attempts to access your network resources and improve your overall security posture.

For further improvements, consider integrating with a logging and SIEM system like **Wazuh** or **Security Onion**, or add geolocation for attacker IPs.

Stay safe!
