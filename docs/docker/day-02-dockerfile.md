# Day 2 — Dockerfile (build your own image)

**Goal:** Write a small Flask app, package it with a `Dockerfile`, build an image, run a container from it, change the code, rebuild.

## Mental model

| Day 1 | Day 2 |
|-------|--------|
| Used someone else’s image (`nginx`) | Build **your** image from a Dockerfile |
| `docker pull` + `docker run` | `docker build` + `docker run` |

```text
app.py + requirements.txt + Dockerfile
        ↓  docker build
   Image (flask-hello:v1)
        ↓  docker run
   Container (api) → http://localhost:5000
```

## Files (in `exercises/day02-flask/`)

- `app.py` — Flask API
- `requirements.txt` — Python dependencies
- `Dockerfile` — recipe to build the image

### What each Dockerfile line does

| Instruction | Purpose |
|-------------|---------|
| `FROM python:3.12-slim` | Start from an official Python base image |
| `WORKDIR /app` | Set working directory inside the image |
| `COPY requirements.txt .` | Copy dependency list |
| `RUN pip install ...` | Install packages **while building** the image |
| `COPY app.py .` | Copy your app code |
| `EXPOSE 5000` | Documents the port the app listens on |
| `CMD ["python", "app.py"]` | Default command when the container starts |

## Steps — run these

```bash
# keep Docker Desktop running
export PATH="$HOME/Applications/Docker.app/Contents/Resources/bin:$PATH"

cd ~/Projects/docker-k8s-lab/exercises/day02-flask

docker build -t flask-hello:v1 .
docker run -d --name api -p 5000:5000 flask-hello:v1
curl http://localhost:5000
```

You should see JSON like `{"message":"hello from docker"}`.

### Change code → new image version

1. Edit `app.py` — change the message text.
2. Rebuild and run as `v2`:

```bash
docker stop api && docker rm api
docker build -t flask-hello:v2 .
docker run -d --name api -p 5000:5000 flask-hello:v2
curl http://localhost:5000
```

**Why rebuild?** The image is a snapshot. Changing files on your Mac does **not** change a container that was built earlier — you build a new image (or mount code live, which is Day 3+).

### Cleanup

```bash
docker stop api && docker rm api
```

## Day 2 complete when

- [x] `docker build` succeeds
- [x] `curl http://localhost:5000` returns your message
- [x] You changed the message, rebuilt as `:v2`, and saw the new text
- [x] You can explain: Dockerfile → image → container

## After you’re done

Update this checklist, commit, and push to GitHub:

```bash
cd ~/Projects/docker-k8s-lab
git add exercises/day02-flask docs/docker/day-02-dockerfile.md README.md
git commit -m "Add Docker Day 2 Dockerfile Flask exercise"
git push
```
