# 4. Testing Tools & Technologies

## Table of Contents
- [TestNG Advanced Features](#testng-advanced-features)
- [Maven/Gradle Build Tools](#mavengradle-build-tools)
- [Git/Version Control](#gitversion-control)
- [Jenkins/CI-CD Integration](#jenkinsci-cd-integration)
- [Docker for Test Automation](#docker-for-test-automation)
- [Cucumber/BDD Framework](#cucumberbdd-framework)

---

## TestNG Advanced Features

### TestNG Basics

```java
import org.testng.annotations.*;
import org.testng.Assert;

public class TestNGBasics {
    
    // Execution Order: @BeforeSuite -> @BeforeTest -> @BeforeClass -> 
    // @BeforeMethod -> @Test -> @AfterMethod -> @AfterClass -> 
    // @AfterTest -> @AfterSuite
    
    @BeforeSuite
    public void beforeSuite() {
        System.out.println("Before Suite - Runs once before all tests in suite");
    }
    
    @BeforeTest
    public void beforeTest() {
        System.out.println("Before Test - Runs once before <test> tag in XML");
    }
    
    @BeforeClass
    public void beforeClass() {
        System.out.println("Before Class - Runs once before first test method");
    }
    
    @BeforeMethod
    public void beforeMethod() {
        System.out.println("Before Method - Runs before each test method");
    }
    
    @Test(priority = 1, description = "First test")
    public void test1() {
        System.out.println("Test 1 executing");
        Assert.assertTrue(true);
    }
    
    @Test(priority = 2, enabled = true)
    public void test2() {
        System.out.println("Test 2 executing");
    }
    
    @Test(enabled = false)
    public void test3() {
        System.out.println("This test is disabled");
    }
    
    @AfterMethod
    public void afterMethod() {
        System.out.println("After Method - Runs after each test method");
    }
    
    @AfterClass
    public void afterClass() {
        System.out.println("After Class - Runs once after all test methods");
    }
    
    @AfterTest
    public void afterTest() {
        System.out.println("After Test - Runs once after <test> tag");
    }
    
    @AfterSuite
    public void afterSuite() {
        System.out.println("After Suite - Runs once after all tests in suite");
    }
}
```

### Advanced TestNG Features

```java
public class TestNGAdvanced {
    
    // 1. Test Dependencies
    @Test
    public void loginTest() {
        System.out.println("Login test");
    }
    
    @Test(dependsOnMethods = {"loginTest"})
    public void homePageTest() {
        System.out.println("Home page test - depends on login");
    }
    
    @Test(dependsOnMethods = {"loginTest", "homePageTest"})
    public void logoutTest() {
        System.out.println("Logout test");
    }
    
    // 2. Groups
    @Test(groups = {"smoke", "regression"})
    public void smokeTest1() {
        System.out.println("Smoke test 1");
    }
    
    @Test(groups = {"regression"})
    public void regressionTest1() {
        System.out.println("Regression test 1");
    }
    
    @Test(groups = {"sanity"})
    public void sanityTest1() {
        System.out.println("Sanity test 1");
    }
    
    // 3. Data Provider
    @DataProvider(name = "loginData")
    public Object[][] getLoginData() {
        return new Object[][] {
            {"user1@test.com", "pass1", "Login Successful"},
            {"user2@test.com", "pass2", "Login Successful"},
            {"invalid@test.com", "wrong", "Login Failed"}
        };
    }
    
    @Test(dataProvider = "loginData")
    public void testLogin(String email, String password, String expectedResult) {
        System.out.println("Testing login with: " + email);
        // Perform login
        // Assert result
    }
    
    // 4. Parallel Data Provider
    @DataProvider(name = "parallelData", parallel = true)
    public Object[][] getParallelData() {
        return new Object[][] {
            {"data1"},
            {"data2"},
            {"data3"}
        };
    }
    
    @Test(dataProvider = "parallelData")
    public void testParallel(String data) {
        System.out.println("Thread: " + Thread.currentThread().getId() + 
                           " - Data: " + data);
    }
    
    // 5. Expected Exceptions
    @Test(expectedExceptions = ArithmeticException.class)
    public void testException() {
        int result = 10 / 0;  // Will throw ArithmeticException
    }
    
    @Test(expectedExceptions = NullPointerException.class, 
          expectedExceptionsMessageRegExp = ".*null.*")
    public void testExceptionWithMessage() {
        String str = null;
        str.length();  // Will throw NPE
    }
    
    // 6. Timeout
    @Test(timeOut = 5000)  // Fail if takes more than 5 seconds
    public void testTimeout() throws InterruptedException {
        Thread.sleep(3000);
        System.out.println("Test completed within timeout");
    }
    
    // 7. Invocation Count
    @Test(invocationCount = 5)
    public void testMultipleTimes() {
        System.out.println("This test runs 5 times");
    }
    
    @Test(invocationCount = 3, threadPoolSize = 3)
    public void testConcurrent() {
        System.out.println("Thread: " + Thread.currentThread().getId());
    }
    
    // 8. Parameters from XML
    @Parameters({"browser", "url"})
    @Test
    public void testWithParameters(String browser, String url) {
        System.out.println("Browser: " + browser + ", URL: " + url);
    }
}
```

### TestNG XML Configuration

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE suite SYSTEM "https://testng.org/testng-1.0.dtd">

<!-- Complete TestNG Suite Configuration -->
<suite name="Complete Test Suite" parallel="tests" thread-count="3" verbose="1">
    
    <!-- Suite-level parameters -->
    <parameter name="browser" value="chrome"/>
    <parameter name="url" value="https://example.com"/>
    
    <!-- Listeners -->
    <listeners>
        <listener class-name="com.listeners.TestListener"/>
        <listener class-name="com.listeners.RetryListener"/>
    </listeners>
    
    <!-- Test 1: Smoke Tests -->
    <test name="Smoke Test Suite" preserve-order="true">
        <groups>
            <run>
                <include name="smoke"/>
            </run>
        </groups>
        <packages>
            <package name="com.tests.*"/>
        </packages>
    </test>
    
    <!-- Test 2: Regression Tests -->
    <test name="Regression Test Suite" parallel="methods" thread-count="5">
        <groups>
            <run>
                <include name="regression"/>
                <exclude name="flaky"/>
            </run>
        </groups>
        <classes>
            <class name="com.tests.LoginTest"/>
            <class name="com.tests.ProductTest">
                <methods>
                    <include name="testAddToCart"/>
                    <include name="testCheckout"/>
                    <exclude name="testWishlist"/>
                </methods>
            </class>
        </classes>
    </test>
    
    <!-- Test 3: Cross-browser Tests -->
    <test name="Chrome Tests">
        <parameter name="browser" value="chrome"/>
        <classes>
            <class name="com.tests.CrossBrowserTest"/>
        </classes>
    </test>
    
    <test name="Firefox Tests">
        <parameter name="browser" value="firefox"/>
        <classes>
            <class name="com.tests.CrossBrowserTest"/>
        </classes>
    </test>
    
</suite>
```

### Custom TestNG Listeners

```java
import org.testng.*;
import java.util.*;

// 1. Test Listener
public class CustomTestListener implements ITestListener {
    
    @Override
    public void onStart(ITestContext context) {
        System.out.println("Test Suite Started: " + context.getName());
    }
    
    @Override
    public void onFinish(ITestContext context) {
        System.out.println("Test Suite Finished: " + context.getName());
        
        // Print summary
        int total = context.getAllTestMethods().length;
        int passed = context.getPassedTests().size();
        int failed = context.getFailedTests().size();
        int skipped = context.getSkippedTests().size();
        
        System.out.println("=== Test Summary ===");
        System.out.println("Total: " + total);
        System.out.println("Passed: " + passed);
        System.out.println("Failed: " + failed);
        System.out.println("Skipped: " + skipped);
    }
    
    @Override
    public void onTestStart(ITestResult result) {
        System.out.println("Test Started: " + result.getName());
    }
    
    @Override
    public void onTestSuccess(ITestResult result) {
        System.out.println("Test Passed: " + result.getName());
    }
    
    @Override
    public void onTestFailure(ITestResult result) {
        System.out.println("Test Failed: " + result.getName());
        System.out.println("Reason: " + result.getThrowable().getMessage());
        
        // Capture screenshot
        String screenshotPath = ScreenshotUtil.capture(result.getName());
        System.out.println("Screenshot saved: " + screenshotPath);
    }
    
    @Override
    public void onTestSkipped(ITestResult result) {
        System.out.println("Test Skipped: " + result.getName());
    }
}

// 2. Suite Listener
public class CustomSuiteListener implements ISuiteListener {
    
    @Override
    public void onStart(ISuite suite) {
        System.out.println("Suite Started: " + suite.getName());
        // Initialize reports, logs, etc.
    }
    
    @Override
    public void onFinish(ISuite suite) {
        System.out.println("Suite Finished: " + suite.getName());
        // Generate final reports
    }
}

// 3. Retry Analyzer
public class RetryAnalyzer implements IRetryAnalyzer {
    private int retryCount = 0;
    private static final int MAX_RETRY_COUNT = 2;
    
    @Override
    public boolean retry(ITestResult result) {
        if (retryCount < MAX_RETRY_COUNT) {
            System.out.println("Retrying test: " + result.getName() + 
                               " for " + (retryCount + 1) + " time");
            retryCount++;
            return true;
        }
        return false;
    }
}

// 4. Annotation Transformer (to apply retry to all tests)
public class AnnotationTransformer implements IAnnotationTransformer {
    
    @Override
    public void transform(ITestAnnotation annotation, Class testClass,
                         Constructor testConstructor, Method testMethod) {
        annotation.setRetryAnalyzer(RetryAnalyzer.class);
    }
}
```

---

## Maven/Gradle Build Tools

### Maven Configuration (pom.xml)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    
    <groupId>com.automation</groupId>
    <artifactId>test-automation-framework</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>jar</packaging>
    
    <name>Test Automation Framework</name>
    
    <properties>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        
        <!-- Dependency Versions -->
        <selenium.version>4.15.0</selenium.version>
        <testng.version>7.8.0</testng.version>
        <rest-assured.version>5.3.2</rest-assured.version>
        <webdrivermanager.version>5.6.2</webdrivermanager.version>
        <extentreports.version>5.1.1</extentreports.version>
        <log4j.version>2.21.1</log4j.version>
        <poi.version>5.2.5</poi.version>
        <gson.version>2.10.1</gson.version>
    </properties>
    
    <dependencies>
        <!-- Selenium -->
        <dependency>
            <groupId>org.seleniumhq.selenium</groupId>
            <artifactId>selenium-java</artifactId>
            <version>${selenium.version}</version>
        </dependency>
        
        <!-- WebDriver Manager -->
        <dependency>
            <groupId>io.github.bonigarcia</groupId>
            <artifactId>webdrivermanager</artifactId>
            <version>${webdrivermanager.version}</version>
        </dependency>
        
        <!-- TestNG -->
        <dependency>
            <groupId>org.testng</groupId>
            <artifactId>testng</artifactId>
            <version>${testng.version}</version>
        </dependency>
        
        <!-- REST Assured -->
        <dependency>
            <groupId>io.rest-assured</groupId>
            <artifactId>rest-assured</artifactId>
            <version>${rest-assured.version}</version>
        </dependency>
        
        <!-- Extent Reports -->
        <dependency>
            <groupId>com.aventstack</groupId>
            <artifactId>extentreports</artifactId>
            <version>${extentreports.version}</version>
        </dependency>
        
        <!-- Log4j -->
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-core</artifactId>
            <version>${log4j.version}</version>
        </dependency>
        
        <!-- Apache POI (Excel) -->
        <dependency>
            <groupId>org.apache.poi</groupId>
            <artifactId>poi-ooxml</artifactId>
            <version>${poi.version}</version>
        </dependency>
        
        <!-- Gson -->
        <dependency>
            <groupId>com.google.code.gson</groupId>
            <artifactId>gson</artifactId>
            <version>${gson.version}</version>
        </dependency>
        
        <!-- JavaFaker (Test Data) -->
        <dependency>
            <groupId>com.github.javafaker</groupId>
            <artifactId>javafaker</artifactId>
            <version>1.0.2</version>
        </dependency>
        
        <!-- Commons IO -->
        <dependency>
            <groupId>commons-io</groupId>
            <artifactId>commons-io</artifactId>
            <version>2.15.0</version>
        </dependency>
    </dependencies>
    
    <build>
        <plugins>
            <!-- Maven Compiler Plugin -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.11.0</version>
                <configuration>
                    <source>11</source>
                    <target>11</target>
                </configuration>
            </plugin>
            
            <!-- Maven Surefire Plugin (for running tests) -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>3.2.2</version>
                <configuration>
                    <suiteXmlFiles>
                        <suiteXmlFile>
                            ${project.basedir}/testng.xml
                        </suiteXmlFile>
                    </suiteXmlFiles>
                    <systemPropertyVariables>
                        <browser>${browser}</browser>
                        <env>${env}</env>
                    </systemPropertyVariables>
                </configuration>
            </plugin>
        </plugins>
    </build>
    
    <!-- Profiles for different environments -->
    <profiles>
        <profile>
            <id>smoke</id>
            <build>
                <plugins>
                    <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-surefire-plugin</artifactId>
                        <configuration>
                            <suiteXmlFiles>
                                <suiteXmlFile>smoke-suite.xml</suiteXmlFile>
                            </suiteXmlFiles>
                        </configuration>
                    </plugin>
                </plugins>
            </build>
        </profile>
        
        <profile>
            <id>regression</id>
            <build>
                <plugins>
                    <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-surefire-plugin</artifactId>
                        <configuration>
                            <suiteXmlFiles>
                                <suiteXmlFile>regression-suite.xml</suiteXmlFile>
                            </suiteXmlFiles>
                        </configuration>
                    </plugin>
                </plugins>
            </build>
        </profile>
    </profiles>
</project>
```

### Maven Commands

```bash
# Compile the project
mvn clean compile

# Run tests
mvn clean test

# Run specific test suite
mvn clean test -DsuiteXmlFile=smoke-suite.xml

# Run with system properties
mvn clean test -Dbrowser=chrome -Denv=qa

# Run specific profile
mvn clean test -P smoke

# Skip tests
mvn clean install -DskipTests

# Run single test class
mvn test -Dtest=LoginTest

# Run specific test method
mvn test -Dtest=LoginTest#testValidLogin

# Generate site/reports
mvn site

# Install to local repository
mvn clean install

# Package project
mvn clean package

# Dependency tree
mvn dependency:tree

# Update dependencies
mvn versions:display-dependency-updates
```

### Gradle Configuration (build.gradle)

```groovy
plugins {
    id 'java'
    id 'maven-publish'
}

group = 'com.automation'
version = '1.0-SNAPSHOT'
sourceCompatibility = '11'

repositories {
    mavenCentral()
}

dependencies {
    // Selenium
    testImplementation 'org.seleniumhq.selenium:selenium-java:4.15.0'
    
    // WebDriver Manager
    testImplementation 'io.github.bonigarcia:webdrivermanager:5.6.2'
    
    // TestNG
    testImplementation 'org.testng:testng:7.8.0'
    
    // REST Assured
    testImplementation 'io.rest-assured:rest-assured:5.3.2'
    
    // Extent Reports
    testImplementation 'com.aventstack:extentreports:5.1.1'
    
    // Log4j
    testImplementation 'org.apache.logging.log4j:log4j-core:2.21.1'
    
    // Apache POI
    testImplementation 'org.apache.poi:poi-ooxml:5.2.5'
    
    // Gson
    testImplementation 'com.google.code.gson:gson:2.10.1'
}

test {
    useTestNG() {
        suites 'testng.xml'
        
        // System properties
        systemProperty 'browser', System.getProperty('browser', 'chrome')
        systemProperty 'env', System.getProperty('env', 'qa')
    }
    
    testLogging {
        events "passed", "skipped", "failed"
        showStandardStreams = true
    }
}

// Task for smoke tests
task smokeTest(type: Test) {
    useTestNG() {
        suites 'smoke-suite.xml'
    }
}

// Task for regression tests
task regressionTest(type: Test) {
    useTestNG() {
        suites 'regression-suite.xml'
    }
}
```

---

## Git/Version Control

### Git Best Practices for Test Automation

```bash
# Initialize repository
git init

# Add .gitignore
cat > .gitignore << EOF
# Maven
target/
pom.xml.tag
pom.xml.releaseBackup
pom.xml.versionsBackup

# Gradle
.gradle/
build/

# IDE
.idea/
*.iml
.vscode/
*.swp
*.swo

# Test Output
test-output/
screenshots/
logs/
*.log

# OS
.DS_Store
Thumbs.db

# Sensitive
**/config/credentials.properties
*.env
EOF

# Clone repository
git clone https://github.com/username/automation-framework.git

# Create feature branch
git checkout -b feature/add-login-tests

# Check status
git status

# Add files
git add .
git add src/test/java/LoginTest.java

# Commit changes
git commit -m "feat: Add login test cases"

# Commit message conventions:
# feat: New feature
# fix: Bug fix
# test: Adding tests
# refactor: Code refactoring
# docs: Documentation changes
# style: Code style changes
# chore: Maintenance tasks

# Push to remote
git push origin feature/add-login-tests

# Pull latest changes
git pull origin main

# Create pull request (on GitHub/GitLab)

# Merge after review
git checkout main
git merge feature/add-login-tests
git push origin main

# Delete feature branch
git branch -d feature/add-login-tests
git push origin --delete feature/add-login-tests

# View commit history
git log --oneline --graph

# Undo last commit (keep changes)
git reset --soft HEAD~1

# Undo last commit (discard changes)
git reset --hard HEAD~1

# Stash changes
git stash
git stash pop

# View branches
git branch -a

# Switch branch
git checkout develop

# Rebase
git rebase main

# Cherry-pick
git cherry-pick <commit-hash>

# Tag release
git tag -a v1.0.0 -m "Release version 1.0.0"
git push origin v1.0.0
```

### Git Workflow for Test Automation Teams

```
1. Main Branch (main/master)
   - Production-ready code
   - Protected branch
   - Requires PR approval

2. Develop Branch
   - Integration branch
   - Contains latest features

3. Feature Branches (feature/*)
   - New test development
   - Bug fixes
   - Create from develop
   - Merge back to develop

4. Release Branches (release/*)
   - Prepare for production release
   - Version bumps
   - Final testing

5. Hotfix Branches (hotfix/*)
   - Critical production fixes
   - Create from main
   - Merge to main and develop

Example workflow:
develop
  │
  ├── feature/login-tests
  ├── feature/api-tests
  ├── feature/performance-tests
  │
  └── release/v1.0.0
       │
       └── main (v1.0.0)
```

---

## Jenkins/CI-CD Integration

### Jenkins Pipeline for Test Automation

```groovy
// Jenkinsfile - Declarative Pipeline
pipeline {
    agent any
    
    tools {
        maven 'Maven3.8'
        jdk 'JDK11'
    }
    
    parameters {
        choice(name: 'BROWSER', choices: ['chrome', 'firefox', 'edge'], 
               description: 'Select browser for test execution')
        choice(name: 'ENVIRONMENT', choices: ['qa', 'staging', 'production'], 
               description: 'Select environment')
        choice(name: 'TEST_SUITE', choices: ['smoke', 'regression', 'sanity'], 
               description: 'Select test suite to run')
        string(name: 'THREAD_COUNT', defaultValue: '3', 
               description: 'Number of parallel threads')
    }
    
    environment {
        SCREENSHOT_PATH = "${WORKSPACE}/screenshots"
        REPORT_PATH = "${WORKSPACE}/test-output/extent-reports"
    }
    
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', 
                    url: 'https://github.com/your-repo/automation.git',
                    credentialsId: 'github-credentials'
                
                echo "Code checked out successfully"
            }
        }
        
        stage('Build') {
            steps {
                sh 'mvn clean compile'
                echo "Build completed"
            }
        }
        
        stage('Run Tests') {
            steps {
                script {
                    try {
                        sh """
                            mvn clean test \
                            -Dbrowser=${params.BROWSER} \
                            -Denv=${params.ENVIRONMENT} \
                            -Dsuite=${params.TEST_SUITE} \
                            -DthreadCount=${params.THREAD_COUNT}
                        """
                    } catch (Exception e) {
                        currentBuild.result = 'UNSTABLE'
                        echo "Tests failed but continuing pipeline"
                    }
                }
            }
        }
        
        stage('Generate Reports') {
            steps {
                script {
                    // Publish HTML Report
                    publishHTML([
                        reportDir: 'test-output/extent-reports',
                        reportFiles: 'index.html',
                        reportName: 'Extent Report',
                        keepAll: true,
                        alwaysLinkToLastBuild: true
                    ])
                    
                    // Publish TestNG Report
                    publishHTML([
                        reportDir: 'test-output',
                        reportFiles: 'index.html',
                        reportName: 'TestNG Report'
                    ])
                }
            }
        }
        
        stage('Archive Artifacts') {
            steps {
                archiveArtifacts artifacts: 'screenshots/**/*.png', 
                                 allowEmptyArchive: true
                archiveArtifacts artifacts: 'logs/**/*.log', 
                                 allowEmptyArchive: true
            }
        }
        
        stage('Publish Test Results') {
            steps {
                junit 'target/surefire-reports/*.xml'
            }
        }
    }
    
    post {
        always {
            echo 'Pipeline completed'
            cleanWs()  // Clean workspace
        }
        
        success {
            echo 'All tests passed!'
            emailext (
                subject: "✅ Test Execution Passed: ${env.JOB_NAME} - ${env.BUILD_NUMBER}",
                body: """
                    <h2>Test Execution Successful</h2>
                    <p><b>Job:</b> ${env.JOB_NAME}</p>
                    <p><b>Build Number:</b> ${env.BUILD_NUMBER}</p>
                    <p><b>Browser:</b> ${params.BROWSER}</p>
                    <p><b>Environment:</b> ${params.ENVIRONMENT}</p>
                    <p><b>Test Suite:</b> ${params.TEST_SUITE}</p>
                    <p><a href="${env.BUILD_URL}">View Build</a></p>
                    <p><a href="${env.BUILD_URL}Extent_20Report/">View Test Report</a></p>
                """,
                to: "team@example.com",
                mimeType: 'text/html'
            )
            
            // Slack Notification
            slackSend (
                color: 'good',
                message: "✅ Tests Passed: ${env.JOB_NAME} - ${env.BUILD_NUMBER}\n" +
                        "Browser: ${params.BROWSER}\n" +
                        "View: ${env.BUILD_URL}"
            )
        }
        
        failure {
            echo 'Tests failed!'
            emailext (
                subject: "❌ Test Execution Failed: ${env.JOB_NAME} - ${env.BUILD_NUMBER}",
                body: """
                    <h2>Test Execution Failed</h2>
                    <p><b>Job:</b> ${env.JOB_NAME}</p>
                    <p><b>Build Number:</b> ${env.BUILD_NUMBER}</p>
                    <p><b>Browser:</b> ${params.BROWSER}</p>
                    <p><b>Environment:</b> ${params.ENVIRONMENT}</p>
                    <p><a href="${env.BUILD_URL}">View Build</a></p>
                    <p><a href="${env.BUILD_URL}Extent_20Report/">View Test Report</a></p>
                """,
                to: "team@example.com",
                mimeType: 'text/html'
            )
            
            slackSend (
                color: 'danger',
                message: "❌ Tests Failed: ${env.JOB_NAME} - ${env.BUILD_NUMBER}\n" +
                        "View: ${env.BUILD_URL}"
            )
        }
        
        unstable {
            echo 'Some tests failed'
            emailext (
                subject: "⚠️ Test Execution Unstable: ${env.JOB_NAME}",
                body: "Some tests failed. Check: ${env.BUILD_URL}",
                to: "team@example.com"
            )
        }
    }
}
```

### Jenkins Job DSL

```groovy
// Create Jenkins jobs programmatically
job('automation-smoke-tests') {
    description('Run smoke test suite')
    
    parameters {
        choiceParam('BROWSER', ['chrome', 'firefox'], 'Browser')
        stringParam('ENVIRONMENT', 'qa', 'Environment')
    }
    
    scm {
        git {
            remote {
                url('https://github.com/your-repo/automation.git')
                credentials('github-credentials')
            }
            branch('main')
        }
    }
    
    triggers {
        cron('H 2 * * *')  // Run daily at 2 AM
    }
    
    steps {
        maven {
            goals('clean test')
            properties('browser': '${BROWSER}', 'env': '${ENVIRONMENT}')
        }
    }
    
    publishers {
        archiveArtifacts('screenshots/**')
        archiveJunit('**/target/surefire-reports/*.xml')
        extendedEmail {
            recipientList('team@example.com')
            defaultSubject('Test Results: $PROJECT_NAME - Build # $BUILD_NUMBER')
        }
    }
}
```

---

## Docker for Test Automation

### Dockerfile for Test Environment

```dockerfile
# Dockerfile for Selenium Tests
FROM maven:3.8-openjdk-11

# Install Chrome
RUN wget -q -O - https://dl-ssl.google.com/linux/linux_signing_key.pub | apt-key add - && \
    echo "deb [arch=amd64] http://dl.google.com/linux/chrome/deb/ stable main" >> /etc/apt/sources.list.d/google.list && \
    apt-get update && \
    apt-get install -y google-chrome-stable && \
    rm -rf /var/lib/apt/lists/*

# Install ChromeDriver
RUN CHROME_DRIVER_VERSION=`curl -sS chromedriver.storage.googleapis.com/LATEST_RELEASE` && \
    wget -N http://chromedriver.storage.googleapis.com/$CHROME_DRIVER_VERSION/chromedriver_linux64.zip && \
    unzip chromedriver_linux64.zip && \
    rm chromedriver_linux64.zip && \
    mv chromedriver /usr/local/bin/chromedriver && \
    chmod +x /usr/local/bin/chromedriver

# Set display port
ENV DISPLAY=:99

# Create app directory
WORKDIR /app

# Copy project files
COPY pom.xml .
COPY src ./src
COPY testng.xml .

# Download dependencies
RUN mvn dependency:resolve

# Run tests
CMD ["mvn", "clean", "test"]
```

### Docker Compose for Selenium Grid

```yaml
# docker-compose.yml
version: '3.8'

services:
  # Selenium Hub
  selenium-hub:
    image: selenium/hub:4.15.0
    container_name: selenium-hub
    ports:
      - "4444:4444"
      - "4442:4442"
      - "4443:4443"
    environment:
      GRID_MAX_SESSION: 10
      GRID_BROWSER_TIMEOUT: 300
      GRID_TIMEOUT: 300
    networks:
      - selenium-grid
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:4444/status"]
      interval: 30s
      timeout: 10s
      retries: 3
  
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
      SE_NODE_SESSION_TIMEOUT: 300
    ports:
      - "6900:5900"  # VNC port
    shm_size: 2gb
    networks:
      - selenium-grid
    deploy:
      replicas: 2
  
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
      - selenium-grid
    deploy:
      replicas: 2
  
  # Test Execution Container
  tests:
    build: .
    depends_on:
      - selenium-hub
    environment:
      SELENIUM_HUB_URL: http://selenium-hub:4444
      BROWSER: chrome
      ENV: qa
    volumes:
      - ./test-output:/app/test-output
      - ./screenshots:/app/screenshots
      - ./logs:/app/logs
    networks:
      - selenium-grid
    command: ["mvn", "clean", "test", "-Dremote=true"]

networks:
  selenium-grid:
    driver: bridge
```

### Docker Commands

```bash
# Build image
docker build -t automation-tests:latest .

# Run container
docker run -it automation-tests:latest

# Run with environment variables
docker run -e BROWSER=chrome -e ENV=qa automation-tests:latest

# Run with volume mounting
docker run -v $(pwd)/test-output:/app/test-output automation-tests:latest

# Docker Compose commands
docker-compose up -d                  # Start all services
docker-compose down                   # Stop all services
docker-compose ps                     # List running services
docker-compose logs selenium-hub      # View logs
docker-compose scale chrome=5         # Scale chrome nodes to 5

# View running containers
docker ps

# Stop container
docker stop <container-id>

# Remove container
docker rm <container-id>

# View images
docker images

# Remove image
docker rmi automation-tests:latest

# Prune unused resources
docker system prune -a
```

---

## Cucumber/BDD Framework

### Cucumber Setup

```xml
<!-- Add to pom.xml -->
<dependencies>
    <dependency>
        <groupId>io.cucumber</groupId>
        <artifactId>cucumber-java</artifactId>
        <version>7.14.0</version>
    </dependency>
    
    <dependency>
        <groupId>io.cucumber</groupId>
        <artifactId>cucumber-testng</artifactId>
        <version>7.14.0</version>
    </dependency>
    
    <dependency>
        <groupId>io.cucumber</groupId>
        <artifactId>cucumber-picocontainer</artifactId>
        <version>7.14.0</version>
    </dependency>
</dependencies>
```

### Feature Files

```gherkin
# login.feature
Feature: User Login Functionality
  As a user
  I want to login to the application
  So that I can access my account

  Background:
    Given user is on login page

  @smoke @regression
  Scenario: Successful login with valid credentials
    When user enters username "user@test.com"
    And user enters password "Test@123"
    And user clicks on login button
    Then user should be redirected to home page
    And welcome message should be displayed

  @regression
  Scenario: Login with invalid credentials
    When user enters username "invalid@test.com"
    And user enters password "wrongpass"
    And user clicks on login button
    Then error message "Invalid credentials" should be displayed
    And user should remain on login page

  @regression
  Scenario Outline: Login with multiple users
    When user enters username "<username>"
    And user enters password "<password>"
    And user clicks on login button
    Then login result should be "<result>"

    Examples:
      | username          | password  | result  |
      | user1@test.com    | Pass@123  | success |
      | user2@test.com    | Test@456  | success |
      | invalid@test.com  | wrong     | failure |

  @smoke
  Scenario: Login with data table
    When user logs in with following credentials:
      | username       | password |
      | test@email.com | Pass@123 |
    Then user should see dashboard
```

### Step Definitions

```java
package stepdefinitions;

import io.cucumber.java.en.*;
import io.cucumber.datatable.DataTable;
import org.testng.Assert;
import pages.LoginPage;
import pages.HomePage;

public class LoginSteps {
    private LoginPage loginPage;
    private HomePage homePage;
    
    @Given("user is on login page")
    public void userIsOnLoginPage() {
        loginPage = new LoginPage();
        loginPage.navigateToLoginPage();
    }
    
    @When("user enters username {string}")
    public void userEntersUsername(String username) {
        loginPage.enterUsername(username);
    }
    
    @When("user enters password {string}")
    public void userEntersPassword(String password) {
        loginPage.enterPassword(password);
    }
    
    @When("user clicks on login button")
    public void userClicksOnLoginButton() {
        homePage = loginPage.clickLogin();
    }
    
    @Then("user should be redirected to home page")
    public void userShouldBeRedirectedToHomePage() {
        Assert.assertTrue(homePage.isHomePageDisplayed());
    }
    
    @Then("welcome message should be displayed")
    public void welcomeMessageShouldBeDisplayed() {
        Assert.assertTrue(homePage.isWelcomeMessageDisplayed());
    }
    
    @Then("error message {string} should be displayed")
    public void errorMessageShouldBeDisplayed(String expectedMessage) {
        String actualMessage = loginPage.getErrorMessage();
        Assert.assertEquals(actualMessage, expectedMessage);
    }
    
    @Then("user should remain on login page")
    public void userShouldRemainOnLoginPage() {
        Assert.assertTrue(loginPage.isLoginPageDisplayed());
    }
    
    @Then("login result should be {string}")
    public void loginResultShouldBe(String expectedResult) {
        if (expectedResult.equals("success")) {
            Assert.assertTrue(homePage.isHomePageDisplayed());
        } else {
            Assert.assertTrue(loginPage.getErrorMessage().length() > 0);
        }
    }
    
    @When("user logs in with following credentials:")
    public void userLogsInWithFollowingCredentials(DataTable dataTable) {
        Map<String, String> credentials = dataTable.asMap(String.class, String.class);
        loginPage.enterUsername(credentials.get("username"));
        loginPage.enterPassword(credentials.get("password"));
        homePage = loginPage.clickLogin();
    }
    
    @Then("user should see dashboard")
    public void userShouldSeeDashboard() {
        Assert.assertTrue(homePage.isDashboardDisplayed());
    }
}
```

### Hooks

```java
package hooks;

import io.cucumber.java.*;
import config.DriverManager;
import utils.ScreenshotUtil;

public class Hooks {
    
    @Before
    public void setUp(Scenario scenario) {
        System.out.println("Starting scenario: " + scenario.getName());
        DriverManager.initDriver();
    }
    
    @Before("@api")
    public void setUpAPI() {
        System.out.println("Setting up API tests");
        // API-specific setup
    }
    
    @After
    public void tearDown(Scenario scenario) {
        if (scenario.isFailed()) {
            // Capture screenshot
            byte[] screenshot = ScreenshotUtil.captureAsBytes();
            scenario.attach(screenshot, "image/png", scenario.getName());
        }
        
        DriverManager.quitDriver();
        System.out.println("Scenario completed: " + scenario.getName());
    }
    
    @AfterStep
    public void afterStep(Scenario scenario) {
        // Capture screenshot after each step (if needed)
        if (scenario.isFailed()) {
            byte[] screenshot = ScreenshotUtil.captureAsBytes();
            scenario.attach(screenshot, "image/png", "Failed Step");
        }
    }
}
```

### Test Runner

```java
package runners;

import io.cucumber.testng.AbstractTestNGCucumberTests;
import io.cucumber.testng.CucumberOptions;
import org.testng.annotations.DataProvider;

@CucumberOptions(
    features = "src/test/resources/features",
    glue = {"stepdefinitions", "hooks"},
    tags = "@smoke or @regression",
    plugin = {
        "pretty",
        "html:target/cucumber-reports/cucumber.html",
        "json:target/cucumber-reports/cucumber.json",
        "junit:target/cucumber-reports/cucumber.xml",
        "com.aventstack.extentreports.cucumber.adapter.ExtentCucumberAdapter:"
    },
    monochrome = true,
    dryRun = false
)
public class TestRunner extends AbstractTestNGCucumberTests {
    
    @Override
    @DataProvider(parallel = true)
    public Object[][] scenarios() {
        return super.scenarios();
    }
}
```

---

**Next:** [Database & SQL](06-database-sql.md)

