# DevSecOps Layer Overview

## Objective

The DevSecOps Layer introduces secure software development and CI/CD capabilities into the Enterprise Home Lab.

This layer integrates with:

- Identity Layer (Active Directory + LDAP)
- Network Layer (pfSense)
- Monitoring Layer (Wazuh SIEM)

The goal is to simulate an enterprise-grade DevSecOps environment while remaining resource-efficient.

---

## Architecture Positioning

The DevSecOps VM is deployed inside the internal lab network and communicates with:

- Active Directory (Authentication & DNS)
- Wazuh Server (Security Monitoring)
- Jump Host (Developer Access)

```

Active Directory (172.16.1.10)
↑
↓
DevSecOps-01 VM (172.16.1.100)
↑
↓
Wazuh Server (172.16.1.50)

```

---

## Design Principles

- CLI-first deployment (resource optimized)
- Docker-based service isolation
- Domain-aware infrastructure
- Security-first build order
- Full monitoring visibility in Wazuh
- Enterprise-style documentation

---

## Resource Constraints & Strategy

Available spare RAM: ~6GB  
Allocated to DevSecOps VM: 4GB  

To remain efficient:

- No heavy GUI platforms
- No Kubernetes (yet)
- Lightweight services (Gitea + Jenkins later)
- Dockerized deployment model

---

## Planned Capabilities

This layer will provide:

- Secure Git Server
- CI/CD pipeline
- Integrated security scanning
- Container build & registry
- Runtime monitoring
- Attack simulation for detection engineering

---

## Security Integration Strategy

Every component in this layer must:

- Send logs to Wazuh
- Be attackable for simulation
- Be detectable via custom rules
- Support SOC-style investigation
