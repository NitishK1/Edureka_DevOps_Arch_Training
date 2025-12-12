# AppleBite CI/CD Project - Quick Reference Guide

## 🛠️ Quick Start Commands

### Initial Setup
```bash
# Navigate to project
cd c:/Users/hardi/HARDIK/Learn/Edureka_DevOps_Arch_Training/AWS_Projects/AppleBite_CICD_Project

# Make scripts executable
chmod +x scripts/*.sh jenkins/*.sh quickstart.sh reset.sh

# Run setup
./scripts/setup.sh

# Or use interactive menu
./quickstart.sh
```

### Deploy Commands
```bash
# Deploy to Test
./scripts/deploy.sh test

# Deploy to Stage
./scripts/deploy.sh stage

# Deploy to Production
./scripts/deploy.sh prod
```

### Test Commands
```bash
# Test Test environment
./scripts/test.sh test

# Test Stage environment
./scripts/test.sh stage

# Test Production
./scripts/test.sh prod
```

### Build Commands
```bash
# Build all environments
./scripts/build.sh all

# Build specific environment
./scripts/build.sh test
./scripts/build.sh stage
./scripts/build.sh prod
```

### Cleanup Commands
```bash
# Stop containers
./scripts/cleanup.sh containers

# Remove images
./scripts/cleanup.sh images

# Remove everything
./scripts/cleanup.sh all

# Complete reset (delete everything including app)
./reset.sh
```

## 🌐 Access URLs

| Environment | URL | Port |
|------------|-----|------|
| **Test** | http://localhost:8080 | 8080 |
| **Stage** | http://localhost:8081 | 8081 |
| **Production** | http://localhost:8082 | 8082 |
| **Jenkins** | http://localhost:8090 | 8090 |

## 🐳 Docker Commands

```bash
# View containers
docker ps -a

# View logs
docker-compose -f docker/docker-compose.test.yml logs -f

# Restart container
docker-compose -f docker/docker-compose.test.yml restart

# Stop environment
docker-compose -f docker/docker-compose.test.yml down

# Start environment
docker-compose -f docker/docker-compose.test.yml up -d

# View resource usage
docker stats
```

## 📊 Jenkins Pipeline

### Trigger Pipeline
1. Push to Git master branch (automatic)
2. Or click "Build Now" in Jenkins UI

### Pipeline Stages
1. Initialize
2. Checkout Code
3. Code Quality Check
4. Build Docker Image
5. Run Unit Tests
6. Provision Test Server
7. Deploy to Test
8. Integration Tests
9. Deploy to Stage
10. Smoke Tests on Stage
11. Approval for Production
12. Deploy to Production
13. Production Health Check

## 🔧 Ansible Commands

```bash
# Test connectivity
ansible all -i ansible/inventory/test.ini -m ping

# Provision server
ansible-playbook -i ansible/inventory/test.ini ansible/playbooks/provision-server.yml

# Deploy application
ansible-playbook -i ansible/inventory/test.ini ansible/playbooks/deploy-app.yml

# Rollback
ansible-playbook -i ansible/inventory/prod.ini ansible/playbooks/rollback.yml
```

## 🔍 Troubleshooting Quick Fixes

### Docker not running
```bash
# Start Docker Desktop (Windows/Mac)
# Or on Linux:
sudo systemctl start docker
```

### Port conflict
```bash
# Find what's using the port
netstat -ano | findstr :8080    # Windows
lsof -i :8080                   # Linux/Mac

# Kill the process or change port in docker-compose files
```

### Container not starting
```bash
# Check logs
docker logs <container-name>

# Restart container
docker restart <container-name>

# Rebuild and restart
docker-compose down
docker-compose build
docker-compose up -d
```

### Application not accessible
```bash
# Check container status
docker ps

# Check logs
docker-compose logs

# Verify health
curl http://localhost:8080
```

## 📚 Documentation Files

- **README.md** - Project overview and features
- **SETUP_GUIDE.md** - Complete setup instructions
- **DEPLOYMENT_GUIDE.md** - Deployment procedures
- **ARCHITECTURE.md** - System architecture details
- **QUICK_REFERENCE.md** - This file

## 🔐 Security Notes

- Never commit `.env` files
- Use strong passwords in production
- Keep Docker images updated
- Scan for vulnerabilities regularly
- Use HTTPS in production

## 📞 Getting Help

1. Check documentation files
2. Review Jenkins console output
3. Check Docker logs
4. Verify all services are running
5. Ensure all prerequisites are installed

## ✅ Daily Workflow

1. Pull latest changes: `git pull`
2. Make code changes
3. Test locally: `./scripts/deploy.sh test`
4. Run tests: `./scripts/test.sh test`
5. Commit: `git commit -m "Description"`
6. Push: `git push origin master`
7. Monitor Jenkins pipeline
8. Approve production deployment if needed



**Version**: 1.0 **Last Updated**: December 11, 2025
