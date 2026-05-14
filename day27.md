# Deploy Apache Web Server on Kubernetes Cluster

# Normal Problem Statement

The DevOps team at xFusionCorp Industries wants to deploy an Apache Web Server on a Kubernetes cluster for hosting static websites and enterprise web applications.

Your task is to create a Kubernetes cluster using Kind, deploy Apache HTTP Server using Kubernetes manifests, expose the application using a Kubernetes Service, and verify deployment on an Ubuntu-based system.

---

# Scenario-Based Problem Statement

xFusionCorp Industries manages enterprise infrastructure hosting:

- Static websites
- Internal portals
- Enterprise dashboards
- Documentation sites
- Web applications

The organization currently faces several operational challenges:

- Manual server management
- Downtime during deployments
- Poor scalability
- No centralized orchestration
- Difficult infrastructure management

To solve these issues, the organization wants to deploy Apache Web Server on Kubernetes.

As a DevOps Engineer, your responsibilities are to:

1. Create a Kubernetes cluster using Kind.
2. Deploy Apache Web Server on Kubernetes.
3. Configure scalable deployments.
4. Expose Apache service externally.
5. Verify web server accessibility.
6. Implement production-grade deployment workflows.

---

# Objectives

1. Install Kubernetes tools.
2. Create Kind cluster.
3. Deploy Apache HTTP Server.
4. Expose Apache Service.
5. Access Apache web page.
6. Scale deployments.
7. Verify Kubernetes resources.

---

# What is Apache HTTP Server?

Apache HTTP Server is one of the most widely used web servers for:

- Static websites
- Web applications
- Enterprise portals
- Reverse proxy
- Load balancing

---

# Architecture Overview

```text
                    +----------------+
                    |     Users      |
                    +--------+-------+
                             |
                             v
                    +----------------+
                    | Apache Service |
                    +--------+-------+
                             |
                             v
                    +----------------+
                    | Apache Pods    |
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
kind create cluster --name apache-cluster
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
apache-cluster-control-plane Ready    control-plane
```

---

# Step 7: Create Namespace

```bash
kubectl create namespace apache-web
```

Verify:

```bash
kubectl get namespaces
```

---

# Step 8: Create Apache Deployment Manifest

```bash
nano apache-deployment.yaml
```

Add the following YAML:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: apache-deployment
  namespace: apache-web

spec:
  replicas: 2

  selector:
    matchLabels:
      app: apache

  template:
    metadata:
      labels:
        app: apache

    spec:
      containers:

        - name: apache-container

          image: httpd:2.4

          ports:
            - containerPort: 80
```

---

# Step 9: Apply Deployment Manifest

```bash
kubectl apply -f apache-deployment.yaml
```

Expected output:

```text
deployment.apps/apache-deployment created
```

---

# Step 10: Verify Deployment

```bash
kubectl get deployments -n apache-web
```

Expected output:

```text
NAME                 READY
apache-deployment    2/2
```

---

# Step 11: Verify Pods

```bash
kubectl get pods -n apache-web
```

Expected output:

```text
NAME                                  READY   STATUS
apache-deployment-xxxxxx              1/1     Running
```

---

# Step 12: Create Apache Service Manifest

```bash
nano apache-service.yaml
```

Add the following YAML:

```yaml
apiVersion: v1
kind: Service

metadata:
  name: apache-service
  namespace: apache-web

spec:
  type: NodePort

  selector:
    app: apache

  ports:
    - port: 80
      targetPort: 80
      nodePort: 32005
```

---

# Step 13: Apply Service Manifest

```bash
kubectl apply -f apache-service.yaml
```

Expected output:

```text
service/apache-service created
```

---

# Step 14: Verify Service

```bash
kubectl get svc -n apache-web
```

Expected output:

```text
NAME              TYPE       PORT(S)
apache-service    NodePort   80:32005/TCP
```

---

# Step 15: Port Forward Apache Service

```bash
kubectl port-forward svc/apache-service \
8080:80 -n apache-web
```

---

# Step 16: Access Apache Web Server

Open browser:

```text
http://localhost:8080
```

Expected page:

```text
It works!
```

or Apache default welcome page.

---

# Step 17: Verify Running Pods

```bash
kubectl get pods -n apache-web
```

Verify:

```text
All Pods Running
```

---

# Step 18: Scale Apache Deployment

Scale replicas:

```bash
kubectl scale deployment apache-deployment \
--replicas=4 -n apache-web
```

Verify:

```bash
kubectl get deployments -n apache-web
```

Expected output:

```text
READY   4/4
```

---

# Step 19: Verify Scaled Pods

```bash
kubectl get pods -n apache-web
```

Expected:

```text
4 Running Pods
```

---

# Step 20: Inspect Deployment

```bash
kubectl describe deployment apache-deployment \
-n apache-web
```

---

# Step 21: Inspect Service

```bash
kubectl describe svc apache-service \
-n apache-web
```

---

# Step 22: Check Pod Logs

Get Pod name:

```bash
kubectl get pods -n apache-web
```

Check logs:

```bash
kubectl logs <pod-name> -n apache-web
```

Example:

```bash
kubectl logs apache-deployment-xxxxx \
-n apache-web
```

---

# Step 23: Verify Cluster Resources

```bash
kubectl get all -n apache-web
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

