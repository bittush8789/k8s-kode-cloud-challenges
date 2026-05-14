# Kubernetes Node Selector, Node Affinity, Taints and Tolerations

# What is Node Scheduling in Kubernetes?

Kubernetes automatically schedules Pods on worker nodes.

But in real enterprise environments, not every workload should run on every node.

Examples:

- AI workloads need GPU nodes
- Banking apps need secure nodes
- Monitoring tools need dedicated nodes
- Databases need high-memory nodes

Kubernetes provides advanced scheduling mechanisms:

| Feature | Purpose |
|---|---|
| Node Selector | Simple node targeting |
| Node Affinity | Advanced node scheduling |
| Taints | Prevent workloads on nodes |
| Tolerations | Allow workloads on tainted nodes |

---

# Real Enterprise Scenario

xFusionCorp Industries manages:

- AI/ML platforms
- Banking applications
- Kubernetes monitoring stack
- Enterprise APIs
- GPU-based workloads

The infrastructure team created different node pools:

| Node Type | Purpose |
|---|---|
| GPU Nodes | AI/ML workloads |
| Secure Nodes | Banking apps |
| Monitoring Nodes | Grafana/Prometheus |
| General Nodes | Regular applications |

The DevOps team needs to ensure:

- AI apps only run on GPU nodes
- Monitoring apps run on monitoring nodes
- Banking apps avoid untrusted nodes
- Critical workloads get isolated infrastructure

---

# Kubernetes Scheduling Architecture

```text
                   +----------------------+
                   | Kubernetes Scheduler |
                   +----------+-----------+
                              |
         +--------------------+--------------------+
         |                    |                    |
         v                    v                    v

 +---------------+   +---------------+   +---------------+
 | Worker Node-1 |   | Worker Node-2 |   | Worker Node-3 |
 | GPU Node      |   | Monitoring    |   | General Node  |
 +---------------+   +---------------+   +---------------+

         ^                    ^                    ^
         |                    |                    |

   AI Workloads        Monitoring Apps       General Apps
```

---

# Kubernetes Scheduling Flow

```text
Pod Created
      |
      v
Scheduler Checks Rules
      |
      +--------------------+
      |                    |
      v                    v

NodeSelector        Affinity Rules
      |                    |
      v                    v

Taints Check ---> Tolerations Check
      |
      v
Best Node Selected
      |
      v
Pod Scheduled
```

---

# Part 1 — Node Selector

# What is Node Selector?

Node Selector is the simplest scheduling method.

It schedules Pods only on nodes with specific labels.

---

# Real-Time Scenario

The AI Engineering team wants:

- TensorFlow workloads
- GPU workloads
- ML inference APIs

to run only on:

```text
gpu-node
```

---

# Step 1: Check Nodes

```bash
kubectl get nodes
```

Expected:

```text
worker-node
```

---

# Step 2: Add Label to Node

```bash
kubectl label nodes <node-name> \
hardware=gpu
```

Example:

```bash
kubectl label nodes worker-node \
hardware=gpu
```

---

# Step 3: Verify Labels

```bash
kubectl get nodes --show-labels
```

Expected:

```text
hardware=gpu
```

---

# Step 4: Create Node Selector Deployment

```bash
nano node-selector-app.yaml
```

Add:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: gpu-app

spec:
  replicas: 2

  selector:
    matchLabels:
      app: gpu-app

  template:
    metadata:
      labels:
        app: gpu-app

    spec:

      nodeSelector:
        hardware: gpu

      containers:

        - name: gpu-container

          image: nginx

          ports:
            - containerPort: 80
