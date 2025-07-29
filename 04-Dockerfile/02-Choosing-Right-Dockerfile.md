# 01. Choosing the Right Dockerfile Instructions When You’re Starting from Zero Main Takeway:
      - Even if you have never written a Dockerfile before, you can select the appropriate instructions by following a decision tree
        determine your base environment, identify what files and commands your app needs, then choose the Dockerfile keywords that
        correspond to those needs.


# 02. Identify Your Starting Point: “FROM”
      - Every Dockerfile begins with the BASE IMAGE, Ask Yourself:
        a. What language or runtime does my app require? (e.g., Node.js, Python, Java)
        b. Do I need a minimal footprint or full-featured OS?
      
      - Real-world analogy: choosing your kitchen
        a. Want a minimal “food truck” (Alpine)? → FROM alpine:latest
        b. Need full “restaurant kitchen” (Ubuntu)? → FROM ubuntu:24.04
        c. Specific language runtime? → FROM node:20-alpine or FROM python:3.11-slim


# 03. Set Where Your Work Happens: “WORKDIR”
      - Think: “Which folder will hold my source code inside the container?”

      - Analogy: designating your workbench in the kitchen.


# 04. Bring in Your Application Files: “COPY” or “ADD”
      - COPY for simple file/folder transfers.

      - ADD when you need extra features (auto-unpacking tar files or fetching remote URLs) but usually stick to COPY for clarity and speed.

      - Analogy: moving ingredients onto your prep table.


# 05. Install Dependencies: “RUN”
      - Whenever you need to execute a command at build-time (install packages, build assets), use RUN.

      - In a Linux-based image:
        # RUN apt-get update && apt-get install -y curl
        # RUN npm install
      
      - In Windows-based images, you might use PowerShell commands.

      - Analogy: cooking or prepping ingredients before service.


# 06. Configure Environment Variables: “ENV”
      - When your application needs configuration ports, connection strings, feature flags set them as environment variables so your Dockerfile 
        remains generic.
      
      - Analogy: placing labeled containers of spices at known spots.

      - ENV NODE_ENV=production  
        ENV PORT=3000


# 07. Expose Ports: “EXPOSE”
      - Declare which network port your app listens on. This doesn’t publish the port, but documents it for users and tools.
      
      - Analogy: pointing out which door customers will enter.
        EXPOSE 80


# 08. Define the Startup Command: “CMD” vs. “ENTRYPOINT”
      - Use CMD to specify default arguments that can be overridden at runtime.

      - Use ENTRYPOINT to set an unchangeable “entry” command, with optional additional default arguments.

      - Analogy: choosing what dish is served when the kitchen opens.

      - Typical web app that can run different commands
        CMD ["npm", "start"]
      
      - Fixed entry with extra flags
        ENTRYPOINT ["python", "app.py"]


# 09. Quick-Start Decision Flow
      a. What runtime? → FROM

      b. Where to work? → WORKDIR

      c. Which files? → COPY

      d. Need build steps? → RUN

      e. Any config? → ENV / ARG

      f. Which port? → EXPOSE

      g. How to start? → CMD / ENTRYPOINT




