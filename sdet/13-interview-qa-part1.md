# 13. Interview Questions & Answers - Part 1

## 📚 Quick Summary

500+ real interview questions with perfect answers - your secret weapon!

**What's Inside:**
- **Core Java Questions**: 100+ questions on multi-threading, collections, Java 8+
- **Selenium Questions**: 80+ questions on waits, POM, Grid, locators
- **Framework Design**: 50+ questions on architecture, design patterns
- **API Testing**: 40+ questions on RestAssured, authentication

**Why This Matters:**
- These are ACTUAL questions from real interviews
- Practice answering aloud for fluency
- Understand not just "what" but "why"
- Many questions have follow-up questions

**How to Use:**
1. Read question
2. Try to answer (without looking)
3. Read provided answer
4. Practice explaining to someone else
5. Mark difficult ones, revisit later

**Interview Tip:**
Don't memorize word-for-word. Understand concepts, then explain in your own words!

---

## 📖 Simple Explanation

**What makes a good interview answer?**

❌ **Bad Answer:**
"Yes, I know HashMap." (Too short, no depth)

✅ **Good Answer:**
"HashMap is a key-value data structure in Java. It uses hashing to store elements, offering O(1) average time for get/put operations. It's not thread-safe, so for concurrent access, I use ConcurrentHashMap. In testing, I use HashMap to store test data, like mapping test IDs to test names or storing configuration."

**Structure: STAR Format (for behavioral)**
- **Situation**: Context
- **Task**: What needed to be done
- **Action**: What YOU did
- **Result**: Outcome and learning

**For Technical Questions:**
1. **Definition**: What is it?
2. **How it works**: Brief explanation
3. **Use case**: When/why to use
4. **Example**: Real scenario from your work

---

## Core Java Questions

### Q1: Explain the difference between HashMap and ConcurrentHashMap

**Answer:**

| Feature | HashMap | ConcurrentHashMap |
|---------|---------|-------------------|
| Thread Safety | Not thread-safe | Thread-safe |
| Null Keys/Values | Allows one null key, multiple null values | Does not allow null keys or values |
| Performance | Faster in single-threaded environment | Optimized for concurrent access |
| Locking | No locking mechanism | Uses segment-level locking (Java 7) or CAS operations (Java 8+) |
| Iteration | Fail-fast iterator | Weakly consistent iterator |

**Code Example:**
```java
// HashMap - Not thread-safe
Map<String, String> hashMap = new HashMap<>();
hashMap.put(null, "value");  // Allowed

// ConcurrentHashMap - Thread-safe
Map<String, String> concurrentMap = new ConcurrentHashMap<>();
// concurrentMap.put(null, "value");  // Throws NullPointerException

// Concurrent access example
ConcurrentHashMap<Integer, String> map = new ConcurrentHashMap<>();

// Multiple threads can safely access
ExecutorService executor = Executors.newFixedThreadPool(3);
for (int i = 0; i < 1000; i++) {
    final int value = i;
    executor.submit(() -> {
        map.put(value, "Value" + value);
    });
}
```

**Follow-up:** What is the internal structure of ConcurrentHashMap?
- Java 7: Divided into segments (default 16), each segment has its own lock
- Java 8+: Uses CAS (Compare-And-Swap) operations and synchronized blocks on individual nodes

---

### Q2: How do you implement a thread-safe Singleton pattern?

**Answer:**

```java
// Method 1: Eager Initialization (Thread-safe but not lazy)
public class Singleton {
    private static final Singleton INSTANCE = new Singleton();
    
    private Singleton() {
        // Prevent reflection
        if (INSTANCE != null) {
            throw new RuntimeException("Use getInstance() method");
        }
    }
    
    public static Singleton getInstance() {
        return INSTANCE;
    }
}

// Method 2: Double-Checked Locking (Lazy + Thread-safe)
public class Singleton {
    private static volatile Singleton instance;
    
    private Singleton() {}
    
    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}

// Method 3: Bill Pugh Singleton (Best approach)
public class Singleton {
    private Singleton() {}
    
    private static class SingletonHelper {
        private static final Singleton INSTANCE = new Singleton();
    }
    
    public static Singleton getInstance() {
        return SingletonHelper.INSTANCE;
    }
}

// Method 4: Enum Singleton (Most robust)
public enum Singleton {
    INSTANCE;
    
    public void doSomething() {
        // Business logic
    }
}
```

