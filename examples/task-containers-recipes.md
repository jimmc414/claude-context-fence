---
description: "Container and environment recipes reference (internal)"
when_to_use: "Receives prepared arguments from task-containers with container context included"
when-to-use: "Receives prepared arguments from task-containers with container context included"
context: fork
model: inherit
allowed-tools: Read
---

# Container & Environment Recipes

**Tools:** docker, docker-compose, dive, ctop, tmux, screen, zellij, unshare, nsenter, chroot, buildx

---

**Task**: $ARGUMENTS

---

## Quick Reference

| Task | Command |
|------|---------|
| Run interactive | `docker run -it --rm ubuntu bash` |
| Run detached | `docker run -d -p 8080:80 --name web nginx` |
| List running | `docker ps` |
| List all | `docker ps -a` |
| Stop container | `docker stop name` |
| View logs | `docker logs -f --tail 100 name` |
| Exec into | `docker exec -it name bash` |
| Build image | `docker build -t name:tag .` |
| Compose up | `docker compose up -d` |
| New tmux | `tmux new -s name` |
| Attach tmux | `tmux attach -t name` |
| System cleanup | `docker system prune -af --volumes` |
| Security scan | `trivy image myapp:latest` |
| Image layers | `dive myapp:latest` |

---

## When to Use What

| Task | Best Tool | Why |
|------|-----------|-----|
| Run containers | docker | Industry standard |
| Rootless containers | podman | Drop-in docker replacement, no daemon |
| Multi-container apps | docker-compose | Declarative service orchestration |
| Container image analysis | dive | Layer-by-layer size inspection |
| Container monitoring | ctop | Top-like UI for containers |
| Terminal multiplexer | tmux | Most popular, rich ecosystem |
| Terminal multiplexer (modern) | zellij | Better defaults, discoverable |
| Session persistence | screen | Pre-installed on most systems |
| Env var management | direnv | Auto-load .envrc per directory |
| Tool version management | mise | Multi-language version manager |

---

## Docker Run

### Basic Run Modes
```bash
docker run -it --rm ubuntu bash           # Interactive, auto-remove on exit
docker run -d --name web nginx            # Detached with name
docker run -it --rm alpine sh             # Lightweight interactive shell
docker run --rm busybox echo "hello"      # One-shot command, auto-remove
```

### Port Mapping
```bash
docker run -d -p 8080:80 nginx            # Map host 8080 to container 80
docker run -d -p 127.0.0.1:8080:80 nginx  # Bind to localhost only
docker run -d -p 8080:80/udp nginx         # UDP port mapping
docker run -d -p 8080:80 -p 8443:443 nginx # Multiple ports
docker run -d -P nginx                     # Map all EXPOSE ports to random host ports
```

### Volume Mounts
```bash
docker run -v $(pwd):/app nginx            # Bind mount current dir
docker run -v mydata:/data nginx           # Named volume
docker run -v $(pwd)/conf:/etc/nginx/conf.d:ro nginx  # Read-only bind mount
docker run --tmpfs /tmp nginx              # tmpfs mount (memory-backed)
docker run --mount type=bind,source=$(pwd),target=/app nginx       # --mount syntax bind
docker run --mount type=volume,source=mydata,target=/data nginx    # --mount syntax volume
docker run --mount type=tmpfs,target=/tmp,tmpfs-size=256m nginx    # tmpfs with size limit
docker run -v /host/path:/container/path:z nginx   # SELinux relabel shared
docker run -v /host/path:/container/path:Z nginx   # SELinux relabel private
```

### Environment Variables
```bash
docker run -e MY_VAR=value nginx           # Single env var
docker run -e MY_VAR=value -e OTHER=foo nginx  # Multiple env vars
docker run --env-file .env nginx           # Load from file
docker run -e DB_HOST -e DB_PORT nginx     # Pass through from host env
```

### Resource Limits
```bash
docker run --memory=512m nginx             # Memory limit
docker run --memory=512m --memory-swap=1g nginx  # Memory + swap
docker run --cpus=1.5 nginx                # CPU limit (1.5 cores)
docker run --cpu-shares=512 nginx          # Relative CPU weight
docker run --pids-limit=100 nginx          # Process count limit
docker run --memory=512m --cpus=2 --pids-limit=100 nginx  # Combined limits
```

### Restart and Lifecycle
```bash
docker run -d --restart=no nginx           # Never restart (default)
docker run -d --restart=always nginx       # Always restart
docker run -d --restart=unless-stopped nginx  # Restart unless manually stopped
docker run -d --restart=on-failure:5 nginx # Restart on failure, max 5 times
docker run -d --init nginx                 # Use tini as PID 1 (proper signal handling)
```

### Entrypoint and Command
```bash
docker run --entrypoint /bin/sh nginx -c "echo hello"  # Override entrypoint
docker run nginx nginx -g "daemon off;"   # Override CMD
docker run -w /app node npm test          # Set working directory
docker run --user 1000:1000 nginx          # Run as specific UID:GID
docker run --user nobody nginx             # Run as named user
```

### Networking
```bash
docker run --network host nginx            # Use host network (no isolation)
docker run --network none nginx            # No network
docker run --network mynet nginx           # Custom network
docker run --dns 8.8.8.8 nginx            # Custom DNS
docker run --hostname myhost nginx         # Set hostname
docker run --add-host myhost:192.168.1.1 nginx  # Add /etc/hosts entry
```

### Security and Capabilities
```bash
docker run --privileged nginx              # Full privileges (avoid if possible)
docker run --cap-add=NET_ADMIN nginx       # Add specific capability
docker run --cap-drop=ALL --cap-add=NET_BIND_SERVICE nginx  # Drop all, add specific
docker run --security-opt no-new-privileges nginx  # Prevent privilege escalation
docker run --read-only nginx               # Read-only root filesystem
docker run --read-only --tmpfs /tmp nginx  # Read-only with writable /tmp
```

