# 10. Agile & SDLC

## Table of Contents
- [Agile Methodologies](#agile-methodologies)
- [Scrum Framework](#scrum-framework)
- [Test-Driven Development (TDD)](#test-driven-development-tdd)
- [Behavior-Driven Development (BDD)](#behavior-driven-development-bdd)
- [Testing in Agile](#testing-in-agile)
- [Sprint Planning](#sprint-planning)

---

## Agile Methodologies

### Agile Principles

```markdown
## 12 Principles of Agile Manifesto

1. **Customer satisfaction** through early and continuous delivery
2. **Welcome changing requirements**, even late in development
3. **Deliver working software frequently** (weeks rather than months)
4. **Business and developers work together daily**
5. **Build projects around motivated individuals**
6. **Face-to-face conversation** is most effective
7. **Working software** is primary measure of progress
8. **Sustainable development** - maintain constant pace
9. **Technical excellence** and good design
10. **Simplicity** - maximize work not done
11. **Self-organizing teams** produce best results
12. **Regular reflection** and adaptation

## Agile Values

**Individuals and interactions** over processes and tools
**Working software** over comprehensive documentation
**Customer collaboration** over contract negotiation
**Responding to change** over following a plan
```

### Agile vs Waterfall

```
┌─────────────────────────────────────────────────────┐
│                    WATERFALL                        │
└─────────────────────────────────────────────────────┘

Requirements → Design → Development → Testing → Deployment
    (3 months)   (2 months)  (6 months)   (3 months)  (1 month)
    
    Pros: Clear structure, easy to manage
    Cons: Late testing, inflexible, high risk

┌─────────────────────────────────────────────────────┐
│                     AGILE                           │
└─────────────────────────────────────────────────────┘

Sprint 1 (2 weeks): Plan → Design → Dev → Test → Deploy
Sprint 2 (2 weeks): Plan → Design → Dev → Test → Deploy
Sprint 3 (2 weeks): Plan → Design → Dev → Test → Deploy
    ...continuous iterations...
    
    Pros: Early testing, flexible, continuous delivery
    Cons: Requires discipline, documentation may suffer
```

---

## Scrum Framework

### Scrum Roles

```markdown
## 1. Product Owner
**Responsibilities:**
- Define product backlog items
- Prioritize features
- Accept or reject work
- Maximize product value

**In Testing Context:**
- Defines acceptance criteria
- Reviews test results
- Approves test coverage
- Makes go/no-go decisions

## 2. Scrum Master
**Responsibilities:**
- Facilitate Scrum events
- Remove impediments
- Coach team on Agile practices
- Protect team from distractions

**In Testing Context:**
- Ensure testing is part of Definition of Done
- Facilitate test automation discussions
- Remove testing blockers
- Promote quality culture

## 3. Development Team
**Responsibilities:**
- Self-organizing
- Cross-functional
- Deliver potentially shippable increment
- Estimate work

**In Testing Context:**
- Developers write unit tests
- QA creates automated tests
- Everyone responsible for quality
- Participate in test planning
```

### Scrum Events

```markdown
## Sprint (2-4 weeks iteration)

### 1. Sprint Planning (4-8 hours for 2-week sprint)
**Purpose:** Plan work for upcoming sprint

**Agenda:**
- Review product backlog
- Select user stories for sprint
- Define sprint goal
- Break down stories into tasks
- Estimate effort

**Testing Activities:**
- Review acceptance criteria
- Identify test scenarios
- Estimate testing effort
- Plan automation tasks
- Setup test environment

### 2. Daily Standup (15 minutes)
**Purpose:** Synchronize team activities

**Three Questions:**
1. What did I do yesterday?
2. What will I do today?
3. Any impediments?

**QA Update Example:**
"Yesterday: Automated 5 test cases for login feature
Today: Will test payment integration
Blockers: Need test data for premium accounts"

### 3. Sprint Review (2-4 hours)
**Purpose:** Demo completed work to stakeholders

**Agenda:**
- Demo working software
- Review what was completed
- Discuss what wasn't completed
- Get feedback
- Update product backlog

**Testing Deliverables:**
- Test execution report
- Defect metrics
- Automation progress
- Risk assessment

### 4. Sprint Retrospective (1.5-3 hours)
**Purpose:** Inspect team process and improve

**Questions:**
- What went well?
- What didn't go well?
- What should we improve?

**Testing Improvements Example:**
- Automate more regression tests
- Improve test data management
- Earlier involvement in story refinement
- Better integration test coverage

### 5. Backlog Refinement (Ongoing)
**Purpose:** Prepare upcoming stories

**Activities:**
- Break down large stories
- Add acceptance criteria
- Estimate stories
- Clarify requirements

**QA Contribution:**
- Review testability
- Suggest test scenarios
- Identify testing risks
- Estimate testing effort
```

### Scrum Artifacts

```markdown
## 1. Product Backlog
**Prioritized list of features/requirements**

Example Format:
```
User Story: As a user, I want to filter products by price
Priority: High
Story Points: 5
Acceptance Criteria:
  - Price range slider visible on product page
  - Results update in real-time
  - URL reflects filter selection
  - Works on mobile devices
Test Scenarios:
  - Verify filter with min/max values
  - Test with no products in range
  - Verify URL parameters
  - Cross-browser testing
```

## 2. Sprint Backlog
**Selected stories + tasks for current sprint**

Example:
```
Sprint 5 Backlog
Goal: Complete user authentication module

User Story 1: Login functionality (8 pts)
  Tasks:
  - Design login page [Dev] - 4h
  - Implement authentication [Dev] - 8h
  - Write unit tests [Dev] - 4h
  - Create UI automation [QA] - 6h
  - API testing [QA] - 4h
  - Security testing [QA] - 4h

User Story 2: Password reset (5 pts)
  ...
```

## 3. Increment
**Potentially shippable product at sprint end**

**Definition of Done (DoD):**
- [ ] Code complete and reviewed
- [ ] Unit tests written and passing
- [ ] Integration tests passing
- [ ] UI automation complete
- [ ] No critical/blocker defects
- [ ] Documentation updated
- [ ] Acceptance criteria met
- [ ] Deployed to staging
- [ ] Product owner approved
```

---

## Test-Driven Development (TDD)

### TDD Cycle (Red-Green-Refactor)

```
┌──────────────────────────────────┐
│     TDD Cycle (Red-Green-Refactor)     │
└──────────────────────────────────┘

1. RED: Write failing test
     ↓
2. GREEN: Write minimal code to pass
     ↓
3. REFACTOR: Improve code quality
     ↓
   (Repeat)
```

### TDD Example

```java
// Step 1: Write failing test (RED)
@Test
public void testCalculateDiscount() {
    Product product = new Product("Laptop", 1000.00);
    double discount = product.calculateDiscount(10);
    assertEquals(100.00, discount, 0.01);
}

// Test fails because calculateDiscount() doesn't exist yet

// Step 2: Write minimal code to pass (GREEN)
public class Product {
    private String name;
    private double price;
    
    public Product(String name, double price) {
        this.name = name;
        this.price = price;
    }
    
    public double calculateDiscount(int percentage) {
        return price * percentage / 100;
    }
}

// Test now passes

// Step 3: Refactor (improve code)
public class Product {
    private static final int MAX_DISCOUNT_PERCENTAGE = 50;
    
    private String name;
    private double price;
    
    public Product(String name, double price) {
        validatePrice(price);
        this.name = name;
        this.price = price;
    }
    
    public double calculateDiscount(int percentage) {
        validateDiscountPercentage(percentage);
        return price * percentage / 100.0;
    }
    
    private void validatePrice(double price) {
        if (price < 0) {
            throw new IllegalArgumentException("Price cannot be negative");
        }
    }
    
    private void validateDiscountPercentage(int percentage) {
        if (percentage < 0 || percentage > MAX_DISCOUNT_PERCENTAGE) {
            throw new IllegalArgumentException(
                "Discount must be between 0 and " + MAX_DISCOUNT_PERCENTAGE);
        }
    }
}

// Add more tests for edge cases
@Test(expectedExceptions = IllegalArgumentException.class)
public void testInvalidDiscount() {
    Product product = new Product("Laptop", 1000.00);
    product.calculateDiscount(60); // Should throw exception
}

@Test(expectedExceptions = IllegalArgumentException.class)
public void testNegativePrice() {
    Product product = new Product("Laptop", -100.00);
}
```

### TDD Best Practices

```java
public class TDDGuidelines {
    
    /*
     * 1. Write test first, code second
     * 2. Test one thing at a time
     * 3. Keep tests simple and readable
     * 4. Use descriptive test names
     * 5. Follow AAA pattern (Arrange, Act, Assert)
     * 6. Test behavior, not implementation
     * 7. Keep tests independent
     * 8. Run tests frequently
     */
    
    // Good: Descriptive name, tests one thing
    @Test
    public void shouldCalculateDiscountCorrectlyForValidPercentage() {
        // Arrange
        Product product = new Product("Laptop", 1000.00);
        
        // Act
        double discount = product.calculateDiscount(10);
        
        // Assert
        assertEquals(100.00, discount, 0.01);
    }
    
    // Bad: Vague name, tests multiple things
    @Test
    public void test1() {
        Product product = new Product("Laptop", 1000.00);
        assertEquals(100.00, product.calculateDiscount(10), 0.01);
        assertEquals(200.00, product.calculateDiscount(20), 0.01);
        assertEquals(500.00, product.calculateDiscount(50), 0.01);
    }
}
```

---

## Behavior-Driven Development (BDD)

### BDD Approach

```markdown
## BDD Principles

**Goal:** Bridge communication gap between business and technical teams

**Format:** Given-When-Then (Gherkin syntax)

**Benefits:**
- Living documentation
- Shared understanding
- Executable specifications
- Focus on behavior, not implementation
```

### BDD Examples (Gherkin)

```gherkin
# Feature file: login.feature

Feature: User Login
  As a registered user
  I want to login to the system
  So that I can access my account

  Background:
    Given the user is on the login page

  Scenario: Successful login with valid credentials
    Given user has valid credentials
    When user enters username "john@example.com"
    And user enters password "ValidPass123"
    And user clicks login button
    Then user should be redirected to home page
    And welcome message "Welcome, John" should be displayed

  Scenario: Login fails with invalid credentials
    When user enters username "invalid@example.com"
    And user enters password "WrongPassword"
    And user clicks login button
    Then error message "Invalid credentials" should be displayed
    And user should remain on login page

  Scenario Outline: Login with different user types
    When user logs in with "<username>" and "<password>"
    Then user should see "<dashboard>" dashboard
    
    Examples:
      | username           | password  | dashboard |
      | admin@example.com  | Admin@123 | Admin     |
      | user@example.com   | User@123  | User      |
      | guest@example.com  | Guest@123 | Guest     |

  Scenario: Account locked after multiple failed attempts
    Given user has failed login 2 times already
    When user enters username "john@example.com"
    And user enters wrong password
    And user clicks login button
    Then user account should be locked
    And message "Account locked. Contact support" should be displayed
```

### Step Definitions

```java
import io.cucumber.java.en.*;
import org.testng.Assert;

public class LoginSteps {
    private LoginPage loginPage;
    private HomePage homePage;
    private String username;
    private String password;
    
    @Given("the user is on the login page")
    public void userIsOnLoginPage() {
        loginPage = new LoginPage();
        loginPage.navigateTo();
    }
    
    @Given("user has valid credentials")
    public void userHasValidCredentials() {
        username = TestDataFactory.getValidUser().getUsername();
        password = TestDataFactory.getValidUser().getPassword();
    }
    
    @When("user enters username {string}")
    public void userEntersUsername(String username) {
        loginPage.enterUsername(username);
    }
    
    @When("user enters password {string}")
    public void userEntersPassword(String password) {
        loginPage.enterPassword(password);
    }
    
    @When("user clicks login button")
    public void userClicksLoginButton() {
        homePage = loginPage.clickLogin();
    }
    
    @Then("user should be redirected to home page")
    public void userShouldBeRedirectedToHomePage() {
        Assert.assertTrue(homePage.isHomePageDisplayed(),
            "Home page should be displayed");
    }
    
    @Then("welcome message {string} should be displayed")
    public void welcomeMessageShouldBeDisplayed(String expectedMessage) {
        String actualMessage = homePage.getWelcomeMessage();
        Assert.assertTrue(actualMessage.contains(expectedMessage),
            "Welcome message should contain: " + expectedMessage);
    }
    
    @Then("error message {string} should be displayed")
    public void errorMessageShouldBeDisplayed(String expectedError) {
        String actualError = loginPage.getErrorMessage();
        Assert.assertEquals(actualError, expectedError,
            "Error message mismatch");
    }
    
    @Then("user should remain on login page")
    public void userShouldRemainOnLoginPage() {
        Assert.assertTrue(loginPage.isLoginPageDisplayed(),
            "Should remain on login page");
    }
}
```

---

## Testing in Agile

### Testing Quadrants

```
┌─────────────────────────────────────────────────────────┐
│              Agile Testing Quadrants                    │
└─────────────────────────────────────────────────────────┘

        Supporting the Team  |  Critique Product
    ─────────────────────────┼─────────────────────────
    Q1: Technology-Facing    │  Q2: Business-Facing
    Automated                │  Manual/Automated
    ─────────────────────────│─────────────────────────
    - Unit Tests             │  - Functional Tests
    - Component Tests        │  - Story Tests
    - API Tests              │  - Prototypes
                            │  - Simulations
    ─────────────────────────┼─────────────────────────
    Q3: Business-Facing      │  Q4: Technology-Facing
    Manual                   │  Tools/Automated
    ─────────────────────────│─────────────────────────
    - Exploratory Testing    │  - Performance Tests
    - Usability Testing      │  - Load Tests
    - UAT                    │  - Security Tests
    - Alpha/Beta Testing     │  - "-ility" Tests
```

### Test Automation Pyramid

```
        /\
       /  \      E2E/UI Tests (10%)
      /____\     - Selenium Tests
     /      \    - Slow, Brittle
    /        \
   /  Service \  Integration/API Tests (30%)
  /____________\ - RestAssured Tests
 /              \ - Medium Speed
/______________  \
    Unit Tests    Unit Tests (60%)
                  - JUnit/TestNG
                  - Fast, Reliable
```

### Testing Activities in Agile Sprint

```markdown
## Sprint Day-by-Day Testing Activities

### Day 1-2: Sprint Planning
- [ ] Review user stories
- [ ] Define acceptance criteria
- [ ] Identify test scenarios
- [ ] Estimate testing effort
- [ ] Plan automation tasks
- [ ] Setup test environment

### Day 3-4: Development Start
- [ ] Write unit tests (TDD)
- [ ] Create test data
- [ ] Setup test automation framework
- [ ] Write API test cases
- [ ] Review code changes

### Day 5-7: Mid-Sprint
- [ ] Execute smoke tests daily
- [ ] Automate regression tests
- [ ] Perform exploratory testing
- [ ] Report and track defects
- [ ] Update test cases
- [ ] API testing

### Day 8-9: Feature Complete
- [ ] Full regression testing
- [ ] Integration testing
- [ ] Cross-browser testing
- [ ] Performance testing
- [ ] Security testing
- [ ] Defect retesting

### Day 10: Sprint End
- [ ] Final regression
- [ ] Test sign-off
- [ ] Demo preparation
- [ ] Update test metrics
- [ ] Sprint retrospective
- [ ] Plan next sprint testing
```

---

## Sprint Planning

### Sprint Planning for QA

```java
public class SprintPlanning {
    
    public static class UserStory {
        private String id;
        private String title;
        private String description;
        private String acceptanceCriteria;
        private int storyPoints;
        private List<String> testScenarios;
        private int testingEffortHours;
        
        public UserStory(String id, String title, int storyPoints) {
            this.id = id;
            this.title = title;
            this.storyPoints = storyPoints;
            this.testScenarios = new ArrayList<>();
        }
        
        public void addTestScenario(String scenario) {
            testScenarios.add(scenario);
        }
        
        public void estimateTestingEffort() {
            // Rule of thumb: Testing effort = 30-40% of development effort
            int devHours = storyPoints * 2; // Assuming 1 SP = 2 hours
            testingEffortHours = (int) (devHours * 0.35);
        }
        
        public void printStoryDetails() {
            System.out.println("=== User Story ===");
            System.out.println("ID: " + id);
            System.out.println("Title: " + title);
            System.out.println("Story Points: " + storyPoints);
            System.out.println("Testing Effort: " + testingEffortHours + " hours");
            System.out.println("Test Scenarios:");
            for (String scenario : testScenarios) {
                System.out.println("  - " + scenario);
            }
        }
    }
    
    public static void main(String[] args) {
        // Example: Sprint Planning
        UserStory story = new UserStory(
            "US-101",
            "User should be able to filter products by price",
            5
        );
        
        // Add test scenarios
        story.addTestScenario("Filter with min and max price");
        story.addTestScenario("Filter with only min price");
        story.addTestScenario("Filter with only max price");
        story.addTestScenario("Filter with no products in range");
        story.addTestScenario("Verify URL parameters");
        story.addTestScenario("Test on mobile devices");
        story.addTestScenario("Cross-browser compatibility");
        
        // Estimate testing effort
        story.estimateTestingEffort();
        
        // Print details
        story.printStoryDetails();
    }
}
```

### Definition of Done (DoD)

```markdown
## Definition of Done Checklist

### Code Quality
- [ ] Code complete and committed to repository
- [ ] Code reviewed by at least 2 team members
- [ ] No code quality violations (SonarQube passing)
- [ ] Technical debt documented

### Testing
- [ ] Unit tests written (minimum 80% coverage)
- [ ] Unit tests passing
- [ ] Integration tests written and passing
- [ ] UI automation tests created
- [ ] Regression tests passing
- [ ] No critical or blocker defects open
- [ ] High priority defects fixed
- [ ] Manual exploratory testing completed
- [ ] Cross-browser testing done (Chrome, Firefox, Safari)
- [ ] Mobile responsive testing completed
- [ ] Performance testing done (if applicable)
- [ ] Security testing completed (if applicable)

### Documentation
- [ ] Code comments added where necessary
- [ ] README updated
- [ ] API documentation updated
- [ ] Test cases documented
- [ ] Release notes updated

### Deployment
- [ ] Feature deployed to dev environment
- [ ] Feature deployed to QA environment
- [ ] Smoke tests passing on QA
- [ ] Product owner demo completed
- [ ] Product owner acceptance received

### Non-Functional
- [ ] Accessibility standards met (WCAG 2.1)
- [ ] Performance benchmarks met
- [ ] Security best practices followed
- [ ] Logging and monitoring in place
```

### Test Estimation in Agile

```java
public class AgileTestEstimation {
    
    // Method 1: Story Points
    public int estimateTestingStoryPoints(int devStoryPoints) {
        // Testing typically 30-40% of development effort
        return (int) Math.ceil(devStoryPoints * 0.35);
    }
    
    // Method 2: T-Shirt Sizing
    public enum TestComplexity {
        XS(2),  // 2 hours
        S(4),   // 4 hours
        M(8),   // 8 hours
        L(16),  // 16 hours
        XL(24); // 24 hours
        
        private int hours;
        
        TestComplexity(int hours) {
            this.hours = hours;
        }
        
        public int getHours() {
            return hours;
        }
    }
    
    // Method 3: Three-Point Estimation
    public double estimateTestEffort(int optimistic, int mostLikely, int pessimistic) {
        return (optimistic + (4 * mostLikely) + pessimistic) / 6.0;
    }
    
    // Example usage
    public void estimateSprintTesting() {
        System.out.println("=== Sprint Testing Estimation ===\n");
        
        // Story 1: Login feature
        System.out.println("Story 1: Login Feature");
        System.out.println("Dev Story Points: 8");
        System.out.println("Test Story Points: " + estimateTestingStoryPoints(8));
        System.out.println("Test Complexity: " + TestComplexity.M);
        
        double effort = estimateTestEffort(6, 8, 12);
        System.out.println("Estimated Hours: " + String.format("%.1f", effort));
        System.out.println();
        
        // Story 2: Payment integration
        System.out.println("Story 2: Payment Integration");
        System.out.println("Dev Story Points: 13");
        System.out.println("Test Story Points: " + estimateTestingStoryPoints(13));
        System.out.println("Test Complexity: " + TestComplexity.L);
        
        effort = estimateTestEffort(12, 16, 24);
        System.out.println("Estimated Hours: " + String.format("%.1f", effort));
    }
    
    public static void main(String[] args) {
        AgileTestEstimation estimator = new AgileTestEstimation();
        estimator.estimateSprintTesting();
    }
}
```

### Velocity and Capacity Planning

```markdown
## Sprint Capacity Planning

### Team Capacity
- Sprint Duration: 10 working days (2 weeks)
- Team Size: 5 members
  - 2 Developers
  - 2 QA Engineers
  - 1 Automation Engineer

### Individual Capacity
| Member | Role | Available Hours | Productive Hours |
|--------|------|-----------------|------------------|
| Dev 1  | Developer | 80 | 60 (75%) |
| Dev 2  | Developer | 80 | 60 (75%) |
| QA 1   | QA Engineer | 80 | 65 (80%) |
| QA 2   | QA Engineer | 80 | 65 (80%) |
| Auto 1 | Automation | 80 | 70 (85%) |
| **Total** | | **400** | **320** |

### Planned Activities (QA)
- Test Planning: 8 hours
- Test Case Creation: 20 hours
- Test Execution: 80 hours
- Automation: 60 hours
- Defect Management: 25 hours
- Meetings/Ceremonies: 15 hours
- Regression Testing: 40 hours
- Exploratory Testing: 20 hours
- **Total:** 268 hours (within capacity of 270)

### Velocity (Story Points)
- Last 3 Sprints: 45, 48, 42
- Average Velocity: 45 points
- Team Capacity: 42-48 points
- Planned for Sprint: 45 points
```

---

**Next:** [Leadership & Soft Skills](11-leadership-soft-skills.md)