**Why volatile in double-checked locking?**
- Prevents instruction reordering
- Ensures visibility of changes across threads
- Without volatile, thread may see partially constructed object

---

### Q3: Explain Stream API with complex examples

**Answer:**

```java
List<Employee> employees = Arrays.asList(
    new Employee("John", "IT", 50000, 30),
    new Employee("Alice", "HR", 60000, 25),
    new Employee("Bob", "IT", 55000, 35),
    new Employee("Charlie", "Finance", 70000, 28),
    new Employee("David", "IT", 52000, 32)
);

// 1. Group employees by department
Map<String, List<Employee>> byDept = employees.stream()
    .collect(Collectors.groupingBy(Employee::getDepartment));

// 2. Average salary by department
Map<String, Double> avgSalaryByDept = employees.stream()
    .collect(Collectors.groupingBy(
        Employee::getDepartment,
        Collectors.averagingDouble(Employee::getSalary)
    ));

// 3. Find highest paid employee in each department
Map<String, Optional<Employee>> highestPaidByDept = employees.stream()
    .collect(Collectors.groupingBy(
        Employee::getDepartment,
        Collectors.maxBy(Comparator.comparing(Employee::getSalary))
    ));

// 4. Get names of employees in IT with salary > 52000
List<String> itEmployees = employees.stream()
    .filter(e -> e.getDepartment().equals("IT"))
    .filter(e -> e.getSalary() > 52000)
    .map(Employee::getName)
    .collect(Collectors.toList());

// 5. Partition employees by salary (> 55000 or not)
Map<Boolean, List<Employee>> partitioned = employees.stream()
    .collect(Collectors.partitioningBy(e -> e.getSalary() > 55000));

// 6. Find second highest salary
Double secondHighest = employees.stream()
    .map(Employee::getSalary)
    .distinct()
    .sorted(Comparator.reverseOrder())
    .skip(1)
    .findFirst()
    .orElse(0.0);

// 7. Calculate total salary expense
double totalSalary = employees.stream()
    .mapToDouble(Employee::getSalary)
    .sum();

// 8. Get comma-separated names
String names = employees.stream()
    .map(Employee::getName)
    .collect(Collectors.joining(", "));

// 9. Find employee with longest name
Optional<Employee> longestName = employees.stream()
    .max(Comparator.comparing(e -> e.getName().length()));

// 10. Custom collector - employees by age range
Map<String, List<Employee>> byAgeRange = employees.stream()
    .collect(Collectors.groupingBy(e -> {
        if (e.getAge() < 30) return "Young";
        else if (e.getAge() < 35) return "Mid";
        else return "Senior";
    }));
```

---

## Selenium Questions

### Q4: How do you handle StaleElementReferenceException?

**Answer:**

StaleElementReferenceException occurs when:
1. Element is deleted from DOM
2. Page is refreshed
3. Element is re-rendered

**Solutions:**

```java
// Method 1: Retry Mechanism
public WebElement findElementWithRetry(By locator, int maxRetries) {
    WebElement element = null;
    int attempts = 0;
    
    while (attempts < maxRetries) {
        try {
            element = driver.findElement(locator);
            return element;
        } catch (StaleElementReferenceException e) {
            attempts++;
            System.out.println("Stale element, retry " + attempts);
        }
    }
    throw new RuntimeException("Failed after " + maxRetries + " attempts");
}

// Method 2: ExpectedConditions with retry
public void clickWithRetry(By locator) {
    int attempts = 0;
    while (attempts < 3) {
        try {
            WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
            WebElement element = wait.until(
                ExpectedConditions.elementToBeClickable(locator)
            );
            element.click();
            break;
        } catch (StaleElementReferenceException e) {
            attempts++;
        }
    }
}

// Method 3: Find element fresh each time
public void clickElement(By locator) {
    driver.findElement(locator).click();  // Find and use immediately
}

// Method 4: Custom ExpectedCondition
public void waitForElementRefresh(final By locator) {
    WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
    wait.until((ExpectedCondition<Boolean>) driver -> {
        try {
            driver.findElement(locator);
            return true;
        } catch (StaleElementReferenceException e) {
            return false;
        }
    });
}

// Method 5: Page refresh wrapper
public void performActionWithRetry(WebElement element, Consumer<WebElement> action) {
    int maxAttempts = 3;
    int attempts = 0;
    
    while (attempts < maxAttempts) {
        try {
            action.accept(element);
            return;
        } catch (StaleElementReferenceException e) {
            attempts++;
            if (attempts >= maxAttempts) {
                throw e;
            }
        }
    }
}

// Usage
performActionWithRetry(element, e -> e.click());
performActionWithRetry(element, e -> e.sendKeys("text"));
```

