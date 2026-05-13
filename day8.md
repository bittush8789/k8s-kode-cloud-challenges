# Day 8: Schedule CronJobs in Kubernetes using KIND

# Normal Problem Statement

The DevOps team at xFusionCorp Industries wants to automate scheduled tasks in Kubernetes using CronJobs.

Your task is to create a Kubernetes cluster (if not already available), configure Kubernetes CronJobs using manifest files, and verify automated scheduled task execution on an Ubuntu system.

---

# Scenario-Based Problem Statement

xFusionCorp Industries manages multiple enterprise applications inside Kubernetes clusters.

The organization requires automated scheduled operations such as:

- Database backups
- Log cleanup
- Report generation
- Health checks
- Cache clearing
- ML batch jobs

Currently, these tasks are executed manually, creating several operational challenges:

- Missed scheduled operations
- Manual intervention overhead
- Inconsistent task execution
- Human errors during maintenance
- Lack of centralized job scheduling

To solve these problems, the organization wants to implement Kubernetes CronJobs for automated task scheduling.

As a DevOps Engineer, your responsibility is to:

1. Create a Kubernetes cluster if one does not exist.
2. Configure Kubernetes CronJobs.
3. Schedule automated tasks using YAML manifests.
4. Verify scheduled Job execution and logs.

---

# Objectives

1. Install Docker, kubectl, and KIND.
2. Create a Kubernetes cluster using KIND.
3. Create Kubernetes CronJob manifest files.
4. Schedule recurring tasks automatically.
5. Verify CronJob execution.
6. Monitor Job and Pod logs.
7. Delete CronJobs safely.

---

# Architecture Overview

```text
KIND Kubernetes Cluster
        |
        v
CronJob
        |
        v
Scheduled Job
        |
        v
Temporary Pod
        |
        v
Task Execution
```

---

# Solution

## Step 1: Update Ubuntu Packages

```bash
sudo apt update && sudo apt upgrade -y
```

---

# Step 2: Install Docker

```bash
sudo apt install docker.io -y
```

Enable Docker:

```bash
sudo systemctl enable docker

sudo systemctl start docker
```

Verify Docker:

```bash
docker --version
```

Add current user to Docker group:

```bash
sudo usermod -aG docker $USER
```

Apply changes:

```bash
newgrp docker
```

---

# Step 3: Install kubectl

```bash
sudo apt install -y kubectl
```

Verify installation:

```bash
kubectl version --client
```

---

# Step 4: Install KIND

Download KIND:

```bash
curl -Lo ./kind https://kind.sigs.k8s.io/dl/latest/kind-linux-amd64
```

Make executable:

```bash
chmod +x ./kind
```

Move binary:

```bash
sudo mv ./kind /usr/local/bin/kind
```

Verify installation:

```bash
kind --version
```

---

# Step 5: Create KIND Kubernetes Cluster

```bash
kind create cluster --name cronjob-cluster
```

Expected output:

```text
Creating cluster "cronjob-cluster" ...
Cluster creation complete.
```

---

# Step 6: Verify Kubernetes Cluster

```bash
kubectl cluster-info --context kind-cronjob-cluster
```

Check nodes:

```bash
kubectl get nodes
```

Expected output:

```text
NAME                               STATUS   ROLES
cronjob-cluster-control-plane    Ready    control-plane
```

---

# Step 7: Create Kubernetes Project Directory

```bash
mkdir -p ~/k8s-cronjob-project

cd ~/k8s-cronjob-project
```

---

# Step 8: Create CronJob Manifest File

```bash
nano backup-cronjob.yaml
```

Add the following configuration:

```yaml
apiVersion: batch/v1
kind: CronJob

metadata:
  name: backup-cronjob

spec:
  schedule: "*/1 * * * *"

  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 1

  jobTemplate:
    spec:
      template:
        spec:
          containers:
            - name: backup-container

              image: busybox

              args:
                - /bin/sh
                - -c
                - date; echo "Database backup completed"

          restartPolicy: OnFailure
```

---

# Complete CronJob Manifest File

```yaml
apiVersion: batch/v1
kind: CronJob

metadata:
  name: backup-cronjob

spec:
  schedule: "*/1 * * * *"

  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 1

  jobTemplate:
    spec:
      template:
        spec:
          containers:
            - name: backup-container

              image: busybox

              args:
                - /bin/sh
                - -c
                - date; echo "Database backup completed"

          restartPolicy: OnFailure
```

---

# Cron Schedule Explanation

```text
*/1 * * * *
```

Meaning:

