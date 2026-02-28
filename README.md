# Enterprise Cybersecurity Home Lab

![Status](https://img.shields.io/badge/Status-Active-green)
![Hardware](https://img.shields.io/badge/RAM-16GB-blue)
![Focus](https://img.shields.io/badge/Focus-SOC%20Engineering-red)
![Architecture](https://img.shields.io/badge/Design-Segmented-orange)

A full enterprise-style Security Operations lab built on a 16GB RAM system.

This project simulates a real-world organization with identity management, firewall segmentation, SIEM monitoring, containerized services, Kubernetes workloads, CI/CD automation, red team simulations, and incident response workflows.

This lab focuses on:

- Infrastructure Engineering
- Network Segmentation
- Active Directory Deployment
- SIEM Integration (Wazuh)
- Dockerized Services
- Detection Engineering
- MITRE ATT&CK Mapping
- Incident Response

This lab is built on:

- 16GB RAM
- Limited CPU cores
- Single primary server

Because of hardware limitations:

- Docker is used to reduce memory usage
- Windows Server Core is preferred over GUI
- Lightweight Linux VMs are prioritized
- Services are deployed in phases

---

## High-Level Architecture

Core Hypervisor: Proxmox VE  
Firewall & VPN: pfSense  
Identity Layer: Active Directory (Windows Server Core)  
Monitoring Layer: Wazuh SIEM  
Container Services: Docker (Pi-hole, Nginx Proxy ) ?  
Access Control: Jump Host  
Threat Simulation: Red Team Canary ?
Mapping: MITRE ATT&CK Navigator  
Decoy Assets: Honeypot ?

External Devices:

- User Laptop with following 2 VM
    - Win 11 - (VPN Client + Domain User Simulation)
    - Kali ( Red Team Environment)

---

## Architecture

It is designed as a hardened security ecosystem, where pfSense orchestrates network isolation and Wazuh provides continuous monitoring and threat detection across the Active Directory domain.

<img width="1422" height="1272" alt="labarch-2" src="https://github.com/user-attachments/assets/702911c4-9296-40c8-9546-f88038e24482" />

---
