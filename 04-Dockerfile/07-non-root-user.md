# 01. Understanding Non-Root Users in Docker: Why, What, and How MAIN TAKEWAY:
      - A non-root user in Docker is any user account other than the default root (UID 0) that your container processes run under. 
      - Using non-root users is a critical security practice that limits damage if your container gets compromised, follows the principle 
        of least privilege, and helps meet compliance requirements.


# 02. What Is a Non-Root User?
      - By default, Docker containers run as the root user (superuser with UID 0), which has unlimited privileges inside the container.
      - A non-root user is any other user account with limited permissions typically created specifically for running your application.


# 03. Real-World Analogy:
      - Think of root as the master key to a building that opens every door, safe, and security system. 
      - A non-root user is like giving each employee only the keys they need for their specific job the janitor gets cleaning supply
        closets, but not the CEO's office or the server room.


# 04. Why You Need Non-Root Users
      - Security Risks of Running as Root

      a. Container Escape
         - If attackers break out of the container, they have root privileges on the host	
         - Attacker gains full control of your server
      
      b. File System Access	
         - Root can read/write any file inside the container	
         - Sensitive config files, secrets, or data exposed
      
      c. Process Control	
         - Root can kill any process, modify system settings	
         - Application or other containers disrupted
      
      d. Privilege Escalation	
         - Exploits can leverage root permissions to gain more access	
         - Lateral movement through your infrastructure


# 05. Benefits of Non-Root Users
      a. Principle of Least Privilege: Only grant minimum permissions needed
      b. Compliance Requirements: Many security standards (PCI DSS, SOC 2) require non-root execution
      c. Defense in Depth: Additional security layer if container vulnerabilities exist
      d. Audit & Monitoring: Easier to track which user performed which actions


# 06. How Container Users Work
      a. UID and GID Basics
         - UID: User ID number (e.g., root = 0, regular users = 1000+)
         - GID: Group ID number for group permissions
         - User Mapping: Container UIDs map to host UIDs for file permissions

           # Inside container
           id
           # Output: uid=0(root) gid=0(root) groups=0(root)
           
           # With non-root user
           id
           # Output: uid=1001(appuser) gid=1001(appuser) groups=1001(appuser)


# 07. How to Create Non-Root Users in Docker
      
      a. Method 1: Create User in Dockerfile (Recommended)
         
         FROM node:18-alpine
         
         # Create a non-root user
         RUN addgroup -g 1001 -S nodejs && \
             adduser -S nodejs -u 1001 -G nodejs
         
         # Set working directory and change ownership
         WORKDIR /app
         COPY --chown=nodejs:nodejs package*.json ./
         
         # Install dependencies as root (if needed)
         RUN npm ci --only=production
         
         # Copy application files with correct ownership
         COPY --chown=nodejs:nodejs . .
         
         # Switch to non-root user
         USER nodejs
         
         # Expose port and start application
         EXPOSE 3000
         CMD ["node", "server.js"]

         - Key Points:
           1. addgroup -g 1001 -S nodejs creates a system group with GID 1001
           2. adduser -S nodejs -u 1001 -G nodejs creates system user with UID 1001
           3. --chown=nodejs:nodejs ensures copied files are owned by the correct user
           4. USER nodejs switches to the non-root user for subsequent commands

      
      b. Method 2: Use Existing Users from Base Image
         - Many official images provide predefined non-root users:

         # NGINX provides 'nginx' user
         FROM nginx:alpine
         USER nginx
         CMD ["nginx", "-g", "daemon off;"]
         
         # PostgreSQL provides 'postgres' user  
         FROM postgres:13
         USER postgres
         CMD ["postgres"]
         
         # Node.js provides 'node' user
         FROM node:18-alpine
         USER node
         WORKDIR /home/node/app
         COPY --chown=node:node . .
         CMD ["node", "server.js"]
      
      c. Method 3: Create User at Runtime
         FROM alpine:latest
         
         # Create user at runtime
         RUN adduser -D -s /bin/sh appuser
         
         # Switch to non-root user
         USER appuser
         
         # Set working directory in user's home
         WORKDIR /home/appuser
         
         CMD ["sh"]