```

---

# Step 5: Deploy Application

```bash
kubectl apply -f node-selector-app.yaml
```

---

# Step 6: Verify Pod Scheduling

```bash
kubectl get pods -o wide
```

Expected:

```text
Pods running on GPU node
```

---

# Part 2 — Node Affinity

# What is Node Affinity?

Node Affinity is an advanced version of Node Selector.

It supports:

- Multiple conditions
- Preferred scheduling
- Required scheduling
- Complex rules

---

# Types of Node Affinity

| Type | Meaning |
|---|---|
| requiredDuringSchedulingIgnoredDuringExecution | Mandatory rule |
| preferredDuringSchedulingIgnoredDuringExecution | Preferred rule |

---

# Real-Time Scenario

The Monitoring team wants:

- Prometheus
- Grafana
- Loki

to preferably run on monitoring nodes.

---

# Step 1: Label Monitoring Node

```bash
kubectl label nodes <node-name> \
role=monitoring
```

---

# Step 2: Create Affinity Deployment

```bash
nano node-affinity-app.yaml
```

Add:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: monitoring-app

spec:
  replicas: 2

  selector:
    matchLabels:
      app: monitoring-app

  template:
    metadata:
      labels:
        app: monitoring-app

    spec:

      affinity:

        nodeAffinity:

          requiredDuringSchedulingIgnoredDuringExecution:

            nodeSelectorTerms:

              - matchExpressions:

                  - key: role

                    operator: In

                    values:
                      - monitoring

      containers:

        - name: monitoring-container

          image: nginx

          ports:
            - containerPort: 80
```

---

# Step 3: Deploy Application

```bash
kubectl apply -f node-affinity-app.yaml
```

---

# Step 4: Verify Pod Placement

```bash
kubectl get pods -o wide
```

Expected:

```text
Pods scheduled on monitoring node
```

---

# Node Affinity Operators

| Operator | Meaning |
|---|---|
| In | Value must match |
| NotIn | Value must not match |
| Exists | Label exists |
| DoesNotExist | Label absent |
| Gt | Greater than |
| Lt | Less than |

---

# Preferred Node Affinity Example

```yaml
preferredDuringSchedulingIgnoredDuringExecution:

  - weight: 1

    preference:

      matchExpressions:

        - key: role

          operator: In

          values:
            - monitoring
```

---

# Part 3 — Taints and Tolerations

# What are Taints?

Taints prevent Pods from being scheduled on nodes.

Think of taints as:

```text
Node Repellent
```

---

# What are Tolerations?

Tolerations allow Pods to run on tainted nodes.

Think of tolerations as:

```text
Permission to enter restricted nodes
```

---

# Real-Time Scenario

xFusionCorp Industries has dedicated secure nodes for:

- Banking applications
- Financial systems
- Compliance workloads

No normal applications should run there.

---

# Step 1: Add Taint to Node

```bash
kubectl taint nodes <node-name> \
secure=true:NoSchedule
```

Example:

```bash
kubectl taint nodes worker-node \
secure=true:NoSchedule
```

---

# Step 2: Verify Taints

```bash
kubectl describe node <node-name>
```

Expected:

```text
Taints:
secure=true:NoSchedule
```

---

# Step 3: Deploy Normal App

```bash
nano normal-app.yaml
```

Add:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: normal-app

spec:
  replicas: 1

  selector:
    matchLabels:
      app: normal-app

  template:
    metadata:
      labels:
        app: normal-app

    spec:
      containers:

        - name: nginx

          image: nginx
```

---

# Step 4: Deploy Application

```bash
kubectl apply -f normal-app.yaml
```

---

# Step 5: Verify Scheduling Failure

```bash
kubectl get pods
```

Expected:

```text
Pending
```

---

# Step 6: Describe Pod

```bash
kubectl describe pod <pod-name>
```

Expected:

```text
node(s) had taint that the pod didn't tolerate
```

---

# Root Cause

Node is protected using taint:

```text
secure=true:NoSchedule
```

---

# Step 7: Create Toleration Deployment

```bash
nano toleration-app.yaml
```

Add:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: secure-app

spec:
  replicas: 1

  selector:
    matchLabels:
      app: secure-app

  template:
    metadata:
      labels:
        app: secure-app

    spec:

      tolerations:

        - key: "secure"

          operator: "Equal"

          value: "true"

          effect: "NoSchedule"

      containers:

        - name: secure-container

          image: nginx
```

