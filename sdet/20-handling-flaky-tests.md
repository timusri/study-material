# 20. Handling Flaky Tests

## 📚 Quick Summary

Flaky tests are the #1 pain point for automation engineers - master fixing them!

**What You'll Learn:**
- **What are Flaky Tests**: Tests that pass/fail randomly
- **Root Causes**: Timing, data, environment issues
- **Detection**: Identify flaky tests early
- **Fixing**: Proven strategies to eliminate flakiness
- **Prevention**: Write stable tests from the start
- **Retry**: When and how to use retries

**Why This Matters:**
- **Trust**: Flaky tests destroy confidence in automation
- **Time**: Teams waste hours debugging flaky tests
- **Coverage**: Teams reduce automation due to flakiness
- **Senior Skill**: Debugging flakiness separates junior from senior
- **Interview Favorite**: "How do you handle flaky tests?"

**Reality:**
Google: "If you have 1% flaky tests, with 1000 tests, you have 10 failures per run - unusable!"

---

## 📖 Simple Explanation

**What is a Flaky Test?**
A test that sometimes passes, sometimes fails - without any code changes!

**Example:**
```
Run 1: ✅ Pass
Run 2: ✅ Pass  
Run 3: ❌ Fail (Same code, nothing changed!)
Run 4: ✅ Pass
Run 5: ❌ Fail

= Flaky Test (unreliable)
```

**Why It's a Problem:**
```
Scenario: You have 100 tests, 10 are flaky (10% flaky rate)

Every run:
- 5-10 tests fail randomly
- Developers ignore failures ("probably flaky")
- Real bugs slip through
- Team loses trust in automation
- Eventually: "Let's just test manually"
```

**Top 5 Causes of Flakiness:**

**1. Timing Issues (80% of flakiness)**
```java
❌ Thread.sleep(5000); // Bad: Waits 5 sec even if ready in 1 sec

✅ WebDriverWait wait = new WebDriverWait(driver, 10);
   wait.until(ExpectedConditions.elementToBeClickable(button)); // Good: Waits only as needed
```

**2. Test Data Issues**
```java
❌ Test depends on specific data that doesn't exist
❌ Tests interfere with each other (shared data)

✅ Generate fresh test data for each test
✅ Clean up data after test
```

**3. Stale Elements**
```java
❌ WebElement button = driver.findElement(By.id("btn"));
   // Page refreshes
   button.click(); // Stale!

✅ driver.findElement(By.id("btn")).click(); // Find fresh element
```

**4. Network Issues**
```java
❌ API call fails due to timeout

✅ Add retries for API calls
✅ Use proper timeouts
```

**5. Parallel Execution**
```java
❌ Tests share same WebDriver instance
❌ Tests use same test data

✅ ThreadLocal<WebDriver> for each thread
✅ Isolated test data per test
```

---

