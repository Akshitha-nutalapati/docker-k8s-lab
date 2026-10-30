# Day 1 — Images, containers, ports, logs, exec

**Goal:** Pull an image, run a container, inspect it, stop/remove it, run it again.

## Mental model

| Concept | Meaning |
|---------|---------|
| Image | Read-only template (recipe), e.g. `nginx:alpine` |
| Container | One running (or stopped) instance of an image |
| Port map | Door from your Mac into the container (`8080:80`) |

```text
Image  (nginx:alpine)  →  Container (web)  →  Browser hits localhost:8080
```

## What I did

### 1. Verify Docker works

```bash
docker version
docker run hello-world
```

**Why:** Confirms the Docker client can talk to the Docker Desktop engine. `hello-world` is a smoke test image.

### 2. Pull nginx

```bash
docker pull nginx:alpine
```

**Why:** Downloads the image locally. `alpine` means a small Linux base. Pull once; reuse many times.

### 3. Run nginx in the background

```bash
docker run -d --name web -p 8080:80 nginx:alpine
```

| Flag | Purpose |
|------|---------|
| `-d` | Detached (background) |
| `--name web` | Easy name instead of a random ID |
| `-p 8080:80` | Mac port 8080 → container port 80 |

**Why:** Creates and starts a container. Nginx listens on 80 *inside* the container; Docker forwards traffic from your Mac’s 8080.

Opened http://localhost:8080 and saw the nginx welcome page.

### 4. Inspect

```bash
docker ps          # list running containers
docker logs web    # see nginx startup + HTTP request logs
```

**Why:** `ps` shows what’s alive. `logs` is the main way to debug without entering the container.

### 5. Enter the container

```bash
docker exec -it web sh
```

**Why:** Opens a shell *inside* the container (like a tiny isolated machine).

**Lesson:** Inside the container (`/ #`) there is no `docker` command. `docker` runs on the **host** (Mac). Type `exit` to leave, then run Docker commands again.

### 6. Stop and remove

```bash
exit
docker stop web && docker rm web
```

**Why:**
- `stop` — end the running process
- `rm` — delete that container instance

The **image** stays on disk. Containers are disposable; images are reusable.

### 7. Start again

```bash
docker run -d --name web -p 8080:80 nginx:alpine
docker ps
curl http://localhost:8080
```

**Why:** Same image → new container. Normal cycle: create → use → destroy → create again.

## Commands cheat sheet

```bash
docker pull <image>
docker run -d --name <name> -p <host>:<container> <image>
docker ps
docker logs <name>
docker exec -it <name> sh
docker stop <name>
docker rm <name>
```

## Day 1 complete when

- [x] `hello-world` works
- [x] nginx runs on http://localhost:8080
- [x] Used `ps`, `logs`, `exec`
- [x] Stopped/removed and started again
