

1. **`FROM`** -> setting up the base image. 
	ex : setting up the node, python or even entire lighteweight VMs. 
2. **`RUN`** -> Execute a command during the build process. Used to install software, set things up, etc. 
	ex :  eqvivalent of uv add, or pip install or pnpm install...
3. **`COPY`** -> Copy files from your local machine to into the image. 
	ex: COPY <A> <B> -> copies everything from B to A. 
4.  **`WORKDIR`** -> sets up the working directory inside of the image. similar to doing the cd command .
		1. After this all subsequent commands run from the /app. 
5.  ENV <k=v>         -> setting up the enviorment variables
6. EXPOSE <PORT>  -> Documents to which port the application is listening 
7. **`CMD`** — The default command to run when a container _starts_. Only one CMD per Dockerfile. 




Usuall workflow in the dockerization ? 
SIMPLE : 
1. make dockerfile 
2. docker build -t <app_name>                                  -> build an image from the dockerfile.
3. docker run -p <custom_port> my-                   -> Run docker image from a custom port.


DETAIL : 
--------------------------------------------------------------------------------
## Typical Docker Integration Workflow

Here's how it goes in practice, from dev to production.

---

### Phase 1 — Local Development

You write your app normally, then "dockerize" it:

1. Write your `Dockerfile`
2. Write a `docker-compose.yml` if your app needs multiple services (a database, a cache, etc.)
3. Build and run locally, verify everything works inside the container

**docker-compose** is huge here. Instead of running 3 separate `docker run` commands with long flags, you describe everything in one file:

```yaml
# docker-compose.yml
services:
  app:
    build: .           # build from local Dockerfile
    ports:
      - "5000:5000"
    environment:
      - DATABASE_URL=postgres://user:pass@db:5432/mydb
    depends_on:
      - db

  db:
    image: postgres:15        # pull official image, no Dockerfile needed
    environment:
      - POSTGRES_PASSWORD=pass
      - POSTGRES_USER=user
      - POSTGRES_DB=mydb
    volumes:
      - pgdata:/var/lib/postgresql/data   # persist DB data

volumes:
  pgdata:
```

Then you just run `docker compose up` and your entire stack — app + database — comes up together.

---

### Phase 2 — Push Image to a Registry

A **registry** is like GitHub but for Docker images. Once you build your image, you push it there so servers can pull it.

```bash
# Tag your image
docker tag my-app yourusername/my-app:v1.0

# Push to Docker Hub (or any registry)
docker push yourusername/my-app:v1.0
```

Popular registries: **Docker Hub**, **AWS ECR**, **GitHub Container Registry**, **Google Artifact Registry**.

---

### Phase 3 — CI/CD Pipeline

This is where it gets automated. You connect your Git repo to a CI system (GitHub Actions, GitLab CI, etc.) so that every time you push code:

```
push code → CI runs tests → builds Docker image → pushes to registry → deploys
```

A simple GitHub Actions example:

```yaml
- name: Build and push image
  run: |
    docker build -t myapp:${{ github.sha }} .
    docker push myapp:${{ github.sha }}
```

The image tag is often the **git commit hash** so you can always trace exactly what code is running.

---

### Phase 4 — Production Deployment

Your server/cloud pulls the image from the registry and runs it. How this looks depends on your scale:

|Scale|Tool|
|---|---|
|Single server|`docker run` or `docker compose` on the server|
|Multiple servers|**Docker Swarm**|
|Large scale / cloud-native|**Kubernetes (K8s)**|

Most small-to-mid apps just use `docker compose` on a VPS (like a DigitalOcean droplet). Kubernetes is powerful but complex — you only need it when you're scaling to many machines.

---

### The Full Picture

```
Your Code
    │
    ▼
Dockerfile ──► docker build ──► Image (local)
                                    │
                                    ▼
                              docker push ──► Registry (Docker Hub / ECR)
                                                    │
                                    ┌───────────────┘
                                    ▼
                             Production Server
                           docker pull + docker run
                                    │
                                    ▼
                            Running Container 🚀
```

---

### A Typical Day-to-Day Dev Loop

```bash
# Make code changes
docker compose up --build    # rebuild and restart

# Something broken? Check logs
docker compose logs app

# Shell into a running container to debug
docker exec -it <container_name> bash

# Tear everything down
docker compose down
```
---

The most important thing to internalize: **the image is your deployable artifact**. You build it once, push it, and that exact same thing runs everywhere — your machine, your teammate's machine, staging, production. No more "works on my machine" problems.

Want me to go deeper on docker-compose, CI/CD pipelines, or how Kubernetes fits in? 

