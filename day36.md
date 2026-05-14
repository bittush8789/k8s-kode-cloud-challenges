# Fix Python App Deployed on Kubernetes Cluster

# Normal Problem Statement

The DevOps team at xFusionCorp Industries has deployed a Python application on Kubernetes, but the application is failing due to configuration and deployment issues.

Your task is to create a Kubernetes cluster using Kind, deploy the broken Python application, troubleshoot deployment failures, fix the issues, and restore the application successfully on an Ubuntu-based system.

---

# Scenario-Based Problem Statement

xFusionCorp Industries manages enterprise-grade applications including:

- Python APIs
- Flask applications
- AI/ML services
- Automation platforms
- Internal microservices

The engineering team recently deployed a Python application to Kubernetes, but the deployment is failing in production.

Current issues include:

- CrashLoopBackOff
- Image configuration issues
- Wrong container ports
- Service connectivity failures
- Incorrect environment variables
- Application startup failures

This outage is impacting:

- Customer-facing APIs
- Internal services
- Monitoring systems
- AI platforms

As a DevOps Engineer, your responsibilities are to:

1. Create Kubernetes cluster using Kind.
2. Deploy broken Python application.
3. Diagnose Kubernetes failures.
4. Fix deployment issues.
5. Restore application connectivity.
6. Verify healthy workloads.
7. Implement production-grade troubleshooting workflow.

---

# Objectives

1. Install Kubernetes tools.
2. Create Kind cluster.
3. Deploy broken Python app.
4. Troubleshoot deployment issues.
5. Fix application failures.
6. Verify successful deployment.
7. Learn real-world Kubernetes debugging.

---

# Architecture Overview

```text
                  +----------------------+
                  |        Users         |
                  +----------+-----------+
                             |
                             v
                  +----------------------+
                  | Kubernetes Service   |
                  +----------+-----------+
                             |
                             v
                  +----------------------+
                  | Python App Pods      |
                  +----------+-----------+
                             |
                             v
                  +----------------------+
                  | Kubernetes Cluster   |
                  +----------------------+
```

---

# Common Python App Issues in Kubernetes

| Issue | Description |
|---|---|
| CrashLoopBackOff | App repeatedly crashing |
| Wrong Port | Service cannot connect |
| ImagePullBackOff | Invalid image |
| Missing Env Vars | App startup failure |
| Service Failure | Wrong selectors |

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
kind create cluster --name python-fix-cluster
```

Verify:

```bash
kubectl cluster-info
```

Expected:

```text
Kubernetes control plane is running
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
kubectl create namespace python-app
```

---

# Step 8: Create Broken Python Deployment Manifest

```bash
nano broken-python-app.yaml
```

Add the following YAML:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: python-app
  namespace: python-app

spec:
  replicas: 2

  selector:
    matchLabels:
      app: python-app

  template:
    metadata:
      labels:
        app: python-app

    spec:
      containers:

        - name: python-container

          image: python:wrongtag

          command:
            - python

          args:
            - -m
            - http.server
            - "5000"

          ports:
            - containerPort: 3000
```

---

# Step 9: Apply Broken Deployment

```bash
kubectl apply -f broken-python-app.yaml
```

---

# Step 10: Verify Pods

```bash
kubectl get pods -n python-app
```

Expected:

```text
ImagePullBackOff
```

---

# Step 11: Describe Pod

```bash
kubectl describe pod <pod-name> \
-n python-app
```

Expected:

```text
ErrImagePull
```

---

# Step 12: Fix Invalid Image

```bash
kubectl set image deployment/python-app \
python-container=python:3.11 \
-n python-app
```

---

# Step 13: Verify Rollout

```bash
kubectl rollout status deployment/python-app \
-n python-app
```

---

# Step 14: Verify Pods Again

```bash
kubectl get pods -n python-app
```

Expected:

```text
Running
```

---

# Step 15: Create Broken Service Manifest

```bash
nano broken-python-service.yaml
```

Add:

```yaml
apiVersion: v1
kind: Service

metadata:
  name: python-service
  namespace: python-app

spec:
  type: NodePort

  selector:
    app: wrong-label

  ports:
    - port: 80
      targetPort: 80
      nodePort: 32010
```

---

# Step 16: Apply Service Manifest

```bash
kubectl apply -f broken-python-service.yaml
```

---

# Step 17: Verify Endpoints

```bash
kubectl get endpoints -n python-app
```

Expected:

```text
No endpoints
```

---

# Step 18: Fix Service Selector

```bash
kubectl edit svc python-service \
-n python-app
```

Replace:

```yaml
app: wrong-label
```

with:

```yaml
app: python-app
```

---

# Step 19: Fix Wrong Target Port

Still inside service YAML replace:

```yaml
targetPort: 80
```

with:

```yaml
targetPort: 3000
```

Save and exit.

---

# Step 20: Verify Service

```bash
kubectl get svc -n python-app
```

---

# Step 21: Verify Endpoints Again

```bash
kubectl get endpoints -n python-app
```

Expected:

```text
Pod IPs visible
```

---

# Step 22: Port Forward Service

```bash
kubectl port-forward svc/python-service \
8080:80 -n python-app
```

---

# Step 23: Access Python Application

Open browser:

```text
http://localhost:8080
```

