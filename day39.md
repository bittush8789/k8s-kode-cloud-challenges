# Kubernetes Nginx and PHP-FPM Setup

# Normal Problem Statement

The DevOps team at xFusionCorp Industries wants to deploy an Nginx and PHP-FPM based application architecture on Kubernetes for scalable and high-performance PHP application hosting.

Your task is to create a Kubernetes cluster using Kind, deploy Nginx and PHP-FPM using Kubernetes manifests, configure Services for communication, and verify deployment on an Ubuntu-based system.

---

# Scenario-Based Problem Statement

xFusionCorp Industries manages enterprise-grade applications including:

- PHP applications
- Enterprise dashboards
- CMS platforms
- Banking portals
- Internal company applications

The organization currently faces several infrastructure challenges:

- Poor application scalability
- Downtime during deployments
- Manual web server management
- Lack of orchestration
- Poor traffic handling

To modernize infrastructure, the organization wants to deploy Nginx and PHP-FPM on Kubernetes.

Nginx will handle:

- Web traffic
- Reverse proxy
- Load balancing

PHP-FPM will handle:

- PHP request processing
- Backend execution
- Dynamic content generation

As a DevOps Engineer, your responsibilities are to:

1. Create Kubernetes cluster using Kind.
2. Deploy PHP-FPM backend.
3. Deploy Nginx frontend.
4. Configure Kubernetes Services.
5. Verify connectivity between components.
6. Scale deployments.
7. Build enterprise-grade Kubernetes architecture.

---

# Objectives

1. Install Kubernetes tools.
2. Create Kind cluster.
3. Deploy Nginx.
4. Deploy PHP-FPM.
5. Configure Services.
6. Verify communication.
7. Learn production deployment workflows.

---

# What is PHP-FPM?

PHP-FPM (FastCGI Process Manager) is a high-performance PHP execution engine used for:

- PHP applications
- Dynamic web content
- Enterprise portals
- Backend services

---

# Architecture Overview

```text
                    +----------------------+
                    |       Users          |
                    +----------+-----------+
                               |
                               v
                    +----------------------+
                    | Nginx Service        |
                    +----------+-----------+
                               |
                               v
                    +----------------------+
                    | Nginx Pods           |
                    +----------+-----------+
                               |
                               v
                    +----------------------+
                    | PHP-FPM Service      |
                    +----------+-----------+
                               |
                               v
                    +----------------------+
                    | PHP-FPM Pods         |
                    +----------------------+
```

---

# Real Industry Use Cases

| Use Case | Purpose |
|---|---|
| PHP Applications | Dynamic websites |
| Banking Portals | Backend execution |
| CMS Platforms | Content management |
| Enterprise Apps | Scalable architecture |
| APIs | PHP backend processing |

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
kind create cluster --name nginx-phpfpm-cluster
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
kubectl create namespace web-stack
```

---

# Step 8: Create PHP-FPM Deployment Manifest

```bash
nano php-fpm-deployment.yaml
```

Add the following YAML:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: php-fpm-deployment
  namespace: web-stack

spec:
  replicas: 2

  selector:
    matchLabels:
      app: php-fpm

  template:
    metadata:
      labels:
        app: php-fpm

    spec:
      containers:

        - name: php-fpm-container

          image: php:8.2-fpm

          ports:
            - containerPort: 9000
```

---

# Step 9: Apply PHP-FPM Deployment

```bash
kubectl apply -f php-fpm-deployment.yaml
```

Expected output:

```text
deployment.apps/php-fpm-deployment created
```

---

# Step 10: Verify PHP-FPM Pods

```bash
kubectl get pods -n web-stack
```

Expected:

```text
Running
```

---

# Step 11: Create PHP-FPM Service Manifest

```bash
nano php-fpm-service.yaml
```

Add the following YAML:

```yaml
apiVersion: v1
kind: Service

metadata:
  name: php-fpm-service
  namespace: web-stack

spec:
  selector:
    app: php-fpm

  ports:
    - port: 9000
      targetPort: 9000

  type: ClusterIP
```

---

# Step 12: Apply PHP-FPM Service

```bash
kubectl apply -f php-fpm-service.yaml
```

---

# Step 13: Verify PHP-FPM Service

```bash
kubectl get svc -n web-stack
```

Expected:

```text
php-fpm-service
```

---

