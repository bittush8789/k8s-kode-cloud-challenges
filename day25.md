# Troubleshoot Deployment Issues in Kubernetes

# Normal Problem Statement

The DevOps team at xFusionCorp Industries is facing deployment failures in Kubernetes clusters. Applications are not starting correctly, Pods are crashing, and services are becoming unavailable.

Your task is to troubleshoot Kubernetes deployment issues, identify root causes, fix the problems, and restore healthy application deployments on an Ubuntu-based system.

---

# Scenario-Based Problem Statement

xFusionCorp Industries manages production Kubernetes environments for:

- AI/ML platforms
- Node.js applications
- Java applications
- APIs
- Cloud-native microservices

The engineering team is currently facing critical production issues:

- Pods stuck in CrashLoopBackOff
- ImagePullBackOff errors
- Failed deployments
- Service connectivity failures
- Container startup failures
- Misconfigured YAML manifests

These problems are impacting:

- Customer-facing applications
- Internal dashboards
- CI/CD pipelines
- Monitoring systems

As a DevOps Engineer, your responsibilities are to:

1. Create a Kubernetes cluster.
2. Deploy intentionally broken applications.
3. Diagnose deployment failures.
4. Troubleshoot Pods and Services.
5. Fix YAML misconfigurations.
6. Restore healthy deployments.
7. Build production troubleshooting skills.

---

# Objectives

1. Install Kubernetes tools.
2. Create Kind cluster.
3. Deploy broken applications.
4. Diagnose Kubernetes issues.
5. Fix deployment failures.
6. Verify healthy deployments.
7. Learn production troubleshooting workflows.

---

# Common Kubernetes Deployment Issues

| Issue | Description |
|---|---|
| CrashLoopBackOff | Container repeatedly crashing |
| ImagePullBackOff | Invalid image |
| ErrImagePull | Registry/image issue |
| Pending Pods | Resource scheduling issue |
| Service Unreachable | Incorrect selector/port |
| YAML Errors | Invalid manifest |
| Port Conflicts | Wrong container port |
| Config Errors | Missing environment/config |

---

# Architecture Overview

```text
Broken Deployment
        |
        v
Kubernetes Cluster
        |
        +----------------------+
        |                      |
        v                      v
Pod Failures           Service Failures
        |                      |
        +-----------+----------+
                    |
                    v
             Troubleshooting
                    |
                    v
             Fixed Deployment
```

---

# Solution

# Step 1: Update Ubuntu Packages

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

Verify:

```bash
docker --version
```

---

# Step 3: Install Kind

```bash
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.22.0/kind-linux-amd64
```

Make executable:

```bash
chmod +x ./kind
```

Move binary:

```bash
sudo mv ./kind /usr/local/bin/kind
```

Verify:

```bash
kind --version
```

---

# Step 4: Install kubectl

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s \
https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
```

Make executable:

```bash
chmod +x kubectl
```

Move binary:

```bash
sudo mv kubectl /usr/local/bin/
```

Verify:

```bash
kubectl version --client
```

---

# Step 5: Create Kind Kubernetes Cluster

```bash
kind create cluster --name troubleshoot-cluster
```

Verify cluster:

```bash
kubectl cluster-info
```

---

# Step 6: Verify Nodes

```bash
kubectl get nodes
```

Expected:

```text
STATUS: Ready
```

---

# Step 7: Create Namespace

```bash
kubectl create namespace troubleshooting
```

---

# Step 8: Create Broken Deployment Manifest

```bash
nano broken-deployment.yaml
```

Add the following YAML:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: broken-app
  namespace: troubleshooting

spec:
  replicas: 2

  selector:
    matchLabels:
      app: broken-app

  template:
    metadata:
      labels:
        app: broken-app

    spec:
      containers:
        - name: nginx-container

          image: nginx:invalid-version

          ports:
            - containerPort: 80
```

---

# Step 9: Apply Broken Deployment

```bash
kubectl apply -f broken-deployment.yaml
```

---

# Step 10: Verify Pods

```bash
kubectl get pods -n troubleshooting
```

Expected output:

```text
ImagePullBackOff
```

or

```text
ErrImagePull
```

---

# Step 11: Describe Pod

