# 1. Linux Fundamentals

## 📚 Quick Summary

Linux is the foundation of DevOps - 90%+ of servers run Linux!

**What You'll Learn:**
- **Command Line Mastery**: 50+ essential commands
- **File System**: Permissions, ownership, file operations
- **Process Management**: ps, top, kill, systemd
- **Networking**: TCP/IP, DNS, network troubleshooting
- **Shell Scripting**: Automate tasks with Bash
- **Package Management**: apt, yum, dnf

**Why This Matters:**
- Every DevOps tool runs on Linux
- SSH into servers daily
- Debug production issues via terminal
- Write automation scripts
- Interview questions: 20% are Linux-based

**Interview Reality:**
"Can you check why the service is down?" = You need Linux skills!

---

## 📖 Simple Explanation

**What is Linux?**
Linux is an operating system (like Windows or macOS) that runs most of the internet. It's free, open-source, and powers:
- Web servers (Amazon, Google, Facebook)
- Cloud platforms (AWS, Azure, GCP)
- Docker containers
- Kubernetes nodes
- Your router, Android phone, smart TV

**Why Terminal/Command Line?**
```
GUI (Graphical): Click buttons, drag files
CLI (Command Line): Type commands, super fast, automatable

Example:
GUI: Click 100 files, right-click each, delete
CLI: rm *.log  (delete all .log files in 1 second!)
```

**DevOps Daily Reality:**
```
08:00 AM: SSH into production server
08:05 AM: Check logs: tail -f /var/log/app.log
08:10 AM: Check disk space: df -h
08:15 AM: Restart service: systemctl restart nginx
08:20 AM: Monitor CPU: top
```

---

