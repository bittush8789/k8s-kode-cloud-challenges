# Deploy Jenkins on Kubernetes

# Normal Problem Statement

The DevOps team at xFusionCorp Industries wants to deploy Jenkins on a Kubernetes cluster for building CI/CD pipelines and automating application deployments.

Your task is to create a Kubernetes cluster using Kind, deploy Jenkins using Kubernetes manifests, expose the Jenkins UI, and verify the deployment on an Ubuntu-based system.

---

# Scenario-Based Problem Statement

xFusionCorp Industries is modernizing its DevOps infrastructure.

The organization manages:

- CI/CD pipelines
- Kubernetes deployments
- Docker builds
- Cloud-native applications
- AI/ML platforms

The engineering team currently faces several issues:

- Manual deployments
- No centralized CI/CD system
- Poor build automation
- Unstable deployment workflows
- Lack of scalable Jenkins infrastructure

To solve these problems, the organization wants to deploy Jenkins on Kubernetes.

As a DevOps Engineer, your responsibilities are to:

1. Create a Kubernetes cluster using Kind.
2. Deploy Jenkins on Kubernetes.
3. Configure persistent storage.
4. Expose Jenkins using Kubernetes Service.
5. Access Jenkins UI.
6. Configure Jenkins admin setup.
7. Build scalable CI/CD infrastructure.

---

# Objectives

1. Install Kubernetes tools.
2. Create a Kind Kubernetes cluster.
3. Deploy Jenkins application.
4. Configure persistent volume.
5. Expose Jenkins UI.
6. Access Jenkins dashboard.
7. Verify Jenkins deployment.

---

# Architecture Overview

```text
                    +----------------+
                    |   Developers   |
                    +--------+-------+
                             |
                             v
                    +----------------+
                    | Jenkins Service|
                    +--------+-------+
                             |
                             v
                    +----------------+
                    | Jenkins Pod    |
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
kind create cluster --name jenkins-cluster
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
jenkins-cluster-control-plane Ready   control-plane
```

---

# Step 7: Create Jenkins Namespace

```bash
kubectl create namespace jenkins
```

Verify:

```bash
kubectl get namespaces
```

---

# Step 8: Create Jenkins Persistent Volume Manifest

```bash
nano jenkins-pv.yaml
```

Add the following YAML:

```yaml
apiVersion: v1
kind: PersistentVolume

metadata:
  name: jenkins-pv

spec:
  capacity:
    storage: 2Gi

  accessModes:
    - ReadWriteOnce

  hostPath:
    path: /data/jenkins-volume
```

---

# Step 9: Apply Persistent Volume

```bash
kubectl apply -f jenkins-pv.yaml
```

Expected output:

```text
persistentvolume/jenkins-pv created
```

---

# Step 10: Create Persistent Volume Claim

```bash
nano jenkins-pvc.yaml
```

Add the following YAML:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim

metadata:
  name: jenkins-pvc
  namespace: jenkins

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
kubectl apply -f jenkins-pvc.yaml
```

Expected output:

```text
persistentvolumeclaim/jenkins-pvc created
```

---

# Step 12: Create Jenkins Deployment Manifest

```bash
nano jenkins-deployment.yaml
```

Add the following YAML:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: jenkins
  namespace: jenkins

spec:
  replicas: 1

  selector:
    matchLabels:
      app: jenkins

  template:
    metadata:
      labels:
        app: jenkins

    spec:
      containers:

        - name: jenkins

          image: jenkins/jenkins:lts

          ports:
            - containerPort: 8080

          volumeMounts:
            - name: jenkins-home
              mountPath: /var/jenkins_home

      volumes:
        - name: jenkins-home

          persistentVolumeClaim:
            claimName: jenkins-pvc
```

---

# Step 13: Apply Jenkins Deployment

```bash
kubectl apply -f jenkins-deployment.yaml
```

Expected output:

```text
deployment.apps/jenkins created
```

---

# Step 14: Verify Jenkins Pod

```bash
kubectl get pods -n jenkins
```

Expected output:

```text
NAME                       READY   STATUS
jenkins-xxxxxx             1/1     Running
```

---

# Step 15: Create Jenkins Service Manifest

```bash
nano jenkins-service.yaml
```

Add the following YAML:

```yaml
apiVersion: v1
kind: Service

metadata:
  name: jenkins-service
  namespace: jenkins

spec:
  type: NodePort

  selector:
    app: jenkins

  ports:
    - port: 8080
      targetPort: 8080
      nodePort: 32000
```

---

# Step 16: Apply Jenkins Service

```bash
kubectl apply -f jenkins-service.yaml
```

Expected output:

```text
service/jenkins-service created
```

---

# Step 17: Verify Service

```bash
kubectl get svc -n jenkins
```

Expected output:

```text
NAME              TYPE       PORT(S)
jenkins-service   NodePort   8080:32000/TCP
```

---

# Step 18: Port Forward Jenkins Service

```bash
kubectl port-forward svc/jenkins-service \
8080:8080 -n jenkins
```

---

# Step 19: Access Jenkins UI

Open browser:

```text
http://localhost:8080
```

Expected screen:

```text
Unlock Jenkins
```

---

# Step 20: Get Jenkins Initial Admin Password

Open new terminal:

```bash
kubectl get pods -n jenkins
```

Copy pod name.