Get pod name:

```bash
kubectl get pods -n troubleshooting
```

Describe pod:

```bash
kubectl describe pod <pod-name> -n troubleshooting
```

Example:

```bash
kubectl describe pod broken-app-xxxxx -n troubleshooting
```

Expected error:

```text
Failed to pull image
```

---

# Step 12: Check Pod Events

Inside describe output verify:

```text
ErrImagePull
ImagePullBackOff
```

---

# Step 13: Fix Deployment Image

Update deployment:

```bash
kubectl set image deployment/broken-app \
nginx-container=nginx:latest \
-n troubleshooting
```

---

# Step 14: Verify Rollout Status

```bash
kubectl rollout status deployment/broken-app \
-n troubleshooting
```

Expected:

```text
successfully rolled out
```

---

# Step 15: Verify Healthy Pods

```bash
kubectl get pods -n troubleshooting
```

Expected:

```text
Running
```

---

# Step 16: Create Broken Service Manifest

```bash
nano broken-service.yaml
```

Add the following YAML:

```yaml
apiVersion: v1
kind: Service

metadata:
  name: broken-service
  namespace: troubleshooting

spec:
  selector:
    app: wrong-label

  ports:
    - port: 80
      targetPort: 80

  type: NodePort
```

---

# Step 17: Apply Service Manifest

```bash
kubectl apply -f broken-service.yaml
```

---

# Step 18: Verify Service

```bash
kubectl get svc -n troubleshooting
```

---

# Step 19: Check Service Endpoints

```bash
kubectl get endpoints -n troubleshooting
```

Expected:

```text
No endpoints
```

Reason:

```text
Wrong selector label
```

---

# Step 20: Fix Service Selector

Edit service:

```bash
kubectl edit svc broken-service -n troubleshooting
```

Replace:

```yaml
app: wrong-label
```

with:

```yaml
app: broken-app
```

Save and exit.

---

# Step 21: Verify Endpoints Again

```bash
kubectl get endpoints -n troubleshooting
```

Expected:

```text
Pod IPs available
```

---

# Step 22: Verify Service Connectivity

Port forward service:

```bash
kubectl port-forward svc/broken-service \
8080:80 -n troubleshooting
```

---

# Step 23: Access Application

Open browser:

```text
http://localhost:8080
```

Expected:

```text
Welcome to nginx!
```

---

# Step 24: Create CrashLoopBackOff Example

```bash
nano crashloop.yaml
```

Add:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: crash-app
  namespace: troubleshooting

spec:
  replicas: 1

  selector:
    matchLabels:
      app: crash-app

  template:
    metadata:
      labels:
        app: crash-app

    spec:
      containers:
        - name: crash-container

          image: busybox

          command: ["false"]
```

---

# Step 25: Apply CrashLoop Deployment

```bash
kubectl apply -f crashloop.yaml
```

---

# Step 26: Verify CrashLoopBackOff

```bash
kubectl get pods -n troubleshooting
```

Expected:

```text
CrashLoopBackOff
```

---

# Step 27: Check Pod Logs

```bash
kubectl logs <crash-pod-name> -n troubleshooting
```

---

# Step 28: Describe Crash Pod

```bash
kubectl describe pod <crash-pod-name> \
-n troubleshooting
```

Verify:

```text
Container exited
```

---

# Step 29: Fix CrashLoop Deployment

Edit deployment:

```bash
kubectl edit deployment crash-app \
-n troubleshooting
```

Replace:

```yaml
command: ["false"]
```

with:

```yaml
command: ["sleep", "3600"]
```

---

# Step 30: Verify Fixed Deployment

```bash
kubectl rollout status deployment/crash-app \
-n troubleshooting
```

---

# Step 31: Verify Healthy Pods

```bash
kubectl get pods -n troubleshooting
```

Expected:

```text
Running
```

---

# Step 32: Verify All Resources

```bash
kubectl get all -n troubleshooting
```

---

# Step 33: Troubleshooting Commands Summary

| Command | Purpose |
|---|---|
| kubectl get pods | Check Pod status |
| kubectl describe pod | Detailed Pod info |
| kubectl logs | Container logs |
| kubectl get events | Cluster events |
| kubectl rollout status | Deployment rollout |
| kubectl get endpoints | Service endpoints |
| kubectl edit | Fix live resources |

---

# Step 34: Delete Resources

```bash
kubectl delete deployment broken-app -n troubleshooting

