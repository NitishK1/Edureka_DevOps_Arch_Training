# WSL Setup Guide - 3 VM Configuration

## Overview
This guide will help you create and configure 3 separate WSL instances:
1. **Master VM** - Jenkins, Ansible, Git
2. **Test Server** - Target for automated deployment and testing
3. **Production Server** - Final production deployment

## Architecture
```
┌─────────────────────────────────────────────────────────────┐
│                    Windows Host                              │
│  ┌────────────────┐  ┌────────────────┐  ┌────────────────┐│
│  │  WSL: Master   │  │ WSL: Test      │  │ WSL: Prod      ││
│  │  - Jenkins     │─▶│ - Docker       │─▶│ - Docker       ││
│  │  - Ansible     │  │ - Puppet Agent │  │ - Puppet Agent ││
│  │  - Git         │  │                │  │                ││
│  └────────────────┘  └────────────────┘  └────────────────┘│
└─────────────────────────────────────────────────────────────┘
```



## Part 1: Create 3 WSL Instances

### Step 1.1: Install Ubuntu for Master VM

```powershell
# Open PowerShell as Administrator

# Download Ubuntu 22.04 (if not already installed)
wsl --install -d Ubuntu-22.04

# Or if you want to create from existing:
# This will be your Master VM - keep the default name or rename
```

### Step 1.2: Create Test Server WSL Instance

```powershell
# Method 1: Install fresh Ubuntu instance
wsl --install -d Ubuntu

# Then rename it:
wsl --shutdown Ubuntu
# Export the instance
wsl --export Ubuntu C:\temp\ubuntu-backup.tar
# Unregister original
wsl --unregister Ubuntu
# Import as Test Server
wsl --import TestServer C:\WSL\TestServer C:\temp\ubuntu-backup.tar
# Clean up backup
del C:\temp\ubuntu-backup.tar
```

### Step 1.3: Create Production Server WSL Instance

```powershell
# Using the same backup, create Production instance
wsl --import ProdServer C:\WSL\ProdServer C:\temp\ubuntu-backup.tar

# Or download fresh:
# wsl --install -d Ubuntu
# Then export and import with name ProdServer
```

### Step 1.4: Verify All Instances

```powershell
# List all WSL instances
wsl --list --verbose

# You should see:
# - Ubuntu (or Master)
# - TestServer
# - ProdServer
```



## Part 2: Configure Each WSL Instance

### Step 2.1: Setup Master VM (Jenkins, Ansible, Git)

```bash
# Start Master VM
wsl -d Ubuntu  # or your master instance name

# Create user (if needed)
# Follow prompts to create username and password

# Update system
sudo apt update && sudo apt upgrade -y

# Install required packages
sudo apt install -y \
    openjdk-11-jdk \
    git \
    ansible \
    python3 \
    python3-pip \
    openssh-server \
    curl \
    wget

# Install Jenkins
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
sudo apt update
sudo apt install -y jenkins

# Start services
sudo service ssh start
sudo service jenkins start

# Generate SSH keys for connecting to Test and Prod servers
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa -N ""

# Get Master VM IP
hostname -I
# Note this IP - you'll use it later
```

### Step 2.2: Setup Test Server

```bash
# Start Test Server WSL
wsl -d TestServer

# Create user with SAME username as Master (for easier SSH)
# Or create different user and configure accordingly

# Update system
sudo apt update && sudo apt upgrade -y

# Install required packages
sudo apt install -y \
    python3 \
    python3-pip \
    openssh-server \
    git

# Start SSH
sudo service ssh start

# Configure SSH to auto-start (add to ~/.bashrc)
echo 'sudo service ssh status > /dev/null || sudo service ssh start' >> ~/.bashrc

# Get Test Server IP
hostname -I
# Note this IP: This is your TEST_SERVER_IP
```

### Step 2.3: Setup Production Server

```bash
# Start Production Server WSL
wsl -d ProdServer

# Create user (same steps as Test Server)

# Update system
sudo apt update && sudo apt upgrade -y

# Install required packages
sudo apt install -y \
    python3 \
    python3-pip \
    openssh-server \
    git

# Start SSH
sudo service ssh start

# Configure SSH to auto-start
echo 'sudo service ssh status > /dev/null || sudo service ssh start' >> ~/.bashrc

# Get Production Server IP
hostname -I
# Note this IP: This is your PROD_SERVER_IP
```



