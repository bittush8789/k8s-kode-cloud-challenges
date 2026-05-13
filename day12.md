# Day 12: Update Deployment and Service in Kubernetes using KIND

# Normal Problem Statement

The DevOps team at xFusionCorp Industries wants to update Kubernetes Deployments and Services to deploy new application versions and expose applications reliably inside the Kubernetes cluster.

Your task is to create a Kubernetes cluster (if not already available), deploy an application using Kubernetes Deployment and Service manifests, update the application version, and verify Service connectivity on an Ubuntu system.

---

# Scenario-Based Problem Statement

xFusionCorp Industries manages production-grade applications inside Kubernetes clusters.

The organization frequently performs:

- Application version upgrades
- Configuration changes
- Service exposure updates
- Backend container updates
- Rolling deployment operations

Currently, the DevOps team faces several challenges:

- Downtime during application updates
- Inconsistent Service routing
- Manual Deployment management
- Difficulty updating running applications
- Lack of scalable deployment workflows

To solve these issues, the organization wants to implement Kubernetes Deployment and Service updates with rolling update strategies.

As a DevOps Engineer, your responsibility is to:

1. Create a Kubernetes cluster if one does not exist.
2. Deploy applications using Kubernetes Deployments.
3. Expose applications using Services.
4. Update Deployment container versions.
5. Verify Service connectivity after updates.

---

# Objectives

1. Install Docker, kubectl, and KIND.
2. Create a Kubernetes cluster using KIND.
3. Create Deployment and Service manifest files.
4. Deploy applications into Kubernetes.
5. Update Deployment container images.
6. Verify rolling updates.
7. Validate Service accessibility.
8. Delete resources safely.

---

# Architecture Overview

```text
KIND Kubernetes Cluster
        |
        v
Deployment
        |
        v
ReplicaSet
        |
        v
Pods
        |
        v
Service
        |
        v
Application Access
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
kind create cluster --name deployment-service-cluster
```

Expected output:

```text
Creating cluster "deployment-service-cluster" ...
Cluster creation complete.
```

---

# Step 6: Verify Kubernetes Cluster

```bash
kubectl cluster-info --context kind-deployment-service-cluster
```

Check nodes:

```bash
kubectl get nodes
```

Expected output:

```text
NAME                                         STATUS   ROLES
deployment-service-cluster-control-plane    Ready    control-plane
```

---

# Step 7: Create Kubernetes Project Directory

```bash
mkdir -p ~/k8s-deployment-service-project

cd ~/k8s-deployment-service-project
```

---

# Step 8: Create Deployment Manifest File

```bash
nano nginx-deployment.yaml
```

Add the following configuration:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: nginx-deployment

spec:
  replicas: 3

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

# Complete Deployment Manifest File

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: nginx-deployment

spec:
  replicas: 3

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

# Step 9: Create Service Manifest File

```bash
nano nginx-service.yaml
```

Add the following configuration:

```yaml
apiVersion: v1
kind: Service

metadata:
  name: nginx-service

spec:
  selector:
    app: nginx

  ports:
    - protocol: TCP
      port: 80
      targetPort: 80

  type: NodePort
```

---

# Complete Service Manifest File

```yaml
apiVersion: v1
kind: Service

metadata:
  name: nginx-service

spec:
  selector:
    app: nginx

  ports:
    - protocol: TCP
      port: 80
      targetPort: 80

  type: NodePort
```

---

# Step 10: Deploy Deployment and Service

Deploy Deployment:

```bash
kubectl apply -f nginx-deployment.yaml
```

Deploy Service:

```bash
kubectl apply -f nginx-service.yaml
```

Expected output:

```text
deployment.apps/nginx-deployment created
service/nginx-service created
```

---

# Step 11: Verify Deployment

```bash
kubectl get deployments
```

Expected output:

```text
NAME               READY   UP-TO-DATE   AVAILABLE
nginx-deployment   3/3     3            3
```

---

# Step 12: Verify Pods

```bash
kubectl get pods
```

Expected output:

```text
NAME                                READY   STATUS
nginx-deployment-xxxxxxxxxx-abcde   1/1     Running
nginx-deployment-xxxxxxxxxx-fghij   1/1     Running
nginx-deployment-xxxxxxxxxx-klmno   1/1     Running
```

---

# Step 13: Verify Service

```bash
kubectl get services
```

Expected output:

```text
NAME            TYPE       CLUSTER-IP      PORT(S)
nginx-service   NodePort   10.96.120.10   80:30001/TCP
```

---

# Step 14: Access Application

Use port forwarding:

```bash
kubectl port-forward service/nginx-service 8080:80
```

Access application:

```text
http://localhost:8080
```

---

# Step 15: Update Deployment Version

Update container image:

```bash
kubectl set image deployment/nginx-deployment nginx-container=nginx:1.26
```

Expected output:

```text
deployment.apps/nginx-deployment image updated
```

---

# Step 16: Monitor Rolling Update

```bash
kubectl rollout status deployment/nginx-deployment
```

Expected output:

```text
deployment "nginx-deployment" successfully rolled out
```

---

# Step 17: Verify Updated Deployment

```bash
kubectl describe deployment nginx-deployment
```

Expected image:

```text
Image: nginx:1.26
```

---

# Step 18: Verify Service Connectivity After Update

```bash
kubectl get endpoints nginx-service
```

Expected output:

```text
NAME            ENDPOINTS
nginx-service   10.244.0.x:80
```

Test Service again:

```bash
kubectl port-forward service/nginx-service 8080:80
```

Access:

```text
http://localhost:8080
```

---

# Step 19: Scale Deployment

```bash
kubectl scale deployment nginx-deployment --replicas=5
```

Verify scaling:

```bash
kubectl get deployments
```

Expected output:

```text
NAME               READY   UP-TO-DATE   AVAILABLE
nginx-deployment   5/5     5            5
```

---

# Step 20: Delete Resources

Delete Service:

```bash
kubectl delete -f nginx-service.yaml
```

Delete Deployment:

```bash
kubectl delete -f nginx-deployment.yaml
```

---

# Step 21: Delete KIND Cluster

```bash
kind delete cluster --name deployment-service-cluster
```

Expected output:

```text
Deleting cluster "deployment-service-cluster" ...
```

---

# Expected Outcome

- KIND Kubernetes cluster created successfully
- Deployment and Service configured correctly
- Rolling update completed successfully
- Service connectivity maintained during update
- Scalable application deployment achieved
- Kubernetes application lifecycle operational

---

# Real Industry Use Cases

| Team | Use Case |
|---|---|
| DevOps Team | Application lifecycle management |
| Platform Engineering Team | Service exposure |
| SRE Team | Zero-downtime deployments |
| Cloud Team | Container orchestration |
| Enterprise Teams | Production-grade deployments |

---

# Skills Covered

- Kubernetes Deployments
- Kubernetes Services
- Rolling Updates
- NodePort Services
- KIND
- kubectl
- ReplicaSets
- Kubernetes YAML
- DevOps Automation

---

# Expected Project Structure

```text
k8s-deployment-service-project/
├── nginx-deployment.yaml
└── nginx-service.yaml
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

kind create cluster --name deployment-service-cluster

kubectl cluster-info --context kind-deployment-service-cluster

kubectl get nodes

mkdir -p ~/k8s-deployment-service-project

cd ~/k8s-deployment-service-project

nano nginx-deployment.yaml

nano nginx-service.yaml

kubectl apply -f nginx-deployment.yaml

kubectl apply -f nginx-service.yaml

kubectl get deployments

kubectl get pods

kubectl get services

kubectl port-forward service/nginx-service 8080:80

kubectl set image deployment/nginx-deployment nginx-container=nginx:1.26

kubectl rollout status deployment/nginx-deployment

kubectl describe deployment nginx-deployment

kubectl get endpoints nginx-service

kubectl scale deployment nginx-deployment --replicas=5

kubectl get deployments

kubectl delete -f nginx-service.yaml

kubectl delete -f nginx-deployment.yaml

kind delete cluster --name deployment-service-cluster
```