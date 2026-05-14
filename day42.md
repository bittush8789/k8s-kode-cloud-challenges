# Kubernetes Troubleshooting — ImagePullBackOff (Using Private Docker Images)

# Normal Problem Statement

The DevOps team at xFusionCorp Industries is facing an ImagePullBackOff issue while deploying applications on Kubernetes because the container images are stored in a private Docker registry.

Your task is to create a Kubernetes cluster using Kind, configure private registry authentication using Kubernetes Secrets, deploy the application successfully, and troubleshoot ImagePullBackOff issues on an Ubuntu-based system.

---

# Scenario-Based Problem Statement

xFusionCorp Industries manages enterprise-grade Kubernetes workloads including:

- Banking applications
- AI/ML platforms
- Enterprise APIs
- Internal microservices
- Customer-facing applications

The organization recently migrated application images to a private container registry for security and compliance purposes.

However, Kubernetes workloads are failing with:

```text
ImagePullBackOff
```

and

```text
ErrImagePull
```

This issue is affecting:

- Production deployments
- CI/CD pipelines
- Enterprise applications
- Customer-facing APIs

The engineering team needs a secure solution for:

- Pulling private images
- Managing registry authentication
- Securing credentials
- Automating deployments

As a DevOps Engineer, your responsibilities are to:

1. Create Kubernetes cluster using Kind.
2. Diagnose ImagePullBackOff issue.
3. Configure Docker registry authentication.
4. Create Kubernetes image pull secrets.
5. Deploy application successfully.
6. Verify secure image pulling.
7. Build enterprise-grade Kubernetes troubleshooting workflow.

---

# Objectives

1. Install Kubernetes tools.
2. Create Kind cluster.
3. Deploy broken application.
4. Diagnose ImagePullBackOff.
5. Configure private registry secrets.
6. Fix deployment issue.
7. Verify successful rollout.

---

# What is ImagePullBackOff?

ImagePullBackOff occurs when Kubernetes cannot pull a container image.

Common reasons:

- Invalid image name
- Wrong image tag
- Private registry authentication failure
- Network issues
- Docker Hub rate limits

---

# Architecture Overview

```text
                 +----------------------+
                 | Kubernetes Cluster   |
                 +----------+-----------+
                            |
                            v
                 +----------------------+
                 | Kubernetes Secret    |
                 | (Docker Registry)    |
                 +----------+-----------+
                            |
                            v
                 +----------------------+
                 | Private Registry     |
                 | DockerHub / ECR      |
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
| Private DockerHub Images | Secure image storage |
| AWS ECR | Enterprise container registry |
| Azure ACR | Private registry |
| GCP Artifact Registry | Secure deployments |
| CI/CD Pipelines | Automated authentication |

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
kind create cluster --name private-registry-cluster
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
kubectl create namespace private-app
```

---

# Step 8: Create Broken Deployment

```bash
nano private-app.yaml
```

Add the following YAML:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: private-app
  namespace: private-app

spec:
  replicas: 2

  selector:
    matchLabels:
      app: private-app

  template:
    metadata:
      labels:
        app: private-app

    spec:
      containers:

        - name: private-container

          image: mydockerhub/private-app:v1

          ports:
            - containerPort: 80
```

---

# Step 9: Apply Deployment

```bash
kubectl apply -f private-app.yaml
```

Expected output:

```text
deployment.apps/private-app created
```

---

# Step 10: Verify Pod Failure

```bash
kubectl get pods -n private-app
```

Expected:

```text
ImagePullBackOff
```

---

# Step 11: Describe Pod

```bash
kubectl describe pod <pod-name> \
-n private-app
```

Expected error:

```text
Failed to pull image
pull access denied
```

---

# Step 12: Create Docker Registry Secret

Replace values with your DockerHub credentials:

```bash
kubectl create secret docker-registry regcred \
--docker-server=https://index.docker.io/v1/ \
--docker-username=<dockerhub-username> \
--docker-password=<dockerhub-password> \
--docker-email=<email> \
-n private-app
```

Expected:

```text
secret/regcred created
```

---

# Step 13: Verify Secret

```bash
kubectl get secrets -n private-app
```

---

# Step 14: Describe Secret

```bash
kubectl describe secret regcred \
-n private-app
```

---

# Step 15: Edit Deployment to Use Secret

```bash
kubectl edit deployment private-app \
-n private-app
```

Add the following section under:

```yaml
spec:
  template:
    spec:
