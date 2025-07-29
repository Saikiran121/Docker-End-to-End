# 01. Multi-Stage Builds MAIN TAKEWAY
      - A multi-stage build in Docker lets you use one Dockerfile to perform discrete steps like compiling code, assets, and assembling
        a final runtime image, without carrying all build tools into your production image. This results in much smaller, more secure
        containers.


# 02. What Is a Multi-Stage Build?
      - A multi-stage build uses more than one FROM instruction in a single Dockerfile. Each FROM starts a new stage.
      - You name stages with AS <name> and then use COPY --from=<name> to pull only the artifacts you need into the final image.


# 03. Why It Matters:
      - Traditional Dockerfiles bake compiler toolchains, temporary files, and dev libraries into the final image, making it bulky and exposing
        more code to potential attacks. Multi-stage builds let you discard everything except the minimal runtime artifacts.


# 04. Real-World Analogy: Kitchen & Catering
      a. Single-Stage (All-in-One Kitchen):
         - You bring in raw ingredients, chopping boards, heavy cookware, and all utensils to the dining table. 
         - Guests must squeeze around the mess even bulky pots you no longer need.
      
      b. Multi-Stage (Catering Workflow):
         - Prep Kitchen (Build Stage): Chefs chop, cook, and plate dishes in a back-of-house kitchen using all tools.
         - Service Dining Room (Final Stage): Only the plated dishes and minimal serving utensils go to the dining room. The bulky pots and knives 
           stay behind.
      
      - This ensures the dining area stays tidy, focused only on eating, not cooking.


# 05. How It Works: Step-by-Step
      a. Stage 1: Build Environment
         
         FROM golang:1.24 AS builder
         WORKDIR /src
         COPY . .
         RUN go build -o /app/myservice

         - Uses a full Go SDK image to compile your app.
      
      b. Stage 2: Final Runtime
      
         FROM alpine:latest
         COPY --from=builder /app/myservice /usr/local/bin/myservice
         CMD ["myservice"]

         - Starts from a minimal Alpine base and copies only the compiled binary.
      
      c. Build & Run

         docker build -t myservice:1.0 .
         docker run --rm myservice:1.0

         - Final image contains only Alpine plus your binary no Go compiler or source code.


# 06. Key Benefits
      a. Smaller Image Size
         - Excludes build tools and dependencies—only runtime artifacts remain
      
      b. Faster Deployments
         - Less data to transfer over networks, speeding up pulls and starts.
      
      c. Improved Security	
         - Reduces attack surface by discarding compilers, debuggers, and intermediate files.

      d. Better Cache Utilization	
         - Layers from build stages that don’t change (e.g., dependency install) are reused across builds.


# 07. Simple Example: Node.js App

      # Stage 1: Build (install and compile)
      FROM node:18-alpine AS build
      WORKDIR /app
      COPY package*.json ./
      RUN npm ci --only=production
      COPY . .
      RUN npm run build

      # Stage 2: Runtime (serve only dist/)
      FROM nginx:alpine
      COPY --from=build /app/dist /usr/share/nginx/html


# 08. When to Use Multi-Stage Builds
      a. Compiled Languages: Go, Rust, Java—compile in one stage, copy only binaries.
      b. Front-End Frameworks: React, Angular—build and minify assets, then serve static files.
      c. Polyglot Projects: Combine different build tools without bundling them into production images.
      d. CI/CD Pipelines: Speed up builds by caching unchanged stages and reducing image size for deployments.


# 09. Best Practices
      a. Name Your Stages:
         FROM maven:3.8-openjdk AS build
         FROM tomcat:9-jdk11 AS runtime

      b. Order by Change Frequency:
         - Copy dependencies and install first—rarely changes.
         - Copy source code and build later—changes often.

      c. Combine RUN Commands: minimize layers in each stage.

      d. Use Minimal Base Images: e.g., Alpine or scratch for final stage.

      e. Test Intermediate Stages: build with --target to diagnose before finalizing.
         docker build --target build -t myapp:build .
