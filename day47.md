# HELM Explained — Kubernetes Package Manager

# What is HELM?

HELM is the package manager for Kubernetes.

Think of HELM as:

```text
APT for Ubuntu
YUM for CentOS
NPM for Node.js
PIP for Python
```

but for:

```text
Kubernetes Applications
```

---

# Why HELM Was Created

Before HELM:

Deploying applications on Kubernetes required:

- Multiple YAML files
- Manual configuration
- Repetitive deployments
- Complex updates
- Difficult rollback handling

Example:

Deploying Prometheus manually may require:

```text
20+ YAML files
```

including:

- Deployments
- Services
- ConfigMaps
- Secrets
- PVCs
- Ingress
- RBAC

This became difficult to manage.

---

# HELM Solves This Problem

HELM packages all Kubernetes resources into:

```text
HELM Charts
```

You can install applications using:

```bash
helm install
```

just like:

```bash
apt install nginx
```

---

# Real Enterprise Scenario

xFusionCorp Industries manages:

- Kubernetes clusters
- AI/ML platforms
- Monitoring stacks
- Banking applications
- CI/CD pipelines

The DevOps team wants:

- Faster deployments
- Standardized configurations
- Easy rollback
- Environment management
- Reusable templates

They use:

```text
HELM
```

for enterprise Kubernetes deployments.

---

# Real Production Use Cases

| Application | HELM Usage |
|---|---|
| Prometheus | Monitoring |
| Grafana | Dashboards |
| Jenkins | CI/CD |
| ArgoCD | GitOps |
| Elasticsearch | Logging |
| Kafka | Streaming |
| MLflow | MLOps |
| Airflow | Data pipelines |

---

# HELM Architecture

```text
                    +----------------------+
                    |     HELM CLI         |
                    +----------+-----------+
                               |
                               v
                    +----------------------+
                    |     HELM Chart       |
                    +----------+-----------+
                               |
                               v
                    +----------------------+
                    | Kubernetes API Server|
                    +----------+-----------+
                               |
                               v
                    +----------------------+
                    | Kubernetes Resources |
                    +----------------------+
```

---

# Important HELM Concepts

| Concept | Meaning |
|---|---|
| Chart | Kubernetes application package |
| Release | Running instance of chart |
| Repository | Collection of charts |
| Values.yaml | Configuration file |
| Template | Dynamic Kubernetes YAML |

---

# What is a HELM Chart?

A HELM chart is a packaged Kubernetes application.

It contains:

```text
Deployments
Services
Ingress
Secrets
ConfigMaps
PVCs
```

all bundled together.

---

# HELM Chart Structure

```text
mychart/

├── Chart.yaml
├── values.yaml
├── charts/
├── templates/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   └── configmap.yaml
```

---

# What is Release?

When you install a chart:

```bash
helm install myapp nginx-chart
```

HELM creates a:

```text
Release
```

Example:

```text
Release Name: myapp
```

---

# HELM Workflow

```text
Developer
    |
    v
HELM Chart
    |
    v
helm install
    |
    v
Kubernetes Resources Created
    |
    v
Pods Running
```

---

# Install HELM

# Step 1 — Download HELM

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

---

# Step 2 — Verify Installation

```bash
helm version
```

Expected:

```text
version.BuildInfo
```

---

# Add HELM Repository

# What is HELM Repository?

A repository stores HELM charts.

Examples:

| Repository | Purpose |
|---|---|
| Bitnami | Popular apps |
| Prometheus Community | Monitoring |
| Elastic | Logging |
| Grafana | Dashboards |

---

# Step 1 — Add Bitnami Repository

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
```

---

# Step 2 — Update Repository

```bash
helm repo update
```

---

# Step 3 — Search Charts

```bash
helm search repo nginx
```

---

# Install NGINX Using HELM

# Step 1 — Install Chart

```bash
helm install my-nginx bitnami/nginx
```

---

# What Happens Internally?

HELM automatically creates:

- Deployment
- Service
- ConfigMaps
- Secrets
- Pods

---

# Step 2 — Verify Release

```bash
helm list
```

Expected:

```text
my-nginx
```

---

# Step 3 — Verify Kubernetes Resources

```bash
kubectl get all
```

Expected:

```text
Pods
Services
Deployments
ReplicaSets
```

---

# Step 4 — Access Application

```bash
kubectl port-forward svc/my-nginx 8080:80
```

Open browser:

```text
http://localhost:8080
```

---

# HELM vs kubectl

| Feature | kubectl | HELM |
|---|---|---|
| Manual YAML | Yes | No |
| Package Manager | No | Yes |
| Rollbacks | Limited | Easy |
| Reusability | Low | High |
| Templates | No | Yes |
| Environment Management | Hard | Easy |

---

# HELM Values.yaml

# What is values.yaml?

This file stores configuration variables.

Example:

```yaml
replicaCount: 3

image:
  repository: nginx
  tag: latest

service:
  type: NodePort
