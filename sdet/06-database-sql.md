# 6. Database & SQL

## Table of Contents
- [SQL Fundamentals](#sql-fundamentals)
- [Advanced SQL Queries](#advanced-sql-queries)
- [Database Testing](#database-testing)
- [JDBC in Automation](#jdbc-in-automation)
- [NoSQL Basics](#nosql-basics)
- [Data Validation Strategies](#data-validation-strategies)

---

## SQL Fundamentals

### Basic SQL Commands

```sql
-- CREATE DATABASE
CREATE DATABASE testdb;
USE testdb;

-- CREATE TABLE
CREATE TABLE users (
    id INT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(50) NOT NULL UNIQUE,
    email VARCHAR(100) NOT NULL UNIQUE,
    password VARCHAR(255) NOT NULL,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    age INT,
    status VARCHAR(20) DEFAULT 'active',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

CREATE TABLE orders (
    order_id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT,
    product_name VARCHAR(100),
    quantity INT,
    price DECIMAL(10, 2),
    order_date DATE,
    status VARCHAR(20),
    FOREIGN KEY (user_id) REFERENCES users(id)
);

-- INSERT DATA
INSERT INTO users (username, email, password, first_name, last_name, age)
VALUES ('john_doe', 'john@example.com', 'hashed_password', 'John', 'Doe', 30);

INSERT INTO users (username, email, password, first_name, last_name, age)
VALUES 
    ('alice_smith', 'alice@example.com', 'pass123', 'Alice', 'Smith', 25),
    ('bob_jones', 'bob@example.com', 'pass456', 'Bob', 'Jones', 35),
    ('charlie_brown', 'charlie@example.com', 'pass789', 'Charlie', 'Brown', 28);

-- SELECT DATA
SELECT * FROM users;
SELECT username, email FROM users;
SELECT * FROM users WHERE age > 25;
SELECT * FROM users WHERE status = 'active' AND age BETWEEN 25 AND 35;
SELECT * FROM users WHERE username LIKE 'john%';
SELECT * FROM users ORDER BY age DESC;
SELECT * FROM users LIMIT 10;

-- UPDATE DATA
UPDATE users 
SET status = 'inactive' 
WHERE username = 'john_doe';

UPDATE users 
SET age = age + 1 
WHERE age < 30;

-- DELETE DATA
DELETE FROM users WHERE status = 'inactive';

-- ALTER TABLE
ALTER TABLE users ADD COLUMN phone VARCHAR(15);
ALTER TABLE users MODIFY COLUMN age INT NOT NULL;
ALTER TABLE users DROP COLUMN phone;

-- DROP TABLE
DROP TABLE IF EXISTS temp_table;
```

---

## Advanced SQL Queries

### Joins

```sql
-- INNER JOIN
SELECT u.username, u.email, o.order_id, o.product_name, o.price
FROM users u
INNER JOIN orders o ON u.id = o.user_id;

-- LEFT JOIN (Get all users even if they have no orders)
SELECT u.username, u.email, o.order_id, o.product_name
FROM users u
LEFT JOIN orders o ON u.id = o.user_id;

-- RIGHT JOIN (Get all orders even if user doesn't exist)
SELECT u.username, o.order_id, o.product_name
FROM users u
RIGHT JOIN orders o ON u.id = o.user_id;

-- FULL OUTER JOIN (MySQL doesn't support, use UNION)
SELECT u.username, o.order_id
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
UNION
SELECT u.username, o.order_id
FROM users u
RIGHT JOIN orders o ON u.id = o.user_id;

-- SELF JOIN
SELECT e1.name AS Employee, e2.name AS Manager
FROM employees e1
LEFT JOIN employees e2 ON e1.manager_id = e2.id;

-- CROSS JOIN
SELECT u.username, p.product_name
FROM users u
CROSS JOIN products p;
```

### Aggregate Functions

```sql
-- COUNT
SELECT COUNT(*) AS total_users FROM users;
SELECT COUNT(DISTINCT status) AS unique_statuses FROM users;
SELECT status, COUNT(*) AS count FROM users GROUP BY status;

-- SUM
SELECT SUM(price) AS total_revenue FROM orders;
SELECT user_id, SUM(price) AS total_spent 
FROM orders 
GROUP BY user_id;

-- AVG
SELECT AVG(age) AS average_age FROM users;
SELECT status, AVG(age) AS avg_age 
FROM users 
GROUP BY status;

-- MIN and MAX
SELECT MIN(age) AS youngest, MAX(age) AS oldest FROM users;
SELECT user_id, MIN(order_date) AS first_order, MAX(order_date) AS last_order
FROM orders
GROUP BY user_id;

-- GROUP BY with HAVING
SELECT user_id, SUM(price) AS total_spent
FROM orders
GROUP BY user_id
HAVING SUM(price) > 100;

SELECT status, COUNT(*) AS count
FROM users
GROUP BY status
HAVING COUNT(*) > 5;
```

### Subqueries

```sql
-- Subquery in WHERE clause
SELECT username, email
FROM users
WHERE id IN (
    SELECT user_id 
    FROM orders 
    WHERE price > 50
);

-- Subquery in FROM clause
SELECT avg_age.status, avg_age.average_age
FROM (
    SELECT status, AVG(age) AS average_age
    FROM users
    GROUP BY status
) AS avg_age
WHERE avg_age.average_age > 30;

-- Correlated Subquery
SELECT u.username, u.email
FROM users u
WHERE EXISTS (
    SELECT 1 
    FROM orders o 
    WHERE o.user_id = u.id AND o.price > 100
);

-- Subquery with ANY/ALL
SELECT username
FROM users
WHERE age > ANY (SELECT age FROM users WHERE status = 'inactive');

SELECT username
FROM users
WHERE age > ALL (SELECT age FROM users WHERE status = 'inactive');
```

### Window Functions

```sql
-- ROW_NUMBER
SELECT 
    username,
    age,
    ROW_NUMBER() OVER (ORDER BY age DESC) AS row_num
FROM users;

-- RANK
SELECT 
    username,
    age,
    RANK() OVER (ORDER BY age DESC) AS rank
FROM users;

-- DENSE_RANK
SELECT 
    username,
    age,
    DENSE_RANK() OVER (ORDER BY age DESC) AS dense_rank
FROM users;

-- PARTITION BY
SELECT 
    username,
    status,
    age,
    RANK() OVER (PARTITION BY status ORDER BY age DESC) AS rank_in_status
FROM users;

-- RUNNING TOTAL
SELECT 
    order_id,
    user_id,
    price,
    SUM(price) OVER (ORDER BY order_id) AS running_total
FROM orders;
```

### Common Interview Questions

```sql
-- Q1: Find duplicate emails
SELECT email, COUNT(*) AS count
FROM users
GROUP BY email
HAVING COUNT(*) > 1;

-- Q2: Find second highest salary
SELECT MAX(salary) AS second_highest
FROM employees
WHERE salary < (SELECT MAX(salary) FROM employees);

-- OR using LIMIT
SELECT DISTINCT salary
FROM employees
ORDER BY salary DESC
LIMIT 1 OFFSET 1;

-- Q3: Find Nth highest salary
SELECT DISTINCT salary
FROM employees e1
WHERE N-1 = (
    SELECT COUNT(DISTINCT salary)
    FROM employees e2
    WHERE e2.salary > e1.salary
);

-- Q4: Delete duplicate rows
DELETE FROM users
WHERE id NOT IN (
    SELECT MIN(id)
    FROM users
    GROUP BY email
);

-- Q5: Find employees with no orders
SELECT e.name
FROM employees e
LEFT JOIN orders o ON e.id = o.employee_id
WHERE o.id IS NULL;

-- Q6: Get users who ordered in last 30 days
SELECT DISTINCT u.username, u.email
FROM users u
INNER JOIN orders o ON u.id = o.user_id
WHERE o.order_date >= DATE_SUB(CURDATE(), INTERVAL 30 DAY);

-- Q7: Rank users by total order amount
SELECT 
    u.username,
    SUM(o.price) AS total_amount,
    RANK() OVER (ORDER BY SUM(o.price) DESC) AS rank
FROM users u
INNER JOIN orders o ON u.id = o.user_id
GROUP BY u.username;

-- Q8: Find users with more than 5 orders
SELECT u.username, COUNT(o.order_id) AS order_count
FROM users u
INNER JOIN orders o ON u.id = o.user_id
GROUP BY u.username
HAVING COUNT(o.order_id) > 5;

-- Q9: Month-wise revenue
SELECT 
    DATE_FORMAT(order_date, '%Y-%m') AS month,
    SUM(price) AS revenue
FROM orders
GROUP BY DATE_FORMAT(order_date, '%Y-%m')
ORDER BY month;

-- Q10: Running average
SELECT 
    order_id,
    price,
    AVG(price) OVER (ORDER BY order_id ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) AS moving_avg
FROM orders;
```

---

## Database Testing

### Types of Database Testing

```
1. Structural Testing
   - Schema validation
   - Table and column existence
   - Data type validation
   - Constraint checking

2. Functional Testing
   - CRUD operations
   - Stored procedures
   - Triggers
   - Views

3. Non-Functional Testing
   - Performance testing
   - Load testing
   - Security testing

4. Data Integrity Testing
   - Data validation
   - Referential integrity
   - Data consistency
```

### Database Test Scenarios

```sql
-- Test Case 1: Verify table exists
SELECT COUNT(*) 
FROM information_schema.tables 
WHERE table_schema = 'testdb' 
AND table_name = 'users';
-- Expected: 1

-- Test Case 2: Verify column data type
SELECT DATA_TYPE 
FROM information_schema.columns 
WHERE table_name = 'users' 
AND column_name = 'age';
-- Expected: INT

-- Test Case 3: Verify NOT NULL constraint
SELECT IS_NULLABLE 
FROM information_schema.columns 
WHERE table_name = 'users' 
AND column_name = 'username';
-- Expected: NO

-- Test Case 4: Verify default value
SELECT COLUMN_DEFAULT 
FROM information_schema.columns 
WHERE table_name = 'users' 
AND column_name = 'status';
-- Expected: 'active'

-- Test Case 5: Verify foreign key relationship
SELECT 
    CONSTRAINT_NAME,
    REFERENCED_TABLE_NAME,
    REFERENCED_COLUMN_NAME
FROM information_schema.KEY_COLUMN_USAGE
WHERE TABLE_NAME = 'orders' 
AND COLUMN_NAME = 'user_id';

-- Test Case 6: Verify trigger existence
SELECT TRIGGER_NAME, EVENT_MANIPULATION, ACTION_TIMING
FROM information_schema.TRIGGERS
WHERE TRIGGER_SCHEMA = 'testdb';

-- Test Case 7: Verify stored procedure
SELECT ROUTINE_NAME, ROUTINE_TYPE
FROM information_schema.ROUTINES
WHERE ROUTINE_SCHEMA = 'testdb';

-- Test Case 8: Data validation
SELECT * FROM users WHERE email NOT LIKE '%@%';
-- Expected: 0 rows (all emails should have @)

SELECT * FROM users WHERE age < 0 OR age > 150;
-- Expected: 0 rows (invalid ages)

-- Test Case 9: Referential integrity
SELECT o.order_id, o.user_id
FROM orders o
LEFT JOIN users u ON o.user_id = u.id
WHERE u.id IS NULL;
-- Expected: 0 rows (all orders should have valid user)

-- Test Case 10: Data consistency
SELECT 
    u.id,
    COUNT(o.order_id) AS order_count_calc,
    u.order_count AS order_count_stored
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
GROUP BY u.id
HAVING order_count_calc != order_count_stored;
-- Expected: 0 rows (counts should match)
```

---

## JDBC in Automation

### Database Connection Utility

```java
package utils;

import java.sql.*;
import java.util.*;

public class DatabaseUtil {
    private static Connection connection;
    private static Statement statement;
    private static ResultSet resultSet;
    
    // Database Configuration
    private static final String DB_URL = "jdbc:mysql://localhost:3306/testdb";
    private static final String DB_USER = "root";
    private static final String DB_PASSWORD = "password";
    
    // Establish connection
    public static Connection getConnection() {
        try {
            if (connection == null || connection.isClosed()) {
                Class.forName("com.mysql.cj.jdbc.Driver");
                connection = DriverManager.getConnection(DB_URL, DB_USER, DB_PASSWORD);
                System.out.println("Database connection established");
            }
        } catch (ClassNotFoundException | SQLException e) {
            e.printStackTrace();
        }
        return connection;
    }
    
    // Execute Query (SELECT)
    public static ResultSet executeQuery(String query) {
        try {
            connection = getConnection();
            statement = connection.createStatement();
            resultSet = statement.executeQuery(query);
        } catch (SQLException e) {
            e.printStackTrace();
        }
        return resultSet;
    }
    
    // Execute Update (INSERT, UPDATE, DELETE)
    public static int executeUpdate(String query) {
        int rowsAffected = 0;
        try {
            connection = getConnection();
            statement = connection.createStatement();
            rowsAffected = statement.executeUpdate(query);
        } catch (SQLException e) {
            e.printStackTrace();
        }
        return rowsAffected;
    }
    
    // Get row count
    public static int getRowCount(String query) {
        int count = 0;
        try {
            resultSet = executeQuery(query);
            if (resultSet.last()) {
                count = resultSet.getRow();
                resultSet.beforeFirst();
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
        return count;
    }
    
    // Get single cell value
    public static String getCellValue(String query, int row, int column) {
        String cellValue = null;
        try {
            resultSet = executeQuery(query);
            int currentRow = 0;
            while (resultSet.next()) {
                currentRow++;
                if (currentRow == row) {
                    cellValue = resultSet.getString(column);
                    break;
                }
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
        return cellValue;
    }
    
    // Get all data as List of Maps
    public static List<Map<String, String>> getAllData(String query) {
        List<Map<String, String>> dataList = new ArrayList<>();
        try {
            resultSet = executeQuery(query);
            ResultSetMetaData metaData = resultSet.getMetaData();
            int columnCount = metaData.getColumnCount();
            
            while (resultSet.next()) {
                Map<String, String> row = new HashMap<>();
                for (int i = 1; i <= columnCount; i++) {
                    String columnName = metaData.getColumnName(i);
                    String value = resultSet.getString(i);
                    row.put(columnName, value);
                }
                dataList.add(row);
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
        return dataList;
    }
    
    // Get specific column data
    public static List<String> getColumnData(String query, String columnName) {
        List<String> columnData = new ArrayList<>();
        try {
            resultSet = executeQuery(query);
            while (resultSet.next()) {
                columnData.add(resultSet.getString(columnName));
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
        return columnData;
    }
    
    // Verify data exists
    public static boolean verifyDataExists(String query) {
        boolean exists = false;
        try {
            resultSet = executeQuery(query);
            exists = resultSet.next();
        } catch (SQLException e) {
            e.printStackTrace();
        }
        return exists;
    }
    
    // Close connections
    public static void closeConnection() {
        try {
            if (resultSet != null && !resultSet.isClosed()) {
                resultSet.close();
            }
            if (statement != null && !statement.isClosed()) {
                statement.close();
            }
            if (connection != null && !connection.isClosed()) {
                connection.close();
                System.out.println("Database connection closed");
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```

### Database Test Examples

```java
package tests;

import org.testng.Assert;
import org.testng.annotations.*;
import utils.DatabaseUtil;
import java.sql.ResultSet;
import java.util.*;

public class DatabaseTests {
    
    @BeforeClass
    public void setupDatabase() {
        DatabaseUtil.getConnection();
    }
    
    @Test(priority = 1)
    public void testInsertUser() {
        String insertQuery = "INSERT INTO users (username, email, password, first_name, last_name, age) " +
                           "VALUES ('test_user', 'test@example.com', 'pass123', 'Test', 'User', 25)";
        
        int rowsAffected = DatabaseUtil.executeUpdate(insertQuery);
        Assert.assertEquals(rowsAffected, 1, "User should be inserted");
    }
    
    @Test(priority = 2)
    public void testVerifyUserExists() {
        String query = "SELECT * FROM users WHERE username = 'test_user'";
        boolean exists = DatabaseUtil.verifyDataExists(query);
        Assert.assertTrue(exists, "User should exist in database");
    }
    
    @Test(priority = 3)
    public void testGetUserDetails() {
        String query = "SELECT * FROM users WHERE username = 'test_user'";
        List<Map<String, String>> data = DatabaseUtil.getAllData(query);
        
        Assert.assertTrue(data.size() > 0, "User data should be returned");
        
        Map<String, String> user = data.get(0);
        Assert.assertEquals(user.get("email"), "test@example.com");
        Assert.assertEquals(user.get("first_name"), "Test");
        Assert.assertEquals(user.get("last_name"), "User");
        Assert.assertEquals(user.get("age"), "25");
    }
    
    @Test(priority = 4)
    public void testGetRowCount() {
        String query = "SELECT * FROM users";
        int count = DatabaseUtil.getRowCount(query);
        Assert.assertTrue(count > 0, "Should have at least one user");
    }
    
    @Test(priority = 5)
    public void testGetColumnData() {
        String query = "SELECT username FROM users";
        List<String> usernames = DatabaseUtil.getColumnData(query, "username");
        
        Assert.assertTrue(usernames.size() > 0, "Should have usernames");
        Assert.assertTrue(usernames.contains("test_user"), "Should contain test_user");
    }
    
    @Test(priority = 6)
    public void testUpdateUser() {
        String updateQuery = "UPDATE users SET age = 26 WHERE username = 'test_user'";
        int rowsAffected = DatabaseUtil.executeUpdate(updateQuery);
        Assert.assertEquals(rowsAffected, 1, "User should be updated");
        
        // Verify update
        String selectQuery = "SELECT age FROM users WHERE username = 'test_user'";
        String age = DatabaseUtil.getCellValue(selectQuery, 1, 1);
        Assert.assertEquals(age, "26", "Age should be updated to 26");
    }
    
    @Test(priority = 7)
    public void testJoinQuery() {
        // First create an order
        String insertOrder = "INSERT INTO orders (user_id, product_name, quantity, price, order_date, status) " +
                           "VALUES ((SELECT id FROM users WHERE username = 'test_user'), " +
                           "'Test Product', 2, 50.00, CURDATE(), 'completed')";
        DatabaseUtil.executeUpdate(insertOrder);
        
        // Test join query
        String joinQuery = "SELECT u.username, o.product_name, o.price " +
                          "FROM users u INNER JOIN orders o ON u.id = o.user_id " +
                          "WHERE u.username = 'test_user'";
        
        List<Map<String, String>> data = DatabaseUtil.getAllData(joinQuery);
        Assert.assertTrue(data.size() > 0, "Should have order data");
        
        Map<String, String> order = data.get(0);
        Assert.assertEquals(order.get("product_name"), "Test Product");
    }
    
    @Test(priority = 8)
    public void testAggregateFunction() {
        String query = "SELECT COUNT(*) AS user_count FROM users";
        String count = DatabaseUtil.getCellValue(query, 1, 1);
        Assert.assertNotNull(count, "Should return count");
        Assert.assertTrue(Integer.parseInt(count) > 0, "Count should be greater than 0");
    }
    
    @Test(priority = 9)
    public void testDataIntegrity() {
        // Verify no orphaned orders (orders without valid user)
        String query = "SELECT o.order_id FROM orders o " +
                      "LEFT JOIN users u ON o.user_id = u.id " +
                      "WHERE u.id IS NULL";
        
        int orphanedOrders = DatabaseUtil.getRowCount(query);
        Assert.assertEquals(orphanedOrders, 0, "Should have no orphaned orders");
    }
    
    @Test(priority = 10)
    public void testDeleteUser() {
        // First delete orders
        String deleteOrders = "DELETE FROM orders WHERE user_id = " +
                            "(SELECT id FROM users WHERE username = 'test_user')";
        DatabaseUtil.executeUpdate(deleteOrders);
        
        // Then delete user
        String deleteUser = "DELETE FROM users WHERE username = 'test_user'";
        int rowsAffected = DatabaseUtil.executeUpdate(deleteUser);
        Assert.assertEquals(rowsAffected, 1, "User should be deleted");
        
        // Verify deletion
        String verifyQuery = "SELECT * FROM users WHERE username = 'test_user'";
        boolean exists = DatabaseUtil.verifyDataExists(verifyQuery);
        Assert.assertFalse(exists, "User should not exist after deletion");
    }
    
    @AfterClass
    public void tearDown() {
        DatabaseUtil.closeConnection();
    }
}
```

### End-to-End Test with DB Validation

```java
public class E2EWithDatabaseValidation {
    private WebDriver driver;
    
    @Test
    public void testUserRegistrationWithDBValidation() {
        // Step 1: UI - Register user
        driver.get("https://example.com/register");
        
        String username = "testuser_" + System.currentTimeMillis();
        String email = username + "@test.com";
        String password = "Test@123";
        
        driver.findElement(By.id("username")).sendKeys(username);
        driver.findElement(By.id("email")).sendKeys(email);
        driver.findElement(By.id("password")).sendKeys(password);
        driver.findElement(By.id("register")).click();
        
        // Step 2: UI Validation
        WebElement successMessage = driver.findElement(By.className("success"));
        Assert.assertTrue(successMessage.isDisplayed());
        
        // Step 3: Database Validation
        String query = "SELECT * FROM users WHERE username = '" + username + "'";
        List<Map<String, String>> userData = DatabaseUtil.getAllData(query);
        
        Assert.assertEquals(userData.size(), 1, "User should be created in database");
        
        Map<String, String> user = userData.get(0);
        Assert.assertEquals(user.get("username"), username);
        Assert.assertEquals(user.get("email"), email);
        Assert.assertEquals(user.get("status"), "active");
        Assert.assertNotNull(user.get("created_at"));
        
        // Step 4: Cleanup
        String deleteQuery = "DELETE FROM users WHERE username = '" + username + "'";
        DatabaseUtil.executeUpdate(deleteQuery);
    }
}
```

---

## NoSQL Basics

### MongoDB with Java

```java
package utils;

import com.mongodb.client.*;
import org.bson.Document;
import java.util.*;

public class MongoDBUtil {
    private static MongoClient mongoClient;
    private static MongoDatabase database;
    
    // Connect to MongoDB
    public static void connect(String connectionString, String dbName) {
        mongoClient = MongoClients.create(connectionString);
        database = mongoClient.getDatabase(dbName);
        System.out.println("Connected to MongoDB");
    }
    
    // Insert document
    public static void insertDocument(String collectionName, Document document) {
        MongoCollection<Document> collection = database.getCollection(collectionName);
        collection.insertOne(document);
    }
    
    // Find documents
    public static List<Document> findDocuments(String collectionName, Document filter) {
        MongoCollection<Document> collection = database.getCollection(collectionName);
        return collection.find(filter).into(new ArrayList<>());
    }
    
    // Update document
    public static void updateDocument(String collectionName, Document filter, Document update) {
        MongoCollection<Document> collection = database.getCollection(collectionName);
        collection.updateOne(filter, new Document("$set", update));
    }
    
    // Delete document
    public static void deleteDocument(String collectionName, Document filter) {
        MongoCollection<Document> collection = database.getCollection(collectionName);
        collection.deleteOne(filter);
    }
    
    // Close connection
    public static void close() {
        if (mongoClient != null) {
            mongoClient.close();
            System.out.println("MongoDB connection closed");
        }
    }
}

// Usage Example
@Test
public void testMongoDBOperations() {
    MongoDBUtil.connect("mongodb://localhost:27017", "testdb");
    
    // Insert
    Document user = new Document("username", "testuser")
        .append("email", "test@example.com")
        .append("age", 25);
    MongoDBUtil.insertDocument("users", user);
    
    // Find
    Document filter = new Document("username", "testuser");
    List<Document> users = MongoDBUtil.findDocuments("users", filter);
    Assert.assertTrue(users.size() > 0);
    
    // Update
    Document update = new Document("age", 26);
    MongoDBUtil.updateDocument("users", filter, update);
    
    // Delete
    MongoDBUtil.deleteDocument("users", filter);
    
    MongoDBUtil.close();
}
```

---

## Data Validation Strategies

### Test Data Management

```java
public class TestDataManager {
    
    // Create test data before test
    @BeforeMethod
    public void createTestData() {
        // Create user in database
        String query = "INSERT INTO users (username, email, password) " +
                      "VALUES ('test_user', 'test@example.com', 'pass123')";
        DatabaseUtil.executeUpdate(query);
    }
    
    // Cleanup test data after test
    @AfterMethod
    public void cleanupTestData() {
        String query = "DELETE FROM users WHERE username = 'test_user'";
        DatabaseUtil.executeUpdate(query);
    }
    
    // Validate data consistency between UI and DB
    public void validateDataConsistency(String usernameFromUI) {
        String query = "SELECT * FROM users WHERE username = '" + usernameFromUI + "'";
        List<Map<String, String>> dbData = DatabaseUtil.getAllData(query);
        
        Assert.assertEquals(dbData.size(), 1, "User should exist in DB");
        
        // Compare UI and DB data
        Map<String, String> userFromDB = dbData.get(0);
        Assert.assertEquals(userFromDB.get("username"), usernameFromUI);
    }
}
```

---

**Next:** [Test Strategy & Planning](07-test-strategy-planning.md)