Update Apache image:

```bash
kubectl set image deployment/apache-deployment \
apache-container=httpd:latest \
-n apache-web
```

Verify rollout:

```bash
kubectl rollout status deployment/apache-deployment \
-n apache-web
```

Expected:

```text
successfully rolled out
```

---

# Step 25: Verify Rollout History

```bash
kubectl rollout history deployment/apache-deployment \
-n apache-web
```

---

# Step 26: Rollback Deployment

```bash
kubectl rollout undo deployment/apache-deployment \
-n apache-web
```

---

# Step 27: Delete Service

```bash
kubectl delete svc apache-service \
-n apache-web
```

---

# Step 28: Delete Deployment

```bash
kubectl delete deployment apache-deployment \
-n apache-web
```

---

# Step 29: Delete Namespace

```bash
kubectl delete namespace apache-web
```

---

# Step 30: Delete Kind Cluster

```bash
kind delete cluster --name apache-cluster
```

Expected output:

```text
Deleting cluster "apache-cluster"
```

---

# Manifest Files

# apache-deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: apache-deployment
  namespace: apache-web

spec:
  replicas: 2

  selector:
    matchLabels:
      app: apache

  template:
    metadata:
      labels:
        app: apache

    spec:
      containers:

        - name: apache-container

          image: httpd:2.4

          ports:
            - containerPort: 80
```

---

# apache-service.yaml

```yaml
apiVersion: v1
kind: Service

metadata:
  name: apache-service
  namespace: apache-web

spec:
  type: NodePort

  selector:
    app: apache

  ports:
    - port: 80
      targetPort: 80
      nodePort: 32005
```

---

# Expected Outcome

- Kind Kubernetes cluster created successfully
- Apache Web Server deployed successfully
- Kubernetes Service configured
- Apache application accessible
- Deployment scaling operational
- Rolling updates configured
- Enterprise-grade Apache deployment workflow implemented

---

# Real Industry Use Cases

| Use Case | Description |
|---|---|
| Static Websites | Corporate websites |
| Enterprise Portals | Internal dashboards |
| Reverse Proxy | Traffic routing |
| DevOps | Kubernetes deployments |
| Cloud-Native Apps | Containerized web servers |

---

# Skills Covered

- Kubernetes Deployments
- Kubernetes Services
- Apache HTTP Server
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
Apache Pods
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

kind create cluster --name apache-cluster

kubectl cluster-info

kubectl get nodes

kubectl create namespace apache-web

nano apache-deployment.yaml

kubectl apply -f apache-deployment.yaml

kubectl get deployments -n apache-web

kubectl get pods -n apache-web

nano apache-service.yaml

kubectl apply -f apache-service.yaml

kubectl get svc -n apache-web

kubectl port-forward svc/apache-service \
8080:80 -n apache-web

kubectl scale deployment apache-deployment \
--replicas=4 -n apache-web

kubectl describe deployment apache-deployment \
-n apache-web

kubectl describe svc apache-service \
-n apache-web

kubectl logs <pod-name> -n apache-web

kubectl get all -n apache-web

kubectl set image deployment/apache-deployment \
apache-container=httpd:latest \
-n apache-web

kubectl rollout status deployment/apache-deployment \
-n apache-web

kubectl rollout history deployment/apache-deployment \
-n apache-web

kubectl rollout undo deployment/apache-deployment \
-n apache-web

kubectl delete svc apache-service \
-n apache-web

kubectl delete deployment apache-deployment \
-n apache-web

kubectl delete namespace apache-web

kind delete cluster --name apache-cluster
```