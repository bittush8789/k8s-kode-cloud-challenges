# Rolling Updates and Rolling Back Deployments in Kubernetes

# Normal Problem Statement

The DevOps team at xFusionCorp Industries wants to perform rolling updates on Kubernetes Deployments and safely roll back to previous versions if deployment failures occur.

Your task is to create a Kubernetes cluster, deploy an application, perform rolling updates, verify deployment history, and roll back failed deployments on an Ubuntu-based system.

---

# Scenario-Based Problem Statement

xFusionCorp Industries is deploying enterprise applications on Kubernetes.

The organization manages:

- AI platforms
- Web applications
- APIs
- Microservices
- Internal dashboards

The engineering team currently faces several challenges:

- Downtime during deployments
- Failed production releases
- Unsafe manual upgrades
- No rollback strategy
- Unstable application versions

To solve these problems, the organization wants to implement:

- Rolling Updates
- Zero-downtime deployments
- Deployment history tracking
- Safe rollback mechanisms

As a DevOps Engineer, your responsibilities are to:

1. Create a Kubernetes cluster using Kind.
2. Deploy applications using Kubernetes Deployments.
3. Perform rolling updates.
4. Monitor rollout status.
5. Roll back failed deployments.
6. Verify deployment history.
7. Implement production-grade deployment workflows.

---

# Objectives

1. Install Kubernetes tools.
2. Create a Kind Kubernetes cluster.
3. Deploy Nginx application.
4. Perform rolling updates.
5. Monitor deployment rollout.
6. Roll back to previous versions.
7. Implement zero-downtime deployment strategies.

---

# What is a Rolling Update?

A Rolling Update gradually replaces old Pods with new Pods without downtime.

Benefits:

- Zero downtime
- Safer deployments
- Controlled updates
- Production stability

---

# What is Rollback?

Rollback restores a previous working deployment version when a deployment fails.

Benefits:

- Fast recovery
- Reduced downtime
- Safer production releases

---

# Architecture Overview

```text
Old Version Pods
        |
        v
Rolling Update
        |
        v
New Version Pods
        |
        +----------------+
        |                |
        v                v
Successful         Failed Deployment
        |                |
        v                v
Production        Rollback
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
kind create cluster --name rolling-update-cluster
```

Verify cluster:

```bash
kubectl cluster-info
```

Expected output:

```text
Kubernetes control plane is running
```

---

# Step 6: Verify Nodes

```bash
kubectl get nodes
```

Expected output:

```text
NAME                                    STATUS   ROLES
rolling-update-cluster-control-plane    Ready    control-plane
```

---

# Step 7: Create Deployment Manifest

```bash
nano nginx-deployment.yaml
```

Add the following YAML:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: nginx-deployment

spec:
  replicas: 3

  strategy:
    type: RollingUpdate

    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1

  selector:
    matchLabels:
      app: nginx

  template:
    metadata:
      labels:
        app: nginx

    spec:
      containers:
        - name: nginx-container

          image: nginx:1.25

          ports:
            - containerPort: 80
