# 8. Infrastructure as Code (IaC)

## 📚 Quick Summary

Infrastructure as Code transforms manual infrastructure management into version-controlled, automated code!

**What You'll Learn:**
- **IaC Fundamentals**: Declarative vs imperative, benefits, principles
- **Terraform**: Most popular IaC tool, provider ecosystem
- **Ansible**: Configuration management and orchestration
- **CloudFormation**: AWS-native IaC
- **Pulumi**: Modern IaC with real programming languages
- **Best Practices**: State management, modules, testing
- **GitOps**: Infrastructure through Git workflows

**Why This Matters:**
- Manage 1000s of servers with code (not clicks)
- Infrastructure version control and rollback
- Consistent environments (dev = staging = prod)
- Disaster recovery in minutes
- Interview questions: 25% are IaC-based

**Interview Reality:**
"How do you manage infrastructure?" = IaC is the expected answer!

---

## 📖 Simple Explanation

**What is Infrastructure as Code?**

**Before IaC (Manual):**
```
1. Log into AWS console
2. Click "Create EC2 Instance"
3. Fill 20 form fields
4. Repeat for 100 servers
5. Document in Word/Excel
6. Hope you remember next time
😰
```

**With IaC:**
```
1. Write infrastructure in code
2. Run: terraform apply
3. Infrastructure created automatically
4. Version controlled in Git
5. Reproducible anywhere, anytime
😊
```

**Real-World Analogy:**
```
Manual Infrastructure = Building house by hand
- Slow
- Error-prone
- Hard to replicate
- No blueprint

IaC = Building with blueprints
- Fast
- Consistent
- Easy to replicate
- Versioned plans
```

---

**Key Benefits:**

```
1. Version Control
   - Track all changes
   - Rollback capability
   - Audit trail

2. Automation
   - No manual steps
   - Fast provisioning
   - Consistent results

3. Documentation
   - Code IS documentation
   - Always up-to-date
   - Self-documenting

4. Disaster Recovery
   - Rebuild entire infrastructure
   - From code in Git
   - In minutes

5. Testing
   - Test infrastructure changes
   - Before production
   - Automated validation
```

---

