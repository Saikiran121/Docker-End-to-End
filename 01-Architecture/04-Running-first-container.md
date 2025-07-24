# 01. Run the hello-world Container
      - This step both pulls a test image from Docker Hub and runs it:
        # docker run hello-world


# 02. What happens under the hood:
      a. Client request: The Docker CLI sends a “run” request to the Docker daemon.

      b. Image lookup: The daemon checks your local image cache. If the hello-world image isn’t present, it pulls the image layers from Docker Hub.

      c. Container start: The daemon creates a new container from the image, sets up isolated namespaces and cgroups, and executes the tiny “Hello 
         from Docker!” program inside it.
      
      d. Exit: The container prints a welcome message and immediately exits.


# 03. Verify and Explore
      a. List all containers (including exited ones):
         # docker ps -a
      
      b. List downloaded images:
         # docker images 
      
      c. Clean up the test container and image
         # sudo docker rm $(docker ps -aq)        # Remove all containers
         # sudo docker rmi hello-world            # Remove the hello-world image


# 04. After running your first container, you can explore:
      a. Running interactive shells:
         - # docker run -it ubuntu bash
         - This pulls the official Ubuntu image and gives you an interactive shell inside a container.

      b. Trying simple services:
         - # docker run -d -p 8080:80 nginx
         - This launches an NGINX web server in detached mode, visible at http://localhost:8080.