### GPU and Hardware
```bash
docker run --gpus all nvidia/cuda bash     # All GPUs
docker run --gpus '"device=0,1"' nvidia/cuda bash  # Specific GPUs
docker run --device=/dev/sda:/dev/sda ubuntu  # Pass through device
```

### Health Check
```bash
docker run -d --health-cmd="curl -f http://localhost/ || exit 1" \
  --health-interval=30s --health-timeout=10s --health-retries=3 \
  --health-start-period=15s nginx
```

---

## Docker Build

### Basic Build
```bash
docker build -t myapp:latest .             # Build from current dir
docker build -t myapp:v1.0 -f Dockerfile.prod .  # Specific Dockerfile
docker build -t myapp:latest --no-cache .  # Build without cache
docker build -t myapp:latest --pull .      # Always pull base image
docker build --build-arg VERSION=1.0 -t myapp .  # Build argument
docker build --target builder -t myapp-build .    # Build specific stage
```

### BuildKit
```bash
DOCKER_BUILDKIT=1 docker build -t myapp .  # Enable BuildKit
docker buildx build -t myapp .             # Use buildx
docker buildx build --progress=plain -t myapp .   # Verbose build output
docker buildx build --platform linux/amd64,linux/arm64 -t myapp . --push  # Multi-arch
docker buildx create --use --name mybuilder       # Create buildx builder
docker buildx ls                           # List builders
```

### BuildKit Secrets and SSH
```bash
docker buildx build --secret id=mysecret,src=./secret.txt -t myapp .  # Mount secret during build
docker buildx build --ssh default -t myapp .   # Forward SSH agent
# Dockerfile usage: RUN --mount=type=secret,id=mysecret cat /run/secrets/mysecret
# Dockerfile usage: RUN --mount=type=ssh git clone git@github.com:org/repo
```

### Cache Strategies
```bash
docker buildx build --cache-from type=registry,ref=myapp:cache \
  --cache-to type=registry,ref=myapp:cache -t myapp .   # Registry cache
docker buildx build --cache-from type=local,src=/tmp/cache \
  --cache-to type=local,dest=/tmp/cache -t myapp .       # Local cache
```

### Dockerfile Best Practices
```dockerfile
# Order layers from least to most frequently changed
FROM python:3.11-slim

# System deps first (rarely change)
RUN apt-get update && apt-get install -y --no-install-recommends \
    gcc libpq-dev \
    && rm -rf /var/lib/apt/lists/*

# Copy requirements before code (deps change less often)
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy code last (changes most often)
COPY . .

# Non-root user
RUN useradd -m appuser
USER appuser

HEALTHCHECK --interval=30s --timeout=10s --retries=3 \
  CMD curl -f http://localhost:8000/health || exit 1

EXPOSE 8000
CMD ["uvicorn", "app:app", "--host", "0.0.0.0", "--port", "8000"]
```

### Multi-Stage Build
```dockerfile
# Stage 1: Build
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Stage 2: Production (small final image)
FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### Minimal Images
```dockerfile
# Python: slim variant
FROM python:3.11-slim
# Python: distroless
FROM gcr.io/distroless/python3

# Go: scratch (static binary)
FROM golang:1.22 AS builder
WORKDIR /app
COPY . .
RUN CGO_ENABLED=0 go build -o /app/server .
FROM scratch
COPY --from=builder /app/server /server
ENTRYPOINT ["/server"]

# Node: distroless
FROM gcr.io/distroless/nodejs20-debian12
COPY --from=builder /app/dist /app
CMD ["/app/server.js"]
```

### .dockerignore
```
.git
.gitignore
node_modules
*.md
.env
.env.*
docker-compose*.yml
Dockerfile*
.dockerignore
__pycache__
*.pyc
.venv
.mypy_cache
.pytest_cache
coverage/
dist/
build/
```

---

## Docker Image Management

### Inspect and Analyze
```bash
docker images                              # List images
docker images --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}"  # Custom format
docker image inspect myapp                 # Full JSON metadata
docker image inspect --format='{{.Config.Env}}' myapp       # Env vars
docker image inspect --format='{{json .Config.Cmd}}' myapp  # CMD
docker image inspect --format='{{.Config.ExposedPorts}}' myapp  # Ports
docker image inspect --format='{{.RootFS.Layers}}' myapp    # Layer count
docker history myapp                       # Layer history
docker history --no-trunc myapp            # Full commands per layer
docker history --format "{{.Size}}\t{{.CreatedBy}}" myapp   # Size per layer
```

### Tag, Push, Pull
```bash
docker tag myapp:latest myregistry.com/myapp:v1.0  # Tag for registry
docker push myregistry.com/myapp:v1.0      # Push to registry
docker pull myregistry.com/myapp:v1.0      # Pull from registry
docker login myregistry.com                # Login to registry
docker login -u user -p token ghcr.io      # GitHub Container Registry
docker manifest inspect --verbose nginx:latest  # Multi-arch manifest
```

### Save, Load, Export
```bash
docker save -o myapp.tar myapp:latest      # Save image to tar
docker load -i myapp.tar                   # Load image from tar
docker save myapp | gzip > myapp.tar.gz    # Compressed save
docker load < myapp.tar.gz                 # Load compressed
docker export container_id > container.tar # Export running container filesystem
docker import container.tar myimage:snapshot  # Import filesystem as image
```

### Cleanup
```bash
docker image prune                         # Remove dangling images
docker image prune -a                      # Remove all unused images
docker image prune --filter "until=24h"    # Remove images older than 24h
docker rmi myapp:old                       # Remove specific image
docker rmi $(docker images -q -f dangling=true)  # Remove all dangling
```

---

## Docker Volumes & Bind Mounts

### Named Volumes
```bash
docker volume create mydata                # Create named volume
docker volume ls                           # List volumes
docker volume inspect mydata               # Volume details (mount point)
docker volume rm mydata                    # Remove volume
docker volume prune                        # Remove unused volumes
docker volume prune --filter "label!=keep" # Prune with filter
```

### Volume Backup and Restore
```bash
# Backup: mount volume and tar to host
docker run --rm -v mydata:/data -v $(pwd):/backup alpine \
  tar czf /backup/mydata-backup.tar.gz -C /data .