```

Add:

```yaml
imagePullSecrets:
  - name: regcred
```

Final structure:

```yaml
spec:
  template:
    spec:

      imagePullSecrets:
        - name: regcred

      containers:
```

Save and exit.

---

# Step 16: Verify Rollout

```bash
kubectl rollout status deployment/private-app \
-n private-app
```

---

# Step 17: Verify Pods

```bash
kubectl get pods -n private-app
```

Expected:

```text
Running
```

---

# Step 18: Create Application Service

```bash
nano private-app-service.yaml
```

Add:

```yaml
apiVersion: v1
kind: Service

metadata:
  name: private-app-service
  namespace: private-app

spec:
  selector:
    app: private-app

  ports:
    - port: 80
      targetPort: 80
      nodePort: 32011

  type: NodePort
```

---

# Step 19: Apply Service

```bash
kubectl apply -f private-app-service.yaml
```

---

# Step 20: Verify Service

```bash
kubectl get svc -n private-app
```

---

# Step 21: Verify Endpoints

```bash
kubectl get endpoints -n private-app
```

Expected:

```text
Pod IPs visible
```

---

# Step 22: Port Forward Service

```bash
kubectl port-forward svc/private-app-service \
8080:80 -n private-app
```

---

# Step 23: Access Application

Open browser:

```text
http://localhost:8080
```

Expected:

```text
Application page
```

---

# Step 24: Verify Deployment Details

```bash
kubectl describe deployment private-app \
-n private-app
```

---

# Step 25: Check Application Logs

Get Pod names:

```bash
kubectl get pods -n private-app
```

Check logs:

```bash
kubectl logs <pod-name> -n private-app
```

---

# Step 26: Scale Deployment

```bash
kubectl scale deployment private-app \
--replicas=4 -n private-app
```

Verify:

```bash
kubectl get deployments -n private-app
```

Expected:

```text
READY 4/4
```

---

# Step 27: Perform Rolling Update

Update image:

```bash
kubectl set image deployment/private-app \
private-container=mydockerhub/private-app:v2 \
-n private-app
```

---

# Step 28: Verify Rollout

```bash
kubectl rollout status deployment/private-app \
-n private-app
```

---

# Step 29: Verify Rollout History

```bash
kubectl rollout history deployment/private-app \
-n private-app
```

---

# Step 30: Rollback Deployment

```bash
kubectl rollout undo deployment/private-app \
-n private-app
```

---

# Common Private Registry Types

| Registry | Example |
|---|---|
| DockerHub | docker.io |
| AWS ECR | *.amazonaws.com |
| Azure ACR | *.azurecr.io |
| GCP Artifact Registry | *.pkg.dev |
| Harbor Registry | Private enterprise registry |

---

# Example AWS ECR Authentication

```bash
aws ecr get-login-password \
| docker login \
--username AWS \
--password-stdin <account>.dkr.ecr.region.amazonaws.com
```

Create Kubernetes secret:

```bash
kubectl create secret docker-registry ecr-secret \
--docker-server=<ecr-url> \
--docker-username=AWS \
--docker-password=<token>
```

---

# Example Azure ACR Authentication

```bash
az acr login --name myregistry
```

---

# Example GCP Artifact Registry Authentication

```bash
gcloud auth configure-docker
```

---

# Troubleshooting Commands

| Command | Purpose |
|---|---|
| kubectl get pods | Pod status |
| kubectl describe pod | Detailed pod info |
| kubectl logs | Container logs |
| kubectl get events | Cluster events |
| kubectl rollout status | Deployment rollout |

---

# Common Kubernetes Errors

| Error | Meaning |
|---|---|
| ImagePullBackOff | Image pull failed |
| ErrImagePull | Registry/auth issue |
| Unauthorized | Invalid credentials |
| NotFound | Wrong image name |
| CrashLoopBackOff | App crash |

---

# Real Production Troubleshooting Workflow

```text
Application Deployment Failure
              |
              v
