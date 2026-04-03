## The Problem 
```
originService=http://localhost:8000
dial tcp [::1]:8000: connect: connection refused
```
Cloudflared inside the Docker container tries to connect to localhost:8000, but that only points to itself. It needs to connect to the host machine where your app is running.

### Solution 1: Use Host IP Address (Recommended)
1. Get your server's IP address
```bash
hostname -I
# or
ip addr show | grep "inet "
```
2. Update the tunnel route in the dashboard 
    1. Navigate to your tunnel settings: 
        * Tunnel: CCS-AI-INFER-01
    2. Go to the "Published application routes" tab
    3. Edit the route and change the service URL from:
        * http://localhost:8000
        * To:
        * http://YOUR_SERVER_IP:8000
Replace YOUR_SERVER_IP with your actual server IP address.

### Solution 2: Use Docker Network (Alternative)
If you prefer not to expose the IP, you can connect cloudflared to the same Docker network as your app:

```bash
# Stop cloudflared
docker stop cloudflared
# Connect to the same network as your app
docker network connect bridge cloudflared
# Start cloudflared again
docker start cloudflared
# Now update the tunnel route to use the container name
# In dashboard, change originService to: http://countgdpp-server:8000
```

### Verify the Fix
After updating the route:

1. Check the logs again: docker logs -f cloudflared
2. Access your app: https://aicounter.connectionscomputer.com/health
3. Check the tunnel diagnostics: CCS-AI-INFER-01 logs
The logs should show successful connections instead of "connection refused" errors.

----------------------------------------------------------

* executing sh
```sh
docker exec -it countgdpp-server /bin/bash
```
```sh
docker exec -u root -it countgdpp-server /bin/bash
```
``` sh
docker run -it countgdapp-server-gpu /bin/bash
```

* copy file
```sh
docker cp countgd_logging.py countgdpp-server:/app/countgd_logging.py
```

This is a **very common Docker issue**, especially when running ML / inference workloads (model downloads, build caches, large layers). Docker is conservative about deleting things, so storage grows fast unless you clean it intentionally.

Below is a **safe → aggressive cleanup ladder**, plus **ML-specific tips** for inference servers.

---

## 1️⃣ Check what’s actually using disk space (always do this first)

```bash
docker system df
```

Example output:

```
TYPE            TOTAL     ACTIVE    SIZE      RECLAIMABLE
Images          28        4         120GB     95GB (79%)
Containers      14        2         4GB       3GB
Local Volumes   18        6         260GB     210GB
Build Cache     42                  30GB      30GB
```

⚠️ **Volumes are usually the real culprit**, not images.

---

## 2️⃣ Clean **stopped containers** (safe)

```bash
docker container prune
```

✔ Removes containers not running
❌ Does NOT remove images or volumes

---

## 3️⃣ Remove **unused images** (safe)

```bash
docker image prune
```

To remove **all images not used by any container**:

```bash
docker image prune -a
```

⚠️ This will require re-pulling images later.

---

## 4️⃣ Clean **Docker build cache** (VERY important for ML)

Build cache grows fast when rebuilding inference servers.

```bash
docker builder prune
```

Or nuke all build cache:

```bash
docker builder prune --all
```

💡 This is usually **10–50GB reclaimed** on ML machines.

---

## 5️⃣ Remove **unused volumes** (biggest gain, but be careful)

```bash
docker volume prune
```

⚠️ This deletes volumes **not attached to running containers**
✔ Safe if you don’t need old model data, logs, or DBs

To inspect volumes before deleting:

```bash
docker volume ls
docker volume inspect <volume_name>
```

---

## 6️⃣ One-command cleanup (recommended occasionally)

```bash
docker system prune
```

More aggressive:

```bash
docker system prune -a
```

**Max cleanup (images + volumes + cache):**

```bash
docker system prune -a --volumes
```

🚨 **This deletes everything unused**
Only run this if you understand the consequences.

---

## 7️⃣ Find large files inside Docker storage (Linux)

Docker data lives here by default:

```bash
/var/lib/docker
```

Find large directories:

```bash
sudo du -h --max-depth=2 /var/lib/docker | sort -hr | head -20
```

Common space hogs:

* `overlay2/` → image layers
* `volumes/` → model files, caches
* `buildkit/` → build cache

---

## 8️⃣ ML / inference-specific cleanup (VERY relevant for you)

### A. Model caches inside containers

Many frameworks cache models in:

* `/root/.cache`
* `/home/*/.cache`
* `/opt/models`
* `/app/models`

If these are inside a **volume**, they persist forever.

👉 **Solution**
Mount model cache explicitly:

```bash
-v ~/model-cache:/models
```

Then clean manually:

```bash
rm -rf ~/model-cache/*
```

---

### B. HuggingFace / PyTorch cache

Common locations:

```bash
~/.cache/huggingface
~/.cache/torch
```

Clean safely:

```bash
rm -rf ~/.cache/huggingface/*
rm -rf ~/.cache/torch/*
```

Or limit cache size:

```bash
export HF_HOME=/models/hf
export TORCH_HOME=/models/torch
```

---

### C. Logs growing inside containers

Check logs:

```bash
docker ps -q | xargs -I {} docker logs --tail=10 {}
```

Docker log files live in:

```bash
/var/lib/docker/containers/*/*.log
```

Truncate logs safely:

```bash
sudo truncate -s 0 /var/lib/docker/containers/*/*.log
```

---

## 9️⃣ Prevent future disk explosions (strongly recommended)

### Limit log size

Create or edit:

```bash
/etc/docker/daemon.json
```

```json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m",
    "max-file": "3"
  }
}
```

Restart Docker:

```bash
sudo systemctl restart docker
```

---

### Auto-clean build cache (CI style)

```bash
docker builder prune -af --filter until=24h
```

---

## 10️⃣ My recommended cleanup sequence for inference servers

Run this **monthly** (or when disk spikes):

```bash
docker container prune -f
docker image prune -af
docker builder prune -af
docker volume prune -f
```

Then:

```bash
docker system df
```

---

## 🧠 TL;DR (most effective commands)

```bash
docker system df
docker builder prune -af
docker volume prune
docker system prune -a --volumes
```

---
