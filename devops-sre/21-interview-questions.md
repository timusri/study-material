# 21. DevOps/SRE Interview Questions

## 📚 Comprehensive Interview Guide

This file contains real interview questions asked at top tech companies (FAANG, unicorns, startups) for DevOps and SRE roles.

---

## How to Use This Guide

1. **Read the question** - Don't peek at the answer
2. **Think through your answer** - Speak it out loud
3. **Read the provided answer** - Compare with yours
4. **Practice explaining** - Teach someone or record yourself

---

## Question Categories

- [Linux & Shell](#linux--shell)
- [Docker & Containers](#docker--containers)
- [Kubernetes](#kubernetes)
- [CI/CD](#cicd)
- [Infrastructure as Code](#infrastructure-as-code)
- [Monitoring & Logging](#monitoring--logging)
- [Networking](#networking)
- [Security](#security)
- [SRE Concepts](#sre-concepts)
- [Troubleshooting Scenarios](#troubleshooting-scenarios)
- [System Design](#system-design)
- [Behavioral](#behavioral)

---

## Linux & Shell

### Q1: Explain the boot process of a Linux system
**Answer:**
```
1. BIOS/UEFI
   - Hardware initialization
   - POST (Power-On Self-Test)
   - Find bootable device

2. Bootloader (GRUB)
   - Load kernel into memory
   - Pass boot parameters

3. Kernel Initialization
   - Initialize hardware
   - Mount root filesystem (initramfs)
   - Start init process (PID 1)

4. Init System (systemd/init)
   - Start system services
   - Mount filesystems
   - Network configuration

5. Login Prompt
   - User space fully initialized
   - Ready for login
```

---

### Q2: How do you troubleshoot high CPU usage?
**Answer:**
```
Steps:
1. Identify process
   top        # Real-time view
   htop       # Better UI
   ps aux --sort=-%cpu | head

2. Check process details
   ps -p PID -o pid,user,cmd,%cpu,%mem
   lsof -p PID    # Files opened

3. Investigate cause
   strace -p PID  # System calls
   perf top       # Performance profiling

4. Check for:
   - Infinite loops
   - High traffic
   - Resource contention
   - Runaway scripts

5. Resolve:
   - Optimize code
   - Scale resources
   - Kill process if needed
   - Fix bug

6. Monitor:
   - Set up alerts
   - Track trends
```

---

### Q3: Difference between hard link and soft link?
**Answer:**
```
Hard Link:
- Points to same inode
- Same as original file
- Can't link directories
- Can't cross filesystems
- If original deleted, link still works

ln file.txt hardlink.txt

Soft Link (Symbolic):
- Points to file path
- Separate inode
- Can link directories
- Can cross filesystems
- Breaks if original deleted

ln -s file.txt softlink.txt

Use cases:
Hard: Backup, prevent accidental deletion
Soft: Shortcuts, version management
```

---

## Docker & Containers

### Q4: How do you optimize Docker images?
**Answer:**
```
1. Multi-stage builds
FROM node:18 AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:18-slim
COPY --from=build /app/dist ./dist
CMD ["node", "dist/index.js"]

2. Layer optimization
# ✓ GOOD - Stable layers first
COPY package*.json ./
RUN npm ci
COPY . .    # Changes frequently, last

# ✗ BAD - Frequent changes first
COPY . .
RUN npm ci

3. Minimal base images
- alpine (5MB) vs ubuntu (75MB)
- distroless for security

4. .dockerignore
node_modules
.git
*.log

5. Combine RUN statements
RUN apt-get update && \
    apt-get install -y pkg1 pkg2 && \
    rm -rf /var/lib/apt/lists/*

Result: 500MB → 50MB image
```

---

### Q5: Container vs VM?
**Answer:**
```
Virtual Machine:
┌─────────────┐
│ Application │
│ Bins/Libs   │
│ Guest OS    │ ← Full OS (GBs)
│ Hypervisor  │
│ Host OS     │
│ Hardware    │
└─────────────┘
- Slower startup (minutes)
- More isolation
- More resources

Container:
┌─────────────┐
│ Application │
│ Bins/Libs   │
│ Container   │ ← Shared kernel
│ Host OS     │
│ Hardware    │
└─────────────┘
- Fast startup (seconds)
- Less isolation
- Lightweight

Use VMs for: Different OS, strong isolation
Use Containers for: Microservices, CI/CD, dev/prod parity
```

---

## Kubernetes

### Q6: Explain Kubernetes architecture
**Answer:**
```
Control Plane:
- API Server: Frontend, REST API
- etcd: Distributed key-value store
- Scheduler: Assigns pods to nodes
- Controller Manager: Maintains desired state
- Cloud Controller: Cloud-specific logic

Worker Nodes:
- kubelet: Runs on each node, manages pods
- kube-proxy: Network rules, load balancing
- Container Runtime: Docker, containerd

Flow:
1. User → kubectl → API Server
2. API Server → etcd (stores state)
3. Scheduler → assigns pod to node
4. kubelet → starts containers
5. kube-proxy → networking
```

---

### Q7: Pod vs Deployment vs StatefulSet?
**Answer:**
```
Pod:
- Smallest unit
- One or more containers
- Ephemeral
- Use: Rarely directly

Deployment:
- Manages ReplicaSets
- Stateless applications
- Rolling updates
- Use: Web servers, APIs

StatefulSet:
- Ordered, unique pods
- Persistent identity
- Stable network IDs
- Use: Databases, ZooKeeper

DaemonSet:
- One pod per node
- Use: Logging agents, monitoring

Job/CronJob:
- Run to completion
- Use: Batch processing, scheduled tasks
```

---

### Q8: How do you troubleshoot CrashLoopBackOff?
**Answer:**
```
Steps:
1. Check pod status
   kubectl get pods
   kubectl describe pod mypod

2. Check logs
   kubectl logs mypod
   kubectl logs mypod --previous  # Previous crash

3. Check events
   kubectl get events --sort-by=.metadata.creationTimestamp

4. Common causes:
   - Application crash (check logs)
   - Missing config/secrets
   - Insufficient resources
   - Failing liveness probe
   - Wrong image

5. Debug:
   kubectl debug mypod -it --image=busybox
   kubectl exec -it mypod -- sh

6. Fix based on cause:
   - Fix application code
   - Add missing configs
   - Increase resources
   - Adjust probe timings
```

---

## CI/CD

### Q9: Design a CI/CD pipeline for microservices
**Answer:**
```
Pipeline Stages:

1. Code Quality (parallel)
   - Lint
   - Security scan (Snyk, SonarQube)
   - Unit tests

2. Build
   - Build Docker images
   - Tag with commit SHA
   - Push to registry

3. Test (parallel)
   - Integration tests
   - Contract tests
   - E2E tests (subset)

4. Deploy to Dev
   - Automatic
   - Run smoke tests

5. Deploy to Staging
   - Automatic for main branch
   - Full E2E tests
   - Performance tests

6. Deploy to Production
   - Manual approval
   - Canary deployment (10% → 50% → 100%)
   - Monitor metrics
   - Auto-rollback on errors

Key considerations:
- Service dependencies (deploy order)
- Database migrations (before code)
- Feature flags (decouple deploy from release)
- Observability (track deployments)
```

---

### Q10: Blue-Green vs Canary vs Rolling deployment?
**Answer:**
```
Blue-Green:
Old: ███████ 100% traffic
New: ███████ 0% traffic
[Switch]
Old: ███████ 0% traffic
New: ███████ 100% traffic

Pros: Instant rollback, full testing
Cons: 2x resources, all-or-nothing

Canary:
Old: ████████████ 90%
New: ██ 10%
[Monitor, gradually increase]
Old: ████ 0%
New: ████████████ 100%

Pros: Gradual, limited blast radius
Cons: Complex routing, longer rollout

Rolling:
Pod1: █ (v1) → █ (v2)
Pod2: █ (v1) → █ (v2)
Pod3: █ (v1) → █ (v2)

Pros: No extra resources, gradual
Cons: Mixed versions running, slower rollback

Choose based on:
- Risk tolerance
- Resource availability
- Rollback requirements
```

---

## Infrastructure as Code

### Q11: Terraform vs CloudFormation vs Ansible?
**Answer:**
```
Terraform:
+ Multi-cloud (AWS, Azure, GCP)
+ Large provider ecosystem
+ Good state management
+ HCL language
- Requires state management
Use: Multi-cloud infrastructure

CloudFormation:
+ Deep AWS integration
+ No state management
+ Native AWS service
- AWS only
- Slower updates
- Complex syntax
Use: AWS-only shops

Ansible:
+ Agentless
+ Procedural (full control)
+ Easy to learn (YAML)
- Slower at scale
- No native state
Use: Configuration management

Typical stack:
1. Terraform → Provision infrastructure
2. Ansible → Configure servers
3. Kubernetes → Run containers
```

---

### Q12: How do you manage Terraform state in a team?
**Answer:**
```
❌ BAD: Local state file
- Can't collaborate
- No locking
- Risk of conflicts

✓ GOOD: Remote backend

terraform {
  backend "s3" {
    bucket         = "terraform-state"
    key            = "prod/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-lock"  # Locking
  }
}

Best practices:
1. Remote state (S3, Terraform Cloud)
2. State locking (DynamoDB, Terraform Cloud)
3. Encryption at rest
4. Versioning enabled
5. Separate state per environment
6. Never commit state to Git
7. Use workspaces or directories

Security:
- Restrict access (IAM)
- Audit changes
- Backup state regularly
```

---

## Monitoring & Logging

### Q13: What are the Four Golden Signals?
**Answer:**
```
1. Latency
   - Request duration
   - P50, P95, P99 percentiles
   - Target: <200ms web, <100ms API

2. Traffic
   - Requests per second
   - Concurrent users
   - Bandwidth

3. Errors
   - Error rate (%)
   - 4xx vs 5xx
   - Target: <0.1% for 5xx

4. Saturation
   - Resource utilization
   - CPU, memory, disk
   - Queue depths

These 4 metrics tell you system health!

Example queries (Prometheus):
# Latency P95
histogram_quantile(0.95, rate(http_duration_bucket[5m]))

# Error rate
sum(rate(http_requests{status=~"5.."}[5m])) / sum(rate(http_requests[5m]))

# Saturation
rate(node_cpu_seconds_total{mode="idle"}[5m])
```

---

### Q14: Structured vs unstructured logging?
**Answer:**
```
Unstructured:
"2024-01-01 User john purchased item 123 for $50"
- Hard to parse
- Can't filter by field
- No aggregations

Structured (JSON):
{
  "timestamp": "2024-01-01T10:00:00Z",
  "level": "INFO",
  "message": "Purchase completed",
  "user_id": "john",
  "item_id": "123",
  "amount": 50.00,
  "currency": "USD"
}

Benefits:
- Easy to parse
- Filter by any field
- Aggregate (sum(amount) by user_id)
- Search efficiently
- Machine-readable

Always use structured logging in production!

Include:
- Timestamp
- Log level
- Service name
- Request ID (correlation)
- User ID (if applicable)
- Error details
```

---

## Networking

### Q15: Explain OSI Model
**Answer:**
```
7. Application  - HTTP, DNS, SSH
6. Presentation - SSL/TLS, encryption
5. Session      - Session management
4. Transport    - TCP, UDP
3. Network      - IP, routing
2. Data Link    - MAC, switches
1. Physical     - Cables, signals

Practical:
L7: Load balancers (nginx, ALB)
L4: Load balancers (NLB)
L3: Routers, IP addresses
L2: Switches, VLANs

TCP/IP Model (simpler):
4. Application (L5-7)
3. Transport (L4)
2. Internet (L3)
1. Network Access (L1-2)
```

---

### Q16: How does DNS work?
**Answer:**
```
example.com lookup:

1. Browser cache
   └─ Check cache

2. OS cache
   └─ Check /etc/hosts

3. Recursive DNS (ISP)
   └─ Check cache

4. Root nameserver
   └─ Returns .com nameserver

5. TLD nameserver (.com)
   └─ Returns example.com nameserver

6. Authoritative nameserver
   └─ Returns IP: 93.184.216.34

7. Result cached at each level
   - Browser: 5 min
   - OS: 5 min
   - Recursive: 1 hour (TTL)

DNS Records:
- A: IPv4 address
- AAAA: IPv6 address
- CNAME: Alias to another domain
- MX: Mail server
- TXT: Text (SPF, DKIM)
- NS: Nameserver
```

---

## Security

### Q17: How do you secure a Kubernetes cluster?
**Answer:**
```
1. Network Security
   - Network policies (restrict pod communication)
   - Private cluster (no public API)
   - Service mesh (mTLS)

2. RBAC
   - Least privilege
   - ServiceAccounts for pods
   - No default admin access

3. Pod Security
   - Pod Security Standards
   - No privileged containers
   - Read-only root filesystem
   - Run as non-root
   - Resource limits

4. Secrets Management
   - External secrets (Vault, AWS Secrets Manager)
   - Encrypt etcd at rest
   - No secrets in env vars

5. Image Security
   - Scan images (Trivy, Clair)
   - Private registry
   - Signed images
   - Minimal base images

6. Monitoring & Auditing
   - Enable audit logs
   - Monitor API calls
   - Intrusion detection (Falco)

7. Updates
   - Keep Kubernetes updated
   - Patch nodes regularly
```

---

### Q18: How do you manage secrets in CI/CD?
**Answer:**
```
❌ BAD:
- Hardcoded in code
- Committed to Git
- Plain text in CI config

✓ GOOD Options:

1. CI/CD Platform Secrets
   GitHub Actions:
   secrets:
     DB_PASSWORD: ${{ secrets.DB_PASSWORD }}
   
2. External Secret Managers
   - HashiCorp Vault
   - AWS Secrets Manager
   - Azure Key Vault
   
3. Environment Variables
   - Set in CI/CD platform
   - Masked in logs
   
4. Encrypted Files
   - ansible-vault
   - git-crypt

Best practices:
- Rotate regularly
- Audit access
- Principle of least privilege
- Different secrets per environment
- Never log secrets
```

---

## SRE Concepts

### Q19: What is an error budget?
**Answer:**
```
Error budget = 100% - SLO

Example:
SLO: 99.9% uptime
Error budget: 0.1% = 43 minutes/month

Use budget for:
- Feature deployments
- Experiments
- Infrastructure changes

When exhausted:
- Freeze feature development
- Focus on reliability
- Root cause analysis

Benefits:
- Balances reliability vs velocity
- Data-driven decisions
- Aligns dev and ops

Policy:
Budget remaining: Deploy freely
Budget low: Slow down
Budget exhausted: Stop deploys

Calculation:
Uptime: 99.95% (better than SLO)
Budget used: 0.05%
Budget remaining: 0.05%
```

---

### Q20: Explain blameless postmortems
**Answer:**
```
Blameless = Focus on systems, not people

❌ BAD:
"John deployed broken code"

✓ GOOD:
"Deployment lacked sufficient testing"

Elements:
1. Summary (2-3 sentences)
2. Impact (users, revenue, duration)
3. Root cause
4. Timeline
5. Detection
6. Resolution
7. Action items (with owners, dates)
8. Lessons learned

Key principles:
- Assume good intent
- Everyone acted reasonably
- Systems failed, not people
- Focus on prevention

Benefits:
- Psychological safety
- Learning culture
- Honest analysis
- Continuous improvement

Action items must be:
- Specific
- Assigned
- Time-bound
- Tracked to completion
```

---

## Troubleshooting Scenarios

### Q21: Production API is slow. How do you troubleshoot?
**Answer:**
```
1. Verify the issue
   - Check monitoring
   - Reproduce if possible
   - Get exact error/symptoms

2. Check recent changes
   - Recent deployments?
   - Config changes?
   - Infrastructure changes?

3. Application layer
   - Check application logs
   - Response times
   - Error rates
   - Database query times

4. Infrastructure layer
   - CPU, memory, disk on app servers
   - Network latency
   - Load balancer health

5. Dependencies
   - Database performance
   - External APIs
   - Cache hit rate

6. Database
   - Slow queries
   - Connection pool exhaustion
   - Lock contention

7. Network
   - Latency between services
   - DNS issues
   - Timeouts

Tools:
- APM (New Relic, Datadog)
- Distributed tracing
- Database query analyzer
- Network monitoring

Typical causes:
- Slow database query
- External API timeout
- Memory leak
- High traffic
- Resource exhaustion
```

---

### Q22: Kubernetes pod keeps restarting
**Answer:**
```
1. Check pod status
   kubectl get pods
   kubectl describe pod mypod

2. Common states:
   - CrashLoopBackOff
   - ImagePullBackOff
   - Pending
   - OOMKilled

3. Check logs
   kubectl logs mypod
   kubectl logs mypod --previous

4. Common causes:

CrashLoopBackOff:
- Application crashes
- Missing dependencies
- Failing health checks
- Wrong command

ImagePullBackOff:
- Image doesn't exist
- Registry auth failed
- Wrong image name

Pending:
- Insufficient resources
- Node selector mismatch
- Pending PVC

OOMKilled:
- Memory limit too low
- Memory leak

5. Debug
   kubectl exec -it mypod -- sh
   kubectl debug mypod -it --image=busybox

6. Check events
   kubectl get events --field-selector involvedObject.name=mypod

7. Fix based on cause
```

---

## System Design

### Q23: Design a CI/CD system for 100 microservices
**Answer:**
```
Requirements:
- 100 services
- Multiple teams
- Deploy independently
- High frequency

Architecture:

1. Source Control
   - Mono-repo or multi-repo
   - Git with branch protection

2. CI Platform
   - Jenkins/GitHub Actions/GitLab CI
   - Distributed executors
   - Cache dependencies

3. Pipeline per service
   stages:
     - Lint & Test
     - Build Docker image
     - Security scan
     - Deploy to dev (auto)
     - Deploy to staging (auto)
     - Deploy to prod (manual)

4. Artifact Storage
   - Docker registry (ECR, GCR, Harbor)
   - Versioning strategy (git SHA)

5. Deployment
   - Kubernetes
   - Helm charts per service
   - GitOps (ArgoCD, Flux)

6. Testing
   - Unit tests per service
   - Integration tests
   - Contract tests (Pact)
   - E2E tests (critical paths only)

7. Monitoring
   - Track deployments
   - Auto-rollback on errors
   - Metrics per service

8. Service Mesh
   - Istio for traffic management
   - Canary deployments
   - Circuit breakers

Key decisions:
- Mono-repo vs multi-repo
- Shared libraries versioning
- Deploy order (dependencies)
- Testing strategy
- Rollback mechanism
```

---

### Q24: Design monitoring for global application
**Answer:**
```
Requirements:
- Multiple regions
- Millions of users
- 99.99% SLA

Components:

1. Metrics Collection
   - Prometheus in each region
   - Thanos for global view
   - Push gateway for batch jobs

2. Logging
   - Fluent Bit on each node
   - Elasticsearch cluster per region
   - S3 for long-term storage

3. Tracing
   - Jaeger for distributed tracing
   - Sample rate: 1-10%

4. Dashboards
   - Grafana
   - Per-service dashboards
   - Global overview
   - SLO tracking

5. Alerting
   - AlertManager
   - PagerDuty integration
   - Slack notifications
   - Escalation policies

6. Synthetic Monitoring
   - Pingdom/StatusCake
   - Check from each region
   - Alert on latency/availability

7. Real User Monitoring
   - Track actual user experience
   - Geographic performance

8. SLIs/SLOs
   - 99.99% availability
   - P95 latency < 200ms
   - Error rate < 0.01%

Alert strategy:
- Symptom-based alerts
- Multiple severity levels
- Clear runbooks
- Avoid alert fatigue

Retention:
- Raw metrics: 15 days
- Aggregated: 1 year
- Logs: 30 days hot, 1 year cold
```

---

## Behavioral

### Q25: Tell me about a production outage you handled
**Use STAR method**

**Answer Example:**
```
Situation:
E-commerce site, Black Friday, payment processing down

Task:
Restore payments, minimize revenue loss

Action:
1. Gathered team on incident call
2. Checked monitoring - database connection pool exhausted
3. Reviewed recent changes - deployment 2 hours ago
4. Identified connection leak in new code
5. Rolled back deployment
6. Monitored recovery
7. Communicated with stakeholders

Result:
- Restored in 25 minutes
- Lost $50K revenue
- Prevented further loss ($500K/hour)
- Fixed bug properly next day
- Added connection pool monitoring
- Improved canary deployment

Learning:
- Need better integration tests
- Canary deployment too fast
- Connection pool monitoring missing
```

---

### Q26: How do you prioritize tasks when everything is urgent?
**Answer:**
```
Framework:

1. Assess Impact
   Critical: Production down, users affected
   High: Feature broken, some users affected
   Medium: Performance issue
   Low: Nice-to-have

2. Assess Urgency
   - Deadline
   - Stakeholder pressure
   - Dependencies

3. Prioritize
   Urgent + Critical: Do now
   Urgent + Not critical: Schedule soon
   Not urgent + Critical: Plan carefully
   Not urgent + Not critical: Backlog

4. Communicate
   - Set expectations
   - Explain priorities
   - Negotiate deadlines

5. Focus
   - One task at a time
   - Batch similar tasks
   - Delegate when possible

Example:
Production down: Drop everything
Deploy requested for today: Can it wait?
Refactoring project: Schedule for next sprint
```

---

## Quick Interview Prep Checklist

```
Day Before:
□ Review resume - know your projects
□ Prepare STAR stories (5-7 incidents)
□ Review company's tech stack
□ Practice whiteboard explanations
□ Test equipment (camera, mic)
□ Prepare questions for interviewer

During Interview:
□ Think out loud
□ Ask clarifying questions
□ Draw diagrams
□ Explain trade-offs
□ Admit what you don't know
□ Ask for hints if stuck

After Interview:
□ Send thank you email
□ Note topics you struggled with
□ Study those topics
```

---

## Common Topics to Master

```
Must Know:
- Linux fundamentals
- Docker/containers
- Kubernetes basics
- Git workflows
- CI/CD pipelines
- Monitoring basics
- Troubleshooting methodology

Should Know:
- Terraform/Ansible
- Prometheus/Grafana
- Networking basics
- Security best practices
- AWS/Azure/GCP basics

Nice to Have:
- Service mesh
- Advanced Kubernetes
- Multiple cloud platforms
- Programming (Python/Go)
```

---

## Tips for Success

```
1. Practice Explaining
   - Explain to non-technical friend
   - Record yourself
   - Use analogies

2. Hands-On Practice
   - Set up lab environment
   - Break things and fix them
   - Document what you learn

3. Stay Current
   - Follow DevOps blogs
   - Try new tools
   - Contribute to open source

4. Know Fundamentals
   - Don't just memorize commands
   - Understand WHY things work
   - Know trade-offs

5. Be Honest
   - "I don't know, but here's how I'd find out"
   - Better than making things up

6. Show Growth Mindset
   - Talk about what you're learning
   - Share mistakes and lessons
   - Demonstrate curiosity
```

---

**Good luck with your interviews! 🚀**

*"The best way to predict the future is to invent it."*

**Remember:**
- Be confident but humble
- Think out loud
- Ask questions
- Show passion for learning
- You've got this!

---

*Practice makes perfect. Review this guide regularly and you'll ace those interviews!*

