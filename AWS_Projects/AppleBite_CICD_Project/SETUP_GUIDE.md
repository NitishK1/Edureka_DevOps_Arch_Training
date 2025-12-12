# AppleBite CI/CD Project - Setup Guide

## Complete Installation and Configuration Guide

This guide will walk you through setting up the complete CI/CD pipeline for
AppleBite Co.'s PHP application on your local system.



## Table of Contents

1. [Prerequisites](#prerequisites)
2. [System Requirements](#system-requirements)
3. [Initial Setup](#initial-setup)
4. [Docker Configuration](#docker-configuration)
5. [Jenkins Setup](#jenkins-setup)
6. [Ansible Configuration](#ansible-configuration)
7. [Git Configuration](#git-configuration)
8. [Testing the Setup](#testing-the-setup)
9. [Troubleshooting](#troubleshooting)



## Prerequisites

### Required Software

| Software | Version | Purpose |
|----------|---------|---------|
| **Git** | Latest | Version control |
| **Docker Desktop** | Latest | Containerization |
| **Jenkins** | LTS | CI/CD automation |
| **Ansible** | 2.9+ | Configuration management |
| **Bash/WSL** | - | Script execution (Windows) |

### Optional Software

- **Visual Studio Code** - For editing files
- **Postman** - For API testing
- **Git Bash** - Alternative to WSL on Windows



## System Requirements

### Hardware Requirements

- **CPU**: 4 cores minimum (8 cores recommended)
- **RAM**: 8GB minimum (16GB recommended)
- **Disk**: 20GB free space
- **Network**: Internet connection for downloading dependencies

### Operating Systems

- ✅ **Windows 10/11** with WSL2
- ✅ **macOS** 10.15+
- ✅ **Linux** (Ubuntu 20.04+, Debian, CentOS)



## Initial Setup

### Step 1: Navigate to Project Directory

```bash
cd c:/Users/hardi/HARDIK/Learn/Edureka_DevOps_Arch_Training/AWS_Projects/AppleBite_CICD_Project
```

### Step 2: Make Scripts Executable

```bash
chmod +x scripts/*.sh
chmod +x jenkins/*.sh
```

### Step 3: Run Initial Setup Script

```bash
./scripts/setup.sh
```

This script will:
- ✅ Check for required dependencies
- ✅ Clone the PHP application from GitHub
- ✅ Create necessary directory structure
- ✅ Set up Git hooks
- ✅ Configure initial environment

**Expected Output:**
```
======================================
AppleBite CI/CD Project Setup
======================================

Checking prerequisites...

✓ Git is installed
✓ Docker is installed
✓ Docker daemon is running

======================================
Cloning PHP Application
======================================

Cloning projCert repository...
✓ Application cloned successfully

Setup Complete!
```



## Docker Configuration

### Step 1: Verify Docker Installation

```bash
docker --version
docker-compose --version
```

### Step 2: Test Docker

```bash
docker run hello-world
```

### Step 3: Configure Docker Resources

**For Docker Desktop (Windows/Mac):**

1. Open Docker Desktop
2. Go to Settings → Resources
3. Set:
   - **CPUs**: 4 (minimum)
   - **Memory**: 4GB (minimum)
   - **Swap**: 1GB
   - **Disk image size**: 60GB

### Step 4: Build Docker Images

```bash
cd docker
./scripts/build.sh all
```

This builds images for all environments (test, stage, prod).

### Step 5: Verify Images

```bash
docker images | grep applebite
```

**Expected Output:**
```
applebite-app    test    <image-id>    <time>    <size>
applebite-app    stage   <image-id>    <time>    <size>
applebite-app    prod    <image-id>    <time>    <size>
```



## Jenkins Setup

### Option 1: Jenkins in Docker (Recommended for Windows)

```bash
cd jenkins
./jenkins-setup.sh
```

Select option 1 when prompted.

**Access Jenkins:**
- URL: http://localhost:8090
- Get initial password: The script will display it

### Option 2: Native Jenkins Installation (Linux/Mac)

```bash
cd jenkins
sudo ./jenkins-setup.sh
```

**Access Jenkins:**
- URL: http://localhost:8080

### Step 1: Initial Jenkins Configuration

1. **Unlock Jenkins**
   - Enter the initial admin password displayed by the setup script
   - Or get it from:
     `docker exec jenkins-applebite cat /var/jenkins_home/secrets/initialAdminPassword`

2. **Install Plugins**
   - Click "Install suggested plugins"
   - Wait for installation to complete

3. **Install Additional Required Plugins**
   - Go to: Manage Jenkins → Manage Plugins → Available
   - Search and install:
     - ✅ Docker Pipeline
     - ✅ Ansible Plugin
     - ✅ Git Plugin
     - ✅ Pipeline Plugin
     - ✅ GitHub Integration Plugin

4. **Create Admin User**
   - Username: `admin`
   - Password: Choose a strong password
   - Full name: Your name
   - Email: Your email

### Step 2: Configure Jenkins Credentials

1. Go to: Manage Jenkins → Manage Credentials → (global) → Add Credentials

2. **Add Git Credentials** (if using private repo)
   - Kind: Username with password
   - Scope: Global
   - Username: Your GitHub username
   - Password: Your GitHub token
   - ID: `github-credentials`

3. **Add Docker Hub Credentials** (optional)
   - Kind: Username with password
   - Scope: Global
   - Username: Your Docker Hub username
   - Password: Your Docker Hub password
   - ID: `dockerhub-credentials`

### Step 3: Create Jenkins Pipeline Job

1. **New Item**
   - Click "New Item"
   - Name: `AppleBite-CICD-Pipeline`
   - Type: Pipeline
   - Click OK

2. **Configure Pipeline**
   - Description: `CI/CD Pipeline for AppleBite Application`

3. **Pipeline Script Options:**

   **Option A: Pipeline from SCM (Recommended)**
   - Definition: Pipeline script from SCM
   - SCM: Git
   - Repository URL: Your GitHub repo URL
   - Credentials: Select your Git credentials
   - Branch: `*/master`
   - Script Path: `jenkins/Jenkinsfile`

   **Option B: Pipeline Script**
   - Definition: Pipeline script
   - Copy content from `jenkins/Jenkinsfile`
   - Paste into Script section

4. **Build Triggers**
   - ✅ GitHub hook trigger for GITScm polling
   - ✅ Poll SCM: `H/5 * * * *` (check every 5 minutes)

5. **Save**



## Ansible Configuration

### Step 1: Install Ansible (if not installed)

**Ubuntu/Debian:**
```bash
sudo apt-get update
sudo apt-get install -y ansible
```

**macOS:**
```bash
brew install ansible
```

**Windows (WSL):**
```bash
sudo apt-get update
sudo apt-get install -y ansible
```

### Step 2: Verify Installation

```bash
ansible --version
```

### Step 3: Test Ansible Connection

```bash
cd ansible
ansible all -i inventory/test.ini -m ping
```

**Expected Output:**
```
localhost | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

### Step 4: Test Provisioning Playbook

```bash
ansible-playbook -i inventory/test.ini playbooks/provision-server.yml --check
```



## Git Configuration

### Step 1: Configure Git (if not done)

```bash
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
```

### Step 2: Initialize Git Repository (Optional)

If you want to version control your setup:

```bash
cd c:/Users/hardi/HARDIK/Learn/Edureka_DevOps_Arch_Training/AWS_Projects/AppleBite_CICD_Project
git init
git add .
git commit -m "Initial CI/CD project setup"
```

### Step 3: Connect to Remote Repository (Optional)

```bash
git remote add origin <your-github-repo-url>
git push -u origin master
```

### Step 4: Set Up GitHub Webhook (for automatic triggers)

1. Go to your GitHub repository
2. Settings → Webhooks → Add webhook
3. Payload URL: `http://your-jenkins-url:8090/github-webhook/`
4. Content type: `application/json`
5. Events: Just the push event
6. Active: ✅
7. Add webhook



## Testing the Setup

### Test 1: Manual Deployment

```bash
# Deploy to test environment
./scripts/deploy.sh test
```

**Expected Result:**
- Application accessible at http://localhost:8080

### Test 2: Run Tests

```bash
./scripts/test.sh test
```

**Expected Output:**
```
======================================
AppleBite Application Testing
======================================

Testing test environment on port 8080

Test 1: Application Health Check ... PASSED
Test 2: HTTP Response Code 200 ... PASSED
Test 3: Application Content Check ... PASSED
Test 4: Docker Container Running ... PASSED
Test 5: Database Container Running ... PASSED

======================================
Test Summary
======================================

Total Tests: 5
Passed: 5
Failed: 0

All tests passed! ✓
```

### Test 3: Jenkins Pipeline

1. Go to Jenkins dashboard
2. Click on `AppleBite-CICD-Pipeline`
3. Click "Build Now"
4. Watch the pipeline stages execute
5. Check console output for any errors

### Test 4: Full Pipeline Test

```bash
# Make a change to the app
cd app
echo "<!-- Test change -->" >> index.html
git add .
git commit -m "Test CI/CD pipeline"
git push origin master
```

Watch Jenkins automatically trigger the build!



## Troubleshooting

### Common Issues and Solutions

#### Issue 1: Docker Daemon Not Running

**Error:**
```
Cannot connect to the Docker daemon
```

**Solution:**
```bash
# Windows: Start Docker Desktop
# Linux:
sudo systemctl start docker
sudo systemctl enable docker
```

#### Issue 2: Port Already in Use

**Error:**
```
Port 8080 is already allocated
```

**Solution:**
```bash
# Check what's using the port
netstat -ano | findstr :8080  # Windows
lsof -i :8080                 # Linux/Mac

# Stop the conflicting service or change port in docker-compose files
```

#### Issue 3: Permission Denied on Scripts

**Error:**
```
Permission denied: ./setup.sh
```

**Solution:**
```bash
chmod +x scripts/*.sh
chmod +x jenkins/*.sh
```

#### Issue 4: Ansible Connection Failed

**Error:**
```
Failed to connect to the host
```

**Solution:**
```bash
# For localhost, ensure Python is installed
python3 --version

# Check inventory file
cat ansible/inventory/test.ini
```

#### Issue 5: Jenkins Build Failed

**Solution:**
1. Check console output in Jenkins
2. Verify Docker is running
3. Check if ports are available
4. Review Jenkinsfile syntax
5. Ensure all dependencies are installed

#### Issue 6: Application Not Accessible

**Solution:**
```bash
# Check container status
docker ps -a

# Check logs
cd docker
docker-compose -f docker-compose.test.yml logs

# Restart containers
docker-compose -f docker-compose.test.yml restart
```



## Environment Variables

Create `.env` file in `docker/` directory:

```bash
cp docker/.env.example docker/.env
```

Edit `.env` and update:
```env
PROD_DB_ROOT_PASSWORD=your_secure_root_password
PROD_DB_PASSWORD=your_secure_db_password
DOCKER_HUB_USERNAME=your_dockerhub_username
```



## Next Steps

After successful setup:

1. ✅ **Review** the [DEPLOYMENT_GUIDE.md](DEPLOYMENT_GUIDE.md)
2. ✅ **Customize** the application code
3. ✅ **Configure** production environment
4. ✅ **Set up** monitoring and alerting
5. ✅ **Implement** additional security measures
6. ✅ **Document** your specific configurations



## Additional Resources

- [Docker Documentation](https://docs.docker.com/)
- [Jenkins Documentation](https://www.jenkins.io/doc/)
- [Ansible Documentation](https://docs.ansible.com/)
- [PHP Application Repo](https://github.com/edureka-devops/projCert)



## Support

For issues:
1. Check the [Troubleshooting](#troubleshooting) section
2. Review logs in `logs/` directory
3. Check Docker and Jenkins logs
4. Refer to component documentation



**Setup Guide Version**: 1.0 **Last Updated**: December 11, 2025 **Maintainer**:
AppleBite DevOps Team
