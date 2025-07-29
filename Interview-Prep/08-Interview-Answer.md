# 01. Code Changes Not Reflected After Docker Image Rebuild
- **This is actually one of the most frustrating issues developers face with Docker, and I have seen it cause significant delays in both 
  development and production deployments. Let me walk you through my systematic approach based on real incidents I've handled**


#### My Immediate Diagnostic Process
- **First thing I do is verify the fundamentals, am I actually running the new Image? in my experience 70% of these issues come down to 
  Docker's layer caching or running stable containers. I immediately run:**
  ```
  docker images --format "table {{.Repository}}\t{{.Tag}}\t{{.CreatedAt}}"
  docker ps -a --filter ancestor=myapp
  ```
  **This tells me if the image was actually rebuilt recently and what containers are using which image versions.**


# 02. Production War Stories

- **Let me share a critical incident from last year. We had a security patch that needed to go out immediately updated the code, rebuilt the image,
  deployed to staging, and the vulnerability scanner still flagged the old code. Turns out Docker was caching the layer where we copied the source
  code.**

- **The fix was immediate: docker build --no-cache, but the lession was deeper. We realized our Dockerfile was poorly structured:**
  ```
  # Old problematic version
  FROM node:14
  WORKDIR /app
  COPY . .                    # Everything cached together
  RUN npm install
  ```
  **We restructured it to**
  ```
  # Optimized version
  FROM node:14
  WORKDIR /app
  COPY package*.json ./       # Dependencies cached separately
  RUN npm install
  COPY . .                    # Only source code invalidates this layer
  ```

- **This reduced our rebuild time from 5 minutes to 30 seconds and eliminate the caching confusions.**

--------------------------------------------------------------------------------------------------------

#### Another Scenario

- **We had a microservices deployment where a developer was pulling their hair out code changes weren't appearing in Kubernetes.
  After investigation, we found they were building myapp:latest but the K8s deployment was still pulling an old latest from the registry.**

- **The solution was implementing proper tagging strategy:**
  
  ```
  # Instead of always using latest
  docker build -t myapp:$(git rev-parse --short HEAD) .
  docker tag myapp:$(git rev-parse --short HEAD) myapp:latest
  ```

- **Now we have traceability and can easily identify which exact commit is running in production**

----------------------------------------------------------------------------------------------------------------


# 03. My Standard Troubleshooting Playbook

#### When this happens, I follow a systematic approach:
**a. Verify the build actually happened: Check image timestamps and layer history
    b. Confirm container refresh: Stop old containers, not just rebuild images
    c. Clear Docker cache if needed: docker builder prune for stubborn cases
    d. Use explicit tags: Never rely solely on 'latest' in production environments"**


# 04. Advanced Production Considerations

#### In enterprise environments, I've dealt with additional complexities:
**a. Multi-stage builds: Developers building wrong target stage
  b. Registry caching: Corporate registries caching old layers
  c. Build context issues: Wrong files being sent to Docker daemon due to .dockerignore problems
  d. Orchestrator caching: Kubernetes ImagePullPolicy set to 'IfNotPresent' instead of 'Always'"**


# 05. Prevention Strategies I've Implemented

#### Based on these experiences, I've established team practices:
**a. Automated rebuild scripts that handle the full lifecycle: stop → remove → build → run
  b. CI/CD pipelines that use unique tags for every build
  c. Development docker-compose with volume mounts for instant feedback
  d. Build verification steps that actually test the changes are present"**


# 06. Quick Decision Framework

#### When I encounter this issue, I can categorize it quickly:
**a. Cache issues (most common): Force clean build
  b. Container lifecycle: Old containers still running
  c. Registry problems: Push/pull synchronization
  d. Dockerfile optimization: Poor layer structure causing unnecessary caching"**

