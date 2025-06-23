---
title: Wazuh Security Monitoring & Incident Response Project
description: Secure your network for free!
publishDate: "2025-06-24T12:00:00Z"
---

![network.png](/portfolio/wazuh_thumbnail.png)

This project demonstrates the deployment, configuration, and use of Wazuh as a SIEM solution. The environment includes both physical and virtual machines, integrating multiple tools and data sources to provide advanced threat detection, vulnerability management, file integrity monitoring, and automated incident response.<br><br><br><br><br>


## What is Wazuh?
Wazuh is a powerful open-source security platform designed for threat detection, visibility, and compliance. It provides a framework for collecting, analyzing data from various sources in real-time. Wazuh is widely adopted in enterprise environments due to its flexibility, scalability, and integration capabilities, making it a commonly used tool for modern Security Operations Centers (SOCs).


## Environment Setup

In this project both virtual and physical machines were used. A network in the 192.168.88.0/24 subnet was created with the Mikrotik Router acting as a DHCP Server. The Wazuh Manager has a static ip assigned to it so that the connection between agents and the manager can be reestablished when lost.

![topology.png](/portfolio/topology.drawio.svg)
*Figure 1: Detailed network topology diagram illustrating the network architecture*

In this project the follwoing is used: 
- Wazuh Server/Manager: Installed on Ubuntu 22.04 (VM)

- Agents:

 - - Windows 11 (physical)

 - - Ubuntu Linux (VM)

- Network devices: MikroTik router

- IDS: Suricata on Ubuntu agent


### Wazuh Manager Setup

Wazuh typically operates with two types of machines:
- Agents installed on endpoints (Linux, Windows, macOS) collect data and monitor systems.
- A Manager processes and correlates this data using detection rules and decoders.

To setup the Ubuntu Machine as a Wazuh Manager the Wazuh installation assistant is needed. To download it the following command will be entered:

```bash
curl -sO https://packages.wazuh.com/4.12/wazuh-install.sh
```

Then the following command will be used in order to install the Wazuh server. 

```bash
bash wazuh-install.sh -a
```

After running this command at the end of the execution the credentials to access the wazuh dashboard will be displayed. 
The dashboard is located at: https://<wazuh-ip>:443


### Adding agents
Wazuh provides two ways to add agents either via GUI or CLI. 
To add agents via CLI the *manage_agents* script located at */var/ossec/bin/* is used. In this project I used both ways to add agents. Once the agents are added one can see them in the wazuh dashboard. 

![topology.png](/portfolio/agents.png)
*Figure 2: Agents on Wazuh Dashboard*