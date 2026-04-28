# Day 46 – Reusable Workflows & Composite Actions

## What I Built
- A reusable workflow callable by any workflow in the repo
- A caller workflow that passes inputs, secrets, and reads outputs
- A custom composite action that runs as a step

---

## Task 1: Understanding `workflow_call`

**What is a reusable workflow?**
A reusable workflow is a GitHub Actions workflow file designed to be called by other workflows rather than triggered directly by events like `push` or `pull_request`. It works like a function — define the logic once, and any workflow can invoke it with inputs and secrets.

**What is the `workflow_call` trigger?**
`workflow_call` is the event that marks a workflow as reusable. It supports typed `inputs`, `secrets`, and `outputs` — letting callers pass data in and receive data back.

**How is calling a reusable workflow different from using a regular action?**

| | Regular Action (`uses:`) | Reusable Workflow (`uses:` with `.yml`) |
|---|---|---|
| What it is | A single step inside a job | An entire workflow with multiple jobs |
| Runs inside | The caller's job | Its own job(s) with own runners |
| Can have multiple jobs | ❌ | ✅ |
| Syntax | `uses: actions/checkout@v4` | `uses: org/repo/.github/workflows/deploy.yml@main` |

**Where must a reusable workflow file live?**
Inside `.github/workflows/` of a GitHub repository. Called using:
```
uses: ./.github/workflows/filename.yml          # same repo
uses: org/repo/.github/workflows/filename.yml@main  # cross-repo
```

---

## Task 2 & 3: Reusable Workflow + Caller

### `.github/workflows/reusable-build.yml`

```yaml
name: Reusable Build Workflow

on:
  workflow_call:
    inputs:
      app_name:
        type: string
        required: true
      environment:
        type: string
        required: false
        default: staging
    secrets:
      docker_token:
        required: true
    outputs:
      build_version:
        description: "The generated build version string"
        value: ${{ jobs.checkout_code.outputs.build_version }}

jobs:
  checkout_code:
    runs-on: ubuntu-latest
    outputs:
      build_version: ${{ steps.version.outputs.build_version }}
    steps:
      - name: Generate version string
        id: version
        run: echo "build_version=v1.0-${{ github.sha }}" >> $GITHUB_OUTPUT

      - name: Checkout code
        uses: actions/checkout@v4

      - name: Print app name and environment
        run: echo "Building ${{ inputs.app_name }} for ${{ inputs.environment }}"

      - name: Check Docker token
        run: |
          if [ -n "${{ secrets.docker_token }}" ]; then
            echo "Docker token is set: true"
          else
            echo "Docker token is set: false"
          fi
```

### `.github/workflows/call-build.yml`

```yaml
name: Build Caller

on:
  push:
    branches:
      - main

jobs:
  build:
    uses: ./.github/workflows/reusable-build.yml
    with:
      app_name: "my-web-app"
      environment: "production"
    secrets:
      docker_token: ${{ secrets.DOCKER_TOKEN }}

  print_version:
    runs-on: ubuntu-latest
    needs: [build]
    steps:
      - name: Print build version
        run: echo "Build version is ${{ needs.build.outputs.build_version }}"
```

---

## Task 4: Output Bubbling Chain

Outputs travel upward through three levels:

```
step (id: version)
  echo "build_version=v1.0-abc123" >> $GITHUB_OUTPUT
        ↓
job (checkout_code)
  outputs:
    build_version: ${{ steps.version.outputs.build_version }}
        ↓
workflow_call outputs:
  build_version:
    value: ${{ jobs.checkout_code.outputs.build_version }}
        ↓
caller workflow:
  ${{ needs.build.outputs.build_version }}
```

**Result:**
```
Build version is v1.0-3a835c829c525e5e64b886c7919b681be648ebc5
```

---

## Task 5: Composite Action

### `.github/actions/setup-and-greet/action.yml`

```yaml
name: Custom Composite Action
description: Prints a greeting message

inputs:
  name:
    description: "Name of the person to greet"
    required: true
  language:
    description: "Language for greeting"
    required: false
    default: en

outputs:
  greeted:
    description: "Whether the greeting was printed"
    value: ${{ steps.set_output.outputs.greeted }}

runs:
  using: composite
  steps:
    - name: Print greeting
      shell: bash
      run: |
        if [ "${{ inputs.language }}" = "en" ]; then
          echo "Hello, ${{ inputs.name }}!"
        else
          echo "Hello, ${{ inputs.name }}! (unknown language: ${{ inputs.language }})"
        fi

    - name: Print date and runner OS
      shell: bash
      run: |
        echo "Current date: $(date)"
        echo "Runner OS: ${{ runner.os }}"

    - name: Set greeted output
      id: set_output
      shell: bash
      run: echo "greeted=true" >> $GITHUB_OUTPUT
```

### `.github/workflows/greet.yml` (Caller)

```yaml
name: Greet Workflow

on:
  push:
    branches:
      - main

jobs:
  greet:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Run greeting action
        id: greet
        uses: ./.github/actions/setup-and-greet
        with:
          name: "Akash"
          language: "en"

      - name: Print greeted output
        run: echo "Greeted output is ${{ steps.greet.outputs.greeted }}"
```

**Result:**
```
Hello, Akash!
Current date: Tue Apr 28 12:03:22 UTC 2026
Runner OS: Linux
Greeted output is true
```

---

## Task 6: Reusable Workflow vs Composite Action

| | Reusable Workflow | Composite Action |
|---|---|---|
| Triggered by | `workflow_call` | `uses:` in a step |
| Can contain jobs? | ✅ Yes, multiple jobs | ❌ No, steps only |
| Can contain multiple steps? | ✅ Yes, inside jobs | ✅ Yes |
| Lives where? | `.github/workflows/` | `.github/actions/<name>/` |
| Can accept secrets directly? | ✅ Yes, via `secrets:` block | ❌ No, passed as inputs |
| Has its own runner? | ✅ Yes, declares `runs-on` | ❌ No, uses caller's runner |
| Needs `actions/checkout`? | ❌ Not required | ✅ Always required |
| Best for | Sharing full CI/CD pipelines across repos | Packaging reusable step logic |

---

## Key Lessons Learned

- **Underscore vs hyphen**: GitHub-defined keywords use hyphens (`ubuntu-latest`, `workflow_call`). Your own names (inputs, outputs, secrets) should use underscores (`app_name`, `build_version`) to avoid expression parsing issues.
- **Output bubbling**: Outputs must be explicitly passed up through three levels — step → job → workflow — nothing bubbles automatically.
- **Composite actions always need checkout**: The runner needs to find `action.yml` on disk, which only exists after `actions/checkout@v4` runs.
- **`actions` folder conflict**: Git can store a path as either a file or a directory — if the wrong one gets committed, use `git rm` to delete it before recreating the correct structure.

---


