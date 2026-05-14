# Deploy Grafana on Kubernetes Cluster

# Normal Problem Statement

The DevOps team at xFusionCorp Industries wants to deploy Grafana on a Kubernetes cluster for monitoring and visualizing infrastructure and application metrics.

Your task is to create a Kubernetes cluster using Kind, deploy Grafana using Kubernetes manifests, expose the Grafana dashboard, and verify the deployment on an Ubuntu-based system.

---

# Scenario-Based Problem Statement

xFusionCorp Industries manages enterprise-scale platforms including:

- Kubernetes clusters
- AI/ML platforms
- CI/CD pipelines
- Cloud-native applications
- Microservices

The Platform Engineering team currently faces multiple monitoring challenges:

- No centralized dashboards
- Limited infrastructure visibility
- Poor observability
- Difficulty monitoring Kubernetes workloads
- No real-time visualization system

To solve these problems, the organization wants to deploy Grafana on Kubernetes.

As a DevOps Engineer, your responsibilities are to:

1. Create a Kubernetes cluster using Kind.
2. Deploy Grafana on Kubernetes.
3. Configure persistent storage.
4. Expose Grafana dashboard.
5. Access Grafana UI.
6. Configure monitoring dashboards.
7. Build observability infrastructure.

---

# Objectives

1. Install Kubernetes tools.
2. Create a Kind cluster.
3. Deploy Grafana.
4. Configure persistent volumes.
5. Expose Grafana service.
6. Access Grafana dashboard.
7. Verify monitoring setup.

---

# What is Grafana?

Grafana is an open-source monitoring and visualization platform used for:

- Infrastructure monitoring
- Kubernetes monitoring
- Application observability
- Metrics visualization
- Alerting
- Dashboard management

---

# Architecture Overview

```text
                    +----------------+
                    |   Developers   |
                    +--------+-------+
                             |
                             v
                    +----------------+
                    | Grafana Service|
                    +--------+-------+
                             |
                             v
                    +----------------+
                    | Grafana Pod    |
                    +--------+-------+
                             |
                             v
                    +----------------+
                    | Persistent Vol |
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
kind create cluster --name grafana-cluster
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
NAME                          STATUS   ROLES
grafana-cluster-control-plane Ready    control-plane
```

---

# Step 7: Create Grafana Namespace

```bash
kubectl create namespace grafana
```

Verify:

```bash
kubectl get namespaces
```

---

# Step 8: Create Persistent Volume Manifest

```bash
nano grafana-pv.yaml
```

Add the following YAML:

```yaml
apiVersion: v1
kind: PersistentVolume

metadata:
  name: grafana-pv

spec:
  capacity:
    storage: 2Gi

  accessModes:
    - ReadWriteOnce

  hostPath:
    path: /data/grafana-volume
```

---

# Step 9: Apply Persistent Volume

```bash
kubectl apply -f grafana-pv.yaml
```

Expected output:

```text
persistentvolume/grafana-pv created
```

---

# Step 10: Create Persistent Volume Claim

```bash
nano grafana-pvc.yaml
```

Add the following YAML:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim

metadata:
  name: grafana-pvc
  namespace: grafana

spec:
  accessModes:
    - ReadWriteOnce

  resources:
    requests:
      storage: 2Gi
```

---

# Step 11: Apply PVC Manifest

```bash
kubectl apply -f grafana-pvc.yaml
```

Expected output:

```text
persistentvolumeclaim/grafana-pvc created
```

---

# Step 12: Create Grafana Deployment Manifest

```bash
nano grafana-deployment.yaml
```

Add the following YAML:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: grafana
  namespace: grafana

spec:
  replicas: 1

  selector:
    matchLabels:
      app: grafana

  template:
    metadata:
      labels:
        app: grafana

    spec:
      containers:

        - name: grafana

          image: grafana/grafana:latest

          ports:
            - containerPort: 3000

          volumeMounts:
            - name: grafana-storage
              mountPath: /var/lib/grafana

      volumes:
        - name: grafana-storage

          persistentVolumeClaim:
            claimName: grafana-pvc
```

---

# Step 13: Apply Grafana Deployment

```bash
kubectl apply -f grafana-deployment.yaml
```

Expected output:

```text
deployment.apps/grafana created
```

---

# Step 14: Verify Grafana Pod

```bash
kubectl get pods -n grafana
```

Expected output:

```text
NAME                       READY   STATUS
grafana-xxxxxx             1/1     Running
```

---

# Step 15: Create Grafana Service Manifest

```bash
nano grafana-service.yaml
```

Add the following YAML:

```yaml
apiVersion: v1
kind: Service

metadata:
  name: grafana-service
  namespace: grafana

spec:
  type: NodePort

  selector:
    app: grafana

  ports:
    - port: 3000
      targetPort: 3000
      nodePort: 32001
```

---

# Step 16: Apply Grafana Service

```bash
kubectl apply -f grafana-service.yaml
```

Expected output:

```text
service/grafana-service created
```

---

# Step 17: Verify Service

```bash
kubectl get svc -n grafana
```

Expected output:

```text
NAME               TYPE       PORT(S)
grafana-service    NodePort   3000:32001/TCP
```

---

