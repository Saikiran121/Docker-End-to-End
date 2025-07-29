# 01. How to Debug a Live Container: Complete Debugging Guide
- **When debugging a live Docker container, you need to diagnose issues in running containers without stopping or recreating them.
  Think of it like being a doctor examining a patient who's awake you need to check vital signs, symptoms, and internal conditions
  while the system is still functioning.**


## Understanding Live Container Debugging
- **Memory Hook: "Look, Log, Link, Live-test" - Observe status, check logs, connect interactively, and test in real-time.**

- **Live container debugging is different from troubleshooting startup issues because:
  a. The container is running but behaving unexpectedly
  b. You need to preserve the current state for investigation
  c. Changes must be made carefully to avoid disrupting the live system**


# 02. Step-by-Step Debugging Process

#### a. Quick Health Check (External View)
- **Start by examining the container from the outside before going in:**
  ```
  # Check container status and resource usage
  docker ps -a
  docker stats container_name
  
  # Check recent logs for obvious errors
  docker logs --tail 50 container_name
  
  # Follow logs in real-time
  docker logs -f container_name
  ```

- **Example Scenario: Your web application is responding slowly**
  ```
  $ docker stats webapp-container
  CONTAINER     CPU %     MEM USAGE     MEM %     NET I/O
  webapp        85.5%     1.2GB / 2GB   60%       1.2MB / 800KB
  # High CPU usage indicates a performance issue
  ```


#### b. Interactive Shell Access (Internal View)
- **Get inside the container to examine the environment directly:**
  ```
  # Access container shell
  docker exec -it container_name /bin/bash
  # Or if bash isn't available
  docker exec -it container_name /bin/sh
  ```

- **What to check once inside:**
  ```
  # Check running processes
  ps aux
  
  # Check system resources
  top
  free -h
  df -h
  
  # Check network connections
  netstat -tlnp
  
  # Examine application-specific directories
  ls -la /app/
  ls -la /var/log/
  ```

#### c. Application-Level Debugging
- **Example: Node.js Application Issues**
  ```
  # Inside the container
  docker exec -it node-app bash
  
  # Check if the app is actually running
  ps aux | grep node
  
  # Check application logs
  tail -f /app/logs/app.log
  
  # Test internal endpoints
  curl localhost:3000/health
  
  # Check environment variables
  printenv | grep NODE
  ```

- **Example: Database Container Issues**
  ```
  # Access database container
  docker exec -it postgres-db bash
  
  # Connect to database
  psql -U postgres -d mydb
  
  # Check database connections
  SELECT * FROM pg_stat_activity;
  
  # Check database size and performance
  SELECT pg_size_pretty(pg_database_size('mydb'));
  ```


# 03. Real-World Debugging Scenarios

#### Scenario 1: Web App Returns 500 Errors
#### Problem: Application throwing internal server errors randomly
```
# 1. Check recent error logs
docker logs --tail 100 webapp | grep ERROR

# 2. Monitor logs in real-time
docker logs -f webapp &

# 3. Access container for investigation
docker exec -it webapp bash

# 4. Inside container - check application status
ps aux | grep python
curl localhost:8000/health

# 5. Check for resource issues
free -h
df -h

# 6. Examine application logs
tail -f /app/logs/error.log
```

#### What you might find:
```
# Example findings
$ df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1       10G   9.8G  200M  98%   /app/data
# Disk full - causing write failures
```

--------------------------------------------------------------------------------

#### Scenario 2: Database Connection Issues
#### Problem: App can't connect to database container
```
# 1. Test network connectivity between containers
docker exec -it webapp bash
ping database-container
telnet database-container 5432

# 2. Check database container status
docker exec -it database-container bash
ps aux | grep postgres

# 3. Check database is accepting connections
netstat -tlnp | grep 5432

# 4. Test database login
psql -U app_user -d app_db -h localhost
```

----------------------------------------------------------------------------------------

#### Scenario 3: Memory Leak Investigation
#### Problem: Container memory usage keeps increasing
```
# 1. Monitor memory usage over time
docker stats container_name

# 2. Inside container - identify memory-hungry processes
docker exec -it container_name top

# 3. For Node.js apps - check for memory leaks
docker exec -it node-app bash
node --inspect=0.0.0.0:9229 app.js

# 4. Generate heap dump (if tools available)
kill -USR2 <node-process-pid>
```

# 04. Advanced Debugging Techniques

- **Docker Debug Command (New Feature)**
  ```
  # Use Docker's built-in debug tools
  docker debug container_name
  
  # This provides a toolbox with debugging utilities
  # Available in Docker Desktop 4.27+
  ```

