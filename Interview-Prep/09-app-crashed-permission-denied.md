# 01. App Crashes with "Permission Denied" in Container but Works Fine on Localhost: Complete Fix Guide
- **When your app runs perfectly on localhost but crashes with "Permission Denied" inside a Docker Container, it's usually because containers
  run with different user permissions and file ownership rules. Think of it like moving from your own house (where you have full access) 
  to a hotel room (where you can only access what's explicitly allowed).**

## Understanding Why Permission Issues Occur
- **Memory Hook: "Container User != Host User, Permissions = Problems"**

- **The fundamental issue is that:
    a. On localhost: Your app runs as your user account with your permissions
    b. In container: The app runs as a different user (often root or a restricted user) with different file access rights**


# 02. Common Permission Denied Scenarios

#### a. File/Directory Access Issues
##### Example: Your app tries to write log files or create directories
```
# Error you might see
docker run myapp
# Permission denied: cannot create directory '/app/logs'
# Permission denied: cannot open file '/app/data/config.txt'
```

##### Root Cause: The container user doesn't have write permissions to these paths.


#### b. Port Binding Issues
##### Example: App tries to bind to privileged ports (< 1024)
```
# This works on localhost (if you're admin)
app.run(host='0.0.0.0', port=80)

# But fails in container with:
# Permission denied: cannot bind to port 80
```

# 03. Step-by-Step Solutions

#### Solution 1: Fix File Permissions in Dockerfile
```
FROM python:3.9
WORKDIR /app

# Copy requirements first (for better caching)
COPY requirements.txt .
RUN pip install -r requirements.txt

# Copy application code
COPY . .

# Fix permissions for directories that need write access
RUN mkdir -p /app/logs /app/data && \
    chmod 755 /app/logs /app/data

# Make scripts executable
RUN chmod +x /app/start.sh

# Create non-root user for security
RUN groupadd -r appuser && useradd -r -g appuser appuser
RUN chown -R appuser:appuser /app
USER appuser

CMD ["python", "app.py"]
```

#### Solution 2: Handle Port Binding Issues
```
FROM node:14
WORKDIR /app
COPY . .
RUN npm install

# Don't use privileged ports (< 1024)
EXPOSE 3000
# Instead of port 80, use port 3000 and map it externally

CMD ["npm", "start"]
```

#### Run with port mapping:
```
# Map container port 3000 to host port 80
docker run -p 80:3000 myapp
```

#### Solution 3: Create Proper User Context
```
FROM python:3.9-slim

# Create app directory with proper ownership
WORKDIR /app

# Create user before copying files
RUN groupadd -r myapp && \
    useradd -r -g myapp -d /app -s /bin/bash myapp && \
    chown -R myapp:myapp /app

# Install dependencies as root
COPY requirements.txt .
RUN pip install -r requirements.txt

# Copy code and set ownership
COPY . .
RUN chown -R myapp:myapp /app

# Switch to non-root user
USER myapp

CMD ["python", "app.py"]
```


# 04. Real-World Examples with Fixes

#### Example 1: Python Flask App with File Logging
#### Problem: App crashes when trying to write logs

```
# app.py - This works on localhost
import logging
logging.basicConfig(
    filename='/app/logs/app.log',  # Permission denied in container!
    level=logging.INFO
)
```

#### Solution:
```
FROM python:3.9
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .

# Create logs directory and set permissions
RUN mkdir -p /app/logs && chmod 777 /app/logs

CMD ["python", "app.py"]
```

#### Better Solution (with proper user):
```
FROM python:3.9
WORKDIR /app

# Create user and directories
RUN groupadd -r flask && useradd -r -g flask flask
RUN mkdir -p /app/logs && chown flask:flask /app/logs

COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
RUN chown -R flask:flask /app

USER flask
CMD ["python", "app.py"]
```

----------------------------------------------

#### Example 2: Node.js App with File Uploads
#### Problem: Cannot save uploaded files
```
// app.js - Works locally, fails in container
const multer = require('multer');
const upload = multer({ dest: '/app/uploads/' });
// Error: EACCES: permission denied, mkdir '/app/uploads'
```

#### Solution:
```
FROM node:14
WORKDIR /app

# Create uploads directory with proper permissions
RUN mkdir -p /app/uploads && chmod 755 /app/uploads

COPY package*.json ./
RUN npm install
COPY . .

# Set ownership for non-root user
RUN groupadd -r nodeapp && useradd -r -g nodeapp nodeapp
RUN chown -R nodeapp:nodeapp /app

USER nodeapp
EXPOSE 3000
CMD ["npm", "start"]
```

---------------------------------------------------------------------

#### Example 3: Shell Script Execution
#### Problem: Startup script has permission denied
```
# start.sh - Works with ./start.sh locally
#!/bin/bash
echo "Starting application..."
python app.py
```

```
FROM python:3.9
WORKDIR /app
COPY . .
RUN pip install -r requirements.txt

# This fails: Permission denied
CMD ["./start.sh"]
```

#### Solution:
```
FROM python:3.9
WORKDIR /app
COPY . .
RUN pip install -r requirements.txt

# Make script executable
RUN chmod +x start.sh

CMD ["./start.sh"]
```

# 05. Debugging Permission Issues

#### a. Check File Permissions Inside Container
```
# Run container interactively to debug
docker run -it --entrypoint=/bin/bash myapp

# Inside container, check permissions
ls -la /app/
ls -la /app/logs/
whoami
id
```

#### b. Test with Root User Temporarily
```
# Run as root to isolate permission issues
docker run -it --user root myapp

# If it works as root, you know it's a permission issue
```

#### c. Compare Host vs Container Permissions
```
# On host
ls -la ./logs/
stat ./logs/

# In container
docker exec -it container_name ls -la /app/logs/
docker exec -it container_name stat /app/logs/
```

# 06. Advanced Solutions

#### a. Using Docker Volumes with Proper Permissions
```
# Create volume with specific permissions
docker volume create app_logs

# Run container with volume
docker run -v app_logs:/app/logs myapp

# Or use bind mount with user mapping
docker run -v ./logs:/app/logs --user $(id -u):$(id -g) myapp
```

#### b. Multi-Stage Build with Permission Setup
```
# Build stage
FROM python:3.9 AS builder
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt

# Runtime stage
FROM python:3.9-slim AS runtime
WORKDIR /app

# Create user and directories
RUN groupadd -r appuser && \
    useradd -r -g appuser -d /app appuser && \
    mkdir -p /app/logs /app/data && \
    chown -R appuser:appuser /app

# Copy from build stage
COPY --from=builder /usr/local/lib/python3.9/site-packages /usr/local/lib/python3.9/site-packages
COPY --chown=appuser:appuser . .

USER appuser
CMD ["python", "app.py"]
```

# 07. Memory Aid: The "POWER" Method
- a. P - Permissions check file/directory access rights
- b. O - Ownership ensure proper user:group ownership
- c. W - Write access verify directories are writable
- d. E - Execute permissions for scripts and binaries
- e. R - Root vs non-root user context


# 08. Common Fixes Checklist
```
# 1. Make directories writable
RUN mkdir -p /app/logs && chmod 755 /app/logs

# 2. Make scripts executable  
RUN chmod +x /app/start.sh

# 3. Create proper user
RUN groupadd -r myapp && useradd -r -g myapp myapp

# 4. Set ownership
RUN chown -R myapp:myapp /app

# 5. Use non-root user
USER myapp

# 6. Use non-privileged ports
EXPOSE 8080  # Instead of 80
```

# 09. Security Best Practices
- a. Never run containers as root in production unless absolutely necessary
- b. Create specific users for each application
- c. Use minimal permissions - only grant what's needed
- d. Audit file permissions regularly in production images
- e. Use read-only filesystems where possible with writable volumes for data



