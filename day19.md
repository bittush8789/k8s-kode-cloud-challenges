# Print Environment Variables

# Normal Problem Statement

The DevOps team at xFusionCorp Industries wants to inspect and print environment variables inside a Linux system for debugging and application configuration management.

Your task is to print system environment variables, create custom variables, and verify them on an Ubuntu-based system.

---

# Scenario-Based Problem Statement

xFusionCorp Industries is deploying cloud-native applications and CI/CD pipelines across multiple environments.

The DevOps and Platform Engineering teams use environment variables for:

- Database credentials
- API endpoints
- Cloud configurations
- Kubernetes secrets
- CI/CD variables
- Application runtime settings

The engineering team is facing issues:

- Environment variables not loading
- Incorrect runtime configurations
- Missing deployment variables
- Debugging difficulties in production

As a DevOps Engineer, your responsibilities are to:

1. Print Linux environment variables.
2. Create custom environment variables.
3. Verify variable persistence.
4. Debug runtime configurations.
5. Validate application environments.

---

# Objectives

1. Understand Linux environment variables.
2. Print existing environment variables.
3. Create temporary variables.
4. Create persistent variables.
5. Verify environment configurations.

---

# What are Environment Variables?

Environment Variables are key-value pairs used by:

- Operating systems
- Applications
- CI/CD pipelines
- Containers
- Kubernetes
- Cloud platforms

They store configuration information.

Example:

```text
HOME=/home/ubuntu
USER=ubuntu
PATH=/usr/local/bin
```

---

# Architecture Overview

```text
Linux System
      |
      v
Environment Variables
      |
      +------------------+
      |                  |
      v                  v
Applications       Shell Sessions
      |
      v
Runtime Configuration
```

---

# Solution

# Step 1: Update Ubuntu Packages

```bash
sudo apt update && sudo apt upgrade -y
```

---

# Step 2: Print All Environment Variables

```bash
printenv
```

Expected output:

```text
HOME=/home/ubuntu
USER=ubuntu
PATH=/usr/local/bin
```

---

# Step 3: Alternative Method Using env

```bash
env
```

Expected output:

```text
SHELL=/bin/bash
PWD=/home/ubuntu
USER=ubuntu
```

---

# Step 4: Print Specific Environment Variable

Print current user:

```bash
echo $USER
```

Print home directory:

```bash
echo $HOME
```

Print shell:

```bash
echo $SHELL
```

---

# Step 5: Print PATH Variable

```bash
echo $PATH
```

Expected output:

```text
/usr/local/bin:/usr/bin:/bin
```

---

# Step 6: Create Temporary Environment Variable

```bash
export PROJECT_NAME="mlops-project"
```

Verify:

```bash
echo $PROJECT_NAME
```

Expected output:

```text
mlops-project
```

---

# Step 7: Create Multiple Variables

```bash
export ENVIRONMENT="production"

export APP_VERSION="v1.0"

export CLOUD_PROVIDER="AWS"
```

Verify:

```bash
echo $ENVIRONMENT

echo $APP_VERSION

echo $CLOUD_PROVIDER
```

---

# Step 8: List Custom Variables

```bash
printenv | grep PROJECT
```

Example:

```text
PROJECT_NAME=mlops-project
```

---

# Step 9: Create Persistent Environment Variable

Open bash configuration:

```bash
nano ~/.bashrc
```

Add the following lines at the bottom:

```bash
export COMPANY_NAME="xFusionCorp"

export DEVOPS_TOOL="Kubernetes"
```

Save and exit.

---

# Step 10: Reload Bash Configuration

```bash
source ~/.bashrc
```

---

# Step 11: Verify Persistent Variables

```bash
echo $COMPANY_NAME

echo $DEVOPS_TOOL
```

Expected output:

```text
xFusionCorp

Kubernetes
```

---

# Step 12: Print All Bash Variables

```bash
set
```

---

# Step 13: Print Environment Variables Alphabetically

```bash
printenv | sort
```

---

# Step 14: Print Variable Count

```bash
printenv | wc -l
```

Expected output:

```text
Total environment variable count
```

---

# Step 15: Remove Environment Variable

```bash
unset PROJECT_NAME
```

Verify:

```bash
echo $PROJECT_NAME
```

Expected output:

```text
(blank)
```

---

# Step 16: Create Variable for Current Session Only

```bash
TEMP_VAR="temporary"
```

Verify:

```bash
echo $TEMP_VAR
```

Difference:

- `export` makes variable available to child processes
- normal variables remain local

---

# Step 17: Print Variables Inside Script

Create script:

```bash
nano env.sh
```

Add:

```bash
#!/bin/bash

echo "User: $USER"

echo "Home: $HOME"

echo "Shell: $SHELL"

echo "Project: $PROJECT_NAME"
```

Make executable:

```bash
chmod +x env.sh
```

Run script:

```bash
./env.sh
```

---

# Step 18: Verify Variable Persistence

Open new terminal:

```bash
echo $COMPANY_NAME
```

Expected:

```text
xFusionCorp
```

---

# Step 19: Common Environment Variables

| Variable | Purpose |
|---|---|
| HOME | User home directory |
| USER | Current username |
| PATH | Executable search path |
| SHELL | Current shell |
| PWD | Present working directory |
| HOSTNAME | System hostname |

---

# Step 20: Delete Persistent Variable

Edit `.bashrc`:

```bash
nano ~/.bashrc
```

Remove:

```bash
export COMPANY_NAME="xFusionCorp"
```

Reload:

```bash
source ~/.bashrc
```

---

# Expected Outcome

- Environment variables printed successfully
- Custom variables created
- Persistent variables configured
- Runtime configuration verified
- Linux environment debugging operational

---

# Real Industry Use Cases

| Use Case | Example |
|---|---|
| CI/CD | Pipeline variables |
| Kubernetes | Secrets & configs |
| Docker | Runtime variables |
| Cloud | AWS credentials |
| DevOps | Deployment configs |

---

# Skills Covered

- Linux Environment Variables
- Bash Variables
- Shell Scripting
- Runtime Configuration
- Linux Debugging
- DevOps Fundamentals

---

# Real Enterprise Workflow

```text
Application Start
        |
        v
Load Environment Variables
        |
        v
Application Configuration
        |
        v
Runtime Execution
```

---

# Complete Command Sequence

```bash
sudo apt update && sudo apt upgrade -y

printenv

env

echo $USER

echo $HOME

echo $SHELL

echo $PATH

export PROJECT_NAME="mlops-project"

echo $PROJECT_NAME

export ENVIRONMENT="production"

export APP_VERSION="v1.0"

export CLOUD_PROVIDER="AWS"

printenv | grep PROJECT

nano ~/.bashrc

source ~/.bashrc

echo $COMPANY_NAME

echo $DEVOPS_TOOL

set

printenv | sort

printenv | wc -l

unset PROJECT_NAME

echo $PROJECT_NAME

TEMP_VAR="temporary"

echo $TEMP_VAR

nano env.sh

chmod +x env.sh

./env.sh
```