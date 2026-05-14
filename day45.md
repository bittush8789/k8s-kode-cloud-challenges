# Kubernetes Troubleshooting — StatefulSet Persistent Volume Not Working After Cloud Migration

# Real Enterprise Production Scenario

xFusionCorp Industries recently migrated its Kubernetes infrastructure from:

```text
AWS EKS  --->  Azure AKS
```

After migration, critical applications started failing:

- MySQL databases
- MongoDB clusters
- Redis Stateful workloads
- Kafka brokers
- Elasticsearch nodes

The root cause:

```text
Persistent Volumes were not mounting correctly after migration.
```

Applications entered:

```text
CrashLoopBackOff
Pending
ContainerCreating
```

This caused:

- Database downtime
- Customer impact
- Data access failure
- Production outage
- Application crashes

The DevOps team must troubleshoot and restore Stateful workloads immediately.

---

# What is StatefulSet?

StatefulSet is used for applications requiring:

- Persistent storage
- Stable network identity
- Ordered deployment
- Ordered scaling

Examples:

| Application | Why StatefulSet? |
|---|---|
| MySQL | Persistent database |
| MongoDB | Stateful storage |
| Kafka | Stable brokers |
| Redis | Persistent cache |
| Elasticsearch | Persistent indexes |

---

# StatefulSet Architecture

```text
                    +----------------------+
                    |   Kubernetes Cluster |
                    +----------+-----------+
                               |
                               v
                    +----------------------+
                    |    StatefulSet       |
                    +----------+-----------+
                               |
          +--------------------+--------------------+
          |                    |                    |
          v                    v                    v

    +-----------+       +-----------+       +-----------+
    | mysql-0   |       | mysql-1   |       | mysql-2   |
    +-----------+       +-----------+       +-----------+

          |                    |                    |
          v                    v                    v

      PVC-0                PVC-1                PVC-2
          |                    |                    |
          v                    v                    v

      PV-0                 PV-1                 PV-2
```

---

# Common Problems After Cloud Migration

| Problem | Description |
|---|---|
| StorageClass mismatch | Old storage class unavailable |
| PV not bound | PVC stuck pending |
| Volume zone mismatch | Wrong availability zone |
| CSI driver missing | Storage plugin unavailable |
| Access mode mismatch | Invalid volume access |
| Volume attachment failure | Cloud disk issue |
| StatefulSet identity issue | Pod ordering problem |

---

# Real Production Incident

During cloud migration:

- Old AWS EBS StorageClass:
  
```text
gp2
```

- New Azure StorageClass:

```text
managed-premium
```

But StatefulSet YAML still referenced:

```yaml
storageClassName: gp2
```

Result:

```text
PVC Pending
Pods Stuck
Database Down
```

---

# Enterprise Troubleshooting Workflow

```text
Production Database Down
           |
           v
Check StatefulSet Pods
           |
           v
Pods Pending / CrashLoopBackOff
           |
           v
Check PVC Status
           |
           v
PVC Pending
           |
           v
Check StorageClass
           |
           v
Identify Migration Mismatch
           |
           v
Fix Storage Configuration
           |
           v
Restore Stateful Workloads
```

---

# Step 1 — Create Broken StatefulSet

# Broken Scenario

Old StorageClass:

```text
gp2
```

does not exist after migration.

---

# Create StatefulSet Manifest

```bash
nano broken-statefulset.yaml
```

Add:

```yaml
apiVersion: apps/v1
kind: StatefulSet

metadata:
  name: mysql-db

spec:
  serviceName: mysql-service

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

          volumeMounts:
            - name: mysql-storage
              mountPath: /var/lib/mysql

  volumeClaimTemplates:

    - metadata:
        name: mysql-storage

      spec:

        accessModes:
          - ReadWriteOnce

        storageClassName: gp2

        resources:
          requests:
            storage: 2Gi
```

---

# Step 2 — Deploy StatefulSet