## Part 3: Configure SSH Between Instances

### Step 3.1: Copy SSH Key from Master to Test and Prod

```bash
# From Master VM (Ubuntu)

# Copy public key to Test Server
ssh-copy-id username@TEST_SERVER_IP

# Copy public key to Production Server
ssh-copy-id username@PROD_SERVER_IP

# Test connections
ssh username@TEST_SERVER_IP
exit
ssh username@PROD_SERVER_IP
exit
```

### Step 3.2: Alternative Manual Method

```bash
# In Master VM, get public key
cat ~/.ssh/id_rsa.pub

# Copy the output

# In Test Server WSL
wsl -d TestServer
mkdir -p ~/.ssh
echo "PASTE_PUBLIC_KEY_HERE" >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
chmod 700 ~/.ssh

# In Production Server WSL
wsl -d ProdServer
mkdir -p ~/.ssh
echo "PASTE_PUBLIC_KEY_HERE" >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
chmod 700 ~/.ssh
```



## Part 4: Get IP Addresses

### Step 4.1: Collect All IPs

```powershell
# From PowerShell

# Master VM IP
wsl -d Ubuntu hostname -I

# Test Server IP
wsl -d TestServer hostname -I

# Production Server IP
wsl -d ProdServer hostname -I
```

### Example Output:
```
Master VM:      192.168.128.100
Test Server:    192.168.128.101
Production:     192.168.128.102
```

**Note**: These IPs may change after Windows restart!



## Part 5: Configure Project Files

### Step 5.1: Update Ansible Inventory

File: `ansible/inventory/hosts`

```ini
[test_server]
test-server ansible_host=192.168.128.101 ansible_user=YOUR_USERNAME ansible_ssh_private_key_file=~/.ssh/id_rsa

[prod_server]
prod-server ansible_host=192.168.128.102 ansible_user=YOUR_USERNAME ansible_ssh_private_key_file=~/.ssh/id_rsa

[all:vars]
ansible_python_interpreter=/usr/bin/python3
```

### Step 5.2: Update Jenkinsfile

Choose either `Jenkinsfile` or `Jenkinsfile-with-prod` and update:

```groovy
environment {
    DOCKER_IMAGE = 'applebite-php-app'
    DOCKER_TAG = "${BUILD_NUMBER}"
    CONTAINER_NAME = 'applebite-container'
    TEST_SERVER = '192.168.128.101'      // Update this
    TEST_SERVER_USER = 'YOUR_USERNAME'    // Update this
    PROD_SERVER = '192.168.128.102'      // Update this
    PROD_SERVER_USER = 'YOUR_USERNAME'    // Update this
}
```



## Part 6: Start All Services

### Step 6.1: Start Master VM Services

```bash
wsl -d Ubuntu

# Start SSH
sudo service ssh start

# Start Jenkins
sudo service jenkins start

# Check Jenkins is running
sudo service jenkins status

# Get initial Jenkins password
sudo cat /var/lib/jenkins/secrets/initialAdminPassword

# Access Jenkins from Windows browser:
# http://localhost:8080
# Or http://MASTER_VM_IP:8080
```

### Step 6.2: Start Test Server Services

```bash
wsl -d TestServer

# Start SSH
sudo service ssh start

# Docker will be installed by Ansible during pipeline
```

### Step 6.3: Start Production Server Services

```bash
wsl -d ProdServer

# Start SSH
sudo service ssh start

# Docker will be installed by Ansible during pipeline
```



## Part 7: Quick Reference Commands

### Start All WSL Instances
```powershell
# From PowerShell
wsl -d Ubuntu
wsl -d TestServer
wsl -d ProdServer
```

### Check All WSL IPs
```powershell
# Create a script: check-wsl-ips.ps1
Write-Host "Master VM:    " -NoNewline; wsl -d Ubuntu hostname -I
Write-Host "Test Server:  " -NoNewline; wsl -d TestServer hostname -I
Write-Host "Prod Server:  " -NoNewline; wsl -d ProdServer hostname -I
```

### Shutdown All WSL Instances
```powershell
wsl --shutdown
```

### Start Services After Windows Reboot

