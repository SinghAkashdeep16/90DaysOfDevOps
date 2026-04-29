# Day 47 – Advanced Triggers: PR Events, Cron Schedules & Event-Driven Pipelines

---

## Task 1: PR Lifecycle — `pr-lifecycle.yml`

```yaml
name: PR Lifecycle

on:
  pull_request:
    types: [opened, synchronize, reopened, closed]

jobs:
  pr-info:
    runs-on: ubuntu-latest
    steps:
      - name: Print event type
        run: echo "Event: ${{ github.event.action }}"

      - name: Print PR title
        run: echo "Title: ${{ github.event.pull_request.title }}"

      - name: Print PR author
        run: echo "Author: ${{ github.event.pull_request.user.login }}"

      - name: Print branches
        run: |
          echo "Source: ${{ github.head_ref }}"
          echo "Target: ${{ github.base_ref }}"

      - name: Run only on merge
        if: github.event.pull_request.merged == true
        run: echo "PR was merged!"
```

---

## Task 2: PR Validation — `pr-checks.yml`

```yaml
name: PR CHECKS

on:
  pull_request:
    branches:
      - main

jobs:
  file-size-check:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Fails if file > 1 MB
        run: |
          find . -size +1M -not -path './.git/*' > large_files.txt
          if [ -s large_files.txt ]; then
            echo "Files larger than 1MB found:"
            cat large_files.txt
            exit 1
          fi

  branch-name-check:
    runs-on: ubuntu-latest
    steps:
      - name: Read branch name
        run: |
          BRANCH="${{ github.head_ref }}"
          if [[ ! "$BRANCH" =~ ^(feature|fix|docs)/ ]]; then
            echo "Bad branch name: $BRANCH"
            exit 1
          fi

  pr-body-check:
    runs-on: ubuntu-latest
    steps:
      - name: PR body check
        run: |
          msg="${{ github.event.pull_request.body }}"
          if [ -z "$msg" ]; then
            echo "Warning: PR description is empty"
          fi
```

---

## Task 3: Scheduled Workflows — `scheduled-tasks.yml`

```yaml
name: Scheduled Tasks

on:
  workflow_dispatch:
  schedule:
    - cron: '30 2 * * 1'      # Every Monday at 2:30 AM UTC
    - cron: '0 */6 * * *'     # Every 6 hours

jobs:
  scheduled:
    runs-on: ubuntu-latest
    steps:
      - name: Print which schedule triggered
        run: echo "Triggered by schedule: ${{ github.event.schedule }}"

      - name: Health check
        run: |
          response=$(curl -o /dev/null -s -w "%{http_code}" https://google.com)
          if [ "$response" -ne 200 ]; then
            echo "Health check failed: $response"
            exit 1
          fi
          echo "Health check passed: $response"
```

### Cron Notes

| Description | Expression |
|---|---|
| Every weekday at 9 AM IST (3:30 AM UTC) | `30 3 * * 1-5` |
| First day of every month at midnight | `0 0 1 * *` |

**Why GitHub may delay/skip scheduled workflows:**
GitHub automatically disables scheduled workflows on repos with no activity for 60 days. Even on active repos, scheduled jobs run on shared runners and can be delayed during high traffic — GitHub does not guarantee exact timing.

---

## Task 4: Path & Branch Filters

### `smart-triggers.yml` — Run only when src/ or app/ changes

```yaml
name: Smart Triggers

on:
  push:
    branches:
      - main
      - 'release/*'
    paths:
      - 'src/**'
      - 'app/**'

jobs:
  smart-trigger-job:
    runs-on: ubuntu-latest
    steps:
      - name: Print trigger info
        run: echo "Triggered by ${{ github.event_name }} on ${{ github.ref }}"
```

### `ignore-docs-changes.yml` — Skip when only docs change

```yaml
name: Ignore Docs Changes

on:
  push:
    branches:
      - main
      - 'release/*'
    paths-ignore:
      - '*.md'
      - 'docs/**'

jobs:
  ignore-docs-job:
    runs-on: ubuntu-latest
    steps:
      - name: Print trigger info
        run: echo "Triggered by ${{ github.event_name }} on ${{ github.ref }}"
```

### `paths` vs `paths-ignore`

| | When to use |
|---|---|
| `paths` | Run **only** for specific folders — whitelist ✅ |
| `paths-ignore` | Run for **everything except** certain files — blacklist ❌ |

---

## Task 5: Chain Workflows — `workflow_run`

### `tests.yml`

```yaml
name: Run Tests

on:
  push:

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Run tests
        run: echo "Running tests..."
```

### `deploy-after-tests.yml`

```yaml
name: Deploy After Tests

on:
  workflow_run:
    workflows: ["Run Tests"]
    types: [completed]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Check if tests passed
        run: |
          if [ "${{ github.event.workflow_run.conclusion }}" == "success" ]; then
            echo "Deploying!"
          else
            echo "Tests failed! Skipping deploy."
            exit 1
          fi
```

### `workflow_run` vs `workflow_call`

| | `workflow_run` | `workflow_call` |
|---|---|---|
| Trigger | Automatically after another workflow finishes | Explicitly called like a function |
| Use case | Chain independent workflows | Reuse workflow logic across repos |
| Access to artifacts | Yes | Yes |
| Coupling | Loose | Tight |

---

## Task 6: External Trigger — `external-trigger.yml`

```yaml
name: External Trigger

on:
  repository_dispatch:
    types: [deploy-request]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Print environment
        run: echo "${{ github.event.client_payload.environment }}"
```

### Trigger via GitHub CLI

```bash
gh api repos/SinghAkashdeep16/github_actions_practice/dispatches -f event_type=deploy-request -f client_payload='{"environment":"production"}'
```

### When would an external system trigger a pipeline?

- **Slack bot** — deploy button pressed by team member
- **Monitoring tool** — alert fires, triggers auto-rollback
- **Another repo** — finishes building a dependency, triggers downstream
- **External CI/CD** — pipeline outside GitHub wants to notify and trigger a job

`repository_dispatch` is essentially a **webhook you control** — any system that can make an HTTP request can trigger your GitHub Actions pipeline, passing custom data via `client_payload`.

---

`#90DaysOfDevOps` `#DevOpsKaJosh` `#TrainWithShubham`