# Step 18: Port Forward Grafana Service

```bash
kubectl port-forward svc/grafana-service \
3000:3000 -n grafana
```

---

# Step 19: Access Grafana Dashboard

Open browser:

```text
http://localhost:3000
```

Expected screen:

```text
Grafana Login Page
```

---

# Step 20: Login to Grafana

Default credentials:

```text
Username: admin

Password: admin
```

Grafana will ask to change password.

---

# Step 21: Verify Grafana Dashboard

Expected:

```text
Grafana Home Dashboard
```

---

# Step 22: Verify Deployment

```bash
kubectl get deployments -n grafana
```

Expected output:

```text
NAME      READY
grafana   1/1
```

---

# Step 23: Verify Persistent Storage

Delete Pod:

```bash
kubectl delete pod <grafana-pod-name> -n grafana
```

Verify new pod:

```bash
kubectl get pods -n grafana
```

Grafana data should persist.

---

# Step 24: Verify Persistent Volume

```bash
kubectl get pv
```

Expected output:

```text
NAME          CAPACITY
grafana-pv    2Gi
```

---

# Step 25: Verify Persistent Volume Claim

```bash
kubectl get pvc -n grafana
```

Expected output:

```text
NAME           STATUS
grafana-pvc    Bound
```

---

# Step 26: Delete Grafana Service

```bash
kubectl delete svc grafana-service -n grafana
```

---

# Step 27: Delete Deployment

```bash
kubectl delete deployment grafana -n grafana
```

---

# Step 28: Delete PVC

```bash
kubectl delete pvc grafana-pvc -n grafana
```

---

# Step 29: Delete PV

```bash
kubectl delete pv grafana-pv
```

---

# Step 30: Delete Namespace

```bash
kubectl delete namespace grafana
```

---

# Step 31: Delete Kind Cluster

```bash
kind delete cluster --name grafana-cluster
```

Expected output:

```text
Deleting cluster "grafana-cluster"
```

---

# Manifest Files

# grafana-pv.yaml

```yaml
apiVersion: v1
kind: PersistentVolume

metadata:
  name: grafana-pv

spec:
  capacity:
    storage: 2Gi

  accessModes:
    - ReadWriteOnce

  hostPath:
    path: /data/grafana-volume
```

---

# grafana-pvc.yaml

```yaml
apiVersion: v1
kind: PersistentVolumeClaim

metadata:
  name: grafana-pvc
  namespace: grafana

spec:
  accessModes:
    - ReadWriteOnce

  resources:
    requests:
      storage: 2Gi
```

---

# grafana-deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: grafana
  namespace: grafana

spec:
  replicas: 1

  selector:
    matchLabels:
      app: grafana

  template:
    metadata:
      labels:
        app: grafana

    spec:
      containers:

        - name: grafana

          image: grafana/grafana:latest

          ports:
            - containerPort: 3000

          volumeMounts:
            - name: grafana-storage
              mountPath: /var/lib/grafana

      volumes:
        - name: grafana-storage

          persistentVolumeClaim:
            claimName: grafana-pvc
```

---

# grafana-service.yaml

```yaml
apiVersion: v1
kind: Service

metadata:
  name: grafana-service
  namespace: grafana

spec:
  type: NodePort

  selector:
    app: grafana

  ports:
    - port: 3000
      targetPort: 3000
      nodePort: 32001
```

---

# Expected Outcome

- Kind Kubernetes cluster created successfully
- Grafana deployed successfully
- Persistent storage configured
- Grafana dashboard accessible
- Monitoring platform operational
- Kubernetes observability environment configured

---

# Real Industry Use Cases

| Use Case | Description |
|---|---|
| Kubernetes Monitoring | Cluster dashboards |
| Infrastructure Monitoring | CPU, memory metrics |
| AI/ML Platforms | ML observability |
| DevOps | Real-time monitoring |
| Cloud Platforms | Application metrics |

---

# Skills Covered

- Kubernetes Deployments
- Grafana on Kubernetes
- Persistent Volumes
- Persistent Volume Claims
- Kubernetes Services
- Monitoring
- Observability
- DevOps

---

# Real Enterprise Workflow

```text
Applications
      |
      v
Metrics Collection
      |
      v
Grafana Dashboards
      |
      v
Monitoring & Alerts
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

kind create cluster --name grafana-cluster

kubectl cluster-info

kubectl get nodes

kubectl create namespace grafana

nano grafana-pv.yaml

kubectl apply -f grafana-pv.yaml

nano grafana-pvc.yaml

kubectl apply -f grafana-pvc.yaml

nano grafana-deployment.yaml

kubectl apply -f grafana-deployment.yaml

kubectl get pods -n grafana

nano grafana-service.yaml

kubectl apply -f grafana-service.yaml

kubectl get svc -n grafana

kubectl port-forward svc/grafana-service \
3000:3000 -n grafana

kubectl get deployments -n grafana

kubectl get pv

kubectl get pvc -n grafana

kubectl delete svc grafana-service -n grafana

kubectl delete deployment grafana -n grafana

kubectl delete pvc grafana-pvc -n grafana

kubectl delete pv grafana-pv

kubectl delete namespace grafana

kind delete cluster --name grafana-cluster
```