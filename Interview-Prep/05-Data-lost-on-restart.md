# 01. Data Lost When Container Stops and Restarts: Complete Fix Guide

- **When Docker Containers lose data on restart, its because containers are ephemeral by design ther are meant to be disposable. Think of a 
  container like a notebook page that gets thrown away every time you finish using it. If you want to keep your notes, you need a separate filing cabinet.**


#### Understanding Why Data Loss Happens

- **Key Concept to Remember: By default, all data written inside a container exists only in the container's writable layer. When the container
    stops, the layer disappears forever, taking your data with it.**
<br>

- **Memory Hook: "Container = Temporary, Volume = Permanent"**


### The Solutions: Three Ways to Persist Data

#### a. Docker Volumes (Recommended for Production)
- **Docker Volumes are managed storage area that live outside the container. They're the preferred method because Docker handles all the complexity
    for you.**
<br>

- **Creating and Using Volumes:**
  ```
  # Create a named volume
  docker volume create myapp_data

  # Run container with volume attached
  docker run -d -v myapp_data:/app/data myapp:latest

  # Or let Docker create the volume automatically
  docker run -d -v myapp_data:/app/data myapp:latest
  ```

- **Real-World Example - Database Persistence:**
  ```
  # Problem: MySQL data lost on restart
  docker run -d --name mysql-container mysql:8.0
  # Data disappears when container stops!

  # Solution: Use volume for database files
  docker volume create mysql_data

  docker run -d --name mysql-container \
    -v mysql_data:/var/lib/mysql \
    -e MYSQL_ROOT_PASSWORD=secret \
    mysql:8.0
  
  # Data now persists between restarts!
  ```


#### b. Bind Mounts (Great for Development)
- **Bind mounts directly connect a host directory to a container directory. You control exactly where the data is stored on your host machine.**
<br>

- ```
  # Mount host directory into container
  docker run -d -v /host/path:/container/path myapp:latest

  # Example: Web application with persistent uploads
  docker run -d -v /home/user/uploads:/app/uploads webapp:latest
  ```

- **Example Scenario: You're developing a web app that processes user files:**
  ```
  # Create directory on host
  mkdir -p /home/user/project/uploads
  
  # Run container with bind mount
  docker run -d -p 8080:8080 \
    -v /home/user/project/uploads:/app/uploads \
    mywebapp:latest
  
  # Files uploaded through the web app are now saved on your host
  ```


#### c. Docker Compose for Multi-Container Apps
- **For applications with multiple containers (like webapp + database), use Docker Compose to manage volumes automatically**
  
  ```
  version: '3.8'
  services:
    web:
      image: mywebapp:latest
      ports:
        - "8080:8080"
      volumes:
        - app_uploads:/app/uploads
        
    database:
      image: postgres:13
      environment:
        POSTGRES_PASSWORD: secret
      volumes:
        - db_data:/var/lib/postgresql/data
  
  volumes:
    app_uploads:
    db_data: 
  ```


## Step-by-Step Fix Process

#### Step 1: Identify What Data Needs to Persist
- **Common data directories by application type:
    a. Databases: /var/lib/mysql, /var/lib/postgresql/data
    b. Web servers: /var/www/html, /usr/share/nginx/html
    c. Application data: /app/data, /app/uploads, /app/logs**
<br>

#### Step 2: Choose Your Persistence Method
- ```
  # For production databases - use named volumes
  docker volume create db_data
  docker run -d -v db_data:/var/lib/mysql mysql:8.0
  
  # For development files - use bind mounts 
  docker run -d -v ./local-folder:/app/data myapp:latest
  
  # For configuration files - use read-only bind mounts
  docker run -d -v ./config.yml:/app/config.yml:ro myapp:latest
  ```
<br>

#### Step 3: Verify Data Persistence
- ```
  # Create test data
  docker exec -it container_name touch /app/data/test-file.txt
  
  # Stop and restart container
  docker stop container_name
  docker start container_name

  # Check if data persists
  docker exec -it container_name ls /app/data/
  # You should see test-file.txt


## Common Scenarios and Solutions

#### Scenario 1: Database Data Loss
#### Problem: PostgreSQL container loses all data on restart

- ```
  # Wrong: No volume specified
  docker run -d --name postgres -e POSTGRES_PASSWORD=secret postgres:13
  

#### Solution

- ```
  # Correct: Volume for database files
  docker run -d --name postgres \
    -e POSTGRES_PASSWORD=secret \
    -v postgres_data:/var/lib/postgresql/data \
    postgres:13


#### Scenario 2: Web Application Uploads Disappear
#### Problem: User uploaded files vanish when container restarts

- ```
  # Wrong: Uploads stored in container
  docker run -d mywebapp:latest
  ```

#### Solution

- ```
  Correct: Bind mount for uploads directory
  mkdir -p ./uploads
  docker run -d -v ./uploads:/app/uploads mywebapp:latest


#### Scenario 3: Development Environment Reset
#### Problem: Code changes lost during development

- ```
  # Wrong: Code only exists in container
  docker run -d -p 3000:3000 node-dev:latest
  ```

#### Solution

- ```
  Correct: Bind mount source code directory
  docker run -d -p 3000:3000 \
    -v ./src:/app/src \
    node-dev:latest


## Volume Management Commands

```
# List all Volumes
docker volume ls

# Inspect volume details
docker volume inspect volume_name

# Remove unused volumes
docker volume prune

# Remove specific volume
docker volume rm volume_name

# Backup volume data
docker run --rm -v volume_name:/data -v $(pwd):/backup \
  ubuntu tar czf /backup/backup.tar.gz /data
```


## Memory Aid: The "SAVE" Method
- S - Specify volumes for critical data directories
- A - Attach volumes using -v flag or Docker Compose
- V - Verify data persists after container restart
- E - Establish backup strategy for volume data


## Best Practices to Remember
- Always use volumes for databases. Never run production databases without persistent storage
- Use named volumes over anonymous volumes - Easier to manage and reference
- Implement backup strategies - Volumes can still be lost if the host fails
- Use read-only mounts for configuration files that shouldn't change
- Test persistence - Always verify your data survives container restarts


## Pro Tips for Production
- Use volume drivers for distributed storage in clusters
- Set up automated backups of volume data to external storage
- Monitor volume disk usage to prevent running out of space
- Use security scanning on volumes containing sensitive data