# Step 14: Create Nginx Deployment Manifest

```bash
nano nginx-deployment.yaml
```

Add the following YAML:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: nginx-deployment
  namespace: web-stack

spec:
  replicas: 2

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

          image: nginx

          ports:
            - containerPort: 80
```

---

# Step 15: Apply Nginx Deployment

```bash
kubectl apply -f nginx-deployment.yaml
```

Expected output:

```text
deployment.apps/nginx-deployment created
```

---

# Step 16: Verify Nginx Pods

```bash
kubectl get pods -n web-stack
```

Expected:

```text
Running
```

---

# Step 17: Create Nginx Service Manifest

```bash
nano nginx-service.yaml
```

Add the following YAML:

```yaml
apiVersion: v1
kind: Service

metadata:
  name: nginx-service
  namespace: web-stack

spec:
  selector:
    app: nginx

  ports:
    - port: 80
      targetPort: 80
      nodePort: 32003

  type: NodePort
```

---

# Step 18: Apply Nginx Service

```bash
kubectl apply -f nginx-service.yaml
```

---

# Step 19: Verify Nginx Service

```bash
kubectl get svc -n web-stack
```

Expected:

```text
nginx-service
```

---

# Step 20: Verify Endpoints

```bash
kubectl get endpoints -n web-stack
```

Expected:

```text
Pod IPs visible
```

---

# Step 21: Port Forward Nginx Service

```bash
kubectl port-forward svc/nginx-service \
8080:80 -n web-stack
```

---

# Step 22: Access Application

Open browser:

```text
http://localhost:8080
```

Expected:

```text
Nginx welcome page
```

---

# Step 23: Verify PHP-FPM Connectivity

Run temporary Pod:

```bash
kubectl run test-pod \
-it --rm \
--image=busybox \
--restart=Never \
-n web-stack -- sh
```

Inside Pod:

```bash
nc -zv php-fpm-service 9000
```

Expected:

```text
Connection successful
```

Exit:

```bash
exit
```

---

# Step 24: Describe Deployments

```bash
kubectl describe deployment nginx-deployment \
-n web-stack
```

```bash
kubectl describe deployment php-fpm-deployment \
-n web-stack
```

---

# Step 25: Describe Services

```bash
kubectl describe svc nginx-service \
-n web-stack
```

```bash
kubectl describe svc php-fpm-service \
-n web-stack
```

---

# Step 26: Check Application Logs

Get Pod names:

```bash
kubectl get pods -n web-stack
```

Check Nginx logs:

```bash
kubectl logs <nginx-pod-name> \
-n web-stack
```

Check PHP-FPM logs:

```bash
kubectl logs <php-pod-name> \
-n web-stack
```

---

# Step 27: Scale Nginx Deployment

```bash
kubectl scale deployment nginx-deployment \
--replicas=4 -n web-stack
```

Verify:

```bash
kubectl get deployments -n web-stack
```

Expected:

```text
READY 4/4
```

---

# Step 28: Scale PHP-FPM Deployment

```bash
kubectl scale deployment php-fpm-deployment \
--replicas=4 -n web-stack
```

Verify:

```bash
kubectl get deployments -n web-stack
```

Expected:

```text
READY 4/4
```

---

# Step 29: Verify Scaled Pods

```bash
kubectl get pods -n web-stack
```

Expected:

```text
8 Running Pods
```

---

# Step 30: Perform Rolling Update

Update Nginx image:

```bash
kubectl set image deployment/nginx-deployment \
nginx-container=nginx:latest \
-n web-stack
```

Update PHP-FPM image:

```bash
kubectl set image deployment/php-fpm-deployment \
php-fpm-container=php:8.3-fpm \
-n web-stack
```

---

# Step 31: Verify Rollout Status

```bash
kubectl rollout status deployment/nginx-deployment \
-n web-stack
```

```bash
kubectl rollout status deployment/php-fpm-deployment \
-n web-stack
```

---

# Step 32: Verify Rollout History

```bash
kubectl rollout history deployment/nginx-deployment \
-n web-stack
```

```bash
kubectl rollout history deployment/php-fpm-deployment \
-n web-stack
```

---

# Step 33: Rollback Deployment

```bash
kubectl rollout undo deployment/nginx-deployment \
-n web-stack
```

```bash
kubectl rollout undo deployment/php-fpm-deployment \
-n web-stack
```

---

# Step 34: Troubleshooting Issues

# Check Pods

```bash
kubectl get pods -n web-stack
```

---

# Check Logs

```bash
kubectl logs <pod-name> -n web-stack
```

---

# Describe Pods

```bash
kubectl describe pod <pod-name> \
-n web-stack
```

---

# Common Issues

| Issue | Description |
|---|---|
| CrashLoopBackOff | Container crash |
| ImagePullBackOff | Invalid image |
| No Endpoints | Wrong selector |
| Connection Refused | Service issue |
| Port Issue | Wrong targetPort |

---

# Step 35: Verify Cluster Resources

```bash
kubectl get all -n web-stack
```

Expected:

```text
Pods
Services
Deployments
ReplicaSets
```

---

# Step 36: Delete Resources

```bash
kubectl delete deployment nginx-deployment \
-n web-stack

