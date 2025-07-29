# 01. Docker Container Exits immediately. how will you troubleshoot

##  Professional Interview Response
-   **"Great question. In my experince, container exit issue are quite common, and I have developed a systematic approach to troubleshoot
    them efficiently."**


## My Go-To Troubleshooting Process
-  **First, I always start with the container logs using docker logs <container-id/name> bacause they usually reveal the root cause
   immediately. I have found that about 80% of exit issues can be diagnosed just from the logs.** 
<br>
-  **Next, I check the exit code with docker ps -a. Over the years, I have memorized the key ones, 137 usually means OOM kill, 126 is permission
   issues, and 127 is command not found. These codes instantly point me in the right direction.**


## Real Examples from My Experience
-  **Let me share a recent example, We had a microservice that kept exiting in production. The logs showed 'permission denied' when trying to write 
   to a volume. Turned out the container was running as non-root user, but the mounted directory had root ownership. I fixed it by adjusting the
   Dockerfile to match the host permissions.**
<br>
-  **Another common scenario I've deal with is Java applications exiting due to insufficient heap memory. The container would start fine locally 
   but crash in production under load. I learned to always set appropriate JVM flags like -Xmx based on the container's memory limits.**


## Advanced Troubleshooting Techniques
-  **When logs aren't helpful, I use docker run -it --entrypoint=/bin/bash <image> to get inside and manually debug. I've also used docker exec on 
   failing containers to inspect the runtime environment.**
<br>
-  **For intermittent issues, I've set up health checks and monitoring. Sometimes containers exit due to external dependencies - database 
   connections, API timeouts, etc. Having proper logging and monitoring helps identify these patterns.**


# 02. Production Considerations
      - In production environments, I always ensure we have:
        a. Proper restart policies (--restart=unless-stopped)
        b. Resource limits to prevent one container from affecting others
        c. Centralized logging so we don't lose logs when containers exit
        d. Health checks to catch issues before they cause exits


# 03. Quick Decision Making
      - With experience, I can usually categorize the issue within the first few minutes:
        a. Application errors (fix the code/config)
        b. Environment issues (permissions, missing files, network)
        c. Resource constraints (memory, CPU, disk space)
        d. Infrastructure problems (host issues, orchestration problems)"



