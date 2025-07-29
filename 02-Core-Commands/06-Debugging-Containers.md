# 01. Debugging Common Docker Container Issues MAIN TAKEWAY:
      - When a container misbehaves—refusing to start, crashing, or not—Docker provides tools (logs, status commands, inspect, exec) to pinpoint 
        and fix issues.
      
      - Think of troubleshooting containers like diagnosing a misfiring car engine: gather error codes (logs), inspect parts (inspect), test 
        components live (exec), and adjust settings (restart with new flags).


# 02. Container Won’t Start
      - The container process exits immediately or fails to launch.

      - Real-World Analogy: You press a car’s ignition, but the engine turns over and shuts off instantly no “running” sound.

      - Common Causes & Fixes
        a. Wrong Command or Entrypoint: The image’s default command is invalid.
           - Solution: Override with a known command:
           - # docker run --rm ubuntu:latest echo "OK"
        
        b. Missing Application Files: The container can’t find its app binary.
           - Solution: Use docker exec into a temporary shell to check file paths:
             # docker run --name test my-app-image sh -c "ls /app" 
        
        c. Permission Denied: Startup script lacks execute permission.
           - Solution: Add RUN chmod +x entrypoint.sh in Dockerfile or mount with correct ownership.


# 03. No Output or Silent Failure
      - Container runs but produces no logs or output.

      - A machine hums but the dashboard shows no speed or error lights.

      - Diagnosis & Solution
        a. Check Logs:
           - # docker logs <container-name/id>
        
        b. Ensure STDOUT/STDERR Usage:
           - Apps must write to standard output, not to files inside the container.
           - Solution: Reconfigure the app to log to console or use a logging driver.

        c. Health-Check Misconfiguration:
           - If a HEALTHCHECK in the Dockerfile always fails, Docker may restart or mark unhealthy.
           - Solution: Temporarily disable health check:
             # docker run --no-healthcheck my-app


# 04. Port Mapping/Networking Problems
      - Host can’t reach the containerized service (web server, API).

      - Your house is wired for electricity, but the circuit breaker is off—no power to devices.

      - Common Issues & Fixes
        a. Port Flag Missing or Wrong:
           - Solution: Expose and publish ports:
             # docker run -d -p 8080:80 nginx

        b. Container Listen Address:
           - Apps binding only to 127.0.0.1 inside the container aren’t reachable via Docker’s bridge network.
           - Solution: Configure the service to listen on 0.0.0.0.

        c. Firewall/Host Rules:
           - Ensure host firewall allows the mapped port.


# 05. File Permission and Mount Issues
      - Containers can’t read/write mounted files or directories.
      
      - Analogy: A locked filing cabinet drawer where you lack the key.

      - Troubleshooting Steps
        a. Inspect Mounts:
           - # docker inspect <container> --format '{{json .Mounts}}'  
        
        b. Check Ownership/Permissions:
           - # docker exec -it <container> ls -l /data 
        
        c. Use :rw or :ro Flags Explicitly:
           - # docker run -v /host/data:/data:rw my-app


# 06. Out-of-Memory (OOM) or CPU Throttling
      - Container gets killed by the kernel OOM killer or slows under CPU limits.

      - Analogy: A runner collapses when denied oxygen (memory) or forced to jog (CPU limit).

      - Detection & Resolution
        a. Check Exit Code:
           - Exit code 137 indicates OOM (128 + 9).
           - # docker ps -a --filter "status=exited"
        
        b. Adjust Resource Flags:
           - # docker run --memory=512m --cpus=1.0 my-app
        
        c. Inspect docker stats:
           - # docker stats <container>


# 07. Image Pull or Build Failures
      - Errors fetching image from registry or building a Dockerfile.

      - Analogy: You order a part, but the supplier shows it’s out of stock, or the workshop can’t fabricate it.

      - Common Scenarios
        
        a. Authentication Required: Private registry needs login.
           - # docker login myregistry.com

        b. Typo in Image Name/Tag:
           - Verify spelling and tag:
           - # docker pull ubuntu:20.04

        c. Build Errors:
           - Syntax errors in Dockerfile.
           - Missing files or incorrect paths in COPY commands.
           - Solution: Run docker build with --progress=plain to get full logs.


# 08. Persistent Data Not Saving
      - Data written inside a container disappears after restart.

      - Analogy: You write notes on a chalkboard, then wipe it clean when you leave.

      - Fix with Volumes
        a. Named Volumes:
           - # docker run -v mydata:/var/lib/data my-app

        b. Bind Mounts:
           - # docker run -v "$(pwd)/data":/var/lib/data my-app




