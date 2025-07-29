# 01. Docker Container Exits Immediately: Troubleshooting Guide
      - When a Docker Container Exits Immediately after starting, Its like having a car that starts but shuts off right away, there's usually a 
        specific reason why it can't keep running. 
      
      - Understanding Why Containers Exit:
        Key Concept to remember: Docker Containers are designed to run as long as their main process is running. When that process stops,
        the container stops. Think of it like a light bulb, when the filament breaks, the light goes out.

-----------------------------------------------------------------------------------------------------------------------------------------
# 02. Step-by-Step Troubleshooting Process
      
      a. Check Container Logs (Your First Detective Tool)
         - Check logs of the exited container
           # docker logs <Container-id/name>
         
         - For more detailed logs
           # docker logs --details <Container-id/name>
         
         - Example Scenario: You run docker run nginx and it exits immediately. The logs might show:
           Error: Could not bind to port 80: Permission denied
           This tells you the problem - port 80 is already in use or needs elevated privileges.

      
      b. Examine Exit Code
         - Check Container status and exit code
           # docker ps -a
         
         - Inspect specific container
           docker inspect <container_name_or_id> | grep ExitCode
      
      
      c. Test with Interactive Mode
         - Run container interactively to see what happens
           # docker run -it <image_name> /bin/bash
         
         - Or override the default command
           # docker run -it <image_name> sh
         
         - Example: If your application container exits immediately, try:
           # docker run -it myapp:latest /bin/bash
           This lets you manually explore inside the container and run commands to see what fails.

     
      d. Check the Dockerfile and Entry Point
         - Common Issues to Look For:
           1. Wrong ENTRYPOINT or CMD: The default command might be incorrect
           2. Missing executable permissions: Scripts might not be executable
           3. Wrong Working Directory: Application cant find required files
         
         - Example Fix: 
           1. Problem: Script not executable
              - COPY start.sh /app/
              - CMD ["./start.sh"]
           
           2. Solution: Make it executable
              - COPY start.sh /app/
              - RUN chmod +x /app/start.sh
              - CMD ["./start.sh"]
      
      
      e. Resource and Environment Issues
         - Check if container ran out of memory
           # docker stats <container_name>
         
         - Check environment variables
           # docker run -it <image_name> env
      

      f. Real-World Example: A Java application container that exits immediately might need more memory:
         - Problem: Container exits due to insufficient memory
           # docker run myapp:latest
         
         - Solution: Increase memory limit
           # docker run -m 512m myapp:latest

----------------------------------------------------------------------------------------------------------------------

# 03. Common Scenarios and Solutions

- **Scenario 1: Web Application Exit**
  **Problem: docker run -d nginx exits immediately**
  **Troubleshooting:**
  docker logs <container_id>
  Shows: "nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Address already in use)"

  **Solution: Use different port**
  docker run -d -p 8080:80 nginx

  ---------------------------------------

- **Scenario 2: Database Container Exit**
  **Problem: MySQL container exits with error**
  **Troubleshooting:**
  docker logs mysql-container
  Shows: "ERROR: Database is uninitialized and password option is not specified"

  **Solution: Provide required environment variables**
  docker run -d -e MYSQL_ROOT_PASSWORD=mypassword mysql:latest

  --------------------------------------------

- **Scenario 3: Custom Application Exit**
  **Problem: Your Python app container exits silently**
  **Troubleshooting Steps:**
  a. Check logs (might be empty)
     - docker logs myapp
  
  b. Run interactively
     - docker run -it myapp:latest /bin/bash
  
  c. Manually run the application
     - python app.py
     Error: ModuleNotFoundError: No module named 'flask'
  
  **Solution: Fix Dockerfile to install dependencies**
  RUN pip install flask


# 04. Memory Trick: Common exit codes and their meanings:
      
      a. 0: Clean exit (process completed successfully)

      b. 1: General application error

      c. 125: Docker Daemon error

      d. 126: Container command not executable

      e. 127: Container command not found

      f. 137: Container killed (out of memory)


# 05. Memory Aid: The "EXIT" Method
      a. E - Examine logs first (docker logs)
      b. X - eXamine exit codes (docker ps -a)
      c. I - Interactive mode testing (docker run -it)
      d. T - Test environment and resources


# 06. Pro Tips to Remember
      a. Always check logs first - They're your best friend in troubleshooting

      b. Use interactive mode when logs don't provide enough information

      c. Check resource limits - Memory and CPU constraints can cause silent exits

      d. Verify file permissions - Especially for startup scripts

      e. Test locally first - Make sure your application works outside Docker before containerizing


