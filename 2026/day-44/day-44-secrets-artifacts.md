# Day 44 – Secrets, Artifacts & Running Real Tests in CI

## Overview

In this lab, I implemented key CI/CD concepts including:

* Secure handling of secrets
* Using environment variables in workflows
* Uploading and downloading artifacts
* Running real test scripts in CI
* Using caching to improve performance

---

## Task 1: GitHub Secrets

### Implementation

Created a secret:

* `MY_SECRET_MESSAGE`

Workflow snippet:

```yaml
- name: Check if secret exists
  run: |
    if [ -n "${{ secrets.MY_SECRET_MESSAGE }}" ]; then
      echo "The secret is set: true"
    else
      echo "The secret is set: false"
    fi
```

### Observation

When printing the secret directly:

```yaml
echo "${{ secrets.MY_SECRET_MESSAGE }}"
```

Output:

```
***
```

### Learning

GitHub automatically masks secrets in logs.

### Answer

Secrets should never be printed in CI logs because they may expose sensitive data such as API keys, tokens, or passwords, leading to security risks.

---

## Task 2: Using Secrets as Environment Variables

### Implementation

```yaml
env:
  USERNAME: ${{ secrets.DOCKER_USERNAME }}
  TOKEN: ${{ secrets.DOCKER_TOKEN }}
```

Usage:

```bash
echo "$TOKEN" | docker login -u "$USERNAME" --password-stdin
```

### Learning

Secrets can be safely passed as environment variables and should never be hardcoded.

---

## Task 3: Upload Artifacts

### Implementation

```yaml
- name: Create file
  run: docker --version > docker-info.txt

- name: Upload artifact
  uses: actions/upload-artifact@v4
  with:
    name: workflow-output
    path: docker-info.txt
```

### Verification

Artifact successfully appeared in the **Actions tab** and was downloadable.

---

## Task 4: Download Artifacts Between Jobs

### Implementation

```yaml
jobs:
  job1:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: docker --version > docker-info.txt
      - uses: actions/upload-artifact@v4
        with:
          name: workflow-output
          path: docker-info.txt

  job2:
    runs-on: ubuntu-latest
    needs: job1
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: workflow-output
          path: downloaded

      - run: cat downloaded/docker-info.txt
```

### Learning

Artifacts allow files (logs, builds, reports) to be shared between jobs.

### Answer

Artifacts are used in real pipelines to pass build outputs, logs, and reports between jobs without regenerating them.

---

## Task 5: Run Real Tests in CI

### Script (`script.sh`)

```bash
#!/bin/bash
echo "Running simple CI test..."
echo "Test passed"
exit 0
```

### Workflow

```yaml
name: Run Script in CI

on:
  push:
    branches:
      - main

jobs:
  job1:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: chmod +x .github/workflows/script.sh
      - run: ./.github/workflows/script.sh
```

### Testing Failure

Modified script:

```bash
exit 1
```

Result:

* Pipeline failed ❌

### Fix

Restored:

```bash
exit 0
```

Result:

* Pipeline passed ✅

### Learning

CI pipelines fail automatically when a script exits with a non-zero code.

---

## Task 6: Caching

### Implementation

```yaml
- name: Cache dependencies
  uses: actions/cache@v4
  with:
    path: ~/.cache
    key: cache-${{ runner.os }}
```

### Observation

* First run: slower
* Second run: faster (~2 seconds improvement)

### Learning

GitHub Actions cache stores dependency files and restores them in future runs to improve performance.

### Answer

The cache stores files such as dependencies or downloaded packages, and it is stored in GitHub’s cloud storage associated with the repository and cache key.

---

## Complete Workflow (Combined Example)

```yaml
name: Day 44 CI Pipeline

on:
  push:
    branches:
      - main

jobs:
  job1:
    runs-on: ubuntu-latest

    env:
      USERNAME: ${{ secrets.DOCKER_USERNAME }}
      TOKEN: ${{ secrets.DOCKER_TOKEN }}

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Cache files
        uses: actions/cache@v4
        with:
          path: ~/.cache
          key: cache-${{ runner.os }}

      - name: Login to Docker
        run: echo "$TOKEN" | docker login -u "$USERNAME" --password-stdin

      - name: Create file
        run: docker --version > docker-info.txt

      - name: Upload artifact
        uses: actions/upload-artifact@v4
        with:
          name: workflow-output
          path: docker-info.txt

  job2:
    runs-on: ubuntu-latest
    needs: job1

    steps:
      - name: Download artifact
        uses: actions/download-artifact@v4
        with:
          name: workflow-output
          path: downloaded

      - name: Print file
        run: cat downloaded/docker-info.txt
```

---

## Screenshots (Add Here)

* Artifact download
* Successful CI run
* Failed CI run

---

## Conclusion

This lab helped me understand how real CI pipelines work, including secure secret handling, artifact management, automated testing, and caching optimization.

---

#90DaysOfDevOps #DevOpsKaJosh #TrainWithShubham