```

---

# Why Values.yaml is Powerful

Instead of editing YAML repeatedly:

```yaml
replicas: 3
```

You simply modify:

```yaml
replicaCount: 3
```

---

# Override HELM Values

Example:

```bash
helm install my-nginx bitnami/nginx \
--set replicaCount=4
```

---

# Use Custom Values File

```bash
helm install my-nginx bitnami/nginx \
-f values.yaml
```

---

# Upgrade Applications

# Real Enterprise Scenario

The DevOps team wants:

- New application version
- More replicas
- New environment variables

without downtime.

---

# HELM Upgrade

```bash
helm upgrade my-nginx bitnami/nginx \
--set replicaCount=5
```

---

# Verify Upgrade

```bash
kubectl get deployments
```

Expected:

```text
READY 5/5
```

---

# HELM Rollback

# Why Rollback Matters

Production deployment failed.

Need immediate recovery.

---

# View Revision History

```bash
helm history my-nginx
```

---

# Rollback to Previous Version

```bash
helm rollback my-nginx 1
```

---

# Enterprise Benefit

Instant recovery during production incidents.

---

# Uninstall HELM Release

```bash
helm uninstall my-nginx
```

---

# Create Your Own HELM Chart

# Step 1 — Create Chart

```bash
helm create mychart
```

---

# Generated Structure

```text
mychart/
```

with templates automatically created.

---

# Step 2 — Explore Templates

```bash
cd mychart

ls
```

---

# Step 3 — Install Local Chart

```bash
helm install myapp ./mychart
```

---

# HELM Templating

# Example Deployment Template

```yaml
replicas: {{ .Values.replicaCount }}
```

HELM dynamically replaces:

```text
{{ .Values.replicaCount }}
```

with actual values.

---

# Example Dynamic Image

```yaml
image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
```

---

# Why Templates Matter

One chart can support:

- Dev environment
- QA environment
- Production environment

---

# Enterprise Environment Management

| Environment | Replica Count |
|---|---|
| Dev | 1 |
| QA | 2 |
| Production | 5 |

---

# Example Production Deployment

```bash
helm install prod-app ./mychart \
-f production-values.yaml
```

---

# Real Enterprise CI/CD Workflow

```text
Developer Push
        |
        v
GitHub Actions
        |
        v
Docker Image Build
        |
        v
Push to Registry
        |
        v
HELM Upgrade
        |
        v
Kubernetes Deployment
```

---

# HELM in GitOps

Modern companies use:

| Tool | Purpose |
|---|---|
| ArgoCD | GitOps |
| FluxCD | GitOps |
| HELM | Package Management |

---

# GitOps + HELM Workflow

```text
Git Repository
       |
       v
HELM Charts
       |
       v
ArgoCD Watches Git
       |
       v
Automatic Kubernetes Sync
```

---

# HELM Best Practices

| Best Practice | Purpose |
|---|---|
| Use values.yaml | Reusability |
| Use templates | Dynamic configs |
| Store charts in Git | Version control |
| Use chart versioning | Safe upgrades |
| Use rollback | Disaster recovery |

---

# HELM Chart Versioning

Example:

```yaml
version: 1.0.0
```

---

# Common HELM Commands

# Add Repo

```bash
helm repo add
```

---

# Update Repo

```bash
helm repo update
```

---

# Search Charts

```bash
helm search repo
```

---

# Install Chart

```bash
helm install
```

---

# Upgrade Release

```bash
helm upgrade
```

---

# Rollback Release

```bash
helm rollback
```

---

# Uninstall Release

```bash
helm uninstall
```

---

# List Releases

```bash
helm list
```

---

# View Values

```bash
helm get values <release-name>
```

---

# View Manifest

```bash
helm get manifest <release-name>
```

---

# Real Enterprise Monitoring Stack

Companies deploy:

```text
Prometheus
Grafana
Alertmanager
Loki
```

using:

```bash
helm install
```

instead of manually managing hundreds of YAMLs.

---

# Real Production Architecture

```text
                    DevOps Engineer
                             |
                             v

                      HELM CLI
                             |
                             v

                     HELM Charts
                             |
                             v

                  Kubernetes Cluster
                             |
       +---------------------+---------------------+
       |                     |                     |
       v                     v                     v

   Deployments           Services             ConfigMaps
```

---

# HELM Advantages

| Advantage | Benefit |
|---|---|
| Reusable Charts | Faster deployments |
| Easy Rollback | Production safety |
| Templates | Dynamic configs |
| Versioning | Safe releases |
| Automation | CI/CD integration |

---

# HELM Disadvantages

| Disadvantage | Description |
|---|---|
| Complexity | Advanced templates |
| Debugging | Harder than raw YAML |
| Large Charts | Complex enterprise apps |

---

# HELM vs Kustomize

| Feature | HELM | Kustomize |
|---|---|---|
| Templates | Yes | No |
| Package Manager | Yes | No |
| Overlay Support | Limited | Strong |
| Complexity | Higher | Lower |

---

# Real Enterprise Tools Using HELM

| Tool | Deployment Method |
|---|---|
| Prometheus | HELM |
| Grafana | HELM |
| ArgoCD | HELM |
| Jenkins | HELM |
| MLflow | HELM |
| Airflow | HELM |

---

# Enterprise HELM Deployment Pipeline

```text
GitHub
   |
   v
CI/CD Pipeline
   |
   v
Docker Build
   |
   v
Container Registry
   |
   v
HELM Upgrade
   |
   v
Kubernetes Cluster
```

---

# Expected Outcome

- Understood HELM deeply
- Learned Kubernetes package management
- Installed applications using HELM
- Learned HELM charts
- Learned releases
- Learned upgrades and rollback
- Learned enterprise Kubernetes deployment workflow
- Built production-grade Kubernetes deployment skills

---

# Skills Covered

- HELM
- Kubernetes Package Management
- HELM Charts
- HELM Templates
- HELM Releases
- Kubernetes Deployments
- Kubernetes Automation
- GitOps
- CI/CD
- DevOps
- Cloud-Native Infrastructure
- Production Kubernetes