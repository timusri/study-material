# 18. Accessibility Testing (A11y)

## Table of Contents
- [Why Accessibility Testing Matters](#why-accessibility-testing-matters)
- [WCAG Guidelines](#wcag-guidelines)
- [Automated Accessibility Testing](#automated-accessibility-testing)
- [Manual Accessibility Testing](#manual-accessibility-testing)
- [Screen Reader Testing](#screen-reader-testing)
- [Keyboard Navigation Testing](#keyboard-navigation-testing)
- [Tools and Integration](#tools-and-integration)

---

## Why Accessibility Testing Matters

### Legal and Business Reasons

```markdown
## Legal Compliance

**Laws & Regulations:**
- **ADA (Americans with Disabilities Act)** - US law
- **Section 508** - Federal agencies in US
- **EAA (European Accessibility Act)** - EU law
- **AODA (Accessibility for Ontarians with Disabilities Act)** - Canada

**Legal Risks:**
- Lawsuits and fines
- Loss of business
- Reputation damage

**Recent Lawsuits:**
- Domino's Pizza vs Robles (2019)
- Netflix (2012) - $755,000 settlement
- Target (2008) - $6 million settlement

## Business Benefits

**Broader Audience:**
- 15% of world population has some disability
- 1 billion people globally
- Growing aging population

**Better UX for Everyone:**
- Clear navigation helps all users
- Captions help in noisy environments
- Good contrast helps in bright sunlight

**SEO Benefits:**
- Alt text for images
- Proper heading structure
- Semantic HTML

**Technical Benefits:**
- Better code quality
- Improved maintainability
- Better test coverage
```

### Types of Disabilities

```markdown
## 1. Visual Impairments
- **Blindness** - Screen reader users
- **Low vision** - Need magnification
- **Color blindness** - Red-green, Blue-yellow
- **Light sensitivity** - Need dark mode

## 2. Motor/Physical Impairments
- Cannot use mouse
- Need keyboard navigation
- Voice control users
- Switch device users

## 3. Auditory Impairments
- Deaf - Need captions/transcripts
- Hard of hearing - Need volume control

## 4. Cognitive Impairments
- Dyslexia - Need clear fonts
- ADHD - Need simple layouts
- Memory issues - Need clear labels

## 5. Seizure Disorders
- Photosensitive epilepsy
- No flashing content > 3 times/second
```

---

## WCAG Guidelines

### WCAG 2.1 Levels

```markdown
## Level A (Minimum)
**Must meet for basic accessibility**

Examples:
- Non-text content has alt text
- Video has captions
- Content doesn't rely solely on color
- All functionality available from keyboard

## Level AA (Mid Range) ⭐
**Most commonly required level**
**Target for most websites/apps**

Examples:
- Minimum contrast ratio 4.5:1
- Captions for all videos
- Multiple ways to find pages
- Consistent navigation
- Visible focus indicator

## Level AAA (Highest)
**Not required for full sites**
**Consider for specific content**

Examples:
- Contrast ratio 7:1
- Sign language interpretation
- Enhanced visual presentation
```

### POUR Principles

```markdown
## 1. Perceivable
Users must be able to perceive the information

✅ **Text alternatives** for non-text content
✅ **Captions** and transcripts for multimedia
✅ **Adaptable** content structure
✅ **Distinguishable** - enough contrast

## 2. Operable
Users must be able to operate the interface

✅ **Keyboard accessible** - all functionality
✅ **Enough time** to read content
✅ **Seizures** - no flashing content
✅ **Navigable** - clear navigation

## 3. Understandable
Users must understand the information

✅ **Readable** - clear language
✅ **Predictable** - consistent behavior
✅ **Input assistance** - error prevention and correction

## 4. Robust
Content must work with assistive technologies

✅ **Compatible** with current and future tools
✅ **Valid HTML**
✅ **Proper ARIA** usage
```

---

## Automated Accessibility Testing

### Axe-core Integration with Selenium

```xml
<!-- pom.xml -->
<dependencies>
    <!-- Selenium -->
    <dependency>
        <groupId>org.seleniumhq.selenium</groupId>
        <artifactId>selenium-java</artifactId>
        <version>4.15.0</version>
    </dependency>
    
    <!-- Axe Selenium Integration -->
    <dependency>
        <groupId>com.deque.html.axe-core</groupId>
        <artifactId>selenium</artifactId>
        <version>4.8.0</version>
    </dependency>
    
    <!-- TestNG -->
    <dependency>
        <groupId>org.testng</groupId>
        <artifactId>testng</artifactId>
        <version>7.8.0</version>
    </dependency>
</dependencies>
```

### Basic Axe Test

```java
import com.deque.html.axecore.results.Results;
import com.deque.html.axecore.results.Rule;
import com.deque.html.axecore.selenium.AxeBuilder;
import com.deque.html.axecore.selenium.AxeReporter;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.testng.Assert;
import org.testng.annotations.*;

public class BasicAccessibilityTest {
    
    private WebDriver driver;
    
    @BeforeClass
    public void setup() {
        driver = new ChromeDriver();
    }
    
    @Test
    public void testAccessibility() {
        driver.get("https://example.com");
        
        // Run axe accessibility check
        Results results = new AxeBuilder().analyze(driver);
        
        // Get violations
        List<Rule> violations = results.getViolations();
        
        if (!violations.isEmpty()) {
            // Generate detailed report
            AxeReporter.writeResultsToJsonFile(
                "accessibility-report", 
                results
            );
            
            // Print violations
            System.out.println("Found " + violations.size() + " accessibility violations:");
            for (Rule violation : violations) {
                System.out.println("- " + violation.getId() + ": " + 
                                   violation.getDescription());
                System.out.println("  Impact: " + violation.getImpact());
                System.out.println("  Help: " + violation.getHelpUrl());
            }
        }
        
        // Assert no violations
        Assert.assertEquals(violations.size(), 0, 
            "Accessibility violations found!");
    }
    
    @AfterClass
    public void teardown() {
        if (driver != null) {
            driver.quit();
        }
    }
}
```

### Advanced Axe Configuration

```java
public class AdvancedAccessibilityTest {
    private WebDriver driver;
    
    @Test
    public void testWithOptions() {
        driver.get("https://example.com");
        
        // Configure axe
        Results results = new AxeBuilder()
            // Only check WCAG 2.1 Level AA
            .withTags(Arrays.asList("wcag2a", "wcag2aa", "wcag21a", "wcag21aa"))
            // Exclude specific elements
            .exclude("#advertisements")
            .exclude(".third-party-widget")
            // Only check specific elements
            // .include("#main-content")
            // Disable specific rules
            .disableRules(Arrays.asList("color-contrast"))
            // Set options
            .withOptions("{ runOnly: { type: 'tag', values: ['wcag2a', 'wcag2aa'] } }")
            .analyze(driver);
        
        processResults(results);
    }
    
    @Test
    public void testSpecificElement() {
        driver.get("https://example.com");
        
        // Test only a specific section
        Results results = new AxeBuilder()
            .include("#login-form")
            .analyze(driver);
        
        processResults(results);
    }
    
    @Test
    public void testMultiplePages() {
        String[] pages = {
            "https://example.com",
            "https://example.com/about",
            "https://example.com/contact"
        };
        
        Map<String, Results> allResults = new HashMap<>();
        
        for (String page : pages) {
            driver.get(page);
            Results results = new AxeBuilder().analyze(driver);
            allResults.put(page, results);
        }
        
        // Generate combined report
        generateCombinedReport(allResults);
    }
    
    private void processResults(Results results) {
        List<Rule> violations = results.getViolations();
        List<Rule> passes = results.getPasses();
        List<Rule> incomplete = results.getIncomplete();
        
        System.out.println("Passed checks: " + passes.size());
        System.out.println("Violations: " + violations.size());
        System.out.println("Incomplete: " + incomplete.size());
        
        // Group violations by impact
        Map<String, List<Rule>> violationsByImpact = violations.stream()
            .collect(Collectors.groupingBy(Rule::getImpact));
        
        System.out.println("\nViolations by impact:");
        violationsByImpact.forEach((impact, rules) -> {
            System.out.println(impact + ": " + rules.size());
            rules.forEach(rule -> 
                System.out.println("  - " + rule.getId() + ": " + 
                                   rule.getDescription())
            );
        });
    }
    
    private void generateCombinedReport(Map<String, Results> allResults) {
        int totalViolations = 0;
        
        System.out.println("=== Accessibility Report for All Pages ===\n");
        
        for (Map.Entry<String, Results> entry : allResults.entrySet()) {
            String page = entry.getKey();
            Results results = entry.getValue();
            int violations = results.getViolations().size();
            totalViolations += violations;
            
            System.out.println(page + ": " + violations + " violations");
        }
        
        System.out.println("\nTotal violations across all pages: " + totalViolations);
    }
}
```

### Custom Accessibility Test Framework

```java
public class AccessibilityTestFramework {
    
    @Target(ElementType.METHOD)
    @Retention(RetentionPolicy.RUNTIME)
    public @interface A11yTest {
        String[] wcagLevel() default {"wcag2a", "wcag2aa"};
        String[] excludeRules() default {};
        boolean failOnViolations() default true;
    }
    
    // Base test class
    public abstract class BaseA11yTest {
        protected WebDriver driver;
        
        @BeforeClass
        public void setup() {
            driver = new ChromeDriver();
        }
        
        protected void checkAccessibility() {
            // Get test method annotations
            Method method = getCurrentTestMethod();
            A11yTest annotation = method.getAnnotation(A11yTest.class);
            
            AxeBuilder builder = new AxeBuilder();
            
            if (annotation != null) {
                // Apply WCAG level
                builder.withTags(Arrays.asList(annotation.wcagLevel()));
                
                // Exclude rules
                if (annotation.excludeRules().length > 0) {
                    builder.disableRules(Arrays.asList(annotation.excludeRules()));
                }
            }
            
            Results results = builder.analyze(driver);
            
            // Report results
            reportResults(results);
            
            // Fail if configured
            if (annotation != null && annotation.failOnViolations()) {
                Assert.assertEquals(results.getViolations().size(), 0,
                    "Accessibility violations found");
            }
        }
        
        private void reportResults(Results results) {
            // Generate HTML report
            AccessibilityReporter.generateHtmlReport(results, 
                "target/a11y-reports/" + System.currentTimeMillis() + ".html");
        }
        
        @AfterClass
        public void teardown() {
            if (driver != null) {
                driver.quit();
            }
        }
    }
    
    // Usage
    public class LoginPageA11yTest extends BaseA11yTest {
        
        @Test
        @A11yTest(wcagLevel = {"wcag2a", "wcag2aa", "wcag21aa"})
        public void testLoginPageAccessibility() {
            driver.get("https://example.com/login");
            checkAccessibility();
        }
        
        @Test
        @A11yTest(
            wcagLevel = {"wcag2aaa"},
            excludeRules = {"color-contrast"},
            failOnViolations = false
        )
        public void testLoginPageAAA() {
            driver.get("https://example.com/login");
            checkAccessibility();
        }
    }
}
```

---

## Manual Accessibility Testing

### Keyboard Navigation Checklist

```java
public class KeyboardNavigationTest {
    private WebDriver driver;
    
    @Test
    public void testKeyboardNavigation() {
        driver.get("https://example.com");
        
        // Get the body element
        WebElement body = driver.findElement(By.tagName("body"));
        
        // Tab through all elements
        for (int i = 0; i < 20; i++) {
            body.sendKeys(Keys.TAB);
            
            // Get currently focused element
            WebElement focusedElement = driver.switchTo().activeElement();
            
            System.out.println("Focused on: " + focusedElement.getTagName() + 
                             " - " + focusedElement.getAttribute("id"));
            
            // Verify focus is visible
            String outline = focusedElement.getCssValue("outline");
            String border = focusedElement.getCssValue("border");
            
            Assert.assertTrue(
                !outline.equals("none") || !border.equals("none"),
                "Element should have visible focus indicator"
            );
        }
    }
    
    @Test
    public void testFormKeyboardAccessibility() {
        driver.get("https://example.com/form");
        
        WebElement body = driver.findElement(By.tagName("body"));
        
        // Tab to username field
        body.sendKeys(Keys.TAB);
        WebElement username = driver.switchTo().activeElement();
        username.sendKeys("testuser");
        
        // Tab to password field
        username.sendKeys(Keys.TAB);
        WebElement password = driver.switchTo().activeElement();
        password.sendKeys("password123");
        
        // Tab to submit button and press Enter
        password.sendKeys(Keys.TAB);
        WebElement submit = driver.switchTo().activeElement();
        submit.sendKeys(Keys.ENTER);
        
        // Verify form was submitted
        WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
        wait.until(ExpectedConditions.urlContains("dashboard"));
    }
    
    @Test
    public void testSkipLinks() {
        driver.get("https://example.com");
        
        WebElement body = driver.findElement(By.tagName("body"));
        
        // First Tab should reveal "Skip to main content" link
        body.sendKeys(Keys.TAB);
        
        WebElement skipLink = driver.switchTo().activeElement();
        Assert.assertTrue(
            skipLink.getText().toLowerCase().contains("skip"),
            "First focusable element should be skip link"
        );
        
        // Press Enter on skip link
        skipLink.sendKeys(Keys.ENTER);
        
        // Verify focus moved to main content
        WebElement focusedElement = driver.switchTo().activeElement();
        Assert.assertTrue(
            focusedElement.getAttribute("id").equals("main-content") ||
            focusedElement.getAttribute("role").equals("main"),
            "Focus should move to main content"
        );
    }
    
    @Test
    public void testModalKeyboardTrap() {
        driver.get("https://example.com");
        
        // Open modal
        driver.findElement(By.id("open-modal-btn")).click();
        
        // Verify focus is trapped in modal
        WebElement activeElement = driver.switchTo().activeElement();
        
        // Tab through modal elements
        for (int i = 0; i < 10; i++) {
            activeElement.sendKeys(Keys.TAB);
            activeElement = driver.switchTo().activeElement();
            
            // Verify element is within modal
            WebElement modal = driver.findElement(By.id("modal"));
            boolean isInModal = (boolean) ((JavascriptExecutor) driver)
                .executeScript("return arguments[0].contains(arguments[1])", 
                              modal, activeElement);
            
            Assert.assertTrue(isInModal, 
                "Focus should stay within modal (keyboard trap)");
        }
        
        // Press Escape to close modal
        activeElement.sendKeys(Keys.ESCAPE);
        
        // Verify modal closed
        List<WebElement> modals = driver.findElements(By.id("modal"));
        Assert.assertTrue(modals.isEmpty() || !modals.get(0).isDisplayed(),
            "Modal should close on Escape key");
    }
}
```

### Color Contrast Testing

```java
public class ColorContrastTest {
    
    @Test
    public void testColorContrast() {
        driver.get("https://example.com");
        
        // Get text element
        WebElement element = driver.findElement(By.id("main-heading"));
        
        // Get colors
        String color = element.getCssValue("color");
        String backgroundColor = element.getCssValue("background-color");
        
        // Convert to RGB
        Color textColor = Color.fromString(color);
        Color bgColor = Color.fromString(backgroundColor);
        
        // Calculate contrast ratio
        double contrastRatio = calculateContrastRatio(textColor, bgColor);
        
        // Get font size
        String fontSize = element.getCssValue("font-size");
        int fontSizeInt = Integer.parseInt(fontSize.replaceAll("[^0-9]", ""));
        
        // Check WCAG AA compliance
        if (fontSizeInt >= 18) {
            // Large text: ratio should be at least 3:1
            Assert.assertTrue(contrastRatio >= 3.0,
                "Large text contrast ratio should be at least 3:1, got: " + 
                contrastRatio);
        } else {
            // Normal text: ratio should be at least 4.5:1
            Assert.assertTrue(contrastRatio >= 4.5,
                "Normal text contrast ratio should be at least 4.5:1, got: " + 
                contrastRatio);
        }
        
        System.out.println("Contrast ratio: " + contrastRatio + ":1");
    }
    
    private double calculateContrastRatio(Color color1, Color color2) {
        double l1 = getRelativeLuminance(color1);
        double l2 = getRelativeLuminance(color2);
        
        double lighter = Math.max(l1, l2);
        double darker = Math.min(l1, l2);
        
        return (lighter + 0.05) / (darker + 0.05);
    }
    
    private double getRelativeLuminance(Color color) {
        double r = getRGBValue(color.getRed());
        double g = getRGBValue(color.getGreen());
        double b = getRGBValue(color.getBlue());
        
        return 0.2126 * r + 0.7152 * g + 0.0722 * b;
    }
    
    private double getRGBValue(int value) {
        double v = value / 255.0;
        if (v <= 0.03928) {
            return v / 12.92;
        }
        return Math.pow((v + 0.055) / 1.055, 2.4);
    }
}
```

---

## Screen Reader Testing

### ARIA Attributes Testing

```java
public class AriaAttributesTest {
    
    @Test
    public void testButtonHasAriaLabel() {
        driver.get("https://example.com");
        
        WebElement button = driver.findElement(By.id("close-btn"));
        
        // Check for accessible name
        String ariaLabel = button.getAttribute("aria-label");
        String ariaLabelledBy = button.getAttribute("aria-labelledby");
        String buttonText = button.getText();
        
        Assert.assertTrue(
            ariaLabel != null || ariaLabelledBy != null || !buttonText.isEmpty(),
            "Button must have accessible name (aria-label, aria-labelledby, or text)"
        );
    }
    
    @Test
    public void testFormLabels() {
        driver.get("https://example.com/form");
        
        List<WebElement> inputs = driver.findElements(
            By.cssSelector("input[type='text'], input[type='email'], input[type='password']")
        );
        
        for (WebElement input : inputs) {
            String id = input.getAttribute("id");
            String ariaLabel = input.getAttribute("aria-label");
            String ariaLabelledBy = input.getAttribute("aria-labelledby");
            
            // Check if input has associated label
            boolean hasLabel = false;
            
            if (id != null && !id.isEmpty()) {
                List<WebElement> labels = driver.findElements(
                    By.cssSelector("label[for='" + id + "']")
                );
                hasLabel = !labels.isEmpty();
            }
            
            Assert.assertTrue(
                hasLabel || ariaLabel != null || ariaLabelledBy != null,
                "Input field must have associated label"
            );
        }
    }
    
    @Test
    public void testHeadingStructure() {
        driver.get("https://example.com");
        
        // Get all headings
        List<WebElement> h1s = driver.findElements(By.tagName("h1"));
        List<WebElement> h2s = driver.findElements(By.tagName("h2"));
        List<WebElement> h3s = driver.findElements(By.tagName("h3"));
        
        // Should have exactly one h1
        Assert.assertEquals(h1s.size(), 1, "Page should have exactly one h1");
        
        // Verify heading hierarchy
        List<WebElement> allHeadings = driver.findElements(
            By.cssSelector("h1, h2, h3, h4, h5, h6")
        );
        
        int previousLevel = 0;
        for (WebElement heading : allHeadings) {
            String tagName = heading.getTagName();
            int currentLevel = Integer.parseInt(tagName.substring(1));
            
            // Heading level should not skip (h1 -> h3 is not allowed)
            Assert.assertTrue(
                currentLevel <= previousLevel + 1,
                "Heading hierarchy should not skip levels"
            );
            
            previousLevel = currentLevel;
        }
    }
    
    @Test
    public void testAriaLiveRegions() {
        driver.get("https://example.com");
        
        // Click button that updates live region
        driver.findElement(By.id("update-btn")).click();
        
        // Find live region
        WebElement liveRegion = driver.findElement(By.id("notifications"));
        
        String ariaLive = liveRegion.getAttribute("aria-live");
        String role = liveRegion.getAttribute("role");
        
        Assert.assertTrue(
            "polite".equals(ariaLive) || "assertive".equals(ariaLive) ||
            "alert".equals(role) || "status".equals(role),
            "Dynamic content updates should use aria-live or appropriate role"
        );
    }
    
    @Test
    public void testImageAltText() {
        driver.get("https://example.com");
        
        List<WebElement> images = driver.findElements(By.tagName("img"));
        
        for (WebElement image : images) {
            String alt = image.getAttribute("alt");
            String role = image.getAttribute("role");
            
            // Decorative images should have empty alt or role="presentation"
            // Content images should have descriptive alt text
            Assert.assertTrue(
                alt != null,
                "All images must have alt attribute (can be empty for decorative images)"
            );
            
            // Check if image is visible
            if (image.isDisplayed()) {
                boolean isDecorative = alt.isEmpty() || "presentation".equals(role);
                
                if (!isDecorative) {
                    Assert.assertTrue(
                        alt.length() > 0 && alt.length() < 125,
                        "Alt text should be descriptive but concise (1-125 characters)"
                    );
                }
            }
        }
    }
}
```

### Testing with Screen Reader Simulation

```java
public class ScreenReaderSimulation {
    
    @Test
    public void simulateScreenReaderNavigation() {
        driver.get("https://example.com");
        
        // Get all interactive elements in DOM order
        List<WebElement> interactiveElements = driver.findElements(
            By.cssSelector("a, button, input, select, textarea, [tabindex]")
        );
        
        System.out.println("=== Screen Reader would announce: ===\n");
        
        for (WebElement element : interactiveElements) {
            if (element.isDisplayed()) {
                String announcement = getScreenReaderAnnouncement(element);
                System.out.println(announcement);
            }
        }
    }
    
    private String getScreenReaderAnnouncement(WebElement element) {
        String tagName = element.getTagName();
        String role = element.getAttribute("role");
        String ariaLabel = element.getAttribute("aria-label");
        String text = element.getText();
        String title = element.getAttribute("title");
        String alt = element.getAttribute("alt");
        
        // Determine accessible name
        String accessibleName = ariaLabel != null ? ariaLabel :
                               text != null && !text.isEmpty() ? text :
                               alt != null ? alt :
                               title != null ? title : 
                               "[No accessible name]";
        
        // Determine role
        String elementRole = role != null ? role : 
                            getImplicitRole(tagName);
        
        // Build announcement
        String announcement = accessibleName + ", " + elementRole;
        
        // Add state information
        if ("true".equals(element.getAttribute("aria-expanded"))) {
            announcement += ", expanded";
        } else if ("false".equals(element.getAttribute("aria-expanded"))) {
            announcement += ", collapsed";
        }
        
        if ("true".equals(element.getAttribute("aria-checked"))) {
            announcement += ", checked";
        }
        
        if ("true".equals(element.getAttribute("aria-disabled")) || 
            !element.isEnabled()) {
            announcement += ", disabled";
        }
        
        return announcement;
    }
    
    private String getImplicitRole(String tagName) {
        switch (tagName.toLowerCase()) {
            case "a": return "link";
            case "button": return "button";
            case "input": return "text field";
            case "select": return "combo box";
            case "textarea": return "text area";
            case "h1": return "heading level 1";
            case "h2": return "heading level 2";
            case "h3": return "heading level 3";
            case "nav": return "navigation";
            case "main": return "main";
            case "header": return "banner";
            case "footer": return "contentinfo";
            default: return "element";
        }
    }
}
```

---

## Tools and Integration

### Pa11y Integration

```javascript
// pa11y-test.js
const pa11y = require('pa11y');
const htmlReporter = require('pa11y-reporter-html');

async function runAccessibilityTest(url) {
    try {
        const results = await pa11y(url, {
            standard: 'WCAG2AA',
            includeNotices: true,
            includeWarnings: true,
            timeout: 30000,
            wait: 1000,
            chromeLaunchConfig: {
                args: ['--no-sandbox']
            }
        });
        
        // Generate HTML report
        const html = await htmlReporter.results(results);
        require('fs').writeFileSync('pa11y-report.html', html);
        
        console.log(`Issues found: ${results.issues.length}`);
        
        results.issues.forEach(issue => {
            console.log(`${issue.type}: ${issue.message}`);
            console.log(`  Element: ${issue.selector}`);
            console.log(`  Code: ${issue.code}`);
        });
        
        process.exit(results.issues.length > 0 ? 1 : 0);
        
    } catch (error) {
        console.error('Error running pa11y:', error);
        process.exit(1);
    }
}

runAccessibilityTest('https://example.com');
```

### Lighthouse CI Integration

```yaml
# .lighthouserc.json
{
  "ci": {
    "collect": {
      "url": [
        "http://localhost:3000",
        "http://localhost:3000/about",
        "http://localhost:3000/contact"
      ],
      "numberOfRuns": 3,
      "settings": {
        "onlyCategories": ["accessibility"]
      }
    },
    "assert": {
      "assertions": {
        "categories:accessibility": ["error", {"minScore": 0.9}],
        "color-contrast": "off",
        "image-alt": "error",
        "label": "error",
        "button-name": "error"
      }
    },
    "upload": {
      "target": "temporary-public-storage"
    }
  }
}
```

```groovy
// Jenkins Pipeline
pipeline {
    agent any
    
    stages {
        stage('Build') {
            steps {
                sh 'npm install'
                sh 'npm run build'
            }
        }
        
        stage('Start Server') {
            steps {
                sh 'npm start &'
                sh 'sleep 10'
            }
        }
        
        stage('Accessibility Tests') {
            parallel {
                stage('Axe Tests') {
                    steps {
                        sh 'mvn test -Dtest=*A11y*'
                    }
                }
                
                stage('Lighthouse') {
                    steps {
                        sh 'npm install -g @lhci/cli'
                        sh 'lhci autorun'
                    }
                }
                
                stage('Pa11y') {
                    steps {
                        sh 'node pa11y-test.js'
                    }
                }
            }
        }
    }
    
    post {
        always {
            publishHTML([
                reportDir: 'target/a11y-reports',
                reportFiles: '*.html',
                reportName: 'Accessibility Report'
            ])
        }
    }
}
```

---

## Interview Questions

### Q1: What is the difference between accessibility and usability?

**Answer:**
```markdown
**Accessibility (A11y):**
- Ensures people with disabilities can use the product
- Legal requirement in many countries
- WCAG guidelines
- Screen readers, keyboard navigation
- Mandatory compliance

**Usability:**
- Ensures product is easy to use for everyone
- Best practices, not legal requirement
- UX research and testing
- Intuitive design
- Nice to have

**Example:**
A button might be accessible (has proper ARIA label, keyboard accessible) but not usable (too small, confusing label).

**Best Approach:** Aim for both!
```

### Q2: Explain ARIA and when to use it.

**Answer:**
```markdown
**ARIA (Accessible Rich Internet Applications)**

Provides additional semantics to assistive technologies.

**When to Use:**
1. Native HTML doesn't provide needed semantics
2. Custom widgets (tabs, accordions, modals)
3. Dynamic content updates
4. Single Page Applications

**When NOT to Use:**
- Native HTML elements already have semantics
- "No ARIA is better than Bad ARIA"

**Example:**

❌ **Bad (Unnecessary ARIA):**
```html
<button role="button" aria-label="Submit">Submit</button>
<!-- Button already has button role and visible text -->
```

✅ **Good (Necessary ARIA):**
```html
<div role="button" tabindex="0" aria-label="Close">
    <span class="icon-close"></span>
</div>
<!-- Div needs role, tabindex, and label -->
```

✅ **Better (Use Native HTML):**
```html
<button aria-label="Close">
    <span class="icon-close"></span>
</button>
```

### Q3: How do you test for keyboard accessibility?

**Answer:**
```markdown
**Manual Testing Checklist:**

1. ✅ **Tab Key:**
   - Tab through all interactive elements
   - Verify logical tab order
   - Check focus indicators are visible

2. ✅ **Enter/Space:**
   - Enter activates buttons and links
   - Space activates buttons and checkboxes

3. ✅ **Arrow Keys:**
   - Navigate within widgets (tabs, dropdown)
   - Scroll in list boxes

4. ✅ **Escape Key:**
   - Closes modals/dialogs
   - Cancels operations

5. ✅ **Skip Links:**
   - "Skip to main content" link present
   - Keyboard users can bypass navigation

6. ✅ **Focus Management:**
   - Focus moves to opened modals
   - Focus returns after closing modals
   - No keyboard traps (except modals)

**Automated Testing:**
```java
@Test
public void testKeyboardAccessibility() {
    driver.get("https://example.com");
    
    WebElement body = driver.findElement(By.tagName("body"));
    
    // Tab through elements
    for (int i = 0; i < 20; i++) {
        body.sendKeys(Keys.TAB);
        WebElement focused = driver.switchTo().activeElement();
        
        // Verify focus visible
        String outline = focused.getCssValue("outline");
        Assert.assertFalse(outline.contains("none"));
    }
}
```

---

**Next:** [Security Testing for SDET](19-security-testing.md)