```bash
kubectl apply -f broken-statefulset.yaml
```

---

# Step 3 — Check Pods

```bash
kubectl get pods
```

Expected:

```text
Pending
```

---

# Step 4 — Check PVC

```bash
kubectl get pvc
```

Expected:

```text
Pending
```

---

# Step 5 — Describe PVC

```bash
kubectl describe pvc mysql-storage-mysql-db-0
```

Expected:

```text
storageclass.storage.k8s.io "gp2" not found
```

---

# Root Cause

After migration:

```text
AWS gp2 StorageClass no longer exists
```

---

# Step 6 — Check Available StorageClasses

```bash
kubectl get storageclass
```

Expected:

```text
managed-premium
standard
```

---

# Step 7 — Fix StatefulSet

Edit StatefulSet:

```bash
kubectl edit statefulset mysql-db
```

Replace:

```yaml
storageClassName: gp2
```

with:

```yaml
storageClassName: managed-premium
```

---

# Important Note

Kubernetes does NOT allow direct modification of:

```text
volumeClaimTemplates
```

You must:

1. Delete StatefulSet
2. Recreate StatefulSet

without deleting PVC.

---

# Step 8 — Delete StatefulSet

```bash
kubectl delete statefulset mysql-db
```

PVC remains safe.

---

# Step 9 — Recreate Fixed StatefulSet

```bash
nano fixed-statefulset.yaml
```

Add:

```yaml
apiVersion: apps/v1
kind: StatefulSet

metadata:
  name: mysql-db

spec:
  serviceName: mysql-service

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

          volumeMounts:
            - name: mysql-storage
              mountPath: /var/lib/mysql

  volumeClaimTemplates:

    - metadata:
        name: mysql-storage

      spec:

        accessModes:
          - ReadWriteOnce

        storageClassName: managed-premium

        resources:
          requests:
            storage: 2Gi
```

---

# Step 10 — Apply Fixed StatefulSet

```bash
kubectl apply -f fixed-statefulset.yaml
```

---

# Step 11 — Verify PVC

```bash
kubectl get pvc
```

Expected:

```text
Bound
```

---

# Step 12 — Verify Pods

```bash
kubectl get pods
```

Expected:

```text
Running
```

---

# Step 13 — Verify PV

```bash
kubectl get pv
```

Expected:

```text
Bound
```

---

# Real-Time Scenario 2 — Zone Mismatch

# Enterprise Problem

AWS EBS volumes exist in:

```text
us-east-1a
```

But new worker nodes are in:

```text
us-east-1b
```

Result:

```text
Volume attachment failed
```

---

# Error Example

```text
Multi-Attach error
Volume is already attached
```

---

# Diagnose

```bash
kubectl describe pod <pod-name>
```

---

# Fix

Ensure:

- Nodes and volumes are in same zone
- Correct node affinity configured

---

# Example Node Affinity

```yaml
affinity:

  nodeAffinity:

    requiredDuringSchedulingIgnoredDuringExecution:

      nodeSelectorTerms:

        - matchExpressions:

            - key: topology.kubernetes.io/zone

              operator: In

              values:
                - us-east-1a
```

---

# Real-Time Scenario 3 — CSI Driver Missing

# Enterprise Problem

After migration:

- Storage CSI driver not installed
- Kubernetes cannot provision volumes

---

# Symptoms

PVC stuck:

```text
Pending
```

---

# Diagnose

```bash
kubectl get pods -n kube-system
```

Check CSI drivers:

```bash
kubectl get csidrivers
```

---

# Fix

Install proper CSI driver.

Examples:

| Cloud | CSI Driver |
|---|---|
| AWS | EBS CSI Driver |
| Azure | Azure Disk CSI |
| GCP | GCE PD CSI |

---

# AWS EBS CSI Installation

```bash
eksctl create addon \
--name aws-ebs-csi-driver
```

---

# Azure CSI Example

