# Secure Your Database Using Kubernetes Network Policies

# Real Enterprise Production Scenario

xFusionCorp Industries manages enterprise-grade applications on Kubernetes including:

- Banking systems
- AI/ML platforms
- Customer APIs
- Internal dashboards
- Authentication services

The organization recently discovered a serious security issue:

```text
Any Pod inside the Kubernetes cluster could access the production database.
```

This created major risks:

- Unauthorized database access
- Lateral movement attacks
- Internal security breaches
- Compliance violations
- Data leakage risks

The security team mandated:

```text
Only authorized backend applications should communicate with the database.
```

The DevOps team must implement:

```text
Kubernetes Network Policies
```

to secure database communication.

---

# What are Kubernetes Network Policies?

Network Policies control:

```text
Which Pods can communicate with other Pods
```

Think of Network Policies as:

```text
Kubernetes Firewall Rules
```

---

# Without Network Policies

Default Kubernetes behavior:

```text
All Pods can talk to all Pods
```

This is dangerous in production.

---

# With Network Policies

You can restrict:

- Pod-to-Pod traffic
- Namespace traffic
- Database access
- API communication
- Egress internet access

---

# Enterprise Security Architecture

```text
                  +----------------------+
                  | Frontend Pods        |
                  +----------+-----------+
                             |
                             v

                  +----------------------+
                  | Backend API Pods     |
                  +----------+-----------+
                             |
                             v

                  +----------------------+
                  | MySQL Database Pods  |
                  +----------------------+

                    ^                ^
                    |                |

            Allowed Traffic    Blocked Traffic
```

---

# Real Enterprise Use Cases

| Use Case | Purpose |
|---|---|
| Database Isolation | Prevent unauthorized access |
| Banking Apps | Secure sensitive workloads |
| Zero Trust Security | Least privilege networking |
| Multi-Tenant Clusters | Tenant isolation |
| Compliance | PCI DSS / HIPAA |

---

# Kubernetes Networking Flow

```text
Pod Created
      |
      v
Network Policy Engine
      |
      +----------------------+
      |                      |
      v                      v

Traffic Allowed      Traffic Blocked
```

---

# Important Concept

# By Default

If NO Network Policies exist:

```text
All traffic allowed
```

---

# Once Network Policies Applied

Traffic becomes:

```text
Deny by default
```

unless explicitly allowed.

---

# Real Production Architecture

```text
                  Kubernetes Cluster
                           |
      +--------------------+--------------------+
      |                    |                    |
      v                    v                    v

+-------------+     +-------------+     +-------------+
| Frontend    |     | Backend API |     | MySQL DB    |
+-------------+     +-------------+     +-------------+

      |                    |                    |
      |                    +---------> Allowed
      |
      +-------------------> Blocked
```

---

# Step 1 — Create Namespace

```bash
kubectl create namespace secure-app
```

---

# Step 2 — Deploy MySQL Database

```bash
nano mysql-deployment.yaml
```

Add:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: mysql-db
  namespace: secure-app

spec:
  replicas: 1

  selector:
    matchLabels:
      app: mysql

  template:
    metadata:
      labels:
        app: mysql

    spec:
      containers:

        - name: mysql

          image: mysql:8

          env:
            - name: MYSQL_ROOT_PASSWORD
              value: root123

          ports:
            - containerPort: 3306
```

---

# Step 3 — Deploy MySQL Service

```bash
nano mysql-service.yaml
```

Add:

```yaml
apiVersion: v1
kind: Service

metadata:
  name: mysql-service
  namespace: secure-app

spec:
  selector:
    app: mysql

  ports:
    - port: 3306
      targetPort: 3306

  type: ClusterIP
```

---

# Step 4 — Apply MySQL Resources

```bash
kubectl apply -f mysql-deployment.yaml

kubectl apply -f mysql-service.yaml
```

---

# Step 5 — Deploy Backend API

```bash
nano backend-deployment.yaml
```

Add:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: backend-api
  namespace: secure-app

spec:
  replicas: 2

  selector:
    matchLabels:
      app: backend

  template:
    metadata:
      labels:
        app: backend

    spec:
      containers:

        - name: backend

          image: nginx

          ports:
            - containerPort: 80
```

---

# Step 6 — Deploy Unauthorized Pod

This Pod simulates a hacker or unauthorized workload.

```bash
nano hacker-pod.yaml
```

Add:

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: hacker-pod
  namespace: secure-app

  labels:
    app: hacker

spec:
  containers:

    - name: hacker

      image: busybox

      command:
        - sleep
        - "3600"
```

---

# Step 7 — Apply Resources

```bash
kubectl apply -f backend-deployment.yaml

kubectl apply -f hacker-pod.yaml
```

---

# Step 8 — Verify Resources

```bash
kubectl get pods -n secure-app
```

Expected:

```text
mysql-db
backend-api
hacker-pod
```

---

# Step 9 — Verify Database Access BEFORE Network Policy

# Access Hacker Pod

```bash
kubectl exec -it hacker-pod \
-n secure-app -- sh
```

Install MySQL client:

```bash
wget https://busybox.net/downloads/binaries/1.35.0-i686-uclibc/busybox
```

Test DB connectivity:

```bash
nc -zv mysql-service 3306
```

Expected:

```text
Connection successful
```

---

# Security Problem

Unauthorized Pod can access production database.

---

# Step 10 — Create Network Policy

```bash
nano mysql-network-policy.yaml
```

Add:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy

metadata:
  name: mysql-network-policy
  namespace: secure-app

spec:

  podSelector:
    matchLabels:
      app: mysql

  policyTypes:
    - Ingress

  ingress:

    - from:

        - podSelector:
            matchLabels:
              app: backend

      ports:

        - protocol: TCP
          port: 3306
```