---

# Step 8: Deploy Application

```bash
kubectl apply -f toleration-app.yaml
```

---

# Step 9: Verify Pod Scheduling

```bash
kubectl get pods -o wide
```

Expected:

```text
Running on secure node
```

---

# Taint Effects

| Effect | Meaning |
|---|---|
| NoSchedule | No Pods scheduled |
| PreferNoSchedule | Avoid scheduling |
| NoExecute | Remove running Pods |

---

# Remove Taint

```bash
kubectl taint nodes <node-name> \
secure=true:NoSchedule-
```

---

# Real Enterprise Use Cases

| Use Case | Technology |
|---|---|
| GPU Isolation | AI/ML workloads |
| Dedicated Monitoring Nodes | Grafana/Prometheus |
| Secure Banking Nodes | Financial apps |
| High-Memory Nodes | Databases |
| Spot Nodes | Low-cost workloads |

---

# Example GPU Workload

```yaml
nodeSelector:
  accelerator: gpu
```

---

# Example Banking Workload

```yaml
tolerations:

  - key: secure
    operator: Equal
    value: true
    effect: NoSchedule
```

---

# Example Monitoring Workload

```yaml
affinity:

  nodeAffinity:

    requiredDuringSchedulingIgnoredDuringExecution:
```

---

# Important Troubleshooting Commands

# Check Nodes

```bash
kubectl get nodes
```

---

# Show Labels

```bash
kubectl get nodes --show-labels
```

---

# Describe Node

```bash
kubectl describe node <node-name>
```

---

# Check Pod Placement

```bash
kubectl get pods -o wide
```

---

# Describe Pod

```bash
kubectl describe pod <pod-name>
```

---

# Enterprise Scheduling Best Practices

| Best Practice | Purpose |
|---|---|
| Dedicated GPU Nodes | AI workloads |
| Taints for Secure Nodes | Isolation |
| Affinity Rules | Smart scheduling |
| Resource Limits | Prevent overload |
| Monitoring Scheduling | Predictability |

---

# Node Selector vs Affinity

| Feature | Node Selector | Node Affinity |
|---|---|---|
| Simple | Yes | No |
| Advanced Rules | No | Yes |
| Expressions | No | Yes |
| Preferred Scheduling | No | Yes |
| Enterprise Usage | Limited | High |

---

# Taints vs Tolerations

| Feature | Purpose |
|---|---|
| Taints | Block workloads |
| Tolerations | Allow workloads |

---

# Complete Real Enterprise Workflow

```text
Infrastructure Team
        |
        v
Create Specialized Nodes
        |
        v
Add Labels & Taints
        |
        v
DevOps Team Deploys Apps
        |
        v
Scheduler Evaluates Rules
        |
        v
Correct Node Selected
        |
        v
Enterprise Workload Isolation
```

---

# Production Architecture

```text
                     Kubernetes Cluster
                               |
       +-----------------------+----------------------+
       |                       |                      |
       v                       v                      v

+---------------+     +---------------+     +---------------+
| GPU Nodes     |     | Secure Nodes  |     | General Nodes |
| AI Workloads  |     | Banking Apps  |     | APIs/Web Apps |
+---------------+     +---------------+     +---------------+

       ^                       ^                      ^
       |                       |                      |

 NodeSelector            Tolerations            Default Apps
 NodeAffinity
```

---

# Expected Outcome

- Understood Kubernetes scheduling deeply
- Learned Node Selector
- Learned Node Affinity
- Learned Taints
- Learned Tolerations
- Understood enterprise workload isolation
- Built production-grade Kubernetes scheduling skills

---

# Skills Covered

- Kubernetes Scheduling
- Node Selector
- Node Affinity
- Taints
- Tolerations
- Kubernetes Administration
- Kubernetes Architecture
- DevOps
- Cloud-Native Infrastructure
- Production Kubernetes