---
title: Raspberry Pi as a Samba HoneyPot with Automated Alerts
description: one homelab to rule them all!
publishDate: "2025-03-29T12:30:00Z"
---

![logical-network.png](/portfolio/HoneyPi.jpg)

Set up your honeypot in a cost-effective and efficient manner by transforming your Raspberry Pi into a vulnerable Samba file server. This will create an intentionally insecure environment designed to attract potential attackers. By mimicking a typical file-sharing setup with weak security configurations, you can effectively monitor and analyze attack patterns.<br><br><br><br><br>

# **Samba Honeypot on Raspberry Pi**

## **Introduction**
This project demonstrates how to turn a Raspberry Pi into a lightweight honeypot that simulates a Windows file share using Samba. The honeypot is designed to detect unauthorized access attempts, alert administrators in real time, and optionally block malicious IP addresses. It serves as a proactive cybersecurity measure for small networks.

## **Key Features**
- ✅ Simulates a Windows SMB file share to attract intruders  
- ✅ Logs access attempts using OpenCanary  
- ✅ Sends automated email alerts with attacker details  
- ✅ Automatically blocks attacker IPs using iptables  
- ✅ Lightweight and ideal for home or lab environments  

## **Tools & Technologies Used**
- **Hardware**: Raspberry Pi (Model 3B+ or later)  
- **OS**: Raspberry Pi OS (64-bit)  
- **Services & Tools**:
  - `Samba` – for sharing the fake folder
  - `OpenCanary` – for honeypot emulation and logging
  - `incron` – to watch log files for modifications
  - `rsyslog` – for handling log messages
  - `msmtp` – for sending email alerts
  - `iptables` – for blocking suspicious IP addresses

## **Setup Overview**
1. **System Preparation**: Update the Raspberry Pi and install all necessary packages (`Samba`, `OpenCanary`, `msmtp`, `rsyslog`, `incron`).
2. **Configure Samba**: Set up a shared folder with a decoy file to mimic a real Windows share.
3. **OpenCanary Setup**: Install and configure OpenCanary to log SMB access attempts to a specific log file.
4. **Monitoring & Alerts**: Use `incron` to monitor the log file and trigger a custom script when access is detected.
5. **Email Notification**: The script formats log data and sends an alert via `msmtp`.
6. **Blocking Attackers**: The script also blocks attacker IPs using `iptables`.

## **Testing & Outcome**
- Access the shared folder from another machine using:  
  `\<RaspberryPi_IP>`  
- Any access attempt triggers an email alert with the attacker's IP and activity details.
- After the initial access, the attacker's IP is automatically blocked, preventing further access attempts.
