# 7. Test Strategy & Planning

## Table of Contents
- [Test Planning & Estimation](#test-planning--estimation)
- [Risk-Based Testing](#risk-based-testing)
- [Test Prioritization](#test-prioritization)
- [Regression Strategy](#regression-strategy)
- [Shift-Left Testing](#shift-left-testing)
- [Test Coverage Metrics](#test-coverage-metrics)

---

## Test Planning & Estimation

### Test Plan Document Structure

```
1. TEST PLAN TEMPLATE
==================

1. Introduction
   - Purpose
   - Scope
   - Audience

2. Test Strategy
   - Test Levels (Unit, Integration, System, UAT)
   - Test Types (Functional, Non-functional)
   - Entry and Exit Criteria

3. Test Scope
   - In Scope
   - Out of Scope
   - Assumptions
   - Constraints

4. Test Approach
   - Manual vs Automated
   - Tools and Technologies
   - Test Environment
   - Test Data Strategy

5. Resource Planning
   - Team Structure
   - Roles and Responsibilities
   - Training Needs

6. Schedule
   - Milestones
   - Timeline
   - Dependencies

7. Risks and Mitigation
   - Technical Risks
   - Resource Risks
   - Schedule Risks

8. Deliverables
   - Test Cases
   - Test Reports
   - Defect Reports

9. Approval
   - Sign-off
```

### Test Estimation Techniques

```java
public class TestEstimation {
    
    // 1. Work Breakdown Structure (WBS)
    public int estimateUsingWBS() {
        int testDesign = 40;        // hours
        int testExecution = 60;     // hours
        int defectRetesting = 20;   // hours
        int reporting = 10;         // hours
        int buffer = 20;            // 20% buffer
        
        int totalHours = testDesign + testExecution + defectRetesting + reporting;
        totalHours += (totalHours * buffer) / 100;
        
        return totalHours;  // Total: 180 hours
    }
    
    // 2. Three-Point Estimation
    public double threePointEstimation() {
        double optimistic = 100;   // Best case
        double mostLikely = 150;   // Most likely case
        double pessimistic = 250;  // Worst case
        
        // PERT Formula: (O + 4M + P) / 6
        double estimate = (optimistic + (4 * mostLikely) + pessimistic) / 6;
        
        return estimate;  // Result: 158.33 hours
    }
    
    // 3. Function Point Analysis
    public int estimateByFunctionPoints() {
        int numberOfScreens = 10;
        int hoursPerScreen = 8;     // Analysis + Test Case Writing
        int executionPerScreen = 4;
        
        int totalHours = numberOfScreens * (hoursPerScreen + executionPerScreen);
        return totalHours;  // 120 hours
    }
    
    // 4. Test Case Based Estimation
    public int estimateByTestCases() {
        int totalTestCases = 200;
        double avgTimePerTestCase = 0.5;  // hours
        int regressionCycles = 3;
        
        int executionTime = (int)(totalTestCases * avgTimePerTestCase);
        int totalTime = executionTime * regressionCycles;
        
        return totalTime;  // 300 hours
    }
    
    // 5. Story Points (Agile)
    public int estimateByStoryPoints() {
        Map<String, Integer> stories = new HashMap<>();
        stories.put("User Login", 3);
        stories.put("Product Search", 5);
        stories.put("Add to Cart", 3);
        stories.put("Checkout", 8);
        stories.put("Payment", 13);
        
        int totalPoints = stories.values().stream()
            .mapToInt(Integer::intValue)
            .sum();  // 32 points
        
        int velocity = 10;  // points per sprint
        int sprints = (int) Math.ceil((double) totalPoints / velocity);
        
        return sprints;  // 4 sprints
    }
}
```

### Test Plan Example

```markdown
# Test Plan - E-commerce Application

## 1. Introduction

**Purpose:** This document defines the test strategy and approach for the 
E-commerce Application v2.0 release.

**Scope:** Testing will cover all modules including User Management, 
Product Catalog, Shopping Cart, and Payment Processing.

## 2. Test Objectives

- Ensure 95% functional test coverage
- Achieve 80% automation coverage
- Identify critical defects before production
- Validate performance benchmarks

## 3. Test Scope

### In Scope:
- Functional testing of all user workflows
- API testing for backend services
- Cross-browser testing (Chrome, Firefox, Edge)
- Mobile responsive testing
- Security testing (OWASP Top 10)
- Performance testing (load, stress)

### Out of Scope:
- Third-party payment gateway internal testing
- Legacy system integration
- Infrastructure testing

## 4. Test Strategy

### Test Levels:
1. **Unit Testing** - Developers (80% coverage target)
2. **API Testing** - QA Team (Automated with RestAssured)
3. **UI Testing** - QA Team (Selenium automation)
4. **Integration Testing** - QA Team
5. **UAT** - Business stakeholders

### Test Types:
- Functional Testing
- Regression Testing
- Smoke Testing
- Integration Testing
- Performance Testing
- Security Testing
- Usability Testing

## 5. Entry and Exit Criteria

### Entry Criteria:
- Code deployed to test environment
- Test environment stable and accessible
- Test data prepared
- Test cases reviewed and approved

### Exit Criteria:
- 100% test execution completion
- No critical/blocker defects open
- 95% pass rate achieved
- All smoke tests passing
- Performance benchmarks met

## 6. Test Environment

**Environment URL:** https://qa.ecommerce.com
**Database:** MySQL 8.0
**Browsers:** Chrome 120+, Firefox 115+, Edge 120+
**Mobile:** iOS 16+, Android 12+

## 7. Test Deliverables

- Test Plan (This document)
- Test Cases (200+ test cases)
- Test Execution Report
- Defect Report
- Test Summary Report
- Automation Scripts

## 8. Resource Allocation

| Role | Name | Allocation |
|------|------|------------|
| Test Lead | John Doe | 100% |
| Sr. Automation Engineer | Alice Smith | 100% |
| QA Engineer | Bob Jones | 100% |
| QA Engineer | Charlie Brown | 100% |

## 9. Schedule

| Phase | Start Date | End Date | Duration |
|-------|-----------|----------|----------|
| Test Planning | Jan 1 | Jan 5 | 5 days |
| Test Design | Jan 6 | Jan 15 | 10 days |
| Test Execution | Jan 16 | Jan 30 | 15 days |
| Defect Fix & Retest | Jan 31 | Feb 5 | 6 days |
| UAT | Feb 6 | Feb 10 | 5 days |
| Go-Live | Feb 12 | Feb 12 | 1 day |

## 10. Risks and Mitigation

| Risk | Impact | Probability | Mitigation |
|------|--------|-------------|------------|
| Test environment unstable | High | Medium | Daily environment monitoring |
| Resource unavailability | Medium | Low | Cross-training team members |
| Late delivery of features | High | Medium | Daily standup meetings |
| Third-party API issues | High | Low | Mock services for testing |

## 11. Assumptions

- Test environment will be available 95% of the time
- Developers will fix defects within agreed SLA
- Test data will be provided by business team
- All required tools and licenses are available

## 12. Approval

**Prepared by:** Test Lead - John Doe
**Reviewed by:** QA Manager - Mary Johnson
**Approved by:** Project Manager - David Williams

**Date:** January 1, 2024
```

---

## Risk-Based Testing

### Risk Assessment Matrix

```java
public class RiskAssessment {
    
    public enum Probability {
        LOW(1), MEDIUM(2), HIGH(3);
        private int value;
        Probability(int value) { this.value = value; }
        public int getValue() { return value; }
    }
    
    public enum Impact {
        LOW(1), MEDIUM(2), HIGH(3);
        private int value;
        Impact(int value) { this.value = value; }
        public int getValue() { return value; }
    }
    
    public static class RiskItem {
        private String feature;
        private Probability probability;
        private Impact impact;
        private int riskScore;
        private String priority;
        
        public RiskItem(String feature, Probability probability, Impact impact) {
            this.feature = feature;
            this.probability = probability;
            this.impact = impact;
            this.riskScore = probability.getValue() * impact.getValue();
            this.priority = calculatePriority();
        }
        
        private String calculatePriority() {
            if (riskScore >= 6) return "CRITICAL";
            else if (riskScore >= 4) return "HIGH";
            else if (riskScore >= 2) return "MEDIUM";
            else return "LOW";
        }
        
        // Getters
        public String getFeature() { return feature; }
        public int getRiskScore() { return riskScore; }
        public String getPriority() { return priority; }
    }
    
    public static void main(String[] args) {
        List<RiskItem> risks = new ArrayList<>();
        
        // Define risks
        risks.add(new RiskItem("Payment Processing", Probability.HIGH, Impact.HIGH));
        risks.add(new RiskItem("User Login", Probability.MEDIUM, Impact.HIGH));
        risks.add(new RiskItem("Product Search", Probability.LOW, Impact.MEDIUM));
        risks.add(new RiskItem("Email Notifications", Probability.MEDIUM, Impact.LOW));
        risks.add(new RiskItem("Admin Dashboard", Probability.LOW, Impact.LOW));
        
        // Sort by risk score
        risks.sort((r1, r2) -> Integer.compare(r2.getRiskScore(), r1.getRiskScore()));
        
        // Print risk matrix
        System.out.println("=== Risk Assessment Matrix ===");
        System.out.printf("%-25s %-15s %s%n", "Feature", "Risk Score", "Priority");
        System.out.println("-----------------------------------------------------");
        
        for (RiskItem risk : risks) {
            System.out.printf("%-25s %-15d %s%n", 
                risk.getFeature(), 
                risk.getRiskScore(), 
                risk.getPriority());
        }
    }
}
```

### Risk-Based Test Strategy

```markdown
## Risk-Based Testing Strategy

### Critical Risk Features (Priority 1)
**More Testing Effort - 40% of total testing time**

1. **Payment Processing (Risk Score: 9)**
   - Test Coverage: 100%
   - Automation: Yes
   - Test Types: Functional, Security, Integration, Performance
   - Test Cycles: 3+
   - Exploratory Testing: Yes

2. **User Authentication (Risk Score: 6)**
   - Test Coverage: 100%
   - Automation: Yes
   - Test Types: Functional, Security
   - Test Cycles: 3
   - Penetration Testing: Yes

### High Risk Features (Priority 2)
**Moderate Testing Effort - 35% of total testing time**

3. **Order Management (Risk Score: 4)**
   - Test Coverage: 90%
   - Automation: Yes
   - Test Types: Functional, Integration
   - Test Cycles: 2

4. **Shopping Cart (Risk Score: 4)**
   - Test Coverage: 90%
   - Automation: Yes
   - Test Types: Functional
   - Test Cycles: 2

### Medium/Low Risk Features (Priority 3)
**Basic Testing Effort - 25% of total testing time**

5. **Product Search (Risk Score: 2)**
   - Test Coverage: 70%
   - Automation: Partial
   - Test Types: Functional
   - Test Cycles: 1

6. **Email Notifications (Risk Score: 2)**
   - Test Coverage: 60%
   - Automation: No
   - Test Types: Functional
   - Test Cycles: 1
```

---

## Test Prioritization

### Test Case Prioritization Framework

```java
public class TestPrioritization {
    
    public static class TestCase {
        private String id;
        private String name;
        private int businessCriticality;  // 1-5
        private int defectProbability;    // 1-5
        private int executionTime;        // minutes
        private boolean isAutomated;
        private int executionFrequency;   // per sprint
        private double priorityScore;
        
        public TestCase(String id, String name, int businessCriticality, 
                       int defectProbability, int executionTime, 
                       boolean isAutomated, int executionFrequency) {
            this.id = id;
            this.name = name;
            this.businessCriticality = businessCriticality;
            this.defectProbability = defectProbability;
            this.executionTime = executionTime;
            this.isAutomated = isAutomated;
            this.executionFrequency = executionFrequency;
            this.priorityScore = calculatePriority();
        }
        
        private double calculatePriority() {
            // Priority = (BusinessCriticality * 0.4) + (DefectProbability * 0.3) + 
            //            (ExecutionFrequency * 0.2) + (AutomationBonus * 0.1)
            
            double automationBonus = isAutomated ? 5 : 1;
            
            return (businessCriticality * 0.4) + 
                   (defectProbability * 0.3) + 
                   (executionFrequency * 0.2) + 
                   (automationBonus * 0.1);
        }
        
        public double getPriorityScore() {
            return priorityScore;
        }
        
        public String getSummary() {
            return String.format("%-10s %-40s Priority: %.2f Time: %dm Automated: %s",
                id, name, priorityScore, executionTime, isAutomated ? "Yes" : "No");
        }
    }
    
    public static void main(String[] args) {
        List<TestCase> testCases = new ArrayList<>();
        
        // Add test cases
        testCases.add(new TestCase("TC001", "User Login - Valid Credentials", 
            5, 4, 2, true, 5));
        testCases.add(new TestCase("TC002", "Payment Processing - Credit Card", 
            5, 5, 5, true, 3));
        testCases.add(new TestCase("TC003", "Product Search - Keyword", 
            3, 2, 1, true, 4));
        testCases.add(new TestCase("TC004", "Update User Profile", 
            2, 1, 3, false, 1));
        testCases.add(new TestCase("TC005", "Checkout - Complete Order", 
            5, 4, 10, true, 3));
        
        // Sort by priority
        testCases.sort((t1, t2) -> 
            Double.compare(t2.getPriorityScore(), t1.getPriorityScore()));
        
        // Print prioritized list
        System.out.println("=== Prioritized Test Cases ===\n");
        for (int i = 0; i < testCases.size(); i++) {
            System.out.println((i + 1) + ". " + testCases.get(i).getSummary());
        }
    }
}
```

### Automation Priority Decision Matrix

```java
public class AutomationPriority {
    
    public static boolean shouldAutomate(String feature) {
        // Criteria for automation
        boolean isHighPriority = checkBusinessPriority(feature);
        boolean isRepetitive = checkExecutionFrequency(feature);
        boolean isStable = checkStability(feature);
        boolean hasGoodROI = calculateROI(feature) > 0;
        
        return isHighPriority && isRepetitive && isStable && hasGoodROI;
    }
    
    public static double calculateROI(String feature) {
        int manualExecutionTime = 30;  // minutes
        int executionsPerSprint = 5;
        int sprintsPerYear = 24;
        int automationDevelopmentTime = 240;  // minutes
        int maintenanceTime = 30;  // minutes per sprint
        
        int manualTimePerYear = manualExecutionTime * executionsPerSprint * sprintsPerYear;
        int automationCostFirstYear = automationDevelopmentTime + 
                                      (maintenanceTime * sprintsPerYear);
        
        double roi = ((manualTimePerYear - automationCostFirstYear) / 
                     (double) automationCostFirstYear) * 100;
        
        return roi;
    }
    
    private static boolean checkBusinessPriority(String feature) {
        // Check if feature is business critical
        return true;
    }
    
    private static boolean checkExecutionFrequency(String feature) {
        // Check if test is executed frequently
        return true;
    }
    
    private static boolean checkStability(String feature) {
        // Check if feature is stable (not changing frequently)
        return true;
    }
}
```

---

## Regression Strategy

### Regression Test Suite Management

```java
public class RegressionSuite {
    
    // 1. Full Regression Suite
    public void fullRegressionSuite() {
        System.out.println("Running Full Regression Suite");
        System.out.println("- All automated tests");
        System.out.println("- All critical manual tests");
        System.out.println("- Execution Time: 4-6 hours");
        System.out.println("- Frequency: Before major releases");
    }
    
    // 2. Smoke/Sanity Suite
    public void smokeSuite() {
        System.out.println("Running Smoke Test Suite");
        System.out.println("- Critical path tests");
        System.out.println("- Basic functionality validation");
        System.out.println("- Execution Time: 30 minutes");
        System.out.println("- Frequency: Every build");
    }
    
    // 3. Feature-Specific Regression
    public void featureRegression(String feature) {
        System.out.println("Running Feature Regression for: " + feature);
        System.out.println("- Tests related to " + feature);
        System.out.println("- Dependent feature tests");
        System.out.println("- Execution Time: 1-2 hours");
        System.out.println("- Frequency: After feature changes");
    }
    
    // 4. Risk-Based Regression
    public void riskBasedRegression() {
        System.out.println("Running Risk-Based Regression");
        System.out.println("- High-risk area tests");
        System.out.println("- Recently changed modules");
        System.out.println("- Defect-prone areas");
        System.out.println("- Execution Time: 2-3 hours");
        System.out.println("- Frequency: After significant changes");
    }
}
```

### Regression Test Selection

```markdown
## Regression Test Selection Strategy

### Level 1: Smoke Tests (Run on every build)
- Duration: 15-30 minutes
- Test Count: 20-30 tests
- Automation: 100%
- Scope: Critical paths only

**Example Tests:**
- User can login
- User can search products
- User can add to cart
- User can checkout
- Admin can access dashboard

### Level 2: Core Regression (Run daily)
- Duration: 1-2 hours
- Test Count: 100-150 tests
- Automation: 90%
- Scope: Core functionality

**Example Areas:**
- Authentication & Authorization
- Product Management
- Order Processing
- Payment Integration
- User Profile Management

### Level 3: Full Regression (Run weekly/before release)
- Duration: 4-8 hours
- Test Count: 500+ tests
- Automation: 70-80%
- Scope: All functionality

**Example Areas:**
- All Level 1 & 2 tests
- Edge cases
- Negative scenarios
- Integration tests
- Cross-browser tests
- Performance tests

### Level 4: Extended Regression (Run before major release)
- Duration: 16-24 hours
- Test Count: 1000+ tests
- Automation: 60-70%
- Scope: Comprehensive testing

**Example Areas:**
- All Level 1, 2, 3 tests
- Exploratory testing
- Compatibility testing
- Security testing
- Accessibility testing
- Usability testing
```

---

## Shift-Left Testing

### Shift-Left Approach

```markdown
## Shift-Left Testing Strategy

### Traditional Approach (Problems):
```
Requirements → Design → Development → Testing → Deployment
                                      ↑
                                   Testing starts here
                                   (Late defect detection)
```

### Shift-Left Approach (Solution):
```
Requirements → Design → Development → Testing → Deployment
↓           ↓          ↓             ↓
Testing     Testing    Testing       Testing
(Early defect detection at each phase)
```

### Implementation:

#### 1. Requirements Phase
- Review requirements for testability
- Create acceptance criteria
- Identify test scenarios early
- Risk assessment

#### 2. Design Phase
- Review design documents
- Create test strategy
- Identify integration points
- Design test data

#### 3. Development Phase
- Unit testing by developers
- Code reviews
- Static code analysis
- TDD/BDD approach

#### 4. Testing Phase
- Integration testing
- System testing
- UAT preparation
- Automation execution
```

### Shift-Left Implementation Example

```java
public class ShiftLeftTesting {
    
    // 1. Unit Testing (by Developers)
    @Test
    public void testCalculateDiscount() {
        Product product = new Product("Laptop", 1000.00);
        double discount = product.calculateDiscount(10);
        assertEquals(100.00, discount, 0.01);
    }
    
    // 2. Component Testing
    @Test
    public void testShoppingCartComponent() {
        ShoppingCart cart = new ShoppingCart();
        Product product = new Product("Laptop", 1000.00);
        
        cart.addProduct(product);
        assertEquals(1, cart.getItemCount());
        assertEquals(1000.00, cart.getTotalPrice(), 0.01);
    }
    
    // 3. API Testing (Early Integration Testing)
    @Test
    public void testCreateOrderAPI() {
        given()
            .contentType("application/json")
            .body(orderJson)
        .when()
            .post("/api/orders")
        .then()
            .statusCode(201)
            .body("order_id", notNullValue());
    }
    
    // 4. Contract Testing (Define API contracts early)
    @Pact(consumer = "OrderService")
    public RequestResponsePact createOrderPact(PactDslWithProvider builder) {
        return builder
            .given("user exists")
            .uponReceiving("create order request")
            .path("/api/orders")
            .method("POST")
            .willRespondWith()
            .status(201)
            .body(newJsonBody((root) -> {
                root.stringValue("order_id", "12345");
                root.stringValue("status", "created");
            }).build())
            .toPact();
    }
}
```

---

## Test Coverage Metrics

### Coverage Metrics Framework

```java
public class TestCoverageMetrics {
    
    public static class CoverageReport {
        private int totalRequirements;
        private int coveredRequirements;
        private int totalTestCases;
        private int automatedTestCases;
        private int passedTests;
        private int failedTests;
        private int blockedTests;
        
        public double getRequirementsCoverage() {
            return (coveredRequirements * 100.0) / totalRequirements;
        }
        
        public double getAutomationCoverage() {
            return (automatedTestCases * 100.0) / totalTestCases;
        }
        
        public double getPassPercentage() {
            int executedTests = passedTests + failedTests;
            return (passedTests * 100.0) / executedTests;
        }
        
        public double getDefectDensity(int totalDefects, int linesOfCode) {
            return (totalDefects * 1000.0) / linesOfCode;
        }
        
        public void printReport() {
            System.out.println("=== Test Coverage Report ===");
            System.out.printf("Requirements Coverage: %.2f%%%n", 
                getRequirementsCoverage());
            System.out.printf("Automation Coverage: %.2f%%%n", 
                getAutomationCoverage());
            System.out.printf("Pass Percentage: %.2f%%%n", 
                getPassPercentage());
            System.out.printf("Total Test Cases: %d%n", totalTestCases);
            System.out.printf("Automated: %d%n", automatedTestCases);
            System.out.printf("Passed: %d%n", passedTests);
            System.out.printf("Failed: %d%n", failedTests);
            System.out.printf("Blocked: %d%n", blockedTests);
        }
    }
    
    public static void main(String[] args) {
        CoverageReport report = new CoverageReport();
        report.totalRequirements = 150;
        report.coveredRequirements = 142;
        report.totalTestCases = 500;
        report.automatedTestCases = 400;
        report.passedTests = 480;
        report.failedTests = 15;
        report.blockedTests = 5;
        
        report.printReport();
    }
}
```

### Key Testing Metrics

```markdown
## Essential Testing Metrics

### 1. Test Coverage Metrics

**Requirements Coverage**
Formula: (Covered Requirements / Total Requirements) × 100
Target: 95%+

**Code Coverage**
- Line Coverage: 80%+
- Branch Coverage: 75%+
- Function Coverage: 85%+

**Automation Coverage**
Formula: (Automated Tests / Total Tests) × 100
Target: 70-80%

### 2. Test Execution Metrics

**Test Execution Rate**
Formula: Tests Executed / Total Planned Tests
Target: 100%

**Pass Percentage**
Formula: (Passed Tests / Executed Tests) × 100
Target: 95%+

**Test Effectiveness**
Formula: (Defects Found in Testing / Total Defects) × 100
Target: 80%+

### 3. Defect Metrics

**Defect Density**
Formula: (Total Defects / Lines of Code) × 1000
Target: < 1 defect per 1000 LOC

**Defect Removal Efficiency (DRE)**
Formula: (Defects Found Before Release / Total Defects) × 100
Target: 95%+

**Defect Leakage**
Formula: (Defects in Production / Total Defects) × 100
Target: < 5%

### 4. Time & Effort Metrics

**Test Execution Time**
- Smoke Suite: < 30 minutes
- Regression Suite: < 4 hours
- Full Suite: < 8 hours

**Automation ROI**
Formula: (Time Saved - Automation Cost) / Automation Cost × 100
Target: Positive ROI

### 5. Quality Metrics

**Test Case Effectiveness**
Formula: Defects Found / Test Cases Executed
Higher is better (finding more defects)

**Escaped Defects Rate**
Formula: Production Defects / Total Defects × 100
Target: < 5%

**Mean Time to Detect (MTTD)**
Average time to detect a defect
Target: Minimize

**Mean Time to Repair (MTTR)**
Average time to fix a defect
Target: < 48 hours for critical, < 5 days for others
```

---

**Next:** [CI/CD & DevOps](08-cicd-devops.md)

