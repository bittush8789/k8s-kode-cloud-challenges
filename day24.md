# Deploy Node.js App on Kubernetes

# Normal Problem Statement

The DevOps team at xFusionCorp Industries wants to deploy a Node.js application on a Kubernetes cluster for hosting scalable backend APIs and web services.

Your task is to create a Kubernetes cluster using Kind, deploy a Node.js application using Kubernetes manifests, expose the application using a Kubernetes Service, and verify deployment on an Ubuntu-based system.

---

# Scenario-Based Problem Statement

xFusionCorp Industries manages enterprise cloud-native applications including:

- REST APIs
- Backend services
- Microservices
- AI platforms
- Web applications

The organization currently faces several operational challenges:

- Manual deployments
- No scalable orchestration
- Downtime during deployments
- Poor container management
- No centralized deployment platform

To solve these issues, the organization wants to deploy Node.js applications on Kubernetes.

As a DevOps Engineer, your responsibilities are to:

1. Create a Kubernetes cluster using Kind.
2. Deploy Node.js application on Kubernetes.
3. Configure scalable deployments.
4. Expose application using Kubernetes Service.
5. Verify application accessibility.
6. Implement production-style deployment workflows.

---

# Objectives

1. Install Kubernetes tools.
2. Create Kind cluster.
3. Deploy Node.js application.
4. Expose Node.js service.
5. Access application.
6. Scale deployments.
7. Verify Kubernetes resources.

---

# What is Node.js?

Node.js is a JavaScript runtime used for:

- Backend APIs
- Microservices
- Real-time applications
- Web servers
- Enterprise applications

---

# Architecture Overview

```text
                    +----------------+
                    |   Developers   |
                    +--------+-------+
                             |
                             v
                    +----------------+
                    | Node.js Service|
                    +--------+-------+
                             |
                             v
                    +----------------+
                    | Node.js Pods   |
                    +----------------+
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
kind create cluster --name nodejs-cluster
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
NAME                         STATUS   ROLES
nodejs-cluster-control-plane Ready    control-plane
```

---

# Step 7: Create Namespace

```bash
kubectl create namespace nodejs
```

Verify:

```bash
kubectl get namespaces
```

---

# Step 8: Create Node.js Deployment Manifest

```bash
nano nodejs-deployment.yaml
```

Add the following YAML:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: nodejs-deployment
  namespace: nodejs

spec:
  replicas: 2

  selector:
    matchLabels:
      app: nodejs

  template:
    metadata:
      labels:
        app: nodejs

    spec:
      containers:

        - name: nodejs-container

          image: node:18-alpine

          command: ["sh", "-c"]

          args:
            - |
              echo "
              const http = require('http');

              const server = http.createServer((req, res) => {
                res.writeHead(200, {'Content-Type': 'text/plain'});
                res.end('Welcome to Node.js App on Kubernetes');
              });

              server.listen(3000);
              " > app.js && node app.js

          ports:
            - containerPort: 3000
```

---

# Step 9: Apply Deployment Manifest

```bash
kubectl apply -f nodejs-deployment.yaml
```

Expected output:

```text
deployment.apps/nodejs-deployment created
```

---

# Step 10: Verify Deployment

```bash
kubectl get deployments -n nodejs
```

Expected output:

```text
NAME                 READY
nodejs-deployment    2/2
```

---

# Step 11: Verify Pods

```bash
kubectl get pods -n nodejs
```

Expected output:

```text
NAME                                  READY   STATUS
nodejs-deployment-xxxxxx              1/1     Running
```

---

# Step 12: Create Node.js Service Manifest

```bash
nano nodejs-service.yaml
```

Add the following YAML:

```yaml
apiVersion: v1
kind: Service

metadata:
  name: nodejs-service
  namespace: nodejs

spec:
  type: NodePort

  selector:
    app: nodejs

  ports:
    - port: 3000
      targetPort: 3000
      nodePort: 32003
