# 01. Pulling and Running Pre-Built Images from Docker Hub Main Takeaway:
      - Docker Hub is the default public registry of container images. You can “pull” any published image to your local and then “run” it as a 
        container in just two commands.


# 02. Docker Hub and Images
      - Docker Hub is a cloud-hosted registry where anyone can publish container images.

      - An image is a read-only template (e.g., nginx, mysql, node) containing an application and its dependencies.

      - Pulling downloads the image layers to your host; running creates and starts a container from that image.


# 03. Pulling an Image
      - The docker pull command downloads the specified image (and its tags) from Docker Hub:
      
      - Pull the latest official NGINX image
        # docker pull nginx:latest
      
      - Pull a specific version (tag)
        # docker pull nginx:1.25.4
      
      - If no tag is given, :latest is assumed.

      - Layers already present locally are skipped, speeding up pulls.


# 04. Running a Container
      - Use docker run to create and start a container. It will auto-pull the image if not present:

      - Run NGINX in detached mode, mapping port 80 in the container to port 8080 on the host
        # docker run -d -p 8080:80 nginx
      
      - Run an interactive Ubuntu shell
        # docker run -it ubuntu bash
      
      - Run MySQL with an environment-supplied root password
        docker run -d -e MYSQL_ROOT_PASSWORD=my-secret-pw mysql:8.0
      
      - Key Flags:
        a. -d Run in detached (background) mode
        b. -p hostPort:containerPort Publish container’s port to the host
        c. -it Allocate a pseudo-TTY and keep STDIN open (interactive shell)
        d. -e KEY=VALUE Set environment variables inside the container


# 05. Verifying and Managing Containers
      a. List running containers:
         # docker ps
      
      b. List all containers (including stopped):
         # docker ps -a
      
      c. View container logs:
         # docker logs <container-name/id>
      
      d. Stop and remove a container:
         # docker stop <container_id_or_name>
         # docker rm   <container_id_or_name>


# 06. Example: Quick Static Site with HTTPD
      a. Pull the Apache HTTP Server image:
         docker pull httpd
      
      b. Run it, exposing port 80 on your host as port 8080 in the container, and mount a local directory:
         docker run -d \
            -p 8080:80 \
            -v "$(pwd)/site-content":/usr/local/apache2/htdocs:ro \
            httpd
      
      c. Visit http://localhost:8080 in your browser to see your static site.
      