## Table of Contents
- [Understanding Flaky Tests](#understanding-flaky-tests)
- [Root Causes of Flakiness](#root-causes-of-flakiness)
- [Detection Strategies](#detection-strategies)
- [Fixing Common Flaky Patterns](#fixing-common-flaky-patterns)
- [Retry Mechanisms](#retry-mechanisms)
- [Preventing Flakiness](#preventing-flakiness)
- [Organizational Approach](#organizational-approach)

---

## Understanding Flaky Tests

### What are Flaky Tests?

```markdown
## Definition
**Flaky Test:** A test that produces inconsistent results without code changes.

Sometimes passes ✅
Sometimes fails ❌

Same code, same environment, different results!

## Impact

**Development:**
- Lost productivity (investigating false failures)
- Reduced confidence in test suite
- Increased CI/CD time
- Developer frustration

**Business:**
- Delayed releases
- Missed bugs (ignored failures)
- Increased costs
- Lower quality

## Statistics
- Google: 16% of their tests are flaky
- Microsoft: 14% flakiness rate
- Average cost: 30-40% of developer time wasted

## Example
```java
@Test
public void testUserLogin() {
    driver.get("https://example.com/login");
    driver.findElement(By.id("username")).sendKeys("user");
    driver.findElement(By.id("password")).sendKeys("pass123");
    driver.findElement(By.id("submit")).click();
    
    // ❌ Flaky: Sometimes passes, sometimes fails
    // Depending on page load speed
    Assert.assertTrue(driver.findElement(By.id("welcome")).isDisplayed());
}
```
```

---

## Root Causes of Flakiness

### 1. Timing Issues (Most Common - 70%)

```java
public class TimingIssues {
    
    // ❌ BAD: No wait - immediate check
    @Test
    public void flakyNoWait() {
        driver.get("https://example.com");
        driver.findElement(By.id("button")).click();
        
        // Element might not be loaded yet
        WebElement result = driver.findElement(By.id("result"));
        Assert.assertTrue(result.isDisplayed());
    }
    
    // ❌ BAD: Fixed wait (arbitrary delay)
    @Test
    public void flakyFixedWait() throws InterruptedException {
        driver.get("https://example.com");
        driver.findElement(By.id("button")).click();
        
        Thread.sleep(3000); // Sometimes enough, sometimes not
        
        WebElement result = driver.findElement(By.id("result"));
        Assert.assertTrue(result.isDisplayed());
    }
    
    // ✅ GOOD: Explicit wait
    @Test
    public void stableExplicitWait() {
        driver.get("https://example.com");
        driver.findElement(By.id("button")).click();
        
        WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
        WebElement result = wait.until(
            ExpectedConditions.visibilityOfElementLocated(By.id("result"))
        );
        Assert.assertTrue(result.isDisplayed());
    }
    
    // ✅ BETTER: Custom wait condition
    @Test
    public void stableCustomWait() {
        driver.get("https://example.com");
        driver.findElement(By.id("button")).click();
        
        WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
        wait.until(new ExpectedCondition<Boolean>() {
            public Boolean apply(WebDriver driver) {
                WebElement result = driver.findElement(By.id("result"));
                return result.isDisplayed() && 
                       !result.getText().isEmpty() &&
                       result.getCssValue("opacity").equals("1");
            }
        });
    }
}
```

### 2. Race Conditions

```java
public class RaceConditions {
    
    // ❌ FLAKY: Animation not complete
    @Test
    public void flakyAnimation() {
        driver.findElement(By.id("menu-button")).click();
        
        // Menu is animating (sliding in)
        driver.findElement(By.id("menu-item-1")).click(); // Might fail mid-animation
    }
    
    // ✅ STABLE: Wait for animation to complete
    @Test
    public void stableAnimation() {
        driver.findElement(By.id("menu-button")).click();
        
        WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(5));
        
        // Wait for element to be clickable (animation complete)
        WebElement menuItem = wait.until(
            ExpectedConditions.elementToBeClickable(By.id("menu-item-1"))
        );
        menuItem.click();
    }
    
    // ❌ FLAKY: AJAX request not complete
    @Test
    public void flakyAjax() {
        driver.findElement(By.id("load-data")).click();
        
        // Data is being loaded via AJAX
        List<WebElement> rows = driver.findElements(By.cssSelector(".data-row"));
        Assert.assertEquals(rows.size(), 10); // Might be 0 or partial
    }
    
    // ✅ STABLE: Wait for AJAX to complete
    @Test
    public void stableAjax() {
        driver.findElement(By.id("load-data")).click();
        
        WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
        
        // Wait for loading indicator to disappear
        wait.until(ExpectedConditions.invisibilityOfElementLocated(
            By.id("loading-spinner")
        ));
        
        // Wait for expected number of rows
        wait.until((WebDriver d) -> {
            List<WebElement> rows = d.findElements(By.cssSelector(".data-row"));
            return rows.size() == 10;
        });
        
        List<WebElement> rows = driver.findElements(By.cssSelector(".data-row"));
        Assert.assertEquals(rows.size(), 10);
    }
}
```

### 3. Element Staleness

```java
public class StalenessIssues {
    
    // ❌ FLAKY: StaleElementReferenceException
    @Test
    public void flakyStaleElement() {
        driver.get("https://example.com");
        
        WebElement button = driver.findElement(By.id("refresh-button"));
        button.click(); // Triggers page refresh
        
        // Element is now stale (detached from DOM)
        button.click(); // StaleElementReferenceException
    }
    
    // ✅ STABLE: Re-find element
    @Test
    public void stableRefindElement() {
        driver.get("https://example.com");
        
        driver.findElement(By.id("refresh-button")).click();
        
        // Wait for page to refresh
        WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
        wait.until(ExpectedConditions.presenceOfElementLocated(By.id("refresh-button")));
        
        // Re-find element
        driver.findElement(By.id("refresh-button")).click();
    }
    
    // ✅ BEST: Utility method with retry
    public WebElement findElementWithRetry(By locator, int maxRetries) {
        int attempts = 0;
        while (attempts < maxRetries) {
            try {
                WebElement element = driver.findElement(locator);
                element.isDisplayed(); // Trigger staleness check
                return element;
            } catch (StaleElementReferenceException e) {
                attempts++;
                if (attempts >= maxRetries) {
                    throw e;
                }
                try {
                    Thread.sleep(500);
                } catch (InterruptedException ie) {
                    Thread.currentThread().interrupt();
                }
            }
        }
        return null;
    }
}
```

### 4. Test Order Dependencies

```java
public class TestOrderDependencies {
    
    // ❌ FLAKY: Depends on test1 running first
    public class FlakyOrderTests {
        private static String userId;
        
        @Test
        public void test1_CreateUser() {
            userId = createUser("testuser");
            Assert.assertNotNull(userId);
        }
        
        @Test
        public void test2_UpdateUser() {
            // Fails if test1 doesn't run first or fails
            updateUser(userId, "newname");
            Assert.assertEquals(getUser(userId).getName(), "newname");
        }
    }
    
    // ✅ STABLE: Each test is independent
    public class StableIndependentTests {
        
        @Test
        public void testCreateUser() {
            String userId = createUser("testuser");
            Assert.assertNotNull(userId);
            cleanup(userId);
        }
        
        @Test
        public void testUpdateUser() {
            // Create user in this test
            String userId = createUser("testuser");
            
            updateUser(userId, "newname");
            Assert.assertEquals(getUser(userId).getName(), "newname");
            
            cleanup(userId);
        }
    }
    
    // ✅ BETTER: Use @BeforeMethod for setup
    public class BetterIndependentTests {
        private String userId;
        
        @BeforeMethod
        public void setup() {
            userId = createUser("testuser_" + System.currentTimeMillis());
        }
        
        @Test
        public void testUpdateUser() {
            updateUser(userId, "newname");
            Assert.assertEquals(getUser(userId).getName(), "newname");
        }
        
        @AfterMethod
        public void cleanup() {
            deleteUser(userId);
        }
    }
}
```

### 5. Shared State / Test Data Conflicts

```java
public class SharedStateIssues {
    
    // ❌ FLAKY: Tests share same test user
    public class FlakySharedState {
        private static final String TEST_USER = "testuser@example.com";
        
        @Test
        public void testLogin() {
            login(TEST_USER, "password");
            Assert.assertTrue(isLoggedIn());
            logout();
        }
        
        @Test
        public void testDeleteAccount() {
            login(TEST_USER, "password");
            deleteAccount();
            // Now testLogin will fail if this runs first
        }
    }
    
    // ✅ STABLE: Each test uses unique data
    public class StableUniqueData {
        
        @Test
        public void testLogin() {
            String uniqueUser = "user_" + System.currentTimeMillis() + "@example.com";
            createUser(uniqueUser, "password");
            
            login(uniqueUser, "password");
            Assert.assertTrue(isLoggedIn());
            
            cleanup(uniqueUser);
        }
        
        @Test
        public void testDeleteAccount() {
            String uniqueUser = "user_" + System.currentTimeMillis() + "@example.com";
            createUser(uniqueUser, "password");
            
            login(uniqueUser, "password");
            deleteAccount();
            
            Assert.assertFalse(userExists(uniqueUser));
        }
    }
}
```

### 6. Network Issues

```java
public class NetworkIssues {
    
    // ❌ FLAKY: No handling for network delays
    @Test
    public void flakyAPICall() {
        Response response = RestAssured.get("https://api.example.com/data");
        Assert.assertEquals(response.getStatusCode(), 200);
    }
    
    // ✅ STABLE: Retry mechanism for network calls
    @Test
    public void stableAPICall() {
        Response response = callAPIWithRetry("https://api.example.com/data", 3);
        Assert.assertEquals(response.getStatusCode(), 200);
    }
    
    private Response callAPIWithRetry(String url, int maxRetries) {
        int attempts = 0;
        while (attempts < maxRetries) {
            try {
                Response response = RestAssured
                    .given()
                        .config(RestAssured.config()
                            .connectionConfig(connectionConfig()
                                .closeIdleConnectionsAfterEachResponseAfter(3, TimeUnit.SECONDS)))
                    .when()
                        .get(url);
                
                if (response.getStatusCode() == 200) {
                    return response;
                }
            } catch (Exception e) {
                attempts++;
                if (attempts >= maxRetries) {
                    throw e;
                }
                try {
                    Thread.sleep(1000 * attempts); // Exponential backoff
                } catch (InterruptedException ie) {
                    Thread.currentThread().interrupt();
                }
            }
        }
        return null;
    }
}
```

### 7. Environment Issues

```java
public class EnvironmentIssues {
    
    // ❌ FLAKY: Hardcoded environment-specific data
    @Test
    public void flakyEnvironment() {
        driver.get("http://localhost:3000"); // Only works locally
        // Fails in CI where port might be different
    }
    
    // ✅ STABLE: Environment-agnostic
    @Test
    public void stableEnvironment() {
        String baseUrl = System.getProperty("base.url", "http://localhost:3000");
        driver.get(baseUrl);
    }
    
    // ❌ FLAKY: Time zone dependent
    @Test
    public void flakyTimeZone() {
        String displayedDate = driver.findElement(By.id("date")).getText();
        Assert.assertEquals(displayedDate, "2024-01-15 10:00 AM");
        // Fails in different time zones
    }
    
    // ✅ STABLE: Time zone aware
    @Test
    public void stableTimeZone() {
        String displayedDate = driver.findElement(By.id("date")).getText();
        
        // Parse and compare considering time zone
        DateTimeFormatter formatter = DateTimeFormatter
            .ofPattern("yyyy-MM-dd hh:mm a")
            .withZone(ZoneId.of("America/New_York"));
        
        LocalDateTime expected = LocalDateTime.of(2024, 1, 15, 10, 0);
        // Compare logic considering time zone
    }
}
```

---

## Detection Strategies

### 1. Automated Flaky Test Detection

```java
public class FlakyTestDetector {
    
    /**
     * Run each test multiple times to detect flakiness
     */
    public static void main(String[] args) throws Exception {
        int iterations = 10;
        Map<String, TestResult> results = new HashMap<>();
        
        // Get all test methods
        List<Method> testMethods = getTestMethods();
        
        for (Method testMethod : testMethods) {
            int passCount = 0;
            int failCount = 0;
            List<String> failureReasons = new ArrayList<>();
            
            for (int i = 0; i < iterations; i++) {
                try {
                    // Run test
                    testMethod.invoke(testMethod.getDeclaringClass().newInstance());
                    passCount++;
                } catch (Exception e) {
                    failCount++;
                    failureReasons.add(e.getMessage());
                }
            }
            
            // Analyze results
            if (passCount > 0 && failCount > 0) {
                System.out.println("🚨 FLAKY TEST DETECTED: " + testMethod.getName());
                System.out.println("   Passed: " + passCount + "/" + iterations);
                System.out.println("   Failed: " + failCount + "/" + iterations);
                System.out.println("   Flakiness Rate: " + 
                    (failCount * 100.0 / iterations) + "%");
                System.out.println("   Failure Reasons: " + failureReasons);
            } else if (failCount == iterations) {
                System.out.println("❌ CONSISTENTLY FAILING: " + testMethod.getName());
            } else {
                System.out.println("✅ STABLE: " + testMethod.getName());
            }
        }
    }
}
```

### 2. TestNG Retry Analyzer

```java
import org.testng.IRetryAnalyzer;
import org.testng.ITestResult;

public class RetryAnalyzer implements IRetryAnalyzer {
    private int retryCount = 0;
    private static final int MAX_RETRY_COUNT = 3;
    
    // Track which tests are being retried (potential flaky tests)
    private static Map<String, Integer> retryMap = new ConcurrentHashMap<>();
    
    @Override
    public boolean retry(ITestResult result) {
        if (retryCount < MAX_RETRY_COUNT) {
            retryCount++;
            
            String testName = result.getMethod().getMethodName();
            retryMap.put(testName, retryMap.getOrDefault(testName, 0) + 1);
            
            System.out.println("⚠️  Retrying test: " + testName + 
                             " (Attempt " + (retryCount + 1) + ")");
            
            return true;
        }
        return false;
    }
    
    // Report flaky tests at the end
    public static void reportFlakyTests() {
        System.out.println("\n=== FLAKY TESTS REPORT ===");
        
        if (retryMap.isEmpty()) {
            System.out.println("✅ No flaky tests detected!");
        } else {
            retryMap.forEach((testName, retries) -> {
                System.out.println("🚨 " + testName + " - Retried " + 
                                 retries + " times");
            });
        }
    }
}

// Usage in test class
public class MyTest {
    
    @Test(retryAnalyzer = RetryAnalyzer.class)
    public void testSomething() {
        // Test code
    }
}
```

### 3. CI/CD Flaky Test Tracking

```groovy
// Jenkins Pipeline
pipeline {
    agent any
    
    stages {
        stage('Run Tests Multiple Times') {
            steps {
                script {
                    def iterations = 5
                    def results = [:]
                    
                    for (int i = 1; i <= iterations; i++) {
                        echo "Test iteration ${i}/${iterations}"
                        
                        def testResult = sh(
                            script: 'mvn test',
                            returnStatus: true
                        )
                        
                        results["iteration_${i}"] = testResult == 0 ? "PASS" : "FAIL"
                    }
                    
                    // Analyze results
                    def passes = results.values().count("PASS")
                    def fails = results.values().count("FAIL")
                    
                    if (passes > 0 && fails > 0) {
                        echo "🚨 FLAKY TESTS DETECTED"
                        echo "Passed: ${passes}/${iterations}"
                        echo "Failed: ${fails}/${iterations}"
                        
                        // Mark build as unstable
                        currentBuild.result = 'UNSTABLE'
                        
                        // Send notification
                        emailext(
                            subject: "Flaky Tests Detected",
                            body: "Test suite showed inconsistent results",
                            to: "team@example.com"
                        )
                    }
                }
            }
        }
    }
}
```

---

## Fixing Common Flaky Patterns

### Flaky Pattern #1: Clickability Issues

```java
public class ClickabilityFixes {
    
    // ❌ FLAKY: Element not clickable
    public void flakyClick() {
        WebElement button = driver.findElement(By.id("submit"));
        button.click(); // ElementClickInterceptedException
    }
    
    // ✅ FIX 1: Wait for element to be clickable
    public void fixClickableWait() {
        WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
        WebElement button = wait.until(
            ExpectedConditions.elementToBeClickable(By.id("submit"))
        );
        button.click();
    }
    
    // ✅ FIX 2: Scroll into view
    public void fixScrollIntoView() {
        WebElement button = driver.findElement(By.id("submit"));
        ((JavascriptExecutor) driver).executeScript(
            "arguments[0].scrollIntoView({block: 'center'});", 
            button
        );
        
        WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(5));
        wait.until(ExpectedConditions.elementToBeClickable(button));
        button.click();
    }
    
    // ✅ FIX 3: JavaScript click (last resort)
    public void fixJavaScriptClick() {
        WebElement button = driver.findElement(By.id("submit"));
        ((JavascriptExecutor) driver).executeScript(
            "arguments[0].click();", 
            button
        );
    }
    
    // ✅ BEST: Comprehensive click method
    public void robustClick(By locator) {
        WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
        
        try {
            // Method 1: Standard click with wait
            WebElement element = wait.until(
                ExpectedConditions.elementToBeClickable(locator)
            );
            element.click();
        } catch (ElementClickInterceptedException e) {
            // Method 2: Scroll and retry
            WebElement element = driver.findElement(locator);
            ((JavascriptExecutor) driver).executeScript(
                "arguments[0].scrollIntoView({block: 'center'});", 
                element
            );
            Thread.sleep(500);
            element.click();
        } catch (Exception e) {
            // Method 3: JavaScript click
            WebElement element = driver.findElement(locator);
            ((JavascriptExecutor) driver).executeScript(
                "arguments[0].click();", 
                element
            );
        }
    }
}
```

### Flaky Pattern #2: Dropdown Selection

```java
public class DropdownFixes {
    
    // ❌ FLAKY: Dropdown not fully loaded
    public void flakyDropdown() {
        Select dropdown = new Select(driver.findElement(By.id("country")));
        dropdown.selectByVisibleText("United States");
    }
    
    // ✅ FIX: Wait for options to load
    public void fixDropdown() {
        WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
        
        // Wait for dropdown to be present
        wait.until(ExpectedConditions.presenceOfElementLocated(By.id("country")));
        
        // Wait for options to be populated
        wait.until((WebDriver d) -> {
            Select dropdown = new Select(d.findElement(By.id("country")));
            return dropdown.getOptions().size() > 1; // More than just placeholder
        });
        
        Select dropdown = new Select(driver.findElement(By.id("country")));
        dropdown.selectByVisibleText("United States");
        
        // Verify selection
        wait.until((WebDriver d) -> {
            Select dd = new Select(d.findElement(By.id("country")));
            return dd.getFirstSelectedOption().getText().equals("United States");
        });
    }
}
```

### Flaky Pattern #3: File Upload

```java
public class FileUploadFixes {
    
    // ❌ FLAKY: File upload timing issues
    public void flakyFileUpload() {
        WebElement fileInput = driver.findElement(By.id("file-upload"));
        fileInput.sendKeys("/path/to/file.pdf");
        driver.findElement(By.id("submit")).click();
        
        // Immediately check for success (file might still be uploading)
        Assert.assertTrue(driver.findElement(By.id("success")).isDisplayed());
    }
    
    // ✅ FIX: Wait for upload to complete
    public void fixFileUpload() {
        WebElement fileInput = driver.findElement(By.id("file-upload"));
        fileInput.sendKeys("/path/to/file.pdf");
        
        driver.findElement(By.id("submit")).click();
        
        WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(30));
        
        // Wait for upload progress indicator to disappear
        wait.until(ExpectedConditions.invisibilityOfElementLocated(
            By.id("upload-progress")
        ));
        
        // Wait for success message
        wait.until(ExpectedConditions.visibilityOfElementLocated(
            By.id("success")
        ));
        
        Assert.assertTrue(driver.findElement(By.id("success")).isDisplayed());
    }
}
```

---

## Retry Mechanisms

### Smart Retry Strategy

```java
import org.testng.IRetryAnalyzer;
import org.testng.ITestResult;

public class SmartRetryAnalyzer implements IRetryAnalyzer {
    private int retryCount = 0;
    private static final int MAX_RETRY_COUNT = 2;
    
    // Don't retry certain types of failures
    private static final List<Class<? extends Throwable>> NON_RETRYABLE_EXCEPTIONS = Arrays.asList(
        AssertionError.class,  // Assertion failures (test logic issue)
        NullPointerException.class,  // Code bug
        IllegalArgumentException.class  // Code bug
    );
    
    @Override
    public boolean retry(ITestResult result) {
        if (retryCount >= MAX_RETRY_COUNT) {
            return false;
        }
        
        Throwable throwable = result.getThrowable();
        
        // Don't retry for non-retryable exceptions
        if (throwable != null) {
            for (Class<? extends Throwable> exceptionClass : NON_RETRYABLE_EXCEPTIONS) {
                if (exceptionClass.isInstance(throwable)) {
                    System.out.println("Not retrying due to: " + 
                                     exceptionClass.getSimpleName());
                    return false;
                }
            }
        }
        
        // Only retry for flakiness-related exceptions
        if (throwable instanceof TimeoutException ||
            throwable instanceof StaleElementReferenceException ||
            throwable instanceof ElementClickInterceptedException ||
            throwable instanceof NoSuchElementException) {
            
            retryCount++;
            System.out.println("Retrying test (attempt " + (retryCount + 1) + 
                             ") due to: " + throwable.getClass().getSimpleName());
            
            // Add delay between retries
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
            
            return true;
        }
        
        return false;
    }
}
```

---

## Preventing Flakiness

### Best Practices Checklist

```java
public class FlakinessPreventionChecklist {
    
    /**
     * ✅ 1. Use Explicit Waits
     */
    private WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
    
    public void waitForElement() {
        wait.until(ExpectedConditions.visibilityOfElementLocated(By.id("element")));
    }
    
    /**
     * ✅ 2. Independent Tests
     */
    @Test
    public void test1() {
        // Setup its own data
        String userId = createTestUser();
        
        // Test logic
        
        // Cleanup
        deleteUser(userId);
    }
    
    /**
     * ✅ 3. Unique Test Data
     */
    public String getUniqueUsername() {
        return "user_" + System.currentTimeMillis() + "_" + 
               UUID.randomUUID().toString().substring(0, 8);
    }
    
    /**
     * ✅ 4. Idempotent Tests
     */
    @Test
    public void idempotentTest() {
        // Test can run multiple times with same result
        // Clean up state at beginning
        cleanup();
        
        // Test logic
        // ...
        
        // Clean up state at end
        cleanup();
    }
    
    /**
     * ✅ 5. Avoid Thread.sleep()
     */
    // ❌ Don't do this
    // Thread.sleep(5000);
    
    // ✅ Do this instead
    wait.until(ExpectedConditions.visibilityOfElementLocated(By.id("element")));
    
    /**
     * ✅ 6. Handle Stale Elements
     */
    public WebElement findElementSafely(By locator) {
        int attempts = 0;
        while (attempts < 3) {
            try {
                WebElement element = driver.findElement(locator);
                element.isDisplayed();
                return element;
            } catch (StaleElementReferenceException e) {
                attempts++;
            }
        }
        throw new RuntimeException("Element is stale after 3 attempts");
    }
    
    /**
     * ✅ 7. Verify Ajax Complete
     */
    public void waitForAjaxToComplete() {
        wait.until((WebDriver d) -> {
            return (Boolean) ((JavascriptExecutor) d).executeScript(
                "return jQuery.active == 0"
            );
        });
    }
    
    /**
     * ✅ 8. Use Stable Locators
     */
    // ❌ Bad: Fragile locators
    // By.xpath("//div[1]/div[2]/span[3]")
    
    // ✅ Good: Stable locators
    By.id("username")
    By.cssSelector("[data-testid='submit-button']")
    
    /**
     * ✅ 9. Screenshot on Failure
     */
    @AfterMethod
    public void takeScreenshotOnFailure(ITestResult result) {
        if (result.getStatus() == ITestResult.FAILURE) {
            File screenshot = ((TakesScreenshot) driver)
                .getScreenshotAs(OutputType.FILE);
            // Save screenshot
        }
    }
    
    /**
     * ✅ 10. Proper Test Cleanup
     */
    @AfterMethod(alwaysRun = true)
    public void cleanup() {
        // Always cleanup, even if test fails
        deleteTestData();
        logout();
    }
}
```

---

## Organizational Approach

### Flaky Test Management Strategy

```markdown
## 1. Quarantine Flaky Tests

**Approach:**
- Tag flaky tests with @Flaky annotation
- Move to separate test suite
- Don't let them block CI/CD
- Track and fix systematically

```java
@Test(groups = {"flaky"})
@Flaky(reason = "Timing issue with animation", jiraTicket = "QA-123")
public void flakyTest() {
    // Test code
}
```

## 2. Flaky Test Dashboard

**Track:**
- Flakiness rate per test
- Most flaky tests
- Trends over time
- Root cause categories

**Metrics:**
```java
public class FlakyTestMetrics {
    String testName;
    int totalRuns;
    int failures;
    double flakinessRate; // failures/totalRuns
    String rootCause;
    String owner;
    LocalDate firstDetected;
    LocalDate lastOccurrence;
}
```

## 3. Team Culture

**DO:**
✅ Treat flaky tests seriously
✅ Fix flaky tests immediately
✅ Root cause analysis
✅ Share learnings
✅ Code review for flakiness patterns
✅ Continuous improvement

**DON'T:**
❌ Ignore flaky failures
❌ "Just rerun it"
❌ Disable flaky tests permanently
❌ Accept "sometimes it fails"
❌ Blame infrastructure only

## 4. Prevention in Code Reviews

**Review Checklist:**
□ Are explicit waits used?
□ Are tests independent?
□ Is test data unique?
□ Are there hard-coded sleeps?
□ Are locators stable?
□ Is cleanup handled properly?
□ Could this be flaky?
```

---

## Interview Questions

### Q1: What causes flaky tests and how do you handle them?

**Answer:**
```markdown
**Main Causes:**

1. **Timing Issues (70%):**
   - Race conditions
   - Inadequate waits
   - AJAX/animations not complete
   
   **Solution:** Use explicit waits

2. **Test Dependencies:**
   - Tests not isolated
   - Shared test data
   
   **Solution:** Independent tests with unique data

3. **Environment Issues:**
   - Network problems
   - Resource constraints
   
   **Solution:** Retry mechanisms, stable environment

**Handling Strategy:**

1. **Detect:**
   - Run tests multiple times
   - Track retry counts
   - Monitor patterns

2. **Triage:**
   - Quarantine flaky tests
   - Don't block CI/CD
   - Prioritize by impact

3. **Fix:**
   - Root cause analysis
   - Apply proper waits
   - Make tests independent
   
4. **Prevent:**
   - Code review guidelines
   - Best practices
   - Team training
```

### Q2: Should you use retry mechanisms for flaky tests?

**Answer:**
```markdown
**Short term: YES (with caveats)**
- Prevents false negatives
- Keeps CI/CD running
- Buys time for proper fix

**Long term: NO**
- Retries are a band-aid
- Must fix root cause
- Don't hide real issues

**Best Approach:**

1. ✅ Implement smart retry
   - Only for flakiness exceptions
   - Limited retries (2-3 max)
   - Track retry metrics

2. ✅ Track and fix
   - Monitor which tests retry
   - Create tickets for flaky tests
   - Fix systematically

3. ❌ Don't abuse retries
   - Don't retry assertion failures
   - Don't retry indefinitely
   - Don't use as permanent solution

**Example:**
```java
@Test(retryAnalyzer = SmartRetryAnalyzer.class)
public void testLogin() {
    // If fails due to timeout -> retry
    // If fails due to assertion -> don't retry
}
```
```

---

**Next:** [Visual Regression Testing](21-visual-regression-testing.md)

