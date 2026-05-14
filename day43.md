# Kubernetes Troubleshooting — CrashLoopBackOff (3 Real-Time Scenarios)

# What is CrashLoopBackOff?

CrashLoopBackOff occurs when a container starts, crashes, Kubernetes restarts it, and the cycle repeats continuously.

Kubernetes automatically retries the container startup, but after multiple failures it enters:

```text
CrashLoopBackOff
```

This is one of the most common production Kubernetes issues.

---

# Real Enterprise Impact

CrashLoopBackOff can affect:

- Banking applications
- AI/ML APIs
- E-commerce systems
- Authentication services
- Internal enterprise platforms

Production consequences:

- Downtime
- Revenue loss
- API failures
- User login failures
- Service unavailability

---

# Common Reasons for CrashLoopBackOff

| Reason | Description |
|---|---|
| Wrong Environment Variables | Missing configs |
| Database Connection Failure | DB unavailable |
| Application Bug | Code crashes |
| Wrong Startup Command | Invalid entrypoint |
| Missing Secrets | App cannot authenticate |
| Port Misconfiguration | Service failure |
| Memory Limits | OOMKilled |
| Missing Files | Application dependency issue |

---

# Kubernetes Troubleshooting Workflow

```text
Application Failed
        |
        v
kubectl get pods
        |
        v
Identify CrashLoopBackOff
        |
        v
kubectl describe pod
        |
        v
Check Events & Exit Codes
        |
        v
kubectl logs
        |
        v
Identify Root Cause
        |
        v
Fix Configuration
        |
        v
Redeploy Application
```

---

# Scenario 1 — Wrong Environment Variable

# Real-Time Enterprise Scenario

xFusionCorp Industries deployed a Python banking API on Kubernetes.

The application requires:

```text
APP_ENV=production
```

But the DevOps team accidentally configured:

```text
APP_ENV=prod
```

The application crashes during startup validation.

---

# Step 1: Create Broken Deployment

```bash
nano env-crash-app.yaml
```

Add:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: env-crash-app

spec:
  replicas: 1

  selector:
    matchLabels:
      app: env-crash

  template:
    metadata:
      labels:
        app: env-crash

    spec:
      containers:

        - name: env-container

          image: python:3.11

          command:
            - sh
            - -c

          args:
            - |
              if [ "$APP_ENV" != "production" ]; then
                echo "Invalid APP_ENV"
                exit 1
              fi

              python -m http.server 80

          env:
            - name: APP_ENV
              value: prod
```

---

# Step 2: Deploy Application

```bash
kubectl apply -f env-crash-app.yaml
```

---

# Step 3: Verify Pod Failure

```bash
kubectl get pods
```

Expected:

```text
CrashLoopBackOff
```

---

# Step 4: Describe Pod

```bash
kubectl describe pod <pod-name>
```

Expected:

```text
Back-off restarting failed container
```

---

# Step 5: Check Logs

```bash
kubectl logs <pod-name>
```

Expected:

```text
Invalid APP_ENV
```

---

# Root Cause

Wrong environment variable value:

```text
prod
```

Expected:

```text
production
```

---

# Step 6: Fix Deployment

```bash
kubectl edit deployment env-crash-app
```

Replace:

```yaml
value: prod
```

with:

```yaml
value: production
```

Save and exit.

---

# Step 7: Verify Rollout

```bash
kubectl rollout status deployment/env-crash-app
```

---

# Step 8: Verify Pods

```bash
kubectl get pods
```

Expected:

```text
Running
```

---

# Scenario 2 — Database Connection Failure

# Real-Time Enterprise Scenario

An enterprise payroll application starts only if MySQL database is reachable.

However:

- Database service name is incorrect
- Application cannot connect
- Startup script exits immediately

This causes:

```text
CrashLoopBackOff
```

---

# Step 1: Create Broken Deployment

```bash
nano db-crash-app.yaml
```

Add:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: db-crash-app

spec:
  replicas: 1

  selector:
    matchLabels:
      app: db-crash

  template:
    metadata:
      labels:
        app: db-crash

    spec:
      containers:

        - name: db-container

          image: busybox

          command:
            - sh
            - -c

          args:
            - |
              nslookup wrong-mysql-service

              if [ $? -ne 0 ]; then
                echo "Database connection failed"
                exit 1
              fi

              sleep 3600
```

---

# Step 2: Deploy Application

```bash
kubectl apply -f db-crash-app.yaml
```

---

# Step 3: Verify Pod Failure

```bash
kubectl get pods
```

Expected:

```text
CrashLoopBackOff
```

---

# Step 4: Check Logs

```bash
kubectl logs <pod-name>
```

Expected:

```text
Database connection failed
```

---

# Step 5: Describe Pod

```bash
kubectl describe pod <pod-name>
```

Expected:

```text
Error
Exit Code: 1
```

---

# Root Cause

Wrong MySQL service name:

```text
wrong-mysql-service
```

---

# Step 6: Fix Deployment

```bash
kubectl edit deployment db-crash-app
```

Replace:

```bash
wrong-mysql-service
```

with actual service:

```bash
mysql-service
```

