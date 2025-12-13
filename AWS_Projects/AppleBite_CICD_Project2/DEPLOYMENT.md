# Deployment Guide - AppleBite CI/CD Project

## Overview
This guide explains how the CI/CD pipeline works and how to deploy the AppleBite
PHP application.

## Pipeline Workflow

### Automatic Deployment Flow
```
Developer pushes code to Git
    ↓
Git webhook triggers Jenkins
    ↓
Jenkins Pipeline starts
    ↓
┌─────────────────────────────────┐
│ Job 1: Setup Puppet Agent       │
│ - Installs Puppet on slave node │
│ - Configures agent               │
└─────────────────────────────────┘
    ↓
┌─────────────────────────────────┐
│ Job 2: Install Docker           │
│ - Runs Ansible playbook         │
│ - Installs Docker on test server│
└─────────────────────────────────┘
    ↓
┌─────────────────────────────────┐
│ Job 3: Build & Deploy           │
│ - Pulls code from Git           │
│ - Builds Docker image           │
│ - Deploys container             │
│ - Verifies deployment           │
└─────────────────────────────────┘
    ↓
    Success? ──No──→ ┌─────────────────────────┐
    │                 │ Job 4: Cleanup          │
    │                 │ - Removes failed        │
    │                 │   container             │
    │                 └─────────────────────────┘
    Yes
    ↓
Application is live!
```

## Deployment Methods

### Method 1: Automatic Deployment (Recommended)
**Trigger**: Push to Git repository

1. Make changes to your code
2. Commit and push to Git repository:
   ```bash
   git add .
   git commit -m "Update application"
   git push origin main
   ```
3. Jenkins automatically detects the push and starts the pipeline
4. Monitor progress in Jenkins dashboard
5. Application is automatically deployed to test server

### Method 2: Manual Deployment via Jenkins
**Trigger**: Manual build in Jenkins

1. Go to Jenkins Dashboard
2. Select `AppleBite-CICD-Pipeline`
3. Click **Build Now**
4. Monitor the build progress
5. Check console output for deployment status

### Method 3: Manual Deployment via Script
**Trigger**: Direct script execution on test server

```bash
# SSH into test server
ssh ubuntu@<SLAVE_NODE_IP>

# Clone repository (first time only)
git clone <YOUR_REPO_URL> ~/applebite-app
cd ~/applebite-app

# Pull latest changes
git pull

# Run deployment script
chmod +x scripts/deploy.sh
./scripts/deploy.sh v1.0
```

## Pipeline Jobs Explained

### Job 1: Setup Puppet Agent
**Purpose**: Install and configure Puppet agent on the slave node

**What it does**:
- Checks if Puppet is already installed
- Downloads and installs Puppet 7 agent
- Configures Puppet for agent-master communication
- Verifies installation

**Manual execution**:
```bash
ssh ubuntu@<SLAVE_NODE_IP>
sudo bash puppet/setup-agent.sh
```

### Job 2: Install Docker
**Purpose**: Install Docker on the test server using Ansible

**What it does**:
- Updates package cache
- Installs Docker dependencies
- Adds Docker repository
- Installs Docker Engine
- Configures Docker service
- Adds users to docker group

**Manual execution**:
```bash
cd ansible
ansible-playbook -i inventory/hosts playbooks/install-docker.yml
```

### Job 3: Build and Deploy
**Purpose**: Build Docker image and deploy container

**What it does**:
- Pulls latest code from Git repository
- Copies files to test server
- Stops and removes old container
- Builds new Docker image with version tag
- Runs container on port 80
- Verifies deployment with health check

**Manual execution**:
```bash
# On test server
cd ~/applebite-app
git pull
docker build -t applebite-php-app:latest .
docker stop applebite-container 2>/dev/null || true
docker rm applebite-container 2>/dev/null || true
docker run -d --name applebite-container -p 80:80 applebite-php-app:latest
```

### Job 4: Cleanup on Failure
**Purpose**: Clean up failed deployments

**What it does**:
- Triggers only if Job 3 fails
- Stops failed container
- Removes failed container
- Optionally removes failed images
- Cleans up dangling resources

**Manual execution**:
```bash
ssh ubuntu@<SLAVE_NODE_IP>
cd ~/applebite-app
./scripts/cleanup.sh
```

## Verifying Deployment

### Check Container Status
```bash
ssh ubuntu@<SLAVE_NODE_IP>
docker ps | grep applebite-container
```

### Check Container Logs
```bash
ssh ubuntu@<SLAVE_NODE_IP>
docker logs applebite-container
```

### Test Application
```bash
# From any machine
curl http://<SLAVE_NODE_IP>/

# Or access in browser
http://<SLAVE_NODE_IP>/
```

### Check Application Health
```bash
curl http://<SLAVE_NODE_IP>/
curl http://<SLAVE_NODE_IP>/about.php
curl http://<SLAVE_NODE_IP>/contact.php
```