```

---

# Step 13: Apply Service Manifest

```bash
kubectl apply -f nodejs-service.yaml
```

Expected output:

```text
service/nodejs-service created
```

---

# Step 14: Verify Service

```bash
kubectl get svc -n nodejs
```

Expected output:

```text
NAME             TYPE       PORT(S)
nodejs-service   NodePort   3000:32003/TCP
```

---

# Step 15: Port Forward Service

```bash
kubectl port-forward svc/nodejs-service \
3000:3000 -n nodejs
```

---

# Step 16: Access Node.js Application

Open browser:

```text
http://localhost:3000
```

Expected output:

```text
Welcome to Node.js App on Kubernetes
```

---

# Step 17: Verify Running Pods

```bash
kubectl get pods -n nodejs
```

Verify:

```text
All Pods Running
```

---

# Step 18: Scale Node.js Deployment

Scale replicas:

```bash
kubectl scale deployment nodejs-deployment \
--replicas=4 -n nodejs
```

Verify:

```bash
kubectl get deployments -n nodejs
```

Expected output:

```text
READY   4/4
```

---

# Step 19: Verify Scaled Pods

```bash
kubectl get pods -n nodejs
```

Expected:

```text
4 Running Pods
```

---

# Step 20: Inspect Deployment

```bash
kubectl describe deployment nodejs-deployment -n nodejs
```

---

# Step 21: Inspect Service

```bash
kubectl describe svc nodejs-service -n nodejs
```

---

# Step 22: Check Logs

Get Pod name:

```bash
kubectl get pods -n nodejs
```

Check logs:

```bash
kubectl logs <pod-name> -n nodejs
```

Example:

```bash
kubectl logs nodejs-deployment-xxxxx -n nodejs
```

---

# Step 23: Verify Cluster Resources

```bash
kubectl get all -n nodejs
```

Expected:

```text
Pods
Services
Deployments
ReplicaSets
```

---

# Step 24: Perform Rolling Update

Update image:

```bash
kubectl set image deployment/nodejs-deployment \
nodejs-container=node:20-alpine -n nodejs
```

Verify rollout:

```bash
kubectl rollout status deployment/nodejs-deployment -n nodejs
```

---

# Step 25: Verify Rollout History

```bash
kubectl rollout history deployment/nodejs-deployment -n nodejs
```

---

# Step 26: Rollback Deployment

```bash
kubectl rollout undo deployment/nodejs-deployment -n nodejs
```

---

# Step 27: Delete Service

```bash
kubectl delete svc nodejs-service -n nodejs
```

---

# Step 28: Delete Deployment

```bash
kubectl delete deployment nodejs-deployment -n nodejs
```

---

# Step 29: Delete Namespace

```bash
kubectl delete namespace nodejs
```

---

# Step 30: Delete Kind Cluster

```bash
kind delete cluster --name nodejs-cluster
```

Expected output:

```text
Deleting cluster "nodejs-cluster"
```

---

# Manifest Files

# nodejs-deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: nodejs-deployment
  namespace: nodejs

spec:
  replicas: 2

  selector:
    matchLabels:
      app: nodejs

  template:
    metadata:
      labels:
        app: nodejs

    spec:
      containers:

        - name: nodejs-container

          image: node:18-alpine

          command: ["sh", "-c"]

          args:
            - |
              echo "
              const http = require('http');

              const server = http.createServer((req, res) => {
                res.writeHead(200, {'Content-Type': 'text/plain'});
                res.end('Welcome to Node.js App on Kubernetes');
              });

              server.listen(3000);
              " > app.js && node app.js

          ports:
            - containerPort: 3000
```

---

# nodejs-service.yaml

```yaml
apiVersion: v1
kind: Service

metadata:
  name: nodejs-service
  namespace: nodejs

spec:
  type: NodePort

  selector:
    app: nodejs

  ports:
    - port: 3000
      targetPort: 3000
      nodePort: 32003
```

---

# Expected Outcome

- Kind Kubernetes cluster created successfully
- Node.js application deployed successfully
- Kubernetes Service configured
- Node.js application accessible
- Deployment scaling operational
- Rolling updates configured
- Enterprise-grade Node.js deployment workflow implemented

---

# Real Industry Use Cases

| Use Case | Description |
|---|---|
| REST APIs | Backend services |
| Microservices | Node.js workloads |
| Real-Time Apps | Socket applications |
| DevOps | Kubernetes deployments |
| Cloud-Native Apps | Containerized Node.js |

---

# Skills Covered

- Kubernetes Deployments
- Kubernetes Services
- Node.js on Kubernetes
- Scaling Applications
- Rolling Updates
- Kubernetes Troubleshooting
- DevOps
- Cloud-Native Deployment

---

# Real Enterprise Workflow

```text
Developer Push
       |
       v
Docker Image
       |
       v
Kubernetes Deployment
       |
       v
Node.js Pods
       |
       v
Kubernetes Service
       |
       v
User Access
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

kind create cluster --name nodejs-cluster

kubectl cluster-info

kubectl get nodes

kubectl create namespace nodejs

nano nodejs-deployment.yaml

kubectl apply -f nodejs-deployment.yaml

kubectl get deployments -n nodejs

kubectl get pods -n nodejs

nano nodejs-service.yaml

kubectl apply -f nodejs-service.yaml

kubectl get svc -n nodejs

kubectl port-forward svc/nodejs-service \
3000:3000 -n nodejs

kubectl scale deployment nodejs-deployment \
--replicas=4 -n nodejs

kubectl describe deployment nodejs-deployment -n nodejs

kubectl describe svc nodejs-service -n nodejs

kubectl logs <pod-name> -n nodejs

kubectl get all -n nodejs

kubectl set image deployment/nodejs-deployment \
nodejs-container=node:20-alpine -n nodejs

kubectl rollout status deployment/nodejs-deployment -n nodejs

kubectl rollout history deployment/nodejs-deployment -n nodejs

kubectl rollout undo deployment/nodejs-deployment -n nodejs

kubectl delete svc nodejs-service -n nodejs

kubectl delete deployment nodejs-deployment -n nodejs

kubectl delete namespace nodejs

kind delete cluster --name nodejs-cluster
```