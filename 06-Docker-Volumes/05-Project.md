# 01. Complete Docker Volume Example: Create, Attach, Transfer Files, Backup & Restore
      - This hands-on example demonstrates the complete Docker volume lifecycle—creating a container with persistent storage, adding files,
        backing up the volume data, and restoring it to a new container. This workflow is essential for database migrations, development environment
        setup, and disaster recovery scenarios.


# 02. Step-by-Step Complete Example
    
#### a. Create a Named volume
```
# Create a named volume for persistent storage
docker volume create myapp-data

# Verify the volume was created
docker volume ls
```

#### b. Expected Output:
```
DRIVER    VOLUME NAME
local     myapp-data
```

#### c. Create Container and Attach Volume
```
# Run an Ubuntu container with the volume mounted at /data

docker run -it \
    --name data-container \
    -v myapp-data:/data \
    ubuntu:22.04 bash
```

- This starts an interactive Ubuntu container with:
  a. Named volume myapp-data mounted at /data inside the container
  b. Container named data-container for easy reference


#### d. Add Files to the Volume (Inside Container)
- Inside the running container terminal:
```
# Navigate to the mounted volume directory
cd /data

# Create some sample files and directories
echo "Hello from Docker Volume!" > welcome.txt
echo "This is application data" > app-config.txt

# Create a directory with multiple files
mkdir logs
echo "$(date): Application started" > logs/app.log
echo "$(date): Database connected" > logs/db.log

# Create a nested directory structure
mkdir -p backups/2025/july
echo "Important backup data" > backups/2025/july/backup.sql

# List all created content
find /data -type f -exec ls -la {} \;

# Exit the container
exit
```

#### e. Verify Data Persistence
```
# Remove the container (volume data remains safe)
docker rm data-container

# Create a new container with the same volume to verify persistence
docker run --rm -v myapp-data:/data alpine ls -la /data
```

##### Expected Output:
```
total 16
drwxr-xr-x    4 root     root          4096 Jul 30 09:30 .
drwxr-xr-x    1 root     root          4096 Jul 30 09:30 ..
-rw-r--r--    1 root     root            26 Jul 30 09:30 app-config.txt
drwxr-xr-x    3 root     root          4096 Jul 30 09:30 backups
drwxr-xr-x    2 root     root          4096 Jul 30 09:30 logs
-rw-r--r--    1 root     root            27 Jul 30 09:30 welcome.txt
```

#### f. Backup the Volume Data
- Create a backup of all volume data into a compressed archive:
```
# Create backup directory on host
mkdir -p ~/docker-backups

# Backup volume data to a tar.gz file
docker run --rm \
    -v myapp-data:/data:ro \
    -v ~/docker-backups:/backup \
    alpine \
    tar czf /backup/myapp-data-backup-$(date +%Y%m%d-%H%M%S).tar.gz -C /data .

# Verify backup was created
ls -la ~/docker-backups/
```

- What this backup command does:
  a. Mounts the volume as read-only (:ro for safety)
  b. Mounts host backup directory to /backup in container
  c. Creates compressed tar archive with timestamp
  d. Archives all files from /data directory


#### g. Create New Volume and Restore Data
- Simulate a disaster recovery scenario by restoring to a new volume:

```
# Create a new volume for restoration
docker volume create myapp-restored

# Get the backup filename (adjust the timestamp to match your backup)
BACKUP_FILE=$(ls ~/docker-backups/myapp-data-backup-*.tar.gz | head -1)
echo "Restoring from: $BACKUP_FILE"

# Restore data from backup to new volume
docker run --rm \
  -v myapp-restored:/data \
  -v ~/docker-backups:/backup \
  alpine \
  tar xzf /backup/$(basename $BACKUP_FILE) -C /data

# Verify restored data
docker run --rm -v myapp-restored:/data alpine find /data -type f
```

#### h. Test Restored Data in New Container
```
# Create a new application container using restored volume
docker run -it \
  --name restored-container \
  -v myapp-restored:/data \
  ubuntu:22.04 bash
```

###### Inside the new container:
```
# Verify all files were restored correctly
cd /data
cat welcome.txt
cat app-config.txt
cat logs/app.log
cat backups/2025/july/backup.sql

# Add new content to verify write access
echo "$(date): Restored successfully!" >> logs/restore.log

exit
```

# 03. Complete Workflow Script
```
#!/bin/bash

echo "=== Docker Volume Backup & Restore Demo ==="

# Step 1: Create volume and add data
echo "1. Creating volume and adding sample data..."
docker volume create demo-data

docker run --rm -v demo-data:/data alpine sh -c '
  echo "Sample application data" > /data/app.txt
  echo "Database configuration" > /data/config.json
  mkdir -p /data/logs
  echo "Application logs" > /data/logs/app.log
  echo "Files created successfully"
'

# Step 2: List volume contents
echo "2. Current volume contents:"
docker run --rm -v demo-data:/data alpine find /data -type f -exec ls -la {} \;

# Step 3: Create backup
echo "3. Creating backup..."
mkdir -p ~/docker-backups
BACKUP_NAME="demo-backup-$(date +%Y%m%d-%H%M%S).tar.gz"

docker run --rm \
  -v demo-data:/data:ro \
  -v ~/docker-backups:/backup \
  alpine \
  tar czf /backup/$BACKUP_NAME -C /data .

echo "Backup created: ~/docker-backups/$BACKUP_NAME"

# Step 4: Simulate data loss
echo "4. Simulating data loss (removing original volume)..."
docker volume rm demo-data

# Step 5: Restore to new volume
echo "5. Restoring data to new volume..."
docker volume create demo-restored

docker run --rm \
  -v demo-restored:/data \
  -v ~/docker-backups:/backup \
  alpine \
  tar xzf /backup/$BACKUP_NAME -C /data

# Step 6: Verify restoration
echo "6. Verifying restored data:"
docker run --rm -v demo-restored:/data alpine find /data -type f -exec cat {} \;

echo "=== Demo completed successfully! ==="
```

# 04. Real-World Use Cases
#### a. Database Migration Example
```
# Backup production database
docker run --rm \
  -v postgres-data:/var/lib/postgresql/data:ro \
  -v ~/db-backups:/backup \
  postgres:15 \
  pg_dumpall -U postgres > /backup/full-backup.sql

# Restore to new environment
docker volume create postgres-new
docker run --rm \
  -v postgres-new:/var/lib/postgresql/data \
  -v ~/db-backups:/backup \
  postgres:15 \
  psql -U postgres < /backup/full-backup.sql
```

#### b. Development Environment Setup
```
# Backup your development environment
docker run --rm \
  -v dev-environment:/workspace:ro \
  -v ~/dev-backups:/backup \
  alpine \
  tar czf /backup/dev-env-$(date +%Y%m%d).tar.gz -C /workspace .

# Share with team member (restore on their machine)
docker volume create colleague-env
docker run --rm \
  -v colleague-env:/workspace \
  -v ~/dev-backups:/backup \
  alpine \
  tar xzf /backup/dev-env-20250730.tar.gz -C /workspace
```