# Restore: untar from host into volume
docker run --rm -v mydata:/data -v $(pwd):/backup alpine \
  tar xzf /backup/mydata-backup.tar.gz -C /data

# Copy between volumes
docker run --rm -v source_vol:/from -v dest_vol:/to alpine \
  cp -a /from/. /to/
```

### Container File Copy
```bash
docker cp mycontainer:/app/logs/app.log ./app.log  # Copy from container
docker cp ./config.json mycontainer:/app/config.json  # Copy to container
docker cp mycontainer:/app/data/ ./local-data/     # Copy directory from container
```

### Permission Patterns
```bash
# Fix UID/GID mismatch with host
docker run -u $(id -u):$(id -g) -v $(pwd):/app myapp

# In Dockerfile: create user with matching UID
RUN groupadd -g 1000 appgroup && useradd -u 1000 -g appgroup appuser
USER appuser

# Fix volume permissions in entrypoint
# entrypoint.sh: chown -R appuser:appgroup /data && exec gosu appuser "$@"
```

---

## Debug Container Permission Denied

Systematic diagnostic workflow for containers crashing or failing with "permission denied" errors, especially on WSL2.

### Step 1: Identify the Failing Path and User

```bash
# Check container logs for the exact permission error
docker logs mycontainer 2>&1 | grep -i "permission denied"

# Exec into running container (or start interactively if crashed)
docker run -it --rm --entrypoint sh myimage

# Inside the container: check who you are running as
whoami
id
# Shows: uid=1000(appuser) gid=1000(appgroup) groups=...

# Check the file/directory that fails
ls -la /path/to/failing/file
ls -la /path/to/failing/directory/
stat /path/to/failing/file

# Test access explicitly
touch /path/to/target/testfile 2>&1
cat /path/to/target/somefile 2>&1
```

### Step 2: Diagnose Volume Mount Permissions

```bash
# Check mount configuration from host
docker inspect --format='{{json .Mounts}}' mycontainer | jq

# Check host-side permissions of the mounted directory
ls -la /host/path/to/mounted/dir
stat /host/path/to/mounted/dir

# Compare UIDs: host vs container
# On host:
id
ls -ln /host/path/to/mounted/dir
# In container:
id
ls -ln /mounted/path/

# Fix: run container as host user
docker run -u $(id -u):$(id -g) -v $(pwd):/app myimage

# Fix: match container user UID to host UID in Dockerfile
# RUN usermod -u 1000 appuser && groupmod -g 1000 appgroup
```

### Step 3: WSL2-Specific Permission Issues

```bash
# WSL2 mounts Windows drives at /mnt/c, /mnt/d, etc. with specific metadata behavior

# Check WSL mount options (inside WSL, not container)
mount | grep -E '(drvfs|9p)'
# WSL2 default: drvfs mounted with metadata option

# Check /etc/wsl.conf for mount options
cat /etc/wsl.conf
# [automount]
# options = "metadata,umask=022,fmask=011"

# Problem: Windows filesystem (NTFS via drvfs) doesn't support Linux permissions natively
# Files on /mnt/c/ may show as 777 or have mismatched ownership

# Fix: Use Linux-native paths instead of /mnt/c for volumes
# BAD  (Windows path, permission issues):
docker run -v /mnt/c/myproject:/app myimage
# GOOD (Linux-native WSL2 path):
docker run -v ~/myproject:/app myimage
# Linux home (~) is on ext4, supports full Linux permissions

# If you must use /mnt/c, ensure metadata mount option:
# In /etc/wsl.conf:
# [automount]
# options = "metadata"

# Check Docker Desktop WSL2 integration backend
docker info | grep -i "operating system\|storage driver\|server version"

# WSL2 Docker Desktop uses a Linux VM -- bind mounts from /mnt/c
# go through: Windows NTFS -> 9p -> WSL2 -> Docker bind mount
# Each layer can lose or misinterpret permissions
```

### Step 4: Check Container User Mapping

```bash
# Check what user the image runs as
docker image inspect --format='{{.Config.User}}' myimage
# Empty = root, otherwise shows UID or username

# Check effective user inside the container
docker run --rm myimage id
docker run --rm myimage whoami

# Check /etc/passwd inside container for user mapping
docker run --rm myimage cat /etc/passwd

# Run as root to verify permission is the issue (diagnostic only)
docker run --rm -u root -v $(pwd):/app myimage ls -la /app
docker run --rm -u root -v $(pwd):/app myimage touch /app/testfile

# If root works but normal user fails: UID/GID mismatch confirmed
# Fix options:
# 1. Match UIDs
docker run -u $(id -u):$(id -g) -v $(pwd):/app myimage

# 2. Use --group-add for supplementary groups
docker run -u $(id -u):$(id -g) --group-add $(stat -c '%g' /path/to/dir) -v $(pwd):/app myimage

# 3. Use named volumes (Docker manages permissions)
docker volume create mydata
docker run -v mydata:/app/data myimage

# 4. Init container to fix permissions
docker run --rm -v $(pwd):/app alpine chown -R 1000:1000 /app
```

### Step 5: Check Capabilities

```bash
# Check container's effective capabilities
docker inspect --format='{{json .HostConfig.CapAdd}}' mycontainer | jq
docker inspect --format='{{json .HostConfig.CapDrop}}' mycontainer | jq

# Inside container: check process capabilities
cat /proc/1/status | grep -i cap
# CapPrm = permitted, CapEff = effective, CapBnd = bounding set

# Decode capability hex values
capsh --decode=00000000a80425fb

