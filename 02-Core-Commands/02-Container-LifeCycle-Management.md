# 01. Container Lifecycle Management Main Takeaway
      - Container lLifecycle Management is like being a restaurant manager, you control when dishes are prepared (created), served (running),
        temporarily set aside (paused), properly cleaned up (stopped), and finally removed from inventory (deleted).
        Each stage has specific commands and timing that determine how gracefully your "digital kitchen" operates.


# 02. What Is Container Lifecycle Management?
      - Container Lifecycle Management refers to Controlling the different stages a container goes through from creation to deletion.

      - Just like managing a restaurant kitchen, you need to orchestrate when containers are created, started, paused, stopped and
        removed to ensure smooth operations.


# 03. Key Definition:
      - Container lifecycle management is the process of overseeing containers through their complete journey: created → running → paused 
        (optional) → stopped → removed.


# 04. Real-World Analogy: Restaurant Kitchen Management
      - Imagine you're managing a busy restaurant kitchen where each "dish" represents a container:

      a. Created
         - Restaurant Analogy: Recipe prepared, ingredients ready	
         - What's Happening: Container
      
      b. Running
         - Restaurant Analogy: Chef actively cooking the dish
         - What's Happening: Container processes are executing
      
      c. Paused
         - Restaurant Analogy: Put dish on hold, burner turned off temporarily	
         - What's Happening: Processes frozen in memory
      
      d. Stopped
         - Restaurant Analogy: Dish finished, chef cleans station	
         - What's Happening: Container processes ended gracefully
      
      e. Killed
         - Restaurant Analogy: Emergency shutdown, abandon dish immediately	
         - What's Happening: Container processes terminated forcefully
      
      f. Removed
         - Restaurant Analogy: Recipe card thrown away, station cleared	
         - What's Happening: Container completely deleted


# 05. The Six Critical Container States
      a. Created State
         - What it Means: Container exists but hasn't started running yet.

         - docker create --name my-nginx nginx

         - Real-world comparison: Like having all ingredients measured and ready, but the chef hasn't started cooking yet.
           No CPU, No MEMORY is consumed beyond basic overhead.
      
      b. Running State
         - What it Means: Container Processes are actively executing

         - Start the created container
           # docker start my-nginx
         
         - Or create and start in one command
           # docker run -d --name web-server nginx 
         
         - Kitchen analogy: The chef is actively cooking - using heat, stirring, monitoring. Full CPU and memory resources are being used
      
      c. Paused State
         - What it means: All processes inside the container are temporarily suspended but remain in memory.

         - Pause a running container
           # docker pause my-nginx
         
         - Resume it later
           # docker unpause my-nginx
         
         - Key insight: It's like hitting pause on a video game - the container picks up exactly where it left off when unpaused. CPU usage drops 
           to 0% but memory usage remains.
         
         - Technical detail: Uses Linux cgroups freezer to suspend processes without the container being aware it was paused.
      
      d. Stopped/Exited State
         - What it means: Container processes have been gracefully shut down.

         - Gracefully stop a container (sends SIGTERM, then SIGKILL after timeout)
           # docker stop my-nginx
         
         - Process: Docker sends SIGTERM signal → waits 10 seconds → sends SIGKILL if needed.
      
      e. Killed State
         - What it means: Container processes were immediately terminated without cleanup time.

         - Force-kill a container immediately (sends SIGKILL)
           # docker kill my-nginx
         
         - Key difference from stop: docker kill skips straight to SIGKILL, giving no time for graceful shutdown.

      f. Removed/Deleted State
         - What it means: Container is completely deleted from the system.

         - Remove a stopped container
           # docker rm my-nginx
         
         - Force-remove a running container
           # docker rm -f my-nginx


# 06. When to Use Each Command
      a. Use docker stop when:
         - Production environments where data integrity matters
         - Database containers that need to close connections properly
         - Web servers handling active requests
         - You have time for graceful shutdown
      
      b. Use docker kill when:
         - Container is unresponsive or stuck
         - Emergency situations requiring immediate resource cleanup
         - Container won't respond to docker stop
         - You need instant termination (testing scenarios)
      
      c. Use docker pause/unpause when:
         - Resource-intensive tasks that need temporary suspension
         - Development/testing scenarios requiring state preservation
         - System maintenance while keeping container state
         - Debugging timing-related issues


