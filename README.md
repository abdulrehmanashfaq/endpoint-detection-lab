# Local Endpoint Detection Engineering & Incident Response Lab

## 1. Executive Summary & System Architecture
This project demonstrates the end-to-end engineering of a localized Security Operations Center (SOC) log pipeline and Endpoint Detection and Response (EDR) testing environment. The objective is to establish an isolated platform for emulating modern adversary tactics, engineering custom behavioral detection logic, and analyzing granular operating system telemetry without relying on cloud resources or traditional Active Directory infrastructure.

### Architectural Blueprint
- **Security Operations Console (Laptop 1):** Wazuh SIEM Manager (Distributed Single-Node Deployment running on Linux)
- **Target Workstation (Laptop 2):** Windows 10/11 Endpoint Virtual Machine monitored via Microsoft Sysmon and Wazuh Agent
- **Adversary Node (Laptop 2):** Kali Linux Virtual Machine / Local Malicious Execution Context

---

## 2. Phase 1: Security Operations Console Deployment
This phase establishes the central log aggregation, indexing, and visualization cluster using the open-source Wazuh SIEM platform.

### Deployment Log
- **Host Operating System:** Kali Linux (main)
- **Deployment Method:** Wazuh Automated All-in-One Installation Assistant
- **Commands Executed:**
```bash
sudo apt update && sudo apt upgrade -y
curl -sO [https://packages.wazuh.com/4.x/wazuh-install.sh](https://packages.wazuh.com/4.x/wazuh-install.sh)
sudo bash wazuh-install.sh -a
```
Wait for a while.When installtion is completed.Go to your browser and open [https://127.0.0.1](https://127.0.01)
![Click Continue](/images/image1.png)
Click on accept (the risk and continue)
![Credentials](/images/image2.png)
Add your credentails that was given after the installation.
![Setup](/images/image3.png)
Now we have successfully setup our wazuh on our Kali(laptop 1).

## 3. Phase 2 : Setting up Windows Machine

On it we need to install the wazuh agent for integrating windows with our wazuh server

```Powershell
Invoke-WebRequest -Uri https://packages.wazuh.com/4.x/windows/wazuh-agent-4.14.5-1.msi -OutFile $env:tmp\wazuh-agent.msi; msiexec.exe /i $env:tmp\wazuh-agent.msi /q WAZUH_MANAGER="YOUR_KALI_IP"
```
Remeber to replace your ip in this command
Now start your wazuh service on windows 
```Powershell 
Start-Service wazuhsvc
```
Check if is running
```Powershell 
Get-Service wazuhsvc
```
![Command](/images/image4.png)
Now we can look there is an active agent on wazuh server that is our windows machine
![Command](/images/image5.png)
Remeber to setup the sysmon on the windows machine. I already downloaded and isntalled on it.
You can go to this page to understand how to setup sysmon [Sysmon setup on wazuh](https://wazuh.com/blog/using-wazuh-to-monitor-sysmon-events/)

## 4. Phase 3 : Generate a Loud Telemetry Event on Windows

Now we need to verify that our sysmon actully sends the logs or event over the wire
so are going to simple command.
```bash
net user test test /add
```
On our windows terminal we will run this command as administrator.
Event id would be 4720  
Now we head to our discover tab on the wazuh server and type test in the search field
![Command](/images/image6.png)  
Now you can see our logs are piping in to our wazuh sever.