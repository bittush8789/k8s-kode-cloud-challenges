# Day 13: Expose Application Using NodePort Service in Kubernetes using KIND

# Normal Problem Statement

The DevOps team at xFusionCorp Industries wants to expose Kubernetes applications externally using NodePort Services for application accessibility and testing.

Your task is to create a Kubernetes cluster (if not already available), deploy an application using Kubernetes Deployment manifests, expose it using a NodePort Service, and verify external application access on an Ubuntu system.

---

# Scenario-Based Problem Statement

xFusionCorp Industries runs multiple containerized applications inside Kubernetes clusters.

The organization needs external access for:

- Web applications
- APIs
- Monitoring dashboards
- Internal enterprise tools
- Development testing environments

Currently, applications are only accessible internally within the Kubernetes cluster, causing several operational challenges:

- No external user access
- Difficulty testing applications
- Limited developer accessibility
- Inability to expose services externally
- Complex networking management

To solve these problems, the organization wants to use Kubernetes NodePort Services for external application exposure.

As a DevOps Engineer, your responsibility is to:

1. Create a Kubernetes cluster if one does not exist.
2. Deploy applications using Kubernetes Deployments.
3. Expose applications externally using NodePort Services.
4. Verify application accessibility from outside the cluster.

---

# Objectives

1. Install Docker, kubectl, and KIND.
2. Create a Kubernetes cluster using KIND.
3. Create Deployment and NodePort Service manifests.
4. Deploy applications into Kubernetes.
5. Expose applications externally.
6. Verify NodePort connectivity.
7. Scale applications dynamically.
8. Delete resources safely.

---

# Architecture Overview

```text
External User
      |
      v
NodePort Service
      |
      v
Kubernetes Service
      |
      v
Deployment
      |
      v
Pods
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

# Step 5: Create KIND Cluster Configuration File

```bash
nano kind-config.yaml
```

Add the following configuration:

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4

nodes:
  - role: control-plane

    extraPortMappings:
      - containerPort: 30080
        hostPort: 8080
        protocol: TCP
```

---

# Complete KIND Cluster Configuration File

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4

nodes:
  - role: control-plane

    extraPortMappings:
      - containerPort: 30080
        hostPort: 8080
        protocol: TCP
```

---

# Step 6: Create KIND Kubernetes Cluster

```bash
kind create cluster --name nodeport-cluster --config kind-config.yaml
```

Expected output:

```text
Creating cluster "nodeport-cluster" ...
Cluster creation complete.
```

---

# Step 7: Verify Kubernetes Cluster

```bash
kubectl cluster-info --context kind-nodeport-cluster
```

Check nodes:

```bash
kubectl get nodes
```

Expected output:

```text
NAME                               STATUS   ROLES
nodeport-cluster-control-plane    Ready    control-plane
```

---

# Step 8: Create Kubernetes Project Directory

```bash
mkdir -p ~/k8s-nodeport-project

cd ~/k8s-nodeport-project
```

---

# Step 9: Create Deployment Manifest File

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

          image: nginx:latest

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

          image: nginx:latest

          ports:
            - containerPort: 80
```

---

# Step 10: Create NodePort Service Manifest File

```bash
nano nginx-nodeport-service.yaml
```

Add the following configuration:

```yaml
apiVersion: v1
kind: Service

metadata:
  name: nginx-nodeport-service

spec:
  type: NodePort

  selector:
    app: nginx

  ports:
    - protocol: TCP

      port: 80
      targetPort: 80

      nodePort: 30080
```

---

# Complete NodePort Service Manifest File

```yaml
apiVersion: v1
kind: Service

metadata:
  name: nginx-nodeport-service

spec:
  type: NodePort

  selector:
    app: nginx

  ports:
    - protocol: TCP

      port: 80
      targetPort: 80

      nodePort: 30080
```

---

# Step 11: Deploy Application

Deploy Deployment:

```bash
kubectl apply -f nginx-deployment.yaml
```

Deploy Service:

```bash
kubectl apply -f nginx-nodeport-service.yaml
```

Expected output:

```text
deployment.apps/nginx-deployment created
service/nginx-nodeport-service created
```

---

# Step 12: Verify Deployment

```bash
kubectl get deployments
```

Expected output:

```text
NAME               READY   UP-TO-DATE   AVAILABLE
nginx-deployment   3/3     3            3
```

---

# Step 13: Verify Running Pods

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

# Step 14: Verify NodePort Service

```bash
kubectl get services
```

Expected output:

```text
NAME                       TYPE       CLUSTER-IP      PORT(S)
nginx-nodeport-service     NodePort   10.96.120.10   80:30080/TCP
```

---

# Step 15: Access Application Externally

Open browser:

```text
http://localhost:8080
```

OR use curl:

```bash
curl http://localhost:8080
```

Expected output:

```html
Welcome to nginx!
```

---

# Step 16: Verify Service Endpoints

```bash
kubectl get endpoints nginx-nodeport-service
```

Expected output:

```text
NAME                       ENDPOINTS
nginx-nodeport-service     10.244.0.x:80
```

---

# Step 17: Scale Deployment

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

# Step 18: Verify Pods After Scaling

```bash
kubectl get pods
```

Expected output:

```text
5 Pods running successfully
```

---

# Step 19: Delete Resources

Delete Service:

```bash
kubectl delete -f nginx-nodeport-service.yaml
```

Delete Deployment:

```bash
kubectl delete -f nginx-deployment.yaml
```

---

# Step 20: Delete KIND Cluster

```bash
kind delete cluster --name nodeport-cluster
```

Expected output:

```text
Deleting cluster "nodeport-cluster" ...
```

---

# Expected Outcome

- KIND Kubernetes cluster created successfully
- Deployment configured correctly
- NodePort Service exposed externally
- External application access verified
- Scalable application deployment achieved
- Kubernetes networking operational

---

# Real Industry Use Cases

| Team | Use Case |
|---|---|
| DevOps Team | External application exposure |
| Platform Engineering Team | Kubernetes networking |
| SRE Team | Service accessibility |
| Cloud Team | Containerized web applications |
| Enterprise Teams | Internal application access |

---

# Skills Covered

- Kubernetes Services
- NodePort Services
- Kubernetes Deployments
- KIND
- kubectl
- Kubernetes Networking
- YAML Manifests
- Container Orchestration
- DevOps Automation

---

# Expected Project Structure

```text
k8s-nodeport-project/
├── kind-config.yaml
├── nginx-deployment.yaml
└── nginx-nodeport-service.yaml
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

mkdir -p ~/k8s-nodeport-project

cd ~/k8s-nodeport-project

nano kind-config.yaml

kind create cluster --name nodeport-cluster --config kind-config.yaml

kubectl cluster-info --context kind-nodeport-cluster

kubectl get nodes

nano nginx-deployment.yaml

nano nginx-nodeport-service.yaml

kubectl apply -f nginx-deployment.yaml

kubectl apply -f nginx-nodeport-service.yaml

kubectl get deployments

kubectl get pods

kubectl get services

curl http://localhost:8080

kubectl get endpoints nginx-nodeport-service

kubectl scale deployment nginx-deployment --replicas=5

kubectl get deployments

kubectl get pods

kubectl delete -f nginx-nodeport-service.yaml

kubectl delete -f nginx-deployment.yaml

kind delete cluster --name nodeport-cluster
```