# Common capabilities needed for specific operations:
# NET_BIND_SERVICE - bind to ports < 1024
# SYS_PTRACE       - strace/debugging inside container
# DAC_OVERRIDE     - bypass file read/write/execute permission checks
# CHOWN            - change file ownership
# FOWNER           - bypass permission checks on file owner
# SETUID/SETGID    - change process UID/GID

# Add specific capability
docker run --cap-add=DAC_OVERRIDE myimage
docker run --cap-add=CHOWN --cap-add=FOWNER myimage

# Drop all capabilities, add only needed ones (principle of least privilege)
docker run --cap-drop=ALL --cap-add=NET_BIND_SERVICE myimage

# Nuclear option: full privileges (avoid in production)
docker run --privileged myimage
```

### Step 6: Check Seccomp and AppArmor Restrictions

```bash
# Check if seccomp is blocking syscalls
docker inspect --format='{{json .HostConfig.SecurityOpt}}' mycontainer | jq

# Check seccomp profile
docker inspect --format='{{.HostConfig.SecurityOpt}}' mycontainer
# Default: uses Docker's built-in seccomp profile

# Run without seccomp to test (diagnostic only)
docker run --security-opt seccomp=unconfined myimage

# Check AppArmor status
docker inspect --format='{{.AppArmorProfile}}' mycontainer
# On WSL2: AppArmor is typically not active

# Check AppArmor from host
sudo aa-status 2>/dev/null
cat /sys/module/apparmor/parameters/enabled

# Run without AppArmor
docker run --security-opt apparmor=unconfined myimage

# Check if no-new-privileges is set (prevents privilege escalation via setuid binaries)
docker inspect --format='{{json .HostConfig.SecurityOpt}}' mycontainer | grep no-new-privileges

# Disable no-new-privileges if it blocks setuid binaries you need
docker run --security-opt no-new-privileges=false myimage
```

### Step 7: Read-Only Filesystem Issues

```bash
# Check if container has read-only root filesystem
docker inspect --format='{{.HostConfig.ReadonlyRootfs}}' mycontainer

# If read-only: add tmpfs mounts for writable directories
docker run --read-only --tmpfs /tmp --tmpfs /run --tmpfs /var/cache myimage

# Common paths that need to be writable:
docker run --read-only \
  --tmpfs /tmp \
  --tmpfs /run \
  --tmpfs /var/tmp \
  --tmpfs /var/cache \
  -v mydata:/app/data \
  myimage

# Check which paths the container tried to write to
docker diff mycontainer
# A = added, C = changed, D = deleted
```

### Step 8: Use strace to Find the Exact Failing Syscall

```bash
# From host: trace the container's main process
PID=$(docker inspect -f '{{.State.Pid}}' mycontainer)
sudo strace -f -e trace=%file -Z -p $PID 2>&1 | grep EACCES

# Run container with SYS_PTRACE capability for in-container strace
docker run --cap-add=SYS_PTRACE --security-opt seccomp=unconfined -it myimage sh
# Inside: apt install strace (or apk add strace for alpine)
strace -f -e trace=%file -Z ./myapp 2>&1 | grep EACCES

# Check for both EACCES (permission) and EPERM (operation not permitted)
sudo strace -f -e trace=%file,%desc -Z -p $PID 2>&1 | grep -E 'EACCES|EPERM'

# Common findings:
# openat(AT_FDCWD, "/path", O_WRONLY) = -1 EACCES   -> file/dir permission mismatch
# mkdir("/path", 0755) = -1 EACCES                    -> parent dir not writable
# unlink("/path/file") = -1 EPERM                     -> need DAC_OVERRIDE or FOWNER
```

### Diagnostic Decision Tree

```
Container crashes with "permission denied"
|
+-- Is it a volume mount path?
|   |
|   +-- YES: Check host path permissions (Step 2)
|   |   +-- On WSL2 /mnt/c? -> Use Linux-native path (Step 3)
|   |   +-- UID mismatch? -> Run as matching UID (Step 4)
|   |   +-- Host dir owned by root? -> chown or use named volume
|   |
|   +-- NO: Check container filesystem (Step 7)
|       +-- Read-only root? -> Add tmpfs for writable paths
|       +-- Image built with wrong permissions? -> Fix Dockerfile
|
+-- Is it a privileged operation?
|   |
|   +-- Binding port < 1024? -> --cap-add=NET_BIND_SERVICE
|   +-- Changing file ownership? -> --cap-add=CHOWN
|   +-- Debugging/ptrace? -> --cap-add=SYS_PTRACE
|   +-- Raw network access? -> --cap-add=NET_RAW
|
+-- Is it a security policy block?
    |
    +-- Seccomp blocking syscall? -> Test with seccomp=unconfined (Step 6)
    +-- AppArmor blocking? -> Test with apparmor=unconfined (Step 6)
    +-- no-new-privileges blocking setuid? -> Disable if needed (Step 6)
```

### Common Fixes Summary

| Problem | Symptom | Fix |
|---------|---------|-----|
| UID mismatch | Container user can't read/write host-mounted files | `docker run -u $(id -u):$(id -g)` |
| WSL2 Windows mount | Files on /mnt/c have wrong permissions | Use Linux-native path (~/project) |
| Root-owned volume | Non-root container user can't write | Init container to chown, or named volume |
| Missing capability | Operation not permitted on privileged action | `--cap-add=SPECIFIC_CAP` |
| Read-only rootfs | Cannot write to /tmp, /var, etc. | `--tmpfs /tmp --tmpfs /run` |
| Seccomp block | Specific syscall denied | `--security-opt seccomp=unconfined` (test only) |
| AppArmor block | Access denied despite correct permissions | `--security-opt apparmor=unconfined` (test only) |
| no-new-privileges | setuid binary fails | `--security-opt no-new-privileges=false` |
| Group permission | File readable by group but container user not in group | `--group-add GID` |
| Docker socket | Container needs Docker access | `-v /var/run/docker.sock:/var/run/docker.sock --group-add $(stat -c '%g' /var/run/docker.sock)` |

---

## Docker Networking

### Network Management
```bash
docker network create mynet                # Create bridge network
docker network create --driver bridge --subnet 172.20.0.0/16 mynet  # With subnet
docker network create --driver overlay mynet  # Overlay (Swarm)
docker network create --driver macvlan \
  --subnet=192.168.1.0/24 --gateway=192.168.1.1 \
  -o parent=eth0 mymacvlan                 # Macvlan
