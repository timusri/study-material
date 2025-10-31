# DevOps/SRE Quick Reference Guide

## 🚀 Essential Commands Cheat Sheet

This is your quick reference for daily DevOps tasks. Bookmark this page!

---

## Linux/Shell

### File Operations
```bash
# List files
ls -lah                      # Detailed list with hidden files
ls -ltr                      # List by time (recent last)

# Find files
find /path -name "*.log"     # Find files by name
find . -type f -mtime -7     # Files modified in last 7 days
find . -size +100M           # Files larger than 100MB

# Search in files
grep -r "error" /var/log/    # Recursive search
grep -i "warn" app.log       # Case insensitive
grep -A 5 -B 5 "error" app.log  # 5 lines before/after

# Disk usage
df -h                        # Disk space
du -sh *                     # Directory sizes
du -h --max-depth=1 | sort -hr  # Sorted by size

# Process management
ps aux | grep nginx          # Find process
kill -9 PID                  # Force kill
pkill nginx                  # Kill by name
top                          # Live process monitor
htop                         # Better top (if installed)

# Networking
netstat -tuln                # Listening ports
ss -tuln                     # Modern alternative
lsof -i :8080                # What's using port 8080
curl -I https://example.com  # HTTP headers
tcpdump -i eth0              # Packet capture
```

---

## Git

### Daily Workflow
```bash
# Status & changes
git status
git diff
git log --oneline --graph

# Branch operations
git branch                   # List branches
git checkout -b feature      # Create & switch
git merge feature            # Merge branch
git branch -d feature        # Delete branch

# Commit & push
git add .
git commit -m "message"
git push origin main

# Undo operations
git restore file.txt         # Discard changes
git restore --staged file.txt  # Unstage
git reset --soft HEAD~1      # Undo last commit
git revert <commit>          # Revert commit

# Stash
git stash                    # Save work
git stash pop                # Restore work
git stash list               # List stashes
```

---

## Docker

### Container Management
```bash
# Run container
docker run -d -p 80:80 --name web nginx
docker run -it ubuntu bash   # Interactive

# List & inspect
docker ps                    # Running containers
docker ps -a                 # All containers
docker logs web              # View logs
docker logs -f web           # Follow logs
docker inspect web           # Detailed info
docker stats                 # Resource usage

# Execute commands
docker exec -it web bash     # Enter container
docker exec web ls /app      # Run command

# Stop & remove
docker stop web
docker rm web
docker rm -f web             # Force remove

# Clean up
docker system prune          # Remove unused
docker volume prune          # Remove volumes
docker image prune           # Remove images
```

### Image Management
```bash
# Build & push
docker build -t myapp:1.0 .
docker tag myapp:1.0 myapp:latest
docker push myrepo/myapp:1.0

# List & remove
docker images
docker rmi image:tag
docker rmi $(docker images -q)  # Remove all

# Pull images
docker pull nginx:latest
docker pull ubuntu:20.04
```

### Docker Compose
```bash
# Start services
docker-compose up -d
docker-compose ps
docker-compose logs -f

# Stop services
docker-compose down
docker-compose stop

# Scale services
docker-compose up -d --scale web=3
```

---

## Kubernetes

### Kubectl Basics
```bash
# Context & config
kubectl config get-contexts
kubectl config use-context prod
kubectl config set-context --current --namespace=myapp

# Get resources
kubectl get pods
kubectl get svc
kubectl get deploy
kubectl get all                 # All resources
kubectl get pods -A             # All namespaces
kubectl get pods -o wide        # More details
kubectl get pods -o yaml        # YAML output

# Describe & logs
kubectl describe pod mypod
kubectl logs mypod
kubectl logs -f mypod           # Follow logs
kubectl logs mypod -c container # Specific container

# Execute commands
kubectl exec -it mypod -- bash
kubectl exec mypod -- ls /app

# Port forwarding
kubectl port-forward pod/mypod 8080:80
kubectl port-forward svc/myservice 8080:80
```

