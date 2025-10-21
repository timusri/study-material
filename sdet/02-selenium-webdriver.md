# 2. Selenium WebDriver (Expert Level)

## Table of Contents
- [Selenium Architecture](#selenium-architecture)
- [WebDriver Wait Strategies](#webdriver-wait-strategies)
- [Handling Dynamic Elements](#handling-dynamic-elements)
- [Cross-Browser Testing](#cross-browser-testing)
- [Selenium Grid Setup](#selenium-grid-setup)
- [Page Object Model (POM)](#page-object-model-pom)
- [Advanced Element Interactions](#advanced-element-interactions)
- [JavaScript Executor](#javascript-executor)

---

## Selenium Architecture

### Overview
```
Client (Java/Python) --> WebDriver API --> Browser Driver --> Browser
```

**Components:**
1. **Selenium WebDriver**: Language bindings (Java, Python, C#)
2. **Browser Drivers**: ChromeDriver, GeckoDriver, EdgeDriver
3. **Browsers**: Chrome, Firefox, Safari, Edge
4. **JSON Wire Protocol / W3C WebDriver Protocol**: Communication protocol

### WebDriver Initialization

```java
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.chrome.ChromeOptions;
import org.openqa.selenium.firefox.FirefoxDriver;
import org.openqa.selenium.firefox.FirefoxOptions;
import io.github.bonigarcia.wdm.WebDriverManager;

public class WebDriverSetup {
    
    // Method 1: Using WebDriverManager (Recommended)
    public WebDriver setupChromeWithWDM() {
        WebDriverManager.chromedriver().setup();
        return new ChromeDriver();
    }
    
    // Method 2: Manual driver setup
    public WebDriver setupChromeManual() {
        System.setProperty("webdriver.chrome.driver", 
            "/path/to/chromedriver");
        return new ChromeDriver();
    }
    
    // Chrome with options
    public WebDriver setupChromeWithOptions() {
        WebDriverManager.chromedriver().setup();
        
        ChromeOptions options = new ChromeOptions();
        options.addArguments("--start-maximized");
        options.addArguments("--incognito");
        options.addArguments("--disable-notifications");
        options.addArguments("--disable-popup-blocking");
        options.addArguments("--headless");  // For headless mode
        
        // Set download directory
        Map<String, Object> prefs = new HashMap<>();
        prefs.put("download.default_directory", "/path/to/download");
        options.setExperimentalOption("prefs", prefs);
        
        return new ChromeDriver(options);
    }
    
    // Firefox setup
    public WebDriver setupFirefox() {
        WebDriverManager.firefoxdriver().setup();
        
        FirefoxOptions options = new FirefoxOptions();
        options.addArguments("--headless");
        options.setAcceptInsecureCerts(true);
        
        return new FirefoxDriver(options);
    }
    
    // Remote WebDriver (for Grid)
    public WebDriver setupRemoteDriver() throws MalformedURLException {
        ChromeOptions options = new ChromeOptions();
        return new RemoteWebDriver(
            new URL("http://localhost:4444/wd/hub"), options);
    }
}
```

---

## WebDriver Wait Strategies

### Types of Waits

```java
import org.openqa.selenium.*;
import org.openqa.selenium.support.ui.*;
import java.time.Duration;

public class WaitStrategies {
    private WebDriver driver;
    
    public WaitStrategies(WebDriver driver) {
        this.driver = driver;
    }
    
    // 1. Implicit Wait (Not recommended for mixing with explicit waits)
    public void setImplicitWait() {
        driver.manage().timeouts().implicitlyWait(Duration.ofSeconds(10));
    }
    
    // 2. Explicit Wait
    public void explicitWaitExample() {
        WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(20));
        
        // Wait for element to be visible
        WebElement element = wait.until(
            ExpectedConditions.visibilityOfElementLocated(
                By.id("username")));
        
        // Wait for element to be clickable
        WebElement button = wait.until(
            ExpectedConditions.elementToBeClickable(
                By.xpath("//button[@type='submit']")));
        
        // Wait for element to be present
        WebElement ele = wait.until(
            ExpectedConditions.presenceOfElementLocated(
                By.cssSelector(".error-message")));
    }
    
    // 3. Fluent Wait
    public void fluentWaitExample() {
        Wait<WebDriver> wait = new FluentWait<>(driver)
            .withTimeout(Duration.ofSeconds(30))
            .pollingEvery(Duration.ofMillis(500))
            .ignoring(NoSuchElementException.class)
            .ignoring(StaleElementReferenceException.class);
        
        WebElement element = wait.until(driver -> {
            return driver.findElement(By.id("dynamicElement"));
        });
    }
    
    // 4. Custom Wait Conditions
    public void customWaitCondition() {
        WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(20));
        
        // Wait for URL to contain specific text
        wait.until(ExpectedConditions.urlContains("dashboard"));
        
        // Wait for title
        wait.until(ExpectedConditions.titleIs("Home Page"));
        
        // Wait for frame and switch to it
        wait.until(ExpectedConditions.frameToBeAvailableAndSwitchToIt("frameId"));
        
        // Wait for alert
        Alert alert = wait.until(ExpectedConditions.alertIsPresent());
        
        // Wait for text to be present
        wait.until(ExpectedConditions.textToBePresentInElementLocated(
            By.id("message"), "Success"));
    }
    
    // 5. Custom Expected Condition
    public void customExpectedCondition() {
        WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(20));
        
        // Custom condition: Wait for element attribute value
        ExpectedCondition<Boolean> attributeValue = new ExpectedCondition<Boolean>() {
            @Override
            public Boolean apply(WebDriver driver) {
                WebElement element = driver.findElement(By.id("status"));
                String status = element.getAttribute("data-status");
                return status.equals("completed");
            }
        };
        
        wait.until(attributeValue);
    }
    
    // 6. Wait for AJAX calls to complete
    public void waitForAjax() {
        WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(20));
        
        ExpectedCondition<Boolean> ajaxComplete = new ExpectedCondition<Boolean>() {
            @Override
            public Boolean apply(WebDriver driver) {
                JavascriptExecutor js = (JavascriptExecutor) driver;
                return (Boolean) js.executeScript(
                    "return jQuery.active == 0");
            }
        };
        
        wait.until(ajaxComplete);
    }
    
    // 7. Wait for page load
    public void waitForPageLoad() {
        WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(30));
        
        ExpectedCondition<Boolean> pageLoaded = new ExpectedCondition<Boolean>() {
            @Override
            public Boolean apply(WebDriver driver) {
                JavascriptExecutor js = (JavascriptExecutor) driver;
                return js.executeScript("return document.readyState")
                    .equals("complete");
            }
        };
        
        wait.until(pageLoaded);
    }
}

// Reusable Wait Utility Class
class WaitUtil {
    private WebDriver driver;
    private WebDriverWait wait;
    
    public WaitUtil(WebDriver driver, int timeoutInSeconds) {
        this.driver = driver;
        this.wait = new WebDriverWait(driver, Duration.ofSeconds(timeoutInSeconds));
    }
    
    public WebElement waitForVisibility(By locator) {
        return wait.until(ExpectedConditions.visibilityOfElementLocated(locator));
    }
    
    public WebElement waitForClickable(By locator) {
        return wait.until(ExpectedConditions.elementToBeClickable(locator));
    }
    
    public void waitForInvisibility(By locator) {
        wait.until(ExpectedConditions.invisibilityOfElementLocated(locator));
    }
    
    public void waitForTextPresent(By locator, String text) {
        wait.until(ExpectedConditions.textToBePresentInElementLocated(
            locator, text));
    }
    
    public boolean waitForElementStale(WebElement element) {
        return wait.until(ExpectedConditions.stalenessOf(element));
    }
}
```

---

## Handling Dynamic Elements

### Common Strategies

```java
public class DynamicElementHandler {
    private WebDriver driver;
    private WaitUtil waitUtil;
    
    public DynamicElementHandler(WebDriver driver) {
        this.driver = driver;
        this.waitUtil = new WaitUtil(driver, 20);
    }
    
    // 1. Handle StaleElementReferenceException
    public void handleStaleElement() {
        int attempts = 0;
        while (attempts < 3) {
            try {
                WebElement element = driver.findElement(By.id("dynamicId"));
                element.click();
                break;
            } catch (StaleElementReferenceException e) {
                attempts++;
                System.out.println("Stale element, retrying... " + attempts);
            }
        }
    }
    
    // Better approach with retry mechanism
    public WebElement findElementWithRetry(By locator, int maxRetries) {
        WebElement element = null;
        for (int i = 0; i < maxRetries; i++) {
            try {
                element = driver.findElement(locator);
                return element;
            } catch (StaleElementReferenceException e) {
                System.out.println("Stale element, attempt " + (i + 1));
            }
        }
        throw new RuntimeException("Failed to find element after " + 
            maxRetries + " attempts");
    }
    
    // 2. Dynamic XPath strategies
    public void dynamicXPathExamples() {
        // Using contains()
        driver.findElement(By.xpath("//div[contains(@class, 'dynamic-')]"));
        
        // Using starts-with()
        driver.findElement(By.xpath("//input[starts-with(@id, 'user_')]"));
        
        // Using ends-with() - XPath 2.0 (not supported in Selenium)
        // Alternative: Using substring
        driver.findElement(By.xpath(
            "//button[substring(@id, string-length(@id) - 2) = 'btn']"));
        
        // Using text()
        driver.findElement(By.xpath("//span[text()='Submit']"));
        
        // Using normalize-space() for text with spaces
        driver.findElement(By.xpath(
            "//span[normalize-space()='Submit Form']"));
        
        // Dynamic value substitution
        String username = "john";
        driver.findElement(By.xpath(
            String.format("//td[text()='%s']/../td[2]", username)));
    }
    
    // 3. Handle dynamic IDs
    public void handleDynamicIds() {
        // CSS Selector with partial match
        driver.findElement(By.cssSelector("[id^='user_']"));  // Starts with
        driver.findElement(By.cssSelector("[id$='_name']"));  // Ends with
        driver.findElement(By.cssSelector("[id*='user']"));   // Contains
        
        // XPath with multiple conditions
        driver.findElement(By.xpath(
            "//input[@type='text' and @name='username']"));
    }
    
    // 4. Handle elements that appear/disappear
    public void waitForElementToDisappear(By locator) {
        WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
        wait.until(ExpectedConditions.invisibilityOfElementLocated(locator));
    }
    
    public void waitForLoadingSpinner() {
        By spinner = By.cssSelector(".loading-spinner");
        try {
            WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(2));
            wait.until(ExpectedConditions.visibilityOfElementLocated(spinner));
            wait.until(ExpectedConditions.invisibilityOfElementLocated(spinner));
        } catch (TimeoutException e) {
            // Spinner didn't appear, continue
        }
    }
    
    // 5. Handle shadow DOM
    public void handleShadowDOM() {
        JavascriptExecutor js = (JavascriptExecutor) driver;
        
        WebElement shadowHost = driver.findElement(By.id("shadow-host"));
        WebElement shadowRoot = (WebElement) js.executeScript(
            "return arguments[0].shadowRoot", shadowHost);
        WebElement shadowElement = shadowRoot.findElement(
            By.cssSelector(".shadow-element"));
    }
    
    // 6. Handle elements in viewport
    public void scrollIntoView(By locator) {
        WebElement element = driver.findElement(locator);
        JavascriptExecutor js = (JavascriptExecutor) driver;
        js.executeScript("arguments[0].scrollIntoView(true);", element);
    }
    
    // 7. Wait for element to be stable (not moving)
    public WebElement waitForElementStable(By locator) {
        WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(20));
        
        return wait.until(driver -> {
            WebElement element = driver.findElement(locator);
            Point location1 = element.getLocation();
            
            try {
                Thread.sleep(500);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            
            Point location2 = element.getLocation();
            if (location1.equals(location2)) {
                return element;
            }
            return null;
        });
    }
}
```

---

## Cross-Browser Testing

### Browser Configuration Manager

```java
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.chrome.ChromeOptions;
import org.openqa.selenium.firefox.FirefoxDriver;
import org.openqa.selenium.firefox.FirefoxOptions;
import org.openqa.selenium.edge.EdgeDriver;
import org.openqa.selenium.edge.EdgeOptions;
import org.openqa.selenium.safari.SafariDriver;
import io.github.bonigarcia.wdm.WebDriverManager;

public class BrowserManager {
    
    public static WebDriver getDriver(String browserName) {
        WebDriver driver;
        
        switch (browserName.toLowerCase()) {
            case "chrome":
                driver = getChromeDriver();
                break;
            case "firefox":
                driver = getFirefoxDriver();
                break;
            case "edge":
                driver = getEdgeDriver();
                break;
            case "safari":
                driver = getSafariDriver();
                break;
            case "chrome-headless":
                driver = getChromeHeadless();
                break;
            default:
                throw new IllegalArgumentException("Browser not supported: " + 
                    browserName);
        }
        
        configureBrowser(driver);
        return driver;
    }
    
    private static WebDriver getChromeDriver() {
        WebDriverManager.chromedriver().setup();
        ChromeOptions options = new ChromeOptions();
        options.addArguments("--disable-notifications");
        options.addArguments("--disable-popup-blocking");
        options.addArguments("--disable-dev-shm-usage");
        options.addArguments("--no-sandbox");
        return new ChromeDriver(options);
    }
    
    private static WebDriver getFirefoxDriver() {
        WebDriverManager.firefoxdriver().setup();
        FirefoxOptions options = new FirefoxOptions();
        options.setAcceptInsecureCerts(true);
        return new FirefoxDriver(options);
    }
    
    private static WebDriver getEdgeDriver() {
        WebDriverManager.edgedriver().setup();
        EdgeOptions options = new EdgeOptions();
        return new EdgeDriver(options);
    }
    
    private static WebDriver getSafariDriver() {
        return new SafariDriver();
    }
    
    private static WebDriver getChromeHeadless() {
        WebDriverManager.chromedriver().setup();
        ChromeOptions options = new ChromeOptions();
        options.addArguments("--headless");
        options.addArguments("--window-size=1920,1080");
        options.addArguments("--disable-gpu");
        return new ChromeDriver(options);
    }
    
    private static void configureBrowser(WebDriver driver) {
        driver.manage().window().maximize();
        driver.manage().timeouts().pageLoadTimeout(Duration.ofSeconds(30));
        driver.manage().timeouts().scriptTimeout(Duration.ofSeconds(30));
        driver.manage().deleteAllCookies();
    }
}

// TestNG example for cross-browser testing
public class CrossBrowserTest {
    WebDriver driver;
    
    @Parameters("browser")
    @BeforeMethod
    public void setup(String browser) {
        driver = BrowserManager.getDriver(browser);
    }
    
    @Test
    public void loginTest() {
        driver.get("https://example.com/login");
        // Test implementation
    }
    
    @AfterMethod
    public void teardown() {
        if (driver != null) {
            driver.quit();
        }
    }
}
```

**testng.xml for cross-browser testing:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE suite SYSTEM "https://testng.org/testng-1.0.dtd">
<suite name="Cross Browser Suite" parallel="tests" thread-count="3">
    
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
    
    <test name="Edge Tests">
        <parameter name="browser" value="edge"/>
        <classes>
            <class name="com.tests.CrossBrowserTest"/>
        </classes>
    </test>
    
</suite>
```

---

## Selenium Grid Setup

### Grid 4 Architecture
```
Hub (Router + Distributor + Session Map + Event Bus)
  |
  +-- Node 1 (Chrome, Firefox)
  +-- Node 2 (Chrome, Edge)
  +-- Node 3 (Safari)
```

### Starting Selenium Grid

```bash
# Download Selenium Server (Grid 4)
wget https://github.com/SeleniumHQ/selenium/releases/download/selenium-4.15.0/selenium-server-4.15.0.jar

# Start in Standalone mode
java -jar selenium-server-4.15.0.jar standalone

# Start Hub
java -jar selenium-server-4.15.0.jar hub

# Start Node
java -jar selenium-server-4.15.0.jar node --detect-drivers true

# Start Node with specific configuration
java -jar selenium-server-4.15.0.jar node \
  --hub http://localhost:4444 \
  --max-sessions 5 \
  --detect-drivers true
```

### Docker Compose for Grid

```yaml
# docker-compose.yml
version: "3"
services:
  selenium-hub:
    image: selenium/hub:4.15.0
    container_name: selenium-hub
    ports:
      - "4444:4444"
      - "4442:4442"
      - "4443:4443"
    environment:
      - SE_SESSION_REQUEST_TIMEOUT=300
      - SE_NODE_SESSION_TIMEOUT=300
    
  chrome:
    image: selenium/node-chrome:4.15.0
    depends_on:
      - selenium-hub
    environment:
      - SE_EVENT_BUS_HOST=selenium-hub
      - SE_EVENT_BUS_PUBLISH_PORT=4442
      - SE_EVENT_BUS_SUBSCRIBE_PORT=4443
      - SE_NODE_MAX_SESSIONS=5
    ports:
      - "6900:5900"
    shm_size: 2gb
    
  firefox:
    image: selenium/node-firefox:4.15.0
    depends_on:
      - selenium-hub
    environment:
      - SE_EVENT_BUS_HOST=selenium-hub
      - SE_EVENT_BUS_PUBLISH_PORT=4442
      - SE_EVENT_BUS_SUBSCRIBE_PORT=4443
      - SE_NODE_MAX_SESSIONS=5
    ports:
      - "6901:5900"
    shm_size: 2gb
```

```bash
# Start Grid
docker-compose up -d

# Scale nodes
docker-compose up -d --scale chrome=3 --scale firefox=2

# Stop Grid
docker-compose down
```

### Remote WebDriver Configuration

```java
import org.openqa.selenium.remote.RemoteWebDriver;
import org.openqa.selenium.chrome.ChromeOptions;
import java.net.URL;

public class GridTestExample {
    
    public WebDriver setupRemoteDriver() throws MalformedURLException {
        ChromeOptions options = new ChromeOptions();
        options.setCapability("platformName", "linux");
        
        WebDriver driver = new RemoteWebDriver(
            new URL("http://localhost:4444/wd/hub"), 
            options
        );
        
        return driver;
    }
    
    // Parallel execution with TestNG
    @Test
    public void test1() throws MalformedURLException {
        WebDriver driver = setupRemoteDriver();
        driver.get("https://example.com");
        // Test steps
        driver.quit();
    }
}
```

---

## Page Object Model (POM)

### Page Object Pattern

```java
import org.openqa.selenium.*;
import org.openqa.selenium.support.*;
import org.openqa.selenium.support.ui.WebDriverWait;
import java.time.Duration;

// Base Page
public class BasePage {
    protected WebDriver driver;
    protected WebDriverWait wait;
    
    public BasePage(WebDriver driver) {
        this.driver = driver;
        this.wait = new WebDriverWait(driver, Duration.ofSeconds(20));
        PageFactory.initElements(driver, this);
    }
    
    protected void clickElement(WebElement element) {
        wait.until(ExpectedConditions.elementToBeClickable(element));
        element.click();
    }
    
    protected void typeText(WebElement element, String text) {
        wait.until(ExpectedConditions.visibilityOf(element));
        element.clear();
        element.sendKeys(text);
    }
    
    protected String getTextFromElement(WebElement element) {
        wait.until(ExpectedConditions.visibilityOf(element));
        return element.getText();
    }
    
    protected void waitForElementVisible(WebElement element) {
        wait.until(ExpectedConditions.visibilityOf(element));
    }
}

// Login Page
public class LoginPage extends BasePage {
    
    @FindBy(id = "username")
    private WebElement usernameInput;
    
    @FindBy(id = "password")
    private WebElement passwordInput;
    
    @FindBy(xpath = "//button[@type='submit']")
    private WebElement loginButton;
    
    @FindBy(css = ".error-message")
    private WebElement errorMessage;
    
    public LoginPage(WebDriver driver) {
        super(driver);
    }
    
    public LoginPage enterUsername(String username) {
        typeText(usernameInput, username);
        return this;
    }
    
    public LoginPage enterPassword(String password) {
        typeText(passwordInput, password);
        return this;
    }
    
    public HomePage clickLogin() {
        clickElement(loginButton);
        return new HomePage(driver);
    }
    
    public String getErrorMessage() {
        return getTextFromElement(errorMessage);
    }
    
    // Fluent interface
    public HomePage login(String username, String password) {
        enterUsername(username);
        enterPassword(password);
        return clickLogin();
    }
}

// Home Page
public class HomePage extends BasePage {
    
    @FindBy(css = ".welcome-message")
    private WebElement welcomeMessage;
    
    @FindBy(id = "logout")
    private WebElement logoutButton;
    
    @FindBy(css = ".user-profile")
    private WebElement userProfile;
    
    public HomePage(WebDriver driver) {
        super(driver);
    }
    
    public String getWelcomeMessage() {
        return getTextFromElement(welcomeMessage);
    }
    
    public boolean isLoggedIn() {
        try {
            waitForElementVisible(welcomeMessage);
            return true;
        } catch (TimeoutException e) {
            return false;
        }
    }
    
    public LoginPage logout() {
        clickElement(logoutButton);
        return new LoginPage(driver);
    }
}

// Test Class
public class LoginTest {
    WebDriver driver;
    
    @BeforeMethod
    public void setup() {
        driver = new ChromeDriver();
        driver.get("https://example.com/login");
    }
    
    @Test
    public void testSuccessfulLogin() {
        LoginPage loginPage = new LoginPage(driver);
        HomePage homePage = loginPage.login("user@example.com", "password123");
        
        Assert.assertTrue(homePage.isLoggedIn());
        Assert.assertTrue(homePage.getWelcomeMessage().contains("Welcome"));
    }
    
    @Test
    public void testInvalidLogin() {
        LoginPage loginPage = new LoginPage(driver);
        loginPage.enterUsername("invalid@example.com")
                 .enterPassword("wrongpassword")
                 .clickLogin();
        
        Assert.assertTrue(loginPage.getErrorMessage()
            .contains("Invalid credentials"));
    }
    
    @AfterMethod
    public void teardown() {
        if (driver != null) {
            driver.quit();
        }
    }
}
```

### Advanced POM with Page Component Pattern

```java
// Page Component
public class NavigationComponent extends BasePage {
    
    @FindBy(css = ".nav-home")
    private WebElement homeLink;
    
    @FindBy(css = ".nav-products")
    private WebElement productsLink;
    
    @FindBy(css = ".nav-cart")
    private WebElement cartLink;
    
    public NavigationComponent(WebDriver driver) {
        super(driver);
    }
    
    public HomePage navigateToHome() {
        clickElement(homeLink);
        return new HomePage(driver);
    }
    
    public ProductsPage navigateToProducts() {
        clickElement(productsLink);
        return new ProductsPage(driver);
    }
    
    public CartPage navigateToCart() {
        clickElement(cartLink);
        return new CartPage(driver);
    }
}

// Page with Component
public class ProductsPage extends BasePage {
    
    private NavigationComponent navigation;
    
    @FindBy(css = ".product-item")
    private List<WebElement> products;
    
    @FindBy(id = "search-box")
    private WebElement searchBox;
    
    public ProductsPage(WebDriver driver) {
        super(driver);
        this.navigation = new NavigationComponent(driver);
    }
    
    public NavigationComponent getNavigation() {
        return navigation;
    }
    
    public ProductsPage searchProduct(String productName) {
        typeText(searchBox, productName);
        searchBox.sendKeys(Keys.ENTER);
        return this;
    }
    
    public int getProductCount() {
        return products.size();
    }
    
    public ProductDetailsPage selectProduct(int index) {
        clickElement(products.get(index));
        return new ProductDetailsPage(driver);
    }
}
```

---

## Advanced Element Interactions

### Actions Class

```java
import org.openqa.selenium.interactions.Actions;

public class AdvancedInteractions {
    private WebDriver driver;
    private Actions actions;
    
    public AdvancedInteractions(WebDriver driver) {
        this.driver = driver;
        this.actions = new Actions(driver);
    }
    
    // 1. Hover over element
    public void hoverOverElement(By locator) {
        WebElement element = driver.findElement(locator);
        actions.moveToElement(element).perform();
    }
    
    // 2. Right click (Context click)
    public void rightClick(By locator) {
        WebElement element = driver.findElement(locator);
        actions.contextClick(element).perform();
    }
    
    // 3. Double click
    public void doubleClick(By locator) {
        WebElement element = driver.findElement(locator);
        actions.doubleClick(element).perform();
    }
    
    // 4. Drag and Drop
    public void dragAndDrop(By sourceLocator, By targetLocator) {
        WebElement source = driver.findElement(sourceLocator);
        WebElement target = driver.findElement(targetLocator);
        actions.dragAndDrop(source, target).perform();
    }
    
    // 5. Drag and Drop by offset
    public void dragAndDropByOffset(By locator, int xOffset, int yOffset) {
        WebElement element = driver.findElement(locator);
        actions.dragAndDropBy(element, xOffset, yOffset).perform();
    }
    
    // 6. Click and Hold
    public void clickAndHold(By locator) {
        WebElement element = driver.findElement(locator);
        actions.clickAndHold(element)
               .pause(Duration.ofSeconds(2))
               .release()
               .perform();
    }
    
    // 7. Keyboard actions
    public void keyboardActions() {
        WebElement element = driver.findElement(By.id("input"));
        
        // Type in uppercase
        actions.keyDown(Keys.SHIFT)
               .sendKeys("hello")
               .keyUp(Keys.SHIFT)
               .perform();
        
        // Select all and copy
        actions.keyDown(Keys.CONTROL)
               .sendKeys("a")
               .sendKeys("c")
               .keyUp(Keys.CONTROL)
               .perform();
    }
    
    // 8. Complex action chain
    public void complexActionChain() {
        WebElement element1 = driver.findElement(By.id("element1"));
        WebElement element2 = driver.findElement(By.id("element2"));
        
        actions.moveToElement(element1)
               .click()
               .pause(Duration.ofSeconds(1))
               .moveToElement(element2)
               .doubleClick()
               .perform();
    }
    
    // 9. Slider interaction
    public void moveSlider(By sliderLocator, int percentage) {
        WebElement slider = driver.findElement(sliderLocator);
        int width = slider.getSize().getWidth();
        int moveBy = (width * percentage) / 100;
        
        actions.clickAndHold(slider)
               .moveByOffset(moveBy, 0)
               .release()
               .perform();
    }
}
```

### Handling Different Elements

```java
public class ElementHandlers {
    private WebDriver driver;
    
    // 1. Handle Dropdowns
    public void handleDropdown() {
        Select dropdown = new Select(driver.findElement(By.id("country")));
        
        // Select by visible text
        dropdown.selectByVisibleText("United States");
        
        // Select by value
        dropdown.selectByValue("us");
        
        // Select by index
        dropdown.selectByIndex(1);
        
        // Get selected option
        String selected = dropdown.getFirstSelectedOption().getText();
        
        // Get all options
        List<WebElement> allOptions = dropdown.getOptions();
        for (WebElement option : allOptions) {
            System.out.println(option.getText());
        }
    }
    
    // 2. Handle Checkboxes
    public void handleCheckbox() {
        WebElement checkbox = driver.findElement(By.id("terms"));
        
        if (!checkbox.isSelected()) {
            checkbox.click();
        }
        
        // Verify checked
        Assert.assertTrue(checkbox.isSelected());
    }
    
    // 3. Handle Radio Buttons
    public void handleRadioButton() {
        List<WebElement> radioButtons = driver.findElements(
            By.name("gender"));
        
        for (WebElement radio : radioButtons) {
            if (radio.getAttribute("value").equals("male")) {
                radio.click();
                break;
            }
        }
    }
    
    // 4. Handle Alerts
    public void handleAlerts() {
        WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
        
        // Simple alert
        Alert alert = wait.until(ExpectedConditions.alertIsPresent());
        System.out.println("Alert text: " + alert.getText());
        alert.accept();
        
        // Confirmation alert
        driver.findElement(By.id("confirm-button")).click();
        Alert confirmAlert = driver.switchTo().alert();
        confirmAlert.dismiss();  // Click Cancel
        
        // Prompt alert
        driver.findElement(By.id("prompt-button")).click();
        Alert promptAlert = driver.switchTo().alert();
        promptAlert.sendKeys("Test Input");
        promptAlert.accept();
    }
    
    // 5. Handle iFrames
    public void handleIFrames() {
        // Switch by index
        driver.switchTo().frame(0);
        
        // Switch by name or ID
        driver.switchTo().frame("frameName");
        
        // Switch by WebElement
        WebElement frameElement = driver.findElement(By.id("frameId"));
        driver.switchTo().frame(frameElement);
        
        // Perform actions inside frame
        driver.findElement(By.id("insideFrame")).click();
        
        // Switch back to main content
        driver.switchTo().defaultContent();
        
        // Switch to parent frame
        driver.switchTo().parentFrame();
    }
    
    // 6. Handle Multiple Windows
    public void handleWindows() {
        String mainWindow = driver.getWindowHandle();
        
        // Click element that opens new window
        driver.findElement(By.id("newWindow")).click();
        
        // Get all windows
        Set<String> allWindows = driver.getWindowHandles();
        
        // Switch to new window
        for (String window : allWindows) {
            if (!window.equals(mainWindow)) {
                driver.switchTo().window(window);
                // Perform actions in new window
                System.out.println("New window title: " + driver.getTitle());
                driver.close();
            }
        }
        
        // Switch back to main window
        driver.switchTo().window(mainWindow);
    }
    
    // 7. Handle Web Tables
    public void handleWebTable() {
        WebElement table = driver.findElement(By.id("dataTable"));
        List<WebElement> rows = table.findElements(By.tagName("tr"));
        
        for (int i = 1; i < rows.size(); i++) {  // Skip header
            List<WebElement> cells = rows.get(i).findElements(By.tagName("td"));
            
            String name = cells.get(0).getText();
            String email = cells.get(1).getText();
            String age = cells.get(2).getText();
            
            System.out.println(name + " | " + email + " | " + age);
        }
    }
    
    // Get specific cell value
    public String getCellValue(int row, int column) {
        WebElement cell = driver.findElement(
            By.xpath("//table[@id='dataTable']//tr[" + row + "]//td[" + column + "]")
        );
        return cell.getText();
    }
    
    // 8. Handle File Upload
    public void handleFileUpload() {
        WebElement uploadElement = driver.findElement(By.id("fileUpload"));
        String filePath = System.getProperty("user.dir") + "/testfile.pdf";
        uploadElement.sendKeys(filePath);
    }
    
    // 9. Handle File Download
    public void handleFileDownload() {
        driver.findElement(By.id("downloadLink")).click();
        
        // Wait for download to complete
        String downloadPath = System.getProperty("user.home") + "/Downloads";
        File file = new File(downloadPath + "/downloaded-file.pdf");
        
        int timeout = 30;
        while (timeout > 0 && !file.exists()) {
            try {
                Thread.sleep(1000);
                timeout--;
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        
        Assert.assertTrue(file.exists(), "File not downloaded");
    }
}
```

---

## JavaScript Executor

### Common JavaScript Operations

```java
public class JavaScriptExecutorExamples {
    private WebDriver driver;
    private JavascriptExecutor js;
    
    public JavaScriptExecutorExamples(WebDriver driver) {
        this.driver = driver;
        this.js = (JavascriptExecutor) driver;
    }
    
    // 1. Click element (bypass intercepted clicks)
    public void clickUsingJS(By locator) {
        WebElement element = driver.findElement(locator);
        js.executeScript("arguments[0].click();", element);
    }
    
    // 2. Type text
    public void typeUsingJS(By locator, String text) {
        WebElement element = driver.findElement(locator);
        js.executeScript("arguments[0].value='" + text + "';", element);
    }
    
    // 3. Scroll operations
    public void scrollToElement(By locator) {
        WebElement element = driver.findElement(locator);
        js.executeScript("arguments[0].scrollIntoView(true);", element);
    }
    
    public void scrollToTop() {
        js.executeScript("window.scrollTo(0, 0);");
    }
    
    public void scrollToBottom() {
        js.executeScript("window.scrollTo(0, document.body.scrollHeight);");
    }
    
    public void scrollByPixels(int x, int y) {
        js.executeScript("window.scrollBy(" + x + "," + y + ");");
    }
    
    // 4. Highlight element (for debugging)
    public void highlightElement(By locator) {
        WebElement element = driver.findElement(locator);
        String originalStyle = element.getAttribute("style");
        
        js.executeScript(
            "arguments[0].setAttribute('style', 'border: 3px solid red;');",
            element
        );
        
        try {
            Thread.sleep(500);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        
        js.executeScript(
            "arguments[0].setAttribute('style', '" + originalStyle + "');",
            element
        );
    }
    
    // 5. Get page information
    public String getPageTitle() {
        return js.executeScript("return document.title;").toString();
    }
    
    public String getCurrentURL() {
        return js.executeScript("return document.URL;").toString();
    }
    
    public Long getPageHeight() {
        return (Long) js.executeScript("return document.body.scrollHeight;");
    }
    
    // 6. Check if element is visible in viewport
    public boolean isElementInViewport(By locator) {
        WebElement element = driver.findElement(locator);
        return (Boolean) js.executeScript(
            "var elem = arguments[0];" +
            "var rect = elem.getBoundingClientRect();" +
            "return (" +
            "    rect.top >= 0 &&" +
            "    rect.left >= 0 &&" +
            "    rect.bottom <= (window.innerHeight || document.documentElement.clientHeight) &&" +
            "    rect.right <= (window.innerWidth || document.documentElement.clientWidth)" +
            ");",
            element
        );
    }
    
    // 7. Change element attributes
    public void changeElementAttribute(By locator, String attribute, String value) {
        WebElement element = driver.findElement(locator);
        js.executeScript(
            "arguments[0].setAttribute('" + attribute + "', '" + value + "');",
            element
        );
    }
    
    // 8. Remove element attribute
    public void removeElementAttribute(By locator, String attribute) {
        WebElement element = driver.findElement(locator);
        js.executeScript(
            "arguments[0].removeAttribute('" + attribute + "');",
            element
        );
    }
    
    // 9. Wait for page load
    public void waitForPageLoad() {
        WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(30));
        wait.until(driver -> 
            js.executeScript("return document.readyState").equals("complete")
        );
    }
    
    // 10. Refresh page using JS
    public void refreshPage() {
        js.executeScript("history.go(0);");
    }
    
    // 11. Open new tab
    public void openNewTab(String url) {
        js.executeScript("window.open('" + url + "','_blank');");
    }
    
    // 12. Generate alert
    public void generateAlert(String message) {
        js.executeScript("alert('" + message + "');");
    }
    
    // 13. Get element text (including hidden elements)
    public String getElementText(By locator) {
        WebElement element = driver.findElement(locator);
        return js.executeScript("return arguments[0].innerText;", element).toString();
    }
    
    // 14. Zoom page
    public void zoomPage(int percentage) {
        js.executeScript("document.body.style.zoom='" + percentage + "%';");
    }
    
    // 15. Handle calendar date picker
    public void setDatePicker(By locator, String date) {
        WebElement dateField = driver.findElement(locator);
        js.executeScript(
            "arguments[0].removeAttribute('readonly');" +
            "arguments[0].value='" + date + "';",
            dateField
        );
    }
}
```

---

## Practice Questions

### Selenium Expert Level Questions

1. **How do you handle StaleElementReferenceException? Provide multiple approaches.**

2. **Design a custom wait mechanism that waits for an element to stop moving on the page.**

3. **Implement a reusable method to handle dynamic web tables with search functionality.**

4. **Create a framework utility to take screenshots only when tests fail.**

5. **How would you handle a scenario where an element is present but not interactable?**

6. **Implement parallel test execution using Selenium Grid and TestNG.**

7. **Write code to download a file and verify its contents.**

8. **How do you handle infinite scrolling pages (like social media feeds)?**

9. **Create a utility class to handle all types of waits with appropriate logging.**

10. **Implement shadow DOM element interaction in your framework.**

---

**Next:** [Test Automation Frameworks](03-test-automation-frameworks.md)

