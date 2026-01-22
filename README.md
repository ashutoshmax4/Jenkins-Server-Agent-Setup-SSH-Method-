# Jenkins-Server-Agent-Setup-SSH-Method-
# 🚀 Jenkins Server & Agent Setup (SSH Method)

This repository provides a **complete step-by-step guide** to attach a **Jenkins Agent (Node)** to an **existing Jenkins Server (Master)** using **SSH key-based authentication** on **Ubuntu (AWS EC2)**.

This setup is based on a **real-world, successfully working Jenkins configuration**.

---

## 🏗️ Architecture Overview

```text
Jenkins Server (Master)
        |
        |  SSH (Port 22, Private Key)
        |
Agent Server (EC2 Ubuntu)
        |
        └── Executes Jenkins Jobs

🔑 Key Rule

Private Key → Jenkins Server

Public Key → Agent Server (authorized_keys)

✅ Prerequisites
Jenkins Server

Jenkins installed and running

SSH private key available

Network access to agent server

Agent Server (EC2 Ubuntu)

Ubuntu instance running

Port 22 (SSH) open

Java installed

🖥️ Step 1: Agent Server Configuration
Install Java (Mandatory)
sudo apt update
sudo apt install openjdk-17-jdk -y
java -version

Create Jenkins Working Directory
mkdir -p /home/ubuntu/jenkins
chmod 755 /home/ubuntu/jenkins

🔐 Step 2: SSH Setup on Agent Server
Create .ssh Directory
mkdir -p /home/ubuntu/.ssh
chmod 700 /home/ubuntu/.ssh

Add Public Key
nano /home/ubuntu/.ssh/authorized_keys


Paste ONLY the public key:

jenkins-agent-key.pub

Fix Permissions
chmod 600 /home/ubuntu/.ssh/authorized_keys
chown -R ubuntu:ubuntu /home/ubuntu/.ssh

🔑 Step 3: Jenkins Server SSH Setup
Private Key Location
/home/ubuntu/.ssh/jenkins-agent-key

Fix Private Key Permissions
chmod 400 /home/ubuntu/.ssh/jenkins-agent-key

Convert Key to Jenkins-Compatible Format
ssh-keygen -p -m PEM -f /home/ubuntu/.ssh/jenkins-agent-key


Press ENTER (no passphrase)

🧪 Step 4: Manual SSH Test (MOST IMPORTANT)

From Jenkins Server:

ssh -i /home/ubuntu/.ssh/jenkins-agent-key ubuntu@AGENT_PUBLIC_IP

✅ Expected Output
ubuntu@ip-172-31-x-x:~$


⚠️ If this fails, Jenkins agent connection will also fail.

🔐 Step 5: Add SSH Credential in Jenkins

Path:

Manage Jenkins → Credentials → Global → Add Credentials


Fill the following:

Field	Value
Kind	SSH Username with private key
Username	ubuntu
Private Key	Enter directly (paste PRIVATE key)
ID	jenkins-agent-cred

Save the credential.

🧱 Step 6: Create Jenkins Agent (Node)

Path:

Manage Jenkins → Nodes → New Node

Node Configuration
Field	Value
Node Name	aws-agent-1
Type	Permanent Agent
Executors	2
Remote Root Directory	/home/ubuntu/jenkins
Labels	aws-agent
Usage	Only build jobs with label
Launch Method	Launch agents via SSH

Save the node.

🔗 Step 7: SSH Launcher Configuration
Setting	Value
Host	AGENT_PUBLIC_IP
Port	22
Credentials	jenkins-agent-cred
Java Path	/usr/bin/java
Host Key Verification	Non-verifying
Timeout	120

Save configuration.

▶️ Step 8: Launch Agent
Manage Jenkins → Nodes → aws-agent-1 → Launch Agent

✅ Success Message
Agent successfully connected and online

✅ Step 9: Verification

Configure a Jenkins job:

Restrict where this project can be run → aws-agent

Console Output
Running on aws-agent-1

❌ Common Issues & Fixes
Issue	Cause
Permission denied (publickey)	Wrong key or wrong permissions
Authentication failed	Wrong Jenkins credential
SSH works but Jenkins fails	Incorrect credential selected
Public key pasted in Jenkins	Jenkins needs PRIVATE key