### Common Operations
```bash
# Apply configuration
kubectl apply -f deployment.yml
kubectl apply -f ./manifests/

# Edit resources
kubectl edit deployment myapp
kubectl scale deployment myapp --replicas=3
kubectl set image deployment/myapp app=myapp:v2

# Delete resources
kubectl delete pod mypod
kubectl delete -f deployment.yml
kubectl delete deploy myapp

# Rollout management
kubectl rollout status deployment/myapp
kubectl rollout history deployment/myapp
kubectl rollout undo deployment/myapp

# Debug
kubectl top nodes               # Node resources
kubectl top pods                # Pod resources
kubectl get events --sort-by=.metadata.creationTimestamp
```

---

## Terraform

### Workflow
```bash
# Initialize
terraform init

# Plan & apply
terraform plan
terraform plan -out=plan.tfplan
terraform apply
terraform apply -auto-approve
terraform apply plan.tfplan

# Destroy
terraform destroy
terraform destroy -auto-approve

# State management
terraform state list
terraform state show aws_instance.web
terraform state rm aws_instance.web

# Formatting & validation
terraform fmt
terraform validate

# Workspaces
terraform workspace list
terraform workspace new prod
terraform workspace select prod
```

---

## Ansible

### Ad-hoc Commands
```bash
# Ping hosts
ansible all -m ping

# Run commands
ansible all -a "uptime"
ansible webservers -a "systemctl status nginx"

# Package management
ansible all -m apt -a "name=nginx state=present" --become

# File operations
ansible all -m copy -a "src=file.txt dest=/tmp/" 
ansible all -m file -a "path=/tmp/test state=directory"

# Service management
ansible all -m service -a "name=nginx state=restarted" --become
```

### Playbooks
```bash
# Run playbook
ansible-playbook site.yml
ansible-playbook site.yml --check        # Dry run
ansible-playbook site.yml --diff         # Show changes
ansible-playbook site.yml -v             # Verbose
ansible-playbook site.yml --tags nginx   # Specific tags
ansible-playbook site.yml --limit web1   # Specific host

# Vault
ansible-vault create secrets.yml
ansible-vault edit secrets.yml
ansible-vault encrypt vars.yml
ansible-vault decrypt vars.yml
ansible-playbook site.yml --ask-vault-pass
```

---

## Prometheus & Grafana

### PromQL Queries
```promql
# CPU usage
100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

# Memory usage
100 * (1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes))

# Request rate
rate(http_requests_total[5m])

# Error rate
sum(rate(http_requests_total{status=~"5.."}[5m])) / sum(rate(http_requests_total[5m]))

# P95 latency
histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))

# Top 10 endpoints
topk(10, sum by (endpoint) (rate(http_requests_total[5m])))
```

---

## AWS CLI

### EC2
```bash
# List instances
aws ec2 describe-instances
aws ec2 describe-instances --query 'Reservations[].Instances[].[InstanceId,State.Name,PublicIpAddress]' --output table

# Start/stop instances
aws ec2 start-instances --instance-ids i-1234567890abcdef0
aws ec2 stop-instances --instance-ids i-1234567890abcdef0

# Create instance
aws ec2 run-instances --image-id ami-12345 --instance-type t2.micro --key-name mykey
```

### S3
```bash
# List buckets
aws s3 ls
aws s3 ls s3://mybucket

# Copy files
aws s3 cp file.txt s3://mybucket/
aws s3 cp s3://mybucket/file.txt ./
aws s3 sync ./dir s3://mybucket/dir/

# Remove files
aws s3 rm s3://mybucket/file.txt
aws s3 rb s3://mybucket --force  # Remove bucket
```

### IAM
```bash
# List users
aws iam list-users

# Create user
aws iam create-user --user-name john

# Attach policy
aws iam attach-user-policy --user-name john --policy-arn arn:aws:iam::aws:policy/ReadOnlyAccess
```

