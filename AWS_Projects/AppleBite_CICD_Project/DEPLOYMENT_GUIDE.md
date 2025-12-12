# AppleBite CI/CD Project - Deployment Guide

## Complete Deployment Procedures and Operations Guide

This guide covers all deployment scenarios, operations, and best practices for
the AppleBite CI/CD pipeline.



## Table of Contents

1. [Deployment Overview](#deployment-overview)
2. [Pre-Deployment Checklist](#pre-deployment-checklist)
3. [Deployment Procedures](#deployment-procedures)
4. [Pipeline Stages](#pipeline-stages)
5. [Rollback Procedures](#rollback-procedures)
6. [Monitoring and Logging](#monitoring-and-logging)
7. [Best Practices](#best-practices)
8. [Emergency Procedures](#emergency-procedures)



## Deployment Overview

### Deployment Flow

```
Developer Push → Git Webhook → Jenkins Trigger → Build → Test → Deploy Test →
Deploy Stage → Manual Approval → Deploy Production → Health Check → Monitor
```

### Environments

| Environment | Port | Purpose | Auto-Deploy | Approval Required |
|------------|------|---------|-------------|-------------------|
| **Test** | 8080 | Automated testing | ✅ Yes | ❌ No |
| **Stage** | 8081 | Pre-production validation | ✅ Yes | ❌ No |
| **Production** | 8082 | Live application | ⚠️ Manual | ✅ Yes |



## Pre-Deployment Checklist

### Before Every Deployment

- [ ] **Code Review**: All changes reviewed and approved
- [ ] **Tests Passing**: All unit and integration tests pass locally
- [ ] **Dependencies Updated**: Requirements and dependencies are up to date
- [ ] **Documentation**: Changes documented in code and README
- [ ] **Backup**: Current production state backed up
- [ ] **Rollback Plan**: Rollback procedure identified
- [ ] **Notification**: Team notified of upcoming deployment
- [ ] **Resources**: Sufficient server resources available

### Production Deployment Checklist

- [ ] **Staging Validated**: Application working correctly in stage
- [ ] **Load Testing**: Performance tests completed
- [ ] **Security Scan**: No critical vulnerabilities
- [ ] **Database Migrations**: Tested and ready (if applicable)
- [ ] **Monitoring**: Alerts and monitoring configured
- [ ] **Communication**: Stakeholders informed
- [ ] **Maintenance Window**: Scheduled if needed
- [ ] **Team Availability**: DevOps team available for support



## Deployment Procedures

### Method 1: Automated Pipeline (Recommended)

#### Trigger Deployment via Git Push

```bash
# Make your changes
cd app
# Edit files...

# Commit and push
git add .
git commit -m "Feature: Your feature description"
git push origin master
```

**What Happens:**
1. ✅ Git webhook triggers Jenkins
2. ✅ Jenkins pulls latest code
3. ✅ Builds Docker image
4. ✅ Runs automated tests
5. ✅ Deploys to Test environment
6. ✅ Deploys to Stage environment
7. ⏸️ Waits for manual approval
8. ✅ Deploys to Production (after approval)

#### Monitor Pipeline in Jenkins

1. Open Jenkins: http://localhost:8090
2. Click on `AppleBite-CICD-Pipeline`
3. Watch the stage view
4. Approve production deployment when prompted

### Method 2: Manual Deployment via Scripts

#### Deploy to Test Environment

```bash
cd c:/Users/hardi/HARDIK/Learn/Edureka_DevOps_Arch_Training/AWS_Projects/AppleBite_CICD_Project

# Deploy
./scripts/deploy.sh test

# Verify
./scripts/test.sh test

# Check application
curl http://localhost:8080
```

#### Deploy to Stage Environment

```bash
# Deploy
./scripts/deploy.sh stage

# Verify
./scripts/test.sh stage

# Check application
curl http://localhost:8081
```

#### Deploy to Production Environment

```bash
# Deploy (requires confirmation)
./scripts/deploy.sh prod

# You will be prompted:
# WARNING: Deploying to PRODUCTION
# Are you sure? (yes/no): yes

# Verify
./scripts/test.sh prod

# Check application
curl http://localhost:8082
```

### Method 3: Ansible Deployment

#### Full Provisioning and Deployment

```bash
cd ansible

# Provision server
ansible-playbook -i inventory/test.ini playbooks/provision-server.yml

# Deploy application
ansible-playbook -i inventory/test.ini playbooks/deploy-app.yml
```

#### Deploy to Specific Environment

```bash
# Test
ansible-playbook -i inventory/test.ini playbooks/deploy-app.yml -e "environment=test"

# Stage
ansible-playbook -i inventory/stage.ini playbooks/deploy-app.yml -e "environment=stage"

# Production
ansible-playbook -i inventory/prod.ini playbooks/deploy-app.yml -e "environment=production"
```

### Method 4: Direct Docker Compose

#### Quick Deployment

```bash
cd docker

# Test environment
docker-compose -f docker-compose.test.yml up -d

# Stage environment
docker-compose -f docker-compose.stage.yml up -d

# Production environment
docker-compose -f docker-compose.prod.yml up -d
```



## Pipeline Stages

### Stage 1: Initialize

**Purpose**: Set up pipeline environment variables and display build
information.

**Duration**: ~5 seconds

**What to Check**: Build number and branch name are correct.

### Stage 2: Checkout Code

**Purpose**: Clone the application repository.

**Duration**: ~10-30 seconds

**Potential Issues**:
- Git credentials not configured
- Repository URL incorrect
- Network connectivity issues

### Stage 3: Code Quality Check

**Purpose**: Run PHP syntax validation.

**Duration**: ~10-20 seconds

**Potential Issues**:
- PHP syntax errors in code
- PHP not installed in Jenkins environment

### Stage 4: Build Docker Image

**Purpose**: Build containerized application.

**Duration**: ~2-5 minutes

**Potential Issues**:
- Dockerfile syntax errors
- Base image not available
- Insufficient disk space

### Stage 5: Run Unit Tests

**Purpose**: Execute automated unit tests.

**Duration**: ~30 seconds

**Potential Issues**:
- Test failures
- Missing test dependencies

### Stage 6: Provision Test Server

**Purpose**: Ensure test server has required software.

**Duration**: ~1-3 minutes

**Potential Issues**:
- Ansible connection failed
- Server not accessible
- Insufficient permissions

### Stage 7: Deploy to Test

**Purpose**: Deploy application to test environment.

**Duration**: ~1-2 minutes

**Potential Issues**:
- Port conflicts
- Container startup failures
- Volume mount issues

### Stage 8: Integration Tests

**Purpose**: Run integration tests on deployed application.

**Duration**: ~30-60 seconds

**Potential Issues**:
- Application not responding
- Database connection failures
- API endpoints returning errors

### Stage 9: Deploy to Stage

**Purpose**: Deploy to staging environment.

**Duration**: ~1-2 minutes

**When**: Only on master branch

### Stage 10: Smoke Tests on Stage

**Purpose**: Quick validation of stage deployment.

**Duration**: ~30 seconds

### Stage 11: Approval for Production

**Purpose**: Manual gate before production deployment.

**Duration**: Variable (human approval)

**Who Can Approve**: admin, devops (configured in Jenkinsfile)

### Stage 12: Deploy to Production

**Purpose**: Deploy to live environment.

**Duration**: ~2-3 minutes

**Additional Steps**:
- Backup current production
- Graceful shutdown
- Health checks

### Stage 13: Production Health Check

**Purpose**: Verify production deployment success.

**Duration**: ~30 seconds

**Critical**: If this fails, initiate rollback immediately.



## Rollback Procedures

### Automatic Rollback via Ansible

```bash
cd ansible

# Rollback test environment
ansible-playbook -i inventory/test.ini playbooks/rollback.yml -e "environment=test"

# Rollback production
ansible-playbook -i inventory/prod.ini playbooks/rollback.yml -e "environment=production"
```

### Manual Rollback via Docker

```bash
cd docker

# Stop current containers
docker-compose -f docker-compose.prod.yml down

# Restore from backup image (if created)
docker run -d --name applebite-prod-web-backup <backup-image-id>

# Or redeploy previous version
git checkout <previous-commit-hash>
docker-compose -f docker-compose.prod.yml up -d
```

### Emergency Rollback

```bash
# Quick restore to last known good state

# 1. Stop current deployment
cd docker
docker-compose -f docker-compose.prod.yml down

# 2. Pull previous image
docker pull applebite-app:previous-tag

# 3. Update compose to use previous tag
# Edit docker-compose.prod.yml

# 4. Start containers
docker-compose -f docker-compose.prod.yml up -d

# 5. Verify
curl http://localhost:8082
```



## Monitoring and Logging

### View Container Logs

```bash
cd docker

# Test environment logs
docker-compose -f docker-compose.test.yml logs -f

# Specific service logs
docker-compose -f docker-compose.test.yml logs -f web

# Last 100 lines
docker-compose -f docker-compose.test.yml logs --tail=100 web
```

### View Application Logs

```bash
# Access container
docker exec -it applebite-test-web bash

# View Apache logs
tail -f /var/log/apache2/error.log
tail -f /var/log/apache2/access.log
```

### View Jenkins Logs

```bash
# Jenkins container logs
docker logs jenkins-applebite -f

# Specific build logs
# Available in Jenkins UI: Build → Console Output
```

### Health Check Endpoints

```bash
# Check application health
curl http://localhost:8080     # Test
curl http://localhost:8081     # Stage
curl http://localhost:8082     # Production

# Detailed health check with response time
curl -w "@-" -o /dev/null -s -w 'Response Time: %{time_total}s\nHTTP Code: %{http_code}\n' http://localhost:8082
```

### Container Status Monitoring

```bash
# View running containers
docker ps

# Container resource usage
docker stats

# Specific environment
docker ps | grep applebite-test
docker ps | grep applebite-stage
docker ps | grep applebite-prod
```



## Best Practices

### Development Workflow

1. **Create Feature Branch**
   ```bash
   git checkout -b feature/new-feature
   ```

2. **Develop and Test Locally**
   ```bash
   ./scripts/deploy.sh test
   ./scripts/test.sh test
   ```

3. **Commit with Meaningful Messages**
   ```bash
   git commit -m "feat: Add user authentication module"
   ```

4. **Push and Create PR**
   ```bash
   git push origin feature/new-feature
   ```

5. **Merge to Master After Review**

### Deployment Best Practices

✅ **DO:**
- Deploy during low-traffic hours
- Test in stage before production
- Keep deployments small and frequent
- Document all changes
- Monitor after deployment
- Have rollback plan ready

❌ **DON'T:**
- Skip testing stages
- Deploy on Fridays (unless necessary)
- Deploy multiple features at once
- Ignore test failures
- Deploy without backup
- Make direct production changes

### Security Best Practices

- 🔒 Use secrets management (not hardcoded passwords)
- 🔒 Scan Docker images for vulnerabilities
- 🔒 Keep dependencies updated
- 🔒 Use HTTPS in production
- 🔒 Implement proper access controls
- 🔒 Regular security audits



## Emergency Procedures

### Scenario 1: Production Site Down

```bash
# 1. Check container status
docker ps -a | grep prod

# 2. Check logs
cd docker
docker-compose -f docker-compose.prod.yml logs

# 3. Restart containers
docker-compose -f docker-compose.prod.yml restart

# 4. If restart fails, rollback
ansible-playbook -i ansible/inventory/prod.ini ansible/playbooks/rollback.yml
```

### Scenario 2: Database Connection Failed

```bash
# 1. Check database container
docker ps | grep db

# 2. Restart database
docker-compose -f docker-compose.prod.yml restart db

# 3. Check database logs
docker-compose -f docker-compose.prod.yml logs db

# 4. Verify connection
docker exec -it applebite-prod-db mysql -u root -p
```

### Scenario 3: High CPU/Memory Usage

```bash
# 1. Check resource usage
docker stats

# 2. Scale down if using swarm
docker service scale applebite-web=1

# 3. Restart containers
docker-compose -f docker-compose.prod.yml restart

# 4. Review logs for issues
docker-compose -f docker-compose.prod.yml logs --tail=500
```

### Scenario 4: Pipeline Failure

1. **Check Jenkins Console Output**
   - Identify failed stage
   - Review error messages

2. **Common Fixes:**
   - Docker daemon not running → Start Docker
   - Port conflict → Change ports in compose files
   - Test failure → Fix code and re-run
   - Ansible connection → Check SSH keys

3. **Re-run Pipeline**
   - Fix the issue
   - Click "Build Now" in Jenkins



## Contact and Support

### Escalation Path

1. **Level 1**: DevOps Team Lead
2. **Level 2**: System Administrator
3. **Level 3**: CTO / Technical Director

### Communication Channels

- **Slack**: #devops-alerts
- **Email**: devops@applebite.com
- **Phone**: Emergency hotline



## Appendix

### Useful Commands Reference

```bash
# Quick deployment
./scripts/deploy.sh [test|stage|prod]

# Run tests
./scripts/test.sh [test|stage|prod]

# Build images
./scripts/build.sh [test|stage|prod|all]

# Cleanup
./scripts/cleanup.sh [containers|images|volumes|all]

# View logs
docker-compose -f docker/docker-compose.test.yml logs -f

# Container shell access
docker exec -it applebite-test-web bash

# Database access
docker exec -it applebite-test-db mysql -u root -p
```



**Deployment Guide Version**: 1.0 **Last Updated**: December 11, 2025
**Maintainer**: AppleBite DevOps Team
