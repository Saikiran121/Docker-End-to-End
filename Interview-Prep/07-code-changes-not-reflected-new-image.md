# 01. Code Changes Not Reflected After Docker Image Rebuild: Complete Troubleshooting Guide
- **When you make code changes, rebuild your Docker Image, but the changes dont appear, it's usually because Docker is using a cached 
  image or you are running an old container. Think of it like editing a document but still looking at an old printout, the original
  file changed, but you are not seeing the latest version.**


## Understanding Why Changes Don't Appear
- **Memory Hook: "Build != Run, Cache = Confusion" - Building a new image doesn't automatically update running containers, and Docker's
  cache can mask your changes.**
<br>

- **The most common causes are:
  a. Docker Build Cache reusing old layers.
  b. Running old Containers instead of new ones.
  c. Wrong Image Tags or multiple versions.
  d. Cached base images that need updating.**


## Step-by-Step Troubleshooting Process

#### a. Force a Clean Build (No Cache)
```
# Problem: Docker reuses cached layers
docker build -t myapp:latest .

# Solution: Force rebuild without cache
docker build --no-cache -t myapp:latest .

# Or clear build cache entirely
docker builder prune -f
docker build -t myapp:latest .
```

##### Real Example: You change a Python file but Docker cached the COPY . . step:
```
FROM python:3.9
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .              # This layer might be cached!
CMD ["python", "app.py"]
```


#### b. Stop and Remove Old Containers
```
# Check what's currently running
docker ps

# Stop and remove the old container
docker stop myapp-container
docker rm myapp-container

# Run with the new image
docker run -d --name myapp-container myapp:latest
```

##### Better approach - One command:
```
# Force recreate container with new image
docker run -d --name myapp-container --rm myapp:latest
```

#### c. Verify You're Using the Right Image
```
# Check image creation time
docker images myapp

# Verify image ID matches what you just built
docker inspect myapp:latest | grep Id

# Check which image the container is using
docker inspect myapp-container | grep Image
```


# 02. Common Scenarios and Solutions

#### Scenario 1: Node.js Application Changes Not Visible
#### Problem: Modified JavaScript files but app behavior unchanged
```
# Wrong approach - may use cached layers
docker build -t nodeapp .
docker run -d nodeapp
```

#### Solution
```
# Clean build and fresh container
docker build --no-cache -t nodeapp:v2 .
docker stop $(docker ps -q --filter ancestor=nodeapp)
docker run -d --name nodeapp-new nodeapp:v2
```

#### Scenario 2: Python Code Changes Ignored
#### Problem: Updated Python code but container runs old version
```
# Problematic Dockerfile - everything cached together
FROM python:3.9
WORKDIR /app
COPY . .                    # Cache issue here
RUN pip install -r requirements.txt
CMD ["python", "app.py"]
```

#### Solution Optimized Dockerfile:
```
# Better Dockerfile - separates dependencies from code
FROM python:3.9
WORKDIR /app
COPY requirements.txt .     # Cache dependencies separately
RUN pip install -r requirements.txt
COPY . .                    # Code changes invalidate only this layer
CMD ["python", "app.py"]
```

#### Scenario 3: Configuration File Changes Not Applied
#### Problem: Updated config file but application uses old settings
```
# Verify the file actually changed in the image
docker run --rm myapp:latest cat /app/config.json

# If it's still old, force rebuild
docker build --no-cache -t myapp:latest .
```


#### Scenario 4: Development vs Production Image Confusion
#### Problem: Building wrong target in multi-stage Dockerfile
```
# Multi-stage Dockerfile
FROM node:14 AS development
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
CMD ["npm", "run", "dev"]

FROM node:14 AS production
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
CMD ["npm", "start"]
```

```
# Make sure you're building the right target
docker build --target development -t myapp:dev .
docker build --target production -t myapp:prod .
```

# 03. Advanced Troubleshooting Techniques

#### Check Docker Layer History
```
# See which layers changed
docker history myapp:latest

# Compare with previous version
docker history myapp:previous
```

#### Verify File Timestamps Inside Container
```
# Check when files were last modified in the container
docker run --rm myapp:latest ls -la /app/

# Compare with your local files
ls -la ./
```

#### Use .dockerignore to Avoid Cache Issues
```
# Create .dockerignore to exclude files that don't affect the build
echo "node_modules" > .dockerignore
echo ".git" >> .dockerignore
echo "*.log" >> .dockerignore
```


# 04. Development Workflow Best Practices

#### a. Use Unique Tags for Each Build
```
# Instead of always using 'latest'
docker build -t myapp:$(git rev-parse --short HEAD) .
docker build -t myapp:v1.2.3 .

# Or use timestamps
docker build -t myapp:$(date +%Y%m%d-%H%M%S) .
```

#### b. Docker Compose Development Pattern
```
version: '3.8'
services:
  app:
    build: 
      context: .
      dockerfile: Dockerfile
    volumes:
      - ./src:/app/src  # Mount source for live updates
    environment:
      - NODE_ENV=development
```

#### c. Quick Rebuild Script
```
#!/bin/bash
# rebuild.sh - Complete refresh script
docker stop myapp-container 2>/dev/null
docker rm myapp-container 2>/dev/null
docker build --no-cache -t myapp:latest .
docker run -d --name myapp-container -p 8080:8080 myapp:latest
echo "App rebuilt and running on port 8080"
```


# 05. Memory Aid: The "FRESH" Method
- F - Force no-cache build (--no-cache)
- R - Remove old containers
- E - Examine image timestamps and IDs
- S - Stop running old containers
- H - History check what layers changed


# 06. Debugging Commands Checklist
```
# 1. List all images and their ages
docker images --format "table {{.Repository}}\t{{.Tag}}\t{{.CreatedAt}}"

# 2. Check running containers
docker ps -a

# 3. Force clean rebuild
docker build --no-cache -t myapp:fresh .

# 4. Remove all containers using old image
docker ps -a --filter ancestor=myapp --format "{{.ID}}" | xargs docker rm -f

# 5. Run with new image
docker run -d --name myapp-new myapp:fresh

# 6. Verify changes
docker exec myapp-new cat /app/your-changed-file
```

# 07. Production-Ready Solution
```
# Complete refresh workflow
#!/bin/bash
APP_NAME="myapp"
NEW_TAG="${APP_NAME}:$(date +%Y%m%d-%H%M%S)"

echo "Building fresh image: $NEW_TAG"
docker build --no-cache -t $NEW_TAG .

echo "Stopping old containers"
docker ps --filter ancestor=$APP_NAME --format "{{.ID}}" | xargs -r docker stop

echo "Starting new container"
docker run -d --name ${APP_NAME}-$(date +%s) $NEW_TAG

echo "Cleanup old images (keep last 3)"
docker images $APP_NAME --format "{{.ID}}" | tail -n +4 | xargs -r docker rmi
```

