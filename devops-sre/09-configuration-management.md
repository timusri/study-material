# 9. Configuration Management

## 📚 Quick Summary

Configuration Management ensures servers are configured consistently and maintained automatically!

**What You'll Learn:**
- **CM Fundamentals**: Push vs pull models, idempotency
- **Ansible Deep Dive**: Playbooks, roles, best practices
- **Chef**: Ruby-based configuration management
- **Puppet**: Enterprise-grade configuration management  
- **Salt**: High-speed configuration management
- **Comparison**: When to use which tool
- **Best Practices**: Version control, testing, secrets

**Why This Matters:**
- Manage 1000s of servers consistently
- Eliminate configuration drift
- Automate repetitive tasks
- Ensure compliance
- Interview questions: 15% are CM-based

**Interview Reality:**
"How do you ensure 500 servers have identical configuration?" = Configuration Management!

---

## 📖 Simple Explanation

**What is Configuration Management?**

**Before CM:**
```
1. SSH into each server (500 servers)
2. Install packages manually
3. Edit config files manually  
4. Restart services manually
5. Hope you didn't miss anything
6. Servers drift over time
😰
```

**With CM:**
```
1. Write configuration once in code
2. Run CM tool
3. All servers configured identically
4. Automatic, consistent, repeatable
5. Self-healing (drift correction)
😊
```

**Real-World Analogy:**
```
Without CM = Telling each student individually
- Time consuming
- Inconsistent messages
- Easy to forget someone

With CM = School announcement system
- Broadcast once
- Everyone gets same message
- Consistent communication
- Automated
```

---

**Push vs Pull Models:**

```
Push Model (Ansible):
┌─────────┐         ┌─────────┐
│ Control │ Push → │ Server1 │
│  Node   │ ─────→ │ Server2 │
│         │ ─────→ │ Server3 │
└─────────┘         └─────────┘

+ Simple setup (agentless)
+ Immediate execution
+ Good for small/medium scale
- Control node must reach all servers

Pull Model (Chef, Puppet):
┌─────────┐         ┌─────────┐
│  Chef   │ ← Pull  │ Server1 │
│ Server  │ ←────── │ Server2 │
│         │ ←────── │ Server3 │
└─────────┘         └─────────┘

+ Scales to 10,000+ nodes
+ Autonomous (nodes pull updates)
+ Better for large enterprises
- Requires agent on each node
```

---

