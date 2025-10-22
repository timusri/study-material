# 3. Kubernetes Fundamentals

## 📚 Quick Summary

Kubernetes (K8s) is THE container orchestration platform - 96% of organizations use or are evaluating it!

**What You'll Learn:**
- **K8s Architecture**: Control plane, worker nodes, components
- **Core Objects**: Pods, Deployments, Services, ConfigMaps, Secrets
- **kubectl Mastery**: Essential commands for daily work
- **Workload Management**: Deploy, scale, update applications
- **Service Discovery**: How containers find each other
- **Configuration**: External config without rebuilding images
- **Troubleshooting**: Debug pods, check logs, fix issues

**Why This Matters:**
- #1 skill in DevOps job postings
- Every major cloud platform offers K8s (EKS, AKS, GKE)
- Modern apps = microservices on Kubernetes
- Salary premium: K8s skills add $20k-40k to offers
- Interview weight: 25-30% of technical questions

**Interview Reality:**
"Deploy this application on Kubernetes" = Standard technical round!

---

## 📖 Simple Explanation

**What is Kubernetes?**
Kubernetes is like an **orchestra conductor** for containers:
- **Conductor** (K8s) tells **musicians** (containers) what to play
- If a musician stops (container crashes), conductor brings in replacement
- Conductor ensures harmony (load balancing, networking)
- Scales orchestra up/down based on audience (traffic)

**Why Kubernetes?**
```
Without Kubernetes:
- 100 containers across 10 servers
- Container crashes? Manual restart
- Need more capacity? Manually deploy
- Load balancing? Configure manually
- Networking? Complex setup
→ Time-consuming, error-prone

With Kubernetes:
- Declare: "I want 100 containers"
- K8s: Deploys, monitors, restarts, balances, scales
- Self-healing, automatic, declarative
→ Fast, reliable, scalable
```

**Real-World Analogy:**
```
Traditional Deployment:
You: "Server 1, run app v1.0"
You: "Server 2, run app v1.0"
You: "Server 3, run app v1.0"
... (repeat 100 times)
Server 2 crashes → You manually fix

Kubernetes:
You: "K8s, I want 100 app v1.0 instances"
K8s: ✅ Deployed
Pod crashes → K8s automatically restarts
Need 200 instances? → K8s scales automatically
```

---

