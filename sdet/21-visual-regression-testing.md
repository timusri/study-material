# 21. Visual Regression Testing

## Table of Contents
- [What is Visual Regression Testing](#what-is-visual-regression-testing)
- [Tools Overview](#tools-overview)
- [Implementing Visual Tests](#implementing-visual-tests)
- [Handling Dynamic Content](#handling-dynamic-content)
- [Best Practices](#best-practices)
- [CI/CD Integration](#cicd-integration)

---

## What is Visual Regression Testing

### Understanding Visual Bugs

```markdown
## Definition
**Visual Regression:** Unintended changes to the visual appearance of a UI.

## Why Functional Tests Miss Visual Bugs

**Functional Test:**
```java
@Test
public void testLoginButton() {
    WebElement loginButton = driver.findElement(By.id("login-btn"));
    Assert.assertTrue(loginButton.isDisplayed());  // ✅ Passes
    Assert.assertTrue(loginButton.isEnabled());    // ✅ Passes
}
```

**But the button could:**
- Be white text on white background (invisible)
- Be 1px × 1px (technically visible)
- Have overlapping elements
- Wrong font, color, size
- CSS broken

## Visual Bugs Caught by Visual Testing

❌ **Layout Issues:**
- Broken CSS
- Responsive design failures
- Elements overlapping
- Misaligned components

❌ **Styling Issues:**
- Wrong colors
- Wrong fonts
- Missing images
- Broken icons

❌ **Cross-browser Issues:**
- Different rendering
- Font rendering differences
- Browser-specific CSS bugs

❌ **Responsive Issues:**
- Mobile layout broken
- Tablet view issues
- Different screen resolutions

## Business Impact

**Example: Amazon (2008)**
A CSS change accidentally hid the "Add to Cart" button on IE6.
Cost: Millions in lost revenue for hours.

**Example: British Airways (2013)**
Font rendering issue made text unreadable on booking page.
Result: Customers couldn't complete bookings.
```

---

## Tools Overview

### Popular Visual Testing Tools

```markdown
## 1. Applitools Eyes ⭐⭐⭐
**Best for:** Enterprise, AI-powered

**Pros:**
✅ AI-driven matching
✅ Cross-browser testing
✅ Responsive testing
✅ Maintenance-free
✅ Great dashboard

**Cons:**
❌ Expensive
❌ SaaS only (data leaves premise)

## 2. Percy.io ⭐⭐⭐
**Best for:** CI/CD integration

**Pros:**
✅ Easy integration
✅ GitHub/GitLab integration
✅ Parallel testing
✅ Good free tier

**Cons:**
❌ Requires internet
❌ Can be expensive at scale

## 3. BackstopJS ⭐⭐
**Best for:** Open source, self-hosted

**Pros:**
✅ Free and open source
✅ Self-hosted
✅ No data sharing
✅ Good for simple use cases

**Cons:**
❌ Manual maintenance
❌ Pixel-perfect matching (brittle)
❌ Limited smart matching

## 4. Selenium + AShot ⭐⭐
**Best for:** DIY solution

**Pros:**
✅ Free
✅ Full control
✅ Works with existing Selenium tests

**Cons:**
❌ Need to build everything
❌ Pixel-based comparison
❌ High maintenance

## 5. Playwright Visual Comparisons ⭐⭐
**Best for:** Playwright users

**Pros:**
✅ Built-in to Playwright
✅ Free
✅ Good integration

**Cons:**
❌ Basic features
❌ Pixel-perfect matching
❌ Limited cross-browser
```

---

## Implementing Visual Tests

### Using Applitools Eyes

```xml
<!-- pom.xml -->
<dependency>
    <groupId>com.applitools</groupId>
    <artifactId>eyes-selenium-java5</artifactId>
    <version>5.70.0</version>
</dependency>
```

```java
import com.applitools.eyes.selenium.Eyes;
import com.applitools.eyes.selenium.Configuration;
import com.applitools.eyes.RectangleSize;
import com.applitools.eyes.MatchLevel;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.testng.annotations.*;

public class ApplitoolsVisualTest {
    
    private WebDriver driver;
    private Eyes eyes;
    
    @BeforeClass
    public void setup() {
        // Initialize Selenium driver
        driver = new ChromeDriver();
        
        // Initialize Eyes
        eyes = new Eyes();
        eyes.setApiKey("YOUR_API_KEY");
        
        // Configuration
        Configuration config = new Configuration();
        config.setAppName("My Application");
        config.setTestName("Visual Regression Test");
        config.setViewportSize(new RectangleSize(1200, 800));
        config.setMatchLevel(MatchLevel.STRICT); // STRICT, LAYOUT, CONTENT, EXACT
        
        eyes.setConfiguration(config);
    }
    
    @Test
    public void testHomePage() {
        // Start visual test
        eyes.open(driver, "My App", "Home Page Test", 
                 new RectangleSize(1200, 800));
        
        // Navigate to page
        driver.get("https://example.com");
        
        // Check full page
        eyes.checkWindow("Home Page - Full");
        
        // Check specific element
        WebElement loginForm = driver.findElement(By.id("login-form"));
        eyes.checkElement(loginForm, "Login Form");
        
        // Check region
        eyes.checkRegion(By.cssSelector(".header"), "Header Region");
        
        // End test
        eyes.closeAsync();
    }
    
    @Test
    public void testResponsiveLayout() {
        eyes.open(driver, "My App", "Responsive Test",
                 new RectangleSize(1200, 800));
        
        driver.get("https://example.com");
        
        // Test multiple viewport sizes
        int[][] viewports = {
            {1920, 1080},  // Desktop
            {1366, 768},   // Laptop
            {768, 1024},   // Tablet
            {375, 667}     // Mobile
        };
        
        for (int[] viewport : viewports) {
            driver.manage().window().setSize(
                new Dimension(viewport[0], viewport[1])
            );
            eyes.checkWindow("Viewport " + viewport[0] + "x" + viewport[1]);
        }
        
        eyes.closeAsync();
    }
    
    @Test
    public void testWithRegions() {
        eyes.open(driver, "My App", "Region Test",
                 new RectangleSize(1200, 800));
        
        driver.get("https://example.com/dashboard");
        
        // Check with ignore regions (for dynamic content)
        eyes.check(Target.window()
            .fully()
            .ignore(By.id("timestamp"))         // Ignore timestamp
            .ignore(By.className("ad-banner"))  // Ignore ads
            .layout(By.id("sidebar"))           // Only check layout of sidebar
            .strict(By.id("header"))            // Strict check for header
        );
        
        eyes.closeAsync();
    }
    
    @AfterClass
    public void teardown() {
        if (driver != null) {
            driver.quit();
        }
        if (eyes != null) {
            eyes.abortAsync();
        }
    }
}
```

### Using Percy.io

```xml
<!-- pom.xml -->
<dependency>
    <groupId>io.percy</groupId>
    <artifactId>percy-java-selenium</artifactId>
    <version>1.0.0</version>
</dependency>
```

```java
import io.percy.selenium.Percy;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.testng.annotations.*;

public class PercyVisualTest {
    
    private WebDriver driver;
    private Percy percy;
    
    @BeforeClass
    public void setup() {
        driver = new ChromeDriver();
        percy = new Percy(driver);
    }
    
    @Test
    public void testHomePage() {
        driver.get("https://example.com");
        
        // Take snapshot
        percy.snapshot("Home Page");
    }
    
    @Test
    public void testResponsive() {
        driver.get("https://example.com");
        
        // Snapshot with multiple widths
        percy.snapshot("Home Page Responsive", 
            Arrays.asList(375, 768, 1280), 
            null, // No custom CSS
            false // Enable JavaScript
        );
    }
    
    @Test
    public void testWithOptions() {
        driver.get("https://example.com/dashboard");
        
        // Snapshot with options
        Map<String, Object> options = new HashMap<>();
        options.put("widths", Arrays.asList(375, 1280));
        options.put("minHeight", 1024);
        options.put("enableJavaScript", true);
        options.put("percyCSS", ".timestamp { display: none; }"); // Hide dynamic content
        
        percy.snapshot("Dashboard", options);
    }
    
    @AfterClass
    public void teardown() {
        if (driver != null) {
            driver.quit();
        }
    }
}
```

### Using BackstopJS

```json
// backstop.json
{
  "id": "my_project",
  "viewports": [
    {
      "label": "desktop",
      "width": 1920,
      "height": 1080
    },
    {
      "label": "tablet",
      "width": 768,
      "height": 1024
    },
    {
      "label": "mobile",
      "width": 375,
      "height": 667
    }
  ],
  "scenarios": [
    {
      "label": "Home Page",
      "url": "https://example.com",
      "referenceUrl": "",
      "readyEvent": "",
      "readySelector": "",
      "delay": 500,
      "hideSelectors": [".timestamp", ".ad-banner"],
      "removeSelectors": [],
      "hoverSelector": "",
      "clickSelector": "",
      "postInteractionWait": 0,
      "selectors": ["document"],
      "selectorExpansion": true,
      "misMatchThreshold": 0.1
    },
    {
      "label": "Login Page",
      "url": "https://example.com/login",
      "delay": 1000,
      "hideSelectors": [],
      "selectors": ["document"]
    }
  ],
  "paths": {
    "bitmaps_reference": "backstop_data/bitmaps_reference",
    "bitmaps_test": "backstop_data/bitmaps_test",
    "engine_scripts": "backstop_data/engine_scripts",
    "html_report": "backstop_data/html_report",
    "ci_report": "backstop_data/ci_report"
  },
  "report": ["browser", "CI"],
  "engine": "puppeteer",
  "engineOptions": {
    "args": ["--no-sandbox"]
  },
  "asyncCaptureLimit": 5,
  "asyncCompareLimit": 50,
  "debug": false,
  "debugWindow": false
}
```

```bash
# Create reference images (baseline)
backstop reference

# Run test (compare against reference)
backstop test

# Approve changes (update baseline)
backstop approve
```

### DIY with Selenium + AShot

```xml
<!-- pom.xml -->
<dependency>
    <groupId>ru.yandex.qatools.ashot</groupId>
    <artifactId>ashot</artifactId>
    <version>1.5.4</version>
</dependency>
```

```java
import ru.yandex.qatools.ashot.AShot;
import ru.yandex.qatools.ashot.Screenshot;
import ru.yandex.qatools.ashot.shooting.ShootingStrategies;
import ru.yandex.qatools.ashot.comparison.ImageDiff;
import ru.yandex.qatools.ashot.comparison.ImageDiffer;
import javax.imageio.ImageIO;
import java.awt.image.BufferedImage;
import java.io.File;

public class DIYVisualTest {
    
    private WebDriver driver;
    
    @Test
    public void testVisualRegression() throws IOException {
        driver.get("https://example.com");
        
        // Take screenshot
        Screenshot screenshot = new AShot()
            .shootingStrategy(ShootingStrategies.viewportPasting(100))
            .takeScreenshot(driver);
        
        BufferedImage actualImage = screenshot.getImage();
        
        // Path to baseline image
        File baselineFile = new File("baseline/home-page.png");
        
        if (!baselineFile.exists()) {
            // Create baseline if doesn't exist
            ImageIO.write(actualImage, "PNG", baselineFile);
            System.out.println("Baseline created");
            return;
        }
        
        // Load baseline image
        BufferedImage expectedImage = ImageIO.read(baselineFile);
        
        // Compare images
        ImageDiffer imgDiff = new ImageDiffer();
        ImageDiff diff = imgDiff.makeDiff(expectedImage, actualImage);
        
        if (diff.hasDiff()) {
            // Save diff image
            BufferedImage diffImage = diff.getMarkedImage();
            ImageIO.write(diffImage, "PNG", 
                new File("diff/home-page-diff.png"));
            
            int diffPixels = diff.getDiffSize();
            System.out.println("Visual differences found: " + diffPixels + " pixels");
            
            // Calculate difference percentage
            int totalPixels = expectedImage.getWidth() * expectedImage.getHeight();
            double diffPercentage = (diffPixels * 100.0) / totalPixels;
            
            Assert.assertTrue(diffPercentage < 1.0, 
                "Visual difference exceeds threshold: " + diffPercentage + "%");
        } else {
            System.out.println("No visual differences found");
        }
    }
    
    @Test
    public void testSpecificElement() throws IOException {
        driver.get("https://example.com");
        
        WebElement loginForm = driver.findElement(By.id("login-form"));
        
        // Screenshot specific element
        Screenshot screenshot = new AShot()
            .takeScreenshot(driver, loginForm);
        
        BufferedImage actualImage = screenshot.getImage();
        
        File baselineFile = new File("baseline/login-form.png");
        
        if (!baselineFile.exists()) {
            ImageIO.write(actualImage, "PNG", baselineFile);
            return;
        }
        
        BufferedImage expectedImage = ImageIO.read(baselineFile);
        
        ImageDiffer imgDiff = new ImageDiffer();
        ImageDiff diff = imgDiff.makeDiff(expectedImage, actualImage);
        
        Assert.assertFalse(diff.hasDiff(), "Login form has visual changes");
    }
}
```

---

## Handling Dynamic Content

### Strategies for Dynamic Elements

```java
public class DynamicContentHandling {
    
    // Strategy 1: Hide dynamic elements
    @Test
    public void testWithHiddenElements() {
        driver.get("https://example.com");
        
        // Hide timestamp and ads
        JavascriptExecutor js = (JavascriptExecutor) driver;
        js.executeScript(
            "document.getElementById('timestamp').style.display = 'none';" +
            "document.querySelector('.ad-banner').style.display = 'none';"
        );
        
        // Take screenshot
        percy.snapshot("Page without dynamic content");
    }
    
    // Strategy 2: Replace with static content
    @Test
    public void testWithReplacedContent() {
        driver.get("https://example.com");
        
        JavascriptExecutor js = (JavascriptExecutor) driver;
        
        // Replace timestamp with fixed value
        js.executeScript(
            "document.getElementById('timestamp').textContent = '2024-01-15 10:00:00';"
        );
        
        // Replace user name with static value
        js.executeScript(
            "document.getElementById('username').textContent = 'Test User';"
        );
        
        percy.snapshot("Page with static content");
    }
    
    // Strategy 3: Ignore regions (Applitools)
    @Test
    public void testWithIgnoreRegions() {
        driver.get("https://example.com");
        
        eyes.check(Target.window()
            .fully()
            .ignore(By.id("timestamp"))
            .ignore(By.className("ad-banner"))
            .ignore(By.cssSelector(".user-avatar"))
        );
    }
    
    // Strategy 4: Layout-only check (Applitools)
    @Test
    public void testLayoutOnly() {
        driver.get("https://example.com");
        
        // Only check layout, ignore content changes
        eyes.check(Target.window()
            .layout(By.id("news-feed")) // Content changes but layout stays same
        );
    }
    
    // Strategy 5: Mock API responses
    @BeforeMethod
    public void mockDynamicData() {
        // Use tools like WireMock to return consistent data
        // Or use service virtualization
        
        // Set up mock responses
        stubFor(get(urlEqualTo("/api/user"))
            .willReturn(aResponse()
                .withStatus(200)
                .withBody("{\"name\":\"Test User\",\"timestamp\":\"2024-01-15T10:00:00Z\"}")
            ));
    }
}
```

### Handling Animations

```java
public class AnimationHandling {
    
    // Strategy 1: Disable animations with CSS
    @Test
    public void testWithoutAnimations() {
        driver.get("https://example.com");
        
        // Inject CSS to disable all animations
        JavascriptExecutor js = (JavascriptExecutor) driver;
        js.executeScript(
            "const style = document.createElement('style');" +
            "style.textContent = '* {" +
            "  animation: none !important;" +
            "  transition: none !important;" +
            "}';" +
            "document.head.appendChild(style);"
        );
        
        // Wait for animations to be disabled
        Thread.sleep(500);
        
        percy.snapshot("Page without animations");
    }
    
    // Strategy 2: Wait for animations to complete
    @Test
    public void testAfterAnimations() {
        driver.get("https://example.com");
        
        // Wait for specific animation to complete
        WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
        wait.until((WebDriver d) -> {
            JavascriptExecutor js = (JavascriptExecutor) d;
            return (Boolean) js.executeScript(
                "const el = document.getElementById('animated-element');" +
                "const style = window.getComputedStyle(el);" +
                "return style.animationPlayState === 'finished' || " +
                "       style.animationPlayState === 'paused';"
            );
        });
        
        percy.snapshot("Page after animations");
    }
    
    // Strategy 3: Set animation to final state
    @Test
    public void testAnimationFinalState() {
        driver.get("https://example.com");
        
        JavascriptExecutor js = (JavascriptExecutor) driver;
        
        // Set animation to end state
        js.executeScript(
            "const el = document.getElementById('animated-element');" +
            "el.style.animationDelay = '-9999s';" // Jump to end
        );
        
        percy.snapshot("Animation final state");
    }
}
```

---

## Best Practices

### Visual Testing Best Practices

```java
public class VisualTestingBestPractices {
    
    /**
     * ✅ 1. Test Critical User Journeys
     */
    @Test
    public void testCheckoutFlow() {
        // Home page
        driver.get("https://example.com");
        percy.snapshot("1. Home Page");
        
        // Product page
        driver.findElement(By.className("product")).click();
        percy.snapshot("2. Product Page");
        
        // Cart
        driver.findElement(By.id("add-to-cart")).click();
        percy.snapshot("3. Cart");
        
        // Checkout
        driver.findElement(By.id("checkout")).click();
        percy.snapshot("4. Checkout");
    }
    
    /**
     * ✅ 2. Test Multiple Viewports
     */
    @Test
    public void testResponsive() {
        driver.get("https://example.com");
        
        percy.snapshot("Home Responsive", 
            Arrays.asList(375, 768, 1280, 1920));
    }
    
    /**
     * ✅ 3. Test Different States
     */
    @Test
    public void testStates() {
        driver.get("https://example.com");
        
        // Normal state
        percy.snapshot("Button - Normal");
        
        // Hover state
        Actions actions = new Actions(driver);
        WebElement button = driver.findElement(By.id("submit"));
        actions.moveToElement(button).perform();
        percy.snapshot("Button - Hover");
        
        // Disabled state
        JavascriptExecutor js = (JavascriptExecutor) driver;
        js.executeScript("arguments[0].disabled = true;", button);
        percy.snapshot("Button - Disabled");
    }
    
    /**
     * ✅ 4. Wait for Page Stability
     */
    public void waitForPageStability() {
        // Wait for DOM to be stable
        WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
        wait.until((WebDriver d) -> {
            return ((JavascriptExecutor) d).executeScript(
                "return document.readyState"
            ).equals("complete");
        });
        
        // Wait for network idle (no pending requests)
        wait.until((WebDriver d) -> {
            return (Boolean) ((JavascriptExecutor) d).executeScript(
                "return typeof jQuery !== 'undefined' ? jQuery.active === 0 : true"
            );
        });
        
        // Additional wait for animations
        try {
            Thread.sleep(500);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
    
    /**
     * ✅ 5. Descriptive Snapshot Names
     */
    @Test
    public void testWithDescriptiveNames() {
        driver.get("https://example.com");
        
        // ❌ Bad
        percy.snapshot("Test1");
        
        // ✅ Good
        percy.snapshot("Home Page - Logged Out - Desktop");
        percy.snapshot("Home Page - Logged In - Mobile");
        percy.snapshot("Product Page - Out of Stock - Tablet");
    }
    
    /**
     * ✅ 6. Don't Snapshot Everything
     */
    @Test
    public void testFocusedSnapshots() {
        // Don't snapshot admin pages if testing customer site
        // Don't snapshot 404 pages
        // Focus on user-facing critical paths
        
        driver.get("https://example.com/products");
        percy.snapshot("Products Page"); // ✅ Critical
        
        // driver.get("https://example.com/admin/logs");
        // percy.snapshot("Admin Logs"); // ❌ Not critical for users
    }
    
    /**
     * ✅ 7. Set Appropriate Thresholds
     */
    public void setThresholds() {
        // Applitools
        eyes.setMatchLevel(MatchLevel.STRICT); // 99.9% match
        // vs
        eyes.setMatchLevel(MatchLevel.LAYOUT); // Only layout, ignore colors
        
        // Custom threshold (pixel difference)
        // Allow 0.1% pixel difference (for anti-aliasing)
        double threshold = 0.1;
    }
}
```

---

## CI/CD Integration

### Jenkins Pipeline with Percy

```groovy
pipeline {
    agent any
    
    environment {
        PERCY_TOKEN = credentials('percy-token')
    }
    
    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/your-repo/your-project.git'
            }
        }
        
        stage('Build') {
            steps {
                sh 'mvn clean compile'
            }
        }
        
        stage('Visual Tests') {
            steps {
                sh '''
                    export PERCY_TOKEN=${PERCY_TOKEN}
                    mvn test -Dtest=*VisualTest
                '''
            }
        }
    }
    
    post {
        always {
            publishHTML([
                reportDir: 'target/percy-reports',
                reportFiles: 'index.html',
                reportName: 'Percy Visual Report'
            ])
        }
        
        failure {
            emailext(
                subject: "Visual Regression Detected: ${env.JOB_NAME}",
                body: "Visual changes detected. Review at: ${env.BUILD_URL}",
                to: "team@example.com"
            )
        }
    }
}
```

### GitHub Actions with Percy

```yaml
# .github/workflows/visual-tests.yml
name: Visual Regression Tests

on: [push, pull_request]

jobs:
  visual-tests:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Java
        uses: actions/setup-java@v3
        with:
          java-version: '11'
          distribution: 'adopt'
      
      - name: Run Visual Tests
        env:
          PERCY_TOKEN: ${{ secrets.PERCY_TOKEN }}
        run: |
          mvn clean test -Dtest=*VisualTest
      
      - name: Upload Screenshots
        if: failure()
        uses: actions/upload-artifact@v3
        with:
          name: screenshots
          path: target/screenshots/
```

---

## Interview Questions

### Q1: What is visual regression testing and why is it important?

**Answer:**
```markdown
**Definition:**
Visual regression testing detects unintended visual changes in UI.
Compares current UI against baseline (expected) images.

**Why Important:**

1. **Functional tests miss visual bugs:**
   - Element exists ✅
   - Element clickable ✅
   - But invisible (white on white) ❌
   
2. **Real-world impact:**
   - Broken layouts lose customers
   - CSS bugs affect branding
   - Responsive issues on mobile
   
3. **Catches:**
   - CSS regressions
   - Layout shifts
   - Cross-browser rendering issues
   - Responsive design breaks
   - Font/color changes

**Example:**
```java
// Functional test passes
Assert.assertTrue(loginButton.isDisplayed());

// But button is invisible due to CSS bug
// Visual test would catch this
```

### Q2: How do you handle dynamic content in visual testing?

**Answer:**
```markdown
**Strategies:**

1. **Hide Dynamic Elements:**
```javascript
document.getElementById('timestamp').style.display = 'none';
```

2. **Replace with Static Values:**
```javascript
document.getElementById('username').textContent = 'Test User';
```

3. **Ignore Regions:**
```java
eyes.check(Target.window()
    .ignore(By.id("timestamp"))
    .ignore(By.className("ad-banner")));
```

4. **Layout-Only Check:**
```java
// Check layout, ignore content
eyes.check(Target.window()
    .layout(By.id("news-feed")));
```

5. **Mock API Responses:**
- Use consistent test data
- Service virtualization
- Stub dynamic endpoints

**Best Practice:**
Identify dynamic elements upfront and choose appropriate strategy.
```

---

**Next:** [Tool Comparison & Selection Guide](22-tool-comparison-guide.md)

