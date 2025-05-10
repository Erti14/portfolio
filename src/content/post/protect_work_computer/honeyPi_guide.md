---
title: How to Secure a Workplace Computer
description: Secure a woekplace computer. 
publishDate: "2025-05-11T16:00:00Z"
---



## Introduction

In today’s digital workplace, the computer is more than just a tool, it's the gateway to sensitive company data, critical operations, and daily communication. With increasing cyber threats, human error, and device vulnerabilities, securing workplace computers has become a top priority for organizations of all sizes. 


## Threats

There's a lot of threats that can lead to unathorized access. They are divided into two big groups : external and internal threats. 


### 🔐 Internal Threats

These type of threats are caused by people within the organization who have legitimate access. These could be:

- Use of an unlocked PC
- Physical access to hardware (e.g., removing a hard drive)
- Eavesdropping using microphones or cameras
- Weak passwords
- Keyloggers (if installed by internal users)
- Deleting or overwriting data
- License revocation
- Theft of a workstation computer (if committed internally)


### 🌐 External Threats

These come from outside the organization, often via networks or physical intrusions.These could be:

- Eavesdropping on wireless communication
- Denial-of-Service (DoS) attacks
- Power outage or loss of network connectivity (depending on cause, often external)
- Keyloggers (if introduced via external malware)
- Hardware failure or damage (e.g., caused by external factors)
- Theft of a workstation computer (e.g., by external burglars)


## Measures

Protecting a workplace computer requires a multi-layered approach that ranges from highly technical solutions to simple, everyday habits.Both technical and non-technical actions play a crucial role in ensuring the overall security of workplace devices.


### Minimal system

**What is not needed is not installed.** Any hardware, software or service that is not needed should not be installed nor used on the computer. As the number of components in a computer increases, so does the number of potential vulnerabilities.

### Two-Factor-Authentication

There are three ways of authenticating a person. These include: 
- Authentication through something one knows (Knowledge)
- Authentication through something one has (Ownership)
- Authentication through something one is (Biometrics)

In a Two-Factor-Authentication at least two of these ways of authenticating are used. This makes it less likely for the attacker to break in to the computer. 


### Screensaver

Each workplace computer should use a password-protected Screensaver. Often employees leave the workplace without locking or turning off the computer. The screensaver automatically switches on after a certain time and protects the PC against unauthorized access. 


### Passwords

Only strong passwords should be used. They have these requirements: 

- Minimum Length: At least 12 characters (longer is better).

- Character Variety: Includes a mix of:

    - Uppercase letters (A–Z)

    - Lowercase letters (a–z)

    - Numbers (0–9)

    - Special characters (e.g., !, @, #, $, %, ^, &, *)

- No Common Words: Avoid dictionary words, names, or easily guessable patterns.

- No Personal Information: Don’t use your birthdate, username, pet’s name, etc.

- Avoid Reuse: Don’t reuse passwords across multiple accounts.

- Avoid Keyboard Patterns: Don’t use simple sequences like 123456, qwerty, or asdfgh.

- Regular Updates: Change passwords periodically, especially after a breach.

**Passwords should be well protected**. It is not enough to use strong passwords but to also protect them. Passwords of critical users such as an Administrator should be stored on a physical safe. 

Due to passwords being complex most employees write their passwords on sticky notes and stick them at the monitor. This practice poses a serious security risk, as attackers can easily spot and use this password to gain access. Hence they should use a **Password Manager**. 



### Security-Awareness-Trainings

Employees who are not in touch with IT security usually also have no understanding (knowledge) of the topic. Regular employee training helps them point out the dangers.


### Security Policies

Security Policies help communicate the security requirements within an organization. These are documents that contain Information regarding IT-Security. 


### Encryption

Data on workplace computers should be encrypted, especially on mobile devices that are more likely to be lost or stolen. **Key management is critical**. Securely handling and storing encryption keys is just as important as the encryption itself.

There are 4 levels of encryption: 
 - **Partition Encryption** - protects specific sections of a hard drive.
 - **File-level Encryption** - encrypts individual files as needed.
 - **Full disk Encryption** - encrypts the entire hard drive to secure all data at once.
 - **Conatiner Encryption** - stores data within an encrypted "container".


### Anti-Malware

Anti-Malware software stops the download of malware incase of carelessness. There are different types of Anti-Malware Software: 

- Anti-Virus 
- Anti-Spam
- Anti-Phishing
- Pop-Up Blocker
- Anti-Spyware
- Anti-Adware


### Personal Firewall

Each workplace computer should have a personal firewall installed. These firewalls should be managed centrally. 

### Updates / Security Patches

Workplace computers should regularly be updated to ensure they have the latest security patches, performance improvements, and bug fixes. 


### Host Based IDS / IPS

Intrusion Detection Systems (IDS) and Intrusion Prevention Systems (IPS) are tools that use log, system and kernel files to detect and prevent attacks. 


### Network Access Control

Network Access Control (NAC) ensures that devices are checked for compliance before they are allowed full access to the corporate network.When a computer connects to the network, NAC checks whether it is up to date, especially with regard to antivirus software and critical security updates. If the computer is not up to date, it is placed in **quarantine**, often in a separate subnet. In quarantine, the device is updated and brought into compliance. Once the updates are complete, the device is granted access to the company network.


### Whitelisting / Blacklisting

Through software like AppLocker the use of specific programs by users or the system can be restricted or allowed. 


### Device Control

Checks are made regarding which devices may be connected to a system and which are not allowed to be connected. For example only USB sticks from certain manufacturers are allowed. 

### Hardware Security 

To prevent unauthorized physical access or theft, workplace computers and devices should be protected using appropriate hardware security measures:

- Cable locks – Secure laptops and desktop computers to desks to prevent quick theft.

- Lockable cabinets – Store equipment like external drives, backup media, or networking devices in secured cabinets.

- Safes – Especially useful for storing mobile devices (e.g., laptops, tablets) outside of working hours or during transport.


### Logging

Logging refers to the recording of specific actions and events that occur on a system. These logs are essential for monitoring, troubleshooting, and detecting potential security incidents.

Key components that generate important logs include:

- Firewall – Logs incoming and outgoing network traffic, including blocked or suspicious connections.

- Antivirus Software – Records detected threats, scan results, and actions taken against malware.

- Operating System (OS) – Tracks system events such as user logins, software installations, updates, and error messages.


### Secure Data Deletion

When a workplace computer is no longer needed or is being replaced, it's absolutely essential to securely erase the hard drive to prevent unauthorized data recovery.

- Formatting is NOT the same as deleting—formatted drives can often still be recovered with simple tools.

- For secure deletion, special software should be used that doesn’t just delete the data but overwrites it multiple times (in so-called passes or rounds) to make recovery virtually impossible.

- This is especially important for sensitive data stored on company devices.