# 01. Docker host is running out of disk space. How do you clean up?
- **When your Docker host is running out of disk space, it's usually because Docker has accumulated unused images, stopped containers,
  dangling volumes, and build cache over time. Think of it like having a garage that's filled with old boxes you never threw away
  eventually, you can't park your car anymore.**


## Understanding What Takes Up Space in Docker
- **Memory Hook: "Images, Containers, Volumes, Builds - Clean Each For Space Rebuilds"**

- **Docker consumes disk space through:
  a. Docker images - Base OS images, application images (can be GBs each)
  b. Stopped containers - Retain their writable layers
  c. Unused volumes - Persistent data that's no longer needed
  d. Build cache - Intermediate layers from building images
  e. Logs - Container logs that grow over time**


## Quick Assessment: Check What's Using Space
- **Before cleaning, always assess what's consuming disk space:**

```
# See Docker's overall disk usage
docker system df

# More detailed breakdown
docker system df -v

# Check host disk space
df -h /var/lib/docker
```

- **Example output:**
```
TYPE            TOTAL     ACTIVE    SIZE      RECLAIMABLE
Images          15        3         8.5GB     6.2GB (73%)
Containers      25        2         1.2GB     1.1GB (92%)
Local Volumes   10        1         2.8GB     2.3GB (82%)
Build Cache     45        0         3.4GB     3.4GB (100%)
```


# 02. Step-by-Step Cleanup Process

#### a. Nuclear Option: Clean Everything Unused
```
# Remove ALL unused resources (most aggressive)
docker system prune -a -f --volumes

# Explanation of flags:
# -a : Remove all unused images, not just dangling ones
# -f : Force (skip confirmation prompt)
# --volumes : Also remove unused volumes
```

- **Warning: This removes everything not currently in use. Only use this if you're sure you can rebuild what you need.**


#### b. Selective Cleanup (Safer Approach)
- **Clean stopped containers:**
```
# Remove all stopped containers
docker container prune -f

# Or remove specific containers
docker rm $(docker ps -a -q --filter status=exited)
```

- **Clean unused images:**
```
# Remove dangling images (untagged/unreferenced)
docker image prune -f

# Remove ALL unused images (more aggressive)
docker image prune -a -f
```

- **Clean unused volumes:**
```
# Remove unused volumes
docker volume prune -f

# List volumes first to see what will be removed  
docker volume ls
```

- **Clean build cache:**
```
# Remove build cache (can free significant space)
docker builder prune -a -f

# Check build cache usage first
docker builder du
```

# 03. Real-World Scenarios and Solutions

#### Scenario 1: Development Machine Cleanup
#### Problem: Accumulation of old development images and containers

```
# Safe development cleanup routine
echo "=== Docker Cleanup Started ==="

# Remove stopped containers
docker container prune -f
echo "Containers cleaned"

# Remove dangling images only (keeps tagged images)
docker image prune -f
echo "Dangling images cleaned"

# Remove unused volumes (be careful with data!)
docker volume prune -f
echo "Unused volumes cleaned"

# Clean build cache
docker builder prune -f
echo "Build cache cleaned"

echo "=== Cleanup Complete ==="
docker system df
```

----------------------------------------------------

#### Scenario 2: Production Server Emergency Cleanup
#### Problem: Critical disk space shortage in production

```
# Emergency production cleanup (with backups)
#!/bin/bash

# Check available space
echo "Disk space before cleanup:"
df -h /var/lib/docker

# Backup important volumes first
docker run -v important_volume:/data -v /backup:/backup \
  ubuntu tar czf /backup/important_volume_backup.tar.gz /data

# Clean unused resources only (preserve running containers)
docker system prune -f
docker builder prune -a -f

echo "Disk space after cleanup:"
df -h /var/lib/docker
```

---------------------------------------------------------------------

#### Scenario 3: CI/CD Build Server Cleanup
#### Problem: Build cache and intermediate images consuming space

```
# CI/CD cleanup script
#!/bin/bash

# Clean build cache aggressively (CI can rebuild)
docker builder prune -a -f

# Remove images older than 24 hours
docker image prune -a --filter "until=24h" -f

# Remove stopped containers from builds
docker container prune -f

# Keep only last 5 tagged images per repository
docker images --format "{{.Repository}}:{{.Tag}}" | \
  sort | uniq -c | awk '$1 > 5 {print $2}' | \
  head -n -5 | xargs -r docker rmi
```


