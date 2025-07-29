# 01. What is Networking
      - A computer network moves packets, electornic envelopes of data, between processes. Docker creates namespaces so each container feels like
        it has its own network cards, IP addresses, and routing tables, even though the host holds the real hardware.


# 02. Default Objects Created by Docker
      a. docker0 bridge – A virtual switch that all new containers join if you do nothing

      b. Default networks called bridge, host, and none are visible via docker network ls


# 03. Key CLI Teasers
      a. list all networks
         # docker network ls
      
      b. deep JSON details of default bridge  
         # docker network inspect bridge
      
      c. joins docker0 automatically  
         # docker run -it busybox sh 