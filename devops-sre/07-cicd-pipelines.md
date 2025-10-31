# 7. CI/CD Pipelines

## 📚 Quick Summary

CI/CD automates software delivery - the heart of modern DevOps practices!

**What You'll Learn:**
- **CI/CD Fundamentals**: Continuous Integration, Delivery, and Deployment
- **Jenkins**: Most popular CI/CD tool, pipelines as code
- **GitHub Actions**: Cloud-native CI/CD, workflow automation
- **GitLab CI/CD**: Integrated DevOps platform
- **Pipeline Design**: Best practices, patterns, optimization
- **Testing in Pipelines**: Unit, integration, security, performance
- **Deployment Strategies**: Blue-green, canary, rolling updates

**Why This Matters:**
- Automate testing and deployment (save hours daily)
- Catch bugs before production
- Deploy multiple times per day safely
- Core DevOps skill (80% of jobs require it)
- Interview questions: 20% are CI/CD-based

**Interview Reality:**
"Design a CI/CD pipeline for this application" = Must-know for DevOps roles!

---

## 📖 Simple Explanation

**What is CI/CD?**

**Before CI/CD:**
```
Developer writes code → Manual testing → Manual deployment
Takes: Hours/Days
Risk: High (human errors)
Frequency: Weekly/Monthly
😰
```

**With CI/CD:**
```
Developer pushes code → Automated tests → Automated deployment
Takes: Minutes
Risk: Low (automated, consistent)
Frequency: Multiple times per day
😊
```

**Real-World Analogy:**
```
Manual Process = Cooking each meal from scratch
CI/CD = Assembly line in a factory
- Consistent quality
- Fast production
- Automated quality checks
- Immediate feedback
```

---

**Three Key Concepts:**

**1. Continuous Integration (CI)**
```
Every code push triggers:
1. Build
2. Run tests
3. Report results

Benefits:
- Catch bugs early
- Integration issues detected immediately
- Always have a working build
```

**2. Continuous Delivery (CD)**
```
After CI passes:
1. Package application
2. Deploy to staging
3. Ready for production (manual trigger)

Benefits:
- Always ready to deploy
- Reduced deployment risk
- Faster time to market
```

**3. Continuous Deployment**
```
After CI passes:
1. Automatically deploy to production
2. No human intervention
3. Live in minutes

Benefits:
- Fastest feedback loop
- True DevOps automation
- Multiple deploys per day
```

---

