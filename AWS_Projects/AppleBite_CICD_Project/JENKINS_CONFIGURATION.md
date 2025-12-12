# Jenkins Pipeline Configuration Guide

## Problem
Jenkins might be configured to use the wrong Jenkinsfile. There are two Jenkinsfiles in this project:
- ✅ **`jenkins/Jenkinsfile`** - The correct one (Windows-compatible, full CI/CD pipeline)
- ❌ **`app/Jenkinsfile`** - From the submodule (not Windows-compatible, should not be used)

## Solution: Configure Jenkins to Use the Correct Jenkinsfile

### Step 1: Open Jenkins Pipeline Configuration
1. Go to Jenkins at http://127.0.0.1:8090
2. Click on your pipeline job (e.g., **AppleBite-Pipeline**)
3. Click **Configure** on the left menu

### Step 2: Update Pipeline Script Path
Scroll down to the **Pipeline** section and configure:

**Pipeline Definition:** `Pipeline script from SCM`

**SCM:** `Git`

**Repository URL:** `https://github.com/NitishK1/Edureka_DevOps_Arch_Training.git`

**Credentials:** (Select your GitHub credentials if private repo)

**Branch Specifier:** `*/main`

**Script Path:** `AWS_Projects/AppleBite_CICD_Project/jenkins/Jenkinsfile` ⬅️ **IMPORTANT!**

### Step 3: Configure Submodules (if not already done)
Under **Additional Behaviours** → Click **Add** → Select **Advanced sub-modules behaviours**

Check these options:
- ✅ **Recursively update submodules**
- ✅ **Update tracking submodules to tip of branch**

### Step 4: Save and Build
1. Click **Save** at the bottom
2. Click **Build Now** to trigger a new build
3. Monitor the console output

## Verification
After building, verify in the console output:
```
Obtained AWS_Projects/AppleBite_CICD_Project/jenkins/Jenkinsfile from git https://github.com/NitishK1/Edureka_DevOps_Arch_Training.git
```

## Key Differences Between the Two Jenkinsfiles

### jenkins/Jenkinsfile (✅ Use This)
- ✅ Windows and Linux compatible (uses `isUnix()` checks)
- ✅ Correct PROJECT_DIR: `${WORKSPACE}/AWS_Projects/AppleBite_CICD_Project`
- ✅ Proper path conversion for Windows (backslashes)
- ✅ Docker container commands use Linux shell (correct)

### app/Jenkinsfile (❌ Don't Use This)
- ❌ Only Linux compatible (no Windows support)
- ❌ Wrong PROJECT_DIR: `${WORKSPACE}` (missing AWS_Projects path)
- ❌ Will fail on Windows systems

## Why Docker Commands Use Linux Shell
Even on Windows, Docker Desktop runs Linux containers. Therefore, commands **inside** the container must always use Linux shell (`/bin/bash`), regardless of the host OS:

```groovy
// Correct - Works on Windows and Linux hosts
docker run --rm applebite-app:7 /bin/bash -c "echo Running tests"

// Wrong - Only works if container runs Windows (rare)
docker run --rm applebite-app:7 cmd /c "echo Running tests"
```

## Troubleshooting

### If you see "exec: cmd: not found"
This means Jenkins is using the wrong Jenkinsfile. Follow the steps above to configure the correct path.

### If paths are not found
1. Verify the Script Path is exactly: `AWS_Projects/AppleBite_CICD_Project/jenkins/Jenkinsfile`
2. Check that submodules are being checked out recursively
3. Look at console output to verify the path Jenkins is using

### To verify which Jenkinsfile is being used
Check the Jenkins console output at the beginning of the build:
```
Obtained <path> from git <repo-url>
```

The path should show: `AWS_Projects/AppleBite_CICD_Project/jenkins/Jenkinsfile`

## Additional Notes

### GitHub Webhook (Optional)
If you want Jenkins to trigger automatically on git push, see `GITHUB_WEBHOOK_SETUP.md`

### Submodule Updates
When the app code changes in the submodule, Jenkins will automatically pull the latest changes during checkout if configured correctly.

### Branch Strategy
- **main branch:** Runs full pipeline including Test environment
- **master branch:** Would include Stage and Production deployments (when configured)