## Table of Contents
- [Essential Linux Commands](#essential-linux-commands)
- [File System & Permissions](#file-system--permissions)
- [Process Management](#process-management)
- [Networking Basics](#networking-basics)
- [Shell Scripting (Bash)](#shell-scripting-bash)
- [Package Management](#package-management)
- [System Administration](#system-administration)

---

## Essential Linux Commands

### 📖 Simple Explanation

Commands are instructions you type to tell Linux what to do.

**Structure:**
```bash
command [options] [arguments]

Example:
ls -lah /var/log
│   │   └─ argument (what to list)
│   └─ options (how to list)
└─ command (list files)
```

---

### Navigation Commands

```bash
# Print Working Directory (where am I?)
pwd
# Output: /home/devops

# List files
ls                    # basic list
ls -l                 # long format (detailed)
ls -a                 # show hidden files (start with .)
ls -lh                # human-readable sizes
ls -lart              # all, reverse, by time (most recent last)

# Change directory
cd /var/log           # absolute path
cd logs               # relative path
cd ..                 # go up one level
cd ~                  # go to home directory
cd -                  # go to previous directory

# Pro tip: Use Tab for auto-completion!
cd /var/l[Tab]  → cd /var/log/
```

**Real Example:**
```bash
$ pwd
/home/devops

$ ls -lh
total 12K
drwxr-xr-x 2 devops devops 4.0K Oct 22 10:00 scripts
-rw-r--r-- 1 devops devops  156 Oct 22 09:30 config.yaml
-rwxr-xr-x 1 devops devops  2.1K Oct 22 08:15 deploy.sh

$ cd scripts
$ pwd
/home/devops/scripts
```

---

### File Operations

```bash
# Create files
touch file.txt                    # create empty file
echo "Hello" > file.txt          # create with content (overwrites)
echo "World" >> file.txt         # append to file

# View files
cat file.txt                      # display entire file
less file.txt                     # page through file (q to quit)
head -n 10 file.txt              # first 10 lines
tail -n 20 file.txt              # last 20 lines
tail -f /var/log/app.log         # follow file (live updates)

# Copy, Move, Delete
cp source.txt destination.txt     # copy file
cp -r dir1 dir2                  # copy directory recursively
mv old.txt new.txt               # rename file
mv file.txt /tmp/                # move file
rm file.txt                      # delete file
rm -rf directory/                # delete directory (CAREFUL!)

# Search in files
grep "error" app.log             # find lines containing "error"
grep -i "error" app.log          # case-insensitive
grep -r "TODO" src/              # recursive search in directory
grep -c "error" app.log          # count matches
```

**Real Example: Debug Application**
```bash
# Application is slow, check logs
$ tail -100 /var/log/app.log | grep -i "error"
2024-10-22 10:30:15 ERROR Database connection timeout
2024-10-22 10:30:20 ERROR Failed to connect to redis
2024-10-22 10:30:25 ERROR Unable to process request

# Found the issue: Database connection timeout!
```

---

### Directory Operations

```bash
# Create directories
mkdir mydir                       # create directory
mkdir -p path/to/deep/dir        # create parent directories

# Remove directories
rmdir emptydir                   # remove empty directory
rm -rf directory/                # remove directory and contents

# Find files
find /var/log -name "*.log"      # find all .log files
find . -type f -mtime -7         # files modified in last 7 days
find . -size +100M               # files larger than 100MB

# Disk usage
du -sh directory/                # size of directory
du -h --max-depth=1 /var/log/   # size of each subdirectory
```

**Real Example: Find Large Files**
```bash
# Server running out of disk space
$ df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1        50G   48G   2G  96% /

# Find what's using space
$ du -h --max-depth=1 /var/log/ | sort -h
1.5G    /var/log/nginx
2.1G    /var/log/app
5.2G    /var/log/old-backups

# Clean up old backups
$ rm -rf /var/log/old-backups/
```

---

## File System & Permissions

### 📖 Simple Explanation

**Linux File System:**
```
/                       (root, top of everything)
├── /home              (user home directories)
├── /var/log           (application logs)
├── /etc               (configuration files)
├── /usr/bin           (user programs)
├── /tmp               (temporary files)
└── /opt               (optional software)
```

**Permissions:**
Every file has:
- **Owner**: Who owns the file
- **Group**: Group that owns the file
- **Permissions**: What actions are allowed

```
-rw-r--r-- 1 devops devops 1234 Oct 22 10:00 file.txt
│││││││││
│││└─ Others: r (read only)
││└── Group: r (read only)
│└─── Owner: rw (read + write)
└──── File type: - (regular file)
```

---

### Permission Commands

```bash
# View permissions
ls -l file.txt
# -rw-r--r-- 1 devops devops 1234 Oct 22 10:00 file.txt

# Change permissions (chmod)
chmod 755 script.sh              # rwxr-xr-x (common for scripts)
chmod 644 config.yaml            # rw-r--r-- (common for files)
chmod +x script.sh               # add execute permission
chmod -w file.txt                # remove write permission

# Change ownership (chown)
chown user:group file.txt        # change owner and group
chown -R user:group directory/   # recursive
```

**Permission Numbers:**
```
r = 4 (read)
w = 2 (write)
x = 1 (execute)

755 = rwxr-xr-x
7 = 4+2+1 = rwx (owner can read, write, execute)
5 = 4+0+1 = r-x (group can read, execute)
5 = 4+0+1 = r-x (others can read, execute)

644 = rw-r--r--
6 = 4+2+0 = rw- (owner can read, write)
4 = 4+0+0 = r-- (group can read only)
4 = 4+0+0 = r-- (others can read only)
```

**Real Example:**
```bash
# Create deployment script
$ cat > deploy.sh << 'EOF'
#!/bin/bash
echo "Deploying application..."
kubectl apply -f deployment.yaml
EOF

# Try to run it
$ ./deploy.sh
-bash: ./deploy.sh: Permission denied

# Check permissions
$ ls -l deploy.sh
-rw-r--r-- 1 devops devops 89 Oct 22 10:00 deploy.sh

# Add execute permission
$ chmod +x deploy.sh
$ ls -l deploy.sh
-rwxr-xr-x 1 devops devops 89 Oct 22 10:00 deploy.sh

# Now it works!
$ ./deploy.sh
Deploying application...
```

---

## Process Management

### 📖 Simple Explanation

**What is a Process?**
A running program. When you start an application, Linux creates a process.

**Process ID (PID):**
Each process gets a unique number (PID).

**Process States:**
- Running: Currently executing
- Sleeping: Waiting for something
- Stopped: Paused
- Zombie: Finished but parent hasn't acknowledged

---

### Process Commands

```bash
# List all processes
ps aux                           # all processes, all users
ps aux | grep nginx              # find specific process

# Top processes (interactive)
top                              # real-time view (q to quit)
htop                             # better version (if installed)

# Process tree
pstree                           # show process hierarchy

# Find process by name
pgrep nginx                      # get PID of nginx
ps aux | grep -i "docker"        # search for docker processes

# Kill processes
kill 1234                        # gracefully terminate PID 1234
kill -9 1234                     # force kill PID 1234
killall nginx                    # kill all nginx processes
pkill -f "python app.py"         # kill by pattern

# Background and foreground
./long-script.sh &               # run in background
jobs                             # list background jobs
fg %1                            # bring job 1 to foreground
Ctrl+Z                           # pause current process
bg                               # resume in background
```

**Real Example: Restart Stuck Service**
```bash
# Service is not responding
$ ps aux | grep app-server
devops    1234  98.5  12.3  /usr/bin/app-server

# Try graceful shutdown
$ kill 1234
$ sleep 5

# Check if still running
$ ps aux | grep app-server
devops    1234  98.5  12.3  /usr/bin/app-server  # Still there!

# Force kill
$ kill -9 1234

# Verify it's gone
$ ps aux | grep app-server
# No output = process killed

# Restart service
$ systemctl start app-server
```

---

### System Services (systemd)

```bash
# Service management
systemctl status nginx           # check service status
systemctl start nginx            # start service
systemctl stop nginx             # stop service
systemctl restart nginx          # restart service
systemctl reload nginx           # reload config (no downtime)
systemctl enable nginx           # start on boot
systemctl disable nginx          # don't start on boot

# View logs
journalctl -u nginx              # logs for nginx service
journalctl -u nginx -f           # follow logs
journalctl -u nginx --since today
journalctl -u nginx --since "2024-10-22 10:00:00"

# List all services
systemctl list-units --type=service
systemctl list-units --state=failed  # show failed services
```

**Real Example: Debug Service Failure**
```bash
# Check service status
$ systemctl status app-server
● app-server.service - Application Server
   Loaded: loaded (/etc/systemd/system/app-server.service)
   Active: failed (Result: exit-code)

# View logs
$ journalctl -u app-server --since "5 minutes ago"
Oct 22 10:30:15 server app-server[1234]: Error: Cannot connect to database
Oct 22 10:30:15 server app-server[1234]: Failed to start

# Fix the issue (update database config)
$ vim /etc/app-server/config.yaml

# Restart service
$ systemctl restart app-server

# Verify it's running
$ systemctl status app-server
● app-server.service - Application Server
   Loaded: loaded
   Active: active (running) since Oct 22 10:35:00
```

---

## Networking Basics

### 📖 Simple Explanation

**Networking = Computers talking to each other**

**Key Concepts:**
- **IP Address**: Like a phone number for computers (192.168.1.100)
- **Port**: Like apartment number (80 = HTTP, 443 = HTTPS, 22 = SSH)
- **DNS**: Translates names to IPs (google.com → 142.250.185.46)
- **TCP/UDP**: How data is sent (TCP = reliable, UDP = fast)

---

### Network Commands

```bash
# Check network interfaces
ip addr show                     # show all network interfaces
ip addr show eth0               # show specific interface
ifconfig                        # older command (still works)

# Ping (test connectivity)
ping google.com                 # test if host is reachable
ping -c 4 8.8.8.8              # send only 4 packets

# DNS lookup
nslookup google.com             # DNS query
dig google.com                  # detailed DNS query
host google.com                 # simple DNS lookup

# Network connections
netstat -tulpn                  # show listening ports
ss -tulpn                       # modern alternative to netstat
lsof -i :8080                   # what's using port 8080?

# Download files
curl https://example.com        # get content
curl -O https://example.com/file.zip  # download file
wget https://example.com/file.zip     # alternative

# Test HTTP endpoints
curl -I https://example.com     # get headers only
curl -X POST -d '{"key":"value"}' \
  -H "Content-Type: application/json" \
  https://api.example.com/endpoint
```

**Real Example: Debug Network Issue**
```bash
# Application can't connect to database
$ ping db-server.example.com
PING db-server.example.com (10.0.1.50): 56 data bytes
64 bytes from 10.0.1.50: icmp_seq=0 ttl=64 time=0.5 ms
# Network is working

# Check if database port is open
$ telnet 10.0.1.50 5432
Trying 10.0.1.50...
telnet: connect to address 10.0.1.50: Connection refused
# Database is not listening!

# Check what's listening on database server
$ ssh db-server
$ netstat -tulpn | grep 5432
# No output = PostgreSQL is not running

# Start PostgreSQL
$ systemctl start postgresql
$ systemctl status postgresql
Active: active (running)

# Test again
$ telnet 10.0.1.50 5432
Connected to 10.0.1.50.
# Success!
```

---

### Network Troubleshooting Flow

```bash
# 1. Can I reach the internet?
$ ping 8.8.8.8              # Google DNS
# YES → Network works, NO → Check local network

# 2. Can I resolve DNS?
$ ping google.com
# YES → DNS works, NO → DNS issue

# 3. Can I reach the service?
$ ping app-server.com
$ telnet app-server.com 80
# YES → Service is up, NO → Service/firewall issue

# 4. What's the full path?
$ traceroute app-server.com
# Shows each hop, helps identify where it fails
```

---

## Shell Scripting (Bash)

### 📖 Simple Explanation

**What is Shell Scripting?**
Automating commands by writing them in a file.

**Instead of:**
```bash
# Type these 10 commands every day:
$ cd /var/log
$ tar -czf backup-$(date +%Y%m%d).tar.gz app.log
$ mv backup-*.tar.gz /backups/
$ find /backups -mtime +30 -delete
...
```

**Write a script:**
```bash
#!/bin/bash
# backup.sh - Run this once daily
cd /var/log
tar -czf backup-$(date +%Y%m%d).tar.gz app.log
mv backup-*.tar.gz /backups/
find /backups -mtime +30 -delete
echo "Backup complete!"
```

---

### Basic Script Structure

```bash
#!/bin/bash
# Shebang: tells Linux this is a bash script

# Variables
NAME="DevOps"
DATE=$(date +%Y-%m-%d)
COUNT=5

# Echo (print)
echo "Hello, $NAME!"
echo "Today is $DATE"

# Conditions
if [ "$COUNT" -gt 3 ]; then
    echo "Count is greater than 3"
else
    echo "Count is 3 or less"
fi

# Loops
for i in 1 2 3 4 5; do
    echo "Number: $i"
done

# While loop
COUNTER=0
while [ $COUNTER -lt 5 ]; do
    echo "Counter: $COUNTER"
    COUNTER=$((COUNTER + 1))
done

# Functions
my_function() {
    echo "Function called with: $1"
}

my_function "argument"
```

---

### Practical Scripts

**1. Backup Script**
```bash
#!/bin/bash
# backup.sh - Backup application logs

BACKUP_DIR="/backups"
LOG_DIR="/var/log/app"
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="app_backup_${DATE}.tar.gz"

echo "Starting backup..."

# Create backup
tar -czf "${BACKUP_DIR}/${BACKUP_FILE}" "${LOG_DIR}"

if [ $? -eq 0 ]; then
    echo "Backup successful: ${BACKUP_FILE}"
    
    # Delete backups older than 30 days
    find "${BACKUP_DIR}" -name "app_backup_*.tar.gz" -mtime +30 -delete
    echo "Old backups cleaned up"
else
    echo "Backup failed!"
    exit 1
fi
```

**2. Service Health Check**
```bash
#!/bin/bash
# health-check.sh - Check if services are running

SERVICES=("nginx" "postgresql" "redis")

for service in "${SERVICES[@]}"; do
    if systemctl is-active --quiet "$service"; then
        echo "✓ $service is running"
    else
        echo "✗ $service is NOT running"
        echo "Attempting to start $service..."
        systemctl start "$service"
        
        if systemctl is-active --quiet "$service"; then
            echo "✓ $service started successfully"
        else
            echo "✗ Failed to start $service"
            # Send alert
            echo "Service $service failed" | mail -s "Alert" admin@example.com
        fi
    fi
done
```

**3. Disk Space Monitor**
```bash
#!/bin/bash
# disk-monitor.sh - Alert if disk usage > 80%

THRESHOLD=80
EMAIL="admin@example.com"

df -H | grep -vE '^Filesystem|tmpfs|cdrom' | awk '{ print $5 " " $1 }' | while read output;
do
    usage=$(echo $output | awk '{ print $1}' | sed 's/%//g')
    partition=$(echo $output | awk '{ print $2 }')
    
    if [ $usage -ge $THRESHOLD ]; then
        echo "Alert: Disk usage on $partition is at ${usage}%" | \
            mail -s "Disk Space Alert: $partition" $EMAIL
    fi
done
```

**4. Log Analyzer**
```bash
#!/bin/bash
# analyze-logs.sh - Find errors in logs

LOG_FILE="/var/log/app/application.log"
REPORT_FILE="/tmp/log-report-$(date +%Y%m%d).txt"

echo "Log Analysis Report - $(date)" > "$REPORT_FILE"
echo "================================" >> "$REPORT_FILE"
echo "" >> "$REPORT_FILE"

# Count errors
ERROR_COUNT=$(grep -c "ERROR" "$LOG_FILE")
echo "Total Errors: $ERROR_COUNT" >> "$REPORT_FILE"
echo "" >> "$REPORT_FILE"

# Top 10 error messages
echo "Top 10 Error Messages:" >> "$REPORT_FILE"
grep "ERROR" "$LOG_FILE" | sort | uniq -c | sort -rn | head -10 >> "$REPORT_FILE"

echo "" >> "$REPORT_FILE"
echo "Report saved to: $REPORT_FILE"
cat "$REPORT_FILE"
```

---

## Package Management

### 📖 Simple Explanation

**Package Manager = App Store for Linux**

Installing software on Linux:
- **Debian/Ubuntu**: `apt` (Advanced Package Tool)
- **RHEL/CentOS/Fedora**: `yum` or `dnf`
- **Others**: Use appropriate package manager

---

### APT (Debian/Ubuntu)

```bash
# Update package list
sudo apt update

# Upgrade all packages
sudo apt upgrade
sudo apt full-upgrade              # also handles dependencies

# Install package
sudo apt install nginx
sudo apt install docker.io kubernetes-kubeadm

# Remove package
sudo apt remove nginx
sudo apt purge nginx               # also remove config files
sudo apt autoremove                # remove unused dependencies

# Search packages
apt search docker
apt show docker.io                 # detailed info

# List installed
apt list --installed
apt list --installed | grep python
```

---

### YUM/DNF (RHEL/CentOS/Fedora)

```bash
# Update package list
sudo yum update                    # yum
sudo dnf update                    # dnf (newer)

# Install package
sudo yum install nginx
sudo dnf install docker

# Remove package
sudo yum remove nginx
sudo dnf remove docker

# Search packages
yum search docker
dnf info docker

# List installed
yum list installed
dnf list installed | grep python
```

---

### Real Example: Install Docker

```bash
# Ubuntu/Debian
$ sudo apt update
$ sudo apt install -y docker.io
$ sudo systemctl start docker
$ sudo systemctl enable docker
$ docker --version
Docker version 24.0.5

# RHEL/CentOS
$ sudo yum install -y docker
$ sudo systemctl start docker
$ sudo systemctl enable docker
$ docker --version
```

---

## System Administration

### 📖 Simple Explanation

**System Admin Tasks:**
- Monitor system health
- Manage users
- Configure startup services
- Schedule tasks
- View system logs

---

### System Monitoring

```bash
# CPU and Memory
top                              # interactive process viewer
htop                             # better alternative
uptime                           # system uptime and load

# Memory usage
free -h                          # human-readable
vmstat 1 5                       # virtual memory stats

# Disk usage
df -h                            # filesystem disk space
du -sh /var/log/*                # directory sizes

# I/O statistics
iostat                           # CPU and I/O stats
iotop                            # I/O by process (requires sudo)
```

---

### User Management

```bash
# Add user
sudo adduser devops
sudo useradd -m -s /bin/bash devops

# Delete user
sudo deluser devops
sudo userdel -r devops           # also remove home directory

# Change password
sudo passwd devops

# Add user to group
sudo usermod -aG docker devops
sudo usermod -aG sudo devops     # give sudo access

# Switch user
su - devops
sudo -u devops command           # run command as user

# List logged in users
who
w
last                             # login history
```

---

### Cron (Scheduled Tasks)

```bash
# Edit crontab
crontab -e                       # edit your crontab
sudo crontab -e                  # edit root's crontab

# List crontab
crontab -l

# Cron syntax
# ┌───────────── minute (0 - 59)
# │ ┌───────────── hour (0 - 23)
# │ │ ┌───────────── day of month (1 - 31)
# │ │ │ ┌───────────── month (1 - 12)
# │ │ │ │ ┌───────────── day of week (0 - 6) (Sunday=0)
# │ │ │ │ │
# * * * * * command to execute

# Examples
0 2 * * * /scripts/backup.sh           # Daily at 2 AM
*/15 * * * * /scripts/health-check.sh   # Every 15 minutes
0 0 * * 0 /scripts/weekly-report.sh    # Every Sunday midnight
0 9-17 * * 1-5 /scripts/business-hours.sh  # Weekdays 9am-5pm
```

**Real Example:**
```bash
# Schedule backup script to run daily at 2 AM
$ crontab -e

# Add this line:
0 2 * * * /home/devops/scripts/backup.sh >> /var/log/backup.log 2>&1

# Verify
$ crontab -l
0 2 * * * /home/devops/scripts/backup.sh >> /var/log/backup.log 2>&1
```

---

### System Logs

```bash
# System logs location
/var/log/syslog                  # Ubuntu/Debian
/var/log/messages                # RHEL/CentOS

# View logs
tail -f /var/log/syslog          # follow system log
journalctl                       # systemd logs
journalctl -f                    # follow
journalctl -u nginx              # specific service
journalctl --since "1 hour ago"
journalctl --since "2024-10-22 10:00:00"

# Application logs
/var/log/nginx/                  # Nginx logs
/var/log/app/                    # Application logs (custom)
```

---

## Common Pitfalls

### ❌ Mistake 1: Using rm -rf Without Checking
```bash
# DANGEROUS!
$ rm -rf /var/log*      # OOPS! Space before *, deletes everything!

# SAFE:
$ ls /var/log*          # Check first
$ rm -rf /var/log/*     # Then delete
```

### ❌ Mistake 2: Not Using sudo When Needed
```bash
$ systemctl restart nginx
Failed to restart nginx.service: Access denied

# Correct:
$ sudo systemctl restart nginx
```

### ❌ Mistake 3: Forgetting to Make Scripts Executable
```bash
$ ./script.sh
bash: ./script.sh: Permission denied

# Fix:
$ chmod +x script.sh
```

### ❌ Mistake 4: Not Escaping Special Characters
```bash
# Won't work as expected:
$ grep $USER app.log        # Shell expands $USER first

# Correct:
$ grep '\$USER' app.log
$ grep '$USER' app.log      # Single quotes prevent expansion
```

---

## Best Practices

### ✅ 1. Always Test Commands First
```bash
# Before:
$ rm -rf /var/log/*.log

# Test with ls first:
$ ls /var/log/*.log
/var/log/app.log
/var/log/nginx.log

# Then proceed
$ rm -rf /var/log/*.log
```

### ✅ 2. Use -i for Interactive Mode
```bash
$ rm -i file.txt         # Ask before deleting
$ cp -i src dest         # Ask before overwriting
$ mv -i old new          # Ask before overwriting
```

### ✅ 3. Always Backup Before Modifying
```bash
$ sudo cp /etc/nginx/nginx.conf /etc/nginx/nginx.conf.backup
$ sudo vim /etc/nginx/nginx.conf
```

### ✅ 4. Use Absolute Paths in Scripts
```bash
# Bad:
cd logs
rm *.log

# Good:
cd /var/log/app || exit 1
rm /var/log/app/*.log
```

### ✅ 5. Add Error Handling in Scripts
```bash
#!/bin/bash
set -e  # Exit on any error
set -u  # Exit on undefined variable
set -o pipefail  # Exit on pipe failure

# Or check exit codes:
command
if [ $? -ne 0 ]; then
    echo "Command failed"
    exit 1
fi
```

---

## Interview Tips

### 🎯 Most Asked Questions

**Q1: How do you find a process and kill it?**
```bash
# Answer:
ps aux | grep process-name
# Or
pgrep process-name

# Kill gracefully
kill <PID>

# If stuck, force kill
kill -9 <PID>

# Or by name
pkill process-name
```

**Q2: How do you check disk space?**
```bash
# Answer:
df -h           # Overall disk usage
du -sh /var/*   # Usage by directory
```

**Q3: How do you check if a port is listening?**
```bash
# Answer:
netstat -tulpn | grep :8080
# Or
ss -tulpn | grep :8080
# Or
lsof -i :8080
```

**Q4: How do you view logs in real-time?**
```bash
# Answer:
tail -f /var/log/app.log
# Or
journalctl -u service-name -f
```

**Q5: Explain file permissions 755 vs 644**
```
755 = rwxr-xr-x
- Owner: read, write, execute (7 = 4+2+1)
- Group: read, execute (5 = 4+0+1)
- Others: read, execute (5 = 4+0+1)
Common for scripts/executables

644 = rw-r--r--
- Owner: read, write (6 = 4+2+0)
- Group: read only (4 = 4+0+0)
- Others: read only (4 = 4+0+0)
Common for config files
```

---

## Real-World Scenario

### Production Server Troubleshooting

**Scenario:** Web application is slow, users complaining.

**Step-by-Step Debugging:**

```bash
# 1. Check system resources
$ top
# CPU at 95%, memory OK

# 2. Identify the problematic process
$ ps aux | sort -k 3 -rn | head -5
www-data  1234  95.2  2.3  /usr/bin/app-server

# 3. Check what it's doing
$ strace -p 1234
# Shows it's making lots of database calls

# 4. Check network connections
$ netstat -an | grep ESTABLISHED | wc -l
1523
# Too many connections!

# 5. Check application logs
$ tail -100 /var/log/app/error.log
Database connection pool exhausted
Database connection pool exhausted
Database connection pool exhausted

# 6. Check database server
$ ssh db-server
$ psql -c "SELECT count(*) FROM pg_stat_activity;"
 count 
-------
   500
# Max connections is 100, something's wrong!

# 7. Fix: Restart application server
$ systemctl restart app-server

# 8. Monitor
$ watch -n 1 'netstat -an | grep ESTABLISHED | wc -l'
# Connection count stabilizing

# 9. Check logs
$ tail -f /var/log/app/error.log
# No more errors

# 10. Long-term fix
# Edit config to increase connection pool timeout
$ vim /etc/app/config.yaml
database:
  pool_size: 50
  timeout: 30s

# Restart with new config
$ systemctl restart app-server
```

---

## Chapter Summary

### What You Learned

**1. Essential Commands**
- ✅ Navigation: pwd, cd, ls
- ✅ File operations: cp, mv, rm, cat, grep
- ✅ Process management: ps, top, kill, systemctl
- ✅ Networking: ping, netstat, curl, ssh

**2. File System**
- ✅ Linux directory structure (/etc, /var, /home)
- ✅ File permissions (755, 644, chmod, chown)
- ✅ Permission calculation (r=4, w=2, x=1)

**3. Process Management**
- ✅ Start/stop services with systemd
- ✅ View logs with journalctl
- ✅ Kill stuck processes
- ✅ Run processes in background

**4. Networking**
- ✅ Check connectivity (ping, traceroute)
- ✅ DNS resolution (nslookup, dig)
- ✅ Port listening (netstat, ss, lsof)
- ✅ Download files (curl, wget)

**5. Shell Scripting**
- ✅ Variables and conditionals
- ✅ Loops and functions
- ✅ Practical automation scripts
- ✅ Error handling

**6. System Administration**
- ✅ User management
- ✅ Cron jobs for scheduling
- ✅ System monitoring
- ✅ Log management

---

### 🎯 Interview Quick Tips

**Top 5 Must-Know Commands:**
1. `grep` - Search in files
2. `find` - Find files
3. `ps aux` - List processes
4. `systemctl` - Manage services
5. `tail -f` - Follow logs

**Most Common Interview Scenarios:**
1. "Server is slow" → Check top, ps aux, disk space
2. "Service won't start" → Check systemctl status, journalctl
3. "Can't connect to database" → Check ping, telnet, firewall
4. "Disk is full" → Check df -h, du -sh, clean up

---

### 💡 Real-World Tips

**Daily DevOps Tasks:**
```bash
# Morning routine
$ ssh production-server
$ df -h                      # Check disk space
$ free -h                    # Check memory
$ top                        # Check CPU
$ systemctl status app       # Check services
$ tail -50 /var/log/app.log  # Check recent logs
```

**Quick Health Check Script:**
```bash
#!/bin/bash
echo "=== System Health ==="
echo "Disk: $(df -h / | tail -1 | awk '{print $5}') used"
echo "Memory: $(free -h | grep Mem | awk '{print $3 "/" $2}')"
echo "Load: $(uptime | awk -F'load average:' '{print $2}')"
echo "Services:"
systemctl is-active nginx && echo "  ✓ nginx" || echo "  ✗ nginx"
systemctl is-active docker && echo "  ✓ docker" || echo "  ✗ docker"
```

---

### 📚 What's Next?

Now that you have Linux fundamentals, you're ready for:
- **Docker** (runs on Linux)
- **Kubernetes** (Linux containers orchestration)
- **CI/CD** (Jenkins, GitLab on Linux servers)
- **Cloud** (AWS EC2, Azure VMs are Linux)

**Practice Suggestions:**
1. Set up a Linux VM (Ubuntu or CentOS)
2. Practice all commands in this chapter
3. Write 5 automation scripts
4. Schedule cron jobs
5. Troubleshoot intentionally broken services

---

### 📝 Practice Exercises

1. **File Operations**
   - Create 100 files, rename them, delete odd-numbered ones
   - Find all .log files larger than 10MB
   - Change permissions of all scripts to 755

2. **Process Management**
   - Start nginx, monitor with top, restart it
   - Run a command in background, bring to foreground
   - Schedule a task to run every hour

3. **Scripting**
   - Write backup script with error handling
   - Write service health check script
   - Write disk space monitor with email alerts

4. **Troubleshooting**
   - Set up a web server, intentionally break it, fix it
   - Simulate high CPU, identify and kill the process
   - Debug network connectivity issues

---

**Next Chapter:** [Docker & Containers](02-docker-containers.md)

**Prerequisites Covered:** ✅ You're now ready for Docker!

