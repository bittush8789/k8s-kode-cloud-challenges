# Kubectl Port Forward — Complete Real-Time Guide

# What is kubectl port-forward?

`kubectl port-forward` allows you to access a Kubernetes Pod or Service locally from your machine without exposing the application externally.

It creates a secure tunnel between:

```text
Your Local Machine  <----->  Kubernetes Pod/Service
```

---

# Why Port Forward is Important

In real enterprise environments:

- Applications run inside private Kubernetes clusters
- Internal services are not publicly exposed
- Developers need temporary access
- Debugging requires secure connectivity

Instead of creating:

- LoadBalancers
- Ingress
- NodePorts

DevOps engineers use:

```bash
kubectl port-forward
```

for quick and secure access.

---

# Real Enterprise Scenario

xFusionCorp Industries manages:

- AI/ML platforms
- Internal APIs
- Banking applications
- Monitoring tools
- Microservices

The organization does NOT expose:

- Databases
- Internal dashboards
- Admin panels
- Development tools

publicly for security reasons.

Developers use:

```bash
kubectl port-forward
```

to securely access services locally.

---

# Real Production Use Cases

| Use Case | Example |
|---|---|
| Access Internal APIs | Backend debugging |
| Access Databases | MySQL/Postgres |
| Access Grafana | Monitoring |
| Access MLflow | MLOps |
| Access ArgoCD | GitOps |
| Access Kubernetes Dashboard | Cluster management |

---

# Port Forward Architecture

```text
            Local Machine
                   |
                   |
             localhost:8080
                   |
                   v

        kubectl port-forward
                   |
                   v

         Kubernetes API Server
                   |
                   v

           Kubernetes Pod
```

---

# How Port Forward Works

```text
Local Port Opened
        |
        v
Traffic Sent to kubectl
        |
        v
Kubernetes API Server
        |
        v
Target Pod/Service
```

---

# Important Concept

# Port Forward is Temporary

When terminal closes:

```text
Connection closes automatically
```

---

# Port Forward vs NodePort vs LoadBalancer

| Feature | Port Forward | NodePort | LoadBalancer |
|---|---|---|---|
| Temporary | Yes | No | No |
| External Access | No | Yes | Yes |
| Secure | High | Medium | Medium |
| Production Usage | Debugging | Small apps | Production |
| Internet Exposure | No | Possible | Yes |

---

# Lab Setup

We will:

1. Create Kubernetes cluster
2. Deploy NGINX
3. Access app using port-forward
4. Access Pod directly
5. Access Service directly
6. Debug applications

---

# Step 1 — Create NGINX Deployment

```bash
kubectl create deployment nginx-app \
--image=nginx
```

---

# Step 2 — Verify Pods

```bash
kubectl get pods
```

Expected:

```text
Running
```

---

# Step 3 — Expose Deployment

```bash
kubectl expose deployment nginx-app \
--port=80 \
--target-port=80 \
--type=ClusterIP
```

---

# Step 4 — Verify Service

```bash
kubectl get svc
```

Expected:

```text
nginx-app
```

---

# Port Forward to Pod

# Step 1 — Get Pod Name

```bash
kubectl get pods
```

Example:

```text
nginx-app-7d8f9c7f5d-abcd
```

---

# Step 2 — Port Forward Pod

```bash
kubectl port-forward pod/nginx-app-7d8f9c7f5d-abcd \
8080:80
```

---

# What This Means

| Local Port | Container Port |
|---|---|
| 8080 | 80 |

---

# Step 3 — Access Application

Open browser:

```text
http://localhost:8080
```

Expected:

```text
Welcome to nginx
```

---

# Port Forward to Service

# Why Service Port Forwarding is Better

Pods may restart/change names.

Services are stable.

---

# Step 1 — Port Forward Service

```bash
kubectl port-forward svc/nginx-app \
8080:80
```

---

# Step 2 — Access Application

```text
http://localhost:8080
```

---

# Port Forward to Different Local Port

Example:

```bash
kubectl port-forward svc/nginx-app \
9090:80
```

Access:

```text
http://localhost:9090
```

---

# Multiple Port Forwarding

Example:

```bash
kubectl port-forward svc/myapp \
8080:80 \
8443:443
```

---

# Real-Time Scenario 1 — Access Internal MySQL Database

# Enterprise Scenario

Production MySQL is NOT publicly exposed.

DevOps engineer needs temporary access.

---

# Deploy MySQL

```bash
kubectl create deployment mysql-db \
--image=mysql:8
```

---

# Expose MySQL

```bash
kubectl expose deployment mysql-db \
--port=3306 \
--target-port=3306
```

---

# Port Forward MySQL

```bash
kubectl port-forward svc/mysql-db \
3306:3306
```

---

# Connect Locally

```bash
mysql -h 127.0.0.1 -P 3306 -u root -p
```

