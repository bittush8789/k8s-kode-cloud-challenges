# Deploy Iron Gallery App on Kubernetes

# Normal Problem Statement

The DevOps team at xFusionCorp Industries wants to deploy the Iron Gallery application on Kubernetes for scalable and highly available production hosting.

Your task is to create a Kubernetes cluster using Kind, deploy the Iron Gallery application using Kubernetes manifests, expose the application using Services, and verify deployment on an Ubuntu-based system.

---

# Scenario-Based Problem Statement

xFusionCorp Industries manages enterprise-grade applications including:

- Internal dashboards
- Web applications
- Image gallery platforms
- Enterprise portals
- Cloud-native applications

The organization currently faces several infrastructure challenges:

- Manual deployments
- Application downtime
- Poor scalability
- Difficult rollback management
- Lack of container orchestration

To modernize infrastructure, the organization wants to deploy the Iron Gallery application on Kubernetes.

As a DevOps Engineer, your responsibilities are to:

1. Create Kubernetes cluster using Kind.
2. Deploy Iron Gallery application.
3. Configure Kubernetes Services.
4. Expose application externally.
5. Scale application Pods.
6. Perform rolling updates and rollbacks.
7. Build enterprise-grade Kubernetes architecture.

---

# Objectives

1. Install Kubernetes tools.
2. Create Kind cluster.
3. Deploy Iron Gallery application.
4. Configure Deployment and Service.
5. Expose application using NodePort.
6. Verify application accessibility.
7. Learn production deployment workflows.

---

# Architecture Overview

```text
                    +----------------------+
                    |        Users         |
                    +----------+-----------+
                               |
                               v
                    +----------------------+
                    |  NodePort Service    |
                    +----------+-----------+
                               |
                               v
                    +----------------------+
                    | Iron Gallery Pods    |
                    +----------+-----------+
                               |
                               v
                    +----------------------+
                    | Kubernetes Cluster   |
                    +----------------------+
```

---

# Real Industry Use Cases

| Use Case | Purpose |
|---|---|
| Web Hosting | Enterprise apps |
| Media Platforms | Image hosting |
| Dashboards | Internal portals |
| Cloud-Native Apps | Kubernetes workloads |
| DevOps Platforms | Container orchestration |

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
kind create cluster --name iron-gallery-cluster
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
kubectl create namespace iron-gallery
```

---

# Step 8: Create Iron Gallery Deployment Manifest

```bash
nano iron-gallery-deployment.yaml
```

Add the following YAML:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: iron-gallery-app
  namespace: iron-gallery

spec:
  replicas: 2

  selector:
    matchLabels:
      app: iron-gallery

  template:
    metadata:
      labels:
        app: iron-gallery

    spec:
      containers:

        - name: iron-gallery-container

          image: kodekloud/irongallery:latest

          ports:
            - containerPort: 80
```

---

# Step 9: Apply Deployment Manifest

```bash
kubectl apply -f iron-gallery-deployment.yaml
```

Expected output:

```text
deployment.apps/iron-gallery-app created
```

---

# Step 10: Verify Deployment

```bash
kubectl get deployments -n iron-gallery
```

Expected:

```text
READY 2/2
```

---

# Step 11: Verify Pods

```bash
kubectl get pods -n iron-gallery
```

Expected:

```text
Running
```

---

# Step 12: Create NodePort Service Manifest

```bash
nano iron-gallery-service.yaml
```

Add the following YAML:

```yaml
apiVersion: v1
kind: Service

metadata:
  name: iron-gallery-service
  namespace: iron-gallery

spec:
  type: NodePort

  selector:
    app: iron-gallery

  ports:
    - port: 80
      targetPort: 80
      nodePort: 32009
```

---

# Step 13: Apply Service Manifest

```bash
kubectl apply -f iron-gallery-service.yaml
```

Expected output:

```text
service/iron-gallery-service created
```

---

# Step 14: Verify Service

```bash
kubectl get svc -n iron-gallery
```

Expected:

```text
iron-gallery-service
```

---

# Step 15: Verify Endpoints

```bash
kubectl get endpoints -n iron-gallery
```

Expected:

```text
Pod IPs visible
```

---

# Step 16: Port Forward Application

```bash
kubectl port-forward svc/iron-gallery-service \
8080:80 -n iron-gallery
```

---

# Step 17: Access Application

Open browser:

```text
http://localhost:8080
```

Expected:

```text
Iron Gallery application page
```

---

# Step 18: Describe Deployment

```bash
kubectl describe deployment iron-gallery-app \
-n iron-gallery
```

---

# Step 19: Describe Service

```bash
kubectl describe svc iron-gallery-service \
-n iron-gallery
```

---

# Step 20: Check Application Logs

Get Pod names:

```bash
kubectl get pods -n iron-gallery
```

Check logs:

```bash
kubectl logs <pod-name> -n iron-gallery
```

---

# Step 21: Scale Application