**Best Practice:**
- Find element fresh when needed
- Use waits appropriately
- Avoid storing WebElements as instance variables

---

### Q5: Explain different wait strategies and when to use each

**Answer:**

**1. Implicit Wait**
```java
driver.manage().timeouts().implicitlyWait(Duration.ofSeconds(10));
```
- Applied globally for all findElement calls
- Not recommended to mix with explicit waits
- Use case: Simple scripts, quick prototyping

**2. Explicit Wait**
```java
WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(20));
WebElement element = wait.until(
    ExpectedConditions.visibilityOfElementLocated(By.id("element"))
);
```
- Wait for specific condition
- More control and flexibility
- Use case: Wait for specific elements/conditions

**3. Fluent Wait**
```java
Wait<WebDriver> wait = new FluentWait<>(driver)
    .withTimeout(Duration.ofSeconds(30))
    .pollingEvery(Duration.ofMillis(500))
    .ignoring(NoSuchElementException.class);

WebElement element = wait.until(driver -> {
    return driver.findElement(By.id("dynamic"));
});
```
- Customizable polling interval
- Can ignore specific exceptions
- Use case: Elements that load unpredictably

**Comparison Table:**

| Wait Type | Scope | Polling | Exceptions Handled | Best For |
|-----------|-------|---------|-------------------|----------|
| Implicit | Global | Fixed | NoSuchElementException | Simple tests |
| Explicit | Specific | Fixed (500ms) | Customizable | Most scenarios |
| Fluent | Specific | Customizable | Customizable | Complex scenarios |

**Practical Example:**
```java
public class WaitStrategy {
    
    // Wait for element to be visible
    public WebElement waitForVisibility(By locator) {
        WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(20));
        return wait.until(ExpectedConditions.visibilityOfElementLocated(locator));
    }
    
    // Wait for element to be clickable
    public WebElement waitForClickable(By locator) {
        WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(20));
        return wait.until(ExpectedConditions.elementToBeClickable(locator));
    }
    
    // Wait for AJAX to complete
    public void waitForAjax() {
        WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(20));
        wait.until(driver -> {
            JavascriptExecutor js = (JavascriptExecutor) driver;
            return (Boolean) js.executeScript("return jQuery.active == 0");
        });
    }
    
    // Wait for page load
    public void waitForPageLoad() {
        WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(30));
        wait.until(driver -> {
            JavascriptExecutor js = (JavascriptExecutor) driver;
            return js.executeScript("return document.readyState").equals("complete");
        });
    }
    
    // Wait for element to disappear
    public void waitForInvisibility(By locator) {
        WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(20));
        wait.until(ExpectedConditions.invisibilityOfElementLocated(locator));
    }
    
    // Custom wait - element attribute value
    public void waitForAttributeValue(By locator, String attribute, String value) {
        WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(20));
        wait.until(driver -> {
            WebElement element = driver.findElement(locator);
            return element.getAttribute(attribute).equals(value);
        });
    }
}
```

---

### Q6: Design a framework to handle dynamic web tables

**Answer:**

