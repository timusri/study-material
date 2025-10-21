# 23. Test Observability & Monitoring

## 📚 Quick Summary

Can't improve what you don't measure - observability shows what's really happening!

**What You'll Learn:**
- **Test Metrics**: Pass rate, execution time, flakiness
- **Real-time Monitoring**: Watch tests run live
- **Dashboards**: Visualize test health (Grafana, Allure)
- **Alerting**: Get notified of failures instantly
- **Analytics**: Trends, patterns, insights
- **Log Management**: Debug failures faster

**Why This Matters:**
- **Visibility**: Know test health at a glance
- **Fast Debugging**: Logs + screenshots = quick fixes
- **Trends**: Spot deteriorating quality early
- **Stakeholder Communication**: Show value with metrics
- **Continuous Improvement**: Data-driven decisions

**Senior Skill:**
Setting up observability = Lead/Architect level work!

---

## 📖 Simple Explanation

**What is Test Observability?**
Seeing what's happening in your test automation - past, present, and trends.

**Analogy:** Car Dashboard
- **Speed**: How fast tests run
- **Fuel**: Test coverage
- **Check Engine**: Failures, flaky tests
- **Mileage**: Total tests run over time

**3 Pillars of Observability:**

**1. Metrics (Numbers)**
```
- Pass Rate: 95% (goal: >95%)
- Execution Time: 15 minutes (goal: <20 min)
- Flaky Rate: 2% (goal: <1%)
- Code Coverage: 80%
```

**2. Logs (Details)**
```
Test failed at 10:05 AM
Reason: Element not found
Screenshot: login_page_failure.png
Stack trace: ...
```

**3. Traces (Flow)**
```
Test Login → Navigate → Enter username → 
Enter password → Click → Wait for dashboard → ✅
(Which step took longest? Where did it fail?)
```

**Real Example:**
```
Without Observability:
- "Tests are failing!" (Which ones? How many?)
- "Tests are slow!" (How slow? Getting worse?)
- "Too many bugs!" (Compared to what?)

With Observability:
- Dashboard shows: 5 tests failing (out of 500)
- Trend: Pass rate dropped from 98% to 95% this week
- Alert: Execution time increased 20% (investigate!)
- Log shows: All 5 failures are login page (fix one thing!)
```

**Tools:**
- **Allure**: Beautiful test reports
- **ReportPortal**: AI-powered test analytics
- **Grafana**: Real-time dashboards
- **ELK Stack**: Log management

---