docker network ls                          # List networks
docker network inspect mynet               # Network details
docker network rm mynet                    # Remove network
docker network prune                       # Remove unused networks
```

### Connect Containers
```bash
docker network connect mynet container1    # Connect running container
docker network disconnect mynet container1 # Disconnect running container
docker run -d --network mynet --name web nginx      # Start on network
docker run -d --network mynet --name api node       # Same network = DNS resolution
# web can reach api at hostname "api" and vice versa

docker run -d --network mynet --ip 172.20.0.10 --name web nginx  # Static IP
```

### Port Mapping Patterns
```bash
docker run -d -p 8080:80 nginx             # Basic mapping
docker run -d -p 127.0.0.1:8080:80 nginx   # Localhost only
docker run -d -p 8080:80/tcp -p 8080:80/udp nginx  # TCP and UDP
docker run -d -P nginx                     # All EXPOSE ports to random
docker port mycontainer                    # Show port mappings
docker port mycontainer 80                 # Show mapping for port 80
```

### Inter-Container Communication
```bash
# Method 1: Custom bridge (recommended, has DNS)
docker network create app-net
docker run -d --network app-net --name db postgres
docker run -d --network app-net --name web -e DB_HOST=db myapp

# Method 2: Container link (legacy, deprecated)
docker run -d --name db postgres
docker run -d --link db:database myapp     # db accessible as "database"

# Method 3: Host network (no isolation)
docker run -d --network host nginx         # Uses host ports directly
```

### DNS and Hostname
```bash
docker run --dns 8.8.8.8 --dns 8.8.4.4 nginx          # Custom DNS servers
docker run --dns-search example.com nginx               # DNS search domain
docker run --hostname myhost nginx                      # Set container hostname
docker run --add-host dbhost:192.168.1.100 nginx        # Custom /etc/hosts entry
docker run --add-host host.docker.internal:host-gateway nginx  # Access host from container
```

---

## Docker Compose v2

### Basic Commands
```bash
docker compose up                          # Start all services (foreground)
docker compose up -d                       # Start detached
docker compose up -d --build               # Build and start
docker compose up -d web db                # Start specific services only
docker compose down                        # Stop and remove containers
docker compose down -v                     # Stop and remove containers + volumes
docker compose down --rmi all              # Stop and remove containers + images
docker compose stop                        # Stop without removing
docker compose start                       # Start stopped services
docker compose restart                     # Restart all services
docker compose restart web                 # Restart specific service
```

### Build and Run
```bash
docker compose build                       # Build all services
docker compose build --no-cache web        # Rebuild without cache
docker compose build --parallel            # Build in parallel
docker compose run --rm web bash           # One-off command in new container
docker compose run --rm web npm test       # Run tests
docker compose exec web bash               # Exec into running service
docker compose exec -u root web bash       # Exec as root
```

### Logs and Status
```bash
docker compose logs                        # All service logs
docker compose logs -f                     # Follow all logs
docker compose logs -f --tail 100 web      # Follow specific service, last 100
docker compose logs --since 1h web         # Logs since 1 hour ago
docker compose ps                          # List running services
docker compose ps -a                       # All services (including stopped)
docker compose top                         # Processes in services
docker compose stats                       # Resource usage
```

### Scaling and Profiles
```bash
docker compose up -d --scale web=3         # Scale service to 3 instances
docker compose --profile dev up -d         # Start with profile
docker compose --profile dev --profile debug up -d  # Multiple profiles
```

### Compose Watch (live reload)
```bash
docker compose watch                       # Start watch mode
docker compose up --watch                  # Up with watch
# In compose.yml:
# develop:
#   watch:
#     - action: sync
#       path: ./src
#       target: /app/src
#     - action: rebuild
#       path: package.json
```

### Example compose.yml
```yaml
services:
  web:
    build:
      context: .
      dockerfile: Dockerfile
      target: development
      args:
        NODE_ENV: development
    ports:
      - "3000:3000"
    volumes:
      - .:/app
      - node_modules:/app/node_modules
    environment:
      - NODE_ENV=development
      - DATABASE_URL=postgres://user:pass@db:5432/mydb
    env_file:
      - .env
    depends_on:
      db:
        condition: service_healthy
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 15s
    networks:
      - app-net
    deploy:
      resources:
        limits:
          cpus: "1.0"
          memory: 512M
        reservations:
          cpus: "0.5"
          memory: 256M

  db:
    image: postgres:16-alpine
    volumes:
      - pgdata:/var/lib/postgresql/data
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
      POSTGRES_DB: mydb
    ports:
      - "5432:5432"
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user -d mydb"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - app-net

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    command: redis-server --appendonly yes
    networks:
      - app-net

volumes:
  pgdata:
  redis_data:
  node_modules:

networks:
  app-net:
    driver: bridge

secrets:
  db_password:
    file: ./secrets/db_password.txt
