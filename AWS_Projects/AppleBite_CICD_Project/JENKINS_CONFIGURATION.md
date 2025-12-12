# Jenkins Pipeline Configuration Guide

## Problem
Jenkins might be configured to use the wrong Jenkinsfile. There are two
Jenkinsfiles in this project:
- ✅ **`jenkins/Jenkinsfile`** - The correct one (Windows-compatible, full CI/CD
  pipeline)
- ❌ **`app/Jenkinsfile`** - From the submodule (not Windows-compatible, should
  not be used)

## Solution: Configure Jenkins to Use the Correct Jenkinsfile

### Step 1: Open Jenkins Pipeline Configuration
1. Go to Jenkins at http://127.0.0.1:8090
2. Click on your pipeline job (e.g., **AppleBite-Pipeline**)
3. Click **Configure** on the left menu

### Step 2: Update Pipeline Script Path
Scroll down to the **Pipeline** section and configure:

**Pipeline Definition:** `Pipeline script from SCM`

**SCM:** `Git`

**Repository URL:**
`https://github.com/NitishK1/Edureka_DevOps_Arch_Training.git`

**Credentials:** (Select your GitHub credentials if private repo)

**Branch Specifier:** `*/main`

**Script Path:** `AWS_Projects/AppleBite_CICD_Project/jenkins/Jenkinsfile` ⬅️
**IMPORTANT!**

### Step 3: Configure Submodules (if not already done)
Under **Additional Behaviours** → Click **Add** → Select **Advanced sub-modules
behaviours**

Check these options:
- ✅ **Recursively update submodules**
- ✅ **Update tracking submodules to tip of branch**

### Step 4: Enable Automatic Triggers (Choose One Option)

#### Option A: SCM Polling (Recommended for Local Development)
In the **Build Triggers** section, check **Poll SCM** and set the schedule:

```
H/5 * * * *
```

This polls GitHub every 5 minutes for changes. When changes are detected, a build triggers automatically.

**Advantages:**
- ✅ Works with local Jenkins (no public URL needed)
- ✅ Simple to set up
- ✅ No firewall/NAT issues

**Disadvantages:**
- ⚠️ Up to 5-minute delay before build starts
- ⚠️ Makes regular requests to GitHub

#### Option B: GitHub Webhook (Recommended for Production)
In the **Build Triggers** section, check **GitHub hook trigger for GITScm polling**

Then follow the setup in `GITHUB_WEBHOOK_SETUP.md` to configure GitHub webhooks.

**Advantages:**
- ✅ Instant build triggering
- ✅ No polling overhead

**Disadvantages:**
- ⚠️ Requires Jenkins to be publicly accessible
- ⚠️ More complex setup for local development

### Step 5: Save and Build
1. Click **Save** at the bottom
2. Click **Build Now** to trigger a new build
3. Monitor the console output

### Step 6: Test Automatic Triggering
Make a small change, commit, and push to GitHub:
```bash
# Make a change
echo "# Test trigger" >> README.md
git add README.md
git commit -m "Test Jenkins auto-trigger"
git push origin main

# Wait 5 minutes (if using polling) or a few seconds (if using webhook)
# Check Jenkins - a new build should start automatically
```

## Verification
After building, verify in the console output:
```
Obtained AWS_Projects/AppleBite_CICD_Project/jenkins/Jenkinsfile from git https://github.com/NitishK1/Edureka_DevOps_Arch_Training.git
```

## Automatic Build Triggering - Quick Reference

### No Builds Triggering Automatically?
Jenkins won't automatically trigger builds unless you configure a trigger. Check:

1. **Go to:** Jenkins → AppleBite-Pipeline → Configure → Build Triggers
2. **Enable one of:**
   - ✅ **Poll SCM** with schedule `H/5 * * * *` (checks every 5 minutes)
   - ✅ **GitHub hook trigger for GITScm polling** (requires webhook setup)

### How to Verify Polling is Working
After enabling Poll SCM:
1. Go to your Jenkins job dashboard
2. Look for **Polling Log** in the left menu
3. Click it to see when Jenkins last checked for changes
4. You should see entries like:
   ```
   Started on Dec 12, 2025 3:45:00 PM
   Using strategy: Default
   [poll] Last Built Revision: Revision 1125026...
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
Even on Windows, Docker Desktop runs Linux containers. Therefore, commands
**inside** the container must always use Linux shell (`/bin/bash`), regardless
of the host OS:

```groovy
// Correct - Works on Windows and Linux hosts
docker run --rm applebite-app:7 /bin/bash -c "echo Running tests"

// Wrong - Only works if container runs Windows (rare)
docker run --rm applebite-app:7 cmd /c "echo Running tests"
```

## Troubleshooting

### If you see "exec: cmd: not found"
This means Jenkins is using the wrong Jenkinsfile. Follow the steps above to
configure the correct path.

### If paths are not found
1. Verify the Script Path is exactly:
   `AWS_Projects/AppleBite_CICD_Project/jenkins/Jenkinsfile`
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
If you want Jenkins to trigger automatically on git push, see
`GITHUB_WEBHOOK_SETUP.md`

### Submodule Updates
When the app code changes in the submodule, Jenkins will automatically pull the
latest changes during checkout if configured correctly.

### Branch Strategy
- **main branch:** Runs full pipeline including Test environment
- **master branch:** Would include Stage and Production deployments (when
  configured)
