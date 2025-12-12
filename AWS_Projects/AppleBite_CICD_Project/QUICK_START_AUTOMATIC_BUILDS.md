# Quick Start: Enable Automatic Jenkins Builds

## Problem
Jenkins doesn't automatically build when you push to GitHub because **no build
trigger is configured**.

## Solution: Enable SCM Polling (Easiest for Local Jenkins)

### 5-Step Setup (Takes 2 minutes)

#### 1. Open Jenkins Configuration
- Go to: http://127.0.0.1:8090
- Click: **AppleBite-Pipeline** job
- Click: **Configure** (left menu)

#### 2. Scroll to "Build Triggers" Section
Look for the section titled **"Build Triggers"**

#### 3. Enable Poll SCM
- ✅ Check the box: **"Poll SCM"**
- A text box will appear labeled **"Schedule"**

#### 4. Set Polling Schedule
Enter this in the Schedule box:
```
H/5 * * * *
```

**What this means:** Jenkins checks GitHub every 5 minutes for new commits.

**Schedule Syntax Explained:**
- `H/5 * * * *` = Every 5 minutes (recommended)
- `H/2 * * * *` = Every 2 minutes (more frequent)
- `H/10 * * * *` = Every 10 minutes (less frequent)
- `H * * * *` = Every hour

The `H` (hash) spreads load evenly if you have multiple jobs.

#### 5. Save Configuration
- Scroll to bottom
- Click **Save**

## ✅ Done! Test It

### Make a Test Commit
```bash
cd /c/Users/hardi/HARDIK/Learn/Edureka_DevOps_Arch_Training
echo "# Test auto-trigger" >> README.md
git add README.md
git commit -m "Test: Jenkins auto-trigger"
git push origin main
```

### Wait & Verify
1. **Wait 5 minutes** (or whatever schedule you set)
2. Refresh Jenkins dashboard
3. You should see a new build automatically starting! 🎉

### Check Polling Log
To verify polling is working:
- Jenkins → AppleBite-Pipeline
- Click **Polling Log** (left menu)
- You'll see logs like:
  ```
  Started on Dec 12, 2025 3:45:00 PM
  Polling for changes in https://github.com/NitishK1/Edureka_DevOps_Arch_Training.git
  Changes found
  Done. Took 2.3 sec
  ```

## Alternative: GitHub Webhooks (Instant Triggering)

If you want **instant** builds (no 5-minute wait), you need webhooks.

### Requirements for Webhooks:
- Jenkins must be publicly accessible (not just localhost)
- Use ngrok, localtunnel, or cloud-hosted Jenkins
- More setup required

### Quick Webhook Setup:
See `GITHUB_WEBHOOK_SETUP.md` for detailed instructions.

## Comparison: Polling vs Webhooks

| Feature | SCM Polling | GitHub Webhooks |
|---------|------------|-----------------|
| **Setup Complexity** | ⭐ Very Easy | ⭐⭐⭐ Complex |
| **Speed** | 5-minute delay | Instant |
| **Local Jenkins** | ✅ Works | ❌ Needs public URL |
| **GitHub API Calls** | Every 5 min | Only on push |
| **Best For** | Development | Production |

## Troubleshooting

### "Nothing happens after 5 minutes"
1. Check **Polling Log** - is it running?
2. Verify repository URL in Jenkins config
3. Make sure you pushed to the correct branch (`main`)

### "Polling Log shows errors"
- Check credentials if repository is private
- Verify network/internet connectivity
- Check GitHub is accessible from Jenkins

### "I want faster builds"
- Change schedule to `H/2 * * * *` (every 2 minutes)
- Or set up webhooks for instant triggering

### "Builds keep triggering even without changes"
- This shouldn't happen with SCM polling
- Check if someone else is pushing to the repo
- Review recent commits on GitHub

## Best Practices

### Development
- ✅ Use **Poll SCM** with `H/5 * * * *`
- Simple, reliable, no firewall issues

### Production
- ✅ Use **GitHub Webhooks**
- Instant feedback
- No unnecessary API calls

### Branch Strategy
- Poll only the branches you care about
- In Jenkins config: Branch Specifier = `*/main`
- Production builds: `*/master` or `*/production`

## Next Steps

Once automatic builds are working:
1. Review `JENKINS_CONFIGURATION.md` for full pipeline setup
2. Check `GITHUB_WEBHOOK_SETUP.md` if you want webhooks
3. Review `START_HERE.md` for complete project overview

## Summary

**To enable auto-builds:**
1. Jenkins → AppleBite-Pipeline → Configure
2. Build Triggers → ✅ Poll SCM
3. Schedule → `H/5 * * * *`
4. Save

**That's it!** Every push to GitHub will trigger a build within 5 minutes.