```bash
kubectl scale deployment iron-gallery-app \
--replicas=4 -n iron-gallery
```

Verify:

```bash
kubectl get deployments -n iron-gallery
```

Expected:

```text
READY 4/4
```

---

# Step 22: Verify Scaled Pods

```bash
kubectl get pods -n iron-gallery
```

Expected:

```text
4 Running Pods
```

---

# Step 23: Perform Rolling Update

Update image version:

```bash
kubectl set image deployment/iron-gallery-app \
iron-gallery-container=kodekloud/irongallery:2.0 \
-n iron-gallery
```

Verify rollout:

```bash
kubectl rollout status deployment/iron-gallery-app \
-n iron-gallery
```

---

# Step 24: Verify Rollout History

```bash
kubectl rollout history deployment/iron-gallery-app \
-n iron-gallery
```

---

# Step 25: Rollback Deployment

```bash
kubectl rollout undo deployment/iron-gallery-app \
-n iron-gallery
```

---

# Step 26: Verify Rollback

```bash
kubectl rollout history deployment/iron-gallery-app \
-n iron-gallery
```

---

# Step 27: Troubleshooting Deployment Issues

# Check Pods

```bash
kubectl get pods -n iron-gallery
```

---

# Check Logs

```bash
kubectl logs <pod-name> -n iron-gallery
```

---

# Describe Pods

```bash
kubectl describe pod <pod-name> \
-n iron-gallery
```

---

# Check Services

```bash
kubectl get svc -n iron-gallery
```

---

# Common Issues

| Issue | Description |
|---|---|
| CrashLoopBackOff | Application crash |
| ImagePullBackOff | Invalid image |
| No Endpoints | Wrong selector |
| Port Error | Wrong targetPort |
| Service Unreachable | Networking issue |

---

# Step 28: Verify Cluster Resources

```bash
kubectl get all -n iron-gallery
```

Expected:

```text
Pods
Services
Deployments
ReplicaSets
```

---

# Step 29: Delete Resources

```bash
kubectl delete deployment iron-gallery-app \
-n iron-gallery

kubectl delete svc iron-gallery-service \
-n iron-gallery
```

---

# Step 30: Delete Namespace

```bash
kubectl delete namespace iron-gallery
```

---

# Step 31: Delete Kind Cluster

```bash
kind delete cluster --name iron-gallery-cluster
```

---

# Manifest Files

# iron-gallery-deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: iron-gallery-app
  namespace: iron-gallery

spec:
  replicas: 2

  selector:
    matchLabels:
      app: iron-gallery

  template:
    metadata:
      labels:
        app: iron-gallery

    spec:
      containers:

        - name: iron-gallery-container

          image: kodekloud/irongallery:latest

          ports:
            - containerPort: 80
```

---

# iron-gallery-service.yaml

```yaml
apiVersion: v1
kind: Service

metadata:
  name: iron-gallery-service
  namespace: iron-gallery

spec:
  type: NodePort

  selector:
    app: iron-gallery

  ports:
    - port: 80
      targetPort: 80
      nodePort: 32009
```

---

# Expected Outcome

- Kind Kubernetes cluster created successfully
- Iron Gallery application deployed
- Kubernetes Service configured
- Application exposed successfully
- Scaling operational
- Rolling updates operational
- Rollback operational
- Enterprise-grade deployment workflow implemented

---

# Skills Covered

- Kubernetes Deployments
- Kubernetes Services
- NodePort Services
- Scaling Applications
- Rolling Updates
- Rollbacks
- Kubernetes Troubleshooting
- DevOps
- Cloud-Native Deployment

---

# Real Enterprise Workflow

```text
Developer Push
       |
       v
Docker Image Build
       |
       v
Container Registry
       |
       v
Kubernetes Deployment
       |
       v
ReplicaSets
       |
       v
Pods
       |
       v
NodePort Service
       |
       v
Users Access Application
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

kind create cluster --name iron-gallery-cluster

kubectl create namespace iron-gallery

kubectl apply -f iron-gallery-deployment.yaml

kubectl get deployments -n iron-gallery

kubectl get pods -n iron-gallery

kubectl apply -f iron-gallery-service.yaml

kubectl get svc -n iron-gallery

kubectl get endpoints -n iron-gallery

kubectl port-forward svc/iron-gallery-service \
8080:80 -n iron-gallery

kubectl describe deployment iron-gallery-app \
-n iron-gallery

kubectl logs <pod-name> -n iron-gallery

kubectl scale deployment iron-gallery-app \
--replicas=4 -n iron-gallery

kubectl set image deployment/iron-gallery-app \
iron-gallery-container=kodekloud/irongallery:2.0 \
-n iron-gallery

kubectl rollout status deployment/iron-gallery-app \
-n iron-gallery

kubectl rollout undo deployment/iron-gallery-app \
-n iron-gallery

kubectl get all -n iron-gallery

kubectl delete namespace iron-gallery

kind delete cluster --name iron-gallery-cluster
```