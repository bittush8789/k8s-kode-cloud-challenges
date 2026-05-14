# Manage Secrets in Kubernetes

# Normal Problem Statement

The DevOps team at xFusionCorp Industries wants to securely manage sensitive application data in Kubernetes such as passwords, API keys, database credentials, and tokens.

Your task is to create a Kubernetes cluster using Kind, configure Kubernetes Secrets, deploy applications using secrets, and verify secure secret management on an Ubuntu-based system.

---

# Scenario-Based Problem Statement

xFusionCorp Industries manages enterprise-grade Kubernetes workloads including:

- Banking applications
- AI/ML platforms
- Internal APIs
- CI/CD pipelines
- Monitoring systems

The organization currently faces several security challenges:

- Hardcoded passwords inside YAML files
- Exposed API keys
- Insecure environment variables
- Database credential leakage
- Compliance and security risks

Applications affected:

- MySQL databases
- Jenkins pipelines
- APIs
- Enterprise applications

To solve these issues, the organization wants to implement Kubernetes Secrets.

As a DevOps Engineer, your responsibilities are to:

1. Create Kubernetes cluster using Kind.
2. Configure Kubernetes Secrets.
3. Store sensitive credentials securely.
4. Inject secrets into Pods.
5. Verify secret access.
6. Implement enterprise-grade secret management.

---

# Objectives

1. Install Kubernetes tools.
2. Create Kind cluster.
3. Create Kubernetes Secrets.
4. Inject secrets into Pods.
5. Access secrets securely.
6. Troubleshoot secret issues.
7. Learn production secret workflows.

---

# What are Kubernetes Secrets?

Kubernetes Secrets are objects used to securely store sensitive information such as:

- Passwords
- Tokens
- API Keys
- Database credentials
- Certificates

---

# Why Use Secrets?

Without Secrets:

```yaml
password: mypassword123
```

Problems:

- Exposed credentials
- Security risks
- Compliance failures

With Secrets:

- Secure credential storage
- Better security practices
- Production-ready architecture

---

# Architecture Overview

```text
                +----------------------+
                |    Kubernetes        |
                +----------+-----------+
                           |
                           v
                +----------------------+
                | Kubernetes Secret    |
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
| Database Passwords | Secure DB access |
| API Tokens | Authentication |
| CI/CD Credentials | Pipeline security |
| TLS Certificates | HTTPS security |
| Cloud Credentials | AWS/GCP access |

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
kind create cluster --name secret-cluster
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
kubectl create namespace secret-demo
```

---

# Step 8: Create Secret Using kubectl Command

```bash
kubectl create secret generic db-secret \
--from-literal=username=admin \
--from-literal=password=admin123 \
-n secret-demo
```

Expected output:

```text
secret/db-secret created
```

---

# Step 9: Verify Secret

```bash
kubectl get secrets -n secret-demo
```

Expected:

```text
db-secret
```

---

# Step 10: Describe Secret

```bash
kubectl describe secret db-secret \
-n secret-demo
```

Expected:

```text
Type: Opaque
```

Note:

Secret values are hidden for security reasons.

---

# Step 11: View Secret YAML

```bash
kubectl get secret db-secret \
-n secret-demo -o yaml
```

Expected:

```yaml
data:
  username: YWRtaW4=
  password: YWRtaW4xMjM=
```

These values are Base64 encoded.

---

# Step 12: Decode Secret Values

Decode username:

```bash
echo YWRtaW4= | base64 --decode
```

Expected:

```text
admin
```

Decode password:

```bash
echo YWRtaW4xMjM= | base64 --decode
```

Expected:

```text
admin123
```

---

# Step 13: Create Secret Manifest File

```bash
nano app-secret.yaml
```

Add the following YAML:

```yaml
apiVersion: v1
kind: Secret

metadata:
  name: app-secret
  namespace: secret-demo

type: Opaque

data:
  username: YWRtaW4=
  password: YWRtaW4xMjM=
```

---

# Step 14: Apply Secret Manifest

```bash
kubectl apply -f app-secret.yaml
```

Expected output:

```text
secret/app-secret created
```

---

# Step 15: Create Pod Using Secrets

```bash
nano secret-pod.yaml
```

Add the following YAML:

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: secret-pod
  namespace: secret-demo

spec:
  containers:

    - name: nginx-container

      image: nginx

      env:
        - name: DB_USERNAME
          valueFrom:
            secretKeyRef:
              name: app-secret
              key: username

        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: app-secret
              key: password
