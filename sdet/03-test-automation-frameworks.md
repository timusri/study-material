# 3. Test Automation Frameworks

## 📚 Quick Summary

A test automation framework is the foundation of your testing strategy - like the blueprint of a house.

**What You'll Learn:**
- **Framework Types**: Linear, Modular, Data-Driven, Keyword-Driven, Hybrid, BDD
- **Architecture**: How to design scalable frameworks
- **Best Practices**: Configuration, test data, reporting
- **Hybrid Framework**: Combining best of all approaches

**Why This Matters:**
- Interviews ALWAYS ask: "Tell me about your framework"
- Shows your architectural thinking
- Demonstrates real-world experience

**Key Interview Question:**
"Describe your test automation framework" - This chapter prepares you perfectly!

---

## Table of Contents
- [Framework Types](#framework-types)
- [Framework Architecture](#framework-architecture)
- [Hybrid Framework Design](#hybrid-framework-design)
- [Configuration Management](#configuration-management)
- [Test Data Management](#test-data-management)
- [Reporting Mechanisms](#reporting-mechanisms)
- [Framework Best Practices](#framework-best-practices)

---

## Framework Types

### 1. Linear/Record and Playback Framework
```java
// Simple sequential script - Not recommended for production
public class LinearFramework {
    public static void main(String[] args) {
        WebDriver driver = new ChromeDriver();
        driver.get("https://example.com");
        driver.findElement(By.id("username")).sendKeys("user");
        driver.findElement(By.id("password")).sendKeys("pass");
        driver.findElement(By.id("login")).click();
        driver.quit();
    }
}
```

**Pros:** Easy to create, good for POC
**Cons:** No reusability, hard to maintain, not scalable

### 2. Modular Framework
```java
// Reusable modules/functions
public class LoginModule {
    public void login(WebDriver driver, String username, String password) {
        driver.findElement(By.id("username")).sendKeys(username);
        driver.findElement(By.id("password")).sendKeys(password);
        driver.findElement(By.id("login")).click();
    }
}

public class NavigationModule {
    public void navigateToProducts(WebDriver driver) {
        driver.findElement(By.linkText("Products")).click();
    }
}

// Test using modules
public class ModularTest {
    @Test
    public void testProductPage() {
        WebDriver driver = new ChromeDriver();
        LoginModule loginModule = new LoginModule();
        NavigationModule navModule = new NavigationModule();
        
        driver.get("https://example.com");
        loginModule.login(driver, "user", "pass");
        navModule.navigateToProducts(driver);
        
        driver.quit();
    }
}
```

**Pros:** Reusability, better maintenance
**Cons:** Test data hardcoded, limited scalability

### 3. Data-Driven Framework
```java
// Reading test data from external sources
public class DataDrivenFramework {
    
    @DataProvider(name = "loginData")
    public Object[][] getLoginData() throws IOException {
        return ExcelReader.readExcel("testdata.xlsx", "LoginSheet");
    }
    
    @Test(dataProvider = "loginData")
    public void testLogin(String username, String password, String expected) {
        WebDriver driver = new ChromeDriver();
        driver.get("https://example.com");
        
        driver.findElement(By.id("username")).sendKeys(username);
        driver.findElement(By.id("password")).sendKeys(password);
        driver.findElement(By.id("login")).click();
        
        // Validation
        String actual = driver.findElement(By.id("message")).getText();
        Assert.assertEquals(actual, expected);
        
        driver.quit();
    }
}
```

**Pros:** Separation of test data and scripts
**Cons:** Requires data management, more setup

### 4. Keyword-Driven Framework
```java
// Actions defined as keywords
public class KeywordFramework {
    private WebDriver driver;
    
    public void executeKeyword(String keyword, String locator, String value) {
        switch (keyword.toUpperCase()) {
            case "OPEN_BROWSER":
                driver = new ChromeDriver();
                break;
            case "NAVIGATE":
                driver.get(value);
                break;
            case "ENTER_TEXT":
                driver.findElement(By.id(locator)).sendKeys(value);
                break;
            case "CLICK":
                driver.findElement(By.id(locator)).click();
                break;
            case "VERIFY_TEXT":
                String actual = driver.findElement(By.id(locator)).getText();
                Assert.assertEquals(actual, value);
                break;
            case "CLOSE_BROWSER":
                driver.quit();
                break;
        }
    }
    
    public void executeTestSteps(List<TestStep> steps) {
        for (TestStep step : steps) {
            executeKeyword(step.getKeyword(), step.getLocator(), step.getValue());
        }
    }
}

class TestStep {
    private String keyword;
    private String locator;
    private String value;
    
    // Constructor and getters
}
```

**Test Data (Excel/CSV):**
```
Keyword         | Locator    | Value
OPEN_BROWSER    |            | chrome
NAVIGATE        |            | https://example.com
ENTER_TEXT      | username   | testuser
ENTER_TEXT      | password   | testpass
CLICK           | login      |
VERIFY_TEXT     | message    | Welcome
CLOSE_BROWSER   |            |
```

**Pros:** Non-programmers can create tests, highly reusable
**Cons:** Complex setup, maintenance overhead

### 5. Hybrid Framework (Combination)
```java
// Combines multiple approaches
public class HybridFramework {
    // Uses:
    // - Page Object Model (structure)
    // - Data-Driven (test data)
    // - Keyword-Driven (reusable actions)
    // - Modular (reusable components)
}
```

---

## Framework Architecture

### Complete Framework Structure

```
automation-framework/
├── src/
│   ├── main/
│   │   └── java/
│   │       ├── config/
│   │       │   ├── ConfigReader.java
│   │       │   └── DriverManager.java
│   │       ├── pages/
│   │       │   ├── BasePage.java
│   │       │   ├── LoginPage.java
│   │       │   └── HomePage.java
│   │       ├── utils/
│   │       │   ├── WaitHelper.java
│   │       │   ├── ExcelReader.java
│   │       │   ├── JsonReader.java
│   │       │   ├── ScreenshotUtil.java
│   │       │   └── ExtentReportManager.java
│   │       ├── listeners/
│   │       │   ├── TestListener.java
│   │       │   └── RetryAnalyzer.java
│   │       └── constants/
│   │           └── FrameworkConstants.java
│   └── test/
│       └── java/
│           ├── tests/
│           │   ├── BaseTest.java
│           │   ├── LoginTest.java
│           │   └── ProductTest.java
│           └── testdata/
│               └── TestDataProvider.java
├── resources/
│   ├── config.properties
│   ├── log4j2.xml
│   └── testdata/
│       ├── login_data.xlsx
│       └── test_data.json
├── test-output/
├── screenshots/
├── logs/
├── pom.xml
└── testng.xml
```

### Core Framework Components

```java
// 1. ConfigReader
package config;

import java.io.FileInputStream;
import java.io.IOException;
import java.util.Properties;

public class ConfigReader {
    private static Properties properties;
    private static final String CONFIG_PATH = "src/test/resources/config.properties";
    
    static {
        try {
            FileInputStream fis = new FileInputStream(CONFIG_PATH);
            properties = new Properties();
            properties.load(fis);
        } catch (IOException e) {
            e.printStackTrace();
            throw new RuntimeException("Failed to load config file");
        }
    }
    
    public static String getProperty(String key) {
        return properties.getProperty(key);
    }
    
    public static String getBrowser() {
        return properties.getProperty("browser", "chrome");
    }
    
    public static String getBaseUrl() {
        return properties.getProperty("base.url");
    }
    
    public static int getTimeout() {
        return Integer.parseInt(properties.getProperty("timeout", "20"));
    }
    
    public static boolean isHeadless() {
        return Boolean.parseBoolean(properties.getProperty("headless", "false"));
    }
}

// 2. DriverManager with ThreadLocal (for parallel execution)
package config;

import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.chrome.ChromeOptions;
import org.openqa.selenium.firefox.FirefoxDriver;
import io.github.bonigarcia.wdm.WebDriverManager;

public class DriverManager {
    private static ThreadLocal<WebDriver> driver = new ThreadLocal<>();
    
    public static WebDriver getDriver() {
        return driver.get();
    }
    
    public static void setDriver(WebDriver driverInstance) {
        driver.set(driverInstance);
    }
    
    public static void initDriver() {
        String browser = ConfigReader.getBrowser();
        WebDriver webDriver;
        
        switch (browser.toLowerCase()) {
            case "chrome":
                WebDriverManager.chromedriver().setup();
                ChromeOptions chromeOptions = new ChromeOptions();
                if (ConfigReader.isHeadless()) {
                    chromeOptions.addArguments("--headless");
                }
                chromeOptions.addArguments("--disable-notifications");
                chromeOptions.addArguments("--start-maximized");
                webDriver = new ChromeDriver(chromeOptions);
                break;
                
            case "firefox":
                WebDriverManager.firefoxdriver().setup();
                webDriver = new FirefoxDriver();
                break;
                
            default:
                throw new IllegalArgumentException("Browser not supported: " + browser);
        }
        
        setDriver(webDriver);
        getDriver().manage().window().maximize();
        getDriver().manage().deleteAllCookies();
    }
    
    public static void quitDriver() {
        if (getDriver() != null) {
            getDriver().quit();
            driver.remove();
        }
    }
}

// 3. FrameworkConstants
package constants;

public class FrameworkConstants {
    
    // Timeouts
    public static final int EXPLICIT_WAIT = 20;
    public static final int IMPLICIT_WAIT = 10;
    public static final int PAGE_LOAD_TIMEOUT = 30;
    
    // Paths
    public static final String PROJECT_PATH = System.getProperty("user.dir");
    public static final String TEST_DATA_PATH = PROJECT_PATH + "/src/test/resources/testdata/";
    public static final String SCREENSHOT_PATH = PROJECT_PATH + "/screenshots/";
    public static final String REPORT_PATH = PROJECT_PATH + "/test-output/extent-reports/";
    public static final String LOG_PATH = PROJECT_PATH + "/logs/";
    
    // Messages
    public static final String LOGIN_SUCCESS = "Login successful";
    public static final String LOGIN_FAILED = "Invalid credentials";
    
    // URLs
    public static final String LOGIN_URL = "/login";
    public static final String HOME_URL = "/home";
    public static final String PRODUCTS_URL = "/products";
    
    private FrameworkConstants() {
        // Private constructor to prevent instantiation
    }
}

// 4. WaitHelper
package utils;

import org.openqa.selenium.*;
import org.openqa.selenium.support.ui.*;
import java.time.Duration;

public class WaitHelper {
    private WebDriver driver;
    private WebDriverWait wait;
    
    public WaitHelper(WebDriver driver) {
        this.driver = driver;
        this.wait = new WebDriverWait(driver, 
            Duration.ofSeconds(FrameworkConstants.EXPLICIT_WAIT));
    }
    
    public WebElement waitForElementVisible(By locator) {
        return wait.until(ExpectedConditions.visibilityOfElementLocated(locator));
    }
    
    public WebElement waitForElementClickable(By locator) {
        return wait.until(ExpectedConditions.elementToBeClickable(locator));
    }
    
    public boolean waitForElementInvisible(By locator) {
        return wait.until(ExpectedConditions.invisibilityOfElementLocated(locator));
    }
    
    public void waitForPageLoad() {
        wait.until(driver -> 
            ((JavascriptExecutor) driver).executeScript("return document.readyState")
                .equals("complete"));
    }
    
    public Alert waitForAlert() {
        return wait.until(ExpectedConditions.alertIsPresent());
    }
    
    public boolean waitForTextPresent(By locator, String text) {
        return wait.until(ExpectedConditions.textToBePresentInElementLocated(
            locator, text));
    }
}

// 5. ScreenshotUtil
package utils;

import org.apache.commons.io.FileUtils;
import org.openqa.selenium.*;
import java.io.File;
import java.io.IOException;
import java.text.SimpleDateFormat;
import java.util.Date;

public class ScreenshotUtil {
    
    public static String captureScreenshot(WebDriver driver, String testName) {
        String timestamp = new SimpleDateFormat("yyyyMMdd_HHmmss").format(new Date());
        String fileName = testName + "_" + timestamp + ".png";
        String filePath = FrameworkConstants.SCREENSHOT_PATH + fileName;
        
        try {
            TakesScreenshot screenshot = (TakesScreenshot) driver;
            File source = screenshot.getScreenshotAs(OutputType.FILE);
            File destination = new File(filePath);
            FileUtils.copyFile(source, destination);
            return filePath;
        } catch (IOException e) {
            e.printStackTrace();
            return null;
        }
    }
    
    public static String captureElementScreenshot(WebElement element, String elementName) {
        String timestamp = new SimpleDateFormat("yyyyMMdd_HHmmss").format(new Date());
        String fileName = elementName + "_" + timestamp + ".png";
        String filePath = FrameworkConstants.SCREENSHOT_PATH + fileName;
        
        try {
            File source = element.getScreenshotAs(OutputType.FILE);
            File destination = new File(filePath);
            FileUtils.copyFile(source, destination);
            return filePath;
        } catch (IOException e) {
            e.printStackTrace();
            return null;
        }
    }
}

// 6. BaseTest
package tests;

import config.DriverManager;
import config.ConfigReader;
import org.testng.annotations.*;
import utils.ExtentReportManager;

public class BaseTest {
    
    @BeforeMethod(alwaysRun = true)
    public void setUp() {
        DriverManager.initDriver();
        DriverManager.getDriver().get(ConfigReader.getBaseUrl());
    }
    
    @AfterMethod(alwaysRun = true)
    public void tearDown() {
        DriverManager.quitDriver();
    }
    
    @BeforeSuite
    public void beforeSuite() {
        ExtentReportManager.initReports();
    }
    
    @AfterSuite
    public void afterSuite() {
        ExtentReportManager.flushReports();
    }
}

// 7. TestListener
package listeners;

import config.DriverManager;
import org.testng.*;
import utils.ScreenshotUtil;
import utils.ExtentReportManager;

public class TestListener implements ITestListener {
    
    @Override
    public void onTestStart(ITestResult result) {
        ExtentReportManager.createTest(result.getMethod().getMethodName());
    }
    
    @Override
    public void onTestSuccess(ITestResult result) {
        ExtentReportManager.logPass("Test Passed: " + 
            result.getMethod().getMethodName());
    }
    
    @Override
    public void onTestFailure(ITestResult result) {
        String screenshotPath = ScreenshotUtil.captureScreenshot(
            DriverManager.getDriver(), 
            result.getMethod().getMethodName()
        );
        
        ExtentReportManager.logFail("Test Failed: " + 
            result.getMethod().getMethodName());
        ExtentReportManager.attachScreenshot(screenshotPath);
    }
    
    @Override
    public void onTestSkipped(ITestResult result) {
        ExtentReportManager.logSkip("Test Skipped: " + 
            result.getMethod().getMethodName());
    }
}

// 8. RetryAnalyzer (for flaky tests)
package listeners;

import org.testng.IRetryAnalyzer;
import org.testng.ITestResult;

public class RetryAnalyzer implements IRetryAnalyzer {
    private int retryCount = 0;
    private static final int MAX_RETRY = 2;
    
    @Override
    public boolean retry(ITestResult result) {
        if (retryCount < MAX_RETRY) {
            retryCount++;
            return true;
        }
        return false;
    }
}
```

---

## Hybrid Framework Design

### Complete Working Example

```java
// BasePage with common functionalities
package pages;

import config.DriverManager;
import org.openqa.selenium.*;
import org.openqa.selenium.support.PageFactory;
import org.openqa.selenium.support.ui.*;
import java.time.Duration;

public class BasePage {
    protected WebDriver driver;
    protected WebDriverWait wait;
    protected JavascriptExecutor js;
    
    public BasePage() {
        this.driver = DriverManager.getDriver();
        this.wait = new WebDriverWait(driver, Duration.ofSeconds(20));
        this.js = (JavascriptExecutor) driver;
        PageFactory.initElements(driver, this);
    }
    
    // Common actions
    protected void click(WebElement element) {
        wait.until(ExpectedConditions.elementToBeClickable(element));
        element.click();
    }
    
    protected void type(WebElement element, String text) {
        wait.until(ExpectedConditions.visibilityOf(element));
        element.clear();
        element.sendKeys(text);
    }
    
    protected String getText(WebElement element) {
        wait.until(ExpectedConditions.visibilityOf(element));
        return element.getText();
    }
    
    protected void clickUsingJS(WebElement element) {
        js.executeScript("arguments[0].click();", element);
    }
    
    protected void scrollToElement(WebElement element) {
        js.executeScript("arguments[0].scrollIntoView(true);", element);
    }
    
    protected boolean isDisplayed(WebElement element) {
        try {
            return element.isDisplayed();
        } catch (NoSuchElementException e) {
            return false;
        }
    }
    
    protected void selectDropdownByText(WebElement element, String text) {
        Select select = new Select(element);
        select.selectByVisibleText(text);
    }
    
    protected void waitForPageLoad() {
        wait.until(driver -> 
            js.executeScript("return document.readyState").equals("complete"));
    }
}

// LoginPage
package pages;

import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.FindBy;

public class LoginPage extends BasePage {
    
    @FindBy(id = "username")
    private WebElement usernameInput;
    
    @FindBy(id = "password")
    private WebElement passwordInput;
    
    @FindBy(xpath = "//button[@type='submit']")
    private WebElement loginButton;
    
    @FindBy(css = ".error-message")
    private WebElement errorMessage;
    
    @FindBy(linkText = "Forgot Password?")
    private WebElement forgotPasswordLink;
    
    public LoginPage enterUsername(String username) {
        type(usernameInput, username);
        return this;
    }
    
    public LoginPage enterPassword(String password) {
        type(passwordInput, password);
        return this;
    }
    
    public HomePage clickLogin() {
        click(loginButton);
        return new HomePage();
    }
    
    public String getErrorMessage() {
        return getText(errorMessage);
    }
    
    public boolean isErrorDisplayed() {
        return isDisplayed(errorMessage);
    }
    
    public HomePage performLogin(String username, String password) {
        enterUsername(username);
        enterPassword(password);
        return clickLogin();
    }
}

// LoginTest with Data Provider
package tests;

import config.DriverManager;
import org.testng.Assert;
import org.testng.annotations.*;
import pages.LoginPage;
import pages.HomePage;

public class LoginTest extends BaseTest {
    private LoginPage loginPage;
    
    @BeforeMethod
    public void setupTest() {
        loginPage = new LoginPage();
    }
    
    @Test(priority = 1, description = "Valid login test")
    public void testValidLogin() {
        HomePage homePage = loginPage.performLogin(
            ConfigReader.getProperty("valid.username"),
            ConfigReader.getProperty("valid.password")
        );
        
        Assert.assertTrue(homePage.isWelcomeMessageDisplayed(),
            "Welcome message not displayed");
    }
    
    @Test(priority = 2, description = "Invalid login test", 
          dataProvider = "invalidLoginData")
    public void testInvalidLogin(String username, String password, 
                                  String expectedError) {
        loginPage.enterUsername(username)
                 .enterPassword(password)
                 .clickLogin();
        
        Assert.assertTrue(loginPage.isErrorDisplayed(),
            "Error message not displayed");
        Assert.assertEquals(loginPage.getErrorMessage(), expectedError,
            "Error message mismatch");
    }
    
    @DataProvider(name = "invalidLoginData")
    public Object[][] getInvalidLoginData() {
        return new Object[][] {
            {"invalid@email.com", "wrongpass", "Invalid credentials"},
            {"", "password", "Username is required"},
            {"user@email.com", "", "Password is required"}
        };
    }
    
    @Test(priority = 3, description = "Login with Excel data",
          dataProvider = "excelLoginData", 
          dataProviderClass = TestDataProvider.class)
    public void testLoginWithExcelData(String username, String password, 
                                       String expectedResult) {
        if (expectedResult.equals("Pass")) {
            HomePage homePage = loginPage.performLogin(username, password);
            Assert.assertTrue(homePage.isWelcomeMessageDisplayed());
        } else {
            loginPage.performLogin(username, password);
            Assert.assertTrue(loginPage.isErrorDisplayed());
        }
    }
}
```

---

## Configuration Management

### config.properties
```properties
# Browser Configuration
browser=chrome
headless=false

# Environment URLs
base.url=https://example.com
staging.url=https://staging.example.com
production.url=https://www.example.com

# Timeouts
implicit.wait=10
explicit.wait=20
page.load.timeout=30

# Credentials
valid.username=testuser@example.com
valid.password=Test@123

# Database
db.url=jdbc:mysql://localhost:3306/testdb
db.username=root
db.password=password

# API
api.base.url=https://api.example.com
api.key=your-api-key

# Test Execution
retry.count=2
thread.count=3
screenshot.on.failure=true

# Reporting
report.path=test-output/extent-reports/
screenshot.path=screenshots/
log.path=logs/
```

### Environment-specific Configuration

```java
package config;

public enum Environment {
    DEV("https://dev.example.com"),
    STAGING("https://staging.example.com"),
    PRODUCTION("https://www.example.com");
    
    private String url;
    
    Environment(String url) {
        this.url = url;
    }
    
    public String getUrl() {
        return url;
    }
}

public class EnvironmentManager {
    private static Environment currentEnvironment;
    
    static {
        String env = System.getProperty("env", "DEV");
        currentEnvironment = Environment.valueOf(env.toUpperCase());
    }
    
    public static String getBaseUrl() {
        return currentEnvironment.getUrl();
    }
    
    public static Environment getCurrentEnvironment() {
        return currentEnvironment;
    }
}

// Run tests with: mvn test -Denv=staging
```

---

## Test Data Management

### 1. Excel Reader

```java
package utils;

import org.apache.poi.ss.usermodel.*;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;
import java.io.FileInputStream;
import java.io.IOException;
import java.util.ArrayList;
import java.util.List;

public class ExcelReader {
    
    public static Object[][] readExcel(String filePath, String sheetName) {
        List<List<String>> data = new ArrayList<>();
        
        try (FileInputStream fis = new FileInputStream(filePath);
             Workbook workbook = new XSSFWorkbook(fis)) {
            
            Sheet sheet = workbook.getSheet(sheetName);
            int rowCount = sheet.getLastRowNum();
            
            for (int i = 1; i <= rowCount; i++) {  // Skip header row
                Row row = sheet.getRow(i);
                int cellCount = row.getLastCellNum();
                
                List<String> rowData = new ArrayList<>();
                for (int j = 0; j < cellCount; j++) {
                    Cell cell = row.getCell(j);
                    rowData.add(getCellValue(cell));
                }
                data.add(rowData);
            }
            
        } catch (IOException e) {
            e.printStackTrace();
        }
        
        // Convert List to 2D array
        Object[][] result = new Object[data.size()][];
        for (int i = 0; i < data.size(); i++) {
            result[i] = data.get(i).toArray();
        }
        
        return result;
    }
    
    private static String getCellValue(Cell cell) {
        if (cell == null) {
            return "";
        }
        
        switch (cell.getCellType()) {
            case STRING:
                return cell.getStringCellValue();
            case NUMERIC:
                if (DateUtil.isCellDateFormatted(cell)) {
                    return cell.getDateCellValue().toString();
                }
                return String.valueOf((int) cell.getNumericCellValue());
            case BOOLEAN:
                return String.valueOf(cell.getBooleanCellValue());
            case FORMULA:
                return cell.getCellFormula();
            default:
                return "";
        }
    }
    
    public static void writeExcel(String filePath, String sheetName, 
                                   int row, int col, String value) {
        try (FileInputStream fis = new FileInputStream(filePath);
             Workbook workbook = new XSSFWorkbook(fis)) {
            
            Sheet sheet = workbook.getSheet(sheetName);
            Row sheetRow = sheet.getRow(row);
            if (sheetRow == null) {
                sheetRow = sheet.createRow(row);
            }
            
            Cell cell = sheetRow.getCell(col);
            if (cell == null) {
                cell = sheetRow.createCell(col);
            }
            
            cell.setCellValue(value);
            
            // Write back to file
            try (FileOutputStream fos = new FileOutputStream(filePath)) {
                workbook.write(fos);
            }
            
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

### 2. JSON Reader

```java
package utils;

import com.google.gson.Gson;
import com.google.gson.JsonObject;
import java.io.FileReader;
import java.io.IOException;

public class JsonReader {
    
    public static JsonObject readJson(String filePath) {
        try (FileReader reader = new FileReader(filePath)) {
            Gson gson = new Gson();
            return gson.fromJson(reader, JsonObject.class);
        } catch (IOException e) {
            e.printStackTrace();
            return null;
        }
    }
    
    public static <T> T readJsonToObject(String filePath, Class<T> classType) {
        try (FileReader reader = new FileReader(filePath)) {
            Gson gson = new Gson();
            return gson.fromJson(reader, classType);
        } catch (IOException e) {
            e.printStackTrace();
            return null;
        }
    }
}

// User POJO
class User {
    private String username;
    private String password;
    private String email;
    private String role;
    
    // Getters and setters
}

// Usage
User user = JsonReader.readJsonToObject("testdata/user.json", User.class);
```

### 3. Test Data Provider

```java
package testdata;

import org.testng.annotations.DataProvider;
import utils.ExcelReader;
import constants.FrameworkConstants;

public class TestDataProvider {
    
    @DataProvider(name = "excelLoginData")
    public static Object[][] getLoginDataFromExcel() {
        String filePath = FrameworkConstants.TEST_DATA_PATH + "login_data.xlsx";
        return ExcelReader.readExcel(filePath, "LoginData");
    }
    
    @DataProvider(name = "jsonUserData")
    public static Object[][] getUserDataFromJson() {
        // Read JSON and convert to Object[][]
        return new Object[][] {
            // Data from JSON
        };
    }
    
    @DataProvider(name = "inlineData", parallel = true)
    public static Object[][] getInlineData() {
        return new Object[][] {
            {"user1@example.com", "Pass@123"},
            {"user2@example.com", "Test@456"},
            {"user3@example.com", "Demo@789"}
        };
    }
}
```

---

## Reporting Mechanisms

### Extent Reports Integration

```java
package utils;

import com.aventstack.extentreports.*;
import com.aventstack.extentreports.reporter.ExtentSparkReporter;
import com.aventstack.extentreports.reporter.configuration.Theme;
import constants.FrameworkConstants;

public class ExtentReportManager {
    private static ExtentReports extent;
    private static ThreadLocal<ExtentTest> test = new ThreadLocal<>();
    
    public static void initReports() {
        if (extent == null) {
            String reportPath = FrameworkConstants.REPORT_PATH + 
                "TestReport_" + System.currentTimeMillis() + ".html";
            
            ExtentSparkReporter sparkReporter = new ExtentSparkReporter(reportPath);
            sparkReporter.config().setTheme(Theme.STANDARD);
            sparkReporter.config().setDocumentTitle("Automation Test Report");
            sparkReporter.config().setReportName("Test Execution Report");
            sparkReporter.config().setTimeStampFormat("dd-MM-yyyy HH:mm:ss");
            
            extent = new ExtentReports();
            extent.attachReporter(sparkReporter);
            extent.setSystemInfo("OS", System.getProperty("os.name"));
            extent.setSystemInfo("Java Version", System.getProperty("java.version"));
            extent.setSystemInfo("Browser", ConfigReader.getBrowser());
            extent.setSystemInfo("Environment", EnvironmentManager.getCurrentEnvironment().name());
        }
    }
    
    public static void createTest(String testName) {
        ExtentTest extentTest = extent.createTest(testName);
        test.set(extentTest);
    }
    
    public static ExtentTest getTest() {
        return test.get();
    }
    
    public static void logPass(String message) {
        getTest().log(Status.PASS, message);
    }
    
    public static void logFail(String message) {
        getTest().log(Status.FAIL, message);
    }
    
    public static void logSkip(String message) {
        getTest().log(Status.SKIP, message);
    }
    
    public static void logInfo(String message) {
        getTest().log(Status.INFO, message);
    }
    
    public static void attachScreenshot(String screenshotPath) {
        try {
            getTest().addScreenCaptureFromPath(screenshotPath);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
    
    public static void flushReports() {
        if (extent != null) {
            extent.flush();
        }
    }
}
```

---

## Framework Best Practices

### 1. Exception Handling

```java
package exceptions;

public class FrameworkException extends RuntimeException {
    public FrameworkException(String message) {
        super(message);
    }
    
    public FrameworkException(String message, Throwable cause) {
        super(message, cause);
    }
}

public class ElementNotFoundException extends FrameworkException {
    public ElementNotFoundException(String elementName) {
        super("Element not found: " + elementName);
    }
}

public class PageNotLoadedException extends FrameworkException {
    public PageNotLoadedException(String pageName) {
        super("Page failed to load: " + pageName);
    }
}
```

### 2. Logging

```xml
<!-- log4j2.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="WARN">
    <Properties>
        <Property name="logPath">logs</Property>
    </Properties>
    
    <Appenders>
        <Console name="Console" target="SYSTEM_OUT">
            <PatternLayout pattern="%d{HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n"/>
        </Console>
        
        <RollingFile name="RollingFile" fileName="${logPath}/automation.log"
                     filePattern="${logPath}/automation-%d{yyyy-MM-dd}.log">
            <PatternLayout pattern="%d{yyyy-MM-dd HH:mm:ss} [%t] %-5level %logger{36} - %msg%n"/>
            <Policies>
                <TimeBasedTriggeringPolicy interval="1"/>
            </Policies>
        </RollingFile>
    </Appenders>
    
    <Loggers>
        <Root level="info">
            <AppenderRef ref="Console"/>
            <AppenderRef ref="RollingFile"/>
        </Root>
    </Loggers>
</Configuration>
```

```java
// Usage in code
import org.apache.logging.log4j.LogManager;
import org.apache.logging.log4j.Logger;

public class LoginPage extends BasePage {
    private static final Logger logger = LogManager.getLogger(LoginPage.class);
    
    public HomePage performLogin(String username, String password) {
        logger.info("Attempting to login with username: " + username);
        enterUsername(username);
        enterPassword(password);
        clickLogin();
        logger.info("Login successful");
        return new HomePage();
    }
}
```

### 3. Framework Utilities

```java
// DateUtil
package utils;

import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;

public class DateUtil {
    
    public static String getCurrentDateTime(String pattern) {
        DateTimeFormatter formatter = DateTimeFormatter.ofPattern(pattern);
        return LocalDateTime.now().format(formatter);
    }
    
    public static String getTimestamp() {
        return getCurrentDateTime("yyyyMMdd_HHmmss");
    }
}

// RandomDataGenerator
package utils;

import com.github.javafaker.Faker;

public class RandomDataGenerator {
    private static Faker faker = new Faker();
    
    public static String getRandomEmail() {
        return faker.internet().emailAddress();
    }
    
    public static String getRandomName() {
        return faker.name().fullName();
    }
    
    public static String getRandomPhone() {
        return faker.phoneNumber().cellPhone();
    }
    
    public static String getRandomAddress() {
        return faker.address().fullAddress();
    }
}
```

---

**Next:** [Testing Tools & Technologies](04-testing-tools-technologies.md)

