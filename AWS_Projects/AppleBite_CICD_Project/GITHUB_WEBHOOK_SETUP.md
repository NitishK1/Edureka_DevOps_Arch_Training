# GitHub Webhook Setup Guide

This guide explains how to configure GitHub webhooks to automatically trigger
Jenkins builds when you push code to your repository.

## Overview

The project uses:
- **Main Repository**:
  `https://github.com/NitishK1/Edureka_DevOps_Arch_Training.git`
- **App Submodule**: `https://github.com/edureka-devops/projCert.git` (in
  `AWS_Projects/AppleBite_CICD_Project/app`)

Jenkins will trigger on pushes to the **main repository**. The app code is
included as a git submodule.

## Prerequisites

1. Jenkins installed and running (local or Docker)
2. Jenkins accessible from the internet (for GitHub webhooks)
   - For local development, you can use tools like:
     - ngrok: `ngrok http 8090`
     - localtunnel: `lt --port 8090`
     - GitHub polling (alternative to webhooks)

## Method 1: GitHub Webhook (Recommended for Production)

### Step 1: Make Jenkins Publicly Accessible

If Jenkins is running locally, you need to expose it to the internet:

```bash
# Using ngrok (install from https://ngrok.com/)
ngrok http 8090

# Copy the forwarding URL (e.g., https://abc123.ngrok.io)
```

### Step 2: Install GitHub Plugin in Jenkins

1. Go to Jenkins → **Manage Jenkins** → **Manage Plugins**
2. Click **Available** tab
3. Search for "GitHub Plugin"
4. Install and restart Jenkins if needed

### Step 3: Configure Jenkins Job for GitHub

1. Go to your Jenkins pipeline job (**AppleBite-Pipeline**)
2. Click **Configure**
3. Under **Build Triggers**, check:
   - ☑ **GitHub hook trigger for GITScm polling**
4. Under **Pipeline** section:
   - Definition: **Pipeline script from SCM**
   - SCM: **Git**
   - Repository URL:
     `https://github.com/NitishK1/Edureka_DevOps_Arch_Training.git`
   - Branch Specifier: `*/main`
   - Script Path: `AWS_Projects/AppleBite_CICD_Project/jenkins/Jenkinsfile`
5. Click **Save**

### Step 4: Add Webhook in GitHub

1. Go to your GitHub repository:
   `https://github.com/NitishK1/Edureka_DevOps_Arch_Training`
2. Click **Settings** → **Webhooks** → **Add webhook**
3. Configure webhook:
   - **Payload URL**: `http://your-jenkins-url:8090/github-webhook/`
     - Example: `https://abc123.ngrok.io/github-webhook/`
     - Or for local: `http://127.0.0.1:8090/github-webhook/`
   - **Content type**: `application/json`
   - **Secret**: Leave empty (or add for security)
   - **Which events**: Select "Just the push event"
   - ☑ **Active**
4. Click **Add webhook**

### Step 5: Test the Webhook

1. Make a change to your repository
2. Commit and push:
   ```bash
   cd /c/Users/hardi/HARDIK/Learn/Edureka_DevOps_Arch_Training
   git add .
   git commit -m "Test webhook trigger"
   git push origin main
   ```
3. Check Jenkins - build should start automatically!

## Method 2: GitHub Polling (Alternative for Local Development)

If you can't expose Jenkins to the internet, use polling instead:

### Configure Jenkins to Poll GitHub

1. Go to your Jenkins pipeline job (**AppleBite-Pipeline**)
2. Click **Configure**
3. Under **Build Triggers**, check:
   - ☑ **Poll SCM**
   - Schedule: `H/5 * * * *` (polls every 5 minutes)
     - Or `* * * * *` (polls every minute - for testing only)
4. Under **Pipeline** section:
   - Definition: **Pipeline script from SCM**
   - SCM: **Git**
   - Repository URL:
     `https://github.com/NitishK1/Edureka_DevOps_Arch_Training.git`
   - Branch Specifier: `*/main`
   - Script Path: `AWS_Projects/AppleBite_CICD_Project/jenkins/Jenkinsfile`
5. Click **Save**

Jenkins will now check GitHub every 5 minutes for changes and trigger builds
automatically.

