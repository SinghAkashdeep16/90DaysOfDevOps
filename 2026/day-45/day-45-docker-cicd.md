# Day 45 — Dockerizing App with GitHub Actions


> Built a CI/CD pipeline that auto-builds the Flask Task Manager Docker image and pushes it to Docker Hub on every push to `main`.

---

## What I Built

A GitHub Actions workflow connecting Day 36 (Dockerizing Flask app) with real CI/CD automation. Every `git push` to `main` now automatically builds and ships the image to Docker Hub — no manual steps.

---

## Workflow File — `.github/workflows/docker_publish.yml`

```yaml
name: dockerizing application

on:
  push:
    branches:
      - main

jobs:
  docker:
    runs-on: ubuntu-latest
    steps:
      - name: GITHUB CHECKOUT
        uses: actions/checkout@v4

      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ vars.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKER_TOKEN }}

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Build and push
        uses: docker/build-push-action@v6
        with:
          context: 2026/day-45/Task-Manager
          push: true
          tags: akashjaura16/task-manager:latest
```

---

## Secrets & Variables Setup

| Name | Type | Value |
|------|------|-------|
| `DOCKERHUB_USERNAME` | Variable | `akashjaura16` |
| `DOCKER_TOKEN` | Secret | Docker Hub Personal Access Token (Read & Write) |

> Generate token at: hub.docker.com → Account Settings → Personal Access Tokens → Read & Write

---

## Docker Hub Image

```bash
docker pull akashjaura16/task-manager:latest
docker run -p 5000:5000 akashjaura16/task-manager:latest
```

→ https://hub.docker.com/r/akashjaura16/task-manager

---

## Bugs I Hit & Fixed

### Bug 1 — Wrong action names (extra "s" typo)

**Error:** Workflow failed immediately — actions not found

| Wrong (what I used) | Correct (fix) |
|---------------------|---------------|
| `docker/login-actions@v4` | `docker/login-action@v3` |
| `docker/setup-buildx-actions@v4` | `docker/setup-buildx-action@v3` |
| `docker/build-push-actions@v7` | `docker/build-push-action@v6` |

**Lesson:** The official Docker actions have no "s" — `login-action`, not `login-actions`.

---

### Bug 2 — Docker Hub token with read-only permissions

**Error:**
```
Error response from daemon: unauthorized: incorrect username or password
```

The username was correct (`akashjaura16`) but the token was set to **Read-only** — Docker Hub rejected the push.

**Fix:**
1. hub.docker.com → Account Settings → Personal Access Tokens
2. Delete old token
3. Generate new token → set **Read & Write**
4. Update `DOCKER_TOKEN` secret in GitHub repo settings
5. Trigger re-run:
```powershell
git commit --allow-empty -m "fix: docker token read write"
git push
```

---

## Key Concepts Learned

- GitHub Actions can build and push Docker images automatically on every commit
- Docker Hub Personal Access Tokens must have **Read & Write** permissions for push to work
- Action names are case-sensitive and version-specific — always double check the exact name
- `vars.*` is used for non-sensitive config (username), `secrets.*` for sensitive data (token)
- `context:` in `build-push-action` points to the folder containing your Dockerfile

---

## Tasks Completed

- [x] GitHub Actions workflow triggers on push to `main`
- [x] Workflow checks out repo code
- [x] Logs into Docker Hub using secrets
- [x] Builds Docker image from Flask Task Manager app
- [x] Pushes image as `akashjaura16/task-manager:latest`
- [x] Debugged and fixed wrong action names
- [x] Debugged and fixed read-only token issue
- [x] Image live and pullable on Docker Hub

---

## Resources

- [docker/login-action](https://github.com/docker/login-action)
- [docker/build-push-action](https://github.com/docker/build-push-action)
- [Docker Hub Access Tokens](https://docs.docker.com/security/for-developers/access-tokens/)

---

`#90DaysOfDevOps` `#DevOpsKaJosh` `#TrainWithShubham` `#GitHubActions` `#Docker`