---

## Networking

### Troubleshooting
```bash
# Connectivity
ping google.com
ping -c 4 8.8.8.8            # Send 4 packets

# DNS
nslookup google.com
dig google.com
dig +short google.com

# Route tracing
traceroute google.com
mtr google.com               # Better traceroute

# Port testing
telnet example.com 80
nc -zv example.com 80        # Netcat
timeout 5 bash -c '</dev/tcp/example.com/80' && echo "Open" || echo "Closed"

# HTTP testing
curl https://example.com
curl -I https://example.com  # Headers only
curl -X POST -d '{"key":"value"}' -H "Content-Type: application/json" https://api.example.com

# Check listening ports
netstat -tuln
ss -tuln
lsof -i :80
```

---

## SSL/TLS

### Certificate Management
```bash
# Generate private key
openssl genrsa -out private.key 2048

# Generate CSR
openssl req -new -key private.key -out request.csr

# Self-signed certificate
openssl req -x509 -newkey rsa:2048 -keyout key.pem -out cert.pem -days 365 -nodes

# View certificate
openssl x509 -in cert.pem -text -noout

# Test SSL connection
openssl s_client -connect example.com:443
openssl s_client -connect example.com:443 -servername example.com  # SNI

# Check certificate expiry
echo | openssl s_client -servername example.com -connect example.com:443 2>/dev/null | openssl x509 -noout -dates

# Convert formats
openssl x509 -in cert.crt -out cert.pem -outform PEM
```

---

## System Monitoring

### Quick Health Checks
```bash
# CPU
top
htop
mpstat 1 5                   # CPU stats every 1 sec, 5 times
uptime                       # Load average

# Memory
free -h
vmstat 1 5

# Disk
df -h                        # Disk space
iostat -x 1 5                # Disk I/O

# Network
iftop                        # Network traffic (if installed)
nethogs                      # Per-process bandwidth

# All-in-one
glances                      # If installed
```

### Log Analysis
```bash
# Tail logs
tail -f /var/log/syslog
tail -n 100 /var/log/nginx/access.log

# Search logs
grep "error" /var/log/app.log
grep -i "fail" /var/log/syslog | tail -20

# Count occurrences
grep "404" /var/log/nginx/access.log | wc -l

# Top IP addresses
awk '{print $1}' /var/log/nginx/access.log | sort | uniq -c | sort -rn | head -10

# Top requested URLs
awk '{print $7}' /var/log/nginx/access.log | sort | uniq -c | sort -rn | head -10

# Response time stats
awk '{sum+=$NF; count++} END {print "Avg:", sum/count}' /var/log/nginx/access.log
```

---

## Troubleshooting Workflows

### Application Down
```bash
1. Check if process is running
   ps aux | grep app_name
   systemctl status app_name

2. Check logs
   journalctl -u app_name -f
   tail -f /var/log/app/error.log

3. Check port availability
   netstat -tuln | grep :8080
   lsof -i :8080

4. Check system resources
   top
   df -h
   free -h

5. Test connectivity
   curl localhost:8080/health
   telnet localhost 8080

6. Restart service
   systemctl restart app_name
```

### High CPU/Memory
```bash
1. Find culprit process
   top                    # Press Shift+M for memory, Shift+P for CPU
   ps aux --sort=-%mem | head
   ps aux --sort=-%cpu | head

2. Investigate process
   ps -p PID -o pid,user,cmd,%cpu,%mem,time
   lsof -p PID           # Files opened by process
   strace -p PID         # System calls (careful!)

3. Check for runaway loops
   grep -r "while true" /opt/app/
   pgrep -f "infinite_loop.sh"

4. Historical data
   sar -u 1 10           # CPU usage (if sysstat installed)
   sar -r 1 10           # Memory usage
```

