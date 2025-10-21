# 9. Performance & Load Testing

## Table of Contents
- [Performance Testing Fundamentals](#performance-testing-fundamentals)
- [JMeter Setup and Configuration](#jmeter-setup-and-configuration)
- [Performance Metrics](#performance-metrics)
- [Load Testing Strategies](#load-testing-strategies)
- [Performance Test Automation](#performance-test-automation)
- [Bottleneck Identification](#bottleneck-identification)

---

## Performance Testing Fundamentals

### Types of Performance Testing

```markdown
1. **Load Testing**
   - Test system behavior under expected load
   - Example: 100 concurrent users

2. **Stress Testing**
   - Test system beyond normal capacity
   - Find breaking point
   - Example: Increase load until system fails

3. **Spike Testing**
   - Sudden increase/decrease in load
   - Example: Flash sale scenario

4. **Endurance Testing (Soak Testing)**
   - System behavior over extended period
   - Memory leaks, resource depletion
   - Example: 24-hour continuous load

5. **Scalability Testing**
   - Test system's ability to scale
   - Vertical vs Horizontal scaling

6. **Volume Testing**
   - Large amount of data in database
   - Test with millions of records
```

### Performance Testing Process

```
1. Requirements Gathering
   ↓
2. Test Planning
   ↓
3. Test Design
   ↓
4. Test Environment Setup
   ↓
5. Test Execution
   ↓
6. Result Analysis
   ↓
7. Optimization
   ↓
8. Re-test
```

---

## JMeter Setup and Configuration

### JMeter Test Plan Structure

```xml
Test Plan
├── Thread Group (Users)
│   ├── HTTP Request Defaults
│   ├── HTTP Cookie Manager
│   ├── HTTP Cache Manager
│   ├── Samplers
│   │   ├── HTTP Request 1 (Login)
│   │   ├── HTTP Request 2 (Browse Products)
│   │   ├── HTTP Request 3 (Add to Cart)
│   │   └── HTTP Request 4 (Checkout)
│   ├── Config Elements
│   │   ├── CSV Data Set Config
│   │   └── User Defined Variables
│   ├── Assertions
│   │   ├── Response Assertion
│   │   └── Duration Assertion
│   ├── Timers
│   │   └── Constant Timer
│   └── Listeners
│       ├── View Results Tree
│       ├── Summary Report
│       ├── Aggregate Report
│       └── Graph Results
└── Backend Listener (InfluxDB/Grafana)
```

### JMeter Script Example

```xml
<?xml version="1.0" encoding="UTF-8"?>
<jmeterTestPlan version="1.2">
  <hashTree>
    <TestPlan guiclass="TestPlanGui" testclass="TestPlan" testname="E-commerce Performance Test">
      <stringProp name="TestPlan.comments">Performance test for e-commerce application</stringProp>
      <boolProp name="TestPlan.functional_mode">false</boolProp>
      <boolProp name="TestPlan.serialize_threadgroups">false</boolProp>
      
      <elementProp name="TestPlan.user_defined_variables" elementType="Arguments">
        <collectionProp name="Arguments.arguments">
          <elementProp name="BASE_URL" elementType="Argument">
            <stringProp name="Argument.name">BASE_URL</stringProp>
            <stringProp name="Argument.value">https://example.com</stringProp>
          </elementProp>
          <elementProp name="PORT" elementType="Argument">
            <stringProp name="Argument.name">PORT</stringProp>
            <stringProp name="Argument.value">443</stringProp>
          </elementProp>
        </collectionProp>
      </elementProp>
    </TestPlan>
    
    <hashTree>
      <!-- Thread Group -->
      <ThreadGroup guiclass="ThreadGroupGui" testclass="ThreadGroup" testname="User Threads">
        <stringProp name="ThreadGroup.num_threads">100</stringProp>
        <stringProp name="ThreadGroup.ramp_time">60</stringProp>
        <longProp name="ThreadGroup.duration">300</longProp>
        <stringProp name="ThreadGroup.delay">0</stringProp>
        <boolProp name="ThreadGroup.scheduler">true</boolProp>
      </ThreadGroup>
      
      <hashTree>
        <!-- HTTP Request Defaults -->
        <ConfigTestElement guiclass="HttpDefaultsGui" testclass="ConfigTestElement" testname="HTTP Defaults">
          <elementProp name="HTTPsampler.Arguments" elementType="Arguments">
            <collectionProp name="Arguments.arguments"/>
          </elementProp>
          <stringProp name="HTTPSampler.domain">${BASE_URL}</stringProp>
          <stringProp name="HTTPSampler.port">${PORT}</stringProp>
          <stringProp name="HTTPSampler.protocol">https</stringProp>
        </ConfigTestElement>
        
        <!-- HTTP Cookie Manager -->
        <CookieManager guiclass="CookiePanel" testclass="CookieManager" testname="HTTP Cookie Manager">
          <collectionProp name="CookieManager.cookies"/>
          <boolProp name="CookieManager.clearEachIteration">false</boolProp>
        </CookieManager>
        
        <!-- Login Request -->
        <HTTPSamplerProxy guiclass="HttpTestSampleGui" testclass="HTTPSamplerProxy" testname="Login">
          <elementProp name="HTTPsampler.Arguments" elementType="Arguments">
            <collectionProp name="Arguments.arguments">
              <elementProp name="username" elementType="HTTPArgument">
                <boolProp name="HTTPArgument.always_encode">true</boolProp>
                <stringProp name="Argument.value">${username}</stringProp>
                <stringProp name="Argument.metadata">=</stringProp>
                <boolProp name="HTTPArgument.use_equals">true</boolProp>
                <stringProp name="Argument.name">username</stringProp>
              </elementProp>
              <elementProp name="password" elementType="HTTPArgument">
                <boolProp name="HTTPArgument.always_encode">true</boolProp>
                <stringProp name="Argument.value">${password}</stringProp>
                <stringProp name="Argument.metadata">=</stringProp>
                <boolProp name="HTTPArgument.use_equals">true</boolProp>
                <stringProp name="Argument.name">password</stringProp>
              </elementProp>
            </collectionProp>
          </elementProp>
          <stringProp name="HTTPSampler.path">/api/login</stringProp>
          <stringProp name="HTTPSampler.method">POST</stringProp>
        </HTTPSamplerProxy>
        
        <!-- Response Assertion -->
        <ResponseAssertion guiclass="AssertionGui" testclass="ResponseAssertion" testname="Response Code Assertion">
          <collectionProp name="Asserion.test_strings">
            <stringProp name="49586">200</stringProp>
          </collectionProp>
          <stringProp name="Assertion.test_field">Assertion.response_code</stringProp>
          <boolProp name="Assertion.assume_success">false</boolProp>
          <intProp name="Assertion.test_type">8</intProp>
        </ResponseAssertion>
        
        <!-- Duration Assertion -->
        <DurationAssertion guiclass="DurationAssertionGui" testclass="DurationAssertion" testname="Duration Assertion">
          <stringProp name="DurationAssertion.duration">3000</stringProp>
        </DurationAssertion>
      </hashTree>
    </hashTree>
  </hashTree>
</jmeterTestPlan>
```

### JMeter Command Line Execution

```bash
# Basic execution
jmeter -n -t test-plan.jmx -l results.jtl

# With custom properties
jmeter -n -t test-plan.jmx \
  -l results.jtl \
  -Jusers=100 \
  -Jrampup=60 \
  -Jduration=300 \
  -e -o ./html-report

# Distributed testing
jmeter -n -t test-plan.jmx \
  -R server1,server2,server3 \
  -l results.jtl

# Generate report from existing results
jmeter -g results.jtl -o ./html-report
```

---

## Performance Metrics

### Key Performance Indicators (KPIs)

```java
public class PerformanceMetrics {
    
    public static class Metrics {
        // Response Time Metrics
        private double averageResponseTime;
        private double minResponseTime;
        private double maxResponseTime;
        private double percentile90;
        private double percentile95;
        private double percentile99;
        
        // Throughput Metrics
        private double requestsPerSecond;
        private double transactionsPerSecond;
        
        // Error Metrics
        private int totalRequests;
        private int failedRequests;
        private double errorRate;
        
        // Resource Utilization
        private double cpuUsage;
        private double memoryUsage;
        private double diskIO;
        private double networkIO;
        
        // Calculate error rate
        public double calculateErrorRate() {
            return (failedRequests * 100.0) / totalRequests;
        }
        
        // Check if metrics meet SLA
        public boolean meetsSLA() {
            return averageResponseTime < 2000 &&  // < 2 seconds
                   percentile95 < 3000 &&          // 95th percentile < 3 seconds
                   errorRate < 1.0 &&              // < 1% error rate
                   requestsPerSecond > 100;        // > 100 requests/sec
        }
        
        public void printReport() {
            System.out.println("=== Performance Metrics Report ===");
            System.out.println("\nResponse Time:");
            System.out.printf("  Average: %.2f ms%n", averageResponseTime);
            System.out.printf("  Min: %.2f ms%n", minResponseTime);
            System.out.printf("  Max: %.2f ms%n", maxResponseTime);
            System.out.printf("  90th Percentile: %.2f ms%n", percentile90);
            System.out.printf("  95th Percentile: %.2f ms%n", percentile95);
            System.out.printf("  99th Percentile: %.2f ms%n", percentile99);
            
            System.out.println("\nThroughput:");
            System.out.printf("  Requests/sec: %.2f%n", requestsPerSecond);
            System.out.printf("  Transactions/sec: %.2f%n", transactionsPerSecond);
            
            System.out.println("\nErrors:");
            System.out.printf("  Total Requests: %d%n", totalRequests);
            System.out.printf("  Failed Requests: %d%n", failedRequests);
            System.out.printf("  Error Rate: %.2f%%%n", errorRate);
            
            System.out.println("\nResource Utilization:");
            System.out.printf("  CPU Usage: %.2f%%%n", cpuUsage);
            System.out.printf("  Memory Usage: %.2f%%%n", memoryUsage);
            
            System.out.println("\nSLA Status: " + 
                (meetsSLA() ? "✓ PASSED" : "✗ FAILED"));
        }
    }
}
```

### Performance Benchmarks

```markdown
## Performance Acceptance Criteria

### Response Time Targets

| Transaction | Average | 90th %ile | 95th %ile | 99th %ile |
|-------------|---------|-----------|-----------|-----------|
| Homepage    | < 1s    | < 1.5s    | < 2s      | < 3s      |
| Search      | < 2s    | < 3s      | < 4s      | < 5s      |
| Product Page| < 1.5s  | < 2s      | < 2.5s    | < 3.5s    |
| Add to Cart | < 0.5s  | < 1s      | < 1.5s    | < 2s      |
| Checkout    | < 3s    | < 4s      | < 5s      | < 6s      |
| Payment     | < 5s    | < 7s      | < 8s      | < 10s     |

### Throughput Targets

| Scenario | Requests/sec | Users | Duration |
|----------|-------------|-------|----------|
| Normal Load | 100 | 500 | 1 hour |
| Peak Load | 200 | 1000 | 30 min |
| Stress Test | 500+ | 2000+ | 15 min |

### Resource Utilization Limits

- CPU Usage: < 70%
- Memory Usage: < 80%
- Disk I/O: < 70%
- Network Bandwidth: < 60%

### Error Rate Targets

- Error Rate: < 1%
- Timeout Rate: < 0.5%
- Server Errors (5xx): < 0.1%
```

---

## Load Testing Strategies

### Load Test Scenarios

```java
public class LoadTestScenarios {
    
    // 1. Baseline Test
    public void baselineTest() {
        System.out.println("Baseline Test");
        System.out.println("- Users: 10");
        System.out.println("- Duration: 10 minutes");
        System.out.println("- Purpose: Establish baseline metrics");
    }
    
    // 2. Load Test
    public void loadTest() {
        System.out.println("Load Test");
        System.out.println("- Users: 500");
        System.out.println("- Ramp-up: 5 minutes");
        System.out.println("- Duration: 1 hour");
        System.out.println("- Purpose: Test under expected load");
    }
    
    // 3. Stress Test
    public void stressTest() {
        System.out.println("Stress Test");
        System.out.println("- Start: 500 users");
        System.out.println("- Increment: +100 users every 5 min");
        System.out.println("- Duration: Until system breaks");
        System.out.println("- Purpose: Find breaking point");
    }
    
    // 4. Spike Test
    public void spikeTest() {
        System.out.println("Spike Test");
        System.out.println("- Normal: 100 users");
        System.out.println("- Spike: 1000 users (sudden)");
        System.out.println("- Duration: 10 minutes");
        System.out.println("- Purpose: Test sudden traffic increase");
    }
    
    // 5. Endurance Test
    public void enduranceTest() {
        System.out.println("Endurance Test");
        System.out.println("- Users: 300");
        System.out.println("- Duration: 24 hours");
        System.out.println("- Purpose: Detect memory leaks");
    }
}
```

### Think Time and Pacing

```java
public class LoadTestConfiguration {
    
    // Think time - time user spends reading/thinking
    public static final int MIN_THINK_TIME = 3000;  // 3 seconds
    public static final int MAX_THINK_TIME = 10000; // 10 seconds
    
    // Pacing - delay between iterations
    public static final int PACING = 60000; // 1 minute
    
    public void configureRealisticLoad() {
        /*
         * Realistic User Journey:
         * 
         * 1. Homepage (think: 5s)
         * 2. Search (think: 3s)
         * 3. View Product (think: 10s)
         * 4. Add to Cart (think: 2s)
         * 5. Continue Shopping or Checkout
         * 
         * Total journey time: ~2-3 minutes
         * Pacing: 1 minute between iterations
         */
    }
    
    public int calculateRequiredUsers(int targetTPS, int avgResponseTime) {
        // Little's Law: L = λ * W
        // Users = TPS * (Response Time + Think Time)
        
        int thinkTime = (MIN_THINK_TIME + MAX_THINK_TIME) / 2;
        int totalTime = avgResponseTime + thinkTime;
        
        return (targetTPS * totalTime) / 1000;
    }
}
```

---

## Performance Test Automation

### Gatling Framework (Scala-based)

```scala
import io.gatling.core.Predef._
import io.gatling.http.Predef._
import scala.concurrent.duration._

class EcommerceLoadTest extends Simulation {
  
  // HTTP Configuration
  val httpConf = http
    .baseUrl("https://example.com")
    .acceptHeader("application/json")
    .userAgentHeader("Mozilla/5.0")
    .acceptEncodingHeader("gzip, deflate")
  
  // Scenario Definition
  val scn = scenario("User Journey")
    .exec(
      http("Homepage")
        .get("/")
        .check(status.is(200))
    )
    .pause(5.seconds)
    .exec(
      http("Login")
        .post("/api/login")
        .body(StringBody("""{"username":"${username}","password":"${password}"}"""))
        .check(status.is(200))
        .check(jsonPath("$.token").saveAs("authToken"))
    )
    .pause(3.seconds)
    .exec(
      http("Search Products")
        .get("/api/products?q=laptop")
        .header("Authorization", "Bearer ${authToken}")
        .check(status.is(200))
    )
    .pause(2.seconds)
    .exec(
      http("View Product")
        .get("/api/products/123")
        .check(status.is(200))
    )
    .pause(10.seconds)
    .exec(
      http("Add to Cart")
        .post("/api/cart")
        .body(StringBody("""{"productId":"123","quantity":1}"""))
        .header("Authorization", "Bearer ${authToken}")
        .check(status.is(201))
    )
  
  // Load Simulation
  setUp(
    scn.inject(
      rampUsers(100) during (60.seconds),
      constantUsersPerSec(50) during (300.seconds)
    )
  ).protocols(httpConf)
   .assertions(
     global.responseTime.max.lt(5000),
     global.successfulRequests.percent.gt(95)
   )
}
```

### Performance Testing with Java

```java
import org.testng.annotations.Test;
import io.restassured.RestAssured;
import io.restassured.response.Response;
import java.util.concurrent.*;
import java.util.concurrent.atomic.AtomicInteger;

public class PerformanceTest {
    
    private static final String BASE_URL = "https://api.example.com";
    private static final int THREAD_COUNT = 100;
    private static final int DURATION_SECONDS = 300;
    
    private AtomicInteger successCount = new AtomicInteger(0);
    private AtomicInteger failureCount = new AtomicInteger(0);
    private ConcurrentLinkedQueue<Long> responseTimes = new ConcurrentLinkedQueue<>();
    
    @Test
    public void performanceTest() throws InterruptedException {
        ExecutorService executor = Executors.newFixedThreadPool(THREAD_COUNT);
        
        long startTime = System.currentTimeMillis();
        long endTime = startTime + (DURATION_SECONDS * 1000);
        
        // Submit tasks
        for (int i = 0; i < THREAD_COUNT; i++) {
            executor.submit(() -> {
                while (System.currentTimeMillis() < endTime) {
                    executeRequest();
                    try {
                        Thread.sleep(1000); // Think time
                    } catch (InterruptedException e) {
                        break;
                    }
                }
            });
        }
        
        // Wait for completion
        executor.shutdown();
        executor.awaitTermination(DURATION_SECONDS + 60, TimeUnit.SECONDS);
        
        // Calculate and print metrics
        printMetrics();
    }
    
    private void executeRequest() {
        long requestStart = System.currentTimeMillis();
        
        try {
            Response response = RestAssured
                .given()
                .baseUri(BASE_URL)
                .when()
                .get("/api/products")
                .then()
                .extract()
                .response();
            
            long responseTime = System.currentTimeMillis() - requestStart;
            responseTimes.add(responseTime);
            
            if (response.getStatusCode() == 200) {
                successCount.incrementAndGet();
            } else {
                failureCount.incrementAndGet();
            }
        } catch (Exception e) {
            failureCount.incrementAndGet();
            System.err.println("Request failed: " + e.getMessage());
        }
    }
    
    private void printMetrics() {
        int totalRequests = successCount.get() + failureCount.get();
        double errorRate = (failureCount.get() * 100.0) / totalRequests;
        
        // Calculate response time statistics
        List<Long> sortedTimes = new ArrayList<>(responseTimes);
        Collections.sort(sortedTimes);
        
        long avgResponseTime = sortedTimes.stream()
            .mapToLong(Long::longValue)
            .sum() / sortedTimes.size();
        
        long minResponseTime = sortedTimes.get(0);
        long maxResponseTime = sortedTimes.get(sortedTimes.size() - 1);
        
        int p95Index = (int) (sortedTimes.size() * 0.95);
        long p95ResponseTime = sortedTimes.get(p95Index);
        
        // Print report
        System.out.println("\n=== Performance Test Results ===");
        System.out.println("Total Requests: " + totalRequests);
        System.out.println("Successful: " + successCount.get());
        System.out.println("Failed: " + failureCount.get());
        System.out.println("Error Rate: " + String.format("%.2f%%", errorRate));
        System.out.println("\nResponse Times:");
        System.out.println("  Average: " + avgResponseTime + " ms");
        System.out.println("  Min: " + minResponseTime + " ms");
        System.out.println("  Max: " + maxResponseTime + " ms");
        System.out.println("  95th Percentile: " + p95ResponseTime + " ms");
    }
}
```

---

## Bottleneck Identification

### Common Performance Bottlenecks

```markdown
## Performance Bottleneck Checklist

### 1. Application Level
- [ ] Inefficient algorithms (O(n²) complexity)
- [ ] Synchronous processing instead of async
- [ ] Missing database indexes
- [ ] N+1 query problem
- [ ] Large payload sizes
- [ ] No caching implementation
- [ ] Memory leaks

### 2. Database Level
- [ ] Missing indexes
- [ ] Slow queries (> 1 second)
- [ ] Table locks
- [ ] Connection pool exhaustion
- [ ] Inefficient joins
- [ ] Full table scans

### 3. Network Level
- [ ] High latency
- [ ] Bandwidth limitations
- [ ] DNS resolution issues
- [ ] SSL/TLS handshake overhead
- [ ] Large payload transfers

### 4. Server Level
- [ ] CPU bottleneck (> 80% usage)
- [ ] Memory bottleneck (swapping)
- [ ] Disk I/O bottleneck
- [ ] Thread pool exhaustion
- [ ] Connection limits

### 5. Frontend Level
- [ ] Large JavaScript bundles
- [ ] Unoptimized images
- [ ] Too many HTTP requests
- [ ] No CDN usage
- [ ] Render-blocking resources
```

### Performance Monitoring

```java
public class PerformanceMonitor {
    
    // Monitor response times
    public void monitorResponseTime(String endpoint) {
        long start = System.currentTimeMillis();
        
        // Execute request
        executeRequest(endpoint);
        
        long duration = System.currentTimeMillis() - start;
        
        // Log if exceeds threshold
        if (duration > 3000) {
            System.err.println("SLOW REQUEST: " + endpoint + 
                " took " + duration + "ms");
        }
    }
    
    // Monitor memory usage
    public void monitorMemory() {
        Runtime runtime = Runtime.getRuntime();
        
        long totalMemory = runtime.totalMemory();
        long freeMemory = runtime.freeMemory();
        long usedMemory = totalMemory - freeMemory;
        long maxMemory = runtime.maxMemory();
        
        double memoryUsagePercent = (usedMemory * 100.0) / maxMemory;
        
        System.out.println("Memory Usage: " + 
            String.format("%.2f%%", memoryUsagePercent));
        
        if (memoryUsagePercent > 80) {
            System.err.println("WARNING: High memory usage!");
        }
    }
    
    // Monitor thread count
    public void monitorThreads() {
        ThreadGroup rootGroup = Thread.currentThread().getThreadGroup();
        while (rootGroup.getParent() != null) {
            rootGroup = rootGroup.getParent();
        }
        
        int activeThreads = rootGroup.activeCount();
        System.out.println("Active Threads: " + activeThreads);
        
        if (activeThreads > 500) {
            System.err.println("WARNING: High thread count!");
        }
    }
    
    private void executeRequest(String endpoint) {
        // Implementation
    }
}
```

### Performance Analysis Tools

```markdown
## Performance Analysis Tools

### APM Tools (Application Performance Monitoring)
1. **New Relic**
   - Real-time monitoring
   - Transaction tracing
   - Error analytics

2. **Dynatrace**
   - AI-powered monitoring
   - Automatic root cause analysis
   - User experience monitoring

3. **AppDynamics**
   - Business transaction monitoring
   - Code-level diagnostics
   - Infrastructure monitoring

### Profiling Tools
1. **Java Profilers**
   - VisualVM
   - YourKit
   - JProfiler

2. **Database Profilers**
   - SQL Server Profiler
   - MySQL Query Analyzer
   - pgAdmin (PostgreSQL)

### Monitoring Tools
1. **Prometheus + Grafana**
   - Metrics collection
   - Alerting
   - Visualization

2. **ELK Stack (Elasticsearch, Logstash, Kibana)**
   - Log aggregation
   - Search and analysis
   - Visualization

3. **Datadog**
   - Infrastructure monitoring
   - APM
   - Log management

### Load Testing Tools
1. **JMeter** - Open source, Java-based
2. **Gatling** - Scala-based, developer-friendly
3. **K6** - Modern, JavaScript-based
4. **Locust** - Python-based, distributed
5. **Artillery** - Node.js-based
```

### Performance Test Report Template

```markdown
# Performance Test Report

## Executive Summary
- Test Duration: 2024-01-15 10:00 - 12:00 (2 hours)
- Environment: Production-like staging
- Load Profile: 500 concurrent users
- Overall Result: ✓ PASSED

## Test Objectives
- Validate system can handle 500 concurrent users
- Ensure 95th percentile response time < 3 seconds
- Verify error rate < 1%
- Identify performance bottlenecks

## Test Configuration
- **Users:** 500
- **Ramp-up Time:** 5 minutes
- **Test Duration:** 2 hours
- **Think Time:** 3-10 seconds
- **Pacing:** 60 seconds

## Results Summary

### Response Time Metrics
| Transaction | Avg (ms) | Min (ms) | Max (ms) | 95th %ile | Status |
|-------------|----------|----------|----------|-----------|--------|
| Homepage    | 850      | 200      | 2500     | 1800      | ✓ Pass |
| Search      | 1200     | 300      | 4500     | 2800      | ✓ Pass |
| Product Page| 950      | 250      | 3200     | 2100      | ✓ Pass |
| Add to Cart | 450      | 100      | 1800     | 900       | ✓ Pass |
| Checkout    | 2100     | 500      | 5200     | 3500      | ✗ Fail |

### Throughput
- Requests per Second: 125
- Transactions per Second: 25
- Total Requests: 900,000
- Successful Requests: 891,000 (99%)

### Error Analysis
- Total Errors: 9,000 (1%)
- HTTP 500 Errors: 100 (0.01%)
- Timeout Errors: 8,900 (0.99%)

### Resource Utilization
- CPU Usage: 65% (Peak: 78%)
- Memory Usage: 72% (Peak: 85%)
- Disk I/O: 45%
- Network: 42%

## Identified Issues
1. **Checkout Page Performance**
   - Exceeds 95th percentile target
   - Root Cause: Database query optimization needed
   - Recommendation: Add index on orders table

2. **Timeout Errors**
   - 0.99% timeout rate
   - Root Cause: Payment gateway slow response
   - Recommendation: Increase timeout, implement circuit breaker

## Recommendations
1. Optimize checkout page database queries
2. Implement caching for product catalog
3. Add circuit breaker for payment gateway
4. Scale application servers from 4 to 6

## Conclusion
System can handle target load with minor optimizations needed for checkout process. Recommended to address identified issues before production deployment.
```

---

**Next:** [Agile & SDLC](10-agile-sdlc.md)

