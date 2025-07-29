# 01. Data Lost When Container Stops and Restarts

- **Containers are intentionally ephemeral, the moment they stop, the writable layer disappears. In production I have learned that if data matters,
  you must lift it outside the container.**


## My 3-Step Playbook

#### a. Identify What Must Survive
- **First I list every path that holds state: DB files in /var/lib/postgresql/data, user uploads in /app/uploads, logs, etc. Anything that must 
  outlive a redeploy goes on that checklist.**

#### b. Choose the Right Persistence Mechanism
- **Named volumes for services: Volumes are Docker managed and portable: perfect for databases or anything cluster-aware via a plugin driver.**
<br>

- **Bind mounts for dev or host-dependent config: During local debugging I mount ./src so code changes hot-reload, but I never ship that pattern to 
  prod.**
<br>

- **Docker Compose/Kubernetes: Declaratively wire volumes so the whole stack can be torn down and rebuilt without data loss.**

#### c. Verify & Automate
- **I spin up a container, write a sentinel file, restart, and confirm it’s still there. In CI we run the same smoke test before a merge**


## Production War Stories

#### MySQL Wiped Overnight
- **Early in my career we launched MySQL in Swarm without a volume, worked in staging, nuked itself on the first host reboot. We restored from 
  backup and added: docker service create \ --mount type=volume,source=mysql_data,target=/var/lib/mysql ...
  Haven’t lost a byte since**


#### Log Flood on Host but Not in Container
- **A Node API stored logs under /usr/src/app/logs. After a month we saw 0 B inside the container but 40 GB on the host because a colleague had 
  quietly bind-mounted /var/log/api for rotation. Lesson: document every mount in code reviews and monitoring.**


#### K8s StatefulSet Migration
- **During a cluster upgrade we migrated Postgres PVCs across AZs. Because the volume was declared in the StatefulSet, Kubernetes simply detached
  and re-attached it; downtime was minutes instead of hours of dump/restore**


# 02. Best-Practice Cheat Sheet (What I Tell New Hires)

- **Databases → always a named volume or PVC; never rely on the container FS**
- **User assets → bind mount or object storage; enforce backup rotation.**
- **Config → read-only bind mounts or secrets; keeps immutable infrastructure clean.**
- **Backups & Monitoring → snapshot volumes nightly and alert on disk usage.**
- **Disaster drills → rehearse “rm -rf /var/lib/postgresql/data” in staging; prove that restore docs work.**


# 03. Mnemonic to remember: V-SAFE
- V – Volumes for state
- S – Snapshots for backups
- A – Attach at runtime (-v or PVC)
- F – File paths documented
- E – Ephemeral layers are expendable