```

### Variable Substitution
```bash
# In compose.yml: image: ${REGISTRY:-docker.io}/${IMAGE}:${TAG:-latest}
# .env file provides defaults
# Override: TAG=v2 docker compose up -d
```

### Override Files
```bash
# docker-compose.yml        (base)
# docker-compose.override.yml  (auto-loaded, local dev overrides)
# docker-compose.prod.yml   (production)
docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d
```

---

## Container Inspection & Debugging

### Logs
```bash
docker logs mycontainer                    # All logs
docker logs -f mycontainer                 # Follow logs
docker logs --tail 100 mycontainer         # Last 100 lines
docker logs -t mycontainer                 # With timestamps
docker logs --since 1h mycontainer         # Last hour
docker logs --since 2024-01-01T00:00:00 mycontainer  # Since specific time
docker logs --until 30m mycontainer        # Until 30 minutes ago
docker logs -f --tail 0 mycontainer        # Follow from now (no history)
```

### Exec into Container
```bash
docker exec -it mycontainer bash           # Interactive bash
docker exec -it mycontainer sh             # Interactive sh (alpine)
docker exec -u root mycontainer bash       # As root
docker exec -e MY_VAR=value mycontainer bash  # With env var
docker exec -w /app mycontainer ls -la     # With working directory
docker exec mycontainer cat /etc/os-release  # Non-interactive command
```

### Inspect
```bash
docker inspect mycontainer                 # Full JSON
docker inspect --format='{{.State.Status}}' mycontainer        # Status
docker inspect --format='{{.State.Pid}}' mycontainer           # PID on host
docker inspect --format='{{.NetworkSettings.IPAddress}}' mycontainer  # IP address
docker inspect --format='{{json .NetworkSettings.Ports}}' mycontainer | jq  # Port mappings
docker inspect --format='{{json .Mounts}}' mycontainer | jq    # Volume mounts
docker inspect --format='{{json .Config.Env}}' mycontainer | jq  # Environment vars
docker inspect --format='{{json .State.Health}}' mycontainer | jq  # Health status
docker inspect --format='{{.Config.Entrypoint}}' mycontainer   # Entrypoint
docker inspect --format='{{.Config.Cmd}}' mycontainer          # CMD
docker inspect --format='{{.RestartCount}}' mycontainer        # Restart count
docker inspect --format='{{.HostConfig.RestartPolicy}}' mycontainer  # Restart policy
```

### Resource Monitoring
```bash
docker stats                               # Live resource usage (all containers)
docker stats --no-stream                   # One-shot snapshot
docker stats --no-stream --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}"
docker top mycontainer                     # Processes inside container
docker system df                           # Disk usage summary
docker system df -v                        # Verbose disk usage per item
docker events                              # Real-time container events
docker events --filter container=mycontainer  # Events for specific container
docker events --filter event=start --filter event=die  # Specific events
```

### Filesystem and Snapshots
```bash
docker diff mycontainer                    # Filesystem changes (A=added, C=changed, D=deleted)
docker commit mycontainer myimage:snapshot # Create image from running container
docker commit -m "added debug tools" -c 'CMD ["/bin/bash"]' mycontainer debug:latest
```

### Debug with Temporary Container
```bash
# Share network namespace with target container
docker run -it --rm --network container:mycontainer nicolaka/netshoot
# Debug networking: curl, dig, nslookup, tcpdump, iptables, ss

# Share PID namespace for process debugging
docker run -it --rm --pid container:mycontainer alpine ps aux
```

---

## Image Analysis

### Dive (Layer Explorer)
```bash
dive myapp:latest                          # Interactive layer analysis
CI=true dive myapp:latest                  # CI mode (non-interactive, score)
dive myapp:latest --ci                     # Same as above
dive build -t myapp .                      # Build and analyze in one step
# Controls: Tab=switch pane, Space=toggle file tree, Ctrl-A=show aggregated changes
```

### Docker History
```bash
docker history myapp:latest                # Show layers
docker history --no-trunc myapp:latest     # Full commands (not truncated)
docker history --format "{{.Size}}\t{{.CreatedBy}}" myapp  # Size per layer
docker history --format "table {{.ID}}\t{{.Size}}\t{{.CreatedBy}}" --no-trunc myapp
```

### Container Security and Vulnerability Scanning
```bash
trivy image myapp:latest                   # Full vulnerability scan
trivy image --severity HIGH,CRITICAL myapp:latest  # Only high/critical
trivy image --format json myapp:latest > scan.json  # JSON output
trivy image --exit-code 1 --severity CRITICAL myapp  # Fail on critical (CI)
grype myapp:latest                         # Alternative scanner
docker scout cves myapp:latest             # Docker Scout scan
docker scout quickview myapp:latest        # Scout overview
```

### Image Comparison
```bash
# Compare sizes
docker images --format "{{.Repository}}:{{.Tag}}\t{{.Size}}" | sort -k2 -h

