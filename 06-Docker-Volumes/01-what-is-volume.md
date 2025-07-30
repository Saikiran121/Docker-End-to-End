# 01. What Is a Volume?
      - A volume is a specially managed directory on the Docker host or remote driver, mouted into one or more containers.
      - Unlike a containers ephemeral filesystem, volume data persists when containers stops or are removed.


# 02. Why Use Volumes?
      a. Persistence: Data outlives containers.
      b. Isolation: Container layers remain read-only except for volume paths.
      c. Performance: Volumes use native host filesystem drivers for fast I/O.
      d. Portability: Docker can export/import volumes; named volumes decouple filesystem paths.


# 03. Quick Commands
      a. create a named volume
         # docker volume create mydata
      
      b. mount volume at /data
         # docker run -d -v mydata:/data nginx
      
      c. list volumes
         # docker volume ls
      
      d. view details
         # docker volume inspect mydata
      
      e. delete unused volume
         # docker volume rm mydata