| Field | Value | Description |
|---|---|---|
| Minute | */1 | Every minute |
| Hour | * | Every hour |
| Day | * | Every day |
| Month | * | Every month |
| Weekday | * | Every weekday |

---

# Step 9: Deploy CronJob

```bash
kubectl apply -f backup-cronjob.yaml
```

Expected output:

```text
cronjob.batch/backup-cronjob created
```

---

# Step 10: Verify CronJob

```bash
kubectl get cronjobs
```

Expected output:

```text
NAME              SCHEDULE      SUSPEND
backup-cronjob    */1 * * * *   False
```

---

# Step 11: Monitor Scheduled Jobs

Wait 1-2 minutes, then check Jobs:

```bash
kubectl get jobs
```

Expected output:

```text
NAME                         COMPLETIONS   DURATION
backup-cronjob-xxxxx        1/1           5s
```

---

# Step 12: Verify Running Pods

```bash
kubectl get pods
```

Expected output:

```text
NAME                               READY   STATUS
backup-cronjob-xxxxx-abcde        0/1     Completed
```

---

# Step 13: Check CronJob Logs

Get Pod name:

```bash
kubectl get pods
```

View logs:

```bash
kubectl logs <pod-name>
```

Example:

```bash
kubectl logs backup-cronjob-xxxxx-abcde
```

Expected output:

```text
Thu Jan 2 10:00:00 UTC 2026
Database backup completed
```

---

# Step 14: Describe CronJob

```bash
kubectl describe cronjob backup-cronjob
```

Displays:

- Schedule information
- Last schedule time
- Job history
- Events
- Job template details

---

# Step 15: Suspend CronJob

```bash
kubectl patch cronjob backup-cronjob -p '{"spec":{"suspend":true}}'
```

Verify suspension:

```bash
kubectl get cronjobs
```

Expected output:

```text
NAME              SUSPEND
backup-cronjob    True
```

---

# Step 16: Resume CronJob

```bash
kubectl patch cronjob backup-cronjob -p '{"spec":{"suspend":false}}'
```

---

# Step 17: Delete CronJob

```bash
kubectl delete -f backup-cronjob.yaml
```

Expected output:

```text
cronjob.batch "backup-cronjob" deleted
```

---

# Step 18: Delete KIND Cluster

```bash
kind delete cluster --name cronjob-cluster
```

Expected output:

```text
Deleting cluster "cronjob-cluster" ...
```

---

# Expected Outcome

- KIND Kubernetes cluster created successfully
- Kubernetes CronJob configured properly
- Automated scheduled Jobs executed successfully
- CronJob monitoring operational
- Scheduled task automation achieved

---

# Real Industry Use Cases

| Team | Use Case |
|---|---|
| DevOps Team | Automated maintenance tasks |
| Platform Engineering Team | Scheduled infrastructure jobs |
| SRE Team | Automated monitoring scripts |
| Data Engineering Team | Batch data processing |
| Enterprise Teams | Database backup automation |

---

# Skills Covered

- Kubernetes CronJobs
- Scheduled Jobs
- KIND
- kubectl
- Kubernetes YAML
- Job Automation
- Batch Processing
- Container Orchestration
- DevOps Automation

---

# Expected Project Structure

```text
k8s-cronjob-project/
└── backup-cronjob.yaml
```

---

# Complete Command Sequence

```bash
sudo apt update && sudo apt upgrade -y

sudo apt install docker.io -y

sudo systemctl enable docker

sudo systemctl start docker

docker --version

sudo usermod -aG docker $USER

newgrp docker

sudo apt install -y kubectl

kubectl version --client

curl -Lo ./kind https://kind.sigs.k8s.io/dl/latest/kind-linux-amd64

chmod +x ./kind

sudo mv ./kind /usr/local/bin/kind

kind --version

kind create cluster --name cronjob-cluster

kubectl cluster-info --context kind-cronjob-cluster

kubectl get nodes

mkdir -p ~/k8s-cronjob-project

cd ~/k8s-cronjob-project

nano backup-cronjob.yaml

kubectl apply -f backup-cronjob.yaml

kubectl get cronjobs

kubectl get jobs

kubectl get pods

kubectl logs <pod-name>

kubectl describe cronjob backup-cronjob

kubectl patch cronjob backup-cronjob -p '{"spec":{"suspend":true}}'

kubectl patch cronjob backup-cronjob -p '{"spec":{"suspend":false}}'

kubectl delete -f backup-cronjob.yaml

kind delete cluster --name cronjob-cluster
```