# Deploy Tomcat App on Kubernetes

# Normal Problem Statement

The DevOps team at xFusionCorp Industries wants to deploy an Apache Tomcat application on a Kubernetes cluster for hosting Java-based web applications.

Your task is to create a Kubernetes cluster using Kind, deploy Tomcat using Kubernetes manifests, expose the Tomcat application, and verify deployment on an Ubuntu-based system.

---

# Scenario-Based Problem Statement

xFusionCorp Industries manages enterprise applications including:

- Java web applications
- Internal portals
- Banking systems
- Enterprise APIs
- Microservices

The organization currently faces several operational issues:

- Manual deployments
- No scalable infrastructure
- Downtime during upgrades
- Lack of centralized orchestration
- Poor deployment consistency

To solve these issues, the organization wants to deploy Tomcat applications on Kubernetes.

As a DevOps Engineer, your responsibilities are to:

1. Create a Kubernetes cluster using Kind.
2. Deploy Tomcat on Kubernetes.
3. Configure scalable deployments.
4. Expose Tomcat application.
5. Verify application accessibility.
6. Implement production-style deployment workflows.

---

# Objectives

1. Install Kubernetes tools.
2. Create Kind cluster.
3. Deploy Tomcat application.
4. Expose Tomcat service.
5. Access Tomcat UI.
6. Verify deployment health.
7. Implement scalable Java app deployment.

---

# What is Apache Tomcat?

Apache Tomcat is an open-source Java Servlet container used for:

- Hosting Java applications
- Running JSP applications
- Enterprise web applications
- Java APIs
- Backend services

---

# Architecture Overview

```text
                    +----------------+
                    |   Developers   |
                    +--------+-------+
                             |
                             v
                    +----------------+
                    | Tomcat Service |
                    +--------+-------+
                             |
                             v
                    +----------------+
                    | Tomcat Pods    |
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
kind create cluster --name tomcat-cluster
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
tomcat-cluster-control-plane Ready    control-plane
```

---

# Step 7: Create Namespace

```bash
kubectl create namespace tomcat
```

Verify:

```bash
kubectl get namespaces
```

---

# Step 8: Create Tomcat Deployment Manifest

```bash
nano tomcat-deployment.yaml
```

Add the following YAML:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: tomcat-deployment
  namespace: tomcat

spec:
  replicas: 2

  selector:
    matchLabels:
      app: tomcat

  template:
    metadata:
      labels:
        app: tomcat

    spec:
      containers:

        - name: tomcat-container

          image: tomcat:9.0

          ports:
            - containerPort: 8080
```

---

# Step 9: Apply Deployment Manifest

```bash
kubectl apply -f tomcat-deployment.yaml
```

Expected output:

```text
deployment.apps/tomcat-deployment created
```

---

# Step 10: Verify Deployment

```bash
kubectl get deployments -n tomcat
```

Expected output:

```text
NAME                 READY
tomcat-deployment    2/2
```

---

# Step 11: Verify Pods

```bash
kubectl get pods -n tomcat
```

Expected output:

```text
NAME                                 READY   STATUS
tomcat-deployment-xxxxxx             1/1     Running
```

---

# Step 12: Create Tomcat Service Manifest

```bash
nano tomcat-service.yaml
```

Add the following YAML:

```yaml
apiVersion: v1
kind: Service

metadata:
  name: tomcat-service
  namespace: tomcat

spec:
  type: NodePort

  selector:
    app: tomcat

  ports:
    - port: 8080
      targetPort: 8080
      nodePort: 32002