```

---

# Step 16: Apply Pod Manifest

```bash
kubectl apply -f secret-pod.yaml
```

Expected output:

```text
pod/secret-pod created
```

---

# Step 17: Verify Pod

```bash
kubectl get pods -n secret-demo
```

Expected:

```text
Running
```

---

# Step 18: Access Pod Shell

```bash
kubectl exec -it secret-pod \
-n secret-demo -- sh
```

---

# Step 19: Verify Environment Variables

Inside Pod:

```bash
echo $DB_USERNAME
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

# Step 20: Create Secret as Volume

```bash
nano secret-volume-pod.yaml
```

Add:

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: secret-volume-pod
  namespace: secret-demo

spec:
  containers:

    - name: nginx

      image: nginx

      volumeMounts:
        - name: secret-volume
          mountPath: /etc/secret-data
          readOnly: true

  volumes:
    - name: secret-volume

      secret:
        secretName: app-secret
```

---

# Step 21: Apply Volume Secret Pod

```bash
kubectl apply -f secret-volume-pod.yaml
```

---

# Step 22: Verify Pod

```bash
kubectl get pods -n secret-demo
```

---

# Step 23: Access Secret Files

Access shell:

```bash
kubectl exec -it secret-volume-pod \
-n secret-demo -- sh
```

Check secret files:

```bash
ls /etc/secret-data
```

Expected:

```text
password
username
```

View username:

```bash
cat /etc/secret-data/username
```

Expected:

```text
admin
```

Exit shell:

```bash
exit
```

---

# Step 24: Troubleshooting Secret Issues

# Describe Pod

```bash
kubectl describe pod secret-pod \
-n secret-demo
```

---

# Check Events

```bash
kubectl get events -n secret-demo
```

---

# Common Secret Issues

| Issue | Description |
|---|---|
| Secret Not Found | Wrong secret name |
| Key Missing | Invalid secret key |
| Pod Crash | Missing environment variable |
| Access Denied | RBAC issue |

---

# Step 25: Verify All Resources

```bash
kubectl get all -n secret-demo

kubectl get secrets -n secret-demo
```

---

# Step 26: Delete Resources

```bash
kubectl delete pod secret-pod -n secret-demo

kubectl delete pod secret-volume-pod -n secret-demo

kubectl delete secret db-secret -n secret-demo

kubectl delete secret app-secret -n secret-demo
```

---

# Step 27: Delete Namespace

```bash
kubectl delete namespace secret-demo
```

---

# Step 28: Delete Kind Cluster

```bash
kind delete cluster --name secret-cluster
```

---

# Manifest Files

# app-secret.yaml

```yaml
apiVersion: v1
kind: Secret

metadata:
  name: app-secret
  namespace: secret-demo

type: Opaque

data:
  username: YWRtaW4=
  password: YWRtaW4xMjM=
```

---

# secret-pod.yaml

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: secret-pod
  namespace: secret-demo

spec:
  containers:

    - name: nginx-container

      image: nginx

      env:
        - name: DB_USERNAME
          valueFrom:
            secretKeyRef:
              name: app-secret
              key: username

        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: app-secret
              key: password
```

---

# secret-volume-pod.yaml

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: secret-volume-pod
  namespace: secret-demo

spec:
  containers:

    - name: nginx

      image: nginx

      volumeMounts:
        - name: secret-volume
          mountPath: /etc/secret-data
          readOnly: true

  volumes:
    - name: secret-volume

      secret:
        secretName: app-secret
```

---

# Expected Outcome

- Kind Kubernetes cluster created successfully
- Kubernetes Secrets configured
- Sensitive credentials stored securely
- Secrets injected into Pods
- Secret volumes mounted successfully
- Enterprise-grade secret management workflow implemented

---

# Skills Covered

- Kubernetes Secrets
- Secret Management
- Environment Variables
- Secret Volumes
- Kubernetes Security
- DevOps Security
- Cloud-Native Security
- Kubernetes Troubleshooting

---

# Real Enterprise Workflow

```text
Sensitive Credentials
        |
        v
Kubernetes Secret
        |
        v
Injected into Pods
        |
        +----------------+
        |                |
        v                v
Environment Vars    Mounted Files
        |
        v
Secure Application Access
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

kind create cluster --name secret-cluster

kubectl create namespace secret-demo

kubectl create secret generic db-secret \
--from-literal=username=admin \
--from-literal=password=admin123 \
-n secret-demo

kubectl get secrets -n secret-demo

kubectl describe secret db-secret \
-n secret-demo

kubectl get secret db-secret \
-n secret-demo -o yaml

kubectl apply -f app-secret.yaml

kubectl apply -f secret-pod.yaml

kubectl exec -it secret-pod \
-n secret-demo -- sh

kubectl apply -f secret-volume-pod.yaml

kubectl exec -it secret-volume-pod \
-n secret-demo -- sh

kubectl get all -n secret-demo

kubectl delete namespace secret-demo

kind delete cluster --name secret-cluster
```