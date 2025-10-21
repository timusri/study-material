# 17. Mobile Testing with Appium

## Table of Contents
- [Mobile Testing Fundamentals](#mobile-testing-fundamentals)
- [Appium Architecture](#appium-architecture)
- [iOS vs Android Testing](#ios-vs-android-testing)
- [Mobile Gestures & Interactions](#mobile-gestures--interactions)
- [Mobile-Specific Challenges](#mobile-specific-challenges)
- [Mobile CI/CD](#mobile-cicd)

---

## Mobile Testing Fundamentals

### Mobile Testing vs Web Testing

```markdown
| Aspect | Web Testing | Mobile Testing |
|--------|-------------|----------------|
| **Devices** | Browsers | Physical devices, Emulators |
| **Screen Sizes** | Desktop/Laptop | Multiple form factors |
| **Network** | Stable WiFi | WiFi, 3G, 4G, 5G, Offline |
| **Interruptions** | Minimal | Calls, SMS, Low battery |
| **OS Versions** | Few variants | Multiple OS versions |
| **Touch** | Mouse/Keyboard | Touch gestures |
| **Installation** | URL access | App installation |
| **Updates** | Automatic | Manual/Auto updates |
| **Performance** | High resources | Limited battery, memory |
```

### Types of Mobile Apps

```markdown
## 1. Native Apps

**Characteristics:**
- Platform-specific (iOS/Android)
- Best performance
- Full device access
- App store distribution

**Technology:**
- iOS: Swift, Objective-C
- Android: Java, Kotlin

**Testing Tools:**
- iOS: XCUITest
- Android: Espresso
- Cross-platform: Appium

## 2. Hybrid Apps

**Characteristics:**
- Web tech wrapped in native container
- Single codebase
- Moderate performance

**Technology:**
- Cordova, Ionic, React Native

**Testing:**
- Appium (preferred)
- Can test WebView content

## 3. Web Apps

**Characteristics:**
- Browser-based
- Responsive design
- No installation needed

**Testing:**
- Selenium on mobile browsers
- BrowserStack, Sauce Labs
```

---

## Appium Architecture

### Appium Overview

```
┌─────────────────────────────────────────────────┐
│         Test Script (Java/Python/JS)            │
│         (Your Automation Code)                  │
└───────────────────┬─────────────────────────────┘
                    │
                    │ HTTP/JSON Wire Protocol
                    ↓
┌─────────────────────────────────────────────────┐
│           Appium Server (Node.js)               │
│     - Receives commands                         │
│     - Routes to appropriate driver              │
└───────────────────┬─────────────────────────────┘
                    │
        ┌───────────┴───────────┐
        ↓                       ↓
┌───────────────┐       ┌───────────────┐
│  iOS Driver   │       │ Android Driver│
│  (XCUITest)   │       │  (UiAutomator2)│
└───────┬───────┘       └───────┬───────┘
        ↓                       ↓
┌───────────────┐       ┌───────────────┐
│  iOS Device/  │       │ Android Device│
│  Simulator    │       │  /Emulator    │
└───────────────┘       └───────────────┘
```

### Appium Setup

```bash
# Prerequisites
brew install node  # macOS
npm install -g appium

# Install Appium Doctor (checks setup)
npm install -g appium-doctor
appium-doctor --android
appium-doctor --ios

# Install drivers
appium driver install uiautomator2  # Android
appium driver install xcuitest      # iOS

# Start Appium Server
appium --address 127.0.0.1 --port 4723

# Or with logs
appium --address 127.0.0.1 --port 4723 --log appium.log
```

### Appium Java Setup

```xml
<!-- pom.xml -->
<dependencies>
    <!-- Appium Java Client -->
    <dependency>
        <groupId>io.appium</groupId>
        <artifactId>java-client</artifactId>
        <version>8.6.0</version>
    </dependency>
    
    <!-- Selenium -->
    <dependency>
        <groupId>org.seleniumhq.selenium</groupId>
        <artifactId>selenium-java</artifactId>
        <version>4.15.0</version>
    </dependency>
    
    <!-- TestNG -->
    <dependency>
        <groupId>org.testng</groupId>
        <artifactId>testng</artifactId>
        <version>7.8.0</version>
    </dependency>
</dependencies>
```

### Basic Android Test

```java
import io.appium.java_client.android.AndroidDriver;
import io.appium.java_client.android.options.UiAutomator2Options;
import org.openqa.selenium.By;
import org.openqa.selenium.WebElement;
import org.testng.annotations.*;
import java.net.MalformedURLException;
import java.net.URL;
import java.time.Duration;

public class AndroidBasicTest {
    
    private AndroidDriver driver;
    
    @BeforeClass
    public void setup() throws MalformedURLException {
        // Desired Capabilities
        UiAutomator2Options options = new UiAutomator2Options();
        
        // Device details
        options.setDeviceName("Pixel_6_API_33");
        options.setPlatformName("Android");
        options.setPlatformVersion("13.0");
        
        // App details
        options.setApp("/path/to/your/app.apk");
        // OR use app package and activity for installed apps
        // options.setAppPackage("com.example.app");
        // options.setAppActivity("com.example.app.MainActivity");
        
        // Additional capabilities
        options.setAutomationName("UiAutomator2");
        options.setNoReset(false);
        options.setFullReset(false);
        
        // Timeouts
        options.setNewCommandTimeout(Duration.ofSeconds(300));
        
        // Initialize driver
        driver = new AndroidDriver(
            new URL("http://127.0.0.1:4723"), 
            options
        );
        
        driver.manage().timeouts().implicitlyWait(Duration.ofSeconds(10));
    }
    
    @Test
    public void testLogin() {
        // Find elements and interact
        WebElement usernameField = driver.findElement(
            By.xpath("//android.widget.EditText[@resource-id='username']")
        );
        usernameField.sendKeys("testuser");
        
        WebElement passwordField = driver.findElement(
            By.xpath("//android.widget.EditText[@resource-id='password']")
        );
        passwordField.sendKeys("password123");
        
        WebElement loginButton = driver.findElement(
            By.xpath("//android.widget.Button[@text='Login']")
        );
        loginButton.click();
        
        // Verify successful login
        WebElement welcomeText = driver.findElement(
            By.xpath("//android.widget.TextView[contains(@text,'Welcome')]")
        );
        Assert.assertTrue(welcomeText.isDisplayed());
    }
    
    @AfterClass
    public void teardown() {
        if (driver != null) {
            driver.quit();
        }
    }
}
```

### Basic iOS Test

```java
import io.appium.java_client.ios.IOSDriver;
import io.appium.java_client.ios.options.XCUITestOptions;
import org.testng.annotations.*;
import java.net.URL;
import java.time.Duration;

public class IOSBasicTest {
    
    private IOSDriver driver;
    
    @BeforeClass
    public void setup() throws Exception {
        XCUITestOptions options = new XCUITestOptions();
        
        // Device details
        options.setDeviceName("iPhone 14");
        options.setPlatformName("iOS");
        options.setPlatformVersion("16.0");
        
        // App details
        options.setApp("/path/to/your/app.ipa");
        // OR bundle ID for installed apps
        // options.setBundleId("com.example.app");
        
        // Additional capabilities
        options.setAutomationName("XCUITest");
        options.setNoReset(false);
        
        // UDID for real device
        // options.setUdid("your-device-udid");
        
        // Initialize driver
        driver = new IOSDriver(
            new URL("http://127.0.0.1:4723"),
            options
        );
        
        driver.manage().timeouts().implicitlyWait(Duration.ofSeconds(10));
    }
    
    @Test
    public void testLogin() {
        // iOS element interaction
        driver.findElement(By.xpath("//XCUIElementTypeTextField[@name='username']"))
              .sendKeys("testuser");
        
        driver.findElement(By.xpath("//XCUIElementTypeSecureTextField[@name='password']"))
              .sendKeys("password123");
        
        driver.findElement(By.xpath("//XCUIElementTypeButton[@name='Login']"))
              .click();
        
        // Verification
        WebElement welcome = driver.findElement(
            By.xpath("//XCUIElementTypeStaticText[contains(@name,'Welcome')]")
        );
        Assert.assertTrue(welcome.isDisplayed());
    }
    
    @AfterClass
    public void teardown() {
        if (driver != null) {
            driver.quit();
        }
    }
}
```

---

## iOS vs Android Testing

### Element Locator Strategies

```java
public class MobileLocators {
    
    // Android Locators
    public void androidLocators(AndroidDriver driver) {
        // By ID (resource-id)
        driver.findElement(By.id("com.example.app:id/username"));
        
        // By Accessibility ID (content-desc)
        driver.findElement(By.xpath("//*[@content-desc='Username']"));
        
        // By XPath
        driver.findElement(By.xpath("//android.widget.EditText[@text='Username']"));
        
        // By Class Name
        driver.findElement(By.className("android.widget.EditText"));
        
        // By UiAutomator (Android-specific)
        driver.findElement(MobileBy.androidUIAutomator(
            "new UiSelector().text(\"Login\")"
        ));
        
        // By UiAutomator with scrolling
        driver.findElement(MobileBy.androidUIAutomator(
            "new UiScrollable(new UiSelector().scrollable(true))" +
            ".scrollIntoView(new UiSelector().text(\"Settings\"))"
        ));
    }
    
    // iOS Locators
    public void iosLocators(IOSDriver driver) {
        // By ID (accessibility id)
        driver.findElement(By.id("username"));
        
        // By Name (name attribute)
        driver.findElement(By.name("Username"));
        
        // By XPath
        driver.findElement(By.xpath("//XCUIElementTypeTextField[@name='username']"));
        
        // By Class Name
        driver.findElement(By.className("XCUIElementTypeTextField"));
        
        // By iOS Predicate (iOS-specific)
        driver.findElement(MobileBy.iOSNsPredicateString(
            "name == 'username' AND visible == 1"
        ));
        
        // By iOS Class Chain (faster than XPath)
        driver.findElement(MobileBy.iOSClassChain(
            "**/XCUIElementTypeTextField[`name == 'username'`]"
        ));
    }
}
```

### Platform-Specific Handling

```java
public class PlatformSpecificActions {
    
    private AppiumDriver driver;
    
    public void hideKeyboard() {
        if (driver instanceof AndroidDriver) {
            ((AndroidDriver) driver).hideKeyboard();
        } else if (driver instanceof IOSDriver) {
            // iOS: Tap outside keyboard or use Done button
            driver.findElement(By.xpath("//XCUIElementTypeButton[@name='Done']"))
                  .click();
        }
    }
    
    public void swipeToElement(String elementText) {
        if (driver instanceof AndroidDriver) {
            // Android UiAutomator scroll
            driver.findElement(MobileBy.androidUIAutomator(
                "new UiScrollable(new UiSelector().scrollable(true))" +
                ".scrollIntoView(new UiSelector().text(\"" + elementText + "\"))"
            ));
        } else if (driver instanceof IOSDriver) {
            // iOS scroll using mobile:scroll
            Map<String, Object> params = new HashMap<>();
            params.put("direction", "down");
            params.put("name", elementText);
            ((IOSDriver) driver).executeScript("mobile: scroll", params);
        }
    }
    
    public void acceptAlert() {
        if (driver instanceof AndroidDriver) {
            driver.switchTo().alert().accept();
        } else if (driver instanceof IOSDriver) {
            driver.findElement(By.xpath("//XCUIElementTypeButton[@name='Allow']"))
                  .click();
        }
    }
}
```

---

## Mobile Gestures & Interactions

### Touch Actions (Appium 1.x - Legacy)

```java
import io.appium.java_client.TouchAction;
import io.appium.java_client.touch.WaitOptions;
import io.appium.java_client.touch.offset.PointOption;
import java.time.Duration;

public class TouchActionsLegacy {
    private AppiumDriver driver;
    
    // Tap
    public void tapElement(WebElement element) {
        new TouchAction(driver)
            .tap(tapOptions().withElement(element(element)))
            .perform();
    }
    
    // Long Press
    public void longPress(WebElement element) {
        new TouchAction(driver)
            .longPress(longPressOptions()
                .withElement(element(element))
                .withDuration(Duration.ofSeconds(2)))
            .release()
            .perform();
    }
    
    // Swipe
    public void swipeScreen(int startX, int startY, int endX, int endY) {
        new TouchAction(driver)
            .press(point(startX, startY))
            .waitAction(waitOptions(Duration.ofMillis(1000)))
            .moveTo(point(endX, endY))
            .release()
            .perform();
    }
    
    // Scroll Down
    public void scrollDown() {
        Dimension size = driver.manage().window().getSize();
        int startX = size.width / 2;
        int startY = (int) (size.height * 0.8);
        int endY = (int) (size.height * 0.2);
        
        swipeScreen(startX, startY, startX, endY);
    }
}
```

### W3C Actions (Appium 2.x - Current)

```java
import org.openqa.selenium.Point;
import org.openqa.selenium.interactions.Pause;
import org.openqa.selenium.interactions.PointerInput;
import org.openqa.selenium.interactions.Sequence;
import java.time.Duration;
import java.util.Arrays;

public class W3CActions {
    private AppiumDriver driver;
    
    // Tap
    public void tap(WebElement element) {
        Point location = element.getLocation();
        PointerInput finger = new PointerInput(PointerInput.Kind.TOUCH, "finger");
        
        Sequence tap = new Sequence(finger, 0);
        tap.addAction(finger.createPointerMove(Duration.ZERO,
            PointerInput.Origin.viewport(), location.x, location.y));
        tap.addAction(finger.createPointerDown(PointerInput.MouseButton.LEFT.asArg()));
        tap.addAction(new Pause(finger, Duration.ofMillis(100)));
        tap.addAction(finger.createPointerUp(PointerInput.MouseButton.LEFT.asArg()));
        
        driver.perform(Arrays.asList(tap));
    }
    
    // Swipe
    public void swipe(int startX, int startY, int endX, int endY, Duration duration) {
        PointerInput finger = new PointerInput(PointerInput.Kind.TOUCH, "finger");
        
        Sequence swipe = new Sequence(finger, 0);
        swipe.addAction(finger.createPointerMove(Duration.ZERO,
            PointerInput.Origin.viewport(), startX, startY));
        swipe.addAction(finger.createPointerDown(PointerInput.MouseButton.LEFT.asArg()));
        swipe.addAction(new Pause(finger, Duration.ofMillis(100)));
        swipe.addAction(finger.createPointerMove(duration,
            PointerInput.Origin.viewport(), endX, endY));
        swipe.addAction(finger.createPointerUp(PointerInput.MouseButton.LEFT.asArg()));
        
        driver.perform(Arrays.asList(swipe));
    }
    
    // Scroll Down
    public void scrollDown() {
        Dimension size = driver.manage().window().getSize();
        int startX = size.width / 2;
        int startY = (int) (size.height * 0.8);
        int endY = (int) (size.height * 0.2);
        
        swipe(startX, startY, startX, endY, Duration.ofMillis(800));
    }
    
    // Scroll Up
    public void scrollUp() {
        Dimension size = driver.manage().window().getSize();
        int startX = size.width / 2;
        int startY = (int) (size.height * 0.2);
        int endY = (int) (size.height * 0.8);
        
        swipe(startX, startY, startX, endY, Duration.ofMillis(800));
    }
    
    // Horizontal Swipe (for carousels)
    public void swipeLeft() {
        Dimension size = driver.manage().window().getSize();
        int startX = (int) (size.width * 0.8);
        int endX = (int) (size.width * 0.2);
        int y = size.height / 2;
        
        swipe(startX, y, endX, y, Duration.ofMillis(500));
    }
    
    public void swipeRight() {
        Dimension size = driver.manage().window().getSize();
        int startX = (int) (size.width * 0.2);
        int endX = (int) (size.width * 0.8);
        int y = size.height / 2;
        
        swipe(startX, y, endX, y, Duration.ofMillis(500));
    }
    
    // Zoom In (Pinch Out)
    public void zoomIn(WebElement element) {
        Point center = element.getLocation();
        int centerX = center.x + element.getSize().width / 2;
        int centerY = center.y + element.getSize().height / 2;
        
        PointerInput finger1 = new PointerInput(PointerInput.Kind.TOUCH, "finger1");
        PointerInput finger2 = new PointerInput(PointerInput.Kind.TOUCH, "finger2");
        
        Sequence finger1Sequence = new Sequence(finger1, 0);
        Sequence finger2Sequence = new Sequence(finger2, 0);
        
        // Start positions (close together)
        finger1Sequence.addAction(finger1.createPointerMove(Duration.ZERO,
            PointerInput.Origin.viewport(), centerX - 50, centerY));
        finger2Sequence.addAction(finger2.createPointerMove(Duration.ZERO,
            PointerInput.Origin.viewport(), centerX + 50, centerY));
        
        // Press down
        finger1Sequence.addAction(finger1.createPointerDown(PointerInput.MouseButton.LEFT.asArg()));
        finger2Sequence.addAction(finger2.createPointerDown(PointerInput.MouseButton.LEFT.asArg()));
        
        // Move apart (zoom in)
        finger1Sequence.addAction(finger1.createPointerMove(Duration.ofMillis(500),
            PointerInput.Origin.viewport(), centerX - 150, centerY));
        finger2Sequence.addAction(finger2.createPointerMove(Duration.ofMillis(500),
            PointerInput.Origin.viewport(), centerX + 150, centerY));
        
        // Release
        finger1Sequence.addAction(finger1.createPointerUp(PointerInput.MouseButton.LEFT.asArg()));
        finger2Sequence.addAction(finger2.createPointerUp(PointerInput.MouseButton.LEFT.asArg()));
        
        driver.perform(Arrays.asList(finger1Sequence, finger2Sequence));
    }
    
    // Zoom Out (Pinch In)
    public void zoomOut(WebElement element) {
        Point center = element.getLocation();
        int centerX = center.x + element.getSize().width / 2;
        int centerY = center.y + element.getSize().height / 2;
        
        PointerInput finger1 = new PointerInput(PointerInput.Kind.TOUCH, "finger1");
        PointerInput finger2 = new PointerInput(PointerInput.Kind.TOUCH, "finger2");
        
        Sequence finger1Sequence = new Sequence(finger1, 0);
        Sequence finger2Sequence = new Sequence(finger2, 0);
        
        // Start positions (far apart)
        finger1Sequence.addAction(finger1.createPointerMove(Duration.ZERO,
            PointerInput.Origin.viewport(), centerX - 150, centerY));
        finger2Sequence.addAction(finger2.createPointerMove(Duration.ZERO,
            PointerInput.Origin.viewport(), centerX + 150, centerY));
        
        // Press down
        finger1Sequence.addAction(finger1.createPointerDown(PointerInput.MouseButton.LEFT.asArg()));
        finger2Sequence.addAction(finger2.createPointerDown(PointerInput.MouseButton.LEFT.asArg()));
        
        // Move together (zoom out)
        finger1Sequence.addAction(finger1.createPointerMove(Duration.ofMillis(500),
            PointerInput.Origin.viewport(), centerX - 50, centerY));
        finger2Sequence.addAction(finger2.createPointerMove(Duration.ofMillis(500),
            PointerInput.Origin.viewport(), centerX + 50, centerY));
        
        // Release
        finger1Sequence.addAction(finger1.createPointerUp(PointerInput.MouseButton.LEFT.asArg()));
        finger2Sequence.addAction(finger2.createPointerUp(PointerInput.MouseButton.LEFT.asArg()));
        
        driver.perform(Arrays.asList(finger1Sequence, finger2Sequence));
    }
    
    // Drag and Drop
    public void dragAndDrop(WebElement source, WebElement target) {
        Point sourceLocation = source.getLocation();
        Point targetLocation = target.getLocation();
        
        PointerInput finger = new PointerInput(PointerInput.Kind.TOUCH, "finger");
        
        Sequence dragDrop = new Sequence(finger, 0);
        dragDrop.addAction(finger.createPointerMove(Duration.ZERO,
            PointerInput.Origin.viewport(), 
            sourceLocation.x, sourceLocation.y));
        dragDrop.addAction(finger.createPointerDown(PointerInput.MouseButton.LEFT.asArg()));
        dragDrop.addAction(new Pause(finger, Duration.ofMillis(500)));
        dragDrop.addAction(finger.createPointerMove(Duration.ofMillis(1000),
            PointerInput.Origin.viewport(), 
            targetLocation.x, targetLocation.y));
        dragDrop.addAction(finger.createPointerUp(PointerInput.MouseButton.LEFT.asArg()));
        
        driver.perform(Arrays.asList(dragDrop));
    }
}
```

---

## Mobile-Specific Challenges

### 1. Handling App Permissions

```java
public class PermissionHandler {
    
    // Android Permissions
    public void handleAndroidPermissions(AndroidDriver driver) {
        try {
            // Wait for permission dialog
            WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(5));
            
            // Allow location permission
            WebElement allowButton = wait.until(ExpectedConditions.presenceOfElementLocated(
                By.xpath("//android.widget.Button[@text='Allow' or @text='ALLOW']")
            ));
            allowButton.click();
            
        } catch (TimeoutException e) {
            System.out.println("Permission dialog did not appear");
        }
    }
    
    // iOS Permissions
    public void handleIOSPermissions(IOSDriver driver) {
        try {
            WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(5));
            
            // Allow location permission
            WebElement allowButton = wait.until(ExpectedConditions.presenceOfElementLocated(
                By.xpath("//XCUIElementTypeButton[@name='Allow']")
            ));
            allowButton.click();
            
        } catch (TimeoutException e) {
            System.out.println("Permission dialog did not appear");
        }
    }
    
    // Auto-grant permissions (Android)
    public void autoGrantPermissions() {
        UiAutomator2Options options = new UiAutomator2Options();
        options.setAutoGrantPermissions(true);
        // Use this option when initializing driver
    }
}
```

### 2. Handling Network Conditions

```java
public class NetworkTesting {
    
    // Enable Airplane Mode (Android)
    public void enableAirplaneMode(AndroidDriver driver) {
        driver.setConnection(new ConnectionStateBuilder()
            .withAirplaneModeEnabled()
            .build());
    }
    
    // Disable Airplane Mode
    public void disableAirplaneMode(AndroidDriver driver) {
        driver.setConnection(new ConnectionStateBuilder()
            .withWiFiEnabled()
            .withDataEnabled()
            .build());
    }
    
    // Test offline functionality
    @Test
    public void testOfflineMode() {
        // Enable airplane mode
        enableAirplaneMode((AndroidDriver) driver);
        
        // Try to access content
        // Should show cached content or offline message
        
        // Verify offline behavior
        WebElement offlineMessage = driver.findElement(
            By.xpath("//*[contains(@text,'No internet connection')]")
        );
        Assert.assertTrue(offlineMessage.isDisplayed());
        
        // Restore connection
        disableAirplaneMode((AndroidDriver) driver);
    }
}
```

### 3. Handling App States

```java
public class AppStateManagement {
    private AppiumDriver driver;
    
    // Background app
    public void backgroundApp(int seconds) {
        driver.runAppInBackground(Duration.ofSeconds(seconds));
    }
    
    // Close and reopen app
    public void closeAndReopenApp() {
        driver.closeApp();
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        driver.launchApp();
    }
    
    // Test app resume
    @Test
    public void testAppResume() {
        // Perform some action
        driver.findElement(By.id("search")).sendKeys("test");
        
        // Background app for 5 seconds
        backgroundApp(5);
        
        // Verify state is maintained
        WebElement searchField = driver.findElement(By.id("search"));
        Assert.assertEquals(searchField.getText(), "test",
            "Search text should be maintained after backgrounding");
    }
    
    // Test app interruption (incoming call simulation)
    @Test
    public void testIncomingCallInterruption() {
        // This requires additional tools like Xcode Simulator
        // or Android Debug Bridge (adb)
        
        // For Android:
        Runtime.getRuntime().exec("adb shell am start -a android.intent.action.CALL -d tel:1234567890");
        
        // Wait and verify app behavior
        // App should handle gracefully
    }
}
```

### 4. Handling Toasts and Notifications

```java
public class ToastHandler {
    
    // Android Toast
    public String getToastMessage(AndroidDriver driver) {
        WebElement toast = driver.findElement(
            By.xpath("//android.widget.Toast[1]")
        );
        return toast.getAttribute("name");
    }
    
    // Verify toast appears
    @Test
    public void testToastMessage() {
        driver.findElement(By.id("showToastBtn")).click();
        
        WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(5));
        WebElement toast = wait.until(ExpectedConditions.presenceOfElementLocated(
            By.xpath("//android.widget.Toast[1]")
        ));
        
        String toastText = toast.getAttribute("name");
        Assert.assertEquals(toastText, "Success!",
            "Toast message should be 'Success!'");
    }
    
    // iOS doesn't have toasts, uses alerts/banners
    public void handleIOSNotification(IOSDriver driver) {
        // Pull down notification center
        // This requires coordinates specific to device
    }
}
```

---

## Mobile CI/CD

### Jenkins Pipeline for Mobile Testing

```groovy
pipeline {
    agent any
    
    environment {
        ANDROID_HOME = '/Users/jenkins/Library/Android/sdk'
        JAVA_HOME = '/Library/Java/JavaVirtualMachines/jdk-11.jdk/Contents/Home'
    }
    
    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/your-repo/mobile-tests.git'
            }
        }
        
        stage('Start Appium Server') {
            steps {
                sh '''
                    appium --address 127.0.0.1 --port 4723 > appium.log 2>&1 &
                    sleep 5
                '''
            }
        }
        
        stage('Start Android Emulator') {
            steps {
                sh '''
                    $ANDROID_HOME/emulator/emulator -avd Pixel_6_API_33 -no-audio -no-window &
                    $ANDROID_HOME/platform-tools/adb wait-for-device
                    sleep 30
                '''
            }
        }
        
        stage('Run Tests') {
            steps {
                sh 'mvn clean test -Dplatform=android'
            }
        }
        
        stage('Generate Report') {
            steps {
                publishHTML([
                    reportDir: 'test-output',
                    reportFiles: 'index.html',
                    reportName: 'Test Report'
                ])
            }
        }
    }
    
    post {
        always {
            sh '''
                pkill -f appium
                $ANDROID_HOME/platform-tools/adb -s emulator-5554 emu kill
            '''
            
            archiveArtifacts artifacts: 'screenshots/**/*.png', 
                           allowEmptyArchive: true
        }
    }
}
```

### Cloud Testing (BrowserStack/Sauce Labs)

```java
public class BrowserStackTest {
    private AndroidDriver driver;
    
    @BeforeClass
    public void setup() throws MalformedURLException {
        DesiredCapabilities caps = new DesiredCapabilities();
        
        // BrowserStack Hub
        String userName = "YOUR_USERNAME";
        String accessKey = "YOUR_ACCESS_KEY";
        String hub = "https://" + userName + ":" + accessKey + 
                     "@hub-cloud.browserstack.com/wd/hub";
        
        // BrowserStack capabilities
        caps.setCapability("browserstack.user", userName);
        caps.setCapability("browserstack.key", accessKey);
        
        // Device capabilities
        caps.setCapability("device", "Samsung Galaxy S22");
        caps.setCapability("os_version", "12.0");
        caps.setCapability("project", "Mobile Test Project");
        caps.setCapability("build", "Build 1.0");
        caps.setCapability("name", "Login Test");
        
        // App capabilities
        caps.setCapability("app", "bs://your-app-id");
        
        driver = new AndroidDriver(new URL(hub), caps);
    }
    
    @Test
    public void testOnRealDevice() {
        // Your test code
        driver.findElement(By.id("username")).sendKeys("testuser");
        // ...
    }
    
    @AfterClass
    public void teardown() {
        if (driver != null) {
            driver.quit();
        }
    }
}
```

---

## Interview Questions

### Q1: What is the difference between Emulator and Real Device testing?

**Answer:**

```markdown
| Aspect | Emulator/Simulator | Real Device |
|--------|-------------------|-------------|
| **Cost** | Free | Expensive |
| **Performance** | Slower | Real performance |
| **Hardware** | Simulated | Actual sensors |
| **Network** | Simulated | Real network |
| **Camera** | Limited | Actual camera |
| **Battery** | Not testable | Can test battery usage |
| **Gestures** | Mouse clicks | Actual touch |
| **Availability** | Always available | Limited |
| **CI/CD** | Easy to integrate | Difficult |
| **Use Case** | Development, Functional | Final validation, Performance |
```

**Recommendation:** Use emulators for development and quick feedback, real devices for final validation.

### Q2: How do you handle dynamic elements in mobile apps?

**Answer:**

```java
public class DynamicElementHandling {
    
    // Use explicit waits
    public WebElement waitForElement(By locator) {
        WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(20));
        return wait.until(ExpectedConditions.presenceOfElementLocated(locator));
    }
    
    // Use flexible locators
    public WebElement findDynamicElement() {
        // Instead of exact text match
        // Use contains or starts-with
        return driver.findElement(By.xpath(
            "//android.widget.TextView[contains(@text,'Welcome')]"
        ));
    }
    
    // Retry mechanism
    public WebElement findWithRetry(By locator, int maxAttempts) {
        for (int i = 0; i < maxAttempts; i++) {
            try {
                return driver.findElement(locator);
            } catch (NoSuchElementException e) {
                if (i == maxAttempts - 1) throw e;
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException ie) {
                    Thread.currentThread().interrupt();
                }
            }
        }
        return null;
    }
}
```

### Q3: How do you test an app across different screen sizes?

**Answer:**

```java
public class MultiDeviceTesting {
    
    @DataProvider(name = "devices")
    public Object[][] getDevices() {
        return new Object[][] {
            {"Pixel 6", "Android", "13.0"},
            {"Samsung Galaxy S21", "Android", "11.0"},
            {"iPhone 14", "iOS", "16.0"},
            {"iPad Pro", "iOS", "16.0"}
        };
    }
    
    @Test(dataProvider = "devices")
    public void testOnMultipleDevices(String device, String platform, String version) {
        // Initialize driver for specific device
        // Run tests
        // Verify layout adapts properly
    }
    
    // Verify responsive layout
    public void verifyLayout() {
        Dimension screenSize = driver.manage().window().getSize();
        
        // Verify elements are visible
        WebElement header = driver.findElement(By.id("header"));
        Assert.assertTrue(header.isDisplayed());
        
        // Verify element positions
        Point headerLocation = header.getLocation();
        Assert.assertTrue(headerLocation.y < 100, 
            "Header should be at top of screen");
    }
}
```

---

**Next:** [Accessibility Testing](18-accessibility-testing.md)

