Wazuh Local SIEM Deployment — Full Technical Walkthrough
Author: Joshua Fakorede Adeleye
Date: October 31, 2025
Environment: Windows 11 (Host) + Ubuntu 22.04 (Oracle VMware)
Tools Used: VS Code (SSH Remote), Docker Engine, Docker Compose, PowerShell, Wazuh

Background & Motivation
The goal of this project was to deploy a Wazuh SIEM lab environment capable of monitoring multiple endpoints (Linux and Windows) in a contained yet realistic security setup.

Step-by-Step Technical Implementation
1. Setting up the Ubuntu Virtual Machine (Oracle VMware)
Installed Oracle VMware Workstation.
Created a new Ubuntu 22.04 LTS virtual machine.
Before booting the VM, I:
Set Network Adapter to Bridged Mode (to enable host to VM connectivity).
Enabled Promiscuous Mode → Allow All (to allow the VM to capture and send traffic to/from other hosts).
This setup ensured that the Ubuntu VM could communicate directly with my Windows host and any other devices on the same local network.

🌐 Network Verification
To confirm network configuration, I ran:
ip addr
and obtained my VM IP, for example:
192.168.79.xx
This IP became the manager address for agent configurations later.
The bridged adapter ensured that both my Ubuntu VM and my Windows host were on the same subnet, enabling direct communication without NAT translation.


2. SSH Access from VS Code to Ubuntu
Once the Ubuntu VM was running, I connected from VS Code on Windows using the Remote-SSH extension based on my VM Ip adddress.
This allowed me to:
Access the Ubuntu terminal directly within VS Code.
Execute Docker and system commands from Windows seamlessly.
Transfer files and manage repositories between host and VM securely.
The SSH connection confirmed that the bridged network configuration worked properly and that the VM was reachable from the host.


3. 🐋 Installing Docker & Docker Compose on Ubuntu
After establishing SSH access, I installed Docker Engine and Docker Compose inside Ubuntu from Vscode terminal with SSH connection:
sudo apt update && sudo apt install docker.io docker-compose -y
sudo systemctl enable docker
sudo systemctl start docker
docker --version
docker compose version
This created a stable container runtime environment for the Wazuh stack.


4. 🔧 Cloning and Launching Wazuh via Docker Compose
Using VS Code’s integrated terminal, I cloned the official Wazuh Docker repository and deployed the single-node stack:
git clone https://github.com/wazuh/wazuh-docker.git
cd wazuh-docker/single-node
sudo docker compose pull
sudo docker compose up -d
✅ Result:
Wazuh Manager, Indexer, and Dashboard containers all initialized successfully. 

5. Verifying Container Health
Confirmed that all three containers — Manager, Indexer, and Dashboard — were running properly:
sudo docker ps
Then accessed the dashboard via browser:
https://<ubuntu-ip>:5601
(Using the default Wazuh credentials to log in and confirm full stack health.)

6. Installing and Configuring Wazuh Agent on Ubuntu
To simulate a monitored Linux endpoint, I installed the Wazuh agent inside the same Ubuntu VM:
sudo apt install wazuh-agent -y
sudo nano /var/ossec/etc/ossec.conf
In the configuration file, I updated the manager section to:
<server>
  <address>192.168.79.xx</address>
</server>
Then reloaded the service:
sudo systemctl daemon-reload
sudo systemctl restart wazuh-agent
sudo systemctl status wazuh-agent
✅ The agent registered successfully and began sending data to the Wazuh manager.


7. 🪟 Installing the Wazuh Agent on Windows
Next, I installed the Wazuh agent on my Windows 11 host system.
Using PowerShell (Admin):
Invoke-WebRequest -Uri https://packages.wazuh.com/4.x/windows/wazuh-agent-4.9.1-1.msi -OutFile $env:tmp\wazuh-agent.msi
msiexec.exe /i $env:tmp\wazuh-agent.msi /q WAZUH_MANAGER='192.168.79.xx' WAZUH_AGENT_GROUP='default' WAZUH_AGENT_NAME='JoshuawindowsHPpc'
NET START WazuhSvc
Initially, the agent failed to connect.
Troubleshooting steps:
Test-NetConnection 192.168.79.xx -Port 1514
After switching VMware’s network to Promiscuous Mode (Allow All), the port test succeeded:
TcpTestSucceeded : True
Then, restarting the agent:
NET STOP WazuhSvc
NET START WazuhSvc
✅ The Windows agent appeared in the Wazuh dashboard shortly after.

8. 📊 Dashboard Verification
Accessed the Wazuh dashboard from the browser:
https://192.168.79.xx:5601
Logged in and confirmed:
Manager running ✅
Indexer healthy ✅
Dashboard online ✅
Linux and Windows agents active and reporting ✅

Troubleshooting   Summary
Issue	            Diagnosis	                  Resolution
WSL2 Docker       startup failures	           systemd and socket issues	Switched to full Ubuntu VM
Windows agent     not connecting	            Port 1514 unreachable	Enabled Bridged + Promiscuous Mode
Missing agent     visibility	                Configured ossec.conf with correct manager IP	Agent registered successfully


🏁 Final Reflection
This lab demonstrates a complete SIEM deployment workflow — from environment setup and SSH integration to Docker orchestration and agent management.
It showcases real-world DevSecOps practices, adaptability under troubleshooting pressure, and a methodical approach to building a cross-platform, enterprise-grade security monitoring lab.
