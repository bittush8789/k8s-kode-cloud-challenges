# Day 9: Create Countdown Job in Kubernetes using KIND

# Normal Problem Statement

The DevOps team at xFusionCorp Industries wants to execute one-time batch processing tasks inside Kubernetes using Kubernetes Jobs.

Your task is to create a Kubernetes cluster (if not already available), deploy a Countdown Job using Kubernetes manifest files, and verify Job execution on an Ubuntu system.

---

# Scenario-Based Problem Statement

xFusionCorp Industries manages multiple enterprise workloads inside Kubernetes clusters.

The organization frequently runs one-time tasks such as:

- Database migrations
- Batch processing
- ETL jobs
- Report generation
- ML preprocessing tasks
- Maintenance scripts

Currently, these operations are executed manually, causing several operational issues:

- Inconsistent execution
- Human errors
- Failed task recovery problems
- Lack of centralized job management
- No automated retry mechanism

To solve these problems, the organization wants to use Kubernetes Jobs for reliable one-time task execution.

As a DevOps Engineer, your responsibility is to:

1. Create a Kubernetes cluster if one does not exist.
2. Configure Kubernetes Jobs.
3. Execute countdown batch tasks.
4. Verify Job completion and logs.

---

# Objectives

1. Install Docker, kubectl, and KIND.
2. Create a Kubernetes cluster using KIND.
3. Create Kubernetes Job manifest files.
4. Execute batch Jobs.
5. Verify Job execution and completion.
6. Monitor Pod logs.
7. Delete Jobs safely.

---

# Architecture Overview

```text
KIND Kubernetes Cluster
        |
        v
Kubernetes Job
        |
        v
Temporary Pod
        |
        v
Countdown Task Execution
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
kind create cluster --name job-cluster
```

Expected output:

```text
Creating cluster "job-cluster" ...
Cluster creation complete.
```

---

# Step 6: Verify Kubernetes Cluster

```bash
kubectl cluster-info --context kind-job-cluster
```

Check nodes:

```bash
kubectl get nodes
```

Expected output:

```text
NAME                         STATUS   ROLES
job-cluster-control-plane   Ready    control-plane
```

---

# Step 7: Create Kubernetes Project Directory

```bash
mkdir -p ~/k8s-job-project

cd ~/k8s-job-project
```

---

# Step 8: Create Countdown Job Manifest File

```bash
nano countdown-job.yaml
```

Add the following configuration:

```yaml
apiVersion: batch/v1
kind: Job

metadata:
  name: countdown-job

spec:
  backoffLimit: 2

  template:
    spec:
      containers:
        - name: countdown-container

          image: busybox

          command:
            - /bin/sh
            - -c
            - |
              for i in 10 9 8 7 6 5 4 3 2 1
              do
                echo "Countdown: $i"
                sleep 1
              done

              echo "Countdown completed successfully"

      restartPolicy: Never
```

---

# Complete Countdown Job Manifest File

```yaml
apiVersion: batch/v1
kind: Job

metadata:
  name: countdown-job

spec:
  backoffLimit: 2

  template:
    spec:
      containers:
        - name: countdown-container

          image: busybox

          command:
            - /bin/sh
            - -c
            - |
              for i in 10 9 8 7 6 5 4 3 2 1
              do
                echo "Countdown: $i"
                sleep 1
              done

              echo "Countdown completed successfully"

      restartPolicy: Never
```

---

# Manifest Explanation

| Field | Description |
|---|---|
| kind: Job | Creates one-time batch Job |
| backoffLimit | Retry count if Job fails |
| restartPolicy: Never | Pod will not restart automatically |
| busybox | Lightweight Linux container |
| command | Executes countdown script |

---

# Step 9: Deploy Countdown Job

```bash
kubectl apply -f countdown-job.yaml
```

Expected output:

```text
job.batch/countdown-job created
```

---

# Step 10: Verify Job Status

```bash
kubectl get jobs
```

Expected output:

```text
NAME             COMPLETIONS   DURATION
countdown-job    1/1           12s
```

---

# Step 11: Verify Running Pods

```bash
kubectl get pods
```

Expected output during execution:

```text
NAME                    READY   STATUS
countdown-job-xxxxx     1/1     Running
```

Expected output after completion:

```text
NAME                    READY   STATUS
countdown-job-xxxxx     0/1     Completed
```

---

# Step 12: View Countdown Logs

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
kubectl logs countdown-job-xxxxx
```

Expected output:

```text
Countdown: 10
Countdown: 9
Countdown: 8
Countdown: 7
Countdown: 6
Countdown: 5
Countdown: 4
Countdown: 3
Countdown: 2
Countdown: 1
Countdown completed successfully
```

---

# Step 13: Describe Job

```bash
kubectl describe job countdown-job
```

Displays:

- Completion status
- Pod information
- Events
- Retry configuration

---

# Step 14: Verify Pod Details

```bash
kubectl get pods -o wide
```

Expected output:

```text
NAME                    STATUS      IP
countdown-job-xxxxx     Completed   10.244.0.5
```

---

# Step 15: Delete Job

```bash
kubectl delete -f countdown-job.yaml
```

Expected output:

```text
job.batch "countdown-job" deleted
```

---

# Step 16: Delete KIND Cluster

```bash
kind delete cluster --name job-cluster
```

Expected output:

```text
Deleting cluster "job-cluster" ...
```

---

# Expected Outcome

- KIND Kubernetes cluster created successfully
- Kubernetes Job configured correctly
- Countdown batch task executed successfully
- Job completion verified
- Pod logs monitored successfully
- Batch processing automation achieved

---

# Real Industry Use Cases

| Team | Use Case |
|---|---|
| DevOps Team | Maintenance automation |
| Data Engineering Team | ETL batch processing |
| ML Engineering Team | ML preprocessing tasks |
| SRE Team | Infrastructure maintenance |
| Enterprise Teams | One-time operational tasks |

---

# Skills Covered

- Kubernetes Jobs
- Batch Processing
- KIND
- kubectl
- Kubernetes YAML
- Job Automation
- Pod Logs
- Container Orchestration
- DevOps Automation

---

# Expected Project Structure

```text
k8s-job-project/
└── countdown-job.yaml
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

kind create cluster --name job-cluster

kubectl cluster-info --context kind-job-cluster

kubectl get nodes

mkdir -p ~/k8s-job-project

cd ~/k8s-job-project

nano countdown-job.yaml

kubectl apply -f countdown-job.yaml

kubectl get jobs

kubectl get pods

kubectl logs <pod-name>

kubectl describe job countdown-job

kubectl get pods -o wide

kubectl delete -f countdown-job.yaml

kind delete cluster --name job-cluster
```