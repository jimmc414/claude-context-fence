---
description: "Container management - container won't start, image too large, keeps crashing, permission denied in container, volume not mounting, Docker run/build/compose, image optimization, and namespace isolation (docker, podman, docker-compose, tmux)"
when_to_use: "Use when: container won't start or keeps crashing, docker image is too large, permission denied inside container, volume not mounting correctly, can't reach service in container, build is slow, need to debug container logs, running Docker containers, building images, using docker-compose, managing tmux sessions, isolating processes with namespaces, setting up development environments. Triggers: container won't start, container keeps crashing, image too large, permission denied in container, volume not mounting, can't reach container service, build slow, crash loop, docker, container, docker-compose, tmux, screen, namespace, devcontainer, dockerfile, compose, build image, container logs, docker network, docker run, docker build, compose up, multi-stage build, podman, volume mount, container network, security scan, vulnerability scan, trivy, grype, CVE, docker scout."
when-to-use: "Use when: container won't start or keeps crashing, docker image is too large, permission denied inside container, volume not mounting correctly, can't reach service in container, build is slow, need to debug container logs, running Docker containers, building images, using docker-compose, managing tmux sessions, isolating processes with namespaces, setting up development environments. Triggers: container won't start, container keeps crashing, image too large, permission denied in container, volume not mounting, can't reach container service, build slow, crash loop, docker, container, docker-compose, tmux, screen, namespace, devcontainer, dockerfile, compose, build image, container logs, docker network, docker run, docker build, compose up, multi-stage build, podman, volume mount, container network, security scan, vulnerability scan, trivy, grype, CVE, docker scout."
argument-hint: "[describe the container or environment task]"
context: inherit
model: inherit
allowed-tools: Skill, Read
---

# Container & Environment Task Router

You have access to the **full conversation context**. Your job is to analyze the user's container/environment need and build a well-formed argument for the recipes skill.

**User's request**: $ARGUMENTS

---

## Your Task

Container requests often involve multi-step operations. Analyze the conversation to identify:

### 1. Goal
What operation is needed?
- Run a container (what image, ports, volumes?)
- Build an image (Dockerfile context, multi-stage?)
- Compose services (which services, dependencies?)
- Debug a container (logs, exec, inspect?)
- Manage tmux sessions (create, attach, split?)
- Isolate process (namespace type?)
- Optimize image (reduce size, layer caching?)

### 2. Target
What are we working with?
- Container/image name from conversation
- Dockerfile mentioned or discussed
- Compose file referenced
- tmux session name
- Process to isolate

### 3. Environment Context
- Docker Desktop or native Docker?
- Podman instead of Docker?
- WSL2? (mount path differences)
- Docker Compose v1 or v2?
- Resource constraints?

### 4. Symptom (if debugging)
- Container won't start (check logs, entrypoint)
- Crash loop (restart policy, health check)
- Can't reach service (network, port mapping)
- Image too large (multi-stage, layer analysis)
- Permission denied (user mapping, volumes)
- Build slow (cache invalidation, .dockerignore)

### 5. Prior Findings
- Error messages from docker logs
- Dockerfile content discussed
- docker-compose.yml structure

---

## Goal --> Tool Guide

| Goal | Primary Tool | Argument Prefix |
|------|-------------|----------------|
| Run container | docker run | "run container" |
| Build image | docker build | "build image" |
| Compose services | docker compose | "compose" |
| Container debugging | docker logs/exec/inspect | "debug container" |
| Image analysis | dive/docker history | "analyze image" |
| tmux session | tmux | "tmux" |
| Volume management | docker volume | "manage volume" |
| Network setup | docker network | "container network" |
| Namespace isolation | unshare/nsenter | "isolate with namespace" |
| Clean up | docker system prune | "clean up docker" |

---

## Argument Format

```
<goal> for <target> - config: <relevant settings>, context: <prior findings>
```

### Examples

**Run with ports and volumes:**
```
run container nginx:latest - ports: 8080:80, volume: ./html:/usr/share/nginx/html, detached
```

**Debug crashing container:**
```
debug container myapp - crash loop, exit code 137, OOM suspected
```

**Compose with dependencies:**
```
compose up web + db services - web depends on db healthy, need persistent postgres volume
```

**tmux session:**
```
tmux create 4-pane layout - 2x2 grid, name: dev, run different commands in each
```

**Optimize image:**
```
analyze image myapp:latest - 2.1GB, need to reduce size, uses python:3.11
```

---

## Invoke Recipes

Once you've built the argument, use the Skill tool:

```
Skill: task-containers-recipes
Args: <your constructed argument>
```

### When to Ask Instead

**Ask the user if:**
- Image name/tag is unclear
- Multiple containers could be the target
- Compose file structure isn't clear from context

**Use Read tool first if:**
- Need to check Dockerfile mentioned in conversation
- Need to verify docker-compose.yml structure
- Need to inspect container config files