## Rollback Procedure

### Rollback to Previous Version
```bash
# SSH into test server
ssh ubuntu@<SLAVE_NODE_IP>

# List available images
docker images | grep applebite-php-app

# Stop current container
docker stop applebite-container
docker rm applebite-container

# Run previous version
docker run -d --name applebite-container -p 80:80 applebite-php-app:<PREVIOUS_VERSION>
```

### Rollback via Git
```bash
# On your local machine
git log --oneline
git revert <COMMIT_HASH>
git push origin main

# Jenkins will automatically deploy the reverted version
```

## Monitoring and Logs

### Jenkins Logs
- Go to Jenkins Dashboard → Pipeline → Console Output
- Check each stage for errors or warnings

### Docker Logs
```bash
ssh ubuntu@<SLAVE_NODE_IP>
docker logs -f applebite-container
```

### Ansible Logs
```bash
# On master VM
cat /var/log/jenkins/ansible.log
```

### System Logs
```bash
ssh ubuntu@<SLAVE_NODE_IP>
sudo journalctl -u docker
```

## Common Operations

### Update Application
```bash
# Edit files locally
vim app/index.php

# Commit and push
git add .
git commit -m "Updated homepage"
git push origin main

# Jenkins automatically deploys
```

### Scale Application (Manual)
```bash
ssh ubuntu@<SLAVE_NODE_IP>

# Run multiple containers on different ports
docker run -d --name applebite-container-2 -p 8080:80 applebite-php-app:latest
docker run -d --name applebite-container-3 -p 8081:80 applebite-php-app:latest
```

### Clean Up Old Images
```bash
ssh ubuntu@<SLAVE_NODE_IP>

# Remove dangling images
docker image prune -f

# Remove old versions
docker images | grep applebite-php-app
docker rmi applebite-php-app:<OLD_VERSION>
```

## Troubleshooting

### Pipeline Fails at Job 1 (Puppet)
- Check SSH connectivity to slave node
- Verify Puppet repository is accessible
- Check slave node has internet access

### Pipeline Fails at Job 2 (Ansible)
- Verify Ansible inventory file has correct IP
- Check SSH key permissions (`chmod 600 ~/.ssh/id_rsa`)
- Test Ansible connectivity:
  `ansible -i ansible/inventory/hosts test_server -m ping`

### Pipeline Fails at Job 3 (Deploy)
- Check Docker service is running: `sudo systemctl status docker`
- Verify port 80 is not in use: `sudo netstat -tlnp | grep :80`
- Check Dockerfile syntax
- Review Docker build logs

### Application Not Accessible
- Check firewall rules: `sudo ufw status`
- Verify container is running: `docker ps`
- Check container logs: `docker logs applebite-container`
- Test locally on server: `curl http://localhost/`

### Container Keeps Restarting
- Check logs: `docker logs applebite-container`
- Verify base image is available: `docker pull devopsedu/webapp`
- Check for port conflicts
- Verify application files are correct

## Best Practices

1. **Always test locally** before pushing to Git
2. **Use descriptive commit messages** for easier tracking
3. **Monitor Jenkins pipeline** after each push
4. **Check application** after deployment
5. **Keep backups** of working images
6. **Document changes** in commit messages
7. **Review logs** regularly for issues
8. **Update dependencies** periodically

## Environment Variables

The application supports the following environment variables:

- `APP_VERSION`: Application version (set automatically by Jenkins)

To add more environment variables:
```bash
# Update Jenkinsfile
docker run -d \
    --name applebite-container \
    -p 80:80 \
    -e APP_VERSION=${DOCKER_TAG} \
    -e CUSTOM_VAR=value \
    applebite-php-app:latest
```

## Performance Optimization

### Image Size Optimization
- Use `.dockerignore` file to exclude unnecessary files
- Use multi-stage builds if needed
- Clean up temporary files in Dockerfile

### Build Time Optimization
- Cache Docker layers effectively
- Use Docker BuildKit
- Parallelize independent tasks in Jenkins

## Security Checklist

- ✓ Use SSH keys instead of passwords
- ✓ Restrict Jenkins access with authentication
- ✓ Use Jenkins credentials for sensitive data
- ✓ Keep Docker and all tools updated
- ✓ Scan images for vulnerabilities
- ✓ Use firewall rules to restrict access
- ✓ Enable HTTPS for Jenkins and application
- ✓ Regularly review access logs

## Support and Maintenance

### Regular Maintenance Tasks
1. Update Jenkins plugins monthly
2. Update Docker and system packages weekly
3. Clean up old Docker images and containers
4. Review and rotate credentials quarterly
5. Backup Jenkins configuration regularly

### Getting Help
- Check Jenkins console output for errors
- Review Docker logs for container issues
- Test Ansible playbooks manually
- Verify network connectivity between nodes



**Project Completed!** The CI/CD pipeline is now fully functional and ready for
continuous deployment.
