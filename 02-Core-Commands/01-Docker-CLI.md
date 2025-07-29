# 01. Docker CLI    
      - Containers are only as powerful as the commands you use to control them. The Docker Command Line Interface(CLI) is your
        "Remote Control" for the entire Docker platform, letting you build images, run containers, insect problems, and automate
        complex workflows.


# 02. Overview: Why the CLI Matters
      - The Docker CLI is a thin client that talks to the Docker Daemon(dockerd). When you type a command, the CLI sends a REST request
        to the daemon, the daemon does the heavy lifting, and you see the results in your terminal


# 03. Understanding the docker Base Command
      - docker alone is the root of the CLI tree; sub-commands branch from it. Run these explorations:
        docker --help   # List every supported sub-command
        docker version  # Client and server versions
        docker info     # System-wide details: images, containers, driver, cgroup mode
      
      - Key Concepts
        a. Client vs. Server: The CLI is merely a messenger; the daemon executes
        b. Help Flags: Every sub-command supports --help for on-demand examples
    

# 04. Real-World Analogy
      - Think of the CLI as the ordering kiosk at the fast-food chain. You tap buttons (docker run nginx), the kitchen (daemon) cooks, and your 
        meal (container) appears.


# 05. Pulling Images
      - Image = read-only template; similar to a recipe card stored in a cookbook (registry).
      - Pulling downloads only layers you don’t already have, saving bandwidth.

        docker pull nginx        # Latest tag assumed
        docker pull nginx:1.25.4        #  Specific version[6]


# 06. Inspecting Local Images
      - docker images              # Short form list[1]

      - docker image ls             # Identical modern form

      - docker image inspect nginx      # JSON metadata dump


# 07. Removing Images
      - docker rmi nginx:1.25.4             # Delete unused image[1]

      - docker image prune -a               # Auto-remove dangling & untagged images[20]


# 08. Creating & Running
      - docker run is the Swiss-army knife that performs three hidden steps: pull (if needed) → create → start

      - docker run hello-world              # Quick test

      - docker run -d -p 8080:80 nginx      # Background web server

      - docker run -it ubuntu bash          # Interactive shell

      - Meaning:
        a. -d --> Detached (background) --> Cook while you leave the counter
        b. -p a:b --> Map host a → container b  --> Restaurant drive-thru window
        c. --name: --> Custom container name    --> Label your lunchbox


# 09. Listing & Stopping    
      - List running Containers
        # docker ps
      
      - List All Containers
        # docker ps -a 
      
      - Stop a Container
        # docker stop <container-name/id>
      
      - Remove a Stopped Container
        # docker rm <container-name/id>
      
      - Bulk clean-up
        # docker container prune


# 10. Start, Restart, Kill
      - Start a Container
        # docker start <container-name/id>
      
      - Restart a Container
        # docker restart <container-name/id>
      
      - Kill a Container
        # docker kill <container-name/id>


# 11. Logs & Live Output
      - Logs of a Container
        # docker logs <container-name/id>
      
      - Get the Live Logs of a Container
        # docker logs -f --tail 100 <container-name/id>


# 12. Execute Commands
      - Open an Interactive Shell
        # docker exec -it <container-name/id> bash 
      
      - List Contents in a Container 
        # docker exec <container-name/id> ls /usr/share/nginx/html
