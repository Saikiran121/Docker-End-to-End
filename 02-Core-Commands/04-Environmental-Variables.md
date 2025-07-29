# 01. Using Environment Variables with Docker Containers: MAIN TAKEWAY:
      - Environmental Variables let you pass configuration values like API keys, ports, or feature flags into containers at, without 
        baking them into images.
      
      - These keep images generic and secure, and lets you customize container behavior per environment.


# 02. What Are Environment Variables?
      - An environment variable is a named key–value pair available to processes running inside a container. It holds configuration data (e.g., 
        database URLs, credentials, application settings) that your application reads at startup.
      

# 03. Real-World Analogy:
      - Think of baking cookies: the recipe (Docker image) calls for “sprinkles.” Instead of gluing sprinkles into the recipe itself, you tell the 
        baker at the last moment, “Use rainbow sprinkles” or “Use chocolate chips.” You can reuse the same recipe with different toppings—just like you reuse the same image with different environment variables.


# 04. Why Use Environment Variables?
      a. Separation of code and configuration: Keep sensitive or environment-specific data (like passwords or API endpoints) out of your image.

      b. Flexibility: Use one image across development, testing, and production by supplying different values at runtime.

      c. Security: Avoid hard-coding secrets into images or source code; manage them externally.


# 05. Passing Environment Variables
      a. Using -e or --env Flags
         - When you run a container, pass variables with -e NAME=value:

         - Each -e adds one variable.

         - In this example, the container’s process can read DATABASE_URL and LOG_LEVEL from its environment.
           
           docker run -d \
                -e DATABASE_URL="postgres://user:pass@db:5432/mydb" \
                -e LOG_LEVEL=debug \
                --name my-app \
                my-app-image:latest
      

      b. Using an Environment File
         - Store variables in a plain-text file (e.g., .env):
                
                DATABASE_URL=postgres://user:pass@db:5432/mydb
                LOG_LEVEL=debug
                FEATURE_X_ENABLED=true
         
         - Run with --env-file:
         - Docker reads every KEY=value line.
         - Ignore lines starting with # (comments).

                docker run -d \
                    --env-file .env \
                    --name my-app  \
                    my-app-image:latest


# 06. Accessing Variables Inside the Container
      - Inside your application (in any language), read the variable:
        
        a. Bash Shell Example
           echo "Database is at $DATABASE_URL"

        b. Node.js example:
           const dbUrl = process.env.DATABASE_URL;
           console.log(`Connecting to ${dbUrl}`);
        
        c. Python example:
           import os
           db_url = os.getenv("DATABASE_URL")
           print(f"DB URL is {db_url}")


# 07. Best Practices
      a. Keep Secrets out of version control
         - Don’t commit .env files containing passwords.
         - Use Docker secrets or external secret managers for production.
      
      b. Use clear variable names:
         - Prefix with your app name or module, e.g., MYAPP_DB_URL.
      
      c. Provide defaults in code:
         - Fallback to sensible defaults if variables are missing.
      
      d. Validate at startup:
         - Exit early with an error if required variables aren’t set.


# 08. Step-by-Step Example
      
      a. Create an .env file alongside your Dockerfile
         PORT=8080
         GREETING="Hello from Docker"
      
      b. Write a simple app (e.g., app.sh):
         #!/bin/bash
         echo "$GREETING Listening on port $PORT."
         exec some-server --port "$PORT"
      
      c. Build the image:
         docker build -t greeting-app .
      
      d. Run with the env file:
         docker run --env-file .env greeting-app
      
      e. Observe output:
      