- **Attach to Container Process**
  ```
  # Stream container output in real-time
  docker attach container_name
  
  # Note: This shows stdout/stderr but doesn't give you shell access
  # Use Ctrl+C carefully - might stop the container
  ```

- **Copy Files for Analysis**
  ```
  # Copy logs or config files out for analysis
  docker cp container_name:/app/logs/error.log ./local-error.log
  docker cp container_name:/etc/myapp/config.yml ./config-analysis.yml
  
  # Copy files into container for debugging
  docker cp debug-script.sh container_name:/tmp/
  docker exec -it container_name bash -c "chmod +x /tmp/debug-script.sh && /tmp/debug-script.sh"
  ```

- **Network Debugging**
  ```
  # Check container network configuration
  docker inspect container_name | grep -A 20 NetworkSettings
  
  # Test connectivity to external services
  docker exec -it container_name curl -v https://api.external-service.com
  
  # Check DNS resolution
  docker exec -it container_name nslookup database-host
  ```


# 05. Debugging Different Container Types

- **Web Applications**
  ```
  # Check HTTP responses
  docker exec -it webapp curl -I localhost:8080
  
  # Test specific endpoints
  docker exec -it webapp curl -v localhost:8080/api/health
  
  # Check application server status
  docker exec -it webapp ps aux | grep -E "(nginx|apache|node|python)"
  ```

- **Databases**
  ```
  # PostgreSQL debugging
  docker exec -it postgres bash
  psql -U postgres
  \l  # List databases
  \dt # List tables
  SELECT version();
  
  # MySQL debugging  
  docker exec -it mysql bash
  mysql -u root -p
  SHOW DATABASES;
  SHOW PROCESSLIST;
  ```

- **Microservices**
  ```
  # Check service discovery
  docker exec -it service1 nslookup service2
  
  # Test inter-service communication
  docker exec -it service1 curl service2:8080/health
  
  # Check load balancer configuration
  docker exec -it nginx cat /etc/nginx/nginx.conf
  ```


# 06. Memory Aid: The "DEBUG" Method
- a. D - Diagnose externally first (docker stats, docker logs)
- b. E - Enter container with interactive shell
- c. B - Browse processes, files, and configurations
- d. U - Understand the application state and environment
- e. G - Generate test scenarios to reproduce issues


# 07. Production-Safe Debugging Tips

- **Non-Disruptive Commands**
  ```
  # Safe commands that won't affect running processes
  docker logs container_name
  docker stats container_name  
  docker inspect container_name
  docker exec -it container_name ps aux
  ```

-  **Read-Only Operations**
  ```
  # Examine without modifying
  docker exec -it container_name cat /app/config.json
  docker exec -it container_name ls -la /var/log/
  docker exec -it container_name env
  ```

- **Create Debug Snapshots**
  ```
  # Create a snapshot for analysis without affecting live container
  docker commit container_name debug-snapshot
  docker run -it debug-snapshot bash
  ```


# 08. Common Issues and Quick Fixes

#### Issue 1: Application Not Responding
```
# Quick diagnostic
docker exec -it container_name ps aux | grep app
docker exec -it container_name netstat -tlnp
docker exec -it container_name curl localhost:port/health
```

#### Issue 2: Permission Problems
```
# Check file ownership and permissions
docker exec -it container_name ls -la /app/
docker exec -it container_name whoami
docker exec -it container_name id
```

#### Issue 3: Configuration Issues
```
# Verify configuration files
docker exec -it container_name cat /app/config.json
docker exec -it container_name env | grep -i config
```

# 09. Best Practices for Live Debugging
- a. Always backup first - Create snapshots before making changes
- b. Use read-only commands initially - Understand before modifying
- c. Monitor resource impact - Watch CPU/memory while debugging
- d. Document findings - Keep notes of what you discover
- e. Test hypotheses carefully - Make small, controlled changes
- f. Have rollback plan - Know how to revert if debugging breaks something


# 10. Emergency Debugging Checklist
- **When debugging a critical production issue:**
  ```
  # 1. Quick status check
  docker ps | grep problematic-container
  docker logs --tail 20 problematic-container
  
  # 2. Resource check
  docker stats problematic-container --no-stream
  
  # 3. Process check
  docker exec problematic-container ps aux
  
  # 4. Quick connectivity test
  docker exec problematic-container curl localhost:port/health
  
  # 5. If all else fails - snapshot and restart
  docker commit problematic-container emergency-backup
  docker restart problematic-container
  ```












  
  


