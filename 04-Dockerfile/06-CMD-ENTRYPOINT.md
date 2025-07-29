# 01. Understanding CMD and ENTRYPOINT: What Commands to Add and Why Main Takeaway:
      - CMD and ENTRYPOINT tell Docker what command to run when your container starts. Think of them as the "start button" for your applications.
      - The command you add depends on what your application is a web server, a database, a Python script etc.


# 02. What CMD and ENTRYPOINT Do
      a. CMD
         - The default command that runs when your container starts (can be overriden)
      
      b. ENTRYPOINT 
         - A fixed command that always runs (cannot be easily overriden)


# 03. Real-World Analogy:
      - Imagine your Docker container is like a specialized appliance:
        a. CMD is like TV's default channel, it starts on channel 1, but you can change it with the remote
        b. ENTRYPOINT is like a coffee makers "brew" function, it always makes coffee, but you might add different parameters(water amount, 
           strength)


# 04. How to Decide What Command to Add
      - The command you use depends on what your application does:
        
        Application Type                            	Common CMD/ENTRYPOINT Examples
        Web Server                                      CMD ["nginx", "-g", "daemon off;"]

        Node.js App                                      CMD ["node", "server.js"]

        Python Script                                   CMD ["python", "app.py"]

        Java Application                                CMD ["java", "-jar", "app.jar"]

        Database                                        CMD ["mysqld"] or CMD ["postgres"]

        Static Files                                    CMD ["nginx", "-g", "daemon off;"]


# 05. Step-by-Step: Finding the Right Command
      a. Step 1: Identify Your Application Type
         - Ask yourself: "What would I normally type in the terminal to run my app?"

         # If you normally run:
         python app.py
         # Then use: CMD ["python", "app.py"]
         
         # If you normally run:
         node server.js
         # Then use: CMD ["node", "server.js"]
         
         # If you normally run:
         java -jar myapp.jar
         # Then use: CMD ["java", "-jar", "myapp.jar"]
      
      b. Step 2: Choose the Format
         - Docker supports two formats:
           Shell Form (simple)
           CMD python app.py
         
         - Exec Form (recommended):
           CMD ["python", "app.py"]

# 06. For NGINX Web Server
      FROM nginx:alpine
      COPY files/index.html /usr/share/nginx/html/
      COPY files/config/nginx.conf /etc/nginx/nginx.conf
      
      # This starts the NGINX web server
      CMD ["nginx", "-g", "daemon off;"]
      
      - Why this command?
        a. nginx - starts the NGINX web server
        b. -g "daemon off;" - keeps NGINX running in the foreground (required for Docker)


# 07. For Python Web Application:
      FROM python:3.11-slim
      WORKDIR /app
      COPY requirements.txt .
      RUN pip install -r requirements.txt
      COPY app.py .
      
      # This starts your Python web app
      CMD ["python", "app.py"]


# 08. For Node.js Application:
      
      FROM node:18-alpine
      WORKDIR /app
      COPY package*.json ./
      RUN npm install
      COPY . .
      
      # This starts your Node.js server
      CMD ["node", "server.js"]


# 09. CMD vs ENTRYPOINT: When to Use Which
      a. Use CMD When:
         - You want users to be able to override the command easily
         - You're running a simple application with optional parameters

         CMD ["python", "app.py"]
         
         # User can override:
         # docker run myimage python debug.py
      
      b. Use ENTRYPOINT When:
         - You want to ensure a specific program always runs
         - You want to accept parameters but always run the same base command

         ENTRYPOINT ["python", "app.py"]
         
         # User can only add parameters:
         # docker run myimage --debug --port 8080
         # This runs: python app.py --debug --port 8080
      
      c. Use Both Together:
         ENTRYPOINT ["python", "app.py"]
         CMD ["--port", "8000"]
         
         # Default: python app.py --port 8000
         # Override CMD: docker run myimage --port 9000
         # Result: python app.py --port 9000


# 10. Practical Examples for Different Scenarios
      a. Scenario 1: Simple Static Website
         FROM nginx:alpine
         COPY website/ /usr/share/nginx/html/
         
         # Start NGINX to serve static files
         CMD ["nginx", "-g", "daemon off;"]
      
      b. Scenario 2: Python Flask API
         FROM python:3.11-slim
         WORKDIR /app
         COPY requirements.txt .
         RUN pip install -r requirements.txt
         COPY app.py .
         EXPOSE 5000
         
         # Start the Flask development server
         CMD ["python", "app.py"]
      
      c. Scenario 3: Database Container
         FROM mysql:8.0
         ENV MYSQL_ROOT_PASSWORD=mypassword
         ENV MYSQL_DATABASE=myapp
         
         # MySQL starts automatically - CMD is already defined in base image
         # But if you wanted to customize:
         # CMD ["mysqld", "--character-set-server=utf8mb4"]


# 11. How to Test Your CMD/ENTRYPOINT
      a. Step 1: Build Your Image
         docker build -t mytest:1.0 .
      
      b. Step 2: Run and Check
         # Run normally
         docker run -d --name test mytest:1.0
         
         # Check if it's running
         docker ps
         
         # Check logs to see if command worked
         docker logs test
         
         # If something's wrong, run interactively to debug
         docker run -it mytest:1.0 /bin/sh


# 12. Common Mistakes and Solutions
      a. Mistake 1: Command Exits Immediately
         # Wrong - exits immediately
         CMD ["echo", "Hello World"]
         
         # Better - runs a persistent service
         CMD ["python", "-m", "http.server", "8000"]
      
      b. Mistake 2: Not Using Full Paths
         # Might not work if PATH isn't set correctly
         CMD ["node", "app.js"]
         
         # More reliable with full path
         CMD ["/usr/local/bin/node", "/app/app.js"]
      
      c. Mistake 3: Forgetting daemon off for Services
         # Wrong - NGINX exits immediately
         CMD ["nginx"]
         
         # Correct - keeps NGINX running
         CMD ["nginx", "-g", "daemon off;"]
