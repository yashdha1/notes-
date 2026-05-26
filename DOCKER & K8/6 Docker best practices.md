
Great call — solid Docker fundamentals will make Kubernetes much easier later. Here's what's important to not skip:

---

## 1. Multi-Stage Builds

One of the most important Dockerfile patterns. The problem it solves: your build environment (with compilers, dev tools, etc.) shouldn't ship to production — it bloats the image and creates security risks. Build with one image and run the container with much lighter image for better perfoemance in the production. 

```dockerfile
# Stage 1 — Build
FROM node:20 AS builder
WORKDIR /app
COPY package*.json .
RUN npm install
COPY . .
RUN npm run build        # compiles your app

# Stage 2 — Production
FROM node:20-alpine      # much smaller base image
WORKDIR /app
COPY --from=builder /app/dist ./dist    # only copy the output # dist folder has all the output in base js of whatever is in the node project. 
COPY package*.json .
RUN npm install --only=production
CMD ["node", "dist/index.js"]
```

The final image only contains stage 2. The builder stage is thrown away. Image size can go from 1GB → 100MB easily. Multistage builds lets you reduce the size of final images in the production enviorment. In the final output send only the files needed to run the application in the most bare metal way. 
#### What happens to previous stages in a multistage build
When you do a multistage build like this:

```
FROM node:20 AS builder 
# building dependencies 
.
.
.
FROM node:20-alpine  
COPY --from=builder /app/dist ./dist
```


- Docker **creates layers for each stage** (`builder` layers, `final` layers).
- By default, all layers **exist in the local Docker storage** until they are explicitly removed.
- The final image **doesn’t include the builder layers**, but **those layers still exist in your local cache** unless cleaned up. 
- Main diffrence is in the presentation of the final layer. the previous layers still perssist and the cache is somewhere in the memory. 
So, multistage builds **don’t automatically remove previous stages** from disk—they just aren’t included in the final image.

---
 
Remove all dangling layers, unused images, stopped containers  
```
docker system prune -a
```

This will remove:

- Intermediate layers from old builds not referenced by any image
- Dangling images ->  ==image layers that are no longer associated with any tagged, functional, or running containers==. They are created when rebuilding images with the same tag or removing containers, often wasting significant disk space.
- Stopped containers

### Practical takeaway

- **Final image is small:** Because the final stage only copies what you explicitly select.
- **Builder layers may still exist locally:** They are cached unless you clean them.
- **GC doesn’t happen automatically** on every build—you have to run `docker system prune` (or equivalent in CI/CD pipelines).

---

## 2. Image Size & Base Image Choice

The base image you pick matters a lot.
Always prefer `-slim` or `-alpine` variants in production. Smaller images mean faster pulls, less attack surface, cheaper storage.

Ensure that everything else has a platform to run and continue to live on. t ensures consistency across environments, enables efficient image layering (smaller size), and provides a secured, curated starting point, saving time compared to building an environment from scratch. 

---

## 3. Layer Caching — The Right Way

Order of instructions in your Dockerfile directly affects build speed. Docker invalidates cache from the point where something changed downward.

**Bad** — any code change invalidates the dependency install:

```dockerfile
COPY . .                  # copies everything including your code
RUN npm install           # re-runs every time ANY file changes
```

**Good** — dependencies only reinstall when package.json changes:

```dockerfile
COPY package*.json .      # only copy dependency files first
RUN npm install           # cached unless package.json changed
COPY . .                  # copy code last
```

The rule: **put things that change least at the top, things that change most at the bottom.**

---

## 4. .dockerignore

Without this, `COPY . .` copies everything — node_modules, .git, local configs, secrets. Always have one.

```
node_modules
.git
.env
__pycache__
*.pyc
.DS_Store
dist
coverage
```

This also speeds up builds because Docker doesn't send unnecessary files to the build context.

---

## 5. Environment Variables & Secrets

**ENV** is for non-sensitive config:

```dockerfile
ENV NODE_ENV=production
ENV PORT=3000
```

**Never hardcode secrets in a Dockerfile.** They get baked into the image and anyone who pulls it can see them in the layer history.

Instead, pass secrets at runtime:

```bash
docker run -e DATABASE_URL=postgres://... my-app
```

Or use a `.env` file with compose:

```yaml
services:
  app:
    env_file:
      - .env          # loaded at runtime, not baked into image
```

For production, use proper secret management — Docker Secrets, AWS Secrets Manager, Vault, etc.

---

## 6. Volumes — Data Persistence

Containers are ephemeral. When a container dies, all data inside dies with it. Volumes solve this.

```yaml
services:
  db:
    image: postgres:15
    volumes:
      - pgdata:/var/lib/postgresql/data   # named volume (managed by Docker)

  app:
    volumes:
      - ./code:/app     # bind mount (syncs local folder into container)

volumes:
  pgdata:
```

Two types to know:

- **Named volumes** — Docker manages where data lives on the host. Best for databases.
- **Bind mounts** — maps a folder from your machine into the container. Great for local dev so code changes reflect immediately without rebuilding.

---

## 7. Networking

By default, containers in the same `docker compose` file can talk to each other **by service name**. That's why your app can connect to a database using `db` as the hostname:

```
DATABASE_URL=postgres://user:pass@db:5432/mydb
                                   ^^
                            service name in compose
```

Behind the scenes, Compose creates a private network and handles DNS automatically. You don't need to know the container's IP.

---

## 8. Health Checks

Docker can monitor if your container is actually healthy, not just running:

```dockerfile
HEALTHCHECK --interval=30s --timeout=5s --retries=3 \
  CMD curl -f http://localhost:5000/health || exit 1
```

Or in compose:

```yaml
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost:5000/health"]
  interval: 30s
  timeout: 5s
  retries: 3
```

Useful for `depends_on` — you can make your app wait until the database is actually ready, not just started.

---
## NOTE : Decoupling Containers 
Every container should have a single set of responsiblity, Each container should have only one concern. Decoupling applications into multiple containers makes it easier to scale horizontally and reuse containers. 

---

## 9. Running as Non-Root

By default containers run as root, which is a security risk. Add a user:

```dockerfile
RUN addgroup --system appgroup && adduser --system appuser --ingroup appgroup
USER appuser          # all subsequent commands run as this user
```

This is a small change with a meaningful security improvement, especially in production.

---

## 10. Useful Commands You'll Use Daily

```bash
# Build
docker build -t my-app .
docker build --no-cache -t my-app .     # force full rebuild

# Run
docker run -d -p 3000:3000 my-app       # -d = detached (background)
docker run -it my-app bash              # interactive shell

# Inspect & Debug
docker logs <container>                 # view logs
docker logs -f <container>             # follow logs live
docker exec -it <container> bash       # shell into running container
docker inspect <container>             # full details in JSON

# Cleanup (important — Docker eats disk space)
docker system prune                    # remove unused everything
docker image prune                     # unused images only
docker volume prune                    # unused volumes only

# Compose
docker compose up -d                   # start in background
docker compose up --build              # rebuild then start
docker compose down -v                 # stop and delete volumes too
docker compose logs -f app             # follow logs for one service
```

---

## What to Practice

The best way to cement this is to take a real app — even a simple one — and:

1. Write a Dockerfile for it with proper layer caching.
2. Add a `docker-compose.yml` with a database.
3. Try a multi-stage build and compare image sizes.
4. Break something and debug it with `docker logs` and `docker exec` .

Simplest way to do this in production enviorment : 
1. Put stable and static layers first. 
2. Use .dockerignore -> and ignore the frequently changed files. 
3. Using multistage builds. 
4. BuildKit -> better nas faster way to use docker. DOCKER_BUILDKIT=1 docker build . 
		- Better caching
		- Parallel builds
		- Cache mounts
		- Smaller layers