kubectl delete deployment crash-app -n troubleshooting

kubectl delete svc broken-service -n troubleshooting
```

---

# Step 35: Delete Namespace

```bash
kubectl delete namespace troubleshooting
```

---

# Step 36: Delete Kind Cluster

```bash
kind delete cluster --name troubleshoot-cluster
```

---

# Manifest Files

# broken-deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: broken-app
  namespace: troubleshooting

spec:
  replicas: 2

  selector:
    matchLabels:
      app: broken-app

  template:
    metadata:
      labels:
        app: broken-app

    spec:
      containers:
        - name: nginx-container

          image: nginx:invalid-version

          ports:
            - containerPort: 80
```

---

# broken-service.yaml

```yaml
apiVersion: v1
kind: Service

metadata:
  name: broken-service
  namespace: troubleshooting

spec:
  selector:
    app: wrong-label

  ports:
    - port: 80
      targetPort: 80

  type: NodePort
```

---

# crashloop.yaml

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: crash-app
  namespace: troubleshooting

spec:
  replicas: 1

  selector:
    matchLabels:
      app: crash-app

  template:
    metadata:
      labels:
        app: crash-app

    spec:
      containers:
        - name: crash-container

          image: busybox

          command: ["false"]
```

---

# Expected Outcome

- Kind Kubernetes cluster created successfully
- Deployment failures diagnosed
- ImagePullBackOff issue resolved
- Service connectivity issue resolved
- CrashLoopBackOff issue fixed
- Kubernetes troubleshooting workflow implemented
- Production-grade debugging skills developed

---

# Real Industry Use Cases

| Use Case | Description |
|---|---|
| Production Outages | Pod recovery |
| CI/CD Failures | Deployment debugging |
| Kubernetes Operations | Cluster troubleshooting |
| DevOps | Incident resolution |
| Platform Engineering | Root cause analysis |

---

# Skills Covered

- Kubernetes Troubleshooting
- CrashLoopBackOff Debugging
- ImagePullBackOff Resolution
- Service Troubleshooting
- Pod Logs Analysis
- Kubernetes Events
- DevOps Debugging
- Incident Management

---

# Real Enterprise Workflow

```text
Application Failure
        |
        v
Check Pods
        |
        v
Describe Resources
        |
        v
Analyze Logs & Events
        |
        v
Fix Configuration
        |
        v
Restore Application
```

---

# Complete Command Sequence

```bash
sudo apt update && sudo apt upgrade -y

sudo apt install docker.io -y

sudo systemctl enable docker

sudo systemctl start docker

docker --version

curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.22.0/kind-linux-amd64

chmod +x ./kind

sudo mv ./kind /usr/local/bin/kind

kind --version

curl -LO "https://dl.k8s.io/release/$(curl -L -s \
https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

chmod +x kubectl

sudo mv kubectl /usr/local/bin/

kubectl version --client

kind create cluster --name troubleshoot-cluster

kubectl cluster-info

kubectl get nodes

kubectl create namespace troubleshooting

nano broken-deployment.yaml

kubectl apply -f broken-deployment.yaml

kubectl get pods -n troubleshooting

kubectl describe pod <pod-name> -n troubleshooting

kubectl set image deployment/broken-app \
nginx-container=nginx:latest \
-n troubleshooting

kubectl rollout status deployment/broken-app \
-n troubleshooting

nano broken-service.yaml

kubectl apply -f broken-service.yaml

kubectl get endpoints -n troubleshooting

kubectl edit svc broken-service -n troubleshooting

kubectl port-forward svc/broken-service \
8080:80 -n troubleshooting

nano crashloop.yaml

kubectl apply -f crashloop.yaml

kubectl logs <crash-pod-name> -n troubleshooting

kubectl describe pod <crash-pod-name> \
-n troubleshooting

kubectl edit deployment crash-app \
-n troubleshooting

kubectl get all -n troubleshooting

kubectl delete namespace troubleshooting

kind delete cluster --name troubleshoot-cluster
```