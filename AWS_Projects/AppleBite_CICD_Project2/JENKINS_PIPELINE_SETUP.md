# Jenkins Pipeline Creation Guide

Since programmatic creation via XML is encountering issues, please create the
pipeline manually through the Jenkins UI. It only takes 2 minutes:

## Step-by-Step Instructions:

### 1. Access Jenkins
- Open: `http://192.168.128.187:8080`
- Login with:
  - Username: `hardikgangwar`
  - Password: `Nitish@241091`

### 2. Create New Pipeline Job
1. Click **"New Item"** (top left)
2. Enter name: `AppleBite-CICD-Pipeline`
3. Select **"Pipeline"** (scroll down to find it)
4. Click **"OK"**

### 3. Configure the Pipeline

#### General Section:
- Description: `AppleBite CI/CD Pipeline - Auto-triggered every 5 minutes`

#### Build Triggers Section:
- ✅ Check **"Poll SCM"**
- Schedule: `H/5 * * * *` (This polls GitHub every 5 minutes)

#### Pipeline Section:
- Definition: Select **"Pipeline script from SCM"**
- SCM: Select **"Git"**

**Repositories:**
- Repository URL: `https://github.com/NitishK1/Edureka_DevOps_Arch_Training.git`
- Credentials: `- none -` (public repo)

**Branches to build:**
- Branch Specifier: `*/main`

**Script Path:**
- `AWS_Projects/AppleBite_CICD_Project2/Jenkinsfile-with-prod`

#### Additional Behaviors (Optional):
- You can check **"Lightweight checkout"** for faster checkouts

### 4. Save
- Click **"Save"** at the bottom

### 5. Test the Pipeline
- Click **"Build Now"** to trigger the first build manually
- The pipeline will automatically trigger every 5 minutes when it detects
  changes in your Git repo



## Verification

After saving, you should see:
- ✅ Pipeline job listed on Jenkins dashboard
- ✅ SCM Polling log showing poll attempts
- ✅ Build history when you click "Build Now"

## Screenshots Location
Navigate to: Dashboard → AppleBite-CICD-Pipeline

## Polling Schedule Explained
- `H/5 * * * *` means:
  - Check GitHub every 5 minutes
  - `H` adds a hash to distribute load
  - Format: `H/5` (every 5 min) `*` (any hour) `*` (any day) `*` (any month) `*`
    (any day of week)



## Alternative: Quick Create Script (if you want to try again)

If you want to attempt automation again, the issue is with XML formatting in
heredocs. The manual UI method above is faster and more reliable.