## Table of Contents
- [IaC Fundamentals](#iac-fundamentals)
- [Terraform](#terraform)
- [Terraform Advanced](#terraform-advanced)
- [Ansible](#ansible)
- [AWS CloudFormation](#aws-cloudformation)
- [Pulumi](#pulumi)
- [Comparison Guide](#comparison-guide)
- [State Management](#state-management)
- [Testing IaC](#testing-iac)
- [GitOps](#gitops)
- [Best Practices](#best-practices)
- [Interview Questions](#interview-questions)

---

## IaC Fundamentals

### 📖 Simple Explanation

**Two Approaches:**

**1. Declarative (What)**
```
Describe desired state:
"I want 3 web servers with 4GB RAM"

Tool figures out HOW to get there
Examples: Terraform, CloudFormation
```

**2. Imperative (How)**
```
Specify exact steps:
"Create server 1, then server 2, then server 3"

You control the exact process
Examples: Ansible scripts, Python/Boto3
```

---

### IaC vs Configuration Management

```
┌────────────────────────────────────────────────┐
│                                                 │
│  Infrastructure as Code (Terraform)            │
│  - Provision infrastructure                     │
│  - Create VMs, networks, storage               │
│  - Cloud resources                              │
│                                                 │
├────────────────────────────────────────────────┤
│                                                 │
│  Configuration Management (Ansible)            │
│  - Configure existing servers                   │
│  - Install software                             │
│  - Manage users, files                          │
│                                                 │
└────────────────────────────────────────────────┘

Often used together:
1. Terraform creates servers
2. Ansible configures them
```

---

## Terraform

### 📖 Simple Explanation

Terraform is the industry standard for IaC. Think of it as the "Git" of infrastructure.

---

### Installation

```bash
# macOS
brew install terraform

# Linux
wget https://releases.hashicorp.com/terraform/1.6.0/terraform_1.6.0_linux_amd64.zip
unzip terraform_1.6.0_linux_amd64.zip
sudo mv terraform /usr/local/bin/

# Verify
terraform version

# Enable autocomplete
terraform -install-autocomplete
```

---

### Basic Terraform Configuration

**Project Structure:**
```
project/
├── main.tf           # Main configuration
├── variables.tf      # Input variables
├── outputs.tf        # Output values
├── provider.tf       # Provider configuration
└── terraform.tfvars  # Variable values
```

---

**Simple EC2 Instance:**

```hcl
# provider.tf
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = "us-east-1"
}
```

```hcl
# main.tf
resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
  
  tags = {
    Name = "WebServer"
    Environment = "production"
  }
}
```

```hcl
# outputs.tf
output "instance_ip" {
  value = aws_instance.web.public_ip
  description = "The public IP of the web server"
}
```

---

### Terraform Workflow

```bash
# Initialize (download providers)
terraform init

# Format code
terraform fmt

# Validate configuration
terraform validate

# Plan changes (preview)
terraform plan

# Apply changes
terraform apply
# Review plan, type 'yes' to confirm

# Show current state
terraform show

# List resources
terraform state list

# Destroy infrastructure
terraform destroy
```

---

### Terraform Variables

**variables.tf:**
```hcl
variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t2.micro"
}

variable "instance_count" {
  description = "Number of instances"
  type        = number
  default     = 1
}

variable "enable_monitoring" {
  description = "Enable detailed monitoring"
  type        = bool
  default     = false
}

variable "tags" {
  description = "Tags for resources"
  type        = map(string)
  default = {
    Environment = "dev"
    Project     = "myapp"
  }
}

variable "availability_zones" {
  description = "List of AZs"
  type        = list(string)
  default     = ["us-east-1a", "us-east-1b"]
}
```

**Using Variables:**
```hcl
resource "aws_instance" "web" {
  ami           = var.ami_id
  instance_type = var.instance_type
  count         = var.instance_count
  
  tags = var.tags
}
```

**Setting Variables:**
```bash
# Command line
terraform apply -var="instance_type=t2.large"

# terraform.tfvars file
instance_type = "t2.large"
instance_count = 3
tags = {
  Environment = "production"
  Team = "platform"
}

# Environment variables
export TF_VAR_instance_type="t2.large"
terraform apply
```

---

### Complete Example: VPC with Web Servers

```hcl
# main.tf

# VPC
resource "aws_vpc" "main" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_hostnames = true
  enable_dns_support   = true
  
  tags = {
    Name = "main-vpc"
  }
}

# Internet Gateway
resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id
  
  tags = {
    Name = "main-igw"
  }
}

# Public Subnet
resource "aws_subnet" "public" {
  count             = 2
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.${count.index + 1}.0/24"
  availability_zone = data.aws_availability_zones.available.names[count.index]
  
  map_public_ip_on_launch = true
  
  tags = {
    Name = "public-subnet-${count.index + 1}"
  }
}

# Route Table
resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id
  
  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.main.id
  }
  
  tags = {
    Name = "public-rt"
  }
}

# Route Table Association
resource "aws_route_table_association" "public" {
  count          = 2
  subnet_id      = aws_subnet.public[count.index].id
  route_table_id = aws_route_table.public.id
}

# Security Group
resource "aws_security_group" "web" {
  name        = "web-sg"
  description = "Security group for web servers"
  vpc_id      = aws_vpc.main.id
  
  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]  # Restrict this in production!
  }
  
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  tags = {
    Name = "web-sg"
  }
}

# Data source for latest AMI
data "aws_ami" "amazon_linux" {
  most_recent = true
  owners      = ["amazon"]
  
  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*-x86_64-gp2"]
  }
}

# EC2 Instances
resource "aws_instance" "web" {
  count                  = 2
  ami                    = data.aws_ami.amazon_linux.id
  instance_type          = "t2.micro"
  subnet_id              = aws_subnet.public[count.index].id
  vpc_security_group_ids = [aws_security_group.web.id]
  
  user_data = <<-EOF
              #!/bin/bash
              yum update -y
              yum install -y httpd
              systemctl start httpd
              systemctl enable httpd
              echo "<h1>Web Server ${count.index + 1}</h1>" > /var/www/html/index.html
              EOF
  
  tags = {
    Name = "web-server-${count.index + 1}"
  }
}

# Load Balancer
resource "aws_lb" "web" {
  name               = "web-lb"
  internal           = false
  load_balancer_type = "application"
  security_groups    = [aws_security_group.web.id]
  subnets            = aws_subnet.public[*].id
  
  tags = {
    Name = "web-lb"
  }
}

# Target Group
resource "aws_lb_target_group" "web" {
  name     = "web-tg"
  port     = 80
  protocol = "HTTP"
  vpc_id   = aws_vpc.main.id
  
  health_check {
    path                = "/"
    healthy_threshold   = 2
    unhealthy_threshold = 10
  }
}

# Target Group Attachment
resource "aws_lb_target_group_attachment" "web" {
  count            = 2
  target_group_arn = aws_lb_target_group.web.arn
  target_id        = aws_instance.web[count.index].id
  port             = 80
}

# Listener
resource "aws_lb_listener" "web" {
  load_balancer_arn = aws_lb.web.arn
  port              = "80"
  protocol          = "HTTP"
  
  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.web.arn
  }
}

# Outputs
output "load_balancer_dns" {
  value       = aws_lb.web.dns_name
  description = "DNS name of the load balancer"
}

output "instance_ips" {
  value       = aws_instance.web[*].public_ip
  description = "Public IPs of web servers"
}
```

---

## Terraform Advanced

### Modules

**Create Reusable Module:**

```
modules/
└── ec2-instance/
    ├── main.tf
    ├── variables.tf
    └── outputs.tf
```

```hcl
# modules/ec2-instance/main.tf
resource "aws_instance" "this" {
  ami           = var.ami
  instance_type = var.instance_type
  
  tags = var.tags
}
```

```hcl
# modules/ec2-instance/variables.tf
variable "ami" {
  type = string
}

variable "instance_type" {
  type = string
}

variable "tags" {
  type = map(string)
}
```

```hcl
# modules/ec2-instance/outputs.tf
output "instance_id" {
  value = aws_instance.this.id
}

output "public_ip" {
  value = aws_instance.this.public_ip
}
```

**Use Module:**

```hcl
# main.tf
module "web_server" {
  source = "./modules/ec2-instance"
  
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
  tags = {
    Name = "WebServer"
  }
}

output "web_server_ip" {
  value = module.web_server.public_ip
}
```

---

### Dynamic Blocks

```hcl
variable "ingress_rules" {
  type = list(object({
    port        = number
    description = string
  }))
  default = [
    { port = 80, description = "HTTP" },
    { port = 443, description = "HTTPS" },
    { port = 22, description = "SSH" }
  ]
}

resource "aws_security_group" "web" {
  name = "web-sg"
  
  dynamic "ingress" {
    for_each = var.ingress_rules
    content {
      from_port   = ingress.value.port
      to_port     = ingress.value.port
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
      description = ingress.value.description
    }
  }
}
```

---

### Conditionals

```hcl
resource "aws_instance" "web" {
  ami           = var.ami
  instance_type = var.environment == "production" ? "t2.large" : "t2.micro"
  
  # Create only in production
  count = var.environment == "production" ? 3 : 1
  
  tags = {
    Name = "web-${count.index}"
  }
}
```

---

### Loops

```hcl
# for_each with map
variable "instances" {
  type = map(object({
    instance_type = string
    ami           = string
  }))
  default = {
    web = {
      instance_type = "t2.micro"
      ami           = "ami-123"
    }
    db = {
      instance_type = "t2.large"
      ami           = "ami-456"
    }
  }
}

resource "aws_instance" "servers" {
  for_each = var.instances
  
  ami           = each.value.ami
  instance_type = each.value.instance_type
  
  tags = {
    Name = each.key
  }
}

# Access resources
output "instance_ids" {
  value = {
    for name, instance in aws_instance.servers :
    name => instance.id
  }
}
```

---

### Remote State

```hcl
# backend.tf
terraform {
  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "prod/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-lock"
  }
}

# Create S3 bucket for state (run once)
resource "aws_s3_bucket" "terraform_state" {
  bucket = "my-terraform-state"
  
  lifecycle {
    prevent_destroy = true
  }
}

resource "aws_s3_bucket_versioning" "terraform_state" {
  bucket = aws_s3_bucket.terraform_state.id
  
  versioning_configuration {
    status = "Enabled"
  }
}

# DynamoDB for state locking
resource "aws_dynamodb_table" "terraform_lock" {
  name           = "terraform-lock"
  billing_mode   = "PAY_PER_REQUEST"
  hash_key       = "LockID"
  
  attribute {
    name = "LockID"
    type = "S"
  }
}
```

---

### Workspaces

```bash
# List workspaces
terraform workspace list

# Create workspace
terraform workspace new dev
terraform workspace new staging
terraform workspace new prod

# Switch workspace
terraform workspace select dev

# Current workspace
terraform workspace show

# Use in configuration
resource "aws_instance" "web" {
  ami           = var.ami
  instance_type = terraform.workspace == "prod" ? "t2.large" : "t2.micro"
  
  tags = {
    Name        = "web-${terraform.workspace}"
    Environment = terraform.workspace
  }
}
```

---

## Ansible

### 📖 Simple Explanation

Ansible is agentless configuration management. Push changes to servers over SSH.

---

### Installation

```bash
# macOS
brew install ansible

# Ubuntu
sudo apt update
sudo apt install ansible

# Verify
ansible --version
```

---

### Inventory File

```ini
# inventory.ini
[webservers]
web1.example.com
web2.example.com
web3.example.com

[databases]
db1.example.com ansible_host=10.0.1.10
db2.example.com ansible_host=10.0.1.11

[all:vars]
ansible_user=ubuntu
ansible_ssh_private_key_file=~/.ssh/id_rsa
```

**Dynamic Inventory (AWS):**
```yaml
# aws_ec2.yml
plugin: aws_ec2
regions:
  - us-east-1
filters:
  tag:Environment: production
keyed_groups:
  - key: tags.Role
    prefix: role
```

```bash
ansible-inventory -i aws_ec2.yml --graph
```

---

### Ad-hoc Commands

```bash
# Ping all servers
ansible all -i inventory.ini -m ping

# Run command
ansible webservers -i inventory.ini -a "uptime"

# Copy file
ansible webservers -i inventory.ini -m copy -a "src=/local/file dest=/remote/file"

# Install package
ansible webservers -i inventory.ini -m apt -a "name=nginx state=present" --become

# Restart service
ansible webservers -i inventory.ini -m service -a "name=nginx state=restarted" --become

# Gather facts
ansible webservers -i inventory.ini -m setup
```

---

### Ansible Playbook

```yaml
# playbook.yml
---
- name: Configure Web Servers
  hosts: webservers
  become: yes
  
  vars:
    nginx_port: 80
    app_version: "1.0.0"
  
  tasks:
    - name: Update apt cache
      apt:
        update_cache: yes
        cache_valid_time: 3600
    
    - name: Install Nginx
      apt:
        name: nginx
        state: present
    
    - name: Start Nginx
      service:
        name: nginx
        state: started
        enabled: yes
    
    - name: Copy configuration
      template:
        src: nginx.conf.j2
        dest: /etc/nginx/nginx.conf
      notify: Reload Nginx
    
    - name: Deploy application
      copy:
        src: "app-{{ app_version }}.tar.gz"
        dest: /opt/app/
    
    - name: Extract application
      unarchive:
        src: "/opt/app/app-{{ app_version }}.tar.gz"
        dest: /var/www/html/
        remote_src: yes
  
  handlers:
    - name: Reload Nginx
      service:
        name: nginx
        state: reloaded
```

**Run Playbook:**
```bash
ansible-playbook -i inventory.ini playbook.yml

# Dry run (check mode)
ansible-playbook -i inventory.ini playbook.yml --check

# Verbose output
ansible-playbook -i inventory.ini playbook.yml -v
ansible-playbook -i inventory.ini playbook.yml -vvv

# Run specific tags
ansible-playbook -i inventory.ini playbook.yml --tags "nginx"
```

---

### Ansible Roles

**Role Structure:**
```
roles/
└── nginx/
    ├── tasks/
    │   └── main.yml
    ├── handlers/
    │   └── main.yml
    ├── templates/
    │   └── nginx.conf.j2
    ├── files/
    │   └── index.html
    ├── vars/
    │   └── main.yml
    ├── defaults/
    │   └── main.yml
    └── meta/
        └── main.yml
```

**roles/nginx/tasks/main.yml:**
```yaml
---
- name: Install Nginx
  apt:
    name: nginx
    state: present

- name: Configure Nginx
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
  notify: Reload Nginx

- name: Start Nginx
  service:
    name: nginx
    state: started
    enabled: yes
```

**roles/nginx/handlers/main.yml:**
```yaml
---
- name: Reload Nginx
  service:
    name: nginx
    state: reloaded
```

**Use Role:**
```yaml
# playbook.yml
---
- name: Configure Web Servers
  hosts: webservers
  become: yes
  
  roles:
    - nginx
    - app
    - monitoring
```

---

### Ansible Vault (Secrets)

```bash
# Create encrypted file
ansible-vault create secrets.yml

# Edit encrypted file
ansible-vault edit secrets.yml

# Encrypt existing file
ansible-vault encrypt vars.yml

# Decrypt file
ansible-vault decrypt secrets.yml

# View encrypted file
ansible-vault view secrets.yml

# Run playbook with vault
ansible-playbook playbook.yml --ask-vault-pass

# Use password file
ansible-playbook playbook.yml --vault-password-file ~/.vault_pass
```

**secrets.yml:**
```yaml
---
db_password: "super_secret_password"
api_key: "abc123xyz789"
```

**Use in Playbook:**
```yaml
---
- name: Configure App
  hosts: app
  vars_files:
    - secrets.yml
  
  tasks:
    - name: Set database password
      lineinfile:
        path: /etc/app/config.yml
        line: "db_password: {{ db_password }}"
```

---

## AWS CloudFormation

### 📖 Simple Explanation

CloudFormation is AWS-native IaC. Deep AWS integration but vendor lock-in.

---

### Basic Template

```yaml
# template.yml
AWSTemplateFormatVersion: '2010-09-09'
Description: Simple EC2 instance

Parameters:
  InstanceType:
    Type: String
    Default: t2.micro
    AllowedValues:
      - t2.micro
      - t2.small
      - t2.medium
    Description: EC2 instance type
  
  KeyName:
    Type: AWS::EC2::KeyPair::KeyName
    Description: SSH key pair name

Mappings:
  RegionMap:
    us-east-1:
      AMI: ami-0c55b159cbfafe1f0
    us-west-2:
      AMI: ami-0d1cd67c26f5fca19

Resources:
  WebServerSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Enable HTTP and SSH
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 0.0.0.0/0
  
  WebServer:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: !FindInMap [RegionMap, !Ref 'AWS::Region', AMI]
      InstanceType: !Ref InstanceType
      KeyName: !Ref KeyName
      SecurityGroups:
        - !Ref WebServerSecurityGroup
      UserData:
        Fn::Base64: !Sub |
          #!/bin/bash
          yum update -y
          yum install -y httpd
          systemctl start httpd
          systemctl enable httpd
          echo "<h1>Hello from CloudFormation!</h1>" > /var/www/html/index.html
      Tags:
        - Key: Name
          Value: WebServer

Outputs:
  WebsiteURL:
    Description: URL of the website
    Value: !Sub 'http://${WebServer.PublicDnsName}'
  
  PublicIP:
    Description: Public IP address
    Value: !GetAtt WebServer.PublicIp
```

**Deploy Stack:**
```bash
# Create stack
aws cloudformation create-stack \
  --stack-name web-server \
  --template-body file://template.yml \
  --parameters ParameterKey=KeyName,ParameterValue=my-key

# Update stack
aws cloudformation update-stack \
  --stack-name web-server \
  --template-body file://template.yml

# Delete stack
aws cloudformation delete-stack --stack-name web-server

# Describe stack
aws cloudformation describe-stacks --stack-name web-server

# List stacks
aws cloudformation list-stacks
```

---

## Pulumi

### 📖 Simple Explanation

Pulumi uses real programming languages (Python, TypeScript, Go) for IaC.

---

### Example (TypeScript)

```typescript
// index.ts
import * as pulumi from "@pulumi/pulumi";
import * as aws from "@pulumi/aws";

// Create VPC
const vpc = new aws.ec2.Vpc("main", {
    cidrBlock: "10.0.0.0/16",
    enableDnsHostnames: true,
    enableDnsSupport: true,
    tags: {
        Name: "main-vpc",
    },
});

// Create subnet
const subnet = new aws.ec2.Subnet("public", {
    vpcId: vpc.id,
    cidrBlock: "10.0.1.0/24",
    mapPublicIpOnLaunch: true,
    tags: {
        Name: "public-subnet",
    },
});

// Security group
const securityGroup = new aws.ec2.SecurityGroup("web", {
    vpcId: vpc.id,
    description: "Allow HTTP and SSH",
    ingress: [
        { protocol: "tcp", fromPort: 80, toPort: 80, cidrBlocks: ["0.0.0.0/0"] },
        { protocol: "tcp", fromPort: 22, toPort: 22, cidrBlocks: ["0.0.0.0/0"] },
    ],
    egress: [
        { protocol: "-1", fromPort: 0, toPort: 0, cidrBlocks: ["0.0.0.0/0"] },
    ],
});

// Get latest AMI
const ami = aws.ec2.getAmi({
    mostRecent: true,
    owners: ["amazon"],
    filters: [{
        name: "name",
        values: ["amzn2-ami-hvm-*-x86_64-gp2"],
    }],
});

// Create EC2 instance
const server = new aws.ec2.Instance("web", {
    ami: ami.then(ami => ami.id),
    instanceType: "t2.micro",
    subnetId: subnet.id,
    vpcSecurityGroupIds: [securityGroup.id],
    userData: `#!/bin/bash
        yum update -y
        yum install -y httpd
        systemctl start httpd
        systemctl enable httpd
        echo "<h1>Hello from Pulumi!</h1>" > /var/www/html/index.html
    `,
    tags: {
        Name: "web-server",
    },
});

// Export outputs
export const publicIp = server.publicIp;
export const publicDns = server.publicDns;
```

**Deploy:**
```bash
pulumi up
pulumi destroy
pulumi stack output publicIp
```

---

## Comparison Guide

```
┌───────────────┬──────────┬────────────┬────────────┬────────────┐
│ Feature       │Terraform │ Ansible    │CloudForm.  │ Pulumi     │
├───────────────┼──────────┼────────────┼────────────┼────────────┤
│ Language      │ HCL      │ YAML       │ YAML/JSON  │ Real Code  │
│ Approach      │ Declar.  │ Procedural │ Declar.    │ Declar.    │
│ Cloud Support │ Multi    │ Multi      │ AWS only   │ Multi      │
│ State         │ Yes      │ No         │ AWS        │ Yes        │
│ Learning      │ Medium   │ Easy       │ Hard       │ Medium     │
│ Community     │ Large    │ Large      │ Large      │ Growing    │
│ Best For      │ Infra    │ Config     │ AWS-only   │ Devs       │
└───────────────┴──────────┴────────────┴────────────┴────────────┘

Typical Stack:
1. Terraform - Provision infrastructure
2. Ansible - Configure servers
3. Kubernetes - Run containers
```

---

## State Management

### 📖 Simple Explanation

State tracks what infrastructure exists vs what should exist.

---

### Terraform State

```bash
# View state
terraform state list
terraform state show aws_instance.web

# Move resources
terraform state mv aws_instance.web aws_instance.web_server

# Remove from state (doesn't destroy)
terraform state rm aws_instance.web

# Import existing resources
terraform import aws_instance.web i-1234567890abcdef0

# Refresh state
terraform refresh

# Pull state
terraform state pull > terraform.tfstate.backup
```

---

### Remote State Best Practices

```hcl
# Use remote backend
terraform {
  backend "s3" {
    bucket         = "terraform-state-bucket"
    key            = "prod/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-lock"  # State locking
  }
}

# Or use Terraform Cloud
terraform {
  cloud {
    organization = "my-org"
    workspaces {
      name = "production"
    }
  }
}
```

---

## Testing IaC

### Terraform Validation

```bash
# Format check
terraform fmt -check

# Validate syntax
terraform validate

# Security scanning
tfsec .

# Policy as Code (Sentinel)
terraform apply -policy=sentinel.hcl
```

---

### Terratest (Go)

```go
// test/main_test.go
package test

import (
    "testing"
    "github.com/gruntwork-io/terratest/modules/terraform"
    "github.com/stretchr/testify/assert"
)

func TestTerraformEc2(t *testing.T) {
    terraformOptions := &terraform.Options{
        TerraformDir: "../",
        Vars: map[string]interface{}{
            "instance_type": "t2.micro",
        },
    }
    
    defer terraform.Destroy(t, terraformOptions)
    terraform.InitAndApply(t, terraformOptions)
    
    instanceIP := terraform.Output(t, terraformOptions, "instance_ip")
    assert.NotEmpty(t, instanceIP)
}
```

---

### Ansible Testing

```bash
# Syntax check
ansible-playbook playbook.yml --syntax-check

# Dry run
ansible-playbook playbook.yml --check

# Molecule (testing framework)
molecule init scenario
molecule test
```

---

## GitOps

### 📖 Simple Explanation

GitOps = Infrastructure changes through Git pull requests.

---

### GitOps Workflow

```
1. Change infrastructure code
2. Create pull request
3. Automated tests run
4. Code review
5. Merge to main
6. CI/CD auto-applies changes

All changes:
- Version controlled
- Auditable
- Rollback-able
```

---

### Example GitOps Pipeline

```yaml
# .github/workflows/terraform.yml
name: Terraform

on:
  pull_request:
    branches: [main]
  push:
    branches: [main]

jobs:
  terraform:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v2
      
      - name: Terraform Init
        run: terraform init
      
      - name: Terraform Format
        run: terraform fmt -check
      
      - name: Terraform Validate
        run: terraform validate
      
      - name: Terraform Plan
        run: terraform plan -out=tfplan
      
      - name: Terraform Apply
        if: github.ref == 'refs/heads/main'
        run: terraform apply -auto-approve tfplan
```

---

## Best Practices

### 1. Version Everything

```bash
# ✅ Pin versions
terraform {
  required_version = "~> 1.6.0"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

# ❌ Don't use latest
required_version = "latest"  # Bad!
```

---

### 2. Use Modules

```hcl
# ✅ Reusable modules
module "vpc" {
  source = "terraform-aws-modules/vpc/aws"
  
  name = "my-vpc"
  cidr = "10.0.0.0/16"
}

# ❌ Don't duplicate code
resource "aws_vpc" "vpc1" { ... }
resource "aws_vpc" "vpc2" { ... }
resource "aws_vpc" "vpc3" { ... }
```

---

### 3. Separate Environments

```
# ✅ Directory structure
environments/
├── dev/
│   ├── main.tf
│   └── terraform.tfvars
├── staging/
│   ├── main.tf
│   └── terraform.tfvars
└── prod/
    ├── main.tf
    └── terraform.tfvars

# Or use workspaces
terraform workspace new dev
terraform workspace new prod
```

---

### 4. Remote State

```hcl
# ✅ Store state remotely
backend "s3" {
  bucket = "terraform-state"
  key    = "prod/terraform.tfstate"
  dynamodb_table = "terraform-lock"
}

# ❌ Don't commit state to Git
.gitignore:
*.tfstate
*.tfstate.backup
```

---

### 5. Naming Conventions

```hcl
# ✅ Clear naming
resource "aws_instance" "web_server_prod" {
  tags = {
    Name        = "web-server-prod"
    Environment = "production"
    ManagedBy   = "terraform"
  }
}
```

---

## Interview Questions

### Q1: Terraform vs CloudFormation?
**Answer:**
```
Terraform:
+ Multi-cloud (AWS, Azure, GCP)
+ Larger community
+ Better module ecosystem
+ Plan preview before apply
- Requires state management

CloudFormation:
+ Deep AWS integration
+ No state management
+ Native AWS support
- AWS only
- Harder syntax
- Slower updates

Recommendation: Terraform for multi-cloud, CloudFormation if AWS-only
```

---

### Q2: How do you manage Terraform state?
**Answer:**
```
1. Remote Backend (S3 + DynamoDB)
   - Encrypted S3 bucket
   - DynamoDB for locking
   - Prevents concurrent modifications

2. Terraform Cloud
   - Managed state
   - Free tier available
   - Built-in locking

3. Best Practices:
   - Never commit state to Git
   - Enable versioning on state bucket
   - Encrypt state at rest
   - Use state locking
   - Separate state per environment

Example:
terraform {
  backend "s3" {
    bucket         = "terraform-state"
    key            = "prod/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-lock"
  }
}
```

---

### Q3: Declarative vs Imperative IaC?
**Answer:**
```
Declarative (What):
- Describe desired end state
- Tool figures out how to get there
- Idempotent by design
- Examples: Terraform, CloudFormation

resource "aws_instance" "web" {
  ami           = "ami-123"
  instance_type = "t2.micro"
}

Imperative (How):
- Specify exact steps
- You control the process
- Need to handle idempotency
- Examples: Ansible scripts, Python/Boto3

def create_instance():
    ec2 = boto3.client('ec2')
    ec2.run_instances(ImageId='ami-123', InstanceType='t2.micro')

Recommendation: Declarative for infrastructure provisioning
```

---

### Q4: How do you handle secrets in IaC?
**Answer:**
```
Never hardcode secrets!

Solutions:
1. AWS Secrets Manager / Parameter Store
2. HashiCorp Vault
3. Environment variables
4. Terraform/Ansible Vault

Terraform example:
data "aws_secretsmanager_secret_version" "db_password" {
  secret_id = "prod/db/password"
}

resource "aws_db_instance" "main" {
  password = data.aws_secretsmanager_secret_version.db_password.secret_string
}

Ansible example:
ansible-vault encrypt secrets.yml
ansible-playbook playbook.yml --vault-password-file ~/.vault_pass
```

---

### Q5: How to import existing infrastructure?
**Answer:**
```
Terraform Import:

1. Write resource definition
resource "aws_instance" "web" {
  # Leave blank initially
}

2. Import resource
terraform import aws_instance.web i-1234567890abcdef0

3. Run terraform plan to see differences
terraform plan

4. Update resource definition to match reality

5. Verify
terraform plan  # Should show no changes

Alternative: Terraformer
- Auto-generates Terraform code from existing infra
terraformer import aws --resources=vpc,subnet,instance
```

---

### Q6: Handling dependencies in Terraform?
**Answer:**
```
Implicit Dependencies (automatic):
resource "aws_subnet" "main" {
  vpc_id = aws_vpc.main.id  # Implicit dependency on VPC
}

Explicit Dependencies (manual):
resource "aws_instance" "web" {
  depends_on = [aws_iam_role.web_role]
}

Module Dependencies:
module "vpc" {
  source = "./modules/vpc"
}

module "ec2" {
  source = "./modules/ec2"
  vpc_id = module.vpc.vpc_id  # Module dependency
}

Terraform automatically creates dependency graph and
provisions resources in correct order
```

---

## Quick Reference

```bash
# Terraform
terraform init      # Initialize
terraform plan      # Preview changes
terraform apply     # Apply changes
terraform destroy   # Destroy infrastructure
terraform fmt       # Format code
terraform validate  # Validate syntax

# Ansible
ansible all -m ping                    # Test connectivity
ansible-playbook playbook.yml          # Run playbook
ansible-playbook playbook.yml --check  # Dry run
ansible-vault encrypt file.yml         # Encrypt secrets

# CloudFormation
aws cloudformation create-stack --stack-name NAME --template-body file://template.yml
aws cloudformation update-stack --stack-name NAME --template-body file://template.yml
aws cloudformation delete-stack --stack-name NAME
```

---

## Summary

**Key Takeaways:**
1. IaC = Infrastructure in version-controlled code
2. Terraform = Most popular, multi-cloud
3. Ansible = Configuration management
4. Use modules for reusability
5. Remote state for team collaboration
6. GitOps for change management
7. Test before production

**Next Steps:**
1. Create simple Terraform project
2. Deploy infrastructure to AWS/Azure
3. Use modules for organization
4. Set up remote state
5. Implement GitOps workflow
6. Add automated testing

**Remember:**
- Version everything
- Test before apply
- Use modules
- Remote state
- Never commit secrets

**Happy Coding! 🚀**

