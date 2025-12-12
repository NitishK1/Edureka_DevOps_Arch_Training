# Git Submodule Setup - Summary of Changes

## Overview

The project has been reconfigured to use **git submodules** for the application
code:

- **Main Repository**:
  `https://github.com/NitishK1/Edureka_DevOps_Arch_Training.git`
- **App Submodule**: `https://github.com/edureka-devops/projCert.git` (located
  at `AWS_Projects/AppleBite_CICD_Project/app`)

## What Changed

### 1. **setup.sh** - Submodule Initialization
- ✅ Updated to use `git submodule add` instead of `git clone`
- ✅ Automatically detects if in a git repository
- ✅ Falls back to regular clone if not in a git repo
- ✅ Handles existing submodule configurations
- ✅ Initializes and updates submodules recursively

### 2. **Jenkinsfile** - Repository Structure
- ✅ Updated `PROJECT_DIR` to: `${WORKSPACE}/AWS_Projects/AppleBite_CICD_Project`
- ✅ Updated `APP_DIR` to: `${PROJECT_DIR}/app`
- ✅ Changed checkout stage to use main repository with submodule support
- ✅ Configured to checkout from:
  `https://github.com/NitishK1/Edureka_DevOps_Arch_Training.git`
- ✅ Branch changed from `*/master` to `*/main`
- ✅ Added `SubmoduleOption` with `recursiveSubmodules: true`
- ✅ Updated all stage paths to use `${PROJECT_DIR}`

### 3. **configure-jenkins.sh** - Jenkins Configuration
- ✅ Updated repository URL to main repo
- ✅ Added instructions for submodule configuration
- ✅ Updated Script Path to:
  `AWS_Projects/AppleBite_CICD_Project/jenkins/Jenkinsfile`
- ✅ Added webhook and polling setup instructions
- ✅ Updated final summary with submodule information

### 4. **GITHUB_WEBHOOK_SETUP.md** - New Documentation
- ✅ Created comprehensive webhook setup guide
- ✅ Instructions for GitHub webhook configuration
- ✅ Alternative polling method for local development
- ✅ Submodule management commands
- ✅ Troubleshooting section

## How It Works

### Initial Setup (After Reset)

1. **Run quickstart.sh → Option 1 (Initial Setup)**
   ```bash
   ./quickstart.sh
   # Select: 1) Initial Setup (First Time)
   ```

2. **What happens automatically:**
   - Checks if in git repository
   - Adds `projCert` as git submodule at
     `AWS_Projects/AppleBite_CICD_Project/app`
   - Initializes and updates submodules
   - Creates directory structure
   - Sets up git hooks

3. **Install Jenkins → Option 2**
   ```bash
   # Select: 2) Install Jenkins
   ```

4. **Configure Jenkins → Option 3**
   ```bash
   # Select: 3) Configure Jenkins
   ```
   - Follow prompts to configure pipeline
   - Pipeline will be set to use main repository
   - Submodules will be automatically checked out

### Jenkins Pipeline Trigger

**Method 1: GitHub Webhook (Recommended)**
```bash
# Push to main repository triggers Jenkins
cd /c/Users/hardi/HARDIK/Learn/Edureka_DevOps_Arch_Training
git add .
git commit -m "Update configuration"
git push origin main
# → Jenkins automatically builds and deploys
```

**Method 2: GitHub Polling**
- Jenkins checks GitHub every 5 minutes
- Automatically triggers build on changes

**Method 3: Manual Build**
- Click "Build Now" in Jenkins dashboard

### Updating App Code

**Option 1: Update submodule reference**
```bash
cd /c/Users/hardi/HARDIK/Learn/Edureka_DevOps_Arch_Training/AWS_Projects/AppleBite_CICD_Project/app
git checkout master
git pull origin master

cd /c/Users/hardi/HARDIK/Learn/Edureka_DevOps_Arch_Training
git add AWS_Projects/AppleBite_CICD_Project/app
git commit -m "Update app to latest version"
git push origin main
# → Triggers Jenkins build with updated app
```

**Option 2: Let Jenkins pull latest**
- Jenkins always checks out the submodule at the commit reference in
  `.gitmodules`
- To use latest app code, update the submodule reference as shown above

## Files Modified