```bash
kubectl get csidrivers
```

Expected:

```text
disk.csi.azure.com
```

---

# Real-Time Scenario 4 — Access Mode Issue

# Problem

Application requests:

```yaml
ReadWriteMany
```

But storage supports only:

```yaml
ReadWriteOnce
```

Result:

```text
PVC Pending
```

---

# Diagnose

```bash
kubectl describe pvc
```

---

# Fix

Use compatible access mode.

---

# Common Access Modes

| Access Mode | Meaning |
|---|---|
| ReadWriteOnce | Single node |
| ReadOnlyMany | Multiple read-only |
| ReadWriteMany | Multiple read/write |

---

# Real-Time Scenario 5 — StatefulSet Identity Problems

# Problem

Pods lose stable identity after migration.

Expected:

```text
mysql-0
mysql-1
mysql-2
```

But recreated incorrectly.

---

# Enterprise Impact

- Database replication breaks
- Kafka brokers fail
- Elasticsearch cluster unstable

---

# Fix

Never delete PVC accidentally.

Always preserve:

```text
PersistentVolumeClaims
```

---

# StatefulSet Best Practices

| Best Practice | Purpose |
|---|---|
| Use CSI Drivers | Dynamic storage |
| Backup Before Migration | Data protection |
| Validate StorageClass | Avoid mismatch |
| Use Volume Snapshots | Disaster recovery |
| Preserve PVCs | Prevent data loss |

---

# Important Kubernetes Commands

# Check StatefulSets

```bash
kubectl get statefulset
```

---

# Check Pods

```bash
kubectl get pods
```

---

# Check PVC

```bash
kubectl get pvc
```

---

# Check PV

```bash
kubectl get pv
```

---

# Check StorageClass

```bash
kubectl get storageclass
```

---

# Describe PVC

```bash
kubectl describe pvc <pvc-name>
```

---

# Describe Pod

```bash
kubectl describe pod <pod-name>
```

---

# Check Events

```bash
kubectl get events
```

---

# Production Troubleshooting Checklist

| Step | Action |
|---|---|
| 1 | Check Pods |
| 2 | Check PVC |
| 3 | Check PV |
| 4 | Check StorageClass |
| 5 | Verify CSI Driver |
| 6 | Check Zones |
| 7 | Verify Access Modes |
| 8 | Verify Volume Attachments |
| 9 | Check Events |
| 10 | Restore StatefulSet |

---

# Enterprise Cloud Migration Workflow

```text
Old Kubernetes Cluster
         |
         v
Backup Volumes & Snapshots
         |
         v
Provision New Cluster
         |
         v
Install CSI Drivers
         |
         v
Validate StorageClasses
         |
         v
Restore StatefulSets
         |
         v
Reattach Persistent Volumes
         |
         v
Verify Database Recovery
```

---

# Production Architecture

```text
                Kubernetes Cluster
                         |
         +---------------+---------------+
         |                               |
         v                               v

   StatefulSet Pods                Persistent Storage
         |                               |
         v                               v

   mysql-0                         PersistentVolume
   mysql-1                         PersistentVolume
   mysql-2                         PersistentVolume
         |                               |
         +---------------+---------------+
                         |
                         v
                    CSI Drivers
                         |
                         v
                 Cloud Storage Layer
```

---

# Expected Outcome

- Understood StatefulSet deeply
- Learned Persistent Volume troubleshooting
- Diagnosed StorageClass migration issues
- Learned CSI driver troubleshooting
- Learned cloud migration debugging
- Learned enterprise storage architecture
- Built production-grade troubleshooting skills

---

# Skills Covered

- Kubernetes StatefulSet
- Persistent Volumes
- Persistent Volume Claims
- StorageClasses
- CSI Drivers
- Cloud Migration
- Kubernetes Troubleshooting
- DevOps
- Cloud-Native Storage
- Production Kubernetes
- Database Infrastructure
```