Check Pod Status
              |
              v
Describe Pod
              |
              v
Identify Registry Error
              |
              v
Create Registry Secret
              |
              v
Attach imagePullSecrets
              |
              v
Redeploy Application
              |
              v
Verify Healthy Pods
```

---

# Step 31: Verify Cluster Resources

```bash
kubectl get all -n private-app

kubectl get secrets -n private-app
```

Expected:

```text
Pods
Services
Deployments
ReplicaSets
Secrets
```

---

# Step 32: Delete Resources

```bash
kubectl delete deployment private-app \
-n private-app

kubectl delete svc private-app-service \
-n private-app

kubectl delete secret regcred \
-n private-app
```

---

# Step 33: Delete Namespace

```bash
kubectl delete namespace private-app
```

---

# Step 34: Delete Kind Cluster

```bash
kind delete cluster --name private-registry-cluster
```

---

# Manifest Files

# private-app.yaml

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: private-app
  namespace: private-app

spec:
  replicas: 2

  selector:
    matchLabels:
      app: private-app

  template:
    metadata:
      labels:
        app: private-app

    spec:

      imagePullSecrets:
        - name: regcred

      containers:

        - name: private-container

          image: mydockerhub/private-app:v1

          ports:
            - containerPort: 80
```

---

# private-app-service.yaml

```yaml
apiVersion: v1
kind: Service

metadata:
  name: private-app-service
  namespace: private-app

spec:
  selector:
    app: private-app

  ports:
    - port: 80
      targetPort: 80
      nodePort: 32011

  type: NodePort
```

---

# Expected Outcome

- Kind Kubernetes cluster created successfully
- ImagePullBackOff diagnosed successfully
- Docker registry secret configured
- Private image pulled successfully
- Application deployed successfully
- Scaling operational
- Rolling updates operational
- Rollback operational
- Enterprise-grade troubleshooting workflow implemented

---

# Skills Covered

- Kubernetes Troubleshooting
- ImagePullBackOff Resolution
- Private Docker Registries
- Kubernetes Secrets
- imagePullSecrets
- Kubernetes Deployments
- Kubernetes Services
- Rolling Updates
- Rollbacks
- DevOps
- Cloud-Native Security

---

# Real Enterprise Workflow

```text
Developer Push
       |
       v
Private Container Registry
       |
       v
Kubernetes Secret
       |
       v
imagePullSecrets
       |
       v
Kubernetes Deployment
       |
       v
Pods Pull Private Images
       |
       v
Application Running
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

kind create cluster --name private-registry-cluster

kubectl create namespace private-app

kubectl apply -f private-app.yaml

kubectl get pods -n private-app

kubectl describe pod <pod-name> \
-n private-app

kubectl create secret docker-registry regcred \
--docker-server=https://index.docker.io/v1/ \
--docker-username=<dockerhub-username> \
--docker-password=<dockerhub-password> \
--docker-email=<email> \
-n private-app

kubectl edit deployment private-app \
-n private-app

kubectl rollout status deployment/private-app \
-n private-app

kubectl get pods -n private-app

kubectl apply -f private-app-service.yaml

kubectl get svc -n private-app

kubectl get endpoints -n private-app

kubectl port-forward svc/private-app-service \
8080:80 -n private-app

kubectl logs <pod-name> -n private-app

kubectl scale deployment private-app \
--replicas=4 -n private-app

kubectl rollout history deployment/private-app \
-n private-app

kubectl rollout undo deployment/private-app \
-n private-app

kubectl get all -n private-app

kubectl delete namespace private-app

kind delete cluster --name private-registry-cluster
```