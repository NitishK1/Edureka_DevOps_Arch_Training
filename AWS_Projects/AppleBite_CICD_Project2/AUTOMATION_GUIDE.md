# Complete Automation Setup - Quick Reference

## 🚀 Quick Start

Run the interactive menu:
```bash
cd scripts
./wsl-quickstart.sh
```

Then select option **1** for full automated setup!



## Automation Scripts Overview

### 1. **wsl-quickstart.sh** (Main Interactive Menu)
**Purpose**: Interactive menu for all WSL operations

**Features**:
- ✅ **Option 1**: Full automated setup (recommended)
- Manual step-by-step options (2-5)
- Utility options for checking status (6-9)

**Usage**:
```bash
./wsl-quickstart.sh
```



### 2. **auto-create-wsl.sh** (VM Creation)
**Purpose**: Automatically creates TestServer and ProdServer WSL instances

**What it does**:
- Detects your Master VM (Ubuntu or Ubuntu-22.04)
- Exports Master VM as base image
- Creates TestServer from base image
- Creates ProdServer from base image
- Displays all IP addresses

**Manual Usage**:
```bash
./auto-create-wsl.sh
```

**Output**: 3 WSL VMs ready (Master, TestServer, ProdServer)



### 3. **auto-setup-ssh.sh** (SSH Configuration)
**Purpose**: Configures SSH connectivity between all VMs

**What it does**:
- Installs and starts SSH service on all 3 VMs
- Generates SSH keys on Master VM
- Copies public key to TestServer and ProdServer
- Configures SSH config file on Master
- Tests SSH connections

**Manual Usage**:
```bash
./auto-setup-ssh.sh
```

**Output**: Password-less SSH from Master to Test/Prod servers



### 4. **auto-setup-jenkins-ansible.sh** (Tools Installation)
**Purpose**: Installs Jenkins, Ansible, and all prerequisites on Master VM

**What it does**:
- Updates system packages
- Installs Java 11 (for Jenkins)
- Installs Ansible
- Installs Git, Python3, pip
- Installs and starts Jenkins
- Installs and starts SSH service
- Displays Jenkins initial admin password

**Manual Usage**:
```bash
./auto-setup-jenkins-ansible.sh
```

**Output**:
- Jenkins running at http://MASTER_IP:8080
- Ansible ready for configuration management
- Git ready for version control

**Get Jenkins Password**:
```bash
wsl -d Ubuntu-22.04 sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```



### 5. **auto-update-config.sh** (Configuration Updates)
**Purpose**: Automatically updates all configuration files with current WSL IPs

**What it does**:
- Gets current TestServer and ProdServer IPs
- Updates `ansible/inventory/hosts` with correct IPs
- Updates `Jenkinsfile` with correct IPs
- Updates `Jenkinsfile-with-prod` with correct IPs
- Creates backups of original files

**Manual Usage**:
```bash
./auto-update-config.sh
```

**Output**: All configuration files updated with current IPs



### 6. **check-wsl-ips.sh** (IP Checker)
**Purpose**: Display IP addresses of all WSL instances

**Manual Usage**:
```bash
./check-wsl-ips.sh
```



### 7. **start-wsl-services.sh** (Service Starter)
**Purpose**: Start SSH and Jenkins services on all VMs (useful after Windows
reboot)

**Manual Usage**:
```bash
./start-wsl-services.sh
```



## Complete Setup Flow

### Option 1: Full Automated Setup (Recommended)
```bash
cd scripts
./wsl-quickstart.sh
# Choose option 1
```

This runs all 4 automation scripts in sequence:
1. Creates WSL VMs (10-15 minutes)
2. Configures SSH (2-3 minutes)
3. Installs Jenkins & Ansible (5-10 minutes)
4. Updates configuration files (instant)

**Total Time**: ~15-20 minutes



### Option 2: Manual Step-by-Step
If you want more control, run each script individually:

