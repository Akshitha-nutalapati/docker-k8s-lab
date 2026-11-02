# Day 3 — Env vars & volumes

**Goal:** Configure a container without rebuilding (`-e`), and persist / share data with volumes.

## Mental model

| Tool | Purpose |
|------|---------|
| Env var (`-e`) | Pass config into the container at **runtime** |
| Named volume | Docker-managed storage that **survives** container delete |
| Bind mount (`-v $PWD:...`) | Mount a folder from your **Mac** into the container |

```text
Rebuild image  → when CODE changes
Change -e      → when CONFIG changes (no rebuild)
Volume         → when DATA must outlive the container
```

## Part A — Environment variables

App reads `APP_NAME` (see `exercises/day03-env-volumes/app.py`).

```bash
export PATH="$HOME/Applications/Docker.app/Contents/Resources/bin:$PATH"
cd ~/Projects/docker-k8s-lab/exercises/day03-env-volumes

docker build -t flask-hello:v3 .

# default (no APP_NAME) → "demo"
docker run -d --name api -p 5000:5000 flask-hello:v3
curl http://localhost:5000
docker stop api && docker rm api

# custom env — same image, different config
docker run -d --name api -p 5000:5000 -e APP_NAME=akshit flask-hello:v3
curl http://localhost:5000
docker stop api && docker rm api
```

**Why:** One image, many environments (dev/stage/prod) by changing env vars — not by rebuilding.

## Part B — Named volume (Postgres data survives)

```bash
docker volume create pgdata

docker run -d --name db \
  -e POSTGRES_PASSWORD=secret \
  -e POSTGRES_DB=appdb \
  -v pgdata:/var/lib/postgresql/data \
  -p 5432:5432 \
  postgres:16-alpine

# wait a few seconds, then check it's up
docker logs db
```

Delete the **container** but keep the **volume**:

```bash
docker stop db && docker rm db

docker run -d --name db \
  -e POSTGRES_PASSWORD=secret \
  -e POSTGRES_DB=appdb \
  -v pgdata:/var/lib/postgresql/data \
  -p 5432:5432 \
  postgres:16-alpine
```

Data is still there because `pgdata` was reused.

```bash
docker volume ls
docker stop db && docker rm db
# optional — wipe data forever:
# docker volume rm pgdata
```

**Why:** Databases need disks. Containers are disposable; volumes hold the data.

## Part C — Bind mount (see your Mac files inside a container)

```bash
cd ~/Projects/docker-k8s-lab
docker run --rm -v "$PWD":/data alpine ls /data
```

You should see `README.md`, `docs`, `exercises`, etc.

**Why:** Useful for live code, config files, or sharing a project folder with a container.

## Day 3 complete when

- [x] Same image, different `-e APP_NAME`, different JSON response
- [x] Created `pgdata`, ran Postgres, removed container, started again with same volume
- [x] Bind-mounted the lab folder and listed files with Alpine
- [x] Can explain: env = config, volume = data, rebuild = code

## Push when done

```bash
cd ~/Projects/docker-k8s-lab
git add .
git commit -m "Add Docker Day 3 env vars and volumes"
git push
```
