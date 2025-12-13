# Setup Guide - AppleBite CI/CD Project

## Prerequisites

### Master VM Requirements
- Ubuntu 20.04 LTS or later
- Jenkins installed and running
- Ansible installed
- Git installed
- Minimum 2GB RAM, 2 vCPUs
- SSH access to slave node

### Slave Node Requirements
- Ubuntu 20.04 LTS or later (fresh instance)
- Python 3.x installed
- OpenSSH server installed
- Git installed
- Minimum 2GB RAM, 2 vCPUs
- Port 80 open for web access

## Step 1: Prepare Master VM

### 1.1 Install Jenkins
```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install Java
sudo apt install -y openjdk-11-jdk

# Add Jenkins repository
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null

echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null

# Install Jenkins
sudo apt update
sudo apt install -y jenkins

# Start Jenkins
sudo systemctl start jenkins
sudo systemctl enable jenkins

# Get initial admin password
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

### 1.2 Install Ansible
```bash
sudo apt update
sudo apt install -y software-properties-common
sudo add-apt-repository --yes --update ppa:ansible/ansible
sudo apt install -y ansible

# Verify installation
ansible --version
```

### 1.3 Install Git
```bash
sudo apt install -y git
git --version
```

### 1.4 Install Required Jenkins Plugins
1. Access Jenkins at `http://<MASTER_IP>:8080`
2. Go to **Manage Jenkins** → **Manage Plugins**
3. Install the following plugins:
   - Pipeline
   - Git Plugin
   - SSH Agent
   - Build Pipeline Plugin
   - Post-build Task Plugin
   - Ansible Plugin

## Step 2: Prepare Slave Node

### 2.1 Install Required Packages
```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install Python
sudo apt install -y python3 python3-pip

# Install OpenSSH Server
sudo apt install -y openssh-server
sudo systemctl start ssh
sudo systemctl enable ssh

# Install Git
sudo apt install -y git
```

### 2.2 Configure SSH Access
On the Master VM, generate SSH key and copy to slave:
```bash
# Generate SSH key (if not already done)
ssh-keygen -t rsa -b 4096 -C "jenkins@master"

# Copy SSH key to slave node
ssh-copy-id ubuntu@<SLAVE_NODE_IP>

# Test SSH connection
ssh ubuntu@<SLAVE_NODE_IP>
```

## Step 3: Configure Jenkins

### 3.1 Add Slave Node to Jenkins
1. Go to **Manage Jenkins** → **Manage Nodes and Clouds**
2. Click **New Node**
3. Enter node name: `test-server`
4. Select **Permanent Agent**
5. Configure:
   - Remote root directory: `/home/ubuntu/jenkins`
   - Labels: `test-server`
   - Launch method: **Launch agents via SSH**
   - Host: `<SLAVE_NODE_IP>`
   - Credentials: Add SSH credentials
   - Host Key Verification Strategy: **Non verifying**
6. Click **Save**

### 3.2 Configure Git Credentials
1. Go to **Manage Jenkins** → **Manage Credentials**
2. Add credentials for your Git repository
3. Select **Username with password** or **SSH Username with private key**

## Step 4: Clone and Configure Project

### 4.1 Clone Repository
```bash
cd /var/lib/jenkins/workspace
git clone <YOUR_REPO_URL> AppleBite_CICD_Project2
cd AppleBite_CICD_Project2
```

### 4.2 Update Configuration Files

#### Update Ansible Inventory
Edit `ansible/inventory/hosts`:
```ini
[test_server]
test-server ansible_host=<SLAVE_NODE_IP> ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/id_rsa
```

#### Update Jenkinsfile
Edit `Jenkinsfile` and replace `<TEST_SERVER_IP>` with your slave node IP.

## Step 5: Create Jenkins Pipeline

### 5.1 Create New Pipeline Job
1. Go to Jenkins Dashboard
2. Click **New Item**
3. Enter name: `AppleBite-CICD-Pipeline`
4. Select **Pipeline**
5. Click **OK**

### 5.2 Configure Pipeline
1. Under **Pipeline** section:
   - Definition: **Pipeline script from SCM**
   - SCM: **Git**
   - Repository URL: `<YOUR_REPO_URL>`
   - Branch: `*/main` or `*/master`
   - Script Path: `Jenkinsfile`
2. Under **Build Triggers**:
   - Enable **Poll SCM** or **GitHub hook trigger**
3. Click **Save**

## Step 6: Configure Git Webhook (Optional)

### For GitHub:
1. Go to your GitHub repository
2. Navigate to **Settings** → **Webhooks**
3. Click **Add webhook**
4. Payload URL: `http://<JENKINS_IP>:8080/github-webhook/`
5. Content type: `application/json`
6. Select **Just the push event**
7. Click **Add webhook**

## Step 7: Test Setup

### 7.1 Manual Pipeline Execution
1. Go to your pipeline job
2. Click **Build Now**
3. Monitor the console output

### 7.2 Verify Each Stage
- **Job 1**: Check if Puppet agent is installed on slave node
- **Job 2**: Verify Docker is installed on slave node
- **Job 3**: Check if application is deployed and accessible
- **Job 4**: Verify cleanup works on failure

### 7.3 Test Application
```bash
# From master VM
curl http://<SLAVE_NODE_IP>/

# Or open in browser
http://<SLAVE_NODE_IP>/
```

## Troubleshooting

### Jenkins cannot connect to slave node
```bash
# Check SSH connection
ssh ubuntu@<SLAVE_NODE_IP>

# Check SSH service on slave
sudo systemctl status ssh
```

### Ansible playbook fails
```bash
# Test Ansible connectivity
ansible -i ansible/inventory/hosts test_server -m ping

# Run playbook manually
cd ansible
ansible-playbook -i inventory/hosts playbooks/install-docker.yml -vvv
```

### Docker permission issues
```bash
# On slave node, add user to docker group
sudo usermod -aG docker ubuntu
sudo usermod -aG docker jenkins

# Restart SSH session or reboot
sudo reboot
```

### Container fails to start
```bash
# On slave node, check Docker logs
docker logs applebite-container

# Check if port 80 is already in use
sudo netstat -tlnp | grep :80
```

## Security Considerations

1. **Use SSH keys** instead of passwords
2. **Restrict firewall rules** to allow only necessary ports
3. **Use Jenkins credentials** management for sensitive data
4. **Enable HTTPS** for Jenkins
5. **Keep systems updated** with security patches

## Next Steps

Refer to `DEPLOYMENT.md` for deployment workflow and usage instructions.
