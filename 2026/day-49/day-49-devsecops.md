# Day 49 – DevSecOps: Add Security to Your CI/CD Pipeline

## Overview

Today I added automated security scanning to my existing GitHub Actions pipeline. The goal was to catch vulnerabilities **before** they reach production — not after.

---

## What is DevSecOps?

DevSecOps = shift security left. Instead of a separate security team checking things after deployment, security checks run automatically inside the CI/CD pipeline on every PR and every push to main.

---

## What I Built

### Trivy Image Scanning
- Added Trivy to `main-pipeline.yml` — scans the Docker image **after push, before deploy**
- Added Trivy to `pr-pipeline.yml` — builds image locally and scans it **before merge**
- Pipeline **fails automatically** if CRITICAL or HIGH fixable vulnerabilities are found
- `ignore-unfixed: true` skips CVEs with no patch available yet (realistic production approach)

### Dependency Review
- Added `actions/dependency-review-action@v4` to PR pipeline
- Checks any **new packages** added in a PR against a CVE database
- Fails the PR if a new dependency has a critical vulnerability

### GitHub Secret Scanning + Push Protection
- Enabled in repo Settings → Code security and analysis
- Secret scanning detects API keys/tokens already in the repo
- Push protection **blocks the push** before a secret even enters the repo

### Workflow Permissions (Least Privilege)
- Added `permissions: contents: read` to `main-pipeline.yml`
- Added `permissions: contents: read` + `pull-requests: write` to `pr-pipeline.yml`
- Limits what a compromised third-party action can do to the repo

---

## Pipeline Flow

```
PR Opened
  ├── build & test
  ├── dependency-review    ← checks new packages for CVEs
  ├── trivy-scan           ← builds image locally, scans before merge
  └── pr-comment           ← only runs if all above pass

Merge to Main
  ├── build & test
  ├── docker build + push
  ├── trivy-scan           ← scans pushed image, fails on CRITICAL/HIGH
  └── deploy               ← only runs if scan passes

Always Active
  ├── Secret scanning      ← detects secrets already in repo
  └── Push protection      ← blocks secrets before they enter repo
```

---

## Real Scan Results

First run found **114 CVEs (6 CRITICAL, 108 HIGH)** — pipeline blocked deploy as expected.

Root cause: `python:3.9-slim` base image (EOL, no security patches).

### Fixes Applied
| Issue | Fix |
|-------|-----|
| Base image `python:3.9-slim` | Upgraded to `python:3.13-slim` |
| `greenlet==2.0.2` | Upgraded to `3.1.1` (Python 3.13 incompatible) |
| `gunicorn==21.2.0` | Upgraded to `23.0.0` (HTTP smuggling CVEs) |
| Remaining OS CVEs | `ignore-unfixed: true` (no patch available upstream) |

**Final result: All Python packages → 0 vulnerabilities. Pipeline passing ✅**

---

## Key Takeaways

- **Shift left** — a CVE found in a PR takes minutes to fix, in production it takes days
- **Automate everything** — don't rely on someone remembering to check
- **Fail fast** — `exit-code: '1'` means bad images never reach production
- **Least privilege** — limit workflow permissions so compromised actions can't damage the repo
- **Real base images matter** — using an EOL base image is a security risk, always use supported versions

---

## Files Changed
- `.github/workflows/main-pipeline.yml` — added Trivy scan + permissions
- `.github/workflows/pr-pipeline.yml` — added Trivy scan + dependency review + permissions
- `Dockerfile` — upgraded to `python:3.13-slim`, added `apt-get upgrade`
- `requirements.txt` — upgraded all packages to Python 3.13 compatible secure versions

---

\
`#90DaysOfDevOps` `#DevOpsKaJosh` `#TrainWithShubham`
