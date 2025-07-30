# 01. Named Volumes vs. Bind Mounts
| Feature                   | Named Volume                     | Bind Mount                              |
| ------------------------- | -------------------------------- | ---------------------------------------- |
| Path managed by Docker    | Yes                              | No — uses host paths                     |
| Use case                  | Database storage, logs           | Dev-mode code sharing                    |
| Portability               | High                             | Low — tied to host directory             |
| Permissions               | Docker controls                  | Host filesystem permissions apply        |


# 02. Basic Workflow
      
      a. Create Volume
         # docker volume create pgdata
      
      b. Use Volume
         # docker run -d \
                -- name db \
                -v pgdata:/var/lib/postgresql/data \
                postgres:15
      
      c. Inspect and back up
         # docker run --rm -v pgdata:/from -v $(pwd)/backup:/to alpine \
                 sh -c "cp -a /from/. /to/"