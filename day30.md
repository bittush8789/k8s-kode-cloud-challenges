# Persistent Volumes in Kubernetes

# Normal Problem Statement

The DevOps team at xFusionCorp Industries wants to implement Persistent Volumes in Kubernetes to store application data permanently, even if Pods are deleted or restarted.

Your task is to create a Kubernetes cluster using Kind, configure Persistent Volumes (PV) and Persistent Volume Claims (PVC), deploy applications using persistent storage, and verify data persistence on an Ubuntu-based system.

---

# Scenario-Based Problem Statement

xFusionCorp Industries manages enterprise-grade Kubernetes workloads including:

- Databases
- CI/CD systems
- Monitoring platforms
- AI/ML applications
- Web applications

The engineering team currently faces several critical issues:

- Data loss after Pod restart
- Temporary container storage
- Stateless application limitations
- No persistent database storage
- Unstable production environments

Applications affected:

- MySQL databases
- Jenkins CI/CD pipelines
- Grafana dashboards
- Enterprise applications

To solve these problems, the organization wants to implement Persistent Volumes in Kubernetes.

As a DevOps Engineer, your responsibilities are to:

1. Create Kubernetes cluster using Kind.
2. Configure Persistent Volumes.
3. Configure Persistent Volume Claims.
4. Attach persistent storage to Pods.
5. Verify data persistence after Pod deletion.
6. Implement enterprise-grade storage architecture.

---

# Objectives

1. Install Kubernetes tools.
2. Create Kind cluster.
3. Create Persistent Volumes.
4. Create Persistent Volume Claims.
5. Mount storage into Pods.
6. Verify persistent storage.
7. Troubleshoot storage issues.

---

# What is Persistent Volume?

Persistent Volume (PV) is Kubernetes storage that exists independently from Pods.

Benefits:

- Data survives Pod restart
- Shared storage management
- Persistent databases
- Enterprise-grade storage

---

# What is Persistent Volume Claim?

Persistent Volume Claim (PVC) is a storage request made by a Pod.

---

# Architecture Overview

```text
                +----------------------+
                |     Kubernetes       |
                +----------+-----------+
                           |
                           v
                +----------------------+
                | Persistent Volume    |
                +----------+-----------+
                           |
                           v
                +----------------------+
                | Persistent Volume    |
                | Claim (PVC)          |
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
| MySQL Databases | Persistent DB storage |
| Jenkins | Pipeline persistence |
| Grafana | Dashboard storage |
| AI/ML | Model storage |
| Enterprise Apps | Persistent application data |

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
kind create cluster --name pv-cluster
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
kubectl create namespace storage-demo
```

---

# Step 8: Create Persistent Volume Manifest

```bash
nano persistent-volume.yaml
```

Add the following YAML:

```yaml
apiVersion: v1
kind: PersistentVolume

metadata:
  name: app-pv

spec:
  capacity:
    storage: 1Gi

  accessModes:
    - ReadWriteOnce

  hostPath:
    path: /data/app-storage
```

---

# Step 9: Apply Persistent Volume

```bash
kubectl apply -f persistent-volume.yaml
```

Expected output:

```text
persistentvolume/app-pv created
```

---

# Step 10: Verify Persistent Volume

```bash
kubectl get pv
```

Expected:

```text
STATUS: Available
```

---

# Step 11: Create Persistent Volume Claim Manifest

```bash
nano persistent-volume-claim.yaml
```

Add the following YAML:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim

metadata:
  name: app-pvc
  namespace: storage-demo

spec:
  accessModes:
    - ReadWriteOnce

  resources:
    requests:
      storage: 1Gi
```

---

# Step 12: Apply PVC Manifest

```bash
kubectl apply -f persistent-volume-claim.yaml
```

Expected output:

```text
persistentvolumeclaim/app-pvc created
```

---

# Step 13: Verify PVC

```bash
kubectl get pvc -n storage-demo
```

Expected:

```text
STATUS: Bound
```

---

# Step 14: Verify PV Binding

```bash
kubectl get pv
```

Expected:

```text
STATUS: Bound
```

---

# Step 15: Create Pod Using Persistent Storage

```bash
nano app-pod.yaml
```

Add the following YAML:

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: storage-pod
  namespace: storage-demo

spec:
  containers:

    - name: nginx-container

      image: nginx

      volumeMounts:
        - name: app-storage
          mountPath: /usr/share/nginx/html

  volumes:
    - name: app-storage

      persistentVolumeClaim:
        claimName: app-pvc
```

---

# Step 16: Apply Pod Manifest

```bash
kubectl apply -f app-pod.yaml
```

Expected output:

```text
pod/storage-pod created
```

---

# Step 17: Verify Pod

```bash
kubectl get pods -n storage-demo
```

Expected:

```text
Running
```

---

# Step 18: Access Pod Shell

```bash
kubectl exec -it storage-pod \
-n storage-demo -- sh
```

---

# Step 19: Create Persistent File

Inside container:

```bash
echo "Persistent Kubernetes Storage" \
> /usr/share/nginx/html/index.html
```

Verify:

```bash
cat /usr/share/nginx/html/index.html
```

Expected:

```text
Persistent Kubernetes Storage
```

