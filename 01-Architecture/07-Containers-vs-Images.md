# 01. Containers vs. Images  Main Takeaway:
      - An Image is a static blueprint (the recipe or box of ingredients) that describes how to build a container.

      - A Container is the running instance (the cooked meal) created from that blueprint.

      - Images sit on disk; containers run in memory and CPU.


# 02. Definitions
      a. Image
         - A read-only template that includes:
           1. Application code
           2. Runtime and libraries
           3. File system snapshots (layers)
           4. Metadata (environment variables, default commands)

         - Stored in a registry (like Docker Hub) or locally.

         - Immutable: you build an image once, then version or share it unchanged.

      b. Container
         - A runnable, isolated process created from an image.

         - Has a writable layer on top of the image’s read-only layers.

         - Shares the host machine’s OS kernel but maintains its own file, network, and process namespaces.

         - Can be started, stopped, paused, and deleted without altering the original image.


# 03. Real-World Analogy: Recipe vs. Dish
      a. Image (Recipe): You follow step-by-step instructions and measure exact ingredients. You can share the recipe file with anyone, and they’ll 
         get the same result.
      
      b. Container (Dish): When you cook, you create a live dish. You can taste-test, add garnish, or discard it after the meal without changing 
         the original recipe.


# 04. How They Work Together
      a. Build an Image
         - Write a Dockerfile (the recipe) with instructions: base OS, copy app code, install dependencies, set default command.
         - Run docker build to assemble layers and produce an image file.

      b. Store and Share
         - Push the image to a registry (docker push).
         - Others pull it (docker pull) to their own hosts.

      c. Run a Container
         - Execute docker run image-name.
         - Docker daemon:
           1. Allocates namespaces and cgroups
           2. Mounts image layers read-only
           3. Adds a fresh writable layer
           4. Starts the application process inside that isolated environment.

      d. Lifecycle
         - Start/Stop: Containers can be launched or stopped without rebuilding an image.
         - Modify at Runtime: Files you change inside a container go into the writable layer, leaving the base image intact.
         - Recreate: Destroy and recreate containers from the same image to reset to a known state.





