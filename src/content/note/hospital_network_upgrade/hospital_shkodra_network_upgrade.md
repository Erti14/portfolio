---
title: Regional Hospital of Shkodra Network Upgrade
description: one homelab to rule them all!
publishDate: "2025-04-15T12:30:00Z"
---

<div class="flex justify-center my-4">
    <video controls width="50%" style="max-width: 400px;">
        <source src="/portfolio/src/content/public/wlans.mp4" type="video/mp4">
        Your browser does not support the video tag.
    </video>
</div>

This project aimed to modernize and secure the network infrastructure of the Regional Hospital of Shkodra. The existing network equipment was outdated, unreliable, and lacked proper wireless access and segmentation. The goal was to provide a robust, scalable, and secure network setup that supports both staff and guest access while ensuring administrative manageability.<br><br><br><br><br>

## 🏥 Objectives

- Replace legacy switches with modern Layer 3 managed switches.
- Simulate the current and proposed network using Cisco Packet Tracer.
- Introduce centralized wireless control using a Wireless LAN Controller (WLC).
- Provide separate WLANs for hospital staff and guests.
- Improve network security and performance.
- Deploy and physically install Access Points (APs) throughout the hospital.

## 🔧 Implementation Steps

### 1. Network Simulation in Cisco Packet Tracer
- Created a complete topology of the hospital's existing and planned network infrastructure.
- Included VLANs for different departments (e.g., administration, medical, guest Wi-Fi).
- Simulated traffic flow, switch and router configurations.

### 2. Hardware Upgrade – Switch Replacement
- Identified critical points where switches needed replacement.
- Deployed managed switches to allow VLAN tagging, port security, and future expansion.
- Rewired necessary connections to the new switch setup.

### 3. Wireless Network Implementation
- Installed a **Wireless LAN Controller (WLC)** for centralized WLAN management.
- Configured two main SSIDs:
  - **Hospital-Staff**: secured network with WPA2-Enterprise authentication.
  - **Hospital-Guest**: isolated VLAN with bandwidth and access restrictions.

### 4. Access Point Installation
- Strategically placed access points for full hospital coverage.
- Focused on areas with high client density (e.g., waiting rooms, wards, administration offices).
- Performed wireless site survey to avoid dead zones.

### 5. Network Security Enhancements
- Configured VLAN segmentation to isolate guest and staff traffic.
- Enabled port security on switch interfaces to prevent unauthorized access.
- Implemented ACLs to restrict inter-VLAN communication where necessary.
- Planned firewall rules for perimeter protection.

## 📈 Outcomes

- Enhanced network reliability and performance.
- Secured access for internal users and visitors.
- Simplified WLAN management via the WLC.
- Improved wireless coverage hospital-wide.
- Provided a scalable foundation for future expansion (e.g., IP telephony, CCTV, IoT medical devices).

## 🛠 Tools & Technologies Used

- Cisco Packet Tracer (Simulation)
- Cisco Catalyst/Managed Switches
- Cisco WLC and Lightweight APs
- VLANs, Port Security, ACLs, WPA2 Enterprise
- Wireless Site Survey tools

## 📌 Conclusion

The upgrade significantly improved the hospital's network capabilities and readiness for modern digital healthcare operations. With better infrastructure in place, the hospital is now equipped to support secure digital services for staff, patients, and visitors.