```java
public class WebTableHandler {
    private WebDriver driver;
    private By tableLocator;
    
    public WebTableHandler(WebDriver driver, By tableLocator) {
        this.driver = driver;
        this.tableLocator = tableLocator;
    }
    
    // Get row count
    public int getRowCount() {
        WebElement table = driver.findElement(tableLocator);
        List<WebElement> rows = table.findElements(By.tagName("tr"));
        return rows.size() - 1;  // Exclude header
    }
    
    // Get column count
    public int getColumnCount() {
        WebElement table = driver.findElement(tableLocator);
        WebElement headerRow = table.findElement(By.tagName("tr"));
        List<WebElement> headers = headerRow.findElements(By.tagName("th"));
        return headers.size();
    }
    
    // Get cell value
    public String getCellValue(int row, int column) {
        WebElement table = driver.findElement(tableLocator);
        WebElement cell = table.findElement(
            By.xpath(".//tr[" + (row + 1) + "]//td[" + column + "]")
        );
        return cell.getText();
    }
    
    // Get all data from table
    public List<List<String>> getAllData() {
        List<List<String>> tableData = new ArrayList<>();
        int rows = getRowCount();
        int cols = getColumnCount();
        
        for (int i = 1; i <= rows; i++) {
            List<String> rowData = new ArrayList<>();
            for (int j = 1; j <= cols; j++) {
                rowData.add(getCellValue(i, j));
            }
            tableData.add(rowData);
        }
        
        return tableData;
    }
    
    // Search for value in table
    public boolean searchValueInTable(String searchValue) {
        WebElement table = driver.findElement(tableLocator);
        List<WebElement> cells = table.findElements(By.tagName("td"));
        
        return cells.stream()
            .anyMatch(cell -> cell.getText().equals(searchValue));
    }
    
    // Get row number containing value
    public int getRowNumberWithValue(String searchValue, int columnIndex) {
        int rowCount = getRowCount();
        
        for (int i = 1; i <= rowCount; i++) {
            String cellValue = getCellValue(i, columnIndex);
            if (cellValue.equals(searchValue)) {
                return i;
            }
        }
        return -1;
    }
    
    // Get entire row data
    public List<String> getRowData(int rowNumber) {
        List<String> rowData = new ArrayList<>();
        int cols = getColumnCount();
        
        for (int j = 1; j <= cols; j++) {
            rowData.add(getCellValue(rowNumber, j));
        }
        
        return rowData;
    }
    
    // Get column data
    public List<String> getColumnData(int columnIndex) {
        List<String> columnData = new ArrayList<>();
        int rows = getRowCount();
        
        for (int i = 1; i <= rows; i++) {
            columnData.add(getCellValue(i, columnIndex));
        }
        
        return columnData;
    }
    
    // Get headers
    public List<String> getHeaders() {
        WebElement table = driver.findElement(tableLocator);
        List<WebElement> headers = table.findElements(
            By.xpath(".//thead//th | .//tr[1]//th")
        );
        
        return headers.stream()
            .map(WebElement::getText)
            .collect(Collectors.toList());
    }
    
    // Search with multiple criteria
    public List<Integer> searchRows(Map<String, String> criteria) {
        List<Integer> matchingRows = new ArrayList<>();
        List<String> headers = getHeaders();
        int rowCount = getRowCount();
        
        for (int row = 1; row <= rowCount; row++) {
            boolean matches = true;
            
            for (Map.Entry<String, String> criterion : criteria.entrySet()) {
                String columnName = criterion.getKey();
                String expectedValue = criterion.getValue();
                
                int columnIndex = headers.indexOf(columnName) + 1;
                String actualValue = getCellValue(row, columnIndex);
                
                if (!actualValue.equals(expectedValue)) {
                    matches = false;
                    break;
                }
            }
            
            if (matches) {
                matchingRows.add(row);
            }
        }
        
        return matchingRows;
    }
    
    // Click element in specific cell
    public void clickElementInCell(int row, int column, By elementLocator) {
        WebElement table = driver.findElement(tableLocator);
        WebElement cell = table.findElement(
            By.xpath(".//tr[" + (row + 1) + "]//td[" + column + "]")
        );
        WebElement element = cell.findElement(elementLocator);
        element.click();
    }
    
    // Sort table by column
    public void sortByColumn(int columnIndex) {
        WebElement table = driver.findElement(tableLocator);
        WebElement header = table.findElement(
            By.xpath(".//thead//th[" + columnIndex + "]")
        );
        header.click();
    }
    
    // Handle pagination
    public void goToNextPage() {
        WebElement nextButton = driver.findElement(By.xpath("//a[text()='Next']"));
        nextButton.click();
    }
    
    public void goToPreviousPage() {
        WebElement prevButton = driver.findElement(By.xpath("//a[text()='Previous']"));
        prevButton.click();
    }
}

// Usage Example
@Test
public void testWebTable() {
    WebTableHandler table = new WebTableHandler(driver, By.id("dataTable"));
    
    // Get all data
    List<List<String>> data = table.getAllData();
    
    // Search for specific value
    int rowNum = table.getRowNumberWithValue("John Doe", 1);
    
    // Get row data
    List<String> rowData = table.getRowData(rowNum);
    
    // Click button in specific cell
    table.clickElementInCell(rowNum, 5, By.xpath(".//button[@class='edit']"));
    
    // Search with criteria
    Map<String, String> criteria = new HashMap<>();
    criteria.put("Name", "John Doe");
    criteria.put("Status", "Active");
    List<Integer> matchingRows = table.searchRows(criteria);
}
```

---

## Framework Design Questions

