# Environment Variables in Kubernetes

# Normal Problem Statement

The DevOps team at xFusionCorp Industries wants to configure Environment Variables in Kubernetes for managing application configurations such as database hostnames, ports, API URLs, credentials, and runtime settings.

Your task is to create a Kubernetes cluster using Kind, configure environment variables inside Pods and Deployments, and verify application configuration management on an Ubuntu-based system.

---

# Scenario-Based Problem Statement

xFusionCorp Industries manages enterprise Kubernetes workloads including:

- AI/ML platforms
- APIs
- Microservices
- Node.js applications
- Java applications

The organization currently faces several operational challenges:

- Hardcoded application configurations
- Manual configuration updates
- Environment-specific deployment issues
- Poor portability between environments
- Application startup failures

Applications affected:

- Development environments
- Testing environments
- Production workloads
- CI/CD deployments

To solve these issues, the organization wants to implement Environment Variables in Kubernetes.

As a DevOps Engineer, your responsibilities are to:

1. Create Kubernetes cluster using Kind.
2. Configure environment variables.
3. Inject runtime configurations into Pods.
4. Manage application settings dynamically.
5. Verify environment variable access.
6. Implement production-grade configuration management.

---

# Objectives

1. Install Kubernetes tools.
2. Create Kind cluster.
3. Configure environment variables.
4. Use ConfigMaps and Secrets.
5. Inject variables into Pods.
6. Verify runtime configurations.
7. Learn production configuration workflows.

---

# What are Environment Variables?

Environment Variables are runtime configuration values passed into containers.

Examples:

- Database host
- API URL
- Application mode
- Port numbers
- Feature flags

---

# Why Use Environment Variables?

Without environment variables:

```yaml
db-host: production-db.company.com
```

Problems:

- Hardcoded configuration
- Difficult environment switching
- Poor scalability

With environment variables:

- Dynamic configuration
- Better portability
- Easy deployment automation

---

# Architecture Overview

```text
                +----------------------+
                |   Kubernetes         |
                +----------+-----------+
                           |
                           v
                +----------------------+
                | ConfigMap / Secret   |
                +----------+-----------+
                           |
                           v
                +----------------------+
                | Application Pod      |
                +----------------------+
```

---

# Real Industry Use Cases

| Use Case | Purpose |
|---|---|
| Database Config | DB host and port |
| API Endpoints | External APIs |
| Application Mode | Dev/Test/Prod |
| Feature Flags | Enable features |
| Cloud Credentials | Cloud access |

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
kind create cluster --name env-cluster
```

Verify:

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
kubectl create namespace env-demo
```

---

# Step 8: Create Pod with Environment Variables

```bash
nano env-pod.yaml
```

Add the following YAML:

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: env-pod
  namespace: env-demo

spec:
  containers:

    - name: nginx-container

      image: nginx

      env:
        - name: APP_ENV
          value: production

        - name: DB_HOST
          value: mysql-service

        - name: DB_PORT
          value: "3306"
```

---

# Step 9: Apply Pod Manifest

```bash
kubectl apply -f env-pod.yaml
```

Expected output:

```text
pod/env-pod created
```

---

# Step 10: Verify Pod

```bash
kubectl get pods -n env-demo
```

Expected:

```text
Running
```

---

# Step 11: Access Pod Shell

```bash
kubectl exec -it env-pod \
-n env-demo -- sh
```

---

# Step 12: Verify Environment Variables

Inside Pod:

```bash
echo $APP_ENV
```

Expected:

```text
production
```

Check DB host:

```bash
echo $DB_HOST
```

Expected:

```text
mysql-service
```

Check DB port:

```bash
echo $DB_PORT
```

Expected:

```text
3306
```

Exit shell:

```bash
exit
```

---

# Step 13: Create ConfigMap

```bash
kubectl create configmap app-config \
--from-literal=APP_MODE=production \
--from-literal=APP_PORT=8080 \
-n env-demo
```

Expected output:

```text
configmap/app-config created
```

---

# Step 14: Verify ConfigMap

```bash
kubectl get configmap -n env-demo
```

---

# Step 15: Describe ConfigMap

```bash
kubectl describe configmap app-config \
-n env-demo
```

---

# Step 16: Create Pod Using ConfigMap

```bash
nano configmap-pod.yaml
```

Add:

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: configmap-pod
  namespace: env-demo

spec:
  containers:

    - name: nginx

      image: nginx

      env:
        - name: APP_MODE
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: APP_MODE

        - name: APP_PORT
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: APP_PORT
```

---

# Step 17: Apply Pod Manifest

```bash
kubectl apply -f configmap-pod.yaml
```

---

# Step 18: Verify Pod

```bash
kubectl get pods -n env-demo
```

---

# Step 19: Access ConfigMap Pod

```bash
kubectl exec -it configmap-pod \
-n env-demo -- sh
```

Verify variables:

```bash
echo $APP_MODE
```

Expected:

```text
production
```

Check port:

```bash
echo $APP_PORT
```

Expected:

```text
8080
```

Exit shell:

```bash
exit
```

---

# Step 20: Create Secret for Sensitive Variables

```bash
kubectl create secret generic app-secret \
--from-literal=DB_USER=admin \
--from-literal=DB_PASSWORD=admin123 \
-n env-demo
```

---

# Step 21: Verify Secret

```bash
kubectl get secrets -n env-demo
```

