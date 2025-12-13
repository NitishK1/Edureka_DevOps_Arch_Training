# Using Sample PHP Application from GitHub

## Overview
The problem statement mentions using a sample PHP application from:
https://github.com/edureka-devops/projCert.git

You have two options:

## Option 1: Use Our Simple PHP App (Current - Recommended)
The project currently includes a simple, custom-built PHP application in the
`app/` folder with:
- Home page (`index.php`)
- About page (`about.php`)
- Contact page (`contact.php`)
- Modern CSS styling

**Pros:**
- Simple and easy to understand
- No external dependencies
- Already configured and ready to use
- Follows the exact requirements

## Option 2: Use Sample PHP App from GitHub

### Step 1: Remove Current App
```bash
cd AWS_Projects/AppleBite_CICD_Project2
rm -rf app
```

### Step 2: Add as Git Submodule
```bash
# Add the sample PHP app as a submodule
git submodule add https://github.com/edureka-devops/projCert.git app

# Initialize and update submodule
git submodule init
git submodule update
```

### Step 3: Update Dockerfile (if needed)
The current Dockerfile should work with the sample app, but verify the file
structure:

```dockerfile
FROM devopsedu/webapp

LABEL maintainer="applebite@example.com"
LABEL description="AppleBite PHP Application"

WORKDIR /var/www/html

# Copy application files
COPY app/ /var/www/html/

# Set permissions
RUN chown -R www-data:www-data /var/www/html && \
    chmod -R 755 /var/www/html

EXPOSE 80

HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
  CMD curl -f http://localhost/ || exit 1

CMD ["apache2-foreground"]
```

### Step 4: Commit Changes
```bash
git add .gitmodules app
git commit -m "Switched to sample PHP app from GitHub"
git push origin main
```

### Step 5: Update Jenkinsfile (if needed)
The Jenkinsfile should automatically pull the submodule when checking out the
code. If not, add this step:

```groovy
stage('Checkout with Submodules') {
    steps {
        checkout([
            $class: 'GitSCM',
            branches: [[name: '*/main']],
            extensions: [[$class: 'SubmoduleOption',
                         disableSubmodules: false,
                         recursiveSubmodules: true]],
            userRemoteConfigs: [[url: 'https://github.com/NitishK1/Edureka_DevOps_Arch_Training.git']]
        ])
    }
}
```

## Updating Submodule

### Pull Latest Changes from Sample App
```bash
cd app
git pull origin master
cd ..
git add app
git commit -m "Updated sample app to latest version"
git push
```

### Remove Submodule (if reverting)
```bash
# Remove submodule entry from .git/config
git submodule deinit -f app

# Remove submodule directory from git
git rm -f app

# Remove physical directory
rm -rf .git/modules/app

# Commit changes
git commit -m "Removed submodule"
git push
```

## Comparison

| Feature | Our Simple App | GitHub Sample App |
|---------|---------------|-------------------|
| Complexity | Very Simple | More Complex |
| Setup Time | Immediate | Requires submodule |
| Customization | Easy | Need to modify submodule |
| Dependencies | None | Git submodule |
| Maintenance | Easy | Need to sync with upstream |
| Size | Small | Larger |

## Recommendation

**For Learning/Testing**: Use Option 1 (our simple app)
- Faster setup
- Easier to understand
- No external dependencies
- Perfect for demonstrating CI/CD concepts

**For Production-like Demo**: Use Option 2 (GitHub sample)
- More realistic application
- Demonstrates submodule management
- Closer to problem statement

## Current Status

✅ **Currently Using**: Option 1 (Simple custom PHP app) 📝 **To Switch**: Follow
Option 2 steps above



**Both options work perfectly with the CI/CD pipeline!**
