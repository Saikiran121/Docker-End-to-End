# 01. Backup
      docker run --rm \
        -v mydata:/data \
        -v $(pwd):/backup \
        alpine \
        tar czf /backup/data.tgz -C /data .


# 02. Restore
      docker run --rm \
        -v mydata:/data \
        -v $(pwd):/backup \
        alpine \
        tar xzf /backup/data.tgz -C /data


# 03. Volume Pruning
      - docker volume prune   # deletes all unused volumes