```

---

# Step 13: Apply Service Manifest

```bash
kubectl apply -f tomcat-service.yaml
```

Expected output:

```text
service/tomcat-service created
```

---

# Step 14: Verify Service

```bash
kubectl get svc -n tomcat
```

Expected output:

```text
NAME             TYPE       PORT(S)
tomcat-service   NodePort   8080:32002/TCP
```

---

# Step 15: Port Forward Service

```bash
kubectl port-forward svc/tomcat-service \
8080:8080 -n tomcat
```

---

# Step 16: Access Tomcat Application

Open browser:

```text
http://localhost:8080
```

Expected screen:

```text
Apache Tomcat Welcome Page
```

---

# Step 17: Verify Running Pods

```bash
kubectl get pods -n tomcat
```

Verify:

```text
All Pods Running
```

---

# Step 18: Scale Tomcat Deployment

Scale replicas:

```bash
kubectl scale deployment tomcat-deployment \
--replicas=4 -n tomcat
```

Verify:

```bash
kubectl get deployments -n tomcat
```

Expected output:

```text
READY   4/4
```

---

# Step 19: Verify Scaled Pods

```bash
kubectl get pods -n tomcat
```

Expected:

```text
4 Running Pods
```

---

# Step 20: Inspect Deployment

```bash
kubectl describe deployment tomcat-deployment -n tomcat
```

---

# Step 21: Inspect Service

```bash
kubectl describe svc tomcat-service -n tomcat
```

---

# Step 22: Check Logs

Get Pod name:

```bash
kubectl get pods -n tomcat
```

Check logs:

```bash
kubectl logs <pod-name> -n tomcat
```

Example:

```bash
kubectl logs tomcat-deployment-xxxxx -n tomcat
```

---

# Step 23: Verify Cluster Resources

```bash
kubectl get all -n tomcat
```

Expected:

```text
Pods
Services
Deployments
ReplicaSets
```

---

# Step 24: Delete Service

```bash
kubectl delete svc tomcat-service -n tomcat
```

---

# Step 25: Delete Deployment

```bash
kubectl delete deployment tomcat-deployment -n tomcat
```

---

# Step 26: Delete Namespace

```bash
kubectl delete namespace tomcat
```

---

# Step 27: Delete Kind Cluster

```bash
kind delete cluster --name tomcat-cluster
```

Expected output:

```text
Deleting cluster "tomcat-cluster"
```

---

# Manifest Files

# tomcat-deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: tomcat-deployment
  namespace: tomcat

spec:
  replicas: 2

  selector:
    matchLabels:
      app: tomcat

  template:
    metadata:
      labels:
        app: tomcat

    spec:
      containers:

        - name: tomcat-container

          image: tomcat:9.0

          ports:
            - containerPort: 8080
```

---

# tomcat-service.yaml

```yaml
apiVersion: v1
kind: Service

metadata:
  name: tomcat-service
  namespace: tomcat

spec:
  type: NodePort

  selector:
    app: tomcat

  ports:
    - port: 8080
      targetPort: 8080
      nodePort: 32002
```

---

# Expected Outcome

- Kind Kubernetes cluster created successfully
- Tomcat deployed successfully
- Kubernetes Service configured
- Tomcat application accessible
- Deployment scaling operational
- Enterprise-grade Java deployment workflow implemented

---

# Real Industry Use Cases

| Use Case | Description |
|---|---|
| Enterprise Apps | Java web hosting |
| Banking Systems | Internal portals |
| APIs | Java backend services |
| Microservices | Java workloads |
| DevOps | Kubernetes deployments |

---

# Skills Covered

- Kubernetes Deployments
- Kubernetes Services
- Apache Tomcat
- NodePort Services
- Scaling Applications
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
Tomcat Pods
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

kind create cluster --name tomcat-cluster

kubectl cluster-info

kubectl get nodes

kubectl create namespace tomcat

nano tomcat-deployment.yaml

kubectl apply -f tomcat-deployment.yaml

kubectl get deployments -n tomcat

kubectl get pods -n tomcat

nano tomcat-service.yaml

kubectl apply -f tomcat-service.yaml

kubectl get svc -n tomcat

kubectl port-forward svc/tomcat-service \
8080:8080 -n tomcat

kubectl scale deployment tomcat-deployment \
--replicas=4 -n tomcat

kubectl describe deployment tomcat-deployment -n tomcat

kubectl describe svc tomcat-service -n tomcat

kubectl logs <pod-name> -n tomcat

kubectl get all -n tomcat

kubectl delete svc tomcat-service -n tomcat

kubectl delete deployment tomcat-deployment -n tomcat

kubectl delete namespace tomcat

kind delete cluster --name tomcat-cluster
```