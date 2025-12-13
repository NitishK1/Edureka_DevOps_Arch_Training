# Quick Start Guide

## Prerequisites
- Master VM with Jenkins, Ansible, Git installed
- Slave Node with Python, SSH, Git installed
- SSH key copied to slave node

## Quick Setup (5 Steps)

### 1. Update Configuration
```bash
# Edit ansible/inventory/hosts
# Replace <TEST_SERVER_IP> with your slave node IP

# Edit Jenkinsfile
# Replace <TEST_SERVER_IP> with your slave node IP
```

### 2. Setup SSH Access
```bash
# On Master VM
ssh-keygen -t rsa
ssh-copy-id ubuntu@<SLAVE_NODE_IP>
ssh ubuntu@<SLAVE_NODE_IP>  # Test connection
```

### 3. Install Jenkins Plugins
- Pipeline
- Git Plugin
- SSH Agent
- Build Pipeline Plugin
- Post-build Task Plugin

### 4. Create Jenkins Pipeline
1. Jenkins → New Item → Pipeline
2. Name: `AppleBite-CICD-Pipeline`
3. Pipeline script from SCM → Git
4. Repository URL: `<YOUR_REPO_URL>`
5. Script Path: `Jenkinsfile`
6. Save

### 5. Run Pipeline
```bash
# Push code to Git
git add .
git commit -m "Initial commit"
git push origin main

# Or click "Build Now" in Jenkins
```

## Verify Deployment
```bash
# Check application
curl http://<SLAVE_NODE_IP>/

# Or open in browser
http://<SLAVE_NODE_IP>/
```

## Pipeline Jobs
1. **Job 1**: Install Puppet Agent on slave node
2. **Job 2**: Install Docker via Ansible
3. **Job 3**: Build and deploy Docker container
4. **Job 4**: Cleanup on failure (automatic)

## Common Commands

### Check Container Status
```bash
ssh ubuntu@<SLAVE_NODE_IP>
docker ps
```

### View Logs
```bash
docker logs applebite-container
```

### Manual Deployment
```bash
cd scripts
chmod +x deploy.sh
./deploy.sh v1.0
```

### Cleanup
```bash
cd scripts
chmod +x cleanup.sh
./cleanup.sh
```

## Troubleshooting

### Can't SSH to slave
```bash
ssh -v ubuntu@<SLAVE_NODE_IP>
```

### Ansible fails
```bash
ansible -i ansible/inventory/hosts test_server -m ping
```

### Docker issues
```bash
sudo systemctl status docker
sudo usermod -aG docker $USER
```

## Need More Help?
- Full setup: See `SETUP.md`
- Deployment details: See `DEPLOYMENT.md`
- Project overview: See `README.md`


**That's it! Your CI/CD pipeline is ready to use.**
