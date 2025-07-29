# 01. Dockerfile Syntax & Best Practices MAIN TAKEWAY:
      - A Dockerfile is a plain-text script of instructions (“recipe”) that tells Docker how build a container image.

      - By following simple syntactic rules and best practices—like minimizing layers, using official base images, and handling secrets 
        securely you create images that are small, secure, and maintainable.


# 02. What Is a Dockerfile?
      - A Dockerfile is a text file containing a set of commands (instructions) and arguments that define how to assemble a Docker image. Each 
        instruction creates one layer in the final image.
      
      - Analogy: Think of a Dockerfile as a recipe in a cookbook. Each line is a cooking step (e.g., “add eggs,” “mix flour”) that builds toward 
        the final dish (the image).


# 03. Core Dockerfile Instructions
      
      a. FROM
         - Specifies the base image on which you build.	
         - Example: FROM node:18-alpine
      
      b. WORKDIR
         - Sets the working directory inside the image (like cd).	
         - Example: WORKDIR /app
      
      c. COPY
         - Copies files from your local machine into the image.	
         - Example: COPY package.json .
      
      d. RUN
         - Executes a command (e.g., install packages), adding its results as a new image layer.	
         - Example: RUN npm install --production
      
      e. ENV
         - Defines an environment variable available at build and runtime.	
         - Example: ENV NODE_ENV=production
      
      f. EXPOSE
         - Documents the network port the container listens on (does not actually publish it).	
         - Example: EXPOSE 3000
      
      g. CMD
         - Specifies the default command when the container starts (only one CMD per Dockerfile).	
         - Example: CMD ["node", "index.js"]
      
      h. ENTRYPOINT
         - Configures a fixed command that always runs, optionally with arguments from docker run.	
         - Example: ENTRYPOINT ["npm", "start"]
      
      i. ARG
         - Declares a build-time variable you can set with --build-arg.	
         - Example: ARG APP_VERSION
      
      j. LABEL
         - Adds metadata (key/value pairs) to the image (maintainer, version, etc.).	
         - LABEL maintainer="alice@example.com"


# 04. Best Practices
      a. Use Official Base Images
         - Choose minimal, well-maintained images (e.g., alpine, debian-slim) to reduce size and vulnerability surface.

      b. Order Matters for Caching
         - Copy and install dependencies first (package.json), then copy app code.
         - This leverages Docker’s layer cache so you don’t reinstall packages on every code change.
      
      c. Minimize Layers
         - Combine related RUN commands with && to reduce the number of layers and overall image size.

      d. Avoid Secrets in Dockerfile
         - Never hard-code passwords or API keys. Use ARG for build-time variables (not persisted) or Docker secrets/--env-file at runtime.

      e. Specify Exact Versions
         - Pin package versions in package.json and base image tags (e.g., node:18.14.2-alpine) to ensure reproducible builds.

      f. Use .dockerignore
         - Exclude unnecessary files (e.g., node_modules, .git) to speed up build context upload and keep images clean.

      g. Leverage Multi-Stage Builds
         - Separate build and runtime environments to produce lean final images:

      h. Document Metadata
         - Use LABEL to record contact, version, and source repository—helpful for auditing and maintenance.

      i. Clean Up After Installations
         - Remove package caches and temporary files in the same RUN step to avoid leftover artifacts.