### Network Issues
```bash
1. Basic connectivity
   ping 8.8.8.8
   ping google.com

2. DNS resolution
   nslookup google.com
   dig google.com

3. Route tracing
   traceroute google.com
   mtr google.com

4. Firewall/Security groups
   iptables -L -n
   # Check cloud console for security groups

5. Port availability
   telnet host.com 443
   nc -zv host.com 443

6. HTTP/HTTPS specific
   curl -v https://api.example.com
   curl --resolve example.com:443:1.2.3.4 https://example.com
```

### Kubernetes Pod Issues
```bash
1. Check pod status
   kubectl get pods
   kubectl describe pod mypod

2. Check logs
   kubectl logs mypod
   kubectl logs mypod --previous  # Previous container

3. Check events
   kubectl get events --sort-by=.metadata.creationTimestamp

4. Check resources
   kubectl top nodes
   kubectl top pods

5. Debug with ephemeral container
   kubectl debug mypod -it --image=busybox

6. Check configuration
   kubectl get pod mypod -o yaml
   kubectl get configmap myconfig -o yaml
```

---

## CI/CD Pipeline Snippets

### GitHub Actions
```yaml
name: CI
on: [push]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: npm ci
      - run: npm test
      - run: npm run build
```

### GitLab CI
```yaml
stages:
  - build
  - test
  - deploy

build:
  stage: build
  script:
    - docker build -t myapp .

test:
  stage: test
  script:
    - npm test

deploy:
  stage: deploy
  script:
    - kubectl apply -f k8s/
  only:
    - main
```

### Jenkins
```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh 'npm install'
            }
        }
        stage('Test') {
            steps {
                sh 'npm test'
            }
        }
        stage('Deploy') {
            steps {
                sh './deploy.sh'
            }
        }
    }
}
```

---

## Security Quick Checks

### File Permissions
```bash
# Check world-writable files
find / -type f -perm -002 2>/dev/null

# Check SUID files
find / -type f -perm -4000 2>/dev/null

# Fix permissions
chmod 600 ~/.ssh/id_rsa
chmod 644 ~/.ssh/id_rsa.pub
chmod 700 ~/.ssh
```

### Password & Keys
```bash
# Generate strong password
openssl rand -base64 32

# Generate SSH key
ssh-keygen -t ed25519 -C "your_email@example.com"

# Check SSH connection
ssh -T git@github.com
```

### Port Scanning
```bash
# Scan ports (on your own systems only!)
nmap localhost
nmap -p 1-65535 localhost    # All ports
nmap -sV localhost           # Service versions
```

---

## Backup & Recovery

### Database Backups
```bash
# MySQL
mysqldump -u root -p database_name > backup.sql
mysql -u root -p database_name < backup.sql

# PostgreSQL
pg_dump -U postgres database_name > backup.sql
psql -U postgres database_name < backup.sql

# MongoDB
mongodump --db database_name --out /backup/
mongorestore --db database_name /backup/database_name/
```

### File Backups
```bash
# Tar archive
tar -czf backup.tar.gz /path/to/files
tar -xzf backup.tar.gz

# Rsync
rsync -avz /source/ /destination/
rsync -avz /source/ user@remote:/destination/

# Incremental backup with rsync
rsync -avz --link-dest=/backup/last /source/ /backup/current/
```

---

## Performance Testing

### Load Testing
```bash
# Apache Bench
ab -n 1000 -c 10 http://example.com/

# Wrk
wrk -t4 -c100 -d30s http://example.com/

# Hey
hey -n 1000 -c 10 http://example.com/
```

### Stress Testing
```bash
# CPU stress
stress --cpu 4 --timeout 60s

# Memory stress
stress --vm 2 --vm-bytes 1G --timeout 60s

# Disk stress
stress --io 4 --timeout 60s
```

---

## Environment Variables

### Common Variables
```bash
# Set variables
export DB_HOST=localhost
export DB_PORT=5432
export APP_ENV=production

# Load from file
source .env
export $(cat .env | xargs)

# Unset variable
unset DB_HOST

# View all variables
env
printenv
echo $PATH
```