### Q7: How do you design a scalable automation framework from scratch?

**Answer:**

**Key Components:**

1. **Framework Architecture**
```
automation-framework/
├── src/
│   ├── main/java/
│   │   ├── config/          # Configuration management
│   │   ├── pages/           # Page Objects
│   │   ├── utils/           # Utilities
│   │   ├── constants/       # Constants
│   │   ├── listeners/       # TestNG listeners
│   │   └── reports/         # Reporting
│   └── test/java/
│       ├── tests/           # Test classes
│       ├── testdata/        # Data providers
│       └── suites/          # Test suites
├── resources/
│   ├── config/              # Config files
│   ├── testdata/            # Test data files
│   └── schemas/             # JSON schemas
├── logs/
├── screenshots/
├── test-output/
└── pom.xml
```

2. **Design Principles**

- **Modularity**: Separate concerns (pages, tests, utilities)
- **Reusability**: Create reusable components
- **Maintainability**: Easy to update and maintain
- **Scalability**: Can handle growing test suites
- **Readability**: Clear and understandable code

3. **Key Features**

```java
// a) Configuration Management
public class ConfigManager {
    private static Properties properties;
    
    static {
        loadProperties();
    }
    
    private static void loadProperties() {
        String env = System.getProperty("env", "qa");
        String configFile = "config/" + env + ".properties";
        // Load properties
    }
    
    public static String get(String key) {
        return properties.getProperty(key);
    }
}

// b) Driver Factory
public class DriverFactory {
    private static ThreadLocal<WebDriver> driver = new ThreadLocal<>();
    
    public static WebDriver getDriver() {
        return driver.get();
    }
    
    public static void setDriver(WebDriver driverInstance) {
        driver.set(driverInstance);
    }
    
    public static void initDriver(String browser) {
        WebDriver webDriver = createDriver(browser);
        setDriver(webDriver);
    }
    
    private static WebDriver createDriver(String browser) {
        // Implementation
    }
}

// c) Base Page
public abstract class BasePage {
    protected WebDriver driver;
    protected WebDriverWait wait;
    
    public BasePage() {
        this.driver = DriverFactory.getDriver();
        this.wait = new WebDriverWait(driver, Duration.ofSeconds(20));
        PageFactory.initElements(driver, this);
    }
    
    protected void click(WebElement element) {
        wait.until(ExpectedConditions.elementToBeClickable(element));
        element.click();
    }
    
    // Other common methods
}

// d) Base Test
public abstract class BaseTest {
    
    @BeforeMethod
    public void setUp() {
        DriverFactory.initDriver(ConfigManager.get("browser"));
        DriverFactory.getDriver().get(ConfigManager.get("base.url"));
    }
    
    @AfterMethod
    public void tearDown() {
        DriverFactory.getDriver().quit();
    }
}

// e) Reporting
public class ExtentManager {
    private static ExtentReports extent;
    private static ThreadLocal<ExtentTest> test = new ThreadLocal<>();
    
    public static void initReports() {
        // Initialize
    }
    
    public static void createTest(String testName) {
        test.set(extent.createTest(testName));
    }
    
    public static void log(Status status, String message) {
        test.get().log(status, message);
    }
}

// f) Test Listener
public class TestListener implements ITestListener {
    
    @Override
    public void onTestFailure(ITestResult result) {
        // Capture screenshot
        String screenshot = ScreenshotUtil.capture(result.getName());
        ExtentManager.attachScreenshot(screenshot);
    }
}
```

4. **Best Practices**

- Use Page Object Model
- Implement ThreadLocal for parallel execution
- Use Data Providers for data-driven testing
- Implement retry mechanism for flaky tests
- Use proper logging (Log4j)
- Implement custom exceptions
- Use constants instead of hard-coded values
- Version control (Git)
- CI/CD integration (Jenkins)

---

### Q8: How do you integrate automation tests into CI/CD pipeline?

**Answer:**

**1. Jenkins Integration**

