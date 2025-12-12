# 🚀 START HERE - AppleBite CI/CD Project

## Welcome to Your Complete CI/CD Solution!

This project provides a **fully automated CI/CD pipeline** for AppleBite Co.
using Git, Jenkins, Docker, and Ansible.



## 📖 What You Have

A complete, production-ready CI/CD system that:
- ✅ Automatically deploys when you push to Git
- ✅ Uses git submodules for app code management
- ✅ Triggers on main repository pushes
  (https://github.com/NitishK1/Edureka_DevOps_Arch_Training.git)
- ✅ Runs tests at every stage
- ✅ Manages multiple environments (Test/Stage/Production)
- ✅ Includes comprehensive documentation
- ✅ Provides easy-to-use automation scripts



## 📁 Repository Structure

This project uses **git submodules**:
- **Main Repo**: `https://github.com/NitishK1/Edureka_DevOps_Arch_Training.git`
  (CI/CD configuration)
- **App Submodule**: `https://github.com/edureka-devops/projCert.git` (PHP
  application code)

The app is automatically cloned as a submodule at:
`AWS_Projects/AppleBite_CICD_Project/app`

See `SUBMODULE_SETUP.md` for detailed information.



## ⚡ Quick Start (Choose Your Path)

### Path 1: Interactive Menu (Easiest) 🎯

```bash
# Navigate to project
cd c:/Users/hardi/HARDIK/Learn/Edureka_DevOps_Arch_Training/AWS_Projects/AppleBite_CICD_Project

# Make scripts executable
chmod +x *.sh scripts/*.sh jenkins/*.sh

# Launch interactive menu
./quickstart.sh
```

**This menu gives you all options in a user-friendly interface!**



### Path 2: Step-by-Step Setup 📋

```bash
# 1. Verify prerequisites
./verify.sh

# 2. Run initial setup (clones app, sets up directories)
./scripts/setup.sh

# 3. Deploy to test environment
./scripts/deploy.sh test

# 4. Open in browser
# http://localhost:8080

# 5. Run tests
./scripts/test.sh test
```



### Path 3: Full Jenkins Pipeline 🔄

```bash
# 1. Install Jenkins
./jenkins/jenkins-setup.sh

# 2. Access Jenkins
# http://localhost:8090

# 3. Create pipeline job (follow prompts in script)

# 4. Push code to trigger automatic deployment
cd app
git add .
git commit -m "Test deployment"
git push origin master

# 5. Watch Jenkins automatically deploy!
```



## 📂 Project Structure Overview

```
AppleBite_CICD_Project/
│
├── 📘 Documentation (Read These!)
│   ├── START_HERE.md              ← You are here
│   ├── README.md                  ← Project overview
│   ├── SETUP_GUIDE.md            ← Detailed setup
│   ├── DEPLOYMENT_GUIDE.md       ← How to deploy
│   ├── ARCHITECTURE.md           ← System design
│   ├── QUICK_REFERENCE.md        ← Command cheat sheet
│   └── ASSIGNMENT_COMPLETION.md  ← Assignment summary
│
├── 🐳 Docker (Containerization)
│   ├── Dockerfile
│   ├── docker-compose.test.yml
│   ├── docker-compose.stage.yml
│   └── docker-compose.prod.yml
│
├── 🔧 Ansible (Automation)
│   ├── inventory/
│   └── playbooks/
│
├── 🔄 Jenkins (CI/CD)
│   ├── Jenkinsfile
│   └── jenkins-setup.sh
│
└── 📜 Scripts (Automation)
    ├── setup.sh      ← Initial setup
    ├── build.sh      ← Build images
    ├── deploy.sh     ← Deploy app
    ├── test.sh       ← Run tests
    └── cleanup.sh    ← Clean up
```



## 🎯 What Each Script Does

| Script | Purpose | Usage |
|--------|---------|-------|
| `quickstart.sh` | Interactive menu for everything | `./quickstart.sh` |
| `verify.sh` | Check if everything is set up | `./verify.sh` |
| `setup.sh` | Initial project setup | `./scripts/setup.sh` |
| `deploy.sh` | Deploy to environment | `./scripts/deploy.sh test` |
| `test.sh` | Run tests | `./scripts/test.sh test` |
| `build.sh` | Build Docker images | `./scripts/build.sh all` |
| `cleanup.sh` | Clean up resources | `./scripts/cleanup.sh containers` |



## 🌐 Access Points

Once deployed, access your application at:

| Environment | URL | Port |
|------------|-----|------|
| **Test** | http://localhost:8080 | 8080 |
| **Stage** | http://localhost:8081 | 8081 |
| **Production** | http://localhost:8082 | 8082 |
| **Jenkins** | http://localhost:8090 | 8090 |



## ✅ Prerequisites Checklist

Before you start, ensure you have:

- [ ] **Windows 10/11** with WSL2, or **Linux**, or **macOS**
- [ ] **Git** installed (`git --version`)
- [ ] **Docker Desktop** installed and running (`docker --version`)
- [ ] **4GB RAM** minimum (8GB recommended)
- [ ] **20GB disk space** available
- [ ] **Internet connection** (for downloading images)



## 🎓 Your First Deployment

### Complete Workflow (5 minutes)

```bash
# Step 1: Navigate to project
cd c:/Users/hardi/HARDIK/Learn/Edureka_DevOps_Arch_Training/AWS_Projects/AppleBite_CICD_Project

# Step 2: Make scripts executable
chmod +x scripts/*.sh jenkins/*.sh *.sh

# Step 3: Verify everything
./verify.sh

# Step 4: Run setup
./scripts/setup.sh

# Step 5: Deploy to test
./scripts/deploy.sh test

# Step 6: Open browser and visit
# http://localhost:8080

# Step 7: Run tests
./scripts/test.sh test

# 🎉 Success! You just deployed your first application!
```



## 📚 Documentation Guide

Read these in order:

1. **START_HERE.md** (this file) - Get started quickly
2. **README.md** - Understand the project
3. **SETUP_GUIDE.md** - Detailed setup instructions
4. **DEPLOYMENT_GUIDE.md** - Learn deployment procedures
5. **QUICK_REFERENCE.md** - Keep for quick commands
6. **ARCHITECTURE.md** - Understand the system design



## 🔧 Common Commands

### Deployment
```bash
./scripts/deploy.sh test   # Deploy to test
./scripts/deploy.sh stage  # Deploy to stage
./scripts/deploy.sh prod   # Deploy to production
```

### Testing
```bash
./scripts/test.sh test     # Test test environment
./scripts/test.sh stage    # Test stage environment
./scripts/test.sh prod     # Test production
```

### Monitoring
```bash
docker ps                  # View running containers
docker logs <container>    # View container logs
./quickstart.sh           # Use option 10 for status
```

### Cleanup
```bash
./scripts/cleanup.sh containers  # Stop containers
./scripts/cleanup.sh all        # Remove everything
./reset.sh                      # Complete reset (delete all)
```



## 🆘 Need Help?

### Quick Troubleshooting

**Docker not running?**
```bash
# Windows/Mac: Start Docker Desktop
# Linux: sudo systemctl start docker
```

**Port already in use?**
```bash
# Check what's using port 8080
netstat -ano | findstr :8080  # Windows
lsof -i :8080                 # Linux/Mac
```

**Application not accessible?**
```bash
# Check container status
docker ps

# View logs
cd docker
docker-compose -f docker-compose.test.yml logs
```

**Script permission denied?**
```bash
chmod +x scripts/*.sh jenkins/*.sh *.sh
```

### Get More Help

1. Check **SETUP_GUIDE.md** - Detailed troubleshooting section
2. Check **DEPLOYMENT_GUIDE.md** - Emergency procedures
3. Review logs: `docker-compose logs`
4. Verify prerequisites: `./verify.sh`



## 🎯 Learning Path

### Beginner
1. Run `./quickstart.sh` and explore the menu
2. Deploy to test environment
3. Access the application in browser
4. Read README.md

### Intermediate
1. Understand the Dockerfile
2. Learn Docker Compose configurations
3. Study the Jenkins pipeline
4. Read ARCHITECTURE.md

### Advanced
1. Customize the pipeline
2. Add your own tests
3. Configure remote servers
4. Implement monitoring



## 🚀 Next Steps

After your first successful deployment:

1. **Explore Jenkins** - Install and configure Jenkins pipeline
2. **Test Changes** - Make code changes and see automatic deployment
3. **Multi-Environment** - Deploy to stage and production
4. **Customize** - Adapt the setup for your needs
5. **Share** - Push to your own Git repository



## 🎯 Assignment Goals

This project achieves all assignment requirements:

✅ **Git** - Version control implemented ✅ **Jenkins** - CI/CD pipeline automated
✅ **Docker** - Application containerized ✅ **Ansible** - Server provisioning
automated ✅ **Automatic Deployment** - Push to deploy workflow

See **ASSIGNMENT_COMPLETION.md** for full details.



## 💡 Pro Tips

1. **Use the interactive menu** - `./quickstart.sh` is your friend
2. **Read the logs** - They tell you everything
3. **Test locally first** - Always deploy to test before production
4. **Keep scripts executable** - Run `chmod +x` when needed
5. **Read documentation** - We've documented everything!



## 🎉 You're Ready!

Everything is set up and ready to go. Choose your path above and start
deploying!

### Recommended First Step:
```bash
./quickstart.sh
```

This will guide you through everything with an easy-to-use menu.



## 📞 Quick Reference

| Need to... | Command |
|-----------|---------|
| Start fresh | `./scripts/setup.sh` |
| Deploy | `./scripts/deploy.sh test` |
| Test | `./scripts/test.sh test` |
| View logs | `docker-compose logs -f` |
| Stop all | `./scripts/cleanup.sh containers` |
| Get help | Read SETUP_GUIDE.md |



**Ready to begin? Run: `./quickstart.sh`** 🚀



**Version**: 1.0 **Last Updated**: December 11, 2025 **Status**: Ready for Use ✅