---

# What This Policy Does

# Allows:

```text
backend Pods ---> mysql Pods
```

# Blocks:

```text
All other Pods
```

including:

```text
hacker-pod
```

---

# Step 11 — Apply Network Policy

```bash
kubectl apply -f mysql-network-policy.yaml
```

---

# Step 12 — Verify Policy

```bash
kubectl get networkpolicy -n secure-app
```

---

# Step 13 — Test Hacker Pod Again

```bash
kubectl exec -it hacker-pod \
-n secure-app -- sh
```

Test DB access:

```bash
nc -zv mysql-service 3306
```

Expected:

```text
Connection timed out
```

---

# Step 14 — Verify Backend Access

Get backend Pod:

```bash
kubectl get pods -n secure-app
```

Access backend Pod:

```bash
kubectl exec -it <backend-pod> \
-n secure-app -- sh
```

Test DB connectivity:

```bash
nc -zv mysql-service 3306
```

Expected:

```text
Connection successful
```

---

# Network Policy Flow

```text
Traffic Request
      |
      v
Network Policy Evaluated
      |
      +----------------------+
      |                      |
      v                      v

Authorized Pod       Unauthorized Pod
      |                      |
      v                      v

Traffic Allowed      Traffic Blocked
```

---

# Important Kubernetes Concepts

| Concept | Meaning |
|---|---|
| podSelector | Select target Pods |
| ingress | Incoming traffic rules |
| egress | Outgoing traffic rules |
| policyTypes | Traffic direction |

---

# Types of Network Policies

| Type | Purpose |
|---|---|
| Ingress | Control incoming traffic |
| Egress | Control outgoing traffic |
| Both | Full isolation |

---

# Example Egress Policy

Block internet access:

```yaml
policyTypes:
  - Egress
```

---

# Example Namespace Isolation

Allow traffic only from monitoring namespace:

```yaml
from:

  - namespaceSelector:

      matchLabels:
        name: monitoring
```

---

# Example Combined Security Policy

```yaml
ingress:
  - from:
      - podSelector:
          matchLabels:
            app: backend

egress:
  - to:
      - podSelector:
          matchLabels:
            app: mysql
```

---

# Real Production Banking Architecture

```text
                 Banking Namespace
                          |
       +------------------+------------------+
       |                                     |
       v                                     v

+---------------+                   +---------------+
| API Pods      | ----------------> | MySQL DB      |
+---------------+                   +---------------+

        ^
        |
        |

Blocked Unauthorized Traffic
```

---

# Important Troubleshooting Commands

# Check Policies

```bash
kubectl get networkpolicy -A
```

---

# Describe Policy

```bash
kubectl describe networkpolicy \
mysql-network-policy \
-n secure-app
```

---

# Check Pod Labels

```bash
kubectl get pods --show-labels \
-n secure-app
```

---

# Test Connectivity

```bash
nc -zv mysql-service 3306
```

---

# Check Endpoints

```bash
kubectl get endpoints -n secure-app
```

---

# Common Network Policy Problems

| Problem | Cause |
|---|---|
| Policy not working | CNI plugin unsupported |
| Traffic still allowed | Wrong labels |
| Backend blocked | Incorrect selectors |
| No connectivity | Overly restrictive policy |

---

# Important Note About CNI Plugins

Network Policies require supported CNI plugins:

| CNI Plugin | Supports Network Policies |
|---|---|
| Calico | Yes |
| Cilium | Yes |
| Weave | Yes |
| Flannel | Limited |

---

# Verify CNI Plugin

```bash
kubectl get pods -n kube-system
```

---

# Enterprise Security Best Practices

| Best Practice | Purpose |
|---|---|
| Zero Trust Networking | Secure clusters |
| Least Privilege Access | Minimal exposure |
| Namespace Isolation | Multi-tenancy |
| Default Deny Policies | Maximum protection |
| Separate DB Namespace | Strong isolation |

---

# Default Deny Policy Example

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy

metadata:
  name: default-deny

spec:
  podSelector: {}

  policyTypes:
    - Ingress
```

---

# Enterprise Security Workflow

```text
Deploy Application
        |
        v
Apply Default Deny Policy
        |
        v
Allow Required Services
        |
        v
Block Unauthorized Traffic
        |
        v
Continuously Monitor Traffic
        |
        v
Secure Production Cluster
```

---

# Real Production Architecture

```text
                    Kubernetes Cluster
                               |
        +----------------------+----------------------+
        |                                             |
        v                                             v

+------------------+                     +------------------+
| Frontend Pods    |                     | Backend Pods     |
+------------------+                     +------------------+
                                                     |
                                                     v

                                            +------------------+
                                            | MySQL Database   |
                                            +------------------+

                     Network Policies Enforced
```

---

# Expected Outcome

- Understood Kubernetes Network Policies deeply
- Learned database isolation
- Secured MySQL using Network Policies
- Blocked unauthorized Pod access
- Implemented zero-trust networking
- Learned enterprise Kubernetes security
- Built production-grade cluster security skills

---

# Skills Covered

- Kubernetes Network Policies
- Kubernetes Security
- Pod Isolation
- Database Security
- Kubernetes Networking
- DevSecOps
- Cloud-Native Security
- Production Kubernetes
- Zero Trust Networking
- Enterprise Security Architecture