## Table of Contents
- [CI/CD Fundamentals](#cicd-fundamentals)
- [Jenkins](#jenkins)
- [GitHub Actions](#github-actions)
- [GitLab CI/CD](#gitlab-cicd)
- [Azure DevOps Pipelines](#azure-devops-pipelines)
- [Pipeline Design Patterns](#pipeline-design-patterns)
- [Testing in Pipelines](#testing-in-pipelines)
- [Deployment Strategies](#deployment-strategies)
- [Pipeline Security](#pipeline-security)
- [Best Practices](#best-practices)
- [Troubleshooting](#troubleshooting)
- [Interview Questions](#interview-questions)

---

## CI/CD Fundamentals

### 📖 Simple Explanation

**The CI/CD Pipeline Flow:**

```
┌──────────────┐
│ Code Commit  │  Developer pushes code
└──────┬───────┘
       │
       v
┌──────────────┐
│   Build      │  Compile, package
└──────┬───────┘
       │
       v
┌──────────────┐
│   Test       │  Unit, integration, security
└──────┬───────┘
       │
       v
┌──────────────┐
│   Stage      │  Deploy to staging environment
└──────┬───────┘
       │
       v
┌──────────────┐
│   Production │  Deploy to production
└──────────────┘
```

---

### Core CI/CD Principles

**1. Build Automation**
```bash
# Every commit should trigger an automated build
- Compile code
- Run linters
- Generate artifacts
- Create Docker images
```

**2. Test Automation**
```bash
# Multiple levels of testing
- Unit tests (fast, isolated)
- Integration tests (component interaction)
- E2E tests (user workflows)
- Security tests (vulnerability scanning)
- Performance tests (load testing)
```

**3. Fast Feedback**
```bash
# Developers should know within minutes
- Build status
- Test results
- Code quality metrics
- Security issues

Target: < 10 minutes for full pipeline
```

**4. Idempotency**
```bash
# Same inputs = Same outputs
- Reproducible builds
- Versioned dependencies
- Immutable artifacts
- Consistent environments
```

---

### Pipeline Stages

**Typical Pipeline:**

```yaml
Stages:
  1. Checkout      # Get code from repository
  2. Build         # Compile/package application
  3. Test          # Run automated tests
  4. Analysis      # Code quality, security scanning
  5. Artifact      # Store build artifacts
  6. Deploy-Dev    # Deploy to dev environment
  7. Deploy-Stage  # Deploy to staging
  8. Deploy-Prod   # Deploy to production (manual/auto)
```

---

## Jenkins

### 📖 Simple Explanation

Jenkins is the most popular open-source CI/CD tool. Think of it as the "Swiss Army knife" of automation.

---

### Jenkins Installation

```bash
# Docker (easiest way)
docker run -d -p 8080:8080 -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  --name jenkins \
  jenkins/jenkins:lts

# Access Jenkins
# http://localhost:8080

# Get initial admin password
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword

# Install on Ubuntu
wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
sudo apt update
sudo apt install jenkins

# Start Jenkins
sudo systemctl start jenkins
sudo systemctl enable jenkins
```

---

### Jenkinsfile (Pipeline as Code)

**Declarative Pipeline:**

```groovy
// Jenkinsfile
pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = "myapp"
        VERSION = "${env.BUILD_NUMBER}"
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Build') {
            steps {
                sh 'npm install'
                sh 'npm run build'
            }
        }
        
        stage('Test') {
            steps {
                sh 'npm test'
            }
            post {
                always {
                    junit 'test-results/*.xml'
                }
            }
        }
        
        stage('Code Quality') {
            steps {
                sh 'npm run lint'
            }
        }
        
        stage('Security Scan') {
            steps {
                sh 'npm audit'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${DOCKER_IMAGE}:${VERSION} ."
                sh "docker tag ${DOCKER_IMAGE}:${VERSION} ${DOCKER_IMAGE}:latest"
            }
        }
        
        stage('Push to Registry') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub', 
                                 usernameVariable: 'USER', 
                                 passwordVariable: 'PASS')]) {
                    sh 'echo $PASS | docker login -u $USER --password-stdin'
                    sh "docker push ${DOCKER_IMAGE}:${VERSION}"
                    sh "docker push ${DOCKER_IMAGE}:latest"
                }
            }
        }
        
        stage('Deploy to Staging') {
            steps {
                sh """
                    kubectl set image deployment/myapp \
                    myapp=${DOCKER_IMAGE}:${VERSION} \
                    --namespace=staging
                """
            }
        }
        
        stage('Deploy to Production') {
            when {
                branch 'main'
            }
            steps {
                input message: 'Deploy to production?', ok: 'Deploy'
                sh """
                    kubectl set image deployment/myapp \
                    myapp=${DOCKER_IMAGE}:${VERSION} \
                    --namespace=production
                """
            }
        }
    }
    
    post {
        success {
            slackSend color: 'good', message: "Build ${VERSION} succeeded!"
        }
        failure {
            slackSend color: 'danger', message: "Build ${VERSION} failed!"
        }
        always {
            cleanWs()
        }
    }
}
```

---

**Scripted Pipeline (More Flexible):**

```groovy
node {
    def app
    
    stage('Checkout') {
        checkout scm
    }
    
    stage('Build') {
        app = docker.build("myapp:${env.BUILD_NUMBER}")
    }
    
    stage('Test') {
        app.inside {
            sh 'npm test'
        }
    }
    
    stage('Push') {
        docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-credentials') {
            app.push("${env.BUILD_NUMBER}")
            app.push("latest")
        }
    }
    
    stage('Deploy') {
        if (env.BRANCH_NAME == 'main') {
            sh 'kubectl apply -f k8s/'
        }
    }
}
```

---

### Jenkins Pipeline Features

**Parallel Execution:**

```groovy
pipeline {
    agent any
    
    stages {
        stage('Parallel Tests') {
            parallel {
                stage('Unit Tests') {
                    steps {
                        sh 'npm run test:unit'
                    }
                }
                stage('Integration Tests') {
                    steps {
                        sh 'npm run test:integration'
                    }
                }
                stage('E2E Tests') {
                    steps {
                        sh 'npm run test:e2e'
                    }
                }
            }
        }
    }
}
```

**Conditional Stages:**

```groovy
stage('Deploy') {
    when {
        anyOf {
            branch 'main'
            branch 'develop'
        }
    }
    steps {
        sh 'deploy.sh'
    }
}
```

**Parameters:**

```groovy
pipeline {
    agent any
    
    parameters {
        choice(name: 'ENVIRONMENT', choices: ['dev', 'staging', 'prod'], description: 'Target environment')
        booleanParam(name: 'RUN_TESTS', defaultValue: true, description: 'Run tests?')
        string(name: 'VERSION', defaultValue: 'latest', description: 'Version to deploy')
    }
    
    stages {
        stage('Deploy') {
            steps {
                sh "deploy.sh ${params.ENVIRONMENT} ${params.VERSION}"
            }
        }
    }
}
```

---

### Jenkins Plugins (Essential)

```bash
# Plugin Management
Jenkins Dashboard → Manage Jenkins → Manage Plugins

Essential Plugins:
1. Git Plugin - Git integration
2. Docker Pipeline - Docker support
3. Kubernetes - K8s deployments
4. Blue Ocean - Modern UI
5. Pipeline - Pipeline support
6. Credentials - Secure credential storage
7. Slack Notification - Slack integration
8. SonarQube Scanner - Code quality
9. JUnit - Test reporting
10. Artifactory - Artifact management
```

---

## GitHub Actions

### 📖 Simple Explanation

GitHub Actions is CI/CD built into GitHub. No separate server needed!

---

### Basic Workflow

```yaml
# .github/workflows/ci.yml
name: CI/CD Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run linter
        run: npm run lint
      
      - name: Run tests
        run: npm test
      
      - name: Build application
        run: npm run build
      
      - name: Upload artifacts
        uses: actions/upload-artifact@v3
        with:
          name: build
          path: dist/
```

---

### Advanced GitHub Actions

**Multi-Job Workflow:**

```yaml
name: Full CI/CD

on:
  push:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [16, 18, 20]
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v3
        with:
          node-version: ${{ matrix.node-version }}
      
      - run: npm ci
      - run: npm test

  build:
    needs: test
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Build Docker image
        run: |
          docker build -t myapp:${{ github.sha }} .
          docker tag myapp:${{ github.sha }} myapp:latest
      
      - name: Login to Docker Hub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}
      
      - name: Push to Docker Hub
        run: |
          docker push myapp:${{ github.sha }}
          docker push myapp:latest

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment: production
    
    steps:
      - name: Deploy to Kubernetes
        uses: azure/k8s-deploy@v4
        with:
          manifests: |
            k8s/deployment.yml
            k8s/service.yml
          images: myapp:${{ github.sha }}
          kubectl-version: 'latest'
```

---

**Reusable Workflows:**

```yaml
# .github/workflows/reusable-test.yml
name: Reusable Test Workflow

on:
  workflow_call:
    inputs:
      node-version:
        required: true
        type: string

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: ${{ inputs.node-version }}
      - run: npm ci
      - run: npm test
```

```yaml
# .github/workflows/main.yml
name: Main Workflow

on: push

jobs:
  test-node-16:
    uses: ./.github/workflows/reusable-test.yml
    with:
      node-version: '16'
  
  test-node-18:
    uses: ./.github/workflows/reusable-test.yml
    with:
      node-version: '18'
```

---

### GitHub Actions Features

**Caching:**

```yaml
- name: Cache dependencies
  uses: actions/cache@v3
  with:
    path: ~/.npm
    key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
    restore-keys: |
      ${{ runner.os }}-node-
```

**Conditional Steps:**

```yaml
- name: Deploy to production
  if: github.ref == 'refs/heads/main'
  run: ./deploy.sh production
```

**Manual Approval:**

```yaml
deploy:
  runs-on: ubuntu-latest
  environment: production  # Requires approval in GitHub settings
  steps:
    - name: Deploy
      run: ./deploy.sh
```

---

## GitLab CI/CD

### 📖 Simple Explanation

GitLab CI/CD is built into GitLab - complete DevOps platform in one tool!

---

### Basic Pipeline

```yaml
# .gitlab-ci.yml
stages:
  - build
  - test
  - deploy

variables:
  DOCKER_IMAGE: registry.gitlab.com/mygroup/myapp

build:
  stage: build
  image: node:18
  script:
    - npm install
    - npm run build
  artifacts:
    paths:
      - dist/
    expire_in: 1 hour

test:
  stage: test
  image: node:18
  script:
    - npm install
    - npm test
  coverage: '/Coverage: \d+\.\d+%/'

deploy_staging:
  stage: deploy
  image: alpine/k8s:latest
  script:
    - kubectl config use-context staging
    - kubectl apply -f k8s/
  environment:
    name: staging
    url: https://staging.example.com
  only:
    - develop

deploy_production:
  stage: deploy
  image: alpine/k8s:latest
  script:
    - kubectl config use-context production
    - kubectl apply -f k8s/
  environment:
    name: production
    url: https://example.com
  when: manual
  only:
    - main
```

---

### Advanced GitLab CI/CD

**Docker Build and Push:**

```yaml
build_docker:
  stage: build
  image: docker:latest
  services:
    - docker:dind
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
  script:
    - docker build -t $DOCKER_IMAGE:$CI_COMMIT_SHA .
    - docker tag $DOCKER_IMAGE:$CI_COMMIT_SHA $DOCKER_IMAGE:latest
    - docker push $DOCKER_IMAGE:$CI_COMMIT_SHA
    - docker push $DOCKER_IMAGE:latest
```

**Parallel Jobs:**

```yaml
test:
  stage: test
  parallel:
    matrix:
      - NODE_VERSION: ['16', '18', '20']
  image: node:${NODE_VERSION}
  script:
    - npm install
    - npm test
```

**Include External Files:**

```yaml
include:
  - local: '/templates/.gitlab-ci-template.yml'
  - project: 'mygroup/ci-templates'
    file: '/templates/docker.yml'
  - remote: 'https://gitlab.com/example/ci-template/raw/main/.gitlab-ci.yml'
```

---

## Azure DevOps Pipelines

### Basic Pipeline

```yaml
# azure-pipelines.yml
trigger:
  - main

pool:
  vmImage: 'ubuntu-latest'

variables:
  buildConfiguration: 'Release'

stages:
- stage: Build
  jobs:
  - job: BuildJob
    steps:
    - task: NodeTool@0
      inputs:
        versionSpec: '18.x'
    
    - script: npm install
      displayName: 'Install dependencies'
    
    - script: npm run build
      displayName: 'Build application'
    
    - task: PublishBuildArtifacts@1
      inputs:
        pathToPublish: 'dist'
        artifactName: 'drop'

- stage: Test
  jobs:
  - job: TestJob
    steps:
    - script: npm test
      displayName: 'Run tests'

- stage: Deploy
  condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))
  jobs:
  - deployment: DeployWeb
    environment: 'production'
    strategy:
      runOnce:
        deploy:
          steps:
          - script: echo Deploying to production
```

---

## Pipeline Design Patterns

### 📖 Simple Explanation

Design patterns for building efficient, maintainable pipelines.

---

### 1. Fan-out Pattern

```yaml
# Run multiple jobs in parallel
jobs:
  test:
    needs: build
    strategy:
      matrix:
        test-suite: [unit, integration, e2e, security]
    steps:
      - run: npm run test:${{ matrix.test-suite }}
```

---

### 2. Fan-in Pattern

```yaml
# Multiple jobs converge to one
jobs:
  test-1:
    runs-on: ubuntu-latest
    steps:
      - run: npm run test:unit
  
  test-2:
    runs-on: ubuntu-latest
    steps:
      - run: npm run test:integration
  
  deploy:
    needs: [test-1, test-2]  # Wait for both
    runs-on: ubuntu-latest
    steps:
      - run: ./deploy.sh
```

---

### 3. Pipeline as Library

```yaml
# Reusable pipeline components
# .github/workflows/templates/test.yml
test-template:
  steps:
    - uses: actions/checkout@v3
    - run: npm ci
    - run: npm test
```

---

### 4. Multi-Stage Pipeline

```
Build → Test → Stage → Production

Each stage:
- Gates/Approvals
- Environment-specific config
- Rollback capability
```

---

### 5. Trunk-Based CI/CD

```yaml
# Deploy every commit to main
on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to production
        run: ./deploy.sh
```

---

## Testing in Pipelines

### 📖 Simple Explanation

**Testing Pyramid:**

```
         /\
        /E2E\        Few, slow, expensive
       /------\
      / Integ  \     Some, medium speed
     /----------\
    /   Unit     \   Many, fast, cheap
   /--------------\
```

---

### Unit Tests

```yaml
- name: Unit Tests
  run: |
    npm run test:unit
    npm run coverage
  
- name: Upload coverage
  uses: codecov/codecov-action@v3
  with:
    files: ./coverage/lcov.info
```

---

### Integration Tests

```yaml
- name: Integration Tests
  run: |
    docker-compose up -d
    npm run test:integration
    docker-compose down
```

---

### End-to-End Tests

```yaml
- name: E2E Tests
  run: |
    npm run start &
    npx wait-on http://localhost:3000
    npm run test:e2e
```

---

### Security Scanning

```yaml
- name: Security Audit
  run: npm audit --audit-level=high

- name: Dependency Check
  uses: dependency-check/Dependency-Check_Action@main

- name: Container Scan
  uses: aquasecurity/trivy-action@master
  with:
    image-ref: 'myapp:latest'
    severity: 'CRITICAL,HIGH'
```

---

### Performance Tests

```yaml
- name: Load Testing
  run: |
    artillery run load-test.yml
    
- name: Lighthouse CI
  uses: treosh/lighthouse-ci-action@v9
  with:
    urls: https://staging.example.com
```

---

## Deployment Strategies

### 📖 Simple Explanation

Different ways to deploy with minimal downtime and risk.

---

### 1. Blue-Green Deployment

```
Blue (Current):  ●●●●●  ← All traffic
Green (New):     ●●●●●  ← Deploy here, test

Switch:
Blue (Old):      ●●●●●
Green (Current): ●●●●●  ← All traffic switched

Rollback: Switch back to Blue
```

**Implementation:**

```yaml
- name: Deploy Green
  run: |
    kubectl apply -f deployment-green.yml
    kubectl wait --for=condition=available deployment/myapp-green
    
- name: Test Green
  run: |
    curl https://green.example.com/health
    
- name: Switch Traffic
  run: |
    kubectl patch service myapp -p '{"spec":{"selector":{"version":"green"}}}'
```

---

### 2. Canary Deployment

```
Version 1: ●●●●●●●●●●  90% traffic
Version 2: ●          10% traffic

Monitor metrics, gradually increase:
Version 1: ●●●●●      50% traffic
Version 2: ●●●●●      50% traffic

Finally:
Version 1: (removed)
Version 2: ●●●●●●●●●● 100% traffic
```

**Implementation:**

```yaml
- name: Deploy Canary
  run: |
    kubectl apply -f deployment-canary.yml
    kubectl scale deployment myapp-canary --replicas=1
    kubectl scale deployment myapp --replicas=9
    
- name: Monitor Metrics
  run: |
    ./monitor.sh --duration=10m --error-threshold=1%
    
- name: Increase Canary
  run: |
    kubectl scale deployment myapp-canary --replicas=5
    kubectl scale deployment myapp --replicas=5
```

---

### 3. Rolling Update

```
V1: ●●●●●

Step 1: ●●●●○ (replace one)
Step 2: ●●●○○ (replace another)
Step 3: ●●○○○
Step 4: ●○○○○
Step 5: ○○○○○ (all V2)

Kubernetes default strategy
```

**Implementation:**

```yaml
# Kubernetes handles automatically
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 5
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1        # Add 1 extra pod during update
      maxUnavailable: 1   # Max 1 pod down during update
```

---

### 4. Feature Flags

```yaml
- name: Deploy with Feature Flag
  run: |
    kubectl set env deployment/myapp \
      FEATURE_NEW_UI=false
    
# Enable for 10% of users
- name: Enable Canary
  run: |
    curl -X POST https://feature-flags.example.com/api/flags \
      -d '{"flag":"new_ui","rollout":10}'
```

---

## Pipeline Security

### Secret Management

```yaml
# GitHub Actions - Use secrets
- name: Deploy
  env:
    API_KEY: ${{ secrets.API_KEY }}
  run: ./deploy.sh

# Never hardcode secrets!
# ❌ BAD
- run: docker login -u admin -p password123

# ✅ GOOD
- run: echo "${{ secrets.DOCKER_PASSWORD }}" | docker login -u "${{ secrets.DOCKER_USERNAME }}" --password-stdin
```

---

### Least Privilege

```yaml
# Use specific permissions
permissions:
  contents: read        # Read code
  packages: write       # Push packages
  security-events: write # Upload security scans
```

---

### Supply Chain Security

```yaml
- name: Verify Dependencies
  run: npm audit --audit-level=high

- name: SBOM Generation
  run: |
    npm sbom --sbom-format=cyclonedx > sbom.json

- name: Sign Artifacts
  uses: sigstore/cosign-installer@main
  run: |
    cosign sign myapp:latest
```

---

## Best Practices

### 1. Fast Pipelines

```yaml
# ✅ Run fast tests first
stages:
  - lint       # 30 seconds
  - unit       # 2 minutes
  - build      # 5 minutes
  - integration # 10 minutes
  - e2e        # 30 minutes

# ❌ Don't run slow tests first
# Wastes time if quick checks would fail
```

---

### 2. Fail Fast

```yaml
# Stop on first failure
- name: Tests
  run: npm test
  continue-on-error: false  # Default, but explicit

# Parallel execution - all must pass
jobs:
  test-unit:
    runs-on: ubuntu-latest
  test-integration:
    runs-on: ubuntu-latest
  
  deploy:
    needs: [test-unit, test-integration]  # Deploy only if both pass
```

---

### 3. Idempotent Pipelines

```bash
# ✅ Same inputs = Same outputs
# Use locked dependencies
npm ci  # Not npm install

# Pin versions
FROM node:18.17.0  # Not node:18 or node:latest

# Version artifacts
docker tag myapp:${GIT_SHA}
```

---

### 4. Observable Pipelines

```yaml
- name: Build
  run: npm run build
  
- name: Upload Logs
  if: always()
  uses: actions/upload-artifact@v3
  with:
    name: build-logs
    path: build.log

- name: Notify
  if: failure()
  run: |
    curl -X POST $SLACK_WEBHOOK \
      -d '{"text":"Build failed!"}'
```

---

### 5. Pipeline as Code

```bash
# ✅ Store pipeline config in Git
.github/workflows/ci.yml
Jenkinsfile
.gitlab-ci.yml

Benefits:
- Version controlled
- Code review
- Rollback capability
- Consistency
```

---

## Troubleshooting

### Common Issues

**1. Flaky Tests**

```yaml
# Retry flaky tests
- name: Test
  uses: nick-invision/retry@v2
  with:
    timeout_minutes: 10
    max_attempts: 3
    command: npm test
```

**2. Slow Pipelines**

```yaml
# Use caching
- uses: actions/cache@v3
  with:
    path: node_modules
    key: ${{ runner.os }}-npm-${{ hashFiles('package-lock.json') }}

# Parallel execution
jobs:
  test:
    strategy:
      matrix:
        test-suite: [unit, integration, e2e]
```

**3. Insufficient Resources**

```yaml
# Increase timeout
jobs:
  build:
    runs-on: ubuntu-latest
    timeout-minutes: 60  # Default is 360

# Use self-hosted runners
jobs:
  build:
    runs-on: self-hosted
```

**4. Secret Not Working**

```bash
# Debug (don't print secrets!)
- name: Debug
  run: |
    echo "Secret length: ${#API_KEY}"
    echo "First char: ${API_KEY:0:1}"
```

---

### Debugging Pipelines

```yaml
# Enable debug logging
# GitHub Actions: Set secret ACTIONS_STEP_DEBUG=true

# Add debug steps
- name: Debug Info
  run: |
    echo "PWD: $(pwd)"
    echo "User: $(whoami)"
    echo "Node: $(node --version)"
    ls -la
    env | sort

# SSH into runner (tmate)
- name: Setup tmate session
  if: failure()
  uses: mxschmitt/action-tmate@v3
```

---

## Interview Questions

### Q1: Explain CI vs CD vs CD
**Answer:**
```
CI (Continuous Integration):
- Automatically build and test on every commit
- Catch bugs early
- Fast feedback

CD (Continuous Delivery):
- Always ready to deploy
- Manual approval for production
- Netflix, Amazon style

CD (Continuous Deployment):
- Fully automated to production
- No human intervention
- Facebook, Google style

Key difference:
Delivery = Can deploy anytime (manual)
Deployment = Automatically deployed
```

---

### Q2: How do you handle secrets in pipelines?
**Answer:**
```
1. Use platform secret management
   - GitHub Secrets
   - Jenkins Credentials
   - GitLab CI/CD Variables

2. External secret managers
   - HashiCorp Vault
   - AWS Secrets Manager
   - Azure Key Vault

3. Best practices:
   - Never commit secrets to Git
   - Rotate regularly
   - Least privilege access
   - Audit secret usage

Example:
- name: Deploy
  env:
    API_KEY: ${{ secrets.API_KEY }}
  run: ./deploy.sh
```

---

### Q3: Design a CI/CD pipeline for a microservices app
**Answer:**
```
Pipeline stages:
1. Code Quality
   - Linting
   - Security scanning
   
2. Build
   - Build each service (parallel)
   - Create Docker images
   - Tag with commit SHA

3. Test
   - Unit tests (per service)
   - Integration tests
   - Contract tests (service boundaries)
   - E2E tests

4. Deploy
   - Dev (automatic)
   - Staging (automatic)
   - Production (manual approval)

Key considerations:
- Independent deployments
- Shared libraries versioned
- Service mesh for traffic management
- Canary deployments
- Rollback strategy
```

---

### Q4: How do you optimize slow pipelines?
**Answer:**
```
1. Caching
   - Dependencies (npm, pip, maven)
   - Build outputs
   - Docker layers

2. Parallelization
   - Run tests in parallel
   - Matrix builds

3. Fail Fast
   - Quick checks first (lint, compile)
   - Skip subsequent stages on failure

4. Selective Builds
   - Only build changed services
   - Skip tests if only docs changed

5. Optimize Tests
   - Run unit tests before integration
   - Split large test suites
   - Remove flaky tests

6. Hardware
   - Use faster runners
   - Self-hosted runners
   - SSD storage

Target: < 10 minutes for feedback
```

---

### Q5: Explain Blue-Green vs Canary deployment
**Answer:**
```
Blue-Green:
- Two identical environments
- Switch all traffic at once
- Easy rollback (switch back)
- Higher cost (2x infrastructure)

Use when:
- Need instant rollback
- Can test full environment
- Have resources for duplication

Canary:
- Gradual traffic shift (10% → 50% → 100%)
- Monitor metrics at each step
- Lower risk (limited blast radius)
- More complex routing

Use when:
- Want gradual validation
- High-risk changes
- Limited resources

Both are better than rolling update for:
- Immediate rollback
- Production testing
```

---

### Q6: How do you handle database migrations in CI/CD?
**Answer:**
```
Strategies:

1. Backward Compatible Migrations
   - Deploy migration first
   - Then deploy code
   - Allows rollback

2. Blue-Green with Migration
   - Run migration before traffic switch
   - Test on Blue before Green

3. Feature Flags
   - Deploy code with flag OFF
   - Run migration
   - Enable flag

4. Tools
   - Flyway (Java)
   - Liquibase (SQL)
   - Alembic (Python)
   - Rails migrations

Pipeline:
1. Backup database
2. Run migration (staging)
3. Test
4. Run migration (prod)
5. Deploy code
6. Verify

Critical: Always make migrations reversible!
```

---

## Quick Reference

```yaml
# GitHub Actions Template
name: CI/CD
on: [push]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: npm ci
      - run: npm test
      - run: npm run build

# Jenkins Template
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
    }
}

# GitLab CI Template
stages:
  - build
  - test
  - deploy

build:
  script:
    - npm install
    - npm run build

test:
  script:
    - npm test

deploy:
  script:
    - ./deploy.sh
  only:
    - main
```

---

## Summary

**Key Takeaways:**
1. CI/CD is automation of build, test, deploy
2. Fast feedback is critical (< 10 min)
3. Pipeline as Code (version controlled)
4. Security: secrets management, scanning
5. Deployment strategies: blue-green, canary, rolling
6. Fail fast, recover quickly
7. Monitor and measure pipeline performance

**Next Steps:**
1. Set up a simple pipeline for your project
2. Add automated tests
3. Implement deployment to staging
4. Add security scanning
5. Optimize for speed
6. Add monitoring and alerts

**Remember:**
- Automate everything
- Make it fast
- Make it reliable
- Make it secure
- Continuous improvement

**Happy Deploying! 🚀**