### Configuration Files
- ✅ `scripts/setup.sh` - Submodule initialization logic
- ✅ `jenkins/Jenkinsfile` - Repository and path updates
- ✅ `configure-jenkins.sh` - Jenkins setup instructions

### Documentation Files
- ✅ `GITHUB_WEBHOOK_SETUP.md` - New comprehensive guide
- ✅ `SUBMODULE_SETUP.md` - This summary file

### No Changes Needed
- ❌ `docker/Dockerfile` - Already uses correct relative paths
- ❌ `docker-compose*.yml` - Already configured correctly
- ❌ `ansible/` - Already uses relative paths
- ❌ `reset.sh` - Works with submodules automatically

## Testing the Setup

### 1. Test Initial Setup
```bash
cd /c/Users/hardi/HARDIK/Learn/Edureka_DevOps_Arch_Training/AWS_Projects/AppleBite_CICD_Project
./quickstart.sh
# Select: 1) Initial Setup
```

**Expected Output:**
```
Initializing App Submodule
====================
Setting up app as git submodule...
Adding projCert as git submodule...
✓ Application added as submodule
```

### 2. Verify Submodule
```bash
cd /c/Users/hardi/HARDIK/Learn/Edureka_DevOps_Arch_Training
git submodule status
```

**Expected Output:**
```
 <commit-hash> AWS_Projects/AppleBite_CICD_Project/app (heads/master)
```

### 3. Test Jenkins Configuration
```bash
cd /c/Users/hardi/HARDIK/Learn/Edureka_DevOps_Arch_Training/AWS_Projects/AppleBite_CICD_Project
./quickstart.sh
# Select: 3) Configure Jenkins
```

Follow prompts and verify:
- Repository URL: `https://github.com/NitishK1/Edureka_DevOps_Arch_Training.git`
- Branch: `*/main`
- Script Path: `AWS_Projects/AppleBite_CICD_Project/jenkins/Jenkinsfile`
- Submodule checkout enabled

### 4. Test Jenkins Build
1. Go to Jenkins: http://127.0.0.1:8090
2. Click "AppleBite-Pipeline"
3. Click "Build Now"
4. Check console output for:
   - Repository checkout
   - Submodule initialization
   - App code available at correct path
   - Docker build succeeds
   - Deployments succeed

## Troubleshooting

### Submodule Not Found
```bash
# Reinitialize submodules
cd /c/Users/hardi/HARDIK/Learn/Edureka_DevOps_Arch_Training
git submodule update --init --recursive
```

### Jenkins Can't Find Jenkinsfile
- **Check Script Path in Jenkins job config:**
  - Should be: `AWS_Projects/AppleBite_CICD_Project/jenkins/Jenkinsfile`
  - NOT: `jenkins/Jenkinsfile`

### App Directory Empty in Jenkins Build
- **Check Jenkins console output for submodule checkout:**
  - Look for: "Checking out submodules"
  - Verify: "Advanced sub-modules behaviours" is configured in Jenkins job

### Docker Build Fails - File Not Found
- **Check PROJECT_DIR in Jenkinsfile:**
  - Should be: `${WORKSPACE}/AWS_Projects/AppleBite_CICD_Project`
- **Check all `cd ${PROJECT_DIR}` commands in Jenkinsfile**

### Webhook Not Triggering
- See `GITHUB_WEBHOOK_SETUP.md` for detailed troubleshooting

## Benefits of Submodule Approach

✅ **Separation of Concerns**: CI/CD configuration separate from app code ✅
**Version Control**: Track specific app versions in main repo ✅ **Flexibility**:
Can update app independently or together ✅ **Clean Structure**: Main repo owns
the deployment pipeline ✅ **Automatic Deployment**: Push to main repo triggers
deployment ✅ **Easy Rollback**: Just revert submodule reference to previous
commit

## Next Steps

1. ✅ Run `./quickstart.sh` → Option 1 to initialize submodule
2. ✅ Commit and push the `.gitmodules` file:
   ```bash
   cd /c/Users/hardi/HARDIK/Learn/Edureka_DevOps_Arch_Training
   git add .gitmodules AWS_Projects/AppleBite_CICD_Project/app
   git commit -m "Add app as git submodule"
   git push origin main
   ```
3. ✅ Configure Jenkins with webhook or polling
4. ✅ Test the pipeline with a manual build
5. ✅ Make a change and push to trigger automatic build

Happy CI/CD! 🚀
