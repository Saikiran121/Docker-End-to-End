# 01. Port Not Accessible on Localhost Despite Port Mapping

### Professional Interview Response
- **This is actually one of the most common Docker Issues I have encountered in production. Despite having correct port mapping, the application
  isn't accessible from localhost.**


### My Immediate Diagnostic Process
- **First thing I do is verify the application's binding configuration. In my experince 90% of these issues stem from the application listening
  on 127.0.0.1 instead of 0.0.0.0 inside the container. I quickly check this with:**
  ```
  docker exec -it <container> netstat -tlnp
  ```
  **If I see port bound to 127.0.0.1, I know immediately thats the problem, Docker cant forward traffic to a localhost-only listener inside the
  container.**


### Real-World Examples from My Experience
- **I have seen this across different tech stacks. For instance, we had a Node.js microservice where the developer used:**
  ```
  app.listen(3000, 'localhost')  // Problem
  ```
  **The fix was simple, change it to bind to all interfaces:**
  ```
  app.listen(3000, '0.0.0.0')  // Solution
  ```
<br>

- **Similarly, with Flask applications, the default flask run command binds to localhost only. I always ensure our team uses flask run --host=0.0.
  0.0 or configures it properly in the application code.**


# 02. Production War Stories

- **One memorable incident was during a critical deployment. Our spring boot application wasn't accessible despite correct Kubernetes service
  configuration. The issue was in application.yaml**
  ```
  server:
    address: 127.0.0.1  # This was the culprit
  ```
  **We had to do a hotfix removing that line, which defaults to binding all interfaces. Since then I have made it a standard practice to review 
    binding configurations during code reviews.**


# 03. My Troubleshooting Methodology
- **"I follow a structured approach:**
  
  **a. Verify port mapping:** docker ps to confirm the mapping looks correct
  **b. Check application binding:** netstat inside the container
  **c. Test container-to-container:** Sometimes the issue is network related
  **d. Validate with curl inside container:** docker exec -it <container> curl localhost:port
  **e. Check firewall/security groups:** n cloud environments, this can be the culprit


# 04. Prevention and Best Practices
- **Based on these experiences, I've established team standards:**
  
  **a. Code review checklist** includes verifying 0.0.0.0 binding for containerized apps
  **b. Docker templates** with proper configuration examples
  **c. Local development setup** that mirrors production container behavior
  **d. Health check endpoints** that validate external accessibility