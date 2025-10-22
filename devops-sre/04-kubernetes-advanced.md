# 4. Kubernetes Advanced

## 📚 Quick Summary

Master advanced Kubernetes for production-grade, stateful, and complex workloads!

**What You'll Learn:**
- **StatefulSets**: Stateful applications (databases, message queues)
- **DaemonSets**: One pod per node (monitoring, logging agents)
- **Jobs & CronJobs**: Batch processing and scheduled tasks
- **Custom Resources**: Extend Kubernetes with CRDs
- **Operators**: Automate complex applications
- **Advanced Scheduling**: Node affinity, taints, tolerations
- **Horizontal Pod Autoscaler (HPA)**: Auto-scale based on metrics
- **Vertical Pod Autoscaler (VPA)**: Right-size resource requests
- **Persistent Storage**: Volumes, PVs, PVCs, Storage Classes

**Why This Matters:**
- Production K8s needs more than Deployments
- Stateful apps (databases) require StatefulSets
- Operators automate Day-2 operations
- Auto-scaling saves costs and ensures performance
- Storage is critical for data persistence
- Interview weight: 15-20% of K8s questions

**Interview Reality:**
"How would you run a database on Kubernetes?" = StatefulSets!

---

## 📖 Simple Explanation

**Beyond Basic Deployments:**

```
Basic K8s (Chapter 3):
- Deployments (stateless apps)
- Services (networking)
- ConfigMaps (configuration)
→ Covers 70% of use cases

Advanced K8s (Chapter 4):
- StatefulSets (databases, queues)
- DaemonSets (node agents)
- Jobs (batch processing)
- Operators (automation)
- Auto-scaling (HPA, VPA)
- Advanced scheduling
→ Covers remaining 30%, critical for production
```

**Why Advanced Patterns?**
- **Stateful apps need guarantees**: Consistent network identity, ordered startup
- **Node-level services**: Monitoring agent on EVERY node
- **Batch processing**: Run to completion, then stop
- **Custom automation**: Operators codify operational knowledge
- **Resource optimization**: Auto-scale up/down based on load

---

