# 8. CI/CD & DevOps

## Table of Contents
- [CI/CD Fundamentals](#cicd-fundamentals)
- [Jenkins Pipeline](#jenkins-pipeline)
- [Docker & Kubernetes](#docker--kubernetes)
- [Continuous Testing](#continuous-testing)
- [DevOps Best Practices](#devops-best-practices)

---

## CI/CD Fundamentals

### CI/CD Pipeline Overview

```
┌─────────────────────────────────────────────────────────────┐
│                    CI/CD Pipeline                           │
└─────────────────────────────────────────────────────────────┘

Developer Commits Code
        ↓
┌───────────────────┐
│  Source Control   │  (Git/GitHub/GitLab)
│   (SCM Trigger)   │
└─────────┬─────────┘
          ↓
┌───────────────────┐
│      Build        │  (Maven/Gradle)
│   Compile Code    │
└─────────┬─────────┘
          ↓
┌───────────────────┐
│   Static Analysis │  (SonarQube)
│   Code Quality    │
└─────────┬─────────┘
          ↓
┌───────────────────┐
│   Unit Tests      │  (JUnit/TestNG)
│   By Developers   │
└─────────┬─────────┘
          ↓
┌───────────────────┐
│  Integration Tests│  (API Tests)
│    RestAssured    │
└─────────┬─────────┘
          ↓
┌───────────────────┐
│   Deploy to QA    │  (Docker/Kubernetes)
│   Environment     │
└─────────┬─────────┘
          ↓
┌───────────────────┐
│   UI Tests        │  (Selenium)
│   Automated Tests │
└─────────┬─────────┘
          ↓
┌───────────────────┐
│  Performance Tests│  (JMeter)
│   Load Testing    │
└─────────┬─────────┘
          ↓
┌───────────────────┐
│  Security Scans   │  (OWASP ZAP)
│   Vulnerability   │
└─────────┬─────────┘
          ↓
┌───────────────────┐
│ Deploy to Staging │
│    (Manual Gate)  │
└─────────┬─────────┘
          ↓
┌───────────────────┐
│  Smoke Tests      │
│   (Staging)       │
└─────────┬─────────┘
          ↓
┌───────────────────┐
│ Deploy to Prod    │
│  (Approval Gate)  │
└─────────┬─────────┘
          ↓
┌───────────────────┐
│  Monitoring       │  (Grafana/Prometheus)
│  & Alerting       │
└───────────────────┘
```

---

## Jenkins Pipeline

### Declarative Pipeline - Complete Example

```groovy
// Jenkinsfile
pipeline {
    agent any
    
    // Tools configuration
    tools {
        maven 'Maven3.8'
        jdk 'JDK11'
    }
    
    // Environment variables
    environment {
        DOCKER_REGISTRY = 'docker.io'
        IMAGE_NAME = 'automation-tests'
        SONAR_HOST = 'http://sonarqube:9000'
        SLACK_CHANNEL = '#test-automation'
    }
    
    // Build parameters
    parameters {
        choice(
            name: 'ENVIRONMENT',
            choices: ['qa', 'staging', 'production'],
            description: 'Select environment'
        )
        choice(
            name: 'BROWSER',
            choices: ['chrome', 'firefox', 'edge'],
            description: 'Select browser'
        )
        choice(
            name: 'TEST_SUITE',
            choices: ['smoke', 'regression', 'full'],
            description: 'Select test suite'
        )
        string(
            name: 'THREAD_COUNT',
            defaultValue: '5',
            description: 'Number of parallel threads'
        )
        booleanParam(
            name: 'RUN_PERFORMANCE_TESTS',
            defaultValue: false,
            description: 'Run performance tests'
        )
    }
    
    // Trigger configuration
    triggers {
        // Poll SCM every 5 minutes
        pollSCM('H/5 * * * *')
        
        // Scheduled builds
        cron('H 2 * * 1-5')  // Weekdays at 2 AM
    }
    
    // Pipeline stages
    stages {
        
        stage('Checkout') {
            steps {
                script {
                    echo "Checking out code from repository..."
                }
                
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[
                        url: 'https://github.com/your-repo/automation.git',
                        credentialsId: 'github-credentials'
                    ]]
                ])
                
                echo "✓ Code checkout completed"
            }
        }
        
        stage('Build') {
            steps {
                echo "Building the project..."
                sh 'mvn clean compile'
                echo "✓ Build completed successfully"
            }
        }
        
        stage('Code Quality Analysis') {
            steps {
                echo "Running SonarQube analysis..."
                
                withSonarQubeEnv('SonarQube') {
                    sh '''
                        mvn sonar:sonar \
                        -Dsonar.projectKey=automation-tests \
                        -Dsonar.host.url=${SONAR_HOST}
                    '''
                }
                
                // Quality Gate check
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
                
                echo "✓ Code quality analysis passed"
            }
        }
        
        stage('Unit Tests') {
            steps {
                echo "Running unit tests..."
                sh 'mvn test -Dgroups=unit'
                
                // Publish test results
                junit '**/target/surefire-reports/*.xml'
                
                echo "✓ Unit tests completed"
            }
        }
        
        stage('Package') {
            steps {
                echo "Packaging application..."
                sh 'mvn package -DskipTests'
                
                // Archive artifacts
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
                
                echo "✓ Application packaged"
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    echo "Building Docker image..."
                    
                    dockerImage = docker.build("${IMAGE_NAME}:${BUILD_NUMBER}")
                    
                    echo "✓ Docker image built successfully"
                }
            }
        }
        
        stage('Push to Registry') {
            steps {
                script {
                    echo "Pushing Docker image to registry..."
                    
                    docker.withRegistry("https://${DOCKER_REGISTRY}", 'docker-credentials') {
                        dockerImage.push("${BUILD_NUMBER}")
                        dockerImage.push("latest")
                    }
                    
                    echo "✓ Image pushed to registry"
                }
            }
        }
        
        stage('Deploy to Test Environment') {
            steps {
                echo "Deploying to ${params.ENVIRONMENT} environment..."
                
                sh """
                    docker-compose -f docker-compose-${params.ENVIRONMENT}.yml down
                    docker-compose -f docker-compose-${params.ENVIRONMENT}.yml up -d
                """
                
                // Wait for application to be ready
                sh 'sleep 30'
                
                // Health check
                sh '''
                    curl -f http://test-env:8080/health || exit 1
                '''
                
                echo "✓ Deployment completed"
            }
        }
        
        stage('Smoke Tests') {
            steps {
                echo "Running smoke tests..."
                
                sh """
                    mvn test \
                    -Dsuite=smoke \
                    -Dbrowser=${params.BROWSER} \
                    -Denv=${params.ENVIRONMENT}
                """
                
                // Publish results
                publishHTML([
                    reportDir: 'test-output/extent-reports',
                    reportFiles: 'index.html',
                    reportName: 'Smoke Test Report'
                ])
                
                echo "✓ Smoke tests completed"
            }
        }
        
        stage('Regression Tests') {
            when {
                expression { params.TEST_SUITE == 'regression' || params.TEST_SUITE == 'full' }
            }
            steps {
                echo "Running regression tests..."
                
                sh """
                    mvn test \
                    -Dsuite=regression \
                    -Dbrowser=${params.BROWSER} \
                    -Denv=${params.ENVIRONMENT} \
                    -DthreadCount=${params.THREAD_COUNT}
                """
                
                publishHTML([
                    reportDir: 'test-output/extent-reports',
                    reportFiles: 'index.html',
                    reportName: 'Regression Test Report'
                ])
                
                echo "✓ Regression tests completed"
            }
        }
        
        stage('API Tests') {
            steps {
                echo "Running API tests..."
                
                sh """
                    mvn test \
                    -Dgroups=api \
                    -Denv=${params.ENVIRONMENT}
                """
                
                echo "✓ API tests completed"
            }
        }
        
        stage('Performance Tests') {
            when {
                expression { params.RUN_PERFORMANCE_TESTS == true }
            }
            steps {
                echo "Running performance tests..."
                
                sh '''
                    mvn jmeter:jmeter \
                    -DthreadCount=100 \
                    -DrampUp=60 \
                    -Dduration=300
                '''
                
                // Publish performance report
                perfReport sourceDataFiles: 'target/jmeter/results/*.jtl'
                
                echo "✓ Performance tests completed"
            }
        }
        
        stage('Security Scan') {
            steps {
                echo "Running security scan..."
                
                sh '''
                    docker run -t owasp/zap2docker-stable zap-baseline.py \
                    -t http://test-env:8080 \
                    -r security-report.html
                '''
                
                publishHTML([
                    reportDir: '.',
                    reportFiles: 'security-report.html',
                    reportName: 'Security Scan Report'
                ])
                
                echo "✓ Security scan completed"
            }
        }
        
        stage('Generate Reports') {
            steps {
                echo "Generating consolidated reports..."
                
                // TestNG report
                publishHTML([
                    reportDir: 'test-output',
                    reportFiles: 'index.html',
                    reportName: 'TestNG Report'
                ])
                
                // Allure report
                allure([
                    includeProperties: false,
                    jdk: '',
                    properties: [],
                    reportBuildPolicy: 'ALWAYS',
                    results: [[path: 'target/allure-results']]
                ])
                
                echo "✓ Reports generated"
            }
        }
    }
    
    // Post-build actions
    post {
        always {
            echo 'Pipeline execution completed'
            
            // Clean workspace
            cleanWs()
            
            // Archive screenshots
            archiveArtifacts artifacts: 'screenshots/**/*.png',
                           allowEmptyArchive: true
            
            // Archive logs
            archiveArtifacts artifacts: 'logs/**/*.log',
                           allowEmptyArchive: true
        }
        
        success {
            echo '✓ All stages passed successfully!'
            
            // Email notification
            emailext (
                subject: "✅ Build Successful: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
                    <h2>Build Successful</h2>
                    <p><b>Job:</b> ${env.JOB_NAME}</p>
                    <p><b>Build Number:</b> ${env.BUILD_NUMBER}</p>
                    <p><b>Environment:</b> ${params.ENVIRONMENT}</p>
                    <p><b>Browser:</b> ${params.BROWSER}</p>
                    <p><b>Duration:</b> ${currentBuild.durationString}</p>
                    <p><a href="${env.BUILD_URL}">View Build</a></p>
                    <p><a href="${env.BUILD_URL}Extent_20Report/">View Test Report</a></p>
                """,
                to: "team@example.com",
                mimeType: 'text/html'
            )
            
            // Slack notification
            slackSend (
                color: 'good',
                channel: "${SLACK_CHANNEL}",
                message: """
                    ✅ *Build Successful*
                    *Job:* ${env.JOB_NAME}
                    *Build:* #${env.BUILD_NUMBER}
                    *Environment:* ${params.ENVIRONMENT}
                    *Duration:* ${currentBuild.durationString}
                    <${env.BUILD_URL}|View Build>
                """
            )
        }
        
        failure {
            echo '❌ Build failed'
            
            // Email notification
            emailext (
                subject: "❌ Build Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
                    <h2 style="color: red;">Build Failed</h2>
                    <p><b>Job:</b> ${env.JOB_NAME}</p>
                    <p><b>Build Number:</b> ${env.BUILD_NUMBER}</p>
                    <p><b>Failed Stage:</b> ${env.STAGE_NAME}</p>
                    <p><b>Duration:</b> ${currentBuild.durationString}</p>
                    <p><a href="${env.BUILD_URL}console">View Console</a></p>
                    <p><a href="${env.BUILD_URL}">View Build</a></p>
                """,
                to: "team@example.com, leads@example.com",
                mimeType: 'text/html'
            )
            
            // Slack notification
            slackSend (
                color: 'danger',
                channel: "${SLACK_CHANNEL}",
                message: """
                    ❌ *Build Failed*
                    *Job:* ${env.JOB_NAME}
                    *Build:* #${env.BUILD_NUMBER}
                    *Failed Stage:* ${env.STAGE_NAME}
                    <${env.BUILD_URL}console|View Console>
                """
            )
        }
        
        unstable {
            echo '⚠️ Build is unstable'
            
            slackSend (
                color: 'warning',
                channel: "${SLACK_CHANNEL}",
                message: """
                    ⚠️ *Build Unstable*
                    *Job:* ${env.JOB_NAME}
                    *Build:* #${env.BUILD_NUMBER}
                    Some tests failed.
                    <${env.BUILD_URL}|View Build>
                """
            )
        }
    }
}
```

---

## Docker & Kubernetes

### Docker Configuration

```dockerfile
# Multi-stage Dockerfile for Test Automation

# Stage 1: Build stage
FROM maven:3.8-openjdk-11 AS build
WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline
COPY src ./src
RUN mvn clean package -DskipTests

# Stage 2: Test execution stage
FROM selenium/standalone-chrome:latest

# Install Java
USER root
RUN apt-get update && \
    apt-get install -y openjdk-11-jdk maven && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Set working directory
WORKDIR /tests

# Copy test artifacts
COPY --from=build /app/target/*.jar ./
COPY --from=build /app/target/lib ./lib
COPY testng.xml ./
COPY src/test/resources ./resources

# Set environment variables
ENV DISPLAY=:99
ENV BROWSER=chrome
ENV HEADLESS=true

# Create directories for outputs
RUN mkdir -p test-output screenshots logs

# Run tests
CMD ["java", "-jar", "automation-tests.jar"]
```

### Docker Compose for Test Environment

```yaml
# docker-compose.yml
version: '3.8'

services:
  # Application Under Test
  app:
    image: myapp:latest
    container_name: test-app
    ports:
      - "8080:8080"
    environment:
      - SPRING_PROFILES_ACTIVE=test
      - DB_HOST=mysql
    depends_on:
      - mysql
      - redis
    networks:
      - test-network
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  # Database
  mysql:
    image: mysql:8.0
    container_name: test-mysql
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_DATABASE: testdb
      MYSQL_USER: testuser
      MYSQL_PASSWORD: testpass
    ports:
      - "3306:3306"
    volumes:
      - mysql-data:/var/lib/mysql
      - ./init-db.sql:/docker-entrypoint-initdb.d/init.sql
    networks:
      - test-network

  # Redis Cache
  redis:
    image: redis:7-alpine
    container_name: test-redis
    ports:
      - "6379:6379"
    networks:
      - test-network

  # Selenium Hub
  selenium-hub:
    image: selenium/hub:4.15.0
    container_name: selenium-hub
    ports:
      - "4444:4444"
      - "4442:4442"
      - "4443:4443"
    networks:
      - test-network
    environment:
      GRID_MAX_SESSION: 10
      GRID_BROWSER_TIMEOUT: 300

  # Chrome Node
  chrome:
    image: selenium/node-chrome:4.15.0
    depends_on:
      - selenium-hub
    environment:
      SE_EVENT_BUS_HOST: selenium-hub
      SE_EVENT_BUS_PUBLISH_PORT: 4442
      SE_EVENT_BUS_SUBSCRIBE_PORT: 4443
      SE_NODE_MAX_SESSIONS: 3
    ports:
      - "6900:5900"
    shm_size: 2gb
    networks:
      - test-network
    deploy:
      replicas: 3

  # Firefox Node
  firefox:
    image: selenium/node-firefox:4.15.0
    depends_on:
      - selenium-hub
    environment:
      SE_EVENT_BUS_HOST: selenium-hub
      SE_EVENT_BUS_PUBLISH_PORT: 4442
      SE_EVENT_BUS_SUBSCRIBE_PORT: 4443
      SE_NODE_MAX_SESSIONS: 3
    ports:
      - "6901:5900"
    shm_size: 2gb
    networks:
      - test-network
    deploy:
      replicas: 2

  # Test Execution
  tests:
    build: .
    container_name: test-runner
    depends_on:
      - app
      - selenium-hub
    environment:
      - APP_URL=http://app:8080
      - SELENIUM_HUB_URL=http://selenium-hub:4444
      - BROWSER=chrome
      - ENV=test
    volumes:
      - ./test-output:/tests/test-output
      - ./screenshots:/tests/screenshots
      - ./logs:/tests/logs
    networks:
      - test-network
    command: >
      sh -c "
        until curl -f http://app:8080/health; do
          echo 'Waiting for app to be ready...';
          sleep 5;
        done;
        mvn clean test -Dremote=true
      "

  # Allure Report Server
  allure:
    image: frankescobar/allure-docker-service
    container_name: allure-server
    ports:
      - "5050:5050"
    volumes:
      - ./test-output/allure-results:/app/allure-results
      - ./test-output/allure-reports:/app/allure-reports
    networks:
      - test-network

networks:
  test-network:
    driver: bridge

volumes:
  mysql-data:
```

### Kubernetes Deployment

```yaml
# k8s-test-deployment.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: test-automation

---

apiVersion: v1
kind: ConfigMap
metadata:
  name: test-config
  namespace: test-automation
data:
  browser: "chrome"
  environment: "qa"
  parallel-threads: "5"

---

apiVersion: batch/v1
kind: Job
metadata:
  name: automation-tests
  namespace: test-automation
spec:
  parallelism: 3
  completions: 3
  template:
    spec:
      containers:
      - name: test-runner
        image: automation-tests:latest
        env:
        - name: BROWSER
          valueFrom:
            configMapKeyRef:
              name: test-config
              key: browser
        - name: ENV
          valueFrom:
            configMapKeyRef:
              name: test-config
              key: environment
        - name: SELENIUM_HUB_URL
          value: "http://selenium-hub:4444"
        volumeMounts:
        - name: test-results
          mountPath: /tests/test-output
      restartPolicy: Never
      volumes:
      - name: test-results
        persistentVolumeClaim:
          claimName: test-results-pvc

---

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: test-results-pvc
  namespace: test-automation
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 5Gi
```

---

## Continuous Testing

### Continuous Testing Strategy

```markdown
## Continuous Testing Implementation

### 1. Test Pyramid

```
           /\
          /  \
         / UI \         10% - UI Tests (Selenium)
        /______\
       /        \
      / Service  \     30% - API/Service Tests (RestAssured)
     /____________\
    /              \
   /  Unit Tests    \  60% - Unit Tests (JUnit/TestNG)
  /__________________\
```

### 2. Testing Stages in CI/CD

**Stage 1: Commit Stage (< 10 minutes)**
- Unit tests
- Static code analysis
- Fast feedback

**Stage 2: Acceptance Stage (< 30 minutes)**
- API tests
- Integration tests
- Smoke tests

**Stage 3: Capacity Stage (< 2 hours)**
- Performance tests
- Load tests
- Security scans

### 3. Test Execution Strategy

**On Every Commit:**
- Unit tests
- Static analysis
- Smoke tests (critical path)

**On Every PR:**
- Integration tests
- API tests
- Code coverage report

**Daily:**
- Full regression suite
- Security scans

**Weekly:**
- Performance tests
- Cross-browser tests
- Compatibility tests

**Before Release:**
- Full test suite
- UAT
- Production smoke tests
```

### Test Automation in Pipeline

```java
public class ContinuousTestingFramework {
    
    // 1. Fast feedback tests (< 5 minutes)
    @Test(groups = {"smoke", "critical"})
    public void testCriticalUserJourney() {
        // Critical path test
        loginPage.login(validUser, validPassword);
        Assert.assertTrue(homePage.isLoggedIn());
    }
    
    // 2. API contract tests
    @Test(groups = {"api", "contract"})
    public void testAPIContract() {
        given()
            .spec(requestSpec)
        .when()
            .get("/api/users")
        .then()
            .statusCode(200)
            .body(matchesJsonSchema("user-schema.json"));
    }
    
    // 3. Component tests
    @Test(groups = {"component"})
    public void testShoppingCartComponent() {
        ShoppingCart cart = new ShoppingCart();
        cart.addItem(product);
        Assert.assertEquals(1, cart.getItemCount());
    }
    
    // 4. Integration tests
    @Test(groups = {"integration"})
    public void testOrderPlacementIntegration() {
        // Test end-to-end flow
        Order order = orderService.createOrder(user, items);
        Payment payment = paymentService.processPayment(order);
        Assert.assertEquals("SUCCESS", payment.getStatus());
    }
    
    // 5. Performance tests
    @Test(groups = {"performance"})
    public void testPageLoadPerformance() {
        long startTime = System.currentTimeMillis();
        driver.get(url);
        long loadTime = System.currentTimeMillis() - startTime;
        
        Assert.assertTrue(loadTime < 3000, "Page should load in < 3 seconds");
    }
}
```

---

## DevOps Best Practices

### Infrastructure as Code

```groovy
// Jenkins Job DSL - Create jobs as code
job('automation-tests-smoke') {
    description('Smoke test suite - runs on every build')
    
    scm {
        git {
            remote {
                url('https://github.com/your-repo/automation.git')
                credentials('github-creds')
            }
            branch('main')
        }
    }
    
    triggers {
        scm('H/5 * * * *')  // Poll every 5 minutes
    }
    
    steps {
        maven {
            goals('clean test')
            properties([
                'suite': 'smoke',
                'browser': 'chrome',
                'env': 'qa'
            ])
        }
    }
    
    publishers {
        archiveArtifacts('screenshots/**')
        archiveJunit('**/target/surefire-reports/*.xml')
        
        extendedEmail {
            recipientList('team@example.com')
            triggers {
                failure {
                    subject('Smoke Tests Failed')
                    content('Check console output')
                }
            }
        }
    }
}
```

### Monitoring & Observability

```yaml
# Prometheus configuration for test monitoring
# prometheus.yml
global:
  scrape_interval: 15s
  
scrape_configs:
  - job_name: 'test-execution'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'jenkins'
    metrics_path: '/prometheus'
    static_configs:
      - targets: ['jenkins:8080']
```

```java
// Expose test metrics
public class TestMetrics {
    private static Counter testExecuted = Counter.build()
        .name("tests_executed_total")
        .help("Total tests executed")
        .labelNames("suite", "status")
        .register();
    
    private static Histogram testDuration = Histogram.build()
        .name("test_duration_seconds")
        .help("Test execution duration")
        .register();
    
    @AfterMethod
    public void recordMetrics(ITestResult result) {
        String status = result.isSuccess() ? "passed" : "failed";
        testExecuted.labels(suiteName, status).inc();
        
        double duration = (result.getEndMillis() - result.getStartMillis()) / 1000.0;
        testDuration.observe(duration);
    }
}
```

---

**Next:** [Performance & Load Testing](09-performance-load-testing.md)