**Create startup script: `start-wsl-services.ps1`**
```powershell
# Start Master VM services
wsl -d Ubuntu -u root service ssh start
wsl -d Ubuntu -u root service jenkins start

# Start Test Server SSH
wsl -d TestServer -u root service ssh start

# Start Prod Server SSH
wsl -d ProdServer -u root service ssh start

Write-Host "All WSL services started!"
```



## Part 8: Handle WSL IP Changes

### Problem: WSL IPs change after reboot

### Solution 1: Use Static IPs (Advanced)

```bash
# In each WSL instance, edit /etc/wsl.conf
sudo nano /etc/wsl.conf

# Add:
[network]
generateResolvConf = false

# Then configure static IP in /etc/netplan/ or /etc/network/interfaces
```

### Solution 2: Use Script to Update IPs

**Create `update-ips.sh` in Master VM:**
```bash
#!/bin/bash

TEST_IP=$(wsl -d TestServer hostname -I | tr -d ' ')
PROD_IP=$(wsl -d ProdServer hostname -I | tr -d ' ')

echo "Test Server IP: $TEST_IP"
echo "Prod Server IP: $PROD_IP"

# Update Ansible inventory
sed -i "s/ansible_host=.* ansible_user/ansible_host=$TEST_IP ansible_user/" ansible/inventory/hosts

# Update Jenkinsfile
sed -i "s/TEST_SERVER = '.*'/TEST_SERVER = '$TEST_IP'/" Jenkinsfile
sed -i "s/PROD_SERVER = '.*'/PROD_SERVER = '$PROD_IP'/" Jenkinsfile

echo "IPs updated successfully!"
```

### Solution 3: Use Hostnames (Recommended)

```powershell
# Add to Windows hosts file: C:\Windows\System32\drivers\etc\hosts
# (Run as Administrator)

192.168.128.101  wsl-test
192.168.128.102  wsl-prod

# Then use hostnames in configuration instead of IPs
```



## Part 9: Testing the Setup

### Test 1: SSH Connectivity

```bash
# From Master VM
ssh username@TEST_SERVER_IP
ssh username@PROD_SERVER_IP
```

### Test 2: Ansible Connectivity

```bash
# From Master VM
cd /path/to/project
ansible -i ansible/inventory/hosts test_server -m ping
ansible -i ansible/inventory/hosts prod_server -m ping
```

### Test 3: Run Pipeline

```bash
# Access Jenkins: http://localhost:8080
# Run the pipeline manually
# Verify deployment on Test and Prod servers
```



## Part 10: Troubleshooting

### Issue: Can't SSH to WSL instances

```bash
# Check SSH is running
wsl -d TestServer sudo service ssh status

# Start SSH
wsl -d TestServer sudo service ssh start

# Check SSH config
wsl -d TestServer cat /etc/ssh/sshd_config | grep PasswordAuthentication
```

### Issue: Jenkins can't connect to Test/Prod

```bash
# Verify SSH keys
ls -la ~/.ssh/
cat ~/.ssh/authorized_keys  # On Test and Prod

# Test SSH manually
ssh -v username@TEST_SERVER_IP
```

### Issue: IPs changed after reboot

```bash
# Re-check IPs
wsl -d TestServer hostname -I
wsl -d ProdServer hostname -I

# Update configuration files
# Run update-ips.sh script
```



## Part 11: Summary of IPs and Credentials

**Fill this out after setup:**

| Instance | WSL Name | IP Address | Username | SSH Key Location |
|----------|----------|------------|----------|------------------|
| Master | Ubuntu | ___.___.___.___ | ________ | ~/.ssh/id_rsa |
| Test | TestServer | ___.___.___.___ | ________ | Master's key |
| Prod | ProdServer | ___.___.___.___ | ________ | Master's key |

**Jenkins URL**: http://localhost:8080 or http://MASTER_IP:8080 **Test App
URL**: http://TEST_SERVER_IP/ **Prod App URL**: http://PROD_SERVER_IP/



## Next Steps

1. ✅ Create 3 WSL instances
2. ✅ Configure SSH between instances
3. ✅ Install Jenkins on Master
4. ✅ Get all IP addresses
5. ✅ Update project configuration files
6. ✅ Test SSH connectivity
7. ✅ Run Jenkins pipeline
8. ✅ Verify deployment on Test server
9. ✅ Verify deployment on Prod server



**Ready to proceed? Follow these steps in order and you'll have a complete 3-VM
setup!**
