# 01. Volume Drivers
      a. Default Driver: local (uses host filesystem)
      b. Other drivers: nfs, cloud plugins (AWS EFS, Azure File), storage arrays


# 02. Example: NFS volume
      docker volume create \
        --driver local \
        --opt type=nfs \
        --opt o=addr=10.0.0.5,rw \
        --opt device=":/export/data" \
        nfsdata


# 03. Read-Only and Consistency Modes
      docker run -d \
        -v mydata:/data:ro \
        nginx

      - :ro prevents container from writing.
      - :cached, :delegated on Docker Desktop tune performance on macOS/Windows.