kubectl delete deployment php-fpm-deployment \
-n web-stack

kubectl delete svc nginx-service \
-n web-stack

kubectl delete svc php-fpm-service \
-n web-stack
```

---

# Step 37: Delete Namespace

```bash
kubectl delete namespace web-stack
```

---

# Step 38: Delete Kind Cluster

```bash
kind delete cluster --name nginx-phpfpm-cluster
```

---

# Manifest Files

# php-fpm-deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: php-fpm-deployment
  namespace: web-stack

spec:
  replicas: 2

  selector:
    matchLabels:
      app: php-fpm

  template:
    metadata:
      labels:
        app: php-fpm

    spec:
      containers:

        - name: php-fpm-container

          image: php:8.2-fpm

          ports:
            - containerPort: 9000
```

---

# php-fpm-service.yaml

```yaml
apiVersion: v1
kind: Service

metadata:
  name: php-fpm-service
  namespace: web-stack

spec:
  selector:
    app: php-fpm

  ports:
    - port: 9000
      targetPort: 9000

  type: ClusterIP
```

---

# nginx-deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: nginx-deployment
  namespace: web-stack

spec:
  replicas: 2

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

          image: nginx

          ports:
            - containerPort: 80
```

---

# nginx-service.yaml

```yaml
apiVersion: v1
kind: Service

metadata:
  name: nginx-service
  namespace: web-stack

spec:
  selector:
    app: nginx

  ports:
    - port: 80
      targetPort: 80
      nodePort: 32003

  type: NodePort
```

---

# Expected Outcome

- Kind Kubernetes cluster created successfully
- PHP-FPM deployed successfully
- Nginx deployed successfully
- Kubernetes Services configured
- Nginx and PHP-FPM communication verified
- Scaling operational
- Rolling updates operational
- Rollback operational
- Enterprise-grade deployment workflow implemented

---

# Skills Covered

- Nginx on Kubernetes
- PHP-FPM on Kubernetes
- Kubernetes Deployments
- Kubernetes Services
- NodePort Services
- ClusterIP Services
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
       +----------------------+
       |                      |
       v                      v
Nginx Pods             PHP-FPM Pods
       |                      |
       +----------+-----------+
                  |
                  v
            Kubernetes Services
                  |
                  v
               Users
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

kind create cluster --name nginx-phpfpm-cluster

kubectl create namespace web-stack

kubectl apply -f php-fpm-deployment.yaml

kubectl apply -f php-fpm-service.yaml

kubectl apply -f nginx-deployment.yaml

kubectl apply -f nginx-service.yaml

kubectl get pods -n web-stack

kubectl get svc -n web-stack

kubectl get endpoints -n web-stack

kubectl port-forward svc/nginx-service \
8080:80 -n web-stack

kubectl run test-pod \
-it --rm \
--image=busybox \
--restart=Never \
-n web-stack -- sh

nc -zv php-fpm-service 9000

kubectl scale deployment nginx-deployment \
--replicas=4 -n web-stack

kubectl scale deployment php-fpm-deployment \
--replicas=4 -n web-stack

kubectl rollout history deployment/nginx-deployment \
-n web-stack

kubectl rollout history deployment/php-fpm-deployment \
-n web-stack

kubectl rollout undo deployment/nginx-deployment \
-n web-stack

kubectl rollout undo deployment/php-fpm-deployment \
-n web-stack

kubectl get all -n web-stack

kubectl delete namespace web-stack

kind delete cluster --name nginx-phpfpm-cluster
```