## Table of Contents
- [Configuration Management Fundamentals](#configuration-management-fundamentals)
- [Ansible Advanced](#ansible-advanced)
- [Chef](#chef)
- [Puppet](#puppet)
- [SaltStack](#saltstack)
- [Tool Comparison](#tool-comparison)
- [Design Patterns](#design-patterns)
- [Testing Configuration](#testing-configuration)
- [Secrets Management](#secrets-management)
- [Best Practices](#best-practices)
- [Interview Questions](#interview-questions)

---

## Configuration Management Fundamentals

### Core Principles

**1. Idempotency**
```
Running same configuration multiple times = Same result

Example:
Run 1: Install nginx → nginx installed ✓
Run 2: Install nginx → already installed, skip ✓
Run 3: Install nginx → already installed, skip ✓

Non-idempotent (BAD):
echo "config" >> /etc/myapp.conf
# Appends every time! ✗

Idempotent (GOOD):
lineinfile:
  path: /etc/myapp.conf
  line: "config"
# Only adds if not present ✓
```

**2. Convergence**
```
System gradually moves toward desired state

Current: nginx v1.18
Desired: nginx v1.20

CM tool converges system to desired state
```

**3. Declarative**
```
Define WHAT you want, not HOW

✓ GOOD (Declarative):
- name: Ensure nginx is installed
  package:
    name: nginx
    state: present

✗ BAD (Imperative):
- shell: |
    if ! rpm -q nginx; then
      yum install -y nginx
    fi
```

---

## Ansible Advanced

### Complex Playbooks

```yaml
# site.yml - Main playbook
---
- name: Configure Web Tier
  hosts: webservers
  become: yes
  
  pre_tasks:
    - name: Update cache
      apt:
        update_cache: yes
        cache_valid_time: 3600
  
  roles:
    - common
    - security
    - nginx
    - app
    - monitoring
  
  post_tasks:
    - name: Verify deployment
      uri:
        url: "http://localhost"
        status_code: 200

- name: Configure Database Tier
  hosts: databases
  become: yes
  serial: 1  # One at a time
  
  roles:
    - common
    - security
    - mysql
    - backup
```

---

### Ansible Variables (Advanced)

```yaml
# group_vars/all/main.yml
---
app_name: myapp
app_version: "{{ lookup('env', 'APP_VERSION') | default('1.0.0') }}"
app_user: "{{ app_name }}"
app_home: "/opt/{{ app_name }}"

# group_vars/webservers/main.yml
---
nginx_workers: 4
nginx_connections: 1024

# host_vars/web1.example.com/main.yml
---
nginx_workers: 8  # Override for this specific host

# Using variables
- name: Create app user
  user:
    name: "{{ app_user }}"
    home: "{{ app_home }}"
    system: yes

- name: Deploy app
  template:
    src: app.conf.j2
    dest: "{{ app_home }}/config/app.conf"
```

---

### Jinja2 Templates

```jinja2
{# templates/nginx.conf.j2 #}
user {{ nginx_user }};
worker_processes {{ nginx_workers }};

events {
    worker_connections {{ nginx_connections }};
}

http {
    upstream backend {
        {% for host in groups['webservers'] %}
        server {{ hostvars[host]['ansible_default_ipv4']['address'] }}:8080;
        {% endfor %}
    }
    
    server {
        listen 80;
        server_name {{ server_name }};
        
        {% if ssl_enabled %}
        listen 443 ssl;
        ssl_certificate {{ ssl_cert_path }};
        ssl_certificate_key {{ ssl_key_path }};
        {% endif %}
        
        location / {
            proxy_pass http://backend;
        }
    }
}
```

---

### Ansible Conditionals

```yaml
- name: Install Apache (RedHat)
  yum:
    name: httpd
    state: present
  when: ansible_os_family == "RedHat"

- name: Install Apache (Debian)
  apt:
    name: apache2
    state: present
  when: ansible_os_family == "Debian"

- name: Create swap (if memory < 2GB)
  command: dd if=/dev/zero of=/swapfile bs=1M count=2048
  when: ansible_memtotal_mb < 2048

- name: Deploy to production
  shell: ./deploy.sh
  when:
    - environment == "production"
    - deploy_enabled | bool
    - ansible_date_time.weekday != "Friday"  # Never deploy on Friday!
```

---

### Ansible Loops

```yaml
# Simple loop
- name: Install packages
  apt:
    name: "{{ item }}"
    state: present
  loop:
    - nginx
    - git
    - vim
    - htop

# Loop with dict
- name: Create users
  user:
    name: "{{ item.name }}"
    state: present
    groups: "{{ item.groups }}"
  loop:
    - { name: 'alice', groups: 'admin,developers' }
    - { name: 'bob', groups: 'developers' }
    - { name: 'charlie', groups: 'ops' }

# Loop with subelements
- name: Add SSH keys
  authorized_key:
    user: "{{ item.0.name }}"
    key: "{{ item.1 }}"
  loop: "{{ users | subelements('ssh_keys') }}"

# Loop with until (retry)
- name: Wait for service
  uri:
    url: "http://localhost:8080/health"
    status_code: 200
  register: result
  until: result.status == 200
  retries: 10
  delay: 5
```

---

### Ansible Error Handling

```yaml
- name: Try to start service
  service:
    name: myapp
    state: started
  register: result
  ignore_errors: yes

- name: Debug failure
  debug:
    msg: "Service failed: {{ result.msg }}"
  when: result.failed

- name: Rollback on failure
  shell: /opt/rollback.sh
  when: result.failed

# Block error handling
- name: Deploy application
  block:
    - name: Copy files
      copy:
        src: app.tar.gz
        dest: /opt/
    
    - name: Extract
      unarchive:
        src: /opt/app.tar.gz
        dest: /opt/myapp/
    
    - name: Start service
      service:
        name: myapp
        state: started
  
  rescue:
    - name: Rollback
      shell: /opt/rollback.sh
    
    - name: Notify team
      slack:
        msg: "Deployment failed, rolled back"
  
  always:
    - name: Clean up
      file:
        path: /tmp/deployment
        state: absent
```

---

### Ansible Strategies

```yaml
# Serial execution (one by one)
- hosts: databases
  serial: 1  # Or "30%" for percentage
  tasks:
    - name: Upgrade database
      shell: /opt/upgrade.sh

# Free strategy (no waiting)
- hosts: webservers
  strategy: free  # Don't wait for slow hosts
  tasks:
    - name: Clear cache
      shell: rm -rf /var/cache/*

# Batch execution
- hosts: webservers
  serial:
    - 1      # First host
    - 25%    # Then 25% of remaining
    - 100%   # Then all remaining
```

---

## Chef

### 📖 Simple Explanation

Chef uses Ruby DSL to define infrastructure. Enterprise-focused with rich ecosystem.

---

### Chef Architecture

```
┌──────────────┐
│  Chef Server │  Central configuration hub
└──────┬───────┘
       │
       │ (Pull configuration)
       │
┌──────┴───────┬──────────┬──────────┐
│   Node 1     │  Node 2  │  Node 3  │
│ (Chef Client)│          │          │
└──────────────┴──────────┴──────────┘
```

---

### Chef Cookbook

**Directory Structure:**
```
cookbooks/
└── nginx/
    ├── recipes/
    │   ├── default.rb
    │   └── install.rb
    ├── templates/
    │   └── nginx.conf.erb
    ├── attributes/
    │   └── default.rb
    ├── files/
    │   └── index.html
    └── metadata.rb
```

---

**Recipe (recipes/default.rb):**
```ruby
# Install nginx
package 'nginx' do
  action :install
end

# Configure nginx
template '/etc/nginx/nginx.conf' do
  source 'nginx.conf.erb'
  owner 'root'
  group 'root'
  mode '0644'
  variables(
    worker_processes: node['nginx']['workers'],
    worker_connections: node['nginx']['connections']
  )
  notifies :restart, 'service[nginx]', :delayed
end

# Ensure nginx is running
service 'nginx' do
  action [:enable, :start]
  supports restart: true, reload: true
end

# Deploy application
remote_file '/var/www/html/index.html' do
  source 'http://example.com/app.html'
  owner 'www-data'
  group 'www-data'
  mode '0644'
end
```

---

**Attributes (attributes/default.rb):**
```ruby
default['nginx']['workers'] = 4
default['nginx']['connections'] = 1024
default['nginx']['port'] = 80

# Override per environment
case node.chef_environment
when 'production'
  default['nginx']['workers'] = 8
  default['nginx']['connections'] = 2048
when 'development'
  default['nginx']['workers'] = 2
  default['nginx']['connections'] = 512
end
```

---

**Template (templates/nginx.conf.erb):**
```erb
user www-data;
worker_processes <%= @worker_processes %>;

events {
    worker_connections <%= @worker_connections %>;
}

http {
    include /etc/nginx/mime.types;
    
    <% if node['nginx']['ssl_enabled'] %>
    ssl_certificate <%= node['nginx']['ssl_cert'] %>;
    ssl_certificate_key <%= node['nginx']['ssl_key'] %>;
    <% end %>
    
    server {
        listen <%= node['nginx']['port'] %>;
        server_name <%= node['hostname'] %>;
        
        location / {
            root /var/www/html;
            index index.html;
        }
    }
}
```

---

### Chef Resources

```ruby
# File management
file '/etc/motd' do
  content 'Welcome to our server!'
  owner 'root'
  group 'root'
  mode '0644'
end

# Directory
directory '/opt/myapp' do
  owner 'app'
  group 'app'
  mode '0755'
  recursive true
end

# Execute command
execute 'update-system' do
  command 'apt-get update'
  action :run
  not_if { File.exist?('/var/cache/apt/updated') }
end

# Git repository
git '/opt/myapp' do
  repository 'https://github.com/user/repo.git'
  revision 'main'
  action :sync
end

# Cron job
cron 'backup' do
  minute '0'
  hour '2'
  command '/opt/backup.sh'
  user 'root'
end
```

---

### Chef Search

```ruby
# Find all web servers
web_servers = search('node', 'role:webserver AND chef_environment:production')

web_servers.each do |server|
  log "Found server: #{server['hostname']}"
end

# Generate load balancer config
template '/etc/haproxy/haproxy.cfg' do
  source 'haproxy.cfg.erb'
  variables(
    servers: search('node', 'role:webserver')
  )
end
```

---

## Puppet

### 📖 Simple Explanation

Puppet is enterprise-grade CM with declarative language and strong ecosystem.

---

### Puppet Manifest

```puppet
# site.pp
node 'web1.example.com' {
  include nginx
  include myapp
}

node /^web\d+\.example\.com$/ {
  include nginx
  include myapp
}

node default {
  include base
}
```

---

### Puppet Module

```puppet
# modules/nginx/manifests/init.pp
class nginx (
  $worker_processes = 4,
  $worker_connections = 1024,
  $port = 80,
) {
  
  # Install nginx
  package { 'nginx':
    ensure => installed,
  }
  
  # Configure nginx
  file { '/etc/nginx/nginx.conf':
    ensure  => file,
    content => template('nginx/nginx.conf.erb'),
    owner   => 'root',
    group   => 'root',
    mode    => '0644',
    require => Package['nginx'],
    notify  => Service['nginx'],
  }
  
  # Ensure service is running
  service { 'nginx':
    ensure    => running,
    enable    => true,
    subscribe => File['/etc/nginx/nginx.conf'],
  }
}
```

---

### Puppet Resources

```puppet
# File
file { '/etc/motd':
  ensure  => file,
  content => 'Welcome!',
  owner   => 'root',
  group   => 'root',
  mode    => '0644',
}

# User
user { 'deploy':
  ensure     => present,
  home       => '/home/deploy',
  shell      => '/bin/bash',
  managehome => true,
}

# Exec
exec { 'update-cache':
  command => '/usr/bin/apt-get update',
  unless  => '/usr/bin/test -f /var/cache/apt/updated',
}

# Package
package { ['git', 'vim', 'htop']:
  ensure => installed,
}

# Service
service { 'nginx':
  ensure => running,
  enable => true,
}
```

---

### Puppet Ordering

```puppet
# Using arrows
Package['nginx'] -> File['/etc/nginx/nginx.conf'] -> Service['nginx']

# Using require
file { '/etc/nginx/nginx.conf':
  require => Package['nginx'],
}

service { 'nginx':
  require => File['/etc/nginx/nginx.conf'],
}

# Using before
package { 'nginx':
  before => File['/etc/nginx/nginx.conf'],
}

# Using notify (trigger refresh)
file { '/etc/nginx/nginx.conf':
  notify => Service['nginx'],  # Restart service if file changes
}
```

---

## SaltStack

### 📖 Simple Explanation

Salt is high-speed CM tool with both push and pull modes. Great for large scale.

---

### Salt States

```yaml
# /srv/salt/nginx/init.sls
nginx:
  pkg.installed: []
  
  service.running:
    - enable: True
    - reload: True
    - watch:
      - file: /etc/nginx/nginx.conf

/etc/nginx/nginx.conf:
  file.managed:
    - source: salt://nginx/files/nginx.conf
    - user: root
    - group: root
    - mode: 644
    - template: jinja
    - context:
        workers: {{ salt['grains.get']('num_cpus') }}
    - require:
      - pkg: nginx

# Deploy application
/var/www/html/index.html:
  file.managed:
    - source: salt://nginx/files/index.html
    - user: www-data
    - group: www-data
    - mode: 644
```

---

### Salt Commands

```bash
# Test connectivity
salt '*' test.ping

# Run command
salt '*' cmd.run 'uptime'

# Install package
salt '*' pkg.install nginx

# Apply state
salt '*' state.apply nginx

# Grains (system facts)
salt '*' grains.items
salt 'web*' grains.get os

# Pillar (configuration data)
salt '*' pillar.items

# Target by grain
salt -G 'os:Ubuntu' state.apply

# Target by compound
salt -C 'G@os:Ubuntu and web* or E@db[0-9]' test.ping
```

---

### Salt Pillar

```yaml
# /srv/pillar/top.sls
base:
  'web*':
    - nginx
  'db*':
    - mysql

# /srv/pillar/nginx.sls
nginx:
  workers: 4
  connections: 1024
  ssl_enabled: True
  
users:
  alice:
    uid: 1001
    groups: [wheel, developers]
  bob:
    uid: 1002
    groups: [developers]
```

---

## Tool Comparison

```
┌─────────────┬─────────┬───────┬────────┬──────┐
│ Feature     │ Ansible │ Chef  │ Puppet │ Salt │
├─────────────┼─────────┼───────┼────────┼──────┤
│ Agent       │ No      │ Yes   │ Yes    │ Opt. │
│ Language    │ YAML    │ Ruby  │ Puppet │ YAML │
│ Learning    │ Easy    │ Hard  │ Medium │ Med. │
│ Speed       │ Slow    │ Fast  │ Medium │ Fast │
│ Scale       │ 100s    │ 1000s │ 1000s  │ 10K+ │
│ Windows     │ Yes     │ Yes   │ Yes    │ Yes  │
│ Community   │ Large   │ Large │ Large  │ Med. │
└─────────────┴─────────┴───────┴────────┴──────┘

Use Cases:
- Ansible: Cloud, simple setups, agentless
- Chef: Enterprise, complex workflows, Ruby shops
- Puppet: Enterprise, Windows, compliance
- Salt: Massive scale, event-driven, real-time
```

---

**When to Use What:**

```
Ansible:
✓ Starting new project
✓ < 1000 servers
✓ Cloud-native
✓ Quick automation
✓ No agent requirement

Chef:
✓ Ruby developers
✓ Complex business logic
✓ Large enterprise
✓ Mature ecosystem

Puppet:
✓ Heavy Windows environment
✓ Compliance requirements
✓ Large scale
✓ Strong typing needed

Salt:
✓ Massive scale (10,000+ nodes)
✓ Real-time execution needed
✓ Event-driven architecture
✓ High performance critical
```

---

## Design Patterns

### 1. Role-Based Configuration

```yaml
# Ansible
---
- hosts: webservers
  roles:
    - common      # All servers
    - security    # All servers
    - nginx       # Web servers only
    - app         # Web servers only

- hosts: databases
  roles:
    - common
    - security
    - mysql       # DB servers only
    - backup      # DB servers only
```

---

### 2. Environment Separation

```
environments/
├── dev/
│   ├── group_vars/
│   └── inventory
├── staging/
│   ├── group_vars/
│   └── inventory
└── prod/
    ├── group_vars/
    └── inventory

# Run for specific environment
ansible-playbook -i environments/prod/inventory site.yml
```

---

### 3. Secrets Management

```yaml
# Ansible Vault
---
# Create encrypted file
ansible-vault create group_vars/all/vault.yml

# vault.yml (encrypted)
vault_db_password: "super_secret"
vault_api_key: "abc123"

# vars.yml (plain, references vault)
db_password: "{{ vault_db_password }}"
api_key: "{{ vault_api_key }}"

# Use in tasks
- name: Configure database
  template:
    src: db.conf.j2
    dest: /etc/db.conf
  vars:
    password: "{{ db_password }}"
```

---

## Testing Configuration

### Ansible Testing

**Syntax Check:**
```bash
ansible-playbook playbook.yml --syntax-check
```

**Dry Run:**
```bash
ansible-playbook playbook.yml --check --diff
```

**Ansible Lint:**
```bash
pip install ansible-lint
ansible-lint playbook.yml
```

**Molecule (Testing Framework):**
```bash
# Install
pip install molecule molecule-docker

# Initialize
molecule init scenario

# Test lifecycle
molecule create    # Create test instance
molecule converge  # Apply playbook
molecule verify    # Run tests
molecule destroy   # Clean up

# Full test
molecule test
```

---

**Molecule Configuration:**
```yaml
# molecule/default/molecule.yml
---
driver:
  name: docker

platforms:
  - name: ubuntu-20
    image: ubuntu:20.04
    pre_build_image: true

provisioner:
  name: ansible
  playbooks:
    converge: converge.yml

verifier:
  name: testinfra
```

**Test File:**
```python
# molecule/default/tests/test_default.py
def test_nginx_installed(host):
    nginx = host.package('nginx')
    assert nginx.is_installed

def test_nginx_running(host):
    nginx = host.service('nginx')
    assert nginx.is_running
    assert nginx.is_enabled

def test_nginx_listening(host):
    assert host.socket('tcp://0.0.0.0:80').is_listening
```

---

### Chef Testing

**ChefSpec (Unit Tests):**
```ruby
# spec/unit/recipes/default_spec.rb
require 'chefspec'

describe 'nginx::default' do
  let(:chef_run) { ChefSpec::SoloRunner.new.converge(described_recipe) }
  
  it 'installs nginx package' do
    expect(chef_run).to install_package('nginx')
  end
  
  it 'starts nginx service' do
    expect(chef_run).to start_service('nginx')
  end
  
  it 'creates nginx config' do
    expect(chef_run).to create_template('/etc/nginx/nginx.conf')
  end
end
```

**Test Kitchen (Integration Tests):**
```yaml
# .kitchen.yml
---
driver:
  name: docker

provisioner:
  name: chef_zero

platforms:
  - name: ubuntu-20.04

suites:
  - name: default
    run_list:
      - recipe[nginx::default]
```

```bash
kitchen test
```

---

## Secrets Management

### Best Practices

```yaml
# ✅ GOOD: Use vault
---
db_password: "{{ vault_db_password }}"

# ❌ BAD: Hardcoded
---
db_password: "password123"

# ✅ GOOD: External secret manager
---
- name: Get secret from AWS
  set_fact:
    db_password: "{{ lookup('aws_ssm', '/prod/db/password') }}"

# ✅ GOOD: Environment variables
---
db_password: "{{ lookup('env', 'DB_PASSWORD') }}"
```

---

### HashiCorp Vault Integration

```yaml
# Ansible with Vault
---
- name: Get secrets from Vault
  set_fact:
    db_creds: "{{ lookup('hashi_vault', 'secret/data/db/prod') }}"

- name: Configure database
  template:
    src: db.conf.j2
    dest: /etc/db.conf
  vars:
    username: "{{ db_creds.data.data.username }}"
    password: "{{ db_creds.data.data.password }}"
```

---

## Best Practices

### 1. Version Control Everything

```bash
# ✅ Store in Git
git/
├── playbooks/
├── roles/
├── inventories/
└── group_vars/

# ❌ Don't commit secrets
.gitignore:
*vault*password*
*.pem
*.key
```

---

### 2. Idempotency

```yaml
# ✅ Idempotent
- name: Ensure line in file
  lineinfile:
    path: /etc/hosts
    line: "192.168.1.1 example.com"
    state: present

# ❌ Not idempotent
- name: Append to file
  shell: echo "192.168.1.1 example.com" >> /etc/hosts
```

---

### 3. Use Roles/Modules

```yaml
# ✅ Organized with roles
---
- hosts: webservers
  roles:
    - nginx
    - app

# ❌ Everything in one playbook
---
- hosts: webservers
  tasks:
    - name: Install nginx
      apt: name=nginx
    # 500 more tasks...
```

---

### 4. Test Before Production

```bash
# Always test in order:
1. Syntax check
2. Dry run (--check)
3. Dev environment
4. Staging environment
5. Production (slowly)
```

---

### 5. Documentation

```yaml
---
# Role: nginx
# Description: Installs and configures Nginx web server
# Variables:
#   nginx_workers: Number of worker processes (default: 4)
#   nginx_port: Listen port (default: 80)

- name: Install Nginx
  apt:
    name: nginx
    state: present
```

---

## Interview Questions

### Q1: Push vs Pull in Configuration Management?
**Answer:**
```
Push (Ansible):
- Control node pushes config to servers
- Agentless (uses SSH)
- Immediate execution
- Good for: < 1000 servers, ad-hoc tasks

Pull (Chef, Puppet):
- Agents on nodes pull config from server
- Autonomous operation
- Scalable to 10,000+ nodes
- Good for: Large enterprise, continuous enforcement

Hybrid (Salt):
- Supports both push and pull
- Best of both worlds
```

---

### Q2: How do you ensure idempotency?
**Answer:**
```
Idempotency = Same result regardless of times run

Techniques:
1. Use built-in modules (not shell)
   ✓ apt: name=nginx state=present
   ✗ shell: apt-get install nginx

2. Check before action
   - name: Create file
     file: path=/opt/file state=touch
     creates: /opt/file

3. Use state declarations
   service: name=nginx state=started

4. Test by running multiple times
   ansible-playbook site.yml  # Run 3 times
   # Should show: ok=N changed=0
```

---

### Q3: How do you manage secrets?
**Answer:**
```
Options:
1. Ansible Vault
   ansible-vault encrypt secrets.yml
   
2. External Secret Managers
   - HashiCorp Vault
   - AWS Secrets Manager
   - Azure Key Vault
   
3. Environment Variables
   db_pass: "{{ lookup('env', 'DB_PASS') }}"

4. CI/CD Secrets
   - GitHub Secrets
   - GitLab CI Variables

Never:
✗ Commit secrets to Git
✗ Hardcode in playbooks
✗ Store in plain text
```

---

### Q4: Ansible vs Chef vs Puppet?
**Answer:**
```
Ansible:
+ Agentless (SSH)
+ Easy to learn (YAML)
+ Quick to start
- Slower for large scale
- Limited Windows support initially

Chef:
+ Very scalable
+ Ruby DSL (powerful)
+ Mature ecosystem
- Steep learning curve
- Requires agents

Puppet:
+ Enterprise features
+ Great Windows support
+ Strong compliance tools
- Complex syntax
- Requires agents

Recommendation:
- New project: Ansible
- Enterprise: Chef or Puppet
- Massive scale: Salt
```

---

### Q5: How do you test configuration management code?
**Answer:**
```
Testing Pyramid:

1. Syntax/Lint
   ansible-lint playbook.yml
   rubocop cookbook.rb

2. Unit Tests
   - ChefSpec (Chef)
   - rspec-puppet (Puppet)
   - Molecule (Ansible)

3. Integration Tests
   - Test Kitchen (Chef)
   - Kitchen (Puppet)
   - Molecule (Ansible)

4. Staging Environment
   - Test on staging first
   - Verify all changes

5. Canary Production
   - Deploy to 1 server first
   - Monitor metrics
   - Gradually roll out

Full workflow:
lint → unit tests → integration tests → staging → canary → production
```

---

### Q6: Handling configuration drift?
**Answer:**
```
Configuration Drift = Servers diverge from desired state

Solutions:

1. Continuous Enforcement
   # Run CM tool regularly
   cron: */15 * * * * ansible-playbook site.yml

2. Pull Model (Chef/Puppet)
   # Agents check every 30 min
   # Auto-correct drift

3. Immutable Infrastructure
   # Don't fix servers, replace them
   # Kubernetes pods
   # Auto-scaling groups

4. Monitoring
   # Alert on drift
   # Use tools like OSQuery

5. Compliance Tools
   # InSpec (Chef)
   # OpenSCAP
   # Regularly audit

Best Practice: Immutable infrastructure > Drift correction
```

---

## Quick Reference

```bash
# Ansible
ansible all -m ping
ansible-playbook site.yml
ansible-playbook site.yml --check
ansible-vault encrypt secrets.yml

# Chef
knife node list
knife ssh 'role:webserver' 'chef-client'
kitchen test

# Puppet
puppet agent --test
puppet apply site.pp
puppet module install nginx

# Salt
salt '*' test.ping
salt '*' state.apply
salt '*' cmd.run 'uptime'
```

---

## Summary

**Key Takeaways:**
1. CM ensures consistent server configuration
2. Ansible: agentless, easy, YAML
3. Chef/Puppet: enterprise, scalable, agents
4. Idempotency is critical
5. Test before production
6. Version control everything
7. Never commit secrets

**Next Steps:**
1. Choose CM tool for your needs
2. Write first playbook/cookbook
3. Organize with roles/modules
4. Implement testing
5. Set up secrets management
6. Automate everything

**Remember:**
- Idempotency first
- Test extensively
- Document well
- Secure secrets
- Version control

**Happy Automating! 🚀**