**Jenkinsfile:**
```groovy
pipeline {
    agent any
    
    tools {
        maven 'Maven3.8'
        jdk 'JDK11'
    }
    
    parameters {
        choice(name: 'BROWSER', choices: ['chrome', 'firefox', 'edge'], 
               description: 'Select browser')
        choice(name: 'ENVIRONMENT', choices: ['qa', 'staging', 'production'], 
               description: 'Select environment')
        string(name: 'TEST_SUITE', defaultValue: 'smoke', 
               description: 'Test suite to run')
    }
    
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', 
                    url: 'https://github.com/your-repo/automation.git'
            }
        }
        
        stage('Build') {
            steps {
                sh 'mvn clean compile'
            }
        }
        
        stage('Run Tests') {
            steps {
                sh """
                    mvn test \
                    -Dbrowser=${params.BROWSER} \
                    -Denv=${params.ENVIRONMENT} \
                    -Dsuite=${params.TEST_SUITE}
                """
            }
        }
        
        stage('Generate Reports') {
            steps {
                publishHTML([
                    reportDir: 'test-output/extent-reports',
                    reportFiles: 'index.html',
                    reportName: 'Extent Report'
                ])
            }
        }
    }
    
    post {
        always {
            junit 'target/surefire-reports/*.xml'
            archiveArtifacts artifacts: 'screenshots/**', 
                             allowEmptyArchive: true
        }
        
        failure {
            emailext (
                subject: "Test Execution Failed: ${env.JOB_NAME}",
                body: "Check console output at ${env.BUILD_URL}",
                to: "team@example.com"
            )
        }
    }
}
```

**2. GitHub Actions**

```yaml
# .github/workflows/test.yml
name: Automation Tests

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]
  schedule:
    - cron: '0 2 * * *'  # Daily at 2 AM

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        browser: [chrome, firefox]
    
    steps:
    - uses: actions/checkout@v2
    
    - name: Set up JDK 11
      uses: actions/setup-java@v2
      with:
        java-version: '11'
        distribution: 'adopt'
    
    - name: Cache Maven packages
      uses: actions/cache@v2
      with:
        path: ~/.m2
        key: ${{ runner.os }}-m2-${{ hashFiles('**/pom.xml') }}
    
    - name: Run tests
      run: mvn test -Dbrowser=${{ matrix.browser }} -Denv=qa
    
    - name: Upload test results
      if: always()
      uses: actions/upload-artifact@v2
      with:
        name: test-results-${{ matrix.browser }}
        path: test-output/
    
    - name: Upload screenshots
      if: failure()
      uses: actions/upload-artifact@v2
      with:
        name: screenshots-${{ matrix.browser }}
        path: screenshots/
```

**3. Docker Integration**

```dockerfile
# Dockerfile
FROM maven:3.8-openjdk-11

# Install Chrome
RUN wget -q -O - https://dl-ssl.google.com/linux/linux_signing_key.pub | apt-key add -
RUN echo "deb http://dl.google.com/linux/chrome/deb/ stable main" >> /etc/apt/sources.list.d/google.list
RUN apt-get update && apt-get install -y google-chrome-stable

# Copy project
WORKDIR /app
COPY . .

# Run tests
CMD ["mvn", "clean", "test"]
```

```yaml
# docker-compose.yml
version: '3'
services:
  selenium-hub:
    image: selenium/hub:4.15.0
    ports:
      - "4444:4444"
  
  chrome:
    image: selenium/node-chrome:4.15.0
    depends_on:
      - selenium-hub
    environment:
      - SE_EVENT_BUS_HOST=selenium-hub
      - SE_EVENT_BUS_PUBLISH_PORT=4442
      - SE_EVENT_BUS_SUBSCRIBE_PORT=4443
  
  tests:
    build: .
    depends_on:
      - selenium-hub
    environment:
      - SELENIUM_HUB=http://selenium-hub:4444
    volumes:
      - ./test-output:/app/test-output
      - ./screenshots:/app/screenshots
```

**4. Execution Strategy**

```java
// TestNG XML for different suites
// smoke-suite.xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE suite SYSTEM "https://testng.org/testng-1.0.dtd">
<suite name="Smoke Suite" parallel="tests" thread-count="3">
    <test name="Smoke Tests">
        <groups>
            <run>
                <include name="smoke"/>
            </run>
        </groups>
        <packages>
            <package name="com.tests"/>
        </packages>
    </test>
</suite>

// Parameterized execution
mvn test -Dsuite=smoke -Dbrowser=chrome -Denv=qa
```

**5. Best Practices**

- Run smoke tests on every commit
- Run full regression nightly
- Parallel execution for faster feedback
- Docker containers for consistency
- Slack/Email notifications on failures
- Store test artifacts (screenshots, logs, reports)
- Fail-fast approach for critical failures
- Retry mechanism for flaky tests

---

**Next:** [Interview Questions & Answers - Part 2](14-interview-qa-part2.md)

