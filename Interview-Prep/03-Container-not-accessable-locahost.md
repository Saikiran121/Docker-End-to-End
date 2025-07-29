# 01. Port Not Accessible on Localhost Despite Port Mapping: Troubleshooting Guide
- **When your Docker Container port mapping seems correct but localhost still isn't accessible, it's usually because your application inside 
  the container is listening on the wrong interface. Think of it like having a phone that only accepts internal calls when you need it to 
  answer the external ones too.**


## The Root Cause: Interface Binding Problem
-  **Memory Hook: "App Binds Inside, Docker Forwards Outside", your app must listen on the right interface for Docker's port forwarding to work.**
<br>
- **The Most Common issue is that your application inside the container is bound to 127.0.0.1 (localhost) instead of 0.0.0.0 (all interfaces).**


## Understanding the Problem

#### What Happens When Port Mapping "Fails"
- **This looks correct but may not work
  docker run -p 8080:8080 myapp
  Port mapping: Host port 8080 → Container port 8080**
<br>
- **The issue: If your app inside the container only listens on 127.0.0.1:8080, Docker can't forward traffic to it because 127.0.0.1 inside the 
  container refers to container's localhost, not your host machine's localhost**


## Quick Diagnostic Steps

#### a. Check What Your App Is Listening On
```
  # Inside the container, check what ports are listening
    docker exec -it <container_name> netstat -tlnp
  # Look for your port - is it bound to 127.0.0.1 or 0.0.0.0?
    Problem: 127.0.0.1:8080
    Correct: 0.0.0.0:8080
```

#### b. Verify Docker Port Mapping
```
  # Check if Docker mapped the port correctly
    docker ps
  # Should show: 0.0.0.0:8080->8080/tcp

  # Or detailed inspection
    docker port <container_name>
```


# 02. Common Scenarios and Fixes

#### Scenario 1: Node.js/Express Application
#### Problem: App defaults to localhost binding
```
  # This won't work with Docker port mapping
  app.listen(3000, 'localhost')
  app.listen(3000, '127.0.0.1')

  # This will work
  app.listen(3000, '0.0.0.0')

  # Or simply omit the host parameter
  app.listen(3000)
```

#### Dockerfile Example:
```
  FROM node:14
  WORKDIR /app
  COPY . .
  RUN npm install
  EXPOSE 3000
  # Make sure your app.js uses 0.0.0.0
  CMD ["node", "app.js"]
```
-------------------------------------------------------------

#### Scenario 2: Flask Application
#### Problem: Flask defaults to 127.0.0.1
```
  # Won't work with Docker
  app.run(host='127.0.0.1', port=5000)
  
  # Will work with Docker
  app.run(host='0.0.0.0', port=5000)
```

#### Or use Flask CLI:
```
  # Default Flask command
  flask run

  # Specify host for Docker
  flask run --host=0.0.0.0
```

-------------------------------------------------------------------

#### Scenario 3: Java/Spring Boot Application
#### Problem: Server configuration binds to localhost
```
  # application.yml - won't work
  server:
    address: 127.0.0.1
    port: 8080
  
  # application.yml - will work
  server:
    address: 0.0.0.0
    port: 8080

  # Or simply omit the address property
```

-------------------------------------------------------------------------------

#### Scenario 4: Go Application
#### Problem: HTTP server bound to localhost
```
  # Won't work with Docker
  srv := &http.Server{
      Handler: myHandler,
      Addr:    "127.0.0.1:8080",
  }
  
  3 Will work with Docker
  srv := &http.Server{
      Handler: myHandler,
      Addr:    ":8080",  // Binds to all interfaces
  }
```

# 03. Memory Aid: The "BIND" Method
      a. B - Bind to 0.0.0.0, not 127.0.0.1
      b. I - Interface must accept external connections
      c. N - Network traffic flows: Host → Docker → Container
      d. D - Docker forwards only if app listens on all interfaces

