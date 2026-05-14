# Deploy Nginx Web Server on Kubernetes Cluster

# Normal Problem Statement

The DevOps team at xFusionCorp Industries wants to deploy an Nginx web server inside a Kubernetes cluster for hosting internal web applications and testing containerized workloads.

Your task is to create a Kubernetes cluster using Kind, deploy an Nginx web server using Kubernetes Deployment and Service manifests, expose the application, and verify accessibility on an Ubuntu-based system.

---

# Scenario-Based Problem Statement

xFusionCorp Industries is modernizing its infrastructure by migrating applications to Kubernetes.

The organization wants to:

- Deploy containerized web applications
- Host internal dashboards
- Run reverse proxy services
- Build scalable cloud-native platforms
- Implement microservices architecture

The infrastructure team currently faces challenges:

- Manual application deployment
- No orchestration platform
- Difficult scaling workflows
- Lack of container management
- No high availability

To solve these problems, the organization wants to deploy applications on Kubernetes.

As a DevOps Engineer, your responsibilities are to:

1. Create a Kubernetes cluster using Kind.
2. Deploy an Nginx web server.
3. Configure Kubernetes Deployment objects.
4. Expose applications using Services.
5. Verify Pod communication.
6. Troubleshoot Kubernetes workloads.

---

# Objectives

1. Install Kubernetes tools.
2. Create a Kind cluster.
3. Deploy Nginx application.
4. Configure Kubernetes Service.
5. Access the web server externally.
6. Verify application availability.

---

# Architecture Overview

```text
                    +-------------------+
                    |      Users        |
                    +---------+---------+
                              |
                              v
                    +-------------------+
                    | Kubernetes Service|
                    +---------+---------+
                              |
                              v
                    +-------------------+
                    |   Nginx Pods      |
                    +-------------------+
                              |
                              v
                    +-------------------+
                    | Kubernetes Cluster|
                    +-------------------+
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
kind create cluster --name nginx-cluster
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

# Step 6: Verify Cluster Nodes

```bash
kubectl get nodes
```

Expected output:

```text
NAME                         STATUS   ROLES           AGE
nginx-cluster-control-plane  Ready    control-plane   1m
```

---

# Step 7: Create Nginx Deployment Manifest

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

          image: nginx:latest

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
nginx-deployment   2/2     2            2
```

---

# Step 10: Verify Pods

```bash
kubectl get pods
```

Expected output:

```text
NAME                                READY   STATUS
nginx-deployment-xxxx              1/1     Running
nginx-deployment-yyyy              1/1     Running
```

---

# Step 11: Inspect Pod Details

```bash
kubectl describe pod <pod-name>
```

Example:

```bash
kubectl describe pod nginx-deployment-xxxxx
```

Verify:

- Pod status
- Container image
- Container ports
- Events section

---

# Step 12: Create Kubernetes Service Manifest

```bash
nano nginx-service.yaml
```

Add the following YAML:

```yaml
apiVersion: v1
kind: Service

metadata:
  name: nginx-service

spec:
  selector:
    app: nginx

  type: NodePort

  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 30080
```

---

# Step 13: Apply Service Manifest

```bash
kubectl apply -f nginx-service.yaml
```

Expected output:

```text
service/nginx-service created
```

---

# Step 14: Verify Service

```bash
kubectl get svc
```

Expected output:

```text
NAME            TYPE       CLUSTER-IP      PORT(S)
nginx-service   NodePort   10.x.x.x        80:30080/TCP
```

---

# Step 15: Verify Service Endpoints

```bash
kubectl get endpoints
```

Expected output:

```text
nginx-service   <pod-ip>:80,<pod-ip>:80
```

---

# Step 16: Access Nginx Application

For Kind cluster:

Get cluster container:

```bash
docker ps
```

Port-forward service:

```bash
kubectl port-forward service/nginx-service 8080:80
```

Open browser:

```text
http://localhost:8080
```

Expected:

```text
Welcome to nginx!
```

---

# Step 17: Test Application Using Curl

```bash
curl http://localhost:8080
```

Expected output:

```html
Welcome to nginx!
```

---

# Step 18: Scale Deployment

Scale replicas:

```bash
kubectl scale deployment nginx-deployment --replicas=4
```

Verify:

```bash
kubectl get deployments
```

Expected output:

```text
READY   4/4
```

---

# Step 19: Verify New Pods

```bash
kubectl get pods
```

Expected:

```text
4 Running Pods
```

---

# Step 20: Delete One Pod

```bash
kubectl delete pod <pod-name>
```

Example:

```bash
kubectl delete pod nginx-deployment-xxxxx
```

Observe Kubernetes self-healing:

```bash
kubectl get pods
```

New Pod should automatically start.

---

# Step 21: View Deployment Logs

```bash
kubectl logs <pod-name>
```

Example:

```bash
kubectl logs nginx-deployment-xxxxx
```

---

# Step 22: Delete Service

```bash
kubectl delete service nginx-service
```

---

# Step 23: Delete Deployment

```bash
kubectl delete deployment nginx-deployment
```

---

# Step 24: Delete Kind Cluster

```bash
kind delete cluster --name nginx-cluster
```

Expected output:

```text
Deleting cluster "nginx-cluster"
```

---

# Manifest Files

# nginx-deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: nginx-deployment

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

          image: nginx:latest

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

spec:
  selector:
    app: nginx

  type: NodePort

  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 30080
```

---

# Expected Outcome

- Kind Kubernetes cluster created successfully
- Nginx Deployment configured properly
- Multiple Pods running successfully
- Kubernetes Service exposed correctly
- Application accessible externally
- Scaling and self-healing verified

---

# Real Industry Use Cases

| Use Case | Description |
|---|---|
| Reverse Proxy | API gateway |
| Static Hosting | Web applications |
| Load Balancer | Traffic routing |
| Microservices | Frontend applications |
| Kubernetes Testing | Learning environments |

---

# Skills Covered

- Kubernetes Deployment
- Kubernetes Service
- NodePort Service
- Pod Management
- Scaling
- Self-Healing
- Kubernetes Networking
- DevOps
- Cloud-Native Deployment

---

# Real Enterprise Deployment Workflow

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
Pods Created
       |
       v
Service Exposure
       |
       v
Application Access
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

kind create cluster --name nginx-cluster

kubectl cluster-info

kubectl get nodes

nano nginx-deployment.yaml

kubectl apply -f nginx-deployment.yaml

kubectl get deployments

kubectl get pods

kubectl describe pod <pod-name>

nano nginx-service.yaml

kubectl apply -f nginx-service.yaml

kubectl get svc

kubectl get endpoints

kubectl port-forward service/nginx-service 8080:80

curl http://localhost:8080

kubectl scale deployment nginx-deployment --replicas=4

kubectl get pods

kubectl delete pod <pod-name>

kubectl logs <pod-name>

kubectl delete service nginx-service

kubectl delete deployment nginx-deployment

kind delete cluster --name nginx-cluster
```