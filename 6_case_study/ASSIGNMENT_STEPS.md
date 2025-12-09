# DevOps Case Study - Docker Assignment

## Manual Commands to Complete Assignment

### Prerequisites
```bash
# Ensure Podman is running
podman machine start
```



### Step 1: Build the Docker Image

```bash
cd /Users/C177341/Local_Mac/Edureka_DevOps_Arch_Training/6_case_study

podman build --tls-verify=false -t company-website:latest .
```

**Verify:**
```bash
podman images | grep company-website
```



### Step 2: Test Locally (Optional)

```bash
podman run -d --name test-website -p 8080:80 \
  -v "$(pwd)/dockerContent/Case-study app:/var/www/html" \
  company-website:latest

# Test
curl http://localhost:8080

# Cleanup
podman stop test-website
podman rm test-website
```



### Step 3: Push to Docker Hub

```bash
# Login to Docker Hub
podman login docker.io
# Username: hardikgangwar
# Password: [enter your password]

# Tag the image
podman tag company-website:latest docker.io/hardikgangwar/company-website:latest
podman tag company-website:latest docker.io/hardikgangwar/company-website:v1.0

# Push to Docker Hub
podman push --tls-verify=false docker.io/hardikgangwar/company-website:latest
podman push --tls-verify=false docker.io/hardikgangwar/company-website:v1.0
```

**Verify:**
- Visit https://hub.docker.com/r/hardikgangwar/company-website



### Step 4: Initialize Docker Swarm Cluster

```bash
# Initialize Docker Swarm
docker swarm init

# Verify Swarm is active
docker info | grep Swarm
```

**Output should show:** `Swarm: active`



### Step 5: Deploy Service to Swarm with Volumes

```bash
# Create a named volume for persistence
docker volume create company-website-data

# Deploy service with 3 replicas
docker service create \
  --name company-website \
  --replicas 3 \
  --publish 80:80 \
  --mount type=volume,source=company-website-data,target=/var/www/html \
  docker.io/hardikgangwar/company-website:latest

# Alternative: Deploy using docker-compose-swarm.yml
docker stack deploy -c docker-compose-swarm.yml company-website
```

**Verify Service:**
```bash
# Check service status
docker service ls

# Check service replicas
docker service ps company-website

# View service logs
docker service logs company-website
```



### Step 6: Access the Website

Open in browser:
- **Local:** http://localhost
- **Docker Hub:** docker pull docker.io/hardikgangwar/company-website:latest



## Assignment Checklist

- [x] Dockerfile created with proper configuration
- [x] Volume configured for `/var/www/html`
- [x] Image pushed to Docker Hub
- [x] Docker Swarm cluster initialized
- [x] Service deployed with 3 replicas on port 80
- [x] Volume mount configured for persistence
- [x] Port 80 exposed (container) / 8080 (host)



## Quick Commands Reference

```bash
# View running services
docker service ls

# View service details
docker service ps company-website

# Scale service
docker service scale company-website=5

# Update service
docker service update --image docker.io/hardikgangwar/company-website:v1.0 company-website

# Remove service
docker service rm company-website

# Leave swarm (cleanup)
docker swarm leave --force

# View running containers
podman ps

# View container logs
docker service logs company-website

# View images
podman images
```