# Before/after optimization
docker images myapp --format "table {{.Tag}}\t{{.Size}}"
```

---

## tmux

> tmux is included here for managing container development sessions (multi-pane Docker workflows, monitoring logs while exec'ing). See task-network for SSH tunneling, see task-process for general job control.

### Sessions
```bash
tmux new -s dev                            # New session named "dev"
tmux new -s dev -d                         # New detached session
tmux ls                                    # List sessions
tmux attach -t dev                         # Attach to session
tmux attach -t dev -d                      # Attach, detach others
tmux kill-session -t dev                   # Kill session
tmux kill-server                           # Kill tmux server (all sessions)
tmux rename-session -t old new             # Rename session
tmux switch -t dev                         # Switch to session (from within tmux)
tmux has-session -t dev                    # Check if session exists (exit code)
```

### Session Keybindings
```
Ctrl-b d       Detach from session
Ctrl-b $       Rename session
Ctrl-b s       List/switch sessions (interactive)
Ctrl-b (       Previous session
Ctrl-b )       Next session
```

### Windows
```bash
tmux new-window                            # Create window (from within tmux)
tmux new-window -n logs                    # Named window
tmux select-window -t 0                    # Switch to window 0
tmux rename-window -t 0 main              # Rename window 0
tmux kill-window -t 2                      # Kill window 2
tmux move-window -t 5                      # Move current window to position 5
tmux list-windows                          # List windows
```

### Window Keybindings
```
Ctrl-b c       Create new window
Ctrl-b n       Next window
Ctrl-b p       Previous window
Ctrl-b 0-9     Switch to window by number
Ctrl-b w       List windows (interactive picker)
Ctrl-b ,       Rename current window
Ctrl-b &       Kill current window (with confirmation)
Ctrl-b l       Toggle last window
Ctrl-b f       Find window by name
```

### Panes
```bash
tmux split-window -h                       # Split horizontally (left/right)
tmux split-window -v                       # Split vertically (top/bottom)
tmux split-window -h -p 30                 # Split with 30% width
tmux select-pane -t 0                      # Select pane 0
tmux resize-pane -D 5                      # Resize down 5 lines
tmux resize-pane -U 5                      # Resize up 5 lines
tmux resize-pane -L 10                     # Resize left 10 columns
tmux resize-pane -R 10                     # Resize right 10 columns
tmux swap-pane -U                          # Swap with pane above
tmux swap-pane -D                          # Swap with pane below
tmux kill-pane -t 1                        # Kill pane 1
```

### Pane Keybindings
```
Ctrl-b %       Split horizontal (left/right)
Ctrl-b "       Split vertical (top/bottom)
Ctrl-b o       Cycle through panes
Ctrl-b ;       Toggle last pane
Ctrl-b Up/Down/Left/Right   Navigate panes
Ctrl-b z       Toggle pane zoom (fullscreen)
Ctrl-b {       Move pane left
Ctrl-b }       Move pane right
Ctrl-b x       Kill pane (with confirmation)
Ctrl-b !       Convert pane to window
Ctrl-b q       Show pane numbers (press number to jump)
Ctrl-b Space   Cycle through layouts (even-h, even-v, main-h, main-v, tiled)
Ctrl-b Ctrl-Up/Down/Left/Right  Resize pane
```

### Copy Mode
```
Ctrl-b [       Enter copy mode
q              Exit copy mode
/              Search forward
?              Search backward
n              Next search result
N              Previous search result
Space          Start selection (vi mode)
Enter          Copy selection (vi mode)
Ctrl-b ]       Paste buffer
```

### Session Scripting
```bash
# Create a development session with layout
tmux new-session -d -s dev -n editor
tmux send-keys -t dev:editor 'vim .' C-m
tmux new-window -t dev -n server
tmux send-keys -t dev:server 'npm run dev' C-m
tmux new-window -t dev -n logs
tmux send-keys -t dev:logs 'tail -f /var/log/app.log' C-m
tmux select-window -t dev:editor
tmux attach -t dev

# Create 4-pane grid (2x2)
tmux new-session -d -s grid
tmux split-window -h
tmux split-window -v
tmux select-pane -t 0
tmux split-window -v
tmux select-layout tiled
tmux attach -t grid

# Send command to all panes
tmux set-window-option synchronize-panes on
tmux send-keys 'uptime' C-m
tmux set-window-option synchronize-panes off
```

### Pair Programming
```bash
# Host creates shared socket
tmux -S /tmp/shared-tmux new -s pair
chmod 777 /tmp/shared-tmux

# Guest attaches
tmux -S /tmp/shared-tmux attach -t pair

# Read-only guest
tmux -S /tmp/shared-tmux attach -t pair -r
```

### tmux.conf Configuration
```bash
# ~/.tmux.conf
set -g mouse on                            # Enable mouse
set -g history-limit 50000                 # Scroll history
set -g base-index 1                        # Start windows at 1
setw -g pane-base-index 1                  # Start panes at 1
set -g renumber-windows on                 # Renumber on close
set -g default-terminal "screen-256color"  # 256 colors
set -sg escape-time 0                      # No escape delay

# Vi mode
setw -g mode-keys vi
bind-key -T copy-mode-vi v send -X begin-selection
bind-key -T copy-mode-vi y send -X copy-selection

# Easy splits
bind | split-window -h -c "#{pane_current_path}"
bind - split-window -v -c "#{pane_current_path}"

# Reload config
bind r source-file ~/.tmux.conf \; display "Reloaded"

# Status bar
set -g status-style bg=colour235,fg=colour136
set -g status-left '#[fg=green]#S '
set -g status-right '#[fg=yellow]%Y-%m-%d %H:%M'
```

### Pipe and Capture
```bash
tmux capture-pane -t dev:0 -p              # Print pane contents to stdout
tmux capture-pane -t dev:0 -p > output.txt # Save pane output
tmux capture-pane -S -100 -p               # Capture last 100 lines
tmux pipe-pane -t dev:0 'cat >> ~/tmux.log'  # Log pane output to file
tmux pipe-pane -t dev:0                    # Stop logging
```

---

## Namespace Isolation

### unshare
```bash
# PID namespace (isolated process tree)
sudo unshare --pid --fork --mount-proc bash
# ps aux inside only shows processes in this namespace

# Network namespace (isolated networking)
sudo unshare --net bash
# ip a shows only loopback

# Mount namespace (isolated filesystem mounts)
sudo unshare --mount bash
mount -t tmpfs tmpfs /mnt
# Mount only visible in this namespace

# User namespace (fake root)
unshare --user --map-root-user bash
# Shows uid 0 but actually unprivileged

# UTS namespace (isolated hostname)
sudo unshare --uts bash
hostname mycontainer
# Only affects this namespace

# Combined (mini container)
sudo unshare --pid --fork --mount-proc --net --uts --mount bash
```

### nsenter
```bash
# Enter all namespaces of a container
nsenter --target $(docker inspect -f '{{.State.Pid}}' mycontainer) --all

# Enter specific namespaces of running container
PID=$(docker inspect -f '{{.State.Pid}}' mycontainer)
sudo nsenter --target $PID --mount --uts --ipc --net --pid

# Enter only network namespace
sudo nsenter --target $PID --net ip addr

