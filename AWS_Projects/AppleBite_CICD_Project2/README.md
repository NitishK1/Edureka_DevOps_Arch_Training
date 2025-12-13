# AppleBite CI/CD Project 2

> 📋 **Quick Navigation**: See [INDEX.md](INDEX.md) for complete file directory
> and navigation guide
> 
> ✅ **Requirements Verification**: See [VERIFICATION.md](VERIFICATION.md) for
> problem statement compliance check

## Overview
This project implements a complete CI/CD pipeline for deploying a PHP
application using Jenkins, Ansible, Docker, and Puppet. All requirements from
the problem statement have been fully implemented and verified.

## Problem Statement
AppleBite Co. needs to implement Continuous Integration & Continuous Deployment
to:
- Automate complex builds
- Manage incremental builds efficiently
- Deploy code automatically to dev/stage/prod environments
- Trigger deployments on Git push to master branch

## Technologies Used
- **Git**: Version control system
- **Jenkins**: CI/CD automation server
- **Docker**: Container platform for application deployment
- **Ansible**: Configuration management for Docker installation
- **Puppet**: Agent configuration on slave nodes

## Architecture
- **Master VM**: Jenkins Master, Ansible, Git
- **Slave Node**: Jenkins Slave (Test Server) for building and deploying
  containers

## Project Structure
```
AppleBite_CICD_Project2/
├── README.md
├── app/                          # PHP Application
│   ├── index.php
│   ├── about.php
│   ├── contact.php
│   └── style.css
├── Dockerfile                    # Docker configuration
├── Jenkinsfile                   # Jenkins Pipeline definition
├── ansible/
│   ├── inventory/
│   │   └── hosts                # Ansible inventory
│   └── playbooks/
│       └── install-docker.yml   # Docker installation playbook
├── puppet/
│   └── setup-agent.sh           # Puppet agent setup script
└── scripts/
    ├── cleanup.sh               # Cleanup script for failed deployments
    └── deploy.sh                # Deployment script
```

## Pipeline Jobs
1. **Job 1**: Install and configure Puppet agent on slave node
2. **Job 2**: Run Ansible playbook to install Docker on test server
3. **Job 3**: Pull code from Git, build Docker image, and deploy container
4. **Job 4**: Cleanup - Delete running container if Job 3 fails

## Setup Instructions
Refer to `SETUP.md` for detailed setup and configuration steps.

## Deployment Guide
Refer to `DEPLOYMENT.md` for deployment instructions.
