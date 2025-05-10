---
title: Physical and Virtual Network with ProxMox and MikroTik
description: one homelab to rule them all!
publishDate: "2024-06-10T12:00:00Z"
---

![network.png](/portfolio/public/thumbnail_virtual_network.jpg)

This project involved setting up a virtualized network environment using Proxmox, Mikrotik routers, and Windows Server infrastructure. The objective was to simulate a complete network topology with proper routing, domain services, and VPN connectivity between two sites.<br><br><br><br><br>


## 1. Proxmox Topology Setup

The topology was recreated in Proxmox based on provided reference materials. Virtual machines (VMs) were cloned and carefully named according to their designated roles, ensuring ID consistency and clear identification. Each VM was configured with the appropriate number of network interfaces and VLAN tagging where required.

Access permissions were set on the VMs to restrict unauthorized access by other students. This was done by adjusting the ACLs within the Proxmox interface.

![topology.png](/portfolio/public/virtual_topology.png)
*Figure 2: Detailed network topology diagram illustrating the network architecture*


## 2. Mikrotik Router Configuration (Site 1)

The Mikrotik VM served as the main router for the network. The following configurations were applied:

- `Ether1` was configured to obtain an IP address automatically via DHCP for internet access.
- `Ether2` and `Ether3` were configured statically according to the internal IP scheme.
- A DHCP server was set up on `Ether2` to provide IPs to the LAN, using the IP address of the Windows Server (DC1) as the DNS.
- NAT (masquerade) was configured on `Ether1` to allow internal network access to the internet.

## 3. Windows Server Configuration

A Windows Server 2016 VM was used for domain and group policy management. The server underwent the following setup:

- The computer name was updated as per the given convention.
- A static IP address was assigned.
- Active Directory Domain Services were installed, and the domain was created.
- A Windows 10 VM (`W10-Erti-1`) was successfully joined to the domain.
- Organizational Units (OUs), users, and groups were created.
- Shared folders were established with appropriate permissions using NTFS and share-level permissions.
- Group Policies were defined and linked to the OUs to enforce settings for domain users.

## 4. Second Site Deployment (S-Lab)

A second Mikrotik router was configured for the S-Lab environment, simulating a remote location. The router setup included:

- Static interface IP configuration and a DHCP server for the local network.
- A VPN tunnel was created between the S-Lab Mikrotik router and the primary Mikrotik router in Proxmox.
- Routing was configured to ensure all traffic from the S-Lab subnet traversed the VPN to the main network.

## 5. Smartphone VPN Configuration

To enable remote access testing, the OpenVPN configuration file `smartphone-mikrotik-v16.ovpn` was modified with the correct public IP address of the main Mikrotik router.

The modified file was transferred to a smartphone and imported into the OpenVPN mobile application. Upon connection, the VPN successfully linked the smartphone to the internal network through the Mikrotik VPN server.


## Conclusion

This project successfully demonstrated the creation of a secure, segmented, and manageable network environment using virtual infrastructure. The use of Proxmox for virtualization, Mikrotik for routing and VPN, and Windows Server for domain management provided a comprehensive experience in modern network setup and operations.