# Enter mount namespace to inspect filesystem
sudo nsenter --target $PID --mount ls /app
```

### chroot
```bash
# Basic chroot
sudo chroot /path/to/new/root /bin/bash

# Setup minimal chroot
mkdir -p /tmp/myroot/{bin,lib,lib64,proc,sys,dev}
cp /bin/bash /tmp/myroot/bin/
# Copy required shared libraries (use ldd /bin/bash to find them)
sudo mount -t proc proc /tmp/myroot/proc
sudo chroot /tmp/myroot /bin/bash
# Cleanup
sudo umount /tmp/myroot/proc
```

---

## Docker System Cleanup

```bash
docker system prune                        # Remove stopped containers, unused networks, dangling images
docker system prune -a                     # Also remove unused images (not just dangling)
docker system prune -af --volumes          # Nuclear: remove everything unused including volumes
docker system prune --filter "until=168h"  # Remove items older than 7 days
docker container prune                     # Remove stopped containers only
docker image prune                         # Remove dangling images only
docker image prune -a                      # Remove all unused images
docker volume prune                        # Remove unused volumes only
docker network prune                       # Remove unused networks only
docker builder prune                       # Remove build cache
docker builder prune --all                 # Remove all build cache
docker system df                           # Show disk usage
docker system df -v                        # Verbose disk usage
```

### screen

```bash
# Start named session
screen -S mywork                               # Create named session
screen -r mywork                               # Reattach to named session
screen -ls                                     # List sessions
screen -d -r mywork                            # Detach elsewhere and reattach here

# Inside screen:
# Ctrl-a d       — detach
# Ctrl-a c       — new window
# Ctrl-a n/p     — next/previous window
# Ctrl-a "       — list windows
# Ctrl-a S       — split horizontal
# Ctrl-a |       — split vertical
# Ctrl-a [       — scroll mode (PgUp/PgDn, q to exit)
# Ctrl-a k       — kill current window

# Run command in detached screen
screen -dmS backup rsync -av /data /backup     # Start in background
screen -S backup -X stuff "ls\n"               # Send command to session
```

### zellij

```bash
# Start and manage sessions
zellij                                         # Start new session
zellij -s myproject                            # Named session
zellij attach myproject                        # Attach to session
zellij ls                                      # List sessions
zellij delete-session myproject                # Delete session

# Inside zellij:
# Ctrl-p         — pane mode (n: new, x: close, arrow: navigate)
# Ctrl-t         — tab mode (n: new, x: close, r: rename)
# Ctrl-n         — resize mode
# Ctrl-s         — scroll mode
# Ctrl-o         — session mode
# Ctrl-q         — quit

# Layout file (~/.config/zellij/layouts/dev.kdl)
# layout { pane; pane split_direction="vertical" { pane; pane; } }
zellij --layout dev                            # Start with layout
```

### ctop

```bash
ctop                                           # Interactive container monitor
# Inside ctop:
# f — filter containers
# s — sort (CPU, MEM, Net, IO)
# Enter — single container view (logs, inspect, exec)
# h — help
```

### Podman (Docker-Compatible)

```bash
# Podman is a drop-in Docker replacement (rootless by default)
podman run -d -p 8080:80 nginx                 # Run container (no daemon needed)
podman build -t myapp .                        # Build from Dockerfile
podman ps                                      # List running containers
podman images                                  # List images
podman-compose up -d                           # Docker Compose compatible

# Rootless containers (no sudo)
podman run --userns=keep-id -v ./data:/data:Z myapp   # Map UID, SELinux label

# Convert between Docker and Podman
alias docker=podman                            # Drop-in replacement
podman generate systemd --name myapp > myapp.service  # Create systemd unit
```

### Docker Logging Configuration

```bash
# Configure log driver per container
docker run --log-driver json-file --log-opt max-size=10m --log-opt max-file=3 myapp
docker run --log-driver syslog --log-opt syslog-address=udp://loghost:514 myapp
docker run --log-driver local --log-opt max-size=50m myapp  # Faster than json-file

# Check container log config
docker inspect --format '{{.HostConfig.LogConfig}}' mycontainer

# Daemon-wide default (/etc/docker/daemon.json)
# { "log-driver": "json-file", "log-opts": { "max-size": "10m", "max-file": "3" } }
```

### Combined Workflows

```bash
# Container error log analysis
docker logs mycontainer 2>&1 | awk '/ERROR/{print $0}' | sort | uniq -c | sort -rn

# Resource-hungry containers
docker stats --no-stream --format "{{.Name}}\t{{.MemUsage}}\t{{.CPUPerc}}" | sort -t$'\t' -k2 -h -r

# Inspect + jq for networking
docker inspect mycontainer | jq '.[0].NetworkSettings.Networks'

# Bulk cleanup exited containers
docker ps -aq --filter "status=exited" | xargs docker rm

# Image layer size analysis
docker history myimage --format "{{.Size}}\t{{.CreatedBy}}" | head -20
```

---

## Installation

### Docker
```bash
# Ubuntu/Debian - official Docker Engine
sudo apt-get update
sudo apt-get install -y ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] \
  https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Add user to docker group (no sudo needed)
sudo usermod -aG docker $USER
newgrp docker
```

### Additional Tools
```bash
# tmux
sudo apt-get install -y tmux

# dive (image layer analysis)
wget https://github.com/wagoodman/dive/releases/latest/download/dive_0.12.0_linux_amd64.deb
sudo dpkg -i dive_0.12.0_linux_amd64.deb

# ctop (container top)
sudo wget https://github.com/bcicen/ctop/releases/latest/download/ctop-0.7.7-linux-amd64 \
  -O /usr/local/bin/ctop && sudo chmod +x /usr/local/bin/ctop

# trivy (security scanner)
sudo apt-get install -y wget apt-transport-https gnupg lsb-release
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
echo "deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main" | \
  sudo tee /etc/apt/sources.list.d/trivy.list
sudo apt-get update && sudo apt-get install -y trivy
```