# 08. Real-World Examples
      
      a. Example 1: Python Flask Application
         
         FROM python:3.11-slim
         
         # Create non-root user
         RUN groupadd -r flaskapp && \
             useradd -r -g flaskapp -d /home/flaskapp -s /bin/bash flaskapp
         
         # Set up application directory
         WORKDIR /app
         
         # Install dependencies as root
         COPY requirements.txt .
         RUN pip install --no-cache-dir -r requirements.txt
         
         # Copy application files and change ownership
         COPY --chown=flaskapp:flaskapp . .
         
         # Switch to non-root user
         USER flaskapp
         
         # Run application
         EXPOSE 5000
         CMD ["python", "app.py"]
      
      b. Example 2: NGINX with Non-Root
         
         FROM nginx:alpine
         
         # NGINX Alpine already has nginx user
         # Copy configuration and content with correct ownership
         COPY --chown=nginx:nginx files/nginx.conf /etc/nginx/nginx.conf
         COPY --chown=nginx:nginx files/html/ /usr/share/nginx/html/
         
         # Switch to nginx user
         USER nginx
         
         # Start NGINX
         CMD ["nginx", "-g", "daemon off;"]
      
      c. Example 3: Java Spring Boot Application
         
         FROM openjdk:11-jre-slim
         
         # Create spring user
         RUN groupadd -r spring && \
             useradd -r -g spring -d /home/spring -m spring
         
         # Set working directory
         WORKDIR /app
         
         # Copy JAR file with correct ownership
         COPY --chown=spring:spring target/myapp.jar app.jar
         
         # Switch to non-root user
         USER spring
         
         # Run Spring Boot application
         EXPOSE 8080
         CMD ["java", "-jar", "app.jar"]


# 09. Best Practices for Non-Root Users
      a. Choose Appropriate UID/GID
         # Use UIDs above 1000 to avoid conflicts with system users
         RUN adduser -D -u 1001 -s /bin/sh appuser
         USER 1001  # Can reference by UID instead of name
      
      b. Handle File Permissions Correctly
         # Ensure application files are owned by non-root user
         COPY --chown=appuser:appuser . /app/
         RUN chown -R appuser:appuser /app/data
      
      c. Create Users Early in Dockerfile
         FROM alpine:latest
         
         # Create user first
         RUN adduser -D -u 1001 appuser
         
         # Do root-required operations
         RUN apk add --no-cache curl
         
         # Switch to non-root before copying application files
         USER appuser
         WORKDIR /home/appuser
      
      d. Use Multi-Stage Builds with Non-Root
         # Build stage (as root)
         FROM node:18-alpine AS builder
         WORKDIR /app
         COPY package*.json ./
         RUN npm ci --only=production
         
         # Runtime stage (as non-root)
         FROM node:18-alpine AS runtime
         RUN addgroup -g 1001 -S nodejs && \
             adduser -S nodejs -u 1001 -G nodejs
         
         USER nodejs
         WORKDIR /app
         COPY --from=builder --chown=nodejs:nodejs /app/node_modules ./node_modules
         COPY --chown=nodejs:nodejs . .
         
         CMD ["node", "server.js"]


# 10. Common Issues and Solutions
      a. Issue 1: Permission Denied Errors
         # Error: Permission denied when writing files
         # Solution: Ensure directories are writable by non-root user

         RUN mkdir -p /app/logs && \
             chown -R appuser:appuser /app/logs && \
             chmod 755 /app/logs
         
         USER appuser
      
      b. Issue 2: Can't Bind to Privileged Ports
         # Error: Cannot bind to port 80 (requires root)
         # Solution: Use non-privileged ports (>1024) or capabilities
         
         # Use port 8080 instead of 80
         EXPOSE 8080
         CMD ["python", "-m", "http.server", "8080"]
         
         # OR grant specific capability
         USER appuser
         # In docker run: --cap-add=NET_BIND_SERVICE

      c. Issue 3: Package Installation Fails
         # Wrong: Try to install packages as non-root user
         USER appuser
         RUN apk add curl  # This will fail
         
         # Correct: Install as root, then switch users
         RUN apk add curl
         USER appuser


# 11. Testing Non-Root Configuration
      - Verify User Inside Container
        
        # Build and run your container
        docker build -t myapp:nonroot .
        docker run -it myapp:nonroot sh
        
        # Check current user
        whoami
        id
        ps aux  # See what user processes are running as
      
      - Test File Permissions
        # Inside container
        touch /tmp/test.txt  # Should work
        touch /etc/test.txt  # Should fail (permission denied)
      
      - Security Scanning
        # Use tools to verify security configuration
        docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
          aquasec/trivy image myapp:nonroot


# 12. Non-Root in Production
      a. Kubernetes Security Context
         
         apiVersion: v1
         kind: Pod
         spec:
           securityContext:
             runAsNonRoot: true
             runAsUser: 1001
             runAsGroup: 1001
             fsGroup: 1001
           containers:
           - name: myapp
             image: myapp:nonroot
             securityContext:
               allowPrivilegeEscalation: false
               readOnlyRootFilesystem: true
               capabilities:
                 drop:
                 - ALL
      
      b. Docker Compose Example
         version: '3.8'
         services:
           web:
             build: .
             user: "1001:1001"  # Override user at runtime
             read_only: true
             tmpfs:
               - /tmp
             cap_drop:
               - ALL