---

# Step 7: Verify Rollout

```bash
kubectl rollout status deployment/db-crash-app
```

---

# Step 8: Verify Pod Status

```bash
kubectl get pods
```

Expected:

```text
Running
```

---

# Scenario 3 — Wrong Startup Command

# Real-Time Enterprise Scenario

A Node.js microservice was deployed with an invalid startup command.

Instead of:

```bash
npm start
```

the deployment used:

```bash
npm run wrong-script
```

The application crashes immediately.

---

# Step 1: Create Broken Deployment

```bash
nano command-crash-app.yaml
```

Add:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: command-crash-app

spec:
  replicas: 1

  selector:
    matchLabels:
      app: command-crash

  template:
    metadata:
      labels:
        app: command-crash

    spec:
      containers:

        - name: command-container

          image: node:20

          command:
            - sh
            - -c

          args:
            - |
              npm run wrong-script
```

---

# Step 2: Deploy Application

```bash
kubectl apply -f command-crash-app.yaml
```

---

# Step 3: Verify Failure

```bash
kubectl get pods
```

Expected:

```text
CrashLoopBackOff
```

---

# Step 4: Check Logs

```bash
kubectl logs <pod-name>
```

Expected:

```text
missing script: wrong-script
```

---

# Root Cause

Invalid startup command.

---

# Step 5: Fix Deployment

```bash
kubectl edit deployment command-crash-app
```

Replace:

```bash
npm run wrong-script
```

with:

```bash
npm start
```

---

# Step 6: Verify Rollout

```bash
kubectl rollout status deployment/command-crash-app
```

---

# Step 7: Verify Pods

```bash
kubectl get pods
```

Expected:

```text
Running
```

---

# Important Kubernetes Troubleshooting Commands

# Check Pod Status

```bash
kubectl get pods
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

# Follow Logs Live

```bash
kubectl logs -f <pod-name>
```

---

# Check Previous Container Logs

```bash
kubectl logs --previous <pod-name>
```

---

# Describe Deployment

```bash
kubectl describe deployment <deployment-name>
```

---

# Check Events

```bash
kubectl get events
```

---

# Restart Deployment

```bash
kubectl rollout restart deployment <deployment-name>
```

---

# Enterprise CrashLoopBackOff Troubleshooting Checklist

| Step | Action |
|---|---|
| 1 | Check Pod status |
| 2 | Describe Pod |
| 3 | Check Events |
| 4 | Check Logs |
| 5 | Identify Exit Code |
| 6 | Verify Environment Variables |
| 7 | Verify Secrets |
| 8 | Verify Service Connectivity |
| 9 | Verify Startup Commands |
| 10 | Redeploy Application |

---

# Important Exit Codes

| Exit Code | Meaning |
|---|---|
| 0 | Success |
| 1 | Application error |
| 126 | Permission issue |
| 127 | Command not found |
| 137 | OOMKilled |
| 139 | Segmentation fault |

---

# Memory-Related CrashLoopBackOff

Example:

```text
OOMKilled
```

Check:

```bash
kubectl describe pod <pod-name>
```

Fix by increasing memory:

```yaml
resources:
  requests:
    memory: "256Mi"

  limits:
    memory: "512Mi"
```

---

# Real Production Workflow

```text
Production Outage
        |
        v
PagerDuty Alert
        |
        v
Engineer Checks Pods
        |
        v
CrashLoopBackOff Detected
        |
        v
Check Logs & Events
        |
        v
Root Cause Analysis
        |
        v
Fix Deployment
        |
        v
Rolling Update
        |
        v
Application Restored
```

---

# Enterprise Best Practices

| Best Practice | Purpose |
|---|---|
| Readiness Probes | Traffic only to healthy pods |
| Liveness Probes | Auto-restart unhealthy apps |
| Centralized Logging | Faster debugging |
| Monitoring | Detect failures quickly |
| CI/CD Validation | Prevent bad deployments |
| Secrets Management | Secure credentials |

---

# Example Readiness Probe

```yaml
readinessProbe:
  httpGet:
    path: /
    port: 80

  initialDelaySeconds: 5
  periodSeconds: 10
```

---

# Example Liveness Probe

```yaml
livenessProbe:
  httpGet:
    path: /
    port: 80

  initialDelaySeconds: 15
  periodSeconds: 20
```

---

# Expected Outcome

- Understood CrashLoopBackOff deeply
- Diagnosed environment variable issues
- Diagnosed database connection failures
- Diagnosed startup command issues
- Learned Kubernetes troubleshooting workflow
- Learned enterprise debugging techniques
- Built production troubleshooting skills

---

# Skills Covered

- Kubernetes Troubleshooting
- CrashLoopBackOff Resolution
- Pod Debugging
- Logs Analysis
- Deployment Debugging
- DevOps
- Cloud-Native Debugging
- Kubernetes Production Support
- Root Cause Analysis

---

# Real Enterprise Architecture

```text
Users
   |
   v
Load Balancer
   |
   v
Kubernetes Services
   |
   v
Application Pods
   |
   v
Monitoring + Logging
   |
   v
Alerting Systems
```