Example:

```bash
kubectl logs <jenkins-pod-name> -n jenkins
```

Example:

```bash
kubectl logs jenkins-7bdf9d7f6f-abcde -n jenkins
```

Find:

```text
Please use the following password to proceed to installation:
```

Copy password.

---

# Step 21: Unlock Jenkins

Paste password into Jenkins UI.

Click:

```text
Continue
```

---

# Step 22: Install Suggested Plugins

Select:

```text
Install Suggested Plugins
```

Wait for installation.

---

# Step 23: Create Admin User

Configure:

- Username
- Password
- Email

Click:

```text
Save and Continue
```

---

# Step 24: Verify Jenkins Dashboard

Expected:

```text
Welcome to Jenkins Dashboard
```

---

# Step 25: Verify Persistent Storage

Delete Pod:

```bash
kubectl delete pod <pod-name> -n jenkins
```

Verify new pod created:

```bash
kubectl get pods -n jenkins
```

Persistent Jenkins data should remain intact.

---

# Step 26: Verify Deployment

```bash
kubectl get deployments -n jenkins
```

Expected output:

```text
NAME      READY
jenkins   1/1
```

---

# Step 27: Delete Jenkins Service

```bash
kubectl delete svc jenkins-service -n jenkins
```

---

# Step 28: Delete Deployment

```bash
kubectl delete deployment jenkins -n jenkins
```

---

# Step 29: Delete PVC

```bash
kubectl delete pvc jenkins-pvc -n jenkins
```

---

# Step 30: Delete PV

```bash
kubectl delete pv jenkins-pv
```

---

# Step 31: Delete Namespace

```bash
kubectl delete namespace jenkins
```

---

# Step 32: Delete Kind Cluster

```bash
kind delete cluster --name jenkins-cluster
```

Expected output:

```text
Deleting cluster "jenkins-cluster"
```

---

# Manifest Files

# jenkins-pv.yaml

```yaml
apiVersion: v1
kind: PersistentVolume

metadata:
  name: jenkins-pv

spec:
  capacity:
    storage: 2Gi

  accessModes:
    - ReadWriteOnce

  hostPath:
    path: /data/jenkins-volume
```

---

# jenkins-pvc.yaml

```yaml
apiVersion: v1
kind: PersistentVolumeClaim

metadata:
  name: jenkins-pvc
  namespace: jenkins

spec:
  accessModes:
    - ReadWriteOnce

  resources:
    requests:
      storage: 2Gi
```

---

# jenkins-deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: jenkins
  namespace: jenkins

spec:
  replicas: 1

  selector:
    matchLabels:
      app: jenkins

  template:
    metadata:
      labels:
        app: jenkins

    spec:
      containers:

        - name: jenkins

          image: jenkins/jenkins:lts

          ports:
            - containerPort: 8080

          volumeMounts:
            - name: jenkins-home
              mountPath: /var/jenkins_home

      volumes:
        - name: jenkins-home

          persistentVolumeClaim:
            claimName: jenkins-pvc
```

---

# jenkins-service.yaml

```yaml
apiVersion: v1
kind: Service

metadata:
  name: jenkins-service
  namespace: jenkins

spec:
  type: NodePort

  selector:
    app: jenkins

  ports:
    - port: 8080
      targetPort: 8080
      nodePort: 32000
```

---

# Expected Outcome

- Kind Kubernetes cluster created successfully
- Jenkins deployed successfully
- Persistent storage configured
- Jenkins UI accessible
- Jenkins admin configured
- Kubernetes-based CI/CD platform operational

---

# Real Industry Use Cases

| Use Case | Description |
|---|---|
| CI/CD | Automated pipelines |
| DevOps | Build automation |
| Kubernetes | Cluster deployments |
| AI/ML | ML pipeline automation |
| Enterprise Apps | Continuous delivery |

---

# Skills Covered

- Kubernetes Deployments
- Jenkins on Kubernetes
- Persistent Volumes
- Persistent Volume Claims
- Kubernetes Services
- DevOps Automation
- CI/CD
- Cloud-Native Deployment

---

# Real Enterprise Workflow

```text
Developer Push
       |
       v
Jenkins Pipeline
       |
       v
Build & Test
       |
       v
Docker Build
       |
       v
Kubernetes Deployment
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

kind create cluster --name jenkins-cluster

kubectl cluster-info

kubectl get nodes

kubectl create namespace jenkins

nano jenkins-pv.yaml

kubectl apply -f jenkins-pv.yaml

nano jenkins-pvc.yaml

kubectl apply -f jenkins-pvc.yaml

nano jenkins-deployment.yaml

kubectl apply -f jenkins-deployment.yaml

kubectl get pods -n jenkins

nano jenkins-service.yaml

kubectl apply -f jenkins-service.yaml

kubectl get svc -n jenkins

kubectl port-forward svc/jenkins-service \
8080:8080 -n jenkins

kubectl logs <jenkins-pod-name> -n jenkins

kubectl delete pod <pod-name> -n jenkins

kubectl get deployments -n jenkins

kubectl delete svc jenkins-service -n jenkins

kubectl delete deployment jenkins -n jenkins

kubectl delete pvc jenkins-pvc -n jenkins

kubectl delete pv jenkins-pv

kubectl delete namespace jenkins

kind delete cluster --name jenkins-cluster
```