## Table of Contents
- [Kubernetes Architecture](#kubernetes-architecture)
- [Core Concepts](#core-concepts)
- [Pods](#pods)
- [Deployments](#deployments)
- [Services](#services)
- [ConfigMaps & Secrets](#configmaps--secrets)
- [Namespaces](#namespaces)
- [kubectl Commands](#kubectl-commands)
- [Troubleshooting](#troubleshooting)

---

## Kubernetes Architecture

### 📖 Simple Explanation

**Kubernetes Cluster = Control Plane + Worker Nodes**

```
┌─────────────────────────────────────────────────┐
│              Control Plane                      │
│  (The Brain - Makes Decisions)                  │
│                                                 │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐     │
│  │   API    │  │ Scheduler │  │Controller│     │
│  │  Server  │  │           │  │ Manager  │     │
│  └──────────┘  └──────────┘  └──────────┘     │
│  ┌──────────┐                                  │
│  │  etcd    │  (Database)                      │
│  └──────────┘                                  │
└─────────────────────────────────────────────────┘
              ↓
┌───────────────┬─────────────────┬──────────────┐
│  Worker Node 1│  Worker Node 2  │ Worker Node 3│
│  (The Muscle) │  (Does the Work)│              │
│               │                 │              │
│  ┌─────────┐  │  ┌─────────┐   │ ┌─────────┐  │
│  │  Pod    │  │  │  Pod    │   │ │  Pod    │  │
│  │ (App 1) │  │  │ (App 2) │   │ │ (App 3) │  │
│  └─────────┘  │  └─────────┘   │ └─────────┘  │
│               │                 │              │
│  ┌─────────┐  │  ┌─────────┐   │ ┌─────────┐  │
│  │  Pod    │  │  │  Pod    │   │ │  Pod    │  │
│  │ (App 1) │  │  │ (App 2) │   │ │ (App 3) │  │
│  └─────────┘  │  └─────────┘   │ └─────────┘  │
└───────────────┴─────────────────┴──────────────┘
```

---

### Control Plane Components

**1. API Server (kube-apiserver)**
- Front door to Kubernetes
- All communication goes through here
- RESTful API
- kubectl talks to API server

**2. etcd**
- Distributed key-value store
- Database for entire cluster state
- Stores all configuration, state
- Highly available

**3. Scheduler (kube-scheduler)**
- Decides which node runs which pod
- Considers: resources, constraints, affinity
- Doesn't actually start pods (kubelet does)

**4. Controller Manager**
- Runs controller processes
- Ensures desired state = actual state
- Examples: ReplicaSet, Deployment, Node controllers

**5. Cloud Controller Manager**
- Interacts with cloud provider (AWS, GCP, Azure)
- Creates load balancers, volumes
- Cloud-specific logic

---

### Worker Node Components

**1. kubelet**
- Agent running on each node
- Ensures containers are running in pods
- Talks to API server
- Starts/stops containers

**2. kube-proxy**
- Network proxy on each node
- Implements Service networking
- Routes traffic to correct pods
- Load balancing

**3. Container Runtime**
- Actually runs containers
- Docker, containerd, CRI-O
- Pulled images, starts containers

---

### How Kubernetes Works

**Example: Deploy nginx**
```
1. kubectl apply -f deployment.yaml
   ↓
2. API Server receives request
   ↓
3. API Server stores in etcd
   ↓
4. Scheduler notices new pods needed
   ↓
5. Scheduler assigns pods to nodes
   ↓
6. kubelet on node sees assignment
   ↓
7. kubelet tells container runtime to start container
   ↓
8. Container starts, runs nginx
   ↓
9. Controller monitors, ensures always running
```

---

## Core Concepts

### 📖 Simple Explanation

**Kubernetes Objects = Building Blocks**

Think of them like LEGO pieces:
- **Pod**: Container of containers (1+ containers)
- **Deployment**: Manages pods, handles updates
- **Service**: Exposes pods to network
- **ConfigMap**: Configuration data
- **Secret**: Sensitive data (passwords)
- **Namespace**: Virtual cluster (isolation)

---

## Pods

### 📖 Simple Explanation

**Pod = Smallest deployable unit in Kubernetes**

**Key Points:**
- Pod wraps one or more containers
- Containers in a pod share network/storage
- Usually 1 container per pod
- Ephemeral (dies, K8s creates new one)

```
Pod = Wrapper
┌─────────────────────────┐
│         Pod             │
│  ┌──────────────────┐   │
│  │   Container      │   │  (Usually just one)
│  │   (nginx)        │   │
│  └──────────────────┘   │
│                         │
│  IP: 10.244.1.5         │
│  Storage: /data         │
└─────────────────────────┘
```

---

### Pod YAML

```yaml
# pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.25
    ports:
    - containerPort: 80
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
```

**Apply it:**
```bash
kubectl apply -f pod.yaml
```

---

### Pod Commands

```bash
# Create pod
kubectl run nginx --image=nginx:1.25

# List pods
kubectl get pods
kubectl get pods -o wide          # more details (IP, node)
kubectl get pods --all-namespaces # all namespaces
kubectl get pods -w               # watch mode (real-time)

# Describe pod (detailed info)
kubectl describe pod nginx

# View logs
kubectl logs nginx
kubectl logs -f nginx             # follow (real-time)
kubectl logs --tail=100 nginx     # last 100 lines
kubectl logs nginx -c container-name  # specific container (multi-container pod)

# Execute command in pod
kubectl exec nginx -- ls /usr/share/nginx/html
kubectl exec -it nginx -- bash    # interactive shell

# Delete pod
kubectl delete pod nginx
kubectl delete -f pod.yaml
```

**Real Example:**
```bash
# Create pod
$ kubectl run nginx --image=nginx:1.25
pod/nginx created

# Check status
$ kubectl get pods
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          10s

# Get more details
$ kubectl get pods -o wide
NAME    READY   STATUS    IP           NODE
nginx   1/1     Running   10.244.1.5   worker-1

# View logs
$ kubectl logs nginx
...
2024/10/22 10:30:00 [notice] 1#1: start worker processes

# Access pod
$ kubectl exec -it nginx -- bash
root@nginx:/# curl localhost
<!DOCTYPE html>
<html>
<head><title>Welcome to nginx!</title></head>
...
root@nginx:/# exit

# Delete pod
$ kubectl delete pod nginx
pod "nginx" deleted
```

---

## Deployments

### 📖 Simple Explanation

**Deployment = Manages Pods + Updates**

**Why not just Pods?**
- Pods are ephemeral (die, gone forever)
- Deployments ensure desired number always running
- Handle updates (rolling, rollback)
- Scaling made easy

```
Deployment (I want 3 nginx pods)
    ↓
ReplicaSet (Ensures 3 replicas exist)
    ↓
Pods (3 nginx pods)

Pod dies? ReplicaSet creates new one
Update image? Deployment handles rolling update
Scale to 10? Deployment creates 7 more
```

---

### Deployment YAML

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3                    # Desired number of pods
  selector:
    matchLabels:
      app: nginx                 # Which pods to manage
  template:                      # Pod template
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.25
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "128Mi"
            cpu: "500m"
        livenessProbe:           # Health check
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:          # Ready to serve traffic?
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
```

---

### Deployment Commands

```bash
# Create deployment
kubectl create deployment nginx --image=nginx:1.25 --replicas=3
# Or
kubectl apply -f deployment.yaml

# List deployments
kubectl get deployments
kubectl get deploy            # shorthand

# Describe deployment
kubectl describe deployment nginx-deployment

# Scale deployment
kubectl scale deployment nginx-deployment --replicas=5

# Update image (rolling update)
kubectl set image deployment/nginx-deployment nginx=nginx:1.26

# Check rollout status
kubectl rollout status deployment/nginx-deployment

# View rollout history
kubectl rollout history deployment/nginx-deployment

# Rollback to previous version
kubectl rollout undo deployment/nginx-deployment

# Rollback to specific revision
kubectl rollout undo deployment/nginx-deployment --to-revision=2

# Pause/Resume rollout
kubectl rollout pause deployment/nginx-deployment
kubectl rollout resume deployment/nginx-deployment

# Delete deployment
kubectl delete deployment nginx-deployment
```

**Real Example: Rolling Update**
```bash
# Create deployment
$ kubectl create deployment webapp --image=myapp:v1.0 --replicas=3
deployment.apps/webapp created

# Check pods
$ kubectl get pods
NAME                      READY   STATUS    RESTARTS   AGE
webapp-7d4f8c9b5f-4xkwl   1/1     Running   0          10s
webapp-7d4f8c9b5f-8m2nq   1/1     Running   0          10s
webapp-7d4f8c9b5f-p7xtj   1/1     Running   0          10s

# Update to v2.0
$ kubectl set image deployment/webapp webapp=myapp:v2.0
deployment.apps/webapp image updated

# Watch rollout
$ kubectl rollout status deployment/webapp
Waiting for deployment "webapp" rollout to finish: 1 out of 3 new replicas...
Waiting for deployment "webapp" rollout to finish: 2 out of 3 new replicas...
deployment "webapp" successfully rolled out

# Check new pods (new ReplicaSet created)
$ kubectl get pods
NAME                      READY   STATUS    RESTARTS   AGE
webapp-8f9c7d6e4g-5ykwm   1/1     Running   0          30s
webapp-8f9c7d6e4g-7mxnp   1/1     Running   0          25s
webapp-8f9c7d6e4g-9pzqr   1/1     Running   0          20s

# Something wrong? Rollback!
$ kubectl rollout undo deployment/webapp
deployment.apps/webapp rolled back

# Verify
$ kubectl rollout history deployment/webapp
REVISION  CHANGE-CAUSE
2         kubectl set image deployment/webapp webapp=myapp:v2.0
3         kubectl rollout undo deployment/webapp
```

---

## Services

### 📖 Simple Explanation

**Service = Stable endpoint for pods**

**Problem:**
- Pods get IPs (10.244.1.5)
- Pod dies → New pod, new IP
- How do other pods find it?

**Solution: Service**
- Stable IP/DNS name
- Load balances across pods
- Automatic service discovery

```
Service (Stable IP: 10.96.0.10)
    ↓ Load Balances
┌─────────┬─────────┬─────────┐
│  Pod 1  │  Pod 2  │  Pod 3  │
│ (10.2.1)│ (10.2.2)│ (10.2.3)│
└─────────┴─────────┴─────────┘

Client connects to Service (10.96.0.10)
Service routes to any healthy pod
```

---

### Service Types

**1. ClusterIP (Default)**
- Internal only
- Other pods can access
- Not accessible from outside cluster

**2. NodePort**
- Exposes on each node's IP
- Port range: 30000-32767
- Accessible from outside: `<NodeIP>:<NodePort>`

**3. LoadBalancer**
- Cloud provider creates external load balancer
- Public IP address
- Most common for production

**4. ExternalName**
- Maps to external DNS name
- No proxying involved

---

### Service YAML

```yaml
# service-clusterip.yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: ClusterIP              # Internal only
  selector:
    app: nginx                 # Route to pods with this label
  ports:
  - port: 80                   # Service port
    targetPort: 80             # Container port
    protocol: TCP
```

```yaml
# service-nodeport.yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-nodeport
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30080            # Accessible on any node IP:30080
```

```yaml
# service-loadbalancer.yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-lb
spec:
  type: LoadBalancer
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
```

---

### Service Commands

```bash
# Create service
kubectl expose deployment nginx --port=80 --type=ClusterIP

# List services
kubectl get services
kubectl get svc              # shorthand

# Describe service
kubectl describe service nginx-service

# Get service endpoints (pods behind service)
kubectl get endpoints nginx-service

# Delete service
kubectl delete service nginx-service
```

**Real Example:**
```bash
# Create deployment
$ kubectl create deployment web --image=nginx:1.25 --replicas=3

# Expose as ClusterIP service
$ kubectl expose deployment web --port=80 --type=ClusterIP
service/web exposed

# Check service
$ kubectl get svc
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)
web          ClusterIP   10.96.45.123    <none>        80/TCP

# Check endpoints (pods)
$ kubectl get endpoints web
NAME   ENDPOINTS                                   AGE
web    10.244.1.5:80,10.244.1.6:80,10.244.1.7:80   1m

# Test from another pod
$ kubectl run test --image=busybox -it --rm -- wget -qO- http://web
<!DOCTYPE html>
<html>
<head><title>Welcome to nginx!</title></head>
...

# Expose as LoadBalancer (cloud only)
$ kubectl expose deployment web --port=80 --type=LoadBalancer --name=web-lb
service/web-lb exposed

# Check external IP (takes a minute)
$ kubectl get svc web-lb
NAME     TYPE           CLUSTER-IP     EXTERNAL-IP      PORT(S)
web-lb   LoadBalancer   10.96.45.124   203.0.113.42     80:31234/TCP

# Access from internet
$ curl http://203.0.113.42
<!DOCTYPE html>...
```

---

## ConfigMaps & Secrets

### 📖 Simple Explanation

**ConfigMap = Configuration data (non-sensitive)**
**Secret = Sensitive data (passwords, tokens)**

**Why?**
- Decouple config from container image
- Change config without rebuilding image
- One image → multiple environments (dev, staging, prod)

```
Without ConfigMap:
- Build image with hardcoded config
- Different image per environment
- Change config → rebuild image

With ConfigMap:
- One image
- Pass config at deployment
- Change config → restart pods (no rebuild)
```

---

### ConfigMap Examples

```bash
# Create ConfigMap from literal
kubectl create configmap app-config \
  --from-literal=DATABASE_URL=postgres://db:5432/mydb \
  --from-literal=REDIS_URL=redis://redis:6379

# Create from file
kubectl create configmap nginx-config --from-file=nginx.conf

# Create from YAML
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  DATABASE_URL: "postgres://db:5432/mydb"
  REDIS_URL: "redis://redis:6379"
  LOG_LEVEL: "info"
  config.json: |
    {
      "server": {
        "port": 8080,
        "host": "0.0.0.0"
      }
    }
EOF
```

**Use ConfigMap in Pod:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp
spec:
  containers:
  - name: app
    image: myapp:latest
    # Environment variables from ConfigMap
    env:
    - name: DATABASE_URL
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: DATABASE_URL
    - name: LOG_LEVEL
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: LOG_LEVEL
    # Or mount entire ConfigMap as volume
    volumeMounts:
    - name: config-volume
      mountPath: /etc/config
  volumes:
  - name: config-volume
    configMap:
      name: app-config
```

---

### Secret Examples

```bash
# Create secret from literal
kubectl create secret generic db-secret \
  --from-literal=username=admin \
  --from-literal=password=secretpass

# Create from file
kubectl create secret generic tls-secret \
  --from-file=tls.crt=server.crt \
  --from-file=tls.key=server.key

# Create TLS secret
kubectl create secret tls tls-secret \
  --cert=server.crt \
  --key=server.key

# Create from YAML (base64 encoded)
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  username: YWRtaW4=       # base64 encoded "admin"
  password: c2VjcmV0cGFzcw==  # base64 encoded "secretpass"
EOF

# Encode/decode base64
echo -n "admin" | base64        # YWRtaW4=
echo "YWRtaW4=" | base64 -d     # admin
```

**Use Secret in Pod:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp
spec:
  containers:
  - name: app
    image: myapp:latest
    # Environment variables from Secret
    env:
    - name: DB_USERNAME
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: username
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: password
    # Or mount as volume
    volumeMounts:
    - name: secret-volume
      mountPath: /etc/secrets
      readOnly: true
  volumes:
  - name: secret-volume
    secret:
      secretName: db-secret
```

---

## Namespaces

### 📖 Simple Explanation

**Namespace = Virtual cluster**

**Why Namespaces?**
- Organize resources
- Multi-tenancy (multiple teams/projects)
- Resource isolation
- Access control

```
Cluster
├── default (default namespace)
├── dev (development environment)
├── staging (staging environment)
├── production (production environment)
└── kube-system (K8s system components)
```

---

### Namespace Commands

```bash
# List namespaces
kubectl get namespaces
kubectl get ns               # shorthand

# Create namespace
kubectl create namespace dev

# View resources in namespace
kubectl get pods -n dev
kubectl get all -n dev       # all resources

# Set default namespace (for kubectl)
kubectl config set-context --current --namespace=dev

# Delete namespace (deletes all resources in it!)
kubectl delete namespace dev
```

**Namespace YAML:**
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: production
  labels:
    name: production
---
# Resource quota for namespace
apiVersion: v1
kind: ResourceQuota
metadata:
  name: prod-quota
  namespace: production
spec:
  hard:
    requests.cpu: "100"
    requests.memory: "100Gi"
    pods: "100"
```

---

## kubectl Commands

### 📖 Simple Explanation

**kubectl = Kubernetes command-line tool**

**Command Structure:**
```bash
kubectl [command] [TYPE] [NAME] [flags]

kubectl get pods nginx -o yaml
│       │   │    │     └─ output format
│       │   │    └─ resource name
│       │   └─ resource type
│       └─ action (get, create, delete, etc.)
└─ kubectl
```

---

### Essential kubectl Commands

```bash
# Cluster info
kubectl cluster-info
kubectl get nodes
kubectl version

# Create resources
kubectl apply -f file.yaml          # create/update
kubectl create -f file.yaml         # create only
kubectl run nginx --image=nginx     # create pod
kubectl expose deployment nginx --port=80  # create service

# View resources
kubectl get pods
kubectl get deployments
kubectl get services
kubectl get all                     # all resources
kubectl get pods -o wide            # more details
kubectl get pods -o yaml            # YAML output
kubectl get pods --show-labels      # show labels

# Describe (detailed info)
kubectl describe pod nginx
kubectl describe node worker-1

# Logs
kubectl logs pod-name
kubectl logs -f pod-name            # follow
kubectl logs pod-name -c container  # specific container

# Execute commands
kubectl exec pod-name -- ls /app
kubectl exec -it pod-name -- bash   # interactive shell

# Port forwarding
kubectl port-forward pod-name 8080:80  # localhost:8080 → pod:80

# Copy files
kubectl cp pod-name:/path/file ./file  # from pod
kubectl cp ./file pod-name:/path/file  # to pod

# Edit resources
kubectl edit deployment nginx       # opens in editor
kubectl patch deployment nginx -p '{"spec":{"replicas":5}}'

# Delete resources
kubectl delete pod nginx
kubectl delete -f file.yaml
kubectl delete pods --all           # all pods
kubectl delete all --all            # all resources (careful!)

# Scale
kubectl scale deployment nginx --replicas=5

# Rollout management
kubectl rollout status deployment/nginx
kubectl rollout history deployment/nginx
kubectl rollout undo deployment/nginx

# Labels and selectors
kubectl label pods nginx env=prod    # add label
kubectl get pods -l env=prod        # select by label

# Debugging
kubectl top nodes                   # node resource usage
kubectl top pods                    # pod resource usage
kubectl get events                  # cluster events
kubectl get events --sort-by=.metadata.creationTimestamp

# Config and context
kubectl config view                 # view config
kubectl config get-contexts         # list contexts
kubectl config use-context cluster-name  # switch context
kubectl config set-context --current --namespace=dev

# Apply multiple files
kubectl apply -f directory/
kubectl apply -k kustomization/     # kustomize
```

---

### kubectl Shortcuts

```bash
# Resource type shortcuts
po     = pods
svc    = services
deploy = deployments
rs     = replicasets
ns     = namespaces
cm     = configmaps
pv     = persistentvolumes
pvc    = persistentvolumeclaims

# Examples
kubectl get po          # instead of kubectl get pods
kubectl get svc         # instead of kubectl get services
kubectl describe deploy nginx  # instead of deployment
```

---

### kubectl Output Formats

```bash
# Wide output (more columns)
kubectl get pods -o wide

# YAML format
kubectl get pod nginx -o yaml

# JSON format
kubectl get pod nginx -o json

# JSONPath (extract specific fields)
kubectl get pods -o jsonpath='{.items[*].metadata.name}'
kubectl get pods -o jsonpath='{.items[*].status.podIP}'

# Custom columns
kubectl get pods -o custom-columns=NAME:.metadata.name,IP:.status.podIP

# No headers
kubectl get pods --no-headers

# Watch mode (real-time updates)
kubectl get pods -w
```

---

## Troubleshooting

### 📖 Simple Explanation

**Common Issues & Solutions:**

**1. Pod won't start**
**2. CrashLoopBackOff**
**3. ImagePullBackOff**
**4. Can't connect to service**

---

### Troubleshooting Workflow

```bash
# 1. Check pod status
kubectl get pods

# 2. Describe pod (events section is key!)
kubectl describe pod problem-pod

# 3. Check logs
kubectl logs problem-pod
kubectl logs problem-pod --previous  # previous container logs

# 4. Execute commands in pod
kubectl exec -it problem-pod -- sh

# 5. Check events
kubectl get events --sort-by=.metadata.creationTimestamp

# 6. Check node status
kubectl get nodes
kubectl describe node worker-1
```

---

### Common Issues

**Issue 1: ImagePullBackOff**
```bash
# Symptom
$ kubectl get pods
NAME    READY   STATUS             RESTARTS   AGE
myapp   0/1     ImagePullBackOff   0          2m

# Diagnose
$ kubectl describe pod myapp
Events:
  Warning  Failed     Failed to pull image "myapp:v1.0": not found

# Causes
1. Image doesn't exist
2. Wrong image name/tag
3. Private registry (no credentials)

# Solution
# Fix image name
kubectl set image deployment/myapp myapp=myapp:latest

# Or add image pull secret
kubectl create secret docker-registry regcred \
  --docker-server=myregistry.com \
  --docker-username=user \
  --docker-password=pass

# Use in deployment
spec:
  imagePullSecrets:
  - name: regcred
```

**Issue 2: CrashLoopBackOff**
```bash
# Symptom
$ kubectl get pods
NAME    READY   STATUS             RESTARTS   AGE
myapp   0/1     CrashLoopBackOff   5          5m

# Diagnose
$ kubectl logs myapp
Error: Cannot connect to database
Database connection failed

# Check previous logs
$ kubectl logs myapp --previous

# Causes
1. Application error
2. Missing dependencies
3. Wrong configuration

# Solution
# Check ConfigMap/Secret
kubectl get configmap app-config -o yaml
kubectl get secret db-secret -o yaml

# Update configuration
kubectl edit configmap app-config

# Restart pods
kubectl rollout restart deployment/myapp
```

**Issue 3: Pending Pod**
```bash
# Symptom
$ kubectl get pods
NAME    READY   STATUS    RESTARTS   AGE
myapp   0/1     Pending   0          5m

# Diagnose
$ kubectl describe pod myapp
Events:
  Warning  FailedScheduling  0/3 nodes available: insufficient memory

# Causes
1. Not enough resources
2. Node selector doesn't match
3. Taints on nodes

# Solution
# Check node resources
kubectl top nodes

# Scale down other deployments
kubectl scale deployment other-app --replicas=0

# Or reduce resource requests
kubectl edit deployment myapp
# Change requests.memory from 2Gi to 512Mi
```

**Issue 4: Can't Connect to Service**
```bash
# Check service exists
$ kubectl get svc myapp
NAME    TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)
myapp   ClusterIP   10.96.123.45    <none>        80/TCP

# Check endpoints (are there pods?)
$ kubectl get endpoints myapp
NAME    ENDPOINTS                                   AGE
myapp   10.244.1.5:80,10.244.1.6:80,10.244.1.7:80   10m

# If no endpoints, check pod labels
$ kubectl get pods --show-labels
$ kubectl describe service myapp

# Test connection from another pod
$ kubectl run test --image=busybox -it --rm -- wget -qO- http://myapp
```

---

## Real-World Scenario

### Deploy Complete Application

**Scenario:** Deploy a web app with database

```yaml
# 1. Namespace
apiVersion: v1
kind: Namespace
metadata:
  name: myapp

---
# 2. Database Secret
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
  namespace: myapp
type: Opaque
stringData:
  username: postgres
  password: supersecret

---
# 3. PostgreSQL Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres
  namespace: myapp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres:15
        env:
        - name: POSTGRES_USER
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: username
        - name: POSTGRES_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: password
        - name: POSTGRES_DB
          value: myapp
        ports:
        - containerPort: 5432
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"

---
# 4. PostgreSQL Service
apiVersion: v1
kind: Service
metadata:
  name: postgres
  namespace: myapp
spec:
  selector:
    app: postgres
  ports:
  - port: 5432
    targetPort: 5432

---
# 5. Web App ConfigMap
apiVersion: v1
kind: ConfigMap
metadata:
  name: webapp-config
  namespace: myapp
data:
  DATABASE_URL: "postgresql://postgres:5432/myapp"

---
# 6. Web App Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
  namespace: myapp
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
      containers:
      - name: webapp
        image: mywebapp:v1.0
        env:
        - name: DATABASE_URL
          valueFrom:
            configMapKeyRef:
              name: webapp-config
              key: DATABASE_URL
        - name: DB_USER
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: username
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: password
        ports:
        - containerPort: 8080
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5
        resources:
          requests:
            memory: "128Mi"
            cpu: "250m"
          limits:
            memory: "256Mi"
            cpu: "500m"

---
# 7. Web App Service
apiVersion: v1
kind: Service
metadata:
  name: webapp
  namespace: myapp
spec:
  type: LoadBalancer
  selector:
    app: webapp
  ports:
  - port: 80
    targetPort: 8080
```

**Deploy:**
```bash
# Apply all resources
kubectl apply -f application.yaml

# Check deployment
kubectl get all -n myapp

# Check pods are running
kubectl get pods -n myapp -w

# Get service external IP
kubectl get svc webapp -n myapp

# Test application
curl http://<EXTERNAL-IP>

# Check logs
kubectl logs -f deployment/webapp -n myapp

# Scale if needed
kubectl scale deployment webapp --replicas=5 -n myapp
```

---

## Chapter Summary

### What You Learned

**1. Kubernetes Architecture**
- ✅ Control plane (API server, etcd, scheduler, controllers)
- ✅ Worker nodes (kubelet, kube-proxy, container runtime)
- ✅ How K8s makes decisions and runs containers

**2. Core Objects**
- ✅ Pods (smallest unit, 1+ containers)
- ✅ Deployments (manage pods, rolling updates)
- ✅ Services (stable endpoints, load balancing)
- ✅ ConfigMaps (configuration data)
- ✅ Secrets (sensitive data)
- ✅ Namespaces (virtual clusters)

**3. kubectl Mastery**
- ✅ 50+ essential commands
- ✅ Create, read, update, delete resources
- ✅ Debugging and troubleshooting
- ✅ Logs, exec, port-forward

**4. Practical Skills**
- ✅ Deploy applications
- ✅ Scale workloads
- ✅ Rolling updates and rollbacks
- ✅ Troubleshoot common issues

---

### 🎯 Interview Quick Tips

**Most Asked Questions:**

**Q1: What is Kubernetes?**
"Container orchestration platform that automates deployment, scaling, and management of containerized applications. Provides self-healing, load balancing, and declarative configuration."

**Q2: Explain Pod vs Deployment**
"Pod is smallest unit, wraps containers. Ephemeral. Deployment manages pods, ensures desired number running, handles updates and rollbacks. Always use Deployments, not raw Pods."

**Q3: What are Services?**
"Stable network endpoint for pods. Provides load balancing and service discovery. Types: ClusterIP (internal), NodePort (node IP), LoadBalancer (cloud LB)."

**Q4: ConfigMap vs Secret?**
"Both store configuration. ConfigMap for non-sensitive (URLs, flags). Secret for sensitive (passwords, keys). Secrets are base64 encoded, access-controlled."

**Q5: How does rolling update work?**
"Deployment creates new ReplicaSet with updated image, gradually scales up new pods while scaling down old. Ensures zero downtime. Can rollback if issues."

---

### 💡 Best Practices

**Production Checklist:**
- ✅ Always use Deployments (not Pods)
- ✅ Set resource requests/limits
- ✅ Add liveness and readiness probes
- ✅ Use namespaces for isolation
- ✅ Store config in ConfigMaps/Secrets
- ✅ Use labels for organization
- ✅ Define appropriate replica count
- ✅ Implement health checks
- ✅ Use services for networking
- ✅ Version your YAML files (Git)

**Common Mistakes to Avoid:**
- ❌ Running as root in containers
- ❌ No resource limits (can crash node)
- ❌ No health checks (K8s can't detect failures)
- ❌ Hardcoded config in images
- ❌ Using latest tag (unpredictable)
- ❌ No labels (hard to manage)
- ❌ Single replica in production
- ❌ Exposing unnecessary ports

---

### 📚 What's Next?

Now that you know Kubernetes basics, you're ready for:
- **Kubernetes Advanced** (StatefulSets, CRDs, Operators)
- **CI/CD** (Automate deployments to K8s)
- **Monitoring** (Prometheus + Grafana on K8s)
- **Cloud Platforms** (EKS, AKS, GKE managed K8s)

**Practice Suggestions:**
1. Set up local K8s (Minikube/Kind)
2. Deploy sample applications
3. Practice kubectl commands daily
4. Break things intentionally, fix them
5. Deploy full-stack app (web + database)

---

### 📝 Practice Exercises

1. **Deploy nginx with 3 replicas, expose via LoadBalancer**
2. **Create ConfigMap with app config, use in deployment**
3. **Deploy PostgreSQL with Secret for password**
4. **Perform rolling update, then rollback**
5. **Debug a CrashLoopBackOff pod**
6. **Scale deployment from 3 to 10 replicas**
7. **Create multi-tier app (frontend, backend, database)**
8. **Use namespaces to separate dev and prod**

---

**Next Chapter:** [Kubernetes Advanced](04-kubernetes-advanced.md)

**Prerequisites Covered:** ✅ Ready for advanced K8s topics!