# 04. Advanced Cleanup Strategies

#### a. Time-Based Cleanup
```
# Remove resources older than specific time
docker container prune --filter "until=72h" -f
docker image prune -a --filter "until=48h" -f
docker volume prune --filter "until=168h" -f  # 1 week
```

#### b. Label-Based Cleanup
```
# Remove images with specific labels
docker image prune --filter "label=environment=testing" -f

# Remove containers with specific labels
docker container prune --filter "label=temporary=true" -f
```

#### c.Size-Based Cleanup
```
# Find and remove largest images
docker images --format "table {{.Size}}\t{{.Repository}}:{{.Tag}}" | \
  sort -hr | head -10

# Remove specific large images
docker rmi $(docker images --filter "dangling=true" -q --no-trunc)
```

#### d. Automated Cleanup Script
```
#!/bin/bash
# docker-cleanup.sh - Weekly maintenance script

set -e

echo "🧹 Starting Docker cleanup routine..."

# Get initial disk usage
BEFORE=$(df -h /var/lib/docker | awk 'NR==2{print $3}')

# Cleanup sequence
echo "📦 Cleaning containers..."
docker container prune -f

echo "🖼️  Cleaning images..."
docker image prune -f

echo "💽 Cleaning volumes..."
docker volume prune -f

echo "🏗️  Cleaning build cache..."
docker builder prune -f

# Get final disk usage
AFTER=$(df -h /var/lib/docker | awk 'NR==2{print $3}')

echo "✅ Cleanup complete!"
echo "Disk usage: $BEFORE → $AFTER"

# Show current usage
docker system df
```

# 05. Platform-Specific Considerations

#### a. Docker Desktop (Windows/Mac)
```
# For Docker Desktop users
# Use the GUI: Settings > Resources > Advanced > Disk image location
# Or command line:
docker run --privileged --pid=host docker/desktop-reclaim-space
```

#### b. WSL2 (Windows)
```
# Compact WSL2 virtual disk after cleanup
wsl --shutdown
# Then in PowerShell as Admin:
Optimize-VHD -Path C:\Users\%USERNAME%\AppData\Local\Docker\wsl\data\ext4.vhdx -Mode Full
```

#### c. Linux Production Servers
```
# Check Docker root directory usage
du -sh /var/lib/docker/
du -sh /var/lib/docker/overlay2/  # Main storage driver
du -sh /var/lib/docker/containers/  # Container logs
```

# 06. Prevention Best Practices

#### a. Implement Log Rotation
```
// /etc/docker/daemon.json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  }
}
```

#### b. Use Multi-Stage Builds
```
# Use multi-stage builds to reduce final image size
FROM node:16 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

FROM node:16-alpine AS runtime
WORKDIR /app
COPY --from=builder /app/node_modules ./node_modules
COPY . .
CMD ["npm", "start"]
```

#### c. Regular Maintenance Cron Job
```
# Add to crontab (weekly cleanup)
0 2 * * 0 /usr/local/bin/docker-cleanup.sh >> /var/log/docker-cleanup.log 2>&1
```

# 07. Memory Aid: The "SPACE" Method
- a. S - System prune for comprehensive cleanup
- b. P - Prune individual components (containers, images, volumes)
- c. A - Assess disk usage before and after
- d. C - Cache cleanup (build cache can be huge)
- e. E - Emergency procedures for critical situations


# 08. Emergency Disk Space Commands
```
# When disk is critically full (95%+)
# Quick wins - most space with least risk:

# 1. Build cache (usually safest and biggest)
docker builder prune -a -f

# 2. Stopped containers
docker container prune -f  

# 3. Dangling images  
docker image prune -f

# 4. Check what you gained
docker system df
```

# 09. Monitoring and Alerts
```
# Create a monitoring script
#!/bin/bash
USAGE=$(df /var/lib/docker | awk 'NR==2{print $5}' | sed 's/%//')
if [ $USAGE -gt 80 ]; then
    echo "Docker disk usage is ${USAGE}% - cleanup needed!" | \
    mail -s "Docker Disk Alert" admin@company.com
    # Auto-cleanup if above 90%
    if [ $USAGE -gt 90 ]; then
        docker system prune -f
    fi
fi
```