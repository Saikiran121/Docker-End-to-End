# 01. Image Operations Main Takeaway:
      - Docker Images are immutable blueprints, read-only snapshots of application code, runtime libraries, and metadata that you 
        can build, tag, push, pull, inspect, export, and delete.
      
      - Mastering image operations ensures you package, version, distribute, and maintain your application artifacts consistently from development 
        through production.


# 02. What Is a Docker Image?
      - A Docker Image is a layered, read-only filesystem snapshot. Each layer represents a filesystem change (adding files, 
        installing packages, setting environmental variables). Together, the layers form a complete package that can be instantiated as a container.
      
      - Images never run by themselves—they serve as blueprints for containers.

      - Layers are cached and reused to minimize storage and accelerate builds.


# 03. Core Image Operations & Commands
      a. List Images
         - docker images
         - Show all locally stored images
      
      b. Pull Image
         - docker pull <image-name>
         - Download image from a registry
      
      c. Build Image
         - docker build -t <name>:<tag> .
         - Create an image from a Dockerfile in your folder
      
      d. Tag Image
         - docker tag <src> <repo>/<name>:<tag>
         - Assign a new name or version label to an image
      
      e. Push Image
         - docker push <repo>/<name>:<tag>
         - Upload a tagged image to a remote registry
      
      f. Inspect Image
         - docker inspect <image-name>:<tag>
         - Display metadata (layers, size, environment)
      
      g. View History
         - docker history <name>:<tag>
         - List the build commands and layer sizes
      
      h. Remove Image
         - docker rmi <name>:<tag>
         - Delete one or more local images
      
      i. Prune Images
         - docker image prune [-a]
         - Remove dangling (unused) images; add -a for all unused
      
      j. Save Image
         - docker image save -o <file.tar> <name>:<tag>
         - Export image(s) to a tar archive
      
      k. Load Image
         - docker image load -i <file.tar>	
         - Import image(s) from a tar archive


# 04. Step-by-Step Example
      a. Build an Image
         - You write a Dockerfile (recipe) to assemble your app.
         - # docker build -t <name>:<tag> .
      
      b. List Available Images
         - # docker images
      
      c. Tag for Distribution
         - Label it for your Docker Hub repo.
         - # docker image tag myapp:1.0 username/myapp:1.0
      
      d. Push to Registry
         - Publish your recipe online so others can pull it.
         - # docker image push username/myapp:1.0
      
      e. Pull on Another Host
         - A teammate or server retrieves your image.
         - # docker pull username/myapp:1.0
      
      f. Clean Up Old Images
         - Remove outdated recipes to free disk space.
         - # docker image prune -a


# 05. Best Practices
      a. Use Explicit Tags: Avoid latest; use version tags (e.g., v1.0) for repeatable builds.

      b. Multi-Stage Builds: Minimize final image size by copying only necessary artifacts into the runtime image.

      c. Label Metadata: Add LABEL maintainer="you@example.com" in Dockerfiles for provenance.

      d. Automate in CI/CD: Integrate build, tag, scan, and push steps into pipelines to ensure consistency.

      e. Prune Regularly: Schedule docker image prune to prevent disk-space bloat from unused images.