## Table of Contents
- [StatefulSets](#statefulsets)
- [DaemonSets](#daemonsets)
- [Jobs & CronJobs](#jobs--cronjobs)
- [Horizontal Pod Autoscaler (HPA)](#horizontal-pod-autoscaler-hpa)
- [Vertical Pod Autoscaler (VPA)](#vertical-pod-autoscaler-vpa)
- [Persistent Volumes & Claims](#persistent-volumes--claims)
- [Storage Classes](#storage-classes)
- [Advanced Scheduling](#advanced-scheduling)
- [Custom Resource Definitions (CRDs)](#custom-resource-definitions-crds)
- [Operators](#operators)

---

## StatefulSets

### 📖 Simple Explanation

**StatefulSet = For stateful applications**

**Deployment vs StatefulSet:**

```
Deployment (Stateless):
- Pods are interchangeable
- Any pod can die, new one replaces it
- Random names: web-abc123, web-def456
- No guaranteed order
- Use case: Web servers, APIs

StatefulSet (Stateful):
- Pods have stable identity
- Predictable names: db-0, db-1, db-2
- Ordered startup/shutdown
- Persistent storage per pod
- Stable network identity
- Use case: Databases, message queues, Zookeeper
```

**Why StatefulSets?**

Example: PostgreSQL cluster with replication
```
Requirements:
1. Master starts before replicas (order matters)
2. Each instance needs own storage (not shared)
3. Clients connect to master by name (stable DNS)
4. Pod restart = same identity, same storage

Deployment? ❌ Can't guarantee these
StatefulSet? ✅ Built for this
```

---

### StatefulSet Components

**1. Stable Network Identity:**
```
StatefulSet: postgresql
Pods:
- postgresql-0.postgresql.default.svc.cluster.local
- postgresql-1.postgresql.default.svc.cluster.local
- postgresql-2.postgresql.default.svc.cluster.local

Each pod has predictable DNS name!
```

**2. Ordered Deployment/Scaling:**
```
Startup: 0 → 1 → 2 (waits for each to be Ready)
Shutdown: 2 → 1 → 0 (reverse order)
Scaling up: Creates pods in order
Scaling down: Removes pods in reverse order
```

**3. Persistent Storage:**
```
Each pod gets its own PersistentVolumeClaim
Pod deleted? Storage persists
Pod recreated? Attaches to same storage
```

---

### StatefulSet YAML

```yaml
# statefulset.yaml
apiVersion: v1
kind: Service
metadata:
  name: postgresql
spec:
  selector:
    app: postgresql
  clusterIP: None              # Headless service (required for StatefulSet)
  ports:
  - port: 5432
    targetPort: 5432

---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgresql
spec:
  serviceName: postgresql      # Links to headless service
  replicas: 3
  selector:
    matchLabels:
      app: postgresql
  template:
    metadata:
      labels:
        app: postgresql
    spec:
      containers:
      - name: postgresql
        image: postgres:15
        env:
        - name: POSTGRES_PASSWORD
          value: secret
        ports:
        - containerPort: 5432
          name: postgres
        volumeMounts:
        - name: data
          mountPath: /var/lib/postgresql/data
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
  volumeClaimTemplates:        # Creates PVC for each pod
  - metadata:
      name: data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 10Gi
```

---

### StatefulSet Commands

```bash
# Create StatefulSet
kubectl apply -f statefulset.yaml

# List StatefulSets
kubectl get statefulsets
kubectl get sts              # shorthand

# Watch pods starting in order
kubectl get pods -w

# Check persistent volume claims
kubectl get pvc

# Describe StatefulSet
kubectl describe statefulset postgresql

# Scale StatefulSet
kubectl scale statefulset postgresql --replicas=5

# Delete StatefulSet (keeps PVCs by default)
kubectl delete statefulset postgresql

# Delete including PVCs
kubectl delete statefulset postgresql
kubectl delete pvc -l app=postgresql

# Access specific pod
kubectl exec -it postgresql-0 -- bash
kubectl exec -it postgresql-1 -- bash
```

**Real Example:**
```bash
# Deploy StatefulSet
$ kubectl apply -f statefulset.yaml
service/postgresql created
statefulset.apps/postgresql created

# Watch pods starting in order
$ kubectl get pods -w
NAME           READY   STATUS              AGE
postgresql-0   0/1     ContainerCreating   5s
# Waits for postgresql-0 to be Running
postgresql-0   1/1     Running             15s
postgresql-1   0/1     Pending             0s
postgresql-1   0/1     ContainerCreating   0s
# Waits for postgresql-1 to be Running
postgresql-1   1/1     Running             25s
postgresql-2   0/1     Pending             0s
postgresql-2   0/1     ContainerCreating   0s
postgresql-2   1/1     Running             35s

# Check PVCs (one per pod)
$ kubectl get pvc
NAME                STATUS   VOLUME     CAPACITY
data-postgresql-0   Bound    pvc-abc    10Gi
data-postgresql-1   Bound    pvc-def    10Gi
data-postgresql-2   Bound    pvc-ghi    10Gi

# Connect to master (pod 0)
$ kubectl exec -it postgresql-0 -- psql -U postgres
postgres=# CREATE DATABASE test;
postgres=# \q

# Delete pod (it recreates with same identity)
$ kubectl delete pod postgresql-1
pod "postgresql-1" deleted

# New pod gets same name and same storage
$ kubectl get pods
NAME           READY   STATUS    AGE
postgresql-0   1/1     Running   5m
postgresql-1   1/1     Running   10s   ← Same name!
postgresql-2   1/1     Running   5m

# Scale up (adds postgresql-3)
$ kubectl scale sts postgresql --replicas=4
statefulset.apps/postgresql scaled

# Scale down (removes postgresql-3, then postgresql-2)
$ kubectl scale sts postgresql --replicas=2
```

---

## DaemonSets

### 📖 Simple Explanation

**DaemonSet = Run one pod per node**

**Use Cases:**
- **Monitoring agents**: Collect metrics from every node
- **Log collectors**: Forward logs from every node (Fluentd, Logstash)
- **Network plugins**: CNI (Calico, Flannel)
- **Storage daemons**: Ceph, Gluster
- **Security agents**: Falco (runtime security)

**How it works:**
```
Cluster with 5 nodes
DaemonSet: monitoring-agent

Kubernetes automatically runs:
- monitoring-agent-abc on node-1
- monitoring-agent-def on node-2
- monitoring-agent-ghi on node-3
- monitoring-agent-jkl on node-4
- monitoring-agent-mno on node-5

New node added? → Pod automatically created
Node removed? → Pod automatically deleted
```

---

### DaemonSet YAML

```yaml
# daemonset.yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: node-exporter
  labels:
    app: node-exporter
spec:
  selector:
    matchLabels:
      app: node-exporter
  template:
    metadata:
      labels:
        app: node-exporter
    spec:
      hostNetwork: true        # Use host network (access node metrics)
      hostPID: true            # Access host PIDs
      containers:
      - name: node-exporter
        image: prom/node-exporter:latest
        ports:
        - containerPort: 9100
          hostPort: 9100       # Expose on node
        volumeMounts:
        - name: proc
          mountPath: /host/proc
          readOnly: true
        - name: sys
          mountPath: /host/sys
          readOnly: true
        resources:
          requests:
            memory: "50Mi"
            cpu: "100m"
          limits:
            memory: "100Mi"
            cpu: "200m"
      volumes:
      - name: proc
        hostPath:
          path: /proc
      - name: sys
        hostPath:
          path: /sys
      tolerations:             # Run on master nodes too
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
```

---

### DaemonSet Commands

```bash
# Create DaemonSet
kubectl apply -f daemonset.yaml

# List DaemonSets
kubectl get daemonsets
kubectl get ds               # shorthand

# Describe DaemonSet
kubectl describe daemonset node-exporter

# Check pods (one per node)
kubectl get pods -l app=node-exporter -o wide

# Delete DaemonSet
kubectl delete daemonset node-exporter
```

**Real Example:**
```bash
# Deploy DaemonSet
$ kubectl apply -f daemonset.yaml
daemonset.apps/node-exporter created

# Check DaemonSet
$ kubectl get ds
NAME            DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR
node-exporter   3         3         3       3            3           <none>

# Check pods (one per node)
$ kubectl get pods -l app=node-exporter -o wide
NAME                  READY   STATUS    NODE
node-exporter-4xkwl   1/1     Running   worker-1
node-exporter-8m2nq   1/1     Running   worker-2
node-exporter-p7xtj   1/1     Running   worker-3

# Access metrics from any node
$ curl http://worker-1:9100/metrics
# HELP node_cpu_seconds_total Seconds the CPUs spent in each mode.
# TYPE node_cpu_seconds_total counter
node_cpu_seconds_total{cpu="0",mode="idle"} 1234567.89
...
```

---

## Jobs & CronJobs

### 📖 Simple Explanation

**Job = Run to completion**
**CronJob = Scheduled Job**

**Difference from Deployment:**
```
Deployment:
- Runs forever
- Restarts if crashes
- Use case: Web server

Job:
- Runs once, then stops
- Success = exits with code 0
- Use case: Data processing, backup

CronJob:
- Scheduled Job
- Runs at specific times
- Use case: Daily backups, reports
```

---

### Job YAML

```yaml
# job.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: data-processing
spec:
  completions: 3           # Run 3 times
  parallelism: 2           # 2 pods at a time
  backoffLimit: 4          # Retry 4 times on failure
  template:
    spec:
      containers:
      - name: processor
        image: myapp/processor:latest
        command: ["python"]
        args: ["process_data.py"]
        env:
        - name: BATCH_SIZE
          value: "1000"
      restartPolicy: Never  # Required for Jobs
```

**Types of Jobs:**

**1. Non-parallel (completions=1, parallelism=1)**
```yaml
# Run once, one pod
spec:
  completions: 1
  parallelism: 1
```

**2. Parallel with fixed completion count**
```yaml
# Run 10 times, 3 pods at a time
spec:
  completions: 10
  parallelism: 3
```

**3. Parallel with work queue**
```yaml
# Run until queue empty, 5 pods
spec:
  parallelism: 5
  # No completions (pods coordinate via queue)
```

---

### CronJob YAML

```yaml
# cronjob.yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: backup-database
spec:
  schedule: "0 2 * * *"    # Every day at 2 AM (cron syntax)
  successfulJobsHistoryLimit: 3  # Keep last 3 successful
  failedJobsHistoryLimit: 1      # Keep last 1 failed
  concurrencyPolicy: Forbid      # Don't run if previous still running
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: backup
            image: postgres:15
            command: ["/bin/sh"]
            args:
            - -c
            - |
              pg_dump -h postgres -U postgres mydb > /backup/backup-$(date +%Y%m%d).sql
              echo "Backup completed"
            env:
            - name: PGPASSWORD
              valueFrom:
                secretKeyRef:
                  name: db-secret
                  key: password
            volumeMounts:
            - name: backup
              mountPath: /backup
          volumes:
          - name: backup
            persistentVolumeClaim:
              claimName: backup-pvc
          restartPolicy: OnFailure
```

**Cron Schedule Syntax:**
```
┌───────────── minute (0 - 59)
│ ┌───────────── hour (0 - 23)
│ │ ┌───────────── day of month (1 - 31)
│ │ │ ┌───────────── month (1 - 12)
│ │ │ │ ┌───────────── day of week (0 - 6) (Sunday=0)
│ │ │ │ │
* * * * *

Examples:
"0 2 * * *"        # Every day at 2 AM
"*/15 * * * *"     # Every 15 minutes
"0 */4 * * *"      # Every 4 hours
"0 0 * * 0"        # Every Sunday midnight
"0 9-17 * * 1-5"   # Weekdays 9am-5pm
```

---

### Job/CronJob Commands

```bash
# Create Job
kubectl apply -f job.yaml

# List Jobs
kubectl get jobs

# Watch Job progress
kubectl get pods -w

# View Job logs
kubectl logs job/data-processing

# Delete Job
kubectl delete job data-processing

# Create CronJob
kubectl apply -f cronjob.yaml

# List CronJobs
kubectl get cronjobs
kubectl get cj               # shorthand

# Manually trigger CronJob
kubectl create job --from=cronjob/backup-database manual-backup

# Suspend CronJob (stop scheduling)
kubectl patch cronjob backup-database -p '{"spec":{"suspend":true}}'

# Resume CronJob
kubectl patch cronjob backup-database -p '{"spec":{"suspend":false}}'

# Delete CronJob
kubectl delete cronjob backup-database
```

**Real Example:**
```bash
# Create Job
$ kubectl apply -f job.yaml
job.batch/data-processing created

# Watch pods
$ kubectl get pods -w
NAME                      READY   STATUS    AGE
data-processing-abcd1     1/1     Running   5s
data-processing-efgh2     1/1     Running   5s
# Pods complete
data-processing-abcd1     0/1     Completed 30s
data-processing-efgh2     0/1     Completed 32s
data-processing-ijkl3     1/1     Running   0s
data-processing-ijkl3     0/1     Completed 28s

# Check Job status
$ kubectl get job
NAME                COMPLETIONS   DURATION   AGE
data-processing     3/3           90s        2m

# View logs
$ kubectl logs job/data-processing
Processed 1000 records
Processed 1000 records
Processed 1000 records

# Create CronJob
$ kubectl apply -f cronjob.yaml
cronjob.batch/backup-database created

# Check schedule
$ kubectl get cronjob
NAME              SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE
backup-database   0 2 * * *     False     0        <none>

# Manually trigger
$ kubectl create job --from=cronjob/backup-database manual-backup-001
job.batch/manual-backup-001 created

# Check created job
$ kubectl get jobs
NAME                COMPLETIONS   DURATION   AGE
manual-backup-001   1/1           15s        20s
```

---

## Horizontal Pod Autoscaler (HPA)

### 📖 Simple Explanation

**HPA = Auto-scale pods based on metrics**

**How it works:**
```
1. Monitor metrics (CPU, memory, custom)
2. CPU > 80%? → Scale up (add pods)
3. CPU < 30%? → Scale down (remove pods)
4. Stay within min/max limits
```

**Example:**
```
Deployment: webapp
Min replicas: 2
Max replicas: 10
Target CPU: 70%

Current: 2 pods, CPU at 90%
HPA: Scales to 4 pods
CPU drops to 60%

Traffic spike! CPU at 95%
HPA: Scales to 8 pods
CPU drops to 50%

Traffic decreases, CPU at 20%
HPA: Scales down to 3 pods
```

---

### HPA YAML

```yaml
# hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: webapp-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: webapp
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70    # Target 70% CPU
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80    # Target 80% memory
  behavior:                       # Scaling behavior
    scaleDown:
      stabilizationWindowSeconds: 300  # Wait 5min before scaling down
      policies:
      - type: Percent
        value: 50                 # Max 50% pods at a time
        periodSeconds: 60
    scaleUp:
      stabilizationWindowSeconds: 0    # Scale up immediately
      policies:
      - type: Percent
        value: 100                # Double pods if needed
        periodSeconds: 15
```

---

### HPA Commands

```bash
# Create HPA (imperative)
kubectl autoscale deployment webapp --cpu-percent=70 --min=2 --max=10

# Create HPA (declarative)
kubectl apply -f hpa.yaml

# List HPAs
kubectl get hpa

# Describe HPA
kubectl describe hpa webapp-hpa

# Watch HPA in action
kubectl get hpa -w

# Delete HPA
kubectl delete hpa webapp-hpa
```

**Real Example:**
```bash
# Create deployment
$ kubectl create deployment webapp --image=nginx:latest --replicas=2

# Set resource requests (required for CPU-based HPA)
$ kubectl set resources deployment webapp \
  --requests=cpu=100m,memory=128Mi \
  --limits=cpu=500m,memory=256Mi

# Create HPA
$ kubectl autoscale deployment webapp --cpu-percent=70 --min=2 --max=10
horizontalpodautoscaler.autoscaling/webapp autoscaled

# Check HPA
$ kubectl get hpa
NAME     REFERENCE           TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
webapp   Deployment/webapp   45%/70%   2         10        2          1m

# Generate load (simulate traffic)
$ kubectl run load-generator --image=busybox --restart=Never -- \
  /bin/sh -c "while true; do wget -q -O- http://webapp; done"

# Watch HPA scale up
$ kubectl get hpa -w
NAME     REFERENCE           TARGETS    MINPODS   MAXPODS   REPLICAS
webapp   Deployment/webapp   45%/70%    2         10        2
webapp   Deployment/webapp   98%/70%    2         10        2
webapp   Deployment/webapp   98%/70%    2         10        4      # Scaled up!
webapp   Deployment/webapp   55%/70%    2         10        4
webapp   Deployment/webapp   52%/70%    2         10        4

# Stop load
$ kubectl delete pod load-generator

# Watch HPA scale down (after 5min stabilization)
$ kubectl get hpa -w
webapp   Deployment/webapp   15%/70%    2         10        4
# ... waits 5 minutes ...
webapp   Deployment/webapp   15%/70%    2         10        2      # Scaled down!
```

---

## Vertical Pod Autoscaler (VPA)

### 📖 Simple Explanation

**VPA = Auto-adjust resource requests/limits**

**HPA vs VPA:**
```
HPA (Horizontal):
- Adds/removes pods
- Scales OUT (1 pod → 10 pods)
- Handles traffic spikes

VPA (Vertical):
- Adjusts CPU/memory per pod
- Scales UP (100m CPU → 500m CPU)
- Right-sizes resource requests
```

**Use Case:**
```
Problem:
- Set request: 100m CPU
- Actually uses: 500m CPU
- Pod gets throttled!

VPA Solution:
- Monitors actual usage
- Updates request to 500m CPU
- Restarts pod with new resources
```

---

### VPA YAML

```yaml
# vpa.yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: webapp-vpa
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: webapp
  updatePolicy:
    updateMode: "Auto"       # Auto, Recreate, Initial, Off
  resourcePolicy:
    containerPolicies:
    - containerName: webapp
      minAllowed:
        cpu: 100m
        memory: 128Mi
      maxAllowed:
        cpu: 2
        memory: 2Gi
      controlledResources: ["cpu", "memory"]
```

**Update Modes:**
- **Off**: Only recommendations, no changes
- **Initial**: Set requests on pod creation only
- **Recreate**: Update requests, restart pods
- **Auto**: Update requests, restart pods automatically

---

## Persistent Volumes & Claims

### 📖 Simple Explanation

**Problem:** Container dies → data lost!

**Solution: Persistent Volumes**

**3 Components:**
1. **PersistentVolume (PV)**: Actual storage (admin creates)
2. **PersistentVolumeClaim (PVC)**: Request for storage (user creates)
3. **Pod**: Uses PVC

```
PersistentVolume (PV)
    ↓ Bound
PersistentVolumeClaim (PVC)
    ↓ Mounted
Pod → Data persists!
```

**Lifecycle:**
```
1. Admin: Creates PV (100GB disk)
2. User: Creates PVC (requests 10GB)
3. K8s: Binds PVC to PV
4. User: Pod mounts PVC
5. Pod deleted? → Data remains in PV
6. New pod? → Can mount same PVC
```

---

### PersistentVolume YAML

```yaml
# pv.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-data
spec:
  capacity:
    storage: 10Gi
  accessModes:
  - ReadWriteOnce        # RWO: Single node, RW
  # - ReadOnlyMany       # ROX: Multiple nodes, RO
  # - ReadWriteMany      # RWX: Multiple nodes, RW
  persistentVolumeReclaimPolicy: Retain  # Retain, Delete, Recycle
  storageClassName: standard
  hostPath:              # For local testing (don't use in production)
    path: /mnt/data

# Production examples:
---
# AWS EBS
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-ebs
spec:
  capacity:
    storage: 50Gi
  accessModes:
  - ReadWriteOnce
  awsElasticBlockStore:
    volumeID: vol-0123456789abcdef0
    fsType: ext4

---
# NFS
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-nfs
spec:
  capacity:
    storage: 100Gi
  accessModes:
  - ReadWriteMany
  nfs:
    server: nfs-server.example.com
    path: /exports/data
```

---

### PersistentVolumeClaim YAML

```yaml
# pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: data-claim
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: standard
```

**Using PVC in Pod:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp
spec:
  containers:
  - name: app
    image: myapp:latest
    volumeMounts:
    - name: data
      mountPath: /data
  volumes:
  - name: data
    persistentVolumeClaim:
      claimName: data-claim    # References PVC
```

---

### Storage Commands

```bash
# List PersistentVolumes
kubectl get pv

# List PersistentVolumeClaims
kubectl get pvc

# Describe PV
kubectl describe pv pv-data

# Describe PVC
kubectl describe pvc data-claim

# Create PV and PVC
kubectl apply -f pv.yaml
kubectl apply -f pvc.yaml

# Check binding
kubectl get pvc
# NAME         STATUS   VOLUME    CAPACITY   ACCESS MODES
# data-claim   Bound    pv-data   10Gi       RWO

# Delete PVC (check reclaimPolicy!)
kubectl delete pvc data-claim

# Delete PV
kubectl delete pv pv-data
```

---

## Storage Classes

### 📖 Simple Explanation

**StorageClass = Dynamic PV provisioning**

**Without StorageClass:**
```
1. Admin manually creates 100 PVs
2. User creates PVC
3. K8s binds PVC to existing PV
Problem: Admin overhead!
```

**With StorageClass:**
```
1. Admin creates StorageClass (once)
2. User creates PVC with storageClassName
3. K8s automatically provisions PV
4. PV created on-demand!
```

---

### StorageClass YAML

```yaml
# storageclass.yaml

# AWS EBS
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-ssd
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp3
  iops: "3000"
  throughput: "125"
volumeBindingMode: WaitForFirstConsumer  # Create PV when pod scheduled
reclaimPolicy: Delete

---
# GCP Persistent Disk
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-ssd
  replication-type: regional-pd

---
# Azure Disk
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: managed-premium
provisioner: kubernetes.io/azure-disk
parameters:
  storageaccounttype: Premium_LRS
  kind: Managed
```

**Using StorageClass:**
```yaml
# pvc-dynamic.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: dynamic-claim
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 50Gi
  storageClassName: fast-ssd  # Uses StorageClass
```

---

## Advanced Scheduling

### 📖 Simple Explanation

**Scheduling = Deciding which node runs which pod**

**Default Scheduler:**
- Considers resources (CPU, memory available)
- Spreads pods across nodes
- Respects node constraints

**Advanced Scheduling:**
- **Node Selector**: Simple key-value matching
- **Node Affinity**: Complex rules (required/preferred)
- **Taints & Tolerations**: Repel pods (unless tolerant)
- **Pod Affinity/Anti-Affinity**: Pod-to-pod rules

---

### Node Selector

**Simplest scheduling constraint**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: gpu-pod
spec:
  nodeSelector:
    gpu: "true"         # Only nodes with label gpu=true
  containers:
  - name: ml-app
    image: ml-training:latest
```

**Label nodes:**
```bash
# Add label to node
kubectl label nodes worker-1 gpu=true

# Remove label
kubectl label nodes worker-1 gpu-

# View node labels
kubectl get nodes --show-labels
```

---

### Node Affinity

**More flexible than nodeSelector**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp
spec:
  affinity:
    nodeAffinity:
      # REQUIRED: Pod MUST run on nodes matching this
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: zone
            operator: In
            values:
            - us-west-1a
            - us-west-1b
      # PREFERRED: Prefers nodes matching this (soft rule)
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        preference:
          matchExpressions:
          - key: instance-type
            operator: In
            values:
            - c5.xlarge
            - c5.2xlarge
  containers:
  - name: webapp
    image: webapp:latest
```

**Operators:**
- `In`: Label value in list
- `NotIn`: Label value not in list
- `Exists`: Label exists (any value)
- `DoesNotExist`: Label doesn't exist
- `Gt`: Greater than (numeric)
- `Lt`: Less than (numeric)

---

### Taints & Tolerations

**Taint = Repel pods (unless they tolerate it)**

**Use Cases:**
- Dedicate nodes for specific workloads
- Reserve nodes for system components
- Evict pods during maintenance

```bash
# Add taint to node
kubectl taint nodes worker-1 gpu=true:NoSchedule

# Taint effects:
# NoSchedule: Don't schedule new pods
# PreferNoSchedule: Try not to schedule
# NoExecute: Evict existing pods + don't schedule

# Remove taint
kubectl taint nodes worker-1 gpu=true:NoSchedule-

# View taints
kubectl describe node worker-1 | grep Taints
```

**Toleration (allow pod on tainted node):**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: gpu-pod
spec:
  tolerations:
  - key: "gpu"
    operator: "Equal"
    value: "true"
    effect: "NoSchedule"
  containers:
  - name: ml-app
    image: ml-training:latest
```

---

### Pod Affinity/Anti-Affinity

**Pod Affinity: Schedule near other pods**
**Pod Anti-Affinity: Schedule away from other pods**

```yaml
# Anti-affinity example (don't schedule 2 web pods on same node)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: webapp
  template:
    metadata:
      labels:
        app: webapp
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - webapp
            topologyKey: kubernetes.io/hostname  # Don't schedule on same node
      containers:
      - name: webapp
        image: webapp:latest
```

**Result:** 3 replicas spread across 3 different nodes (high availability)

---

## Custom Resource Definitions (CRDs)

### 📖 Simple Explanation

**CRD = Extend Kubernetes API**

**Built-in resources:**
- Pod, Deployment, Service, etc.

**Custom resources:**
- Database, Certificate, Application, etc.
- You define them!

**Example:**
```yaml
# Want to create a Database like this:
apiVersion: mydb.example.com/v1
kind: Database
metadata:
  name: users-db
spec:
  engine: postgresql
  version: "15"
  storage: 100Gi

# CRD makes this possible!
```

---

### CRD Example

```yaml
# crd.yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: databases.mydb.example.com
spec:
  group: mydb.example.com
  names:
    kind: Database
    plural: databases
    singular: database
    shortNames:
    - db
  scope: Namespaced
  versions:
  - name: v1
    served: true
    storage: true
    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            properties:
              engine:
                type: string
                enum: ["postgresql", "mysql", "mongodb"]
              version:
                type: string
              storage:
                type: string
            required:
            - engine
            - version
            - storage
```

**Create and use custom resource:**
```bash
# Install CRD
kubectl apply -f crd.yaml

# Now you can create Database resources!
cat <<EOF | kubectl apply -f -
apiVersion: mydb.example.com/v1
kind: Database
metadata:
  name: users-db
spec:
  engine: postgresql
  version: "15"
  storage: 100Gi
EOF

# List databases
kubectl get databases
kubectl get db        # shortName

# Describe
kubectl describe database users-db
```

**Note:** CRD alone doesn't do anything! You need a **Controller/Operator** to watch and act on custom resources.

---

## Operators

### 📖 Simple Explanation

**Operator = Automate complex applications**

**Operator = CRD + Controller (custom logic)**

**How it works:**
```
1. You create custom resource (e.g., Database)
2. Operator watches for Database resources
3. Operator creates Pods, Services, PVCs automatically
4. Operator handles backups, scaling, upgrades
5. Operator monitors and fixes issues
```

**Example: PostgreSQL Operator**
```yaml
# You create this simple resource
apiVersion: acid.zalan.do/v1
kind: postgresql
metadata:
  name: my-postgres
spec:
  teamId: "myteam"
  volume:
    size: 100Gi
  numberOfInstances: 3
  users:
    alice: []
  databases:
    mydb: alice
  postgresql:
    version: "15"

# Operator automatically creates:
# - StatefulSet (3 replicas)
# - Services (master, replicas)
# - PVCs (persistent storage)
# - Secrets (passwords)
# - Configures replication
# - Handles failover
# - Performs backups
```

---

### Popular Operators

**1. Prometheus Operator**
```bash
# Install
kubectl apply -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/main/bundle.yaml

# Create Prometheus instance
kubectl apply -f - <<EOF
apiVersion: monitoring.coreos.com/v1
kind: Prometheus
metadata:
  name: prometheus
spec:
  replicas: 2
  serviceAccountName: prometheus
  serviceMonitorSelector:
    matchLabels:
      team: frontend
EOF
```

**2. Cert-Manager (TLS certificates)**
```bash
# Install
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.13.0/cert-manager.yaml

# Create certificate
kubectl apply -f - <<EOF
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: example-com
spec:
  secretName: example-com-tls
  issuerRef:
    name: letsencrypt-prod
    kind: ClusterIssuer
  dnsNames:
  - example.com
  - www.example.com
EOF
```

**3. Istio Operator (Service Mesh)**
**4. ArgoCD Operator (GitOps)**
**5. Kafka Operator (Strimzi)**

---

## Real-World Scenario

### Deploy Stateful Application (PostgreSQL Cluster)

```yaml
# Complete StatefulSet with all best practices

# 1. Namespace
apiVersion: v1
kind: Namespace
metadata:
  name: database

---
# 2. StorageClass
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-ssd
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp3
volumeBindingMode: WaitForFirstConsumer

---
# 3. Headless Service
apiVersion: v1
kind: Service
metadata:
  name: postgres
  namespace: database
  labels:
    app: postgres
spec:
  ports:
  - port: 5432
    name: postgres
  clusterIP: None
  selector:
    app: postgres

---
# 4. Service (for clients)
apiVersion: v1
kind: Service
metadata:
  name: postgres-lb
  namespace: database
spec:
  type: LoadBalancer
  selector:
    app: postgres
    role: master
  ports:
  - port: 5432
    targetPort: 5432

---
# 5. Secret
apiVersion: v1
kind: Secret
metadata:
  name: postgres-secret
  namespace: database
type: Opaque
stringData:
  POSTGRES_PASSWORD: supersecretpassword
  REPLICATION_PASSWORD: replicationsecret

---
# 6. ConfigMap
apiVersion: v1
kind: ConfigMap
metadata:
  name: postgres-config
  namespace: database
data:
  postgresql.conf: |
    max_connections = 200
    shared_buffers = 256MB
    effective_cache_size = 1GB
    max_wal_size = 2GB

---
# 7. StatefulSet
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
  namespace: database
spec:
  serviceName: postgres
  replicas: 3
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchLabels:
                app: postgres
            topologyKey: kubernetes.io/hostname
      containers:
      - name: postgres
        image: postgres:15
        ports:
        - containerPort: 5432
          name: postgres
        env:
        - name: POSTGRES_PASSWORD
          valueFrom:
            secretKeyRef:
              name: postgres-secret
              key: POSTGRES_PASSWORD
        - name: POD_IP
          valueFrom:
            fieldRef:
              fieldPath: status.podIP
        volumeMounts:
        - name: data
          mountPath: /var/lib/postgresql/data
        - name: config
          mountPath: /etc/postgresql
        resources:
          requests:
            memory: "512Mi"
            cpu: "500m"
          limits:
            memory: "1Gi"
            cpu: "1000m"
        livenessProbe:
          exec:
            command:
            - /bin/sh
            - -c
            - pg_isready -U postgres
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          exec:
            command:
            - /bin/sh
            - -c
            - pg_isready -U postgres
          initialDelaySeconds: 5
          periodSeconds: 5
      volumes:
      - name: config
        configMap:
          name: postgres-config
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: fast-ssd
      resources:
        requests:
          storage: 100Gi

---
# 8. HPA (if using connection pooler)
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: pgbouncer-hpa
  namespace: database
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: pgbouncer
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70

---
# 9. CronJob for backups
apiVersion: batch/v1
kind: CronJob
metadata:
  name: postgres-backup
  namespace: database
spec:
  schedule: "0 2 * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: backup
            image: postgres:15
            command:
            - /bin/bash
            - -c
            - |
              pg_dumpall -h postgres-0.postgres -U postgres | \
              gzip > /backup/backup-$(date +\%Y\%m\%d).sql.gz
            env:
            - name: PGPASSWORD
              valueFrom:
                secretKeyRef:
                  name: postgres-secret
                  key: POSTGRES_PASSWORD
            volumeMounts:
            - name: backup
              mountPath: /backup
          volumes:
          - name: backup
            persistentVolumeClaim:
              claimName: backup-pvc
          restartPolicy: OnFailure
```

**Deploy:**
```bash
kubectl apply -f postgres-cluster.yaml

# Watch StatefulSet roll out
kubectl get pods -n database -w

# Connect to master
kubectl exec -it postgres-0 -n database -- psql -U postgres

# Test failover
kubectl delete pod postgres-0 -n database
# Kubernetes automatically recreates with same identity
```

---

## Chapter Summary

### What You Learned

**1. StatefulSets**
- ✅ Stable network identity (predictable DNS names)
- ✅ Ordered deployment/scaling
- ✅ Persistent storage per pod
- ✅ Use case: Databases, message queues

**2. DaemonSets**
- ✅ One pod per node
- ✅ Use case: Monitoring agents, log collectors

**3. Jobs & CronJobs**
- ✅ Run-to-completion workloads
- ✅ Scheduled tasks
- ✅ Use case: Batch processing, backups

**4. Auto-scaling**
- ✅ HPA: Scale pods based on metrics
- ✅ VPA: Adjust resource requests
- ✅ Cost optimization + performance

**5. Storage**
- ✅ PersistentVolumes & Claims
- ✅ StorageClasses (dynamic provisioning)
- ✅ Data persistence

**6. Advanced Scheduling**
- ✅ Node affinity/selector
- ✅ Taints & tolerations
- ✅ Pod affinity/anti-affinity
- ✅ Control pod placement

**7. CRDs & Operators**
- ✅ Extend Kubernetes API
- ✅ Automate complex applications
- ✅ Codify operational knowledge

---

### 🎯 Interview Quick Tips

**Q1: When to use StatefulSet vs Deployment?**
```
Deployment: Stateless apps (any pod can handle any request)
- Web servers, APIs, frontend

StatefulSet: Stateful apps (pods need unique identity)
- Databases, ZooKeeper, Kafka
- Need stable DNS names
- Need persistent storage per pod
```

**Q2: How does HPA work?**
```
1. Monitors metrics (CPU, memory, custom)
2. Calculates desired replicas based on target
3. Scales deployment up/down
4. Respects min/max limits
5. Has cooldown period to prevent flapping

Requires: Metrics Server installed
```

**Q3: Explain taints and tolerations**
```
Taint: Repels pods from node (unless tolerant)
Toleration: Allows pod to schedule on tainted node

Example:
Node: Tainted with gpu=true:NoSchedule
Pod without toleration: Won't schedule
Pod with toleration: Can schedule

Use case: Dedicate GPU nodes for ML workloads
```

**Q4: What are Operators?**
```
Operator = CRD + Controller

Automates Day-2 operations:
- Installation
- Upgrades
- Backups
- Scaling
- Failure recovery

Example: PostgreSQL Operator
- You: Create Database resource
- Operator: Creates StatefulSet, Services, handles replication
```

**Q5: StorageClass vs PersistentVolume?**
```
PersistentVolume: Pre-created storage (manual)
StorageClass: Dynamic provisioning (automatic)

Production: Use StorageClass
- Auto-creates PV on-demand
- No admin overhead
- Cloud-native (EBS, GCE PD, Azure Disk)
```

---

### 💡 Best Practices

**Production Checklist:**
- ✅ Use StatefulSets for databases
- ✅ Set pod anti-affinity for high availability
- ✅ Configure HPA with proper targets
- ✅ Use StorageClasses (not manual PVs)
- ✅ Set resource requests/limits
- ✅ Add health checks (liveness, readiness)
- ✅ Use CronJobs for backups
- ✅ Taint GPU nodes
- ✅ Consider Operators for complex apps
- ✅ Test StatefulSet failure scenarios

---

### 📚 What's Next?

Now that you've mastered Kubernetes, you're ready for:
- **Cloud Platforms** (EKS, AKS, GKE managed K8s)
- **Infrastructure as Code** (Terraform to provision K8s)
- **CI/CD** (Deploy to K8s automatically)
- **Monitoring** (Prometheus Operator on K8s)

**Practice Suggestions:**
1. Deploy PostgreSQL with StatefulSet
2. Create HPA, generate load, watch it scale
3. Use taints to dedicate a node
4. Install an Operator (Prometheus, Cert-Manager)
5. Create a CronJob for daily backups

---

**Next Chapter:** [Cloud Platforms (AWS/Azure/GCP)](05-cloud-platforms.md)

**Kubernetes Mastery:** ✅ Complete! You're now a K8s expert!