## Table of Contents
- [What is Test Observability](#what-is-test-observability)
- [Test Metrics & KPIs](#test-metrics--kpis)
- [Test Reporting](#test-reporting)
- [Real-time Monitoring](#real-time-monitoring)
- [Log Management](#log-management)
- [Dashboard Creation](#dashboard-creation)
- [Alerting & Notifications](#alerting--notifications)

---

## What is Test Observability

### Understanding Test Observability

```markdown
## Definition

**Test Observability:** The ability to understand the internal state of your test system by examining its outputs (logs, metrics, traces).

## Traditional Testing vs Observable Testing

**Traditional:**
❌ Test passed or failed
❌ Check logs after failure
❌ Manual investigation
❌ Reactive approach

**Observable:**
✅ **Why** did test pass/fail
✅ Real-time insights
✅ Proactive monitoring
✅ Historical trends
✅ Root cause analysis

## Three Pillars of Observability

### 1. Metrics
**What:** Numerical measurements over time

**Examples:**
- Test execution time
- Pass/fail rate
- Flakiness rate
- Code coverage
- API response times

### 2. Logs
**What:** Detailed event records

**Examples:**
- Test execution logs
- Application logs
- Browser console logs
- Network logs
- Screenshots

### 3. Traces
**What:** Request flow through system

**Examples:**
- Test execution flow
- API call chains
- User journey tracking
- Performance bottlenecks
```

---

## Test Metrics & KPIs

### Essential Test Metrics

```java
public class TestMetrics {
    
    /**
     * 1. Test Execution Metrics
     */
    public class ExecutionMetrics {
        int totalTests;
        int passedTests;
        int failedTests;
        int skippedTests;
        long totalExecutionTime;
        long averageTestTime;
        
        public double getPassRate() {
            return (passedTests * 100.0) / totalTests;
        }
        
        public double getFailRate() {
            return (failedTests * 100.0) / totalTests;
        }
    }
    
    /**
     * 2. Flakiness Metrics
     */
    public class FlakinessMetrics {
        Map<String, FlakyTestInfo> flakyTests;
        
        public double getFlakinessRate(String testName) {
            FlakyTestInfo info = flakyTests.get(testName);
            return (info.failures * 100.0) / info.totalRuns;
        }
        
        public List<String> getTop10FlakyTests() {
            return flakyTests.entrySet().stream()
                .sorted((a, b) -> Double.compare(
                    b.getValue().flakinessRate, 
                    a.getValue().flakinessRate
                ))
                .limit(10)
                .map(Map.Entry::getKey)
                .collect(Collectors.toList());
        }
    }
    
    /**
     * 3. Performance Metrics
     */
    public class PerformanceMetrics {
        Map<String, List<Long>> testExecutionTimes;
        
        public long getAverageTime(String testName) {
            List<Long> times = testExecutionTimes.get(testName);
            return times.stream()
                .mapToLong(Long::longValue)
                .sum() / times.size();
        }
        
        public long getP95Time(String testName) {
            List<Long> times = testExecutionTimes.get(testName);
            Collections.sort(times);
            int index = (int) Math.ceil(0.95 * times.size()) - 1;
            return times.get(index);
        }
        
        public List<String> getSlowestTests() {
            return testExecutionTimes.entrySet().stream()
                .sorted((a, b) -> Long.compare(
                    getAverageTime(b.getKey()),
                    getAverageTime(a.getKey())
                ))
                .limit(10)
                .map(Map.Entry::getKey)
                .collect(Collectors.toList());
        }
    }
    
    /**
     * 4. Coverage Metrics
     */
    public class CoverageMetrics {
        int totalRequirements;
        int coveredRequirements;
        int totalCodeLines;
        int coveredCodeLines;
        
        public double getRequirementCoverage() {
            return (coveredRequirements * 100.0) / totalRequirements;
        }
        
        public double getCodeCoverage() {
            return (coveredCodeLines * 100.0) / totalCodeLines;
        }
    }
    
    /**
     * 5. Defect Metrics
     */
    public class DefectMetrics {
        int defectsFound;
        int defectsPrevented;
        int defectsEscaped; // Found in production
        
        public double getDefectDetectionRate() {
            return (defectsFound * 100.0) / (defectsFound + defectsEscaped);
        }
        
        public double getDefectLeakageRate() {
            return (defectsEscaped * 100.0) / (defectsFound + defectsEscaped);
        }
    }
}
```

### Collecting Metrics

```java
import org.testng.ITestResult;
import org.testng.ITestListener;

public class MetricsCollector implements ITestListener {
    
    private Map<String, TestMetric> metrics = new ConcurrentHashMap<>();
    private MetricsDatabase metricsDB;
    
    @Override
    public void onTestStart(ITestResult result) {
        String testName = result.getName();
        TestMetric metric = new TestMetric();
        metric.testName = testName;
        metric.startTime = System.currentTimeMillis();
        metric.executionDate = LocalDateTime.now();
        metrics.put(testName, metric);
    }
    
    @Override
    public void onTestSuccess(ITestResult result) {
        String testName = result.getName();
        TestMetric metric = metrics.get(testName);
        metric.endTime = System.currentTimeMillis();
        metric.executionTime = metric.endTime - metric.startTime;
        metric.status = "PASSED";
        metric.retryCount = result.getMethod().getCurrentInvocationCount() - 1;
        
        // Save to database
        metricsDB.save(metric);
        
        // Send to monitoring system
        sendToPrometheus(metric);
    }
    
    @Override
    public void onTestFailure(ITestResult result) {
        String testName = result.getName();
        TestMetric metric = metrics.get(testName);
        metric.endTime = System.currentTimeMillis();
        metric.executionTime = metric.endTime - metric.startTime;
        metric.status = "FAILED";
        metric.failureReason = result.getThrowable().getMessage();
        metric.stackTrace = getStackTrace(result.getThrowable());
        metric.screenshot = captureScreenshot();
        
        metricsDB.save(metric);
        sendToPrometheus(metric);
        
        // Send alert for critical test failures
        if (isCriticalTest(testName)) {
            sendAlert(metric);
        }
    }
    
    @Override
    public void onFinish(ITestContext context) {
        // Generate summary metrics
        SummaryMetrics summary = new SummaryMetrics();
        summary.totalTests = context.getAllTestMethods().length;
        summary.passed = context.getPassedTests().size();
        summary.failed = context.getFailedTests().size();
        summary.skipped = context.getSkippedTests().size();
        summary.passRate = (summary.passed * 100.0) / summary.totalTests;
        summary.totalTime = context.getEndDate().getTime() - 
                           context.getStartDate().getTime();
        
        // Send summary
        metricsDB.saveSummary(summary);
        sendToPrometheus(summary);
        
        // Generate report
        generateDashboard(summary);
    }
    
    private void sendToPrometheus(TestMetric metric) {
        // Prometheus metrics
        Counter.builder("test_execution_total")
            .description("Total test executions")
            .tag("test", metric.testName)
            .tag("status", metric.status)
            .register(registry)
            .increment();
        
        Gauge.builder("test_execution_time_seconds", () -> metric.executionTime / 1000.0)
            .description("Test execution time")
            .tag("test", metric.testName)
            .register(registry);
        
        if (metric.retryCount > 0) {
            Counter.builder("test_retry_total")
                .description("Test retries (flaky tests)")
                .tag("test", metric.testName)
                .register(registry)
                .increment(metric.retryCount);
        }
    }
}
```

---

## Test Reporting

### Advanced Test Reports

```java
import io.qameta.allure.Allure;
import io.qameta.allure.Step;
import io.qameta.allure.Attachment;

public class AllureReporting {
    
    @Test
    @Description("Test user login with valid credentials")
    @Severity(SeverityLevel.CRITICAL)
    @Story("User Authentication")
    @Feature("Login")
    public void testLogin() {
        navigateToLoginPage();
        enterCredentials("testuser", "password123");
        clickLoginButton();
        verifyDashboardDisplayed();
    }
    
    @Step("Navigate to login page")
    public void navigateToLoginPage() {
        driver.get("https://example.com/login");
        Allure.addAttachment("Page URL", driver.getCurrentUrl());
    }
    
    @Step("Enter credentials: username={0}")
    public void enterCredentials(String username, String password) {
        driver.findElement(By.id("username")).sendKeys(username);
        driver.findElement(By.id("password")).sendKeys(password);
        
        // Attach screenshot
        attachScreenshot("Credentials entered");
    }
    
    @Step("Click login button")
    public void clickLoginButton() {
        driver.findElement(By.id("login")).click();
        
        // Attach network logs
        attachNetworkLogs();
    }
    
    @Step("Verify dashboard is displayed")
    public void verifyDashboardDisplayed() {
        WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
        WebElement dashboard = wait.until(
            ExpectedConditions.visibilityOfElementLocated(By.id("dashboard"))
        );
        
        Assert.assertTrue(dashboard.isDisplayed());
        
        attachScreenshot("Dashboard displayed");
    }
    
    @Attachment(value = "Screenshot: {0}", type = "image/png")
    public byte[] attachScreenshot(String name) {
        return ((TakesScreenshot) driver).getScreenshotAs(OutputType.BYTES);
    }
    
    @Attachment(value = "Network Logs", type = "application/json")
    public String attachNetworkLogs() {
        LogEntries logs = driver.manage().logs().get(LogType.PERFORMANCE);
        List<LogEntry> entries = logs.getAll();
        
        return new Gson().toJson(entries);
    }
    
    @Attachment(value = "Console Logs", type = "text/plain")
    public String attachConsoleLogs() {
        LogEntries logs = driver.manage().logs().get(LogType.BROWSER);
        StringBuilder sb = new StringBuilder();
        
        for (LogEntry entry : logs) {
            sb.append(entry.getLevel())
              .append(" ")
              .append(entry.getMessage())
              .append("\n");
        }
        
        return sb.toString();
    }
}
```

### Custom HTML Reports

```java
public class CustomReportGenerator {
    
    public void generateHTMLReport(List<TestResult> results) {
        StringBuilder html = new StringBuilder();
        
        html.append("<!DOCTYPE html>\n");
        html.append("<html>\n<head>\n");
        html.append("<title>Test Execution Report</title>\n");
        html.append("<style>\n");
        html.append("body { font-family: Arial, sans-serif; margin: 20px; }\n");
        html.append(".summary { background: #f0f0f0; padding: 20px; margin-bottom: 20px; }\n");
        html.append(".passed { color: green; }\n");
        html.append(".failed { color: red; }\n");
        html.append(".chart { width: 400px; height: 300px; }\n");
        html.append("table { border-collapse: collapse; width: 100%; }\n");
        html.append("th, td { border: 1px solid #ddd; padding: 8px; text-align: left; }\n");
        html.append("th { background-color: #4CAF50; color: white; }\n");
        html.append("</style>\n");
        html.append("<script src='https://cdn.jsdelivr.net/npm/chart.js'></script>\n");
        html.append("</head>\n<body>\n");
        
        // Summary section
        html.append("<div class='summary'>\n");
        html.append("<h1>Test Execution Summary</h1>\n");
        
        int total = results.size();
        int passed = (int) results.stream().filter(r -> r.status.equals("PASSED")).count();
        int failed = (int) results.stream().filter(r -> r.status.equals("FAILED")).count();
        double passRate = (passed * 100.0) / total;
        
        html.append("<p>Total Tests: ").append(total).append("</p>\n");
        html.append("<p class='passed'>Passed: ").append(passed).append("</p>\n");
        html.append("<p class='failed'>Failed: ").append(failed).append("</p>\n");
        html.append("<p>Pass Rate: ").append(String.format("%.2f%%", passRate)).append("</p>\n");
        html.append("</div>\n");
        
        // Chart
        html.append("<canvas id='myChart' class='chart'></canvas>\n");
        html.append("<script>\n");
        html.append("const ctx = document.getElementById('myChart').getContext('2d');\n");
        html.append("new Chart(ctx, {\n");
        html.append("  type: 'pie',\n");
        html.append("  data: {\n");
        html.append("    labels: ['Passed', 'Failed'],\n");
        html.append("    datasets: [{\n");
        html.append("      data: [").append(passed).append(", ").append(failed).append("],\n");
        html.append("      backgroundColor: ['#4CAF50', '#f44336']\n");
        html.append("    }]\n");
        html.append("  }\n");
        html.append("});\n");
        html.append("</script>\n");
        
        // Test results table
        html.append("<h2>Test Results</h2>\n");
        html.append("<table>\n");
        html.append("<tr><th>Test Name</th><th>Status</th><th>Duration</th><th>Failure Reason</th></tr>\n");
        
        for (TestResult result : results) {
            html.append("<tr>\n");
            html.append("<td>").append(result.testName).append("</td>\n");
            html.append("<td class='").append(result.status.toLowerCase()).append("'>")
                .append(result.status).append("</td>\n");
            html.append("<td>").append(result.executionTime).append("ms</td>\n");
            html.append("<td>").append(result.failureReason != null ? result.failureReason : "")
                .append("</td>\n");
            html.append("</tr>\n");
        }
        
        html.append("</table>\n");
        html.append("</body>\n</html>");
        
        // Save to file
        try {
            Files.write(Paths.get("test-report.html"), html.toString().getBytes());
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

---

## Real-time Monitoring

### Prometheus Integration

```java
import io.prometheus.client.*;
import io.prometheus.client.exporter.HTTPServer;

public class TestMetricsExporter {
    
    // Define metrics
    private static final Counter testExecutions = Counter.build()
        .name("test_executions_total")
        .help("Total number of test executions")
        .labelNames("test_name", "status")
        .register();
    
    private static final Histogram testDuration = Histogram.build()
        .name("test_duration_seconds")
        .help("Test execution duration in seconds")
        .labelNames("test_name")
        .buckets(0.1, 0.5, 1, 2, 5, 10, 30, 60)
        .register();
    
    private static final Gauge activeTests = Gauge.build()
        .name("tests_active")
        .help("Number of currently running tests")
        .register();
    
    private static final Counter flakyTests = Counter.build()
        .name("flaky_tests_total")
        .help("Number of test retries due to flakiness")
        .labelNames("test_name")
        .register();
    
    private static final Summary responseTime = Summary.build()
        .name("api_response_time_seconds")
        .help("API response time in seconds")
        .labelNames("endpoint", "method")
        .quantile(0.5, 0.05)   // Median
        .quantile(0.95, 0.01)  // 95th percentile
        .quantile(0.99, 0.001) // 99th percentile
        .register();
    
    public static void main(String[] args) throws Exception {
        // Start Prometheus HTTP server
        HTTPServer server = new HTTPServer(8081);
        System.out.println("Metrics available at http://localhost:8081/metrics");
    }
    
    public void recordTestExecution(String testName, String status, long durationMs) {
        testExecutions.labels(testName, status).inc();
        testDuration.labels(testName).observe(durationMs / 1000.0);
    }
    
    public void recordAPICall(String endpoint, String method, long responseTimeMs) {
        responseTime.labels(endpoint, method).observe(responseTimeMs / 1000.0);
    }
    
    public void recordFlaky(String testName) {
        flakyTests.labels(testName).inc();
    }
    
    // Use in test listener
    @Override
    public void onTestStart(ITestResult result) {
        activeTests.inc();
    }
    
    @Override
    public void onTestSuccess(ITestResult result) {
        activeTests.dec();
        long duration = result.getEndMillis() - result.getStartMillis();
        recordTestExecution(result.getName(), "PASSED", duration);
    }
    
    @Override
    public void onTestFailure(ITestResult result) {
        activeTests.dec();
        long duration = result.getEndMillis() - result.getStartMillis();
        recordTestExecution(result.getName(), "FAILED", duration);
        
        // Check if retried (flaky)
        if (result.getMethod().getCurrentInvocationCount() > 1) {
            recordFlaky(result.getName());
        }
    }
}
```

### Grafana Dashboard Configuration

```json
{
  "dashboard": {
    "title": "Test Automation Dashboard",
    "panels": [
      {
        "title": "Test Pass Rate",
        "type": "gauge",
        "targets": [
          {
            "expr": "(sum(rate(test_executions_total{status=\"PASSED\"}[5m])) / sum(rate(test_executions_total[5m]))) * 100"
          }
        ]
      },
      {
        "title": "Test Execution Rate",
        "type": "graph",
        "targets": [
          {
            "expr": "sum(rate(test_executions_total[5m])) by (status)",
            "legendFormat": "{{status}}"
          }
        ]
      },
      {
        "title": "Test Duration (p95)",
        "type": "graph",
        "targets": [
          {
            "expr": "histogram_quantile(0.95, sum(rate(test_duration_seconds_bucket[5m])) by (le, test_name))",
            "legendFormat": "{{test_name}}"
          }
        ]
      },
      {
        "title": "Flaky Tests",
        "type": "table",
        "targets": [
          {
            "expr": "topk(10, sum(increase(flaky_tests_total[24h])) by (test_name))"
          }
        ]
      },
      {
        "title": "Active Tests",
        "type": "stat",
        "targets": [
          {
            "expr": "tests_active"
          }
        ]
      },
      {
        "title": "API Response Time (p95)",
        "type": "graph",
        "targets": [
          {
            "expr": "api_response_time_seconds{quantile=\"0.95\"}",
            "legendFormat": "{{endpoint}}"
          }
        ]
      }
    ]
  }
}
```

---

## Log Management

### Structured Logging

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import net.logstash.logback.argument.StructuredArguments;

public class StructuredLogging {
    private static final Logger logger = LoggerFactory.getLogger(StructuredLogging.class);
    
    @Test
    public void testWithStructuredLogs() {
        String testName = "testLogin";
        String testId = UUID.randomUUID().toString();
        
        // Structured log with context
        logger.info("Test started",
            StructuredArguments.keyValue("test_name", testName),
            StructuredArguments.keyValue("test_id", testId),
            StructuredArguments.keyValue("environment", "QA"),
            StructuredArguments.keyValue("browser", "chrome")
        );
        
        try {
            // Test execution
            driver.get("https://example.com/login");
            
            logger.info("Page loaded",
                StructuredArguments.keyValue("test_id", testId),
                StructuredArguments.keyValue("url", driver.getCurrentUrl()),
                StructuredArguments.keyValue("load_time_ms", getPageLoadTime())
            );
            
            driver.findElement(By.id("username")).sendKeys("testuser");
            driver.findElement(By.id("password")).sendKeys("password123");
            
            logger.debug("Credentials entered",
                StructuredArguments.keyValue("test_id", testId)
            );
            
            driver.findElement(By.id("login")).click();
            
            logger.info("Login button clicked",
                StructuredArguments.keyValue("test_id", testId)
            );
            
            // Verify
            Assert.assertTrue(driver.getCurrentUrl().contains("dashboard"));
            
            logger.info("Test passed",
                StructuredArguments.keyValue("test_name", testName),
                StructuredArguments.keyValue("test_id", testId),
                StructuredArguments.keyValue("status", "PASSED"),
                StructuredArguments.keyValue("duration_ms", getTestDuration())
            );
            
        } catch (Exception e) {
            logger.error("Test failed",
                StructuredArguments.keyValue("test_name", testName),
                StructuredArguments.keyValue("test_id", testId),
                StructuredArguments.keyValue("status", "FAILED"),
                StructuredArguments.keyValue("error_message", e.getMessage()),
                StructuredArguments.keyValue("error_type", e.getClass().getSimpleName()),
                e
            );
            throw e;
        }
    }
}
```

### ELK Stack Integration

```yaml
# logback.xml
<configuration>
    <appender name="LOGSTASH" class="net.logstash.logback.appender.LogstashTcpSocketAppender">
        <destination>localhost:5000</destination>
        
        <encoder class="net.logstash.logback.encoder.LogstashEncoder">
            <customFields>{"app":"test-automation","environment":"qa"}</customFields>
        </encoder>
    </appender>
    
    <root level="INFO">
        <appender-ref ref="LOGSTASH" />
    </root>
</configuration>
```

```json
// Elasticsearch query
{
  "query": {
    "bool": {
      "must": [
        { "match": { "test_name": "testLogin" }},
        { "match": { "status": "FAILED" }},
        { "range": { "@timestamp": { "gte": "now-24h" }}}
      ]
    }
  },
  "aggs": {
    "failure_reasons": {
      "terms": { "field": "error_message.keyword" }
    }
  }
}
```

---

## Dashboard Creation

### Real-time Test Dashboard

```java
public class RealTimeDashboard {
    
    // WebSocket server for real-time updates
    @ServerEndpoint("/test-updates")
    public class TestUpdateEndpoint {
        
        private static Set<Session> sessions = new ConcurrentHashSet<>();
        
        @OnOpen
        public void onOpen(Session session) {
            sessions.add(session);
            sendInitialData(session);
        }
        
        @OnClose
        public void onClose(Session session) {
            sessions.remove(session);
        }
        
        public static void broadcastUpdate(TestUpdate update) {
            String json = new Gson().toJson(update);
            for (Session session : sessions) {
                try {
                    session.getBasicRemote().sendText(json);
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
    
    // Test listener sends updates
    @Override
    public void onTestStart(ITestResult result) {
        TestUpdate update = new TestUpdate();
        update.type = "TEST_STARTED";
        update.testName = result.getName();
        update.timestamp = System.currentTimeMillis();
        
        TestUpdateEndpoint.broadcastUpdate(update);
    }
    
    @Override
    public void onTestSuccess(ITestResult result) {
        TestUpdate update = new TestUpdate();
        update.type = "TEST_PASSED";
        update.testName = result.getName();
        update.duration = result.getEndMillis() - result.getStartMillis();
        update.timestamp = System.currentTimeMillis();
        
        TestUpdateEndpoint.broadcastUpdate(update);
    }
}
```

```html
<!-- dashboard.html -->
<!DOCTYPE html>
<html>
<head>
    <title>Real-time Test Dashboard</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        body { font-family: Arial; margin: 20px; }
        .stats { display: flex; gap: 20px; margin-bottom: 20px; }
        .stat-box { background: #f0f0f0; padding: 20px; border-radius: 5px; flex: 1; }
        .passed { color: green; }
        .failed { color: red; }
        .running { color: orange; }
        #test-list { border: 1px solid #ddd; padding: 10px; max-height: 400px; overflow-y: auto; }
        .test-item { padding: 10px; margin: 5px 0; border-left: 4px solid #ddd; }
    </style>
</head>
<body>
    <h1>Test Execution Dashboard</h1>
    
    <div class="stats">
        <div class="stat-box">
            <h3>Total Tests</h3>
            <div id="total" style="font-size: 2em;">0</div>
        </div>
        <div class="stat-box passed">
            <h3>Passed</h3>
            <div id="passed" style="font-size: 2em;">0</div>
        </div>
        <div class="stat-box failed">
            <h3>Failed</h3>
            <div id="failed" style="font-size: 2em;">0</div>
        </div>
        <div class="stat-box running">
            <h3>Running</h3>
            <div id="running" style="font-size: 2em;">0</div>
        </div>
    </div>
    
    <canvas id="chart" width="400" height="200"></canvas>
    
    <h2>Recent Tests</h2>
    <div id="test-list"></div>
    
    <script>
        const ws = new WebSocket('ws://localhost:8080/test-updates');
        
        let stats = { total: 0, passed: 0, failed: 0, running: 0 };
        
        const chart = new Chart(document.getElementById('chart'), {
            type: 'line',
            data: {
                labels: [],
                datasets: [{
                    label: 'Pass Rate %',
                    data: [],
                    borderColor: 'green',
                    fill: false
                }]
            },
            options: {
                scales: { y: { beginAtZero: true, max: 100 }}
            }
        });
        
        ws.onmessage = function(event) {
            const update = JSON.parse(event.data);
            
            if (update.type === 'TEST_STARTED') {
                stats.running++;
                addTestItem(update.testName, 'RUNNING');
            } else if (update.type === 'TEST_PASSED') {
                stats.passed++;
                stats.running--;
                updateTestItem(update.testName, 'PASSED', update.duration);
            } else if (update.type === 'TEST_FAILED') {
                stats.failed++;
                stats.running--;
                updateTestItem(update.testName, 'FAILED', update.duration);
            }
            
            stats.total = stats.passed + stats.failed;
            updateStats();
            updateChart();
        };
        
        function updateStats() {
            document.getElementById('total').textContent = stats.total;
            document.getElementById('passed').textContent = stats.passed;
            document.getElementById('failed').textContent = stats.failed;
            document.getElementById('running').textContent = stats.running;
        }
        
        function updateChart() {
            const passRate = stats.total > 0 ? (stats.passed / stats.total * 100).toFixed(2) : 0;
            const time = new Date().toLocaleTimeString();
            
            chart.data.labels.push(time);
            chart.data.datasets[0].data.push(passRate);
            
            // Keep only last 20 points
            if (chart.data.labels.length > 20) {
                chart.data.labels.shift();
                chart.data.datasets[0].data.shift();
            }
            
            chart.update();
        }
        
        function addTestItem(testName, status) {
            const item = document.createElement('div');
            item.className = 'test-item';
            item.id = 'test-' + testName;
            item.innerHTML = `<strong>${testName}</strong> - <span class="${status.toLowerCase()}">${status}</span>`;
            document.getElementById('test-list').insertBefore(item, document.getElementById('test-list').firstChild);
        }
        
        function updateTestItem(testName, status, duration) {
            const item = document.getElementById('test-' + testName);
            if (item) {
                item.innerHTML = `<strong>${testName}</strong> - <span class="${status.toLowerCase()}">${status}</span> (${duration}ms)`;
            }
        }
    </script>
</body>
</html>
```

---

## Alerting & Notifications

### Smart Alerting

```java
public class SmartAlerting {
    
    private SlackClient slack;
    private EmailClient email;
    
    @Override
    public void onFinish(ITestContext context) {
        TestSummary summary = generateSummary(context);
        
        // Alert conditions
        if (shouldSendAlert(summary)) {
            sendAlert(summary);
        }
    }
    
    private boolean shouldSendAlert(TestSummary summary) {
        // Alert if pass rate < 90%
        if (summary.passRate < 90.0) {
            return true;
        }
        
        // Alert if any critical test failed
        if (summary.criticalTestsFailed > 0) {
            return true;
        }
        
        // Alert if flaky tests increased
        if (summary.flakyTestCount > summary.previousFlakyTestCount * 1.5) {
            return true;
        }
        
        // Alert if execution time increased significantly
        if (summary.totalTime > summary.averageTime * 1.5) {
            return true;
        }
        
        return false;
    }
    
    private void sendAlert(TestSummary summary) {
        // Slack notification
        SlackMessage slackMsg = new SlackMessage();
        slackMsg.setChannel("#test-alerts");
        slackMsg.setUsername("Test Bot");
        slackMsg.setIcon(":robot_face:");
        
        if (summary.passRate < 80) {
            slackMsg.setColor("danger");
        } else if (summary.passRate < 90) {
            slackMsg.setColor("warning");
        } else {
            slackMsg.setColor("good");
        }
        
        slackMsg.setText(String.format(
            "Test Execution Alert\n" +
            "Pass Rate: %.2f%%\n" +
            "Passed: %d, Failed: %d\n" +
            "Duration: %d minutes\n" +
            "Build: %s\n" +
            "Report: %s",
            summary.passRate,
            summary.passed,
            summary.failed,
            summary.totalTime / 60000,
            System.getenv("BUILD_URL"),
            getReportUrl()
        ));
        
        slack.send(slackMsg);
        
        // Email for critical failures
        if (summary.criticalTestsFailed > 0) {
            EmailMessage email = new EmailMessage();
            email.setTo("qa-team@example.com");
            email.setSubject("CRITICAL: Test Failures Detected");
            email.setBody(generateEmailBody(summary));
            email.setAttachment(getReportFile());
            
            this.email.send(email);
        }
    }
}
```

---

## Interview Questions

### Q1: What metrics would you track for test automation?

**Answer:**
```markdown
**Essential Metrics:**

1. **Execution Metrics:**
   - Pass rate (target: >95%)
   - Execution time (track trends)
   - Flakiness rate (target: <5%)

2. **Quality Metrics:**
   - Defect detection rate
   - Defect leakage (bugs in prod)
   - Code coverage

3. **Efficiency Metrics:**
   - Time to write test vs manual testing
   - Maintenance time
   - ROI

4. **Health Metrics:**
   - Slowest tests
   - Most flaky tests
   - Test age

**Don't Track Vanity Metrics:**
❌ Number of tests (more ≠ better)
❌ Lines of code
❌ Pass/fail only (need context)

**Example Dashboard:**
- Current pass rate: 96%
- Trend: ↑ 2% from last week
- Flaky tests: 3 (needs attention)
- Avg execution: 15 min (target: <20 min)
```

---

**Next:** [Emerging Technologies in Testing](24-emerging-technologies.md)

