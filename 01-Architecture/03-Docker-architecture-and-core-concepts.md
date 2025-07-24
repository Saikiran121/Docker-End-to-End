# 01. Docker Architecture & Core Concepts Main Takeaway:
      - Docker uses a client-server model to package applications into lightweight, portable containers. Think docker as shipping system;
        you build standardized boxes (images), store and ship them via a registry, and deploy them on any dock (host) using the Docker
        engine. 


# 02. The Client–Server Model
      a. Docker Client: The user interface (CLI or API) where you type commands like docker build or docker run.

      b. Docker Daemon (dockerd): The background service on the host machine that does the heavy lifting—building images, running containers, and 
        managing resources
      
      c. Registry: A repository (public like Docker Hub or private) that stores and distributes Docker images


# 03. Real-World Analogy:
      - The client is like your shipping office where you prepare shipping orders.

      - The daemon is the warehouse staff who actually pack boxes, load trucks, and track inventory.

      - The registry is the distribution center where packed boxes are stored until they’re shipped to destinations.


# 04. Key Docker Objects
      a. Image
         - Definition: Read-only template of instructions (code + dependencies) used to create containers.	
         - Analogy: A sealed, labeled shipping box
      
      b. Container
         - Definition: A running instance of an image, isolated but sharing the host OS kernel.	
         - Analogy: The unpacked box, contents in use
      
      c. Volume
         - Definition: Named, persistent storage attached to one or more containers for data that outlives a container.	
         - Analogy: A reusable storage bin in the warehouse
      
      d. Network
         - Definition: Virtual bridges or overlays connecting containers for communication.	
         - Analogy: The shipping routes linking warehouses


# 05. How It Works: Workflow Step-by-Step
      a. Build Phase
         - Write a Dockerfile (a script of build instructions).
         - Run docker build → Docker daemon executes each step, creating an image layer by layer
      
      b. Ship Phase
         - Push the image to a registry with docker push.
         - Other users or servers pull it via docker pull.

      c. Run Phase
         - Execute docker run image-name: the daemon creates a container, sets up isolated namespaces (processes, filesystems, network) and applies 
           cgroup limits (CPU, memory)


# 06. Analogy:
      a. Build: Packing items into a box following a checklist.

      b. Ship: Sending the box to a central storage facility.

      c. Run: Unpacking the box at the destination and setting up your workspace.


# 07. Core Concepts Under the Hood
      a. Namespaces: Ensure each container has its own view of processes, network interfaces, and filesystems (prevents “cross-talk”).

      b. Control Groups (cgroups): Restrict how much CPU, memory, and I/O each container can use (prevents resource hogging).

      c. Union File Systems: Build image layers atop each other; only changed files are stored in new layers (efficient storage).


# 08. Simple Real-World Example
      - Imagine you’re a food truck owner:
        a. You create a recipe booklet (Dockerfile) describing how to prepare a dish.
        b. You pack ingredients and tools into a labeled box (image).
        c. You send boxes to various event sites (registry).
        d. At each event, you unpack and set up on site (container), using a shared kitchen (host OS) but with your own utensils and inventory 
           controls (namespaces & cgroups).


# 09. Why Docker Matters
      a. Portability: “It works on my machine” becomes “it works everywhere” — any host with Docker.

      b. Speed: Containers start in seconds vs. virtual machines in minutes.

      c. Efficiency: Share one OS kernel; run dozens of containers on the same host with minimal overhead

      d. Consistency: Built-once, run-anywhere images eliminate environment mismatches.