Exit shell:

```bash
exit
```

---

# Step 20: Delete Pod

```bash
kubectl delete pod storage-pod \
-n storage-demo
```

---

# Step 21: Recreate Pod

```bash
kubectl apply -f app-pod.yaml
```

---

# Step 22: Verify Pod Running

```bash
kubectl get pods -n storage-demo
```

---

# Step 23: Verify Data Persistence

Access shell:

```bash
kubectl exec -it storage-pod \
-n storage-demo -- sh
```

Check file:

```bash
cat /usr/share/nginx/html/index.html
```

Expected:

```text
Persistent Kubernetes Storage
```

Data survived Pod deletion.

Exit shell:

```bash
exit
```

---

# Step 24: Create Service for Pod

```bash
nano storage-service.yaml
```

Add:

```yaml
apiVersion: v1
kind: Service

metadata:
  name: storage-service
  namespace: storage-demo

spec:
  selector:
    app: storage-app

  ports:
    - port: 80
      targetPort: 80

  type: NodePort
```

---

# Step 25: Add Labels to Pod

Edit Pod:

```bash
kubectl edit pod storage-pod \
-n storage-demo
```

Add:

```yaml
labels:
  app: storage-app
```

---

# Step 26: Apply Service

```bash
kubectl apply -f storage-service.yaml
```

---

# Step 27: Verify Service

```bash
kubectl get svc -n storage-demo
```

---

# Step 28: Troubleshooting Storage Issues

# Check PV

```bash
kubectl describe pv app-pv
```

---

# Check PVC

```bash
kubectl describe pvc app-pvc \
-n storage-demo
```

---

# Check Pod Events

```bash
kubectl describe pod storage-pod \
-n storage-demo
```

---

# Step 29: Common Storage Issues

| Issue | Description |
|---|---|
| Pending PVC | No matching PV |
| Mount Failure | Invalid mount path |
| Access Denied | Permission issue |
| Lost Data | Wrong storage type |

---

# Step 30: Verify All Resources

```bash
kubectl get all -n storage-demo

kubectl get pv

kubectl get pvc -n storage-demo
```

---

# Step 31: Delete Resources

```bash
kubectl delete pod storage-pod -n storage-demo

kubectl delete pvc app-pvc -n storage-demo

kubectl delete pv app-pv

kubectl delete svc storage-service -n storage-demo
```

---

# Step 32: Delete Namespace

```bash
kubectl delete namespace storage-demo
```

---

# Step 33: Delete Kind Cluster

```bash
kind delete cluster --name pv-cluster
```

---

# Manifest Files

# persistent-volume.yaml

```yaml
apiVersion: v1
kind: PersistentVolume

metadata:
  name: app-pv

spec:
  capacity:
    storage: 1Gi

  accessModes:
    - ReadWriteOnce

  hostPath:
    path: /data/app-storage
```

---

# persistent-volume-claim.yaml

```yaml
apiVersion: v1
kind: PersistentVolumeClaim

metadata:
  name: app-pvc
  namespace: storage-demo

spec:
  accessModes:
    - ReadWriteOnce

  resources:
    requests:
      storage: 1Gi
```

---

# app-pod.yaml

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: storage-pod
  namespace: storage-demo

spec:
  containers:

    - name: nginx-container

      image: nginx

      volumeMounts:
        - name: app-storage
          mountPath: /usr/share/nginx/html

  volumes:
    - name: app-storage

      persistentVolumeClaim:
        claimName: app-pvc
```

---

# storage-service.yaml

```yaml
apiVersion: v1
kind: Service

metadata:
  name: storage-service
  namespace: storage-demo

spec:
  selector:
    app: storage-app

  ports:
    - port: 80
      targetPort: 80

  type: NodePort
```

---

# Expected Outcome

- Kind Kubernetes cluster created successfully
- Persistent Volume configured
- Persistent Volume Claim configured
- Pod attached to persistent storage
- Data persisted after Pod deletion
- Enterprise-grade storage workflow implemented

---

# Skills Covered

- Persistent Volumes
- Persistent Volume Claims
- Kubernetes Storage
- Volume Mounts
- Kubernetes Troubleshooting
- Stateful Applications
- DevOps
- Cloud-Native Storage

---

# Real Enterprise Workflow

```text
Application Request
        |
        v
Persistent Volume Claim
        |
        v
Persistent Volume Bound
        |
        v
Storage Mounted to Pod
        |
        v
Persistent Application Data
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

kind create cluster --name pv-cluster

kubectl create namespace storage-demo

kubectl apply -f persistent-volume.yaml

kubectl get pv

kubectl apply -f persistent-volume-claim.yaml

kubectl get pvc -n storage-demo

kubectl apply -f app-pod.yaml

kubectl get pods -n storage-demo

kubectl exec -it storage-pod \
-n storage-demo -- sh

kubectl delete pod storage-pod \
-n storage-demo

kubectl apply -f app-pod.yaml

kubectl describe pv app-pv

kubectl describe pvc app-pvc \
-n storage-demo

kubectl describe pod storage-pod \
-n storage-demo

kubectl get all -n storage-demo

kubectl delete namespace storage-demo

kind delete cluster --name pv-cluster
```