## Method 3: Manual Trigger

You can always trigger builds manually:

1. Go to Jenkins dashboard
2. Click on **AppleBite-Pipeline**
3. Click **Build Now**

## Git Submodule Configuration

The app code is stored as a git submodule. The Jenkinsfile is configured to
automatically checkout submodules:

```groovy
checkout([
    $class: 'GitSCM',
    branches: [[name: '*/main']],
    extensions: [
        [$class: 'SubmoduleOption',
         disableSubmodules: false,
         recursiveSubmodules: true,
         trackingSubmodules: false,
         reference: '']
    ],
    userRemoteConfigs: [[
        url: 'https://github.com/NitishK1/Edureka_DevOps_Arch_Training.git'
    ]]
])
```

## Adding Submodule Locally

If you need to manually add the app submodule:

```bash
cd /c/Users/hardi/HARDIK/Learn/Edureka_DevOps_Arch_Training

# Add submodule
git submodule add https://github.com/edureka-devops/projCert.git AWS_Projects/AppleBite_CICD_Project/app

# Initialize and update submodules
git submodule update --init --recursive

# Commit the submodule configuration
git add .gitmodules AWS_Projects/AppleBite_CICD_Project/app
git commit -m "Add app as git submodule"
git push origin main
```

## Updating Submodules

To update the app submodule to latest version:

```bash
cd /c/Users/hardi/HARDIK/Learn/Edureka_DevOps_Arch_Training/AWS_Projects/AppleBite_CICD_Project/app

# Pull latest changes from app repository
git checkout master
git pull origin master

# Go back to main repo and commit submodule update
cd /c/Users/hardi/HARDIK/Learn/Edureka_DevOps_Arch_Training
git add AWS_Projects/AppleBite_CICD_Project/app
git commit -m "Update app submodule to latest version"
git push origin main
```

This will trigger a Jenkins build with the updated app code!

## Troubleshooting

### Webhook Not Triggering

1. **Check webhook delivery in GitHub**:
   - Go to Repository → Settings → Webhooks
   - Click on your webhook
   - Check "Recent Deliveries" for errors

2. **Check Jenkins is accessible**:
   ```bash
   curl http://your-jenkins-url:8090/github-webhook/
   ```

3. **Check Jenkins logs**:
   - Go to Jenkins → Manage Jenkins → System Log

### Polling Not Working

1. **Check Jenkins SCM polling log**:
   - Go to your job → "Git Polling Log" (left sidebar)

2. **Verify repository URL is correct**

3. **Check credentials** if repository is private

### Submodule Not Updating

1. **Verify submodule configuration**:
   ```bash
   git submodule status
   ```

2. **Manually update in Jenkins**:
   - The Jenkinsfile includes `recursiveSubmodules: true`
   - Check Jenkins console output for submodule checkout logs

### Build Paths Not Working

If you see "file not found" errors:

1. **Check PROJECT_DIR in Jenkinsfile**:
   - Should be: `${WORKSPACE}/AWS_Projects/AppleBite_CICD_Project`

2. **Verify APP_DIR**:
   - Should be: `${PROJECT_DIR}/app`

3. **Check Jenkinsfile location in job config**:
   - Script Path: `AWS_Projects/AppleBite_CICD_Project/jenkins/Jenkinsfile`

## Security Considerations

### For Public Exposure (ngrok/localtunnel):

1. **Use webhook secrets** in GitHub webhook configuration
2. **Enable authentication** in Jenkins
3. **Use HTTPS** when possible
4. **Limit IP ranges** if possible
5. **Don't expose ngrok URL permanently** - only for testing

### For Production:

1. **Use a proper domain** with SSL certificate
2. **Configure Jenkins security realm**
3. **Use GitHub webhook secrets**
4. **Enable CSRF protection**
5. **Use credentials plugin** for sensitive data

## Next Steps

After webhook setup:

1. ✅ Push code to trigger automatic builds
2. ✅ Monitor Jenkins dashboard for build status
3. ✅ Access deployed environments:
   - Test: http://127.0.0.1:8080
   - Stage: http://127.0.0.1:8081
   - Production: http://127.0.0.1:8082

Happy CI/CD! 🚀