Expected:

```text
Python HTTP server page
```

---

# Step 24: Verify Deployment Details

```bash
kubectl describe deployment python-app \
-n python-app
```

---

# Step 25: Check Application Logs

Get Pod names:

```bash
kubectl get pods -n python-app
```

Check logs:

```bash
kubectl logs <pod-name> -n python-app
```

Expected:

```text
Serving HTTP on 0.0.0.0 port 5000
```

---

# Step 26: Fix Container Port Mismatch

Edit deployment:

```bash
kubectl edit deployment python-app \
-n python-app
```

Replace:

```yaml
containerPort: 3000
```

with:

```yaml
containerPort: 5000
```

Save and exit.

---

# Step 27: Verify Rollout

```bash
kubectl rollout status deployment/python-app \
-n python-app
```

---

# Step 28: Scale Deployment

```bash
kubectl scale deployment python-app \
--replicas=4 -n python-app
```

Verify:

```bash
kubectl get deployments -n python-app
```

Expected:

```text
READY 4/4
```

---

# Step 29: Verify Pods

```bash
kubectl get pods -n python-app
```

Expected:

```text
4 Running Pods
```

---

# Step 30: Perform Rolling Update

Update Python version:

```bash
kubectl set image deployment/python-app \
python-container=python:3.12 \
-n python-app
```

---

# Step 31: Verify Rollout

```bash
kubectl rollout status deployment/python-app \
-n python-app
```

---

# Step 32: Verify Rollout History

```bash
kubectl rollout history deployment/python-app \
-n python-app
```

---

# Step 33: Rollback Deployment

```bash
kubectl rollout undo deployment/python-app \
-n python-app
```

---

# Step 34: Troubleshooting Commands

| Command | Purpose |
|---|---|
| kubectl get pods | Pod status |
| kubectl describe pod | Detailed pod info |
| kubectl logs | Container logs |
| kubectl get svc | Service status |
| kubectl get endpoints | Service endpoints |
| kubectl rollout status | Deployment rollout |

---

# Step 35: Common Kubernetes Errors

| Error | Meaning |
|---|---|
| CrashLoopBackOff | App crash |
| ImagePullBackOff | Invalid image |
| No Endpoints | Wrong selector |
| Connection Refused | Wrong port |
| Pending | Scheduling issue |

---

# Step 36: Verify Cluster Resources

```bash
kubectl get all -n python-app
```

Expected:

```text
Pods
Services
Deployments
ReplicaSets
```

---

# Step 37: Delete Resources

```bash
kubectl delete deployment python-app \
-n python-app

kubectl delete svc python-service \
-n python-app
```

---

# Step 38: Delete Namespace

```bash
kubectl delete namespace python-app
```

---

# Step 39: Delete Kind Cluster

```bash
kind delete cluster --name python-fix-cluster
```

---

# Manifest Files

# broken-python-app.yaml

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: python-app
  namespace: python-app

spec:
  replicas: 2

  selector:
    matchLabels:
      app: python-app

  template:
    metadata:
      labels:
        app: python-app

    spec:
      containers:

        - name: python-container

          image: python:3.11

          command:
            - python

          args:
            - -m
            - http.server
            - "5000"

          ports:
            - containerPort: 5000
```

---

# broken-python-service.yaml

```yaml
apiVersion: v1
kind: Service

metadata:
  name: python-service
  namespace: python-app

spec:
  type: NodePort

  selector:
    app: python-app

  ports:
    - port: 80
      targetPort: 5000
      nodePort: 32010
```

---

# Expected Outcome

- Kind Kubernetes cluster created successfully
- Broken Python app diagnosed successfully
- ImagePullBackOff issue fixed
- Service selector issue fixed
- Port mismatch issue fixed
- Python application restored successfully
- Scaling operational
- Rolling updates operational
- Enterprise-grade troubleshooting workflow implemented

---

# Skills Covered

- Kubernetes Troubleshooting
- Python Applications on Kubernetes
- Deployments
- Services
- NodePort
- Rolling Updates
- Rollbacks
- DevOps
- Cloud-Native Debugging

---

# Real Enterprise Workflow

```text
Broken Application
        |
        v
Check Pod Status
        |
        v
Describe Deployment
        |
        v
Check Logs & Events
        |
        v
Identify Root Cause
        |
        v
Fix Configuration
        |
        v
Redeploy Application
        |
        v
Verify Application Health
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

kind create cluster --name python-fix-cluster

kubectl create namespace python-app

kubectl apply -f broken-python-app.yaml

kubectl get pods -n python-app

kubectl describe pod <pod-name> \
-n python-app

kubectl set image deployment/python-app \
python-container=python:3.11 \
-n python-app

kubectl apply -f broken-python-service.yaml

kubectl edit svc python-service \
-n python-app

kubectl get endpoints -n python-app

kubectl port-forward svc/python-service \
8080:80 -n python-app

kubectl logs <pod-name> -n python-app

kubectl edit deployment python-app \
-n python-app

kubectl scale deployment python-app \
--replicas=4 -n python-app

kubectl rollout history deployment/python-app \
-n python-app

kubectl rollout undo deployment/python-app \
-n python-app

kubectl get all -n python-app

kubectl delete namespace python-app

kind delete cluster --name python-fix-cluster
```