# Reset Script Usage Guide

## Overview

The `reset.sh` script provides a safe and comprehensive way to completely reset
the AppleBite CI/CD project to its initial state.

## What It Does

### Removes:
- ✅ All Docker containers (test, stage, production)
- ✅ All Docker images (applebite-app:*)
- ✅ All Docker volumes (including databases)
- ✅ All Docker networks (applebite-*)
- ✅ Jenkins container and data
- ✅ Application files (app/ directory) - with confirmation
- ✅ Log files
- ✅ Environment files (.env)

### Keeps:
- ✅ Project structure
- ✅ Documentation files
- ✅ Scripts
- ✅ Docker configurations
- ✅ Ansible configurations
- ✅ Jenkinsfile

## Usage

### Method 1: Direct Execution
```bash
cd c:/Users/hardi/HARDIK/Learn/Edureka_DevOps_Arch_Training/AWS_Projects/AppleBite_CICD_Project
chmod +x reset.sh
./reset.sh
```

### Method 2: Via Quick Start Menu
```bash
./quickstart.sh
# Select option 18: Complete Reset
```

## Safety Features

### Multiple Confirmations
The script requires TWO confirmations:
1. Type `yes` to confirm you want to reset
2. Type `DELETE` (case-sensitive) to proceed

### Selective Removal
- Asks before removing app/ directory
- Asks before running Docker system prune
- Displays what will be removed before proceeding

### Verification
After cleanup, the script:
- Shows remaining resources
- Verifies all items were removed
- Provides a summary of actions taken

## When to Use

### Use reset.sh when you want to:
- Start completely fresh
- Clean up after testing
- Remove all deployed applications
- Free up disk space
- Troubleshoot persistent issues
- Prepare for a clean demo

### Don't use reset.sh if:
- You have important data in containers
- You want to keep Docker images (use cleanup.sh instead)
- You're not sure what will be deleted

## After Reset

Once reset is complete, start fresh with:

```bash
# Option 1: Quick start
./quickstart.sh

# Option 2: Manual setup
./scripts/setup.sh
./scripts/deploy.sh test

# Option 3: Verify first
./verify.sh
```

## Example Output

```
 ██████╗ ███████╗███████╗███████╗████████╗
 ██╔══██╗██╔════╝██╔════╝██╔════╝╚══██╔══╝
 ██████╔╝█████╗  ███████╗█████╗     ██║
 ██╔══██╗██╔══╝  ╚════██║██╔══╝     ██║
 ██║  ██║███████╗███████║███████╗   ██║
 ╚═╝  ╚═╝╚══════╝╚══════╝╚══════╝   ╚═╝

 AppleBite CI/CD Project - Complete Reset

WARNING: This script will DELETE:

  • All Docker containers (test, stage, prod)
  • All Docker images (applebite-app)
  • All Docker volumes (including databases)
  • All Docker networks
  • Jenkins container and data
  • Application files (app/ directory)
  • Log files

THIS ACTION CANNOT BE UNDONE!

Are you ABSOLUTELY sure you want to reset everything? Type 'yes' to continue:
```

## Comparison with cleanup.sh

| Feature | cleanup.sh | reset.sh |
|---------|-----------|----------|
| Stop containers | ✅ | ✅ |
| Remove images | Optional | ✅ |
| Remove volumes | Optional | ✅ |
| Remove networks | ❌ | ✅ |
| Remove Jenkins | ❌ | ✅ |
| Remove app/ | ❌ | ✅ (optional) |
| Remove logs | Optional | ✅ |
| Confirmations | 1 | 2 |
| Verification | Basic | Comprehensive |

## Tips

1. **Before Reset**: Take screenshots or backups if needed
2. **After Reset**: Run `./verify.sh` to check prerequisites
3. **Partial Cleanup**: Use `./scripts/cleanup.sh` for less destructive cleanup
4. **Docker Space**: Reset is useful for reclaiming disk space
5. **Fresh Start**: Great for demos or new installations

## Troubleshooting

### Script won't run
```bash
chmod +x reset.sh
```

### Some resources remain
```bash
# Manually remove remaining containers
docker ps -a | grep applebite
docker rm -f <container-id>

# Remove remaining images
docker images | grep applebite
docker rmi -f <image-id>
```

### Permission denied
```bash
# Run with sudo (Linux/Mac)
sudo ./reset.sh

# Or fix permissions
chmod +x reset.sh
```

## Important Notes

⚠️ **Data Loss**: All database data will be permanently deleted ⚠️ **No Undo**:
This action cannot be reversed ⚠️ **Docker Space**: Frees up significant disk
space ⚠️ **Fresh Install**: Like a new installation

## Quick Reference

```bash
# Complete reset
./reset.sh

# Cleanup without deleting app
./scripts/cleanup.sh all

# Stop containers only
./scripts/cleanup.sh containers

# Verify after reset
./verify.sh
```



**Use with caution! This script permanently deletes all created resources.**

**Version**: 1.0 **Last Updated**: December 11, 2025