```bash
# Step 1: Create VMs
./auto-create-wsl.sh

# Step 2: Configure SSH
./auto-setup-ssh.sh

# Step 3: Install Tools
./auto-setup-jenkins-ansible.sh

# Step 4: Update Configs
./auto-update-config.sh
```



## After Setup

### 1. Access Jenkins
```bash
# Get Master VM IP
wsl -d Ubuntu-22.04 hostname -I

# Open in browser
http://MASTER_IP:8080
```

### 2. Get Jenkins Initial Password
```bash
wsl -d Ubuntu-22.04 sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

### 3. Verify Ansible Connectivity
```bash
wsl -d Ubuntu-22.04
cd /mnt/c/Users/hardi/HARDIK/Learn/Edureka_DevOps_Arch_Training/AWS_Projects/AppleBite_CICD_Project2
ansible all -i ansible/inventory/hosts -m ping
```

### 4. Create Jenkins Pipeline
1. Open Jenkins
2. New Item → Pipeline
3. Pipeline script from SCM → Git
4. Repository URL:
   `https://github.com/NitishK1/Edureka_DevOps_Arch_Training.git`
5. Script Path: `AWS_Projects/AppleBite_CICD_Project2/Jenkinsfile-with-prod`



## Troubleshooting

### IPs Changed After Reboot?
```bash
# Update all configs with new IPs
./auto-update-config.sh
```

### Services Not Running?
```bash
# Restart all services
./start-wsl-services.sh
```

### SSH Not Working?
```bash
# Re-run SSH setup
./auto-setup-ssh.sh
```

### Check Current Status
```bash
# View all IPs
./check-wsl-ips.sh

# Test SSH connections
./wsl-quickstart.sh
# Choose option 8
```



## Files Created/Modified

### Created by Scripts:
- `~/WSL-Instances/TestServer/` - TestServer VM files
- `~/WSL-Instances/ProdServer/` - ProdServer VM files
- `~/.ssh/id_rsa` - SSH private key (on Master)
- `~/.ssh/id_rsa.pub` - SSH public key (on Master)
- `~/.ssh/config` - SSH configuration (on Master)
- `~/.ssh/authorized_keys` - Public keys (on Test/Prod)

### Updated by Scripts:
- `ansible/inventory/hosts` - Ansible inventory with IPs
- `Jenkinsfile` - Jenkins pipeline (test only)
- `Jenkinsfile-with-prod` - Jenkins pipeline (test + prod)

### Backups Created:
- `Jenkinsfile.bak` - Original Jenkinsfile
- `Jenkinsfile-with-prod.bak` - Original Jenkinsfile-with-prod



## Architecture

```
┌─────────────────────────────────────────────────┐
│  Windows Host                                   │
│                                                 │
│  ┌──────────────┐  ┌──────────────┐           │
│  │  Master VM   │  │  TestServer  │           │
│  │  (Ubuntu)    │  │              │           │
│  │              │  │              │           │
│  │  - Jenkins   │  │  - Docker    │           │
│  │  - Ansible   │  │  - SSH       │           │
│  │  - Git       │  │              │           │
│  │              │  │              │           │
│  └──────┬───────┘  └──────▲───────┘           │
│         │                  │                    │
│         │    SSH           │                    │
│         └──────────────────┘                    │
│         │                                       │
│         │    SSH                                │
│         ▼                                       │
│  ┌──────────────┐                              │
│  │  ProdServer  │                              │
│  │              │                              │
│  │  - Docker    │                              │
│  │  - SSH       │                              │
│  │              │                              │
│  └──────────────┘                              │
│                                                 │
└─────────────────────────────────────────────────┘
```



## Next Steps

1. ✅ Run full automated setup
2. ✅ Access Jenkins and complete setup wizard
3. ✅ Install Jenkins plugins (Docker, Ansible)
4. ✅ Create Jenkins pipeline job
5. ✅ Push code to Git to trigger pipeline
6. 🎉 Watch automated deployment to Test → Prod!



**Created**: December 2025 **Project**: AppleBite CI/CD Project
**Documentation**: Complete automation for 3-VM WSL setup