---

# Real-Time Scenario 2 — Access Grafana Dashboard

# Enterprise Scenario

Monitoring dashboards are private.

---

# Port Forward Grafana

```bash
kubectl port-forward svc/grafana \
3000:3000
```

Access:

```text
http://localhost:3000
```

---

# Real-Time Scenario 3 — Access MLflow

# Enterprise Scenario

MLflow tracking server should remain internal.

---

# Port Forward MLflow

```bash
kubectl port-forward svc/mlflow \
5000:5000
```

Access:

```text
http://localhost:5000
```

---

# Real-Time Scenario 4 — Access ArgoCD

# Enterprise Scenario

GitOps dashboard should not be internet-facing.

---

# Port Forward ArgoCD

```bash
kubectl port-forward svc/argocd-server \
8080:443
```

---

# Namespace-Based Port Forwarding

# Example

```bash
kubectl port-forward svc/nginx-app \
8080:80 \
-n production
```

---

# Port Forward Background Mode

Linux:

```bash
kubectl port-forward svc/nginx-app \
8080:80 > /dev/null 2>&1 &
```

---

# Check Running Port Forward Processes

```bash
ps -ef | grep port-forward
```

---

# Kill Port Forward Process

```bash
kill -9 <PID>
```

---

# Common Port Forward Errors

| Error | Reason |
|---|---|
| Address already in use | Port occupied locally |
| Pod not found | Wrong Pod name |
| Connection refused | App not listening |
| Service not found | Wrong service name |
| Broken pipe | Network interruption |

---

# Error 1 — Address Already in Use

Example:

```text
unable to listen on port 8080
```

---

# Fix

Use another port:

```bash
kubectl port-forward svc/nginx-app \
9090:80
```

---

# Error 2 — Pod Not Running

Check:

```bash
kubectl get pods
```

---

# Error 3 — Application Not Listening

Check container ports:

```bash
kubectl describe pod <pod-name>
```

---

# Important Troubleshooting Commands

# Check Pods

```bash
kubectl get pods
```

---

# Check Services

```bash
kubectl get svc
```

---

# Describe Service

```bash
kubectl describe svc <service-name>
```

---

# Check Endpoints

```bash
kubectl get endpoints
```

---

# Describe Pod

```bash
kubectl describe pod <pod-name>
```

---

# Check Logs

```bash
kubectl logs <pod-name>
```

---

# Enterprise Security Benefits

| Benefit | Description |
|---|---|
| No Public Exposure | Internal access only |
| Temporary Access | Session-based |
| Secure Tunnel | Via API server |
| Fast Debugging | No ingress required |

---

# Enterprise Best Practices

| Best Practice | Purpose |
|---|---|
| Use Service Forwarding | Stable access |
| Avoid Public Exposure | Security |
| Use Namespaces | Isolation |
| Restrict RBAC | Secure access |
| Monitor Access | Audit security |

---

# Port Forward vs SSH Tunnel

| Feature | kubectl port-forward | SSH Tunnel |
|---|---|---|
| Kubernetes Native | Yes | No |
| Uses API Server | Yes | No |
| Requires SSH Access | No | Yes |
| Easier for DevOps | Yes | Medium |

---

# Real Enterprise Workflow

```text
Developer Needs Access
          |
          v
kubectl port-forward
          |
          v
Secure Tunnel Created
          |
          v
Internal Kubernetes Service
          |
          v
Application Accessed Locally
```

---

# Production Architecture

```text
                 Developer Laptop
                         |
                         v

              kubectl port-forward
                         |
                         v

              Kubernetes API Server
                         |
        +----------------+----------------+
        |                                 |
        v                                 v

   Internal Service                 Internal Pod
        |                                 |
        v                                 v

   Grafana/MySQL                  Backend API
```

---

# Advanced Port Forward Example

# Forward PostgreSQL

```bash
kubectl port-forward svc/postgres \
5432:5432
```

---

# Forward Redis

```bash
kubectl port-forward svc/redis \
6379:6379
```

---

# Forward Elasticsearch

```bash
kubectl port-forward svc/elasticsearch \
9200:9200
```

---

# Forward Kibana

```bash
kubectl port-forward svc/kibana \
5601:5601
```

---

# Expected Outcome

- Understood kubectl port-forward deeply
- Accessed Pods locally
- Accessed Services locally
- Learned enterprise debugging workflows
- Learned secure Kubernetes access
- Learned real production use cases
- Built Kubernetes troubleshooting skills

---

# Skills Covered

- kubectl port-forward
- Kubernetes Networking
- Kubernetes Debugging
- Kubernetes Services
- Kubernetes Pods
- Kubernetes Troubleshooting
- DevOps
- Cloud-Native Infrastructure
- Secure Kubernetes Access
- Production Kubernetes