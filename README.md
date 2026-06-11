# Local Endpoint Detection Engineering & Incident Response Lab

## 1. Executive Summary & System Architecture
This project demonstrates the end-to-end engineering of a localized Security Operations Center (SOC) log pipeline and Endpoint Detection and Response (EDR) testing environment. The objective is to establish an isolated platform for emulating modern adversary tactics, engineering custom behavioral detection logic, and analyzing granular operating system telemetry without relying on cloud resources or traditional Active Directory infrastructure.

### Architectural Blueprint
- **Security Operations Console (Laptop 2):** Wazuh SIEM Manager (Distributed Single-Node Deployment running on Linux)
- **Target Workstation (Laptop 1):** Windows 10/11 Endpoint Virtual Machine monitored via Microsoft Sysmon and Wazuh Agent
- **Adversary Node (Laptop 1):** Kali Linux Virtual Machine / Local Malicious Execution Context

---

## 2. Phase 1: Security Operations Console Deployment
This phase establishes the central log aggregation, indexing, and visualization cluster using the open-source Wazuh SIEM platform.

### Deployment Log
- **Host Operating System:** [Kali Linux (main)]
- **Deployment Method:** Wazuh Automated All-in-One Installation Assistant
- **Commands Executed:**
```bash
  sudo apt update && sudo apt upgrade -y
  curl -sO [https://packages.wazuh.com/4.x/wazuh-install.sh](https://packages.wazuh.com/4.x/wazuh-install.sh)
  sudo bash wazuh-install.sh -a
