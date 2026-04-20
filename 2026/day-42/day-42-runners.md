# 📘 Day 42 – Runners: GitHub-Hosted & Self-Hosted

## 📌 Overview
In this task, I explored GitHub Actions runners, including GitHub-hosted runners and self-hosted runners. I created workflows across multiple operating systems and configured a self-hosted runner on my local machine (WSL Ubuntu).

---

## ✅ Task 1: GitHub-Hosted Runners

I created a workflow with three jobs running on:
- ubuntu-latest
- windows-latest
- macos-latest

Each job printed:
- OS name
- Hostname
- Current user

### 🔹 Learning
A GitHub-hosted runner is a virtual machine provided and managed by GitHub. It automatically executes workflows without requiring any manual setup.

---

## ✅ Task 2: Pre-installed Tools

On the ubuntu-latest runner, I checked:
- Docker version
- Python version
- Node version
- Git version

### 🔹 Learning
GitHub-hosted runners come with pre-installed tools, which saves setup time and allows workflows to run immediately without manual installation.

---

## ✅ Task 3: Self-Hosted Runner Setup

I set up a self-hosted runner on my local machine (WSL Ubuntu).

### 🔹 Steps
- Created a runner folder
- Downloaded the runner package
- Configured using ./config.sh
- Started the runner using ./run.sh

### 🔹 Verification
The runner appeared in GitHub with a green "Idle" status, confirming successful setup.

---

## ✅ Task 4: Using Self-Hosted Runner

I created a workflow using:

runs-on: self-hosted

### 🔹 Steps performed
- Printed hostname → confirmed it was my machine (Asus)
- Printed working directory
- Created a file hello.txt
- Verified the file using ls -l

### 🔹 Local Verification
```bash
cd /home/akash/actions-runner/_work/github_actions_practice/github_actions_practice
ls -l
cat hello.txt