```

---

# Step 8: Apply Deployment Manifest

```bash
kubectl apply -f nginx-deployment.yaml
```

Expected output:

```text
deployment.apps/nginx-deployment created
```

---

# Step 9: Verify Deployment

```bash
kubectl get deployments
```

Expected output:

```text
NAME               READY   UP-TO-DATE   AVAILABLE
nginx-deployment   3/3     3            3
```

---

# Step 10: Verify Pods

```bash
kubectl get pods
```

Expected:

```text
3 Running Pods
```

---

# Step 11: Inspect Deployment

```bash
kubectl describe deployment nginx-deployment
```

Verify:

- RollingUpdate strategy
- Replica count
- Image version

---

# Step 12: Expose Deployment

```bash
kubectl expose deployment nginx-deployment \
--type=NodePort \
--port=80
```

Verify service:

```bash
kubectl get svc
```

---

# Step 13: Perform Rolling Update

Update image version:

```bash
kubectl set image deployment/nginx-deployment \
nginx-container=nginx:1.26
```

Expected output:

```text
deployment.apps/nginx-deployment image updated
```

---

# Step 14: Monitor Rolling Update

```bash
kubectl rollout status deployment/nginx-deployment
```

Expected output:

```text
deployment "nginx-deployment" successfully rolled out
```

---

# Step 15: Watch Pods During Update

```bash
kubectl get pods -w
```

Observe:

- Old Pods terminate gradually
- New Pods created gradually
- No downtime

Press:

```text
CTRL + C
```

to stop watching.

---

# Step 16: Verify Updated Image

```bash
kubectl describe deployment nginx-deployment
```

Verify:

```text
Image: nginx:1.26
```

---

# Step 17: Check Deployment History

```bash
kubectl rollout history deployment/nginx-deployment
```

Expected output:

```text
REVISION  CHANGE-CAUSE
1         Initial deployment
2         Image updated
```

---

# Step 18: Simulate Failed Deployment

Update deployment with invalid image:

```bash
kubectl set image deployment/nginx-deployment \
nginx-container=nginx:invalid-version
```

---

# Step 19: Monitor Failed Rollout

```bash
kubectl rollout status deployment/nginx-deployment
```

Expected:

```text
Deployment failed
```

---

# Step 20: Verify Pod Failure

```bash
kubectl get pods
```

Expected:

```text
ImagePullBackOff
```

or

```text
ErrImagePull
```

---

# Step 21: Roll Back Deployment

```bash
kubectl rollout undo deployment/nginx-deployment
```

Expected output:

```text
deployment.apps/nginx-deployment rolled back
```

---

# Step 22: Verify Rollback Status

```bash
kubectl rollout status deployment/nginx-deployment
```

Expected output:

```text
successfully rolled out
```

---

# Step 23: Verify Restored Pods

```bash
kubectl get pods
```

Expected:

```text
All Pods Running
```

---

# Step 24: Verify Deployment Image

```bash
kubectl describe deployment nginx-deployment
```

Expected:

```text
Image: nginx:1.26
```

Rollback restored working version.

---

# Step 25: View Detailed Rollout History

```bash
kubectl rollout history deployment/nginx-deployment
```

---

# Step 26: Roll Back to Specific Revision

View revisions:

```bash
kubectl rollout history deployment/nginx-deployment
```

Rollback:

```bash
kubectl rollout undo deployment/nginx-deployment \
--to-revision=1
```

---

# Step 27: Verify Specific Rollback

```bash
kubectl describe deployment nginx-deployment
```

Expected:

```text
Image: nginx:1.25
```

---

# Step 28: Delete Service

```bash
kubectl delete svc nginx-deployment
```

---

# Step 29: Delete Deployment

```bash
kubectl delete deployment nginx-deployment
```

---

# Step 30: Delete Cluster

```bash
kind delete cluster --name rolling-update-cluster
```

Expected output:

```text
Deleting cluster "rolling-update-cluster"
```

---

# Manifest File

# nginx-deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: nginx-deployment

spec:
  replicas: 3

  strategy:
    type: RollingUpdate

    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1

  selector:
    matchLabels:
      app: nginx

  template:
    metadata:
      labels:
        app: nginx

    spec:
      containers:
        - name: nginx-container

          image: nginx:1.25

          ports:
            - containerPort: 80
```

---

# Expected Outcome

- Kind Kubernetes cluster created successfully
- Rolling updates configured correctly
- Zero-downtime deployment achieved
- Deployment history tracked successfully
- Failed deployment simulated
- Rollback completed successfully
- Enterprise-grade deployment workflow implemented

---

# Real Industry Use Cases

| Use Case | Description |
|---|---|
| Production Releases | Zero downtime |
| Microservices | Version upgrades |
| AI Platforms | Safe model deployment |
| SaaS Applications | Controlled rollouts |
| Enterprise Apps | Rollback recovery |

---

# Skills Covered

- Kubernetes Deployments
- Rolling Updates
- Rollback Strategies
- Deployment History
- Zero-Downtime Deployments
- Kubernetes Troubleshooting
- DevOps
- Production Release Management

---

# Real Enterprise Workflow

```text
New Application Version
         |
         v
Rolling Update
         |
         +----------------+
         |                |
         v                v
Success           Deployment Failure
         |                |
         v                v
Production        Rollback
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

kind create cluster --name rolling-update-cluster

kubectl cluster-info

kubectl get nodes

nano nginx-deployment.yaml

kubectl apply -f nginx-deployment.yaml

kubectl get deployments

kubectl get pods

kubectl describe deployment nginx-deployment

kubectl expose deployment nginx-deployment \
--type=NodePort \
--port=80

kubectl get svc

kubectl set image deployment/nginx-deployment \
nginx-container=nginx:1.26

kubectl rollout status deployment/nginx-deployment

kubectl get pods -w

kubectl rollout history deployment/nginx-deployment

kubectl set image deployment/nginx-deployment \
nginx-container=nginx:invalid-version

kubectl rollout status deployment/nginx-deployment

kubectl get pods

kubectl rollout undo deployment/nginx-deployment

kubectl rollout status deployment/nginx-deployment

kubectl get pods

kubectl rollout undo deployment/nginx-deployment \
--to-revision=1

kubectl delete svc nginx-deployment

kubectl delete deployment nginx-deployment

kind delete cluster --name rolling-update-cluster
```