---

## Quick Scripts

### Health Check Script
```bash
#!/bin/bash
# health_check.sh

URL="https://example.com"
STATUS=$(curl -s -o /dev/null -w "%{http_code}" $URL)

if [ $STATUS -eq 200 ]; then
    echo "✓ Site is up"
    exit 0
else
    echo "✗ Site is down (Status: $STATUS)"
    exit 1
fi
```

### Log Rotation
```bash
#!/bin/bash
# rotate_logs.sh

LOG_DIR="/var/log/myapp"
MAX_LOGS=5

cd $LOG_DIR
for log in *.log; do
    gzip $log
    mv $log.gz $log.$(date +%Y%m%d).gz
done

# Keep only last 5
ls -t *.gz | tail -n +$((MAX_LOGS+1)) | xargs rm -f
```

### Backup Script
```bash
#!/bin/bash
# backup.sh

BACKUP_DIR="/backup/$(date +%Y%m%d)"
SOURCE="/var/www/html"

mkdir -p $BACKUP_DIR
tar -czf $BACKUP_DIR/www_backup.tar.gz $SOURCE

# Upload to S3
aws s3 cp $BACKUP_DIR/www_backup.tar.gz s3://mybucket/backups/

# Clean old backups (keep 7 days)
find /backup -type d -mtime +7 -exec rm -rf {} \;
```

---

## Useful Aliases

Add to `~/.bashrc` or `~/.zshrc`:

```bash
# Navigation
alias ..='cd ..'
alias ...='cd ../..'
alias ll='ls -lah'

# Git
alias gs='git status'
alias ga='git add'
alias gc='git commit -m'
alias gp='git push'
alias gl='git log --oneline --graph'

# Docker
alias d='docker'
alias dc='docker-compose'
alias dps='docker ps'
alias dlog='docker logs -f'

# Kubernetes
alias k='kubectl'
alias kgp='kubectl get pods'
alias kgs='kubectl get svc'
alias kgd='kubectl get deploy'
alias kl='kubectl logs -f'
alias kx='kubectl exec -it'

# System
alias ports='netstat -tuln'
alias myip='curl ifconfig.me'
alias serve='python3 -m http.server 8000'
```

---

## Keyboard Shortcuts

### Terminal
```
Ctrl + A  - Move to beginning of line
Ctrl + E  - Move to end of line
Ctrl + L  - Clear screen
Ctrl + R  - Search command history
Ctrl + Z  - Suspend process (fg to resume)
Ctrl + C  - Kill process
Ctrl + D  - Exit/logout
```

### Vim
```
i      - Insert mode
Esc    - Normal mode
:w     - Save
:q     - Quit
:wq    - Save and quit
:q!    - Quit without saving
dd     - Delete line
yy     - Copy line
p      - Paste
u      - Undo
/text  - Search
n      - Next search result
```

---

## Emergency Contacts

When things go wrong:

### Runbook Checklist
- [ ] Check monitoring dashboards
- [ ] Review recent deployments
- [ ] Check application logs
- [ ] Verify infrastructure health
- [ ] Check external dependencies
- [ ] Review recent configuration changes
- [ ] Escalate if needed

### Communication
```
Incident Declared:
1. Post in #incidents channel
2. Create incident doc
3. Update status page
4. Notify stakeholders
5. Start video call
```

---

## Remember

**Before Running Commands:**
- ✅ Know what the command does
- ✅ Have backups
- ✅ Test in non-prod first
- ✅ Can you roll back?

**Production Rules:**
- 🚫 Never `rm -rf` without double-checking
- 🚫 Never run untested scripts
- 🚫 Never deploy on Friday afternoon
- ✅ Always have a rollback plan

---

**Bookmark this page! 📌**

**More details in individual topic files →**

---

*Last updated: 2025*