---

# Step 22: Create Pod Using Secret Variables

```bash
nano secret-env-pod.yaml
```

Add:

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: secret-env-pod
  namespace: env-demo

spec:
  containers:

    - name: nginx

      image: nginx

      env:
        - name: DB_USER
          valueFrom:
            secretKeyRef:
              name: app-secret
              key: DB_USER

        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: app-secret
              key: DB_PASSWORD
```

---

# Step 23: Apply Secret Pod Manifest

```bash
kubectl apply -f secret-env-pod.yaml
```

---

# Step 24: Verify Secret Pod

```bash
kubectl get pods -n env-demo
```

---

# Step 25: Verify Secret Variables

Access Pod:

```bash
kubectl exec -it secret-env-pod \
-n env-demo -- sh
```

Check variables:

```bash
echo $DB_USER
```

Expected:

```text
admin
```

Check password:

```bash
echo $DB_PASSWORD
```

Expected:

```text
admin123
```

Exit shell:

```bash
exit
```

---

# Step 26: Create Deployment Using Environment Variables

```bash
nano env-deployment.yaml
```

Add:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: env-deployment
  namespace: env-demo

spec:
  replicas: 2

  selector:
    matchLabels:
      app: env-app

  template:
    metadata:
      labels:
        app: env-app

    spec:
      containers:

        - name: nginx

          image: nginx

          env:
            - name: APP_NAME
              value: KubernetesApp
```

---

# Step 27: Apply Deployment

```bash
kubectl apply -f env-deployment.yaml
```

---

# Step 28: Verify Deployment

```bash
kubectl get deployments -n env-demo
```

---

# Step 29: Troubleshooting Environment Variables

# Describe Pod

```bash
kubectl describe pod env-pod \
-n env-demo
```

---

# Common Issues

| Issue | Description |
|---|---|
| Variable Missing | Wrong variable name |
| ConfigMap Error | Invalid key |
| Secret Not Found | Wrong secret name |
| Pod Crash | Invalid configuration |

---

# Step 30: Verify All Resources

```bash
kubectl get all -n env-demo

kubectl get configmap -n env-demo

kubectl get secrets -n env-demo
```

---

# Step 31: Delete Resources

```bash
kubectl delete pod env-pod -n env-demo

kubectl delete pod configmap-pod -n env-demo

kubectl delete pod secret-env-pod -n env-demo

kubectl delete deployment env-deployment -n env-demo

kubectl delete configmap app-config -n env-demo

kubectl delete secret app-secret -n env-demo
```

---

# Step 32: Delete Namespace

```bash
kubectl delete namespace env-demo
```

---

# Step 33: Delete Kind Cluster

```bash
kind delete cluster --name env-cluster
```

---

# Manifest Files

# env-pod.yaml

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: env-pod
  namespace: env-demo

spec:
  containers:

    - name: nginx-container

      image: nginx

      env:
        - name: APP_ENV
          value: production

        - name: DB_HOST
          value: mysql-service

        - name: DB_PORT
          value: "3306"
```

---

# configmap-pod.yaml

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: configmap-pod
  namespace: env-demo

spec:
  containers:

    - name: nginx

      image: nginx

      env:
        - name: APP_MODE
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: APP_MODE

        - name: APP_PORT
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: APP_PORT
```

---

# secret-env-pod.yaml

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: secret-env-pod
  namespace: env-demo

spec:
  containers:

    - name: nginx

      image: nginx

      env:
        - name: DB_USER
          valueFrom:
            secretKeyRef:
              name: app-secret
              key: DB_USER

        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: app-secret
              key: DB_PASSWORD
```

---

# env-deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: env-deployment
  namespace: env-demo

spec:
  replicas: 2

  selector:
    matchLabels:
      app: env-app

  template:
    metadata:
      labels:
        app: env-app

    spec:
      containers:

        - name: nginx

          image: nginx

          env:
            - name: APP_NAME
              value: KubernetesApp
```

---

# Expected Outcome

- Kind Kubernetes cluster created successfully
- Environment variables configured
- ConfigMaps configured
- Secrets configured
- Runtime configurations injected successfully
- Enterprise-grade configuration workflow implemented

---

# Skills Covered

- Environment Variables
- ConfigMaps
- Secrets
- Kubernetes Deployments
- Runtime Configuration
- Kubernetes Troubleshooting
- DevOps
- Cloud-Native Configuration

---

# Real Enterprise Workflow

```text
Application Configuration
        |
        +----------------------+
        |                      |
        v                      v
ConfigMap                Secret
        |                      |
        +----------+-----------+
                   |
                   v
           Kubernetes Pod
                   |
                   v
        Runtime Application Config
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

kind create cluster --name env-cluster

kubectl create namespace env-demo

kubectl apply -f env-pod.yaml

kubectl exec -it env-pod \
-n env-demo -- sh

kubectl create configmap app-config \
--from-literal=APP_MODE=production \
--from-literal=APP_PORT=8080 \
-n env-demo

kubectl apply -f configmap-pod.yaml

kubectl create secret generic app-secret \
--from-literal=DB_USER=admin \
--from-literal=DB_PASSWORD=admin123 \
-n env-demo

kubectl apply -f secret-env-pod.yaml

kubectl apply -f env-deployment.yaml

kubectl get all -n env-demo

kubectl delete namespace env-demo

kind delete cluster --name env-cluster
```