---
title: Turn Your Personal Hotspot into a Powerful Rogue AP with Captive Portal
description: Hands-on exploration of creating a Rogue Access Point using a personal hotspot and airgeddon with a custom captive portal.
publishDate: "2026-01-05T20:00:00Z"
---

<video autoplay loop muted playsinline controls>
  <source src="/portfolio/rogueap.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video>

*Video 1: Evil Twin Rogue AP serving a fake captive portal*

# Turn Your Personal Hotspot into a Powerful Rogue AP with Captive Portal

**Practical walkthrough of building an Evil Twin Rogue AP using a mobile hotspot and airgeddon.**

---

## Introduction

This project demonstrates how a **personal mobile hotspot** can be abused to create a **Rogue Access Point (Evil Twin)** combined with a **custom captive portal**. The objective is to highlight how easily users can be deceived into connecting to malicious Wi-Fi networks when trust-based naming and familiar login pages are abused.

This lab was performed strictly for **educational and security awareness purposes** in a controlled environment.

---

## Description

A Rogue Access Point attack involves cloning a legitimate wireless network and forcing nearby devices to connect to it. Once connected, victims are redirected to a **captive portal** that can harvest credentials.

In this project:
- A phone hotspot was renamed to **Free Wifi**
- `airgeddon` was used to launch an Evil Twin attack
- A **custom captive portal** was deployed
- The portal replicated an **Instagram login page**
- Credentials were captured using username and password fields

---

## Lab setup

- **Attacker OS**: Kali Linux  
- **Wireless adapter**: Monitor-mode capable chipset  
- **Wi-Fi source**: Mobile phone hotspot  
- **SSID**: `Free Wifi`  
- **Framework**: airgeddon  
- **Captive portal**: Custom HTML (Instagram clone)

Chipset compatibility reference:  
https://github.com/v1s1t0r1sh3r3/airgeddon/wiki/Cards%20and%20Chipsets

---

## Preparation

### Hotspot configuration

- Enabled personal hotspot on phone
- Renamed SSID to **Free Wifi**
- Left network open to increase connection likelihood

---

## Launching airgeddon

Start the framework:

```bash
airgeddon
```

After dependency checks complete, select the wireless interface.

---

## Interface configuration

### Enable monitor mode

From the main menu:

* Press **2** → Enable monitor mode

airgeddon automatically handles conflicting processes and creates a monitor interface.

---

## Evil Twin attack setup

Navigate through the menus:

* Press **7** → Evil Twin Attacks Menu
* Press **9** → Evil Twin AP attack with captive portal

airgeddon begins scanning for nearby access points.

---

## Target selection

When scanning starts:

* Wait until **Free Wifi** appears
* Press **Ctrl + C** to stop scanning
* Select the target network

---

## Deauthentication attack

Select the attack method:

```bash
1. Deauth / disassoc amok mdk4 attack
```

Configuration used:

* DoS attack → **N**
* MAC address spoofing → **Y**
* External handshake capture → **N**

Proceed with default values.

---

## Victim connection phase

* Rogue AP is created
* Legitimate clients are deauthenticated
* Victim device connects to the fake network

Once a device connects:

* Press **Enter** when prompted
* Select language
* Answer **N** to advanced options

---

## Captive portal customization

airgeddon serves captive portals from:

```bash
/tmp/ag1/www/
```

Replace the default portal with the Instagram clone:

```bash
cp -r /instagram_clone/* /tmp/ag1/www/
```

This deploys:

* Custom HTML
* Styling assets
* Login form with username and password fields

---

## Credential harvesting

When victims attempt to access the internet:

* They are redirected to the fake Instagram login page
* Submitted credentials are logged by airgeddon in real time

---

## Results

* Devices automatically connected due to stronger signal
* Users trusted the SSID name
* Familiar branding significantly increased success rate
* No complex infrastructure was required

---

## Attack summary

1. Rename hotspot to **Free Wifi**
2. Enable monitor mode
3. Launch Evil Twin attack
4. Deauthenticate clients
5. Deploy Rogue AP
6. Serve custom captive portal
7. Capture credentials

---

## Security implications & mitigation

* Avoid connecting to open Wi-Fi networks
* Never enter credentials into captive portals
* Use WPA3 / WPA2-Enterprise where possible
* Enable wireless intrusion detection
* Train users to recognize Evil Twin behavior

---

## Conclusion

This project highlights how **trust in public Wi-Fi networks can be weaponized with minimal effort**. Using common tools and a mobile hotspot, it is possible to deploy a convincing Rogue Access Point capable of harvesting credentials.

From a defensive perspective, it reinforces the importance of wireless security controls and user awareness. From an offensive learning standpoint, it demonstrates the real-world impact of Evil Twin attacks and captive portal abuse.

