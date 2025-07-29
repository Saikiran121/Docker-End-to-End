# 01. Copying Files To and From Docker Containers MAIN TAKEWAY:
      - The docker cp command lets you transfer files or folders between your host machine and a runningor stopped) container. It works much like 
        the cp command in Linux, but targets the container’s filesystem.


# 02. What Is docker cp?
      - docker cp is a Docker CLI command to copy files or directories:
        a. From host → container
        b. From container → host
      
      - It does not require mounting volumes; it directly reads and writes via the Docker daemon.


# 03. Real-World Analogy
      - Imagine your computer as your home office desk and your container as a locked filing cabinet in a secure room. You can’t open the cabinet 
        directly, but you have a special courier (the Docker daemon) who can carry papers between your desk and that locked cabinet on request.


# 04. Basic Syntax
      - docker cp [OPTIONS] <source_path> <container>[:<dest_path>]

      - docker cp [OPTIONS] <container>:<source_path> <dest_path>
      
      - <source_path> and <dest_path> can be files or directories.

      - When copying into the container, you specify host_path container_name:/path/.

      - When copying out of the container, you specify container_name:/path/ host_path.


# 05. Copying Files Into a Container
      
      a. Create a sample file on your host:
         echo "ENV=production" > app.env
      
      b. Copy it into a running container named web at /app/:
         docker cp app.env web:/app/app.env
      
      c. Inside the container, verify:
         docker exec web cat /app/app.env


# 06. Copying Files Out of a Container

      a. Assume the container web has a log at /var/log/app.log.

      b. Copy it to your host’s current directory:
         docker cp web:/var/log/app.log ./app.log
      
      c. View it on the host:
         cat app.log


# 07. Copying Directories
      
      a. Into a container
         - Copies the entire my-static-files folder into the container’s web-root.
         - # docker cp ./my-static-files web:/usr/share/nginx/html/
      
      b. Out of a container
         - Pulls the backups directory from inside the container to your host.
         - # docker cp web:/data/backups ./backups


# 08. Common Pitfalls
      a. Paths must exist on the host; Docker will create the target directory inside the container if needed, but not intermediate host paths.

      b. Permissions inside the container matter: copying into a folder owned by root may require running Docker as a user with appropriate 
         permissions.
      
      c. If the container is stopped, you can still use docker cp; the container’s filesystem remains accessible.











