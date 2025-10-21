# 5. API Testing & Web Services

## 📚 Quick Summary

API testing is **70% of modern testing** - faster, more reliable, and more important than UI testing!

**What You'll Learn:**
- **REST API Basics**: GET, POST, PUT, DELETE (CRUD operations)
- **RestAssured**: Most popular Java library for API testing
- **Authentication**: OAuth, JWT, Basic Auth
- **JSON/XML**: Parse and validate responses
- **Automation**: Complete API test framework

**Why This Matters:**
- APIs are the backbone of modern applications
- Faster feedback than UI tests
- More stable (no UI flakiness)
- Test business logic directly
- Microservices = lots of APIs

**Interview Gold:**
"I focus 70% on API testing, 30% on UI" - shows you understand testing pyramid!

---

## 📖 Simple Explanation

**What is an API?**
Think of an API like a waiter in a restaurant:
1. You (client) tell waiter what you want (request)
2. Waiter takes order to kitchen (server)
3. Kitchen prepares food (processes request)
4. Waiter brings food back (response)

**REST API = Menu with standard items:**
- GET = "Show me the menu" (read)
- POST = "Place new order" (create)
- PUT = "Change entire order" (update)
- DELETE = "Cancel order" (delete)

**Why Test APIs?**
- Ensure waiter brings right food
- Correct price charged
- Food delivered on time
- Kitchen handles wrong orders gracefully

---

## Table of Contents
- [REST API Fundamentals](#rest-api-fundamentals)
- [RestAssured Framework](#restassured-framework)
- [API Authentication](#api-authentication)
- [JSON/XML Parsing](#jsonxml-parsing)
- [API Test Automation](#api-test-automation)
- [Postman & Newman](#postman--newman)

---

## REST API Fundamentals

### HTTP Methods

| Method | Description | Idempotent | Safe |
|--------|-------------|------------|------|
| GET | Retrieve resource | Yes | Yes |
| POST | Create resource | No | No |
| PUT | Update/Replace resource | Yes | No |
| PATCH | Partial update | No | No |
| DELETE | Delete resource | Yes | No |
| HEAD | Get headers only | Yes | Yes |
| OPTIONS | Get allowed methods | Yes | Yes |

### HTTP Status Codes

```
1xx - Informational
100 Continue
101 Switching Protocols

2xx - Success
200 OK
201 Created
202 Accepted
204 No Content

3xx - Redirection
301 Moved Permanently
302 Found
304 Not Modified

4xx - Client Errors
400 Bad Request
401 Unauthorized
403 Forbidden
404 Not Found
405 Method Not Allowed
409 Conflict
422 Unprocessable Entity

5xx - Server Errors
500 Internal Server Error
502 Bad Gateway
503 Service Unavailable
504 Gateway Timeout
```

### REST API Design Principles

```
1. Resource-based URLs
   GET    /api/users           (Get all users)
   GET    /api/users/123       (Get user by ID)
   POST   /api/users           (Create user)
   PUT    /api/users/123       (Update user)
   DELETE /api/users/123       (Delete user)

2. Use HTTP methods correctly
3. Stateless communication
4. Use proper status codes
5. Version your API (/api/v1/, /api/v2/)
6. Use JSON for request/response
7. Implement proper error handling
8. Use filtering, sorting, pagination
   GET /api/users?page=1&limit=10&sort=name
```

---

## RestAssured Framework

### Setup and Basic Tests

```java
// pom.xml dependencies
<dependencies>
    <dependency>
        <groupId>io.rest-assured</groupId>
        <artifactId>rest-assured</artifactId>
        <version>5.3.2</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>io.rest-assured</groupId>
        <artifactId>json-schema-validator</artifactId>
        <version>5.3.2</version>
    </dependency>
    <dependency>
        <groupId>com.google.code.gson</groupId>
        <artifactId>gson</artifactId>
        <version>2.10.1</version>
    </dependency>
</dependencies>
```

```java
import io.restassured.RestAssured;
import io.restassured.response.Response;
import io.restassured.specification.RequestSpecification;
import static io.restassured.RestAssured.*;
import static org.hamcrest.Matchers.*;
import org.testng.annotations.*;

public class APIBasicTests {
    
    @BeforeClass
    public void setup() {
        RestAssured.baseURI = "https://jsonplaceholder.typicode.com";
    }
    
    // 1. Simple GET Request
    @Test
    public void testGetAllUsers() {
        given()
            .when()
            .get("/users")
            .then()
            .statusCode(200)
            .body("size()", greaterThan(0))
            .log().all();
    }
    
    // 2. GET with Path Parameter
    @Test
    public void testGetUserById() {
        given()
            .pathParam("userId", 1)
            .when()
            .get("/users/{userId}")
            .then()
            .statusCode(200)
            .body("id", equalTo(1))
            .body("name", notNullValue())
            .body("email", containsString("@"));
    }
    
    // 3. GET with Query Parameters
    @Test
    public void testGetPostsByUser() {
        given()
            .queryParam("userId", 1)
            .when()
            .get("/posts")
            .then()
            .statusCode(200)
            .body("userId", everyItem(equalTo(1)))
            .body("size()", greaterThan(0));
    }
    
    // 4. POST Request
    @Test
    public void testCreateUser() {
        String requestBody = """
            {
                "name": "John Doe",
                "username": "johndoe",
                "email": "john@example.com"
            }
            """;
        
        given()
            .header("Content-Type", "application/json")
            .body(requestBody)
            .when()
            .post("/users")
            .then()
            .statusCode(201)
            .body("name", equalTo("John Doe"))
            .body("email", equalTo("john@example.com"));
    }
    
    // 5. PUT Request
    @Test
    public void testUpdateUser() {
        String requestBody = """
            {
                "id": 1,
                "name": "Updated Name",
                "username": "updateduser",
                "email": "updated@example.com"
            }
            """;
        
        given()
            .header("Content-Type", "application/json")
            .body(requestBody)
            .when()
            .put("/users/1")
            .then()
            .statusCode(200)
            .body("name", equalTo("Updated Name"));
    }
    
    // 6. PATCH Request
    @Test
    public void testPartialUpdateUser() {
        String requestBody = """
            {
                "name": "Partially Updated"
            }
            """;
        
        given()
            .header("Content-Type", "application/json")
            .body(requestBody)
            .when()
            .patch("/users/1")
            .then()
            .statusCode(200)
            .body("name", equalTo("Partially Updated"));
    }
    
    // 7. DELETE Request
    @Test
    public void testDeleteUser() {
        given()
            .when()
            .delete("/users/1")
            .then()
            .statusCode(200);
    }
    
    // 8. Extract Response Data
    @Test
    public void testExtractResponseData() {
        Response response = given()
            .when()
            .get("/users/1");
        
        // Extract entire response
        String responseBody = response.asString();
        System.out.println("Response: " + responseBody);
        
        // Extract specific fields
        String name = response.jsonPath().getString("name");
        String email = response.jsonPath().getString("email");
        int id = response.jsonPath().getInt("id");
        
        System.out.println("Name: " + name);
        System.out.println("Email: " + email);
        System.out.println("ID: " + id);
        
        // Assert on extracted data
        Assert.assertEquals(id, 1);
        Assert.assertNotNull(name);
    }
    
    // 9. Response Time Validation
    @Test
    public void testResponseTime() {
        given()
            .when()
            .get("/users")
            .then()
            .statusCode(200)
            .time(lessThan(2000L));  // Response time < 2 seconds
    }
    
    // 10. Header Validation
    @Test
    public void testResponseHeaders() {
        given()
            .when()
            .get("/users/1")
            .then()
            .statusCode(200)
            .header("Content-Type", containsString("application/json"))
            .header("Server", notNullValue());
    }
}
```

### Advanced RestAssured Features

```java
public class APIAdvancedTests {
    
    // 1. Request and Response Specification
    private static RequestSpecification requestSpec;
    private static ResponseSpecification responseSpec;
    
    @BeforeClass
    public void setupSpecifications() {
        requestSpec = new RequestSpecBuilder()
            .setBaseUri("https://api.example.com")
            .setBasePath("/api/v1")
            .setContentType(ContentType.JSON)
            .addHeader("Accept", "application/json")
            .build();
        
        responseSpec = new ResponseSpecBuilder()
            .expectStatusCode(200)
            .expectContentType(ContentType.JSON)
            .expectResponseTime(lessThan(3000L))
            .build();
        
        RestAssured.requestSpecification = requestSpec;
        RestAssured.responseSpecification = responseSpec;
    }
    
    @Test
    public void testWithSpecifications() {
        given()
            .spec(requestSpec)
            .when()
            .get("/users")
            .then()
            .spec(responseSpec);
    }
    
    // 2. File Upload
    @Test
    public void testFileUpload() {
        File file = new File("src/test/resources/testfile.pdf");
        
        given()
            .multiPart("file", file)
            .when()
            .post("/upload")
            .then()
            .statusCode(200);
    }
    
    // 3. File Download
    @Test
    public void testFileDownload() {
        byte[] fileBytes = given()
            .when()
            .get("/download/report.pdf")
            .then()
            .statusCode(200)
            .extract()
            .asByteArray();
        
        // Save to file
        try {
            Files.write(Paths.get("downloaded_report.pdf"), fileBytes);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    
    // 4. JSON Schema Validation
    @Test
    public void testJsonSchemaValidation() {
        given()
            .when()
            .get("/users/1")
            .then()
            .statusCode(200)
            .body(matchesJsonSchemaInClasspath("schemas/user-schema.json"));
    }
    
    // 5. Response Body Validation - Complex JSON
    @Test
    public void testComplexJsonValidation() {
        given()
            .when()
            .get("/users/1")
            .then()
            .statusCode(200)
            .body("name", equalTo("John Doe"))
            .body("address.city", equalTo("New York"))
            .body("address.geo.lat", notNullValue())
            .body("company.name", containsString("Corp"));
    }
    
    // 6. Array Validation
    @Test
    public void testArrayValidation() {
        given()
            .when()
            .get("/users")
            .then()
            .statusCode(200)
            .body("id", hasItems(1, 2, 3))
            .body("name", everyItem(notNullValue()))
            .body("email", everyItem(containsString("@")))
            .body("findAll { it.id > 5 }.size()", greaterThan(0));  // Groovy GPath
    }
    
    // 7. Serialization - POJO to JSON
    @Test
    public void testSerializationPOJO() {
        User user = new User();
        user.setName("John Doe");
        user.setEmail("john@example.com");
        user.setUsername("johndoe");
        
        given()
            .contentType(ContentType.JSON)
            .body(user)
            .when()
            .post("/users")
            .then()
            .statusCode(201)
            .body("name", equalTo(user.getName()));
    }
    
    // 8. Deserialization - JSON to POJO
    @Test
    public void testDeserializationPOJO() {
        User user = given()
            .when()
            .get("/users/1")
            .then()
            .statusCode(200)
            .extract()
            .as(User.class);
        
        Assert.assertNotNull(user);
        Assert.assertEquals(user.getId(), 1);
        System.out.println("User: " + user.getName());
    }
    
    // 9. Logging
    @Test
    public void testLogging() {
        given()
            .log().all()  // Log request
            .when()
            .get("/users/1")
            .then()
            .log().all()  // Log response
            .statusCode(200);
        
        // Conditional logging
        given()
            .log().ifValidationFails()
            .when()
            .get("/users/1")
            .then()
            .log().ifError()
            .statusCode(200);
    }
    
    // 10. Filters
    @Test
    public void testFilters() {
        given()
            .filter(new RequestLoggingFilter())
            .filter(new ResponseLoggingFilter())
            .when()
            .get("/users/1")
            .then()
            .statusCode(200);
    }
}

// User POJO
class User {
    private int id;
    private String name;
    private String username;
    private String email;
    private Address address;
    private Company company;
    
    // Getters and Setters
    public int getId() { return id; }
    public void setId(int id) { this.id = id; }
    
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    
    public String getUsername() { return username; }
    public void setUsername(String username) { this.username = username; }
    
    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }
    
    // Other getters/setters
}

class Address {
    private String street;
    private String city;
    private String zipcode;
    private Geo geo;
    
    // Getters and Setters
}

class Geo {
    private String lat;
    private String lng;
    
    // Getters and Setters
}

class Company {
    private String name;
    private String catchPhrase;
    
    // Getters and Setters
}
```

---

## API Authentication

### 1. Basic Authentication

```java
public class BasicAuthTest {
    
    @Test
    public void testBasicAuth() {
        given()
            .auth().basic("username", "password")
            .when()
            .get("https://api.example.com/secure")
            .then()
            .statusCode(200);
    }
    
    // Preemptive Basic Auth
    @Test
    public void testPreemptiveBasicAuth() {
        given()
            .auth().preemptive().basic("username", "password")
            .when()
            .get("https://api.example.com/secure")
            .then()
            .statusCode(200);
    }
}
```

### 2. Bearer Token Authentication

```java
public class BearerTokenTest {
    private String token;
    
    @BeforeClass
    public void getToken() {
        // Get token from login endpoint
        String loginBody = """
            {
                "username": "testuser",
                "password": "testpass"
            }
            """;
        
        token = given()
            .contentType(ContentType.JSON)
            .body(loginBody)
            .when()
            .post("https://api.example.com/login")
            .then()
            .statusCode(200)
            .extract()
            .jsonPath()
            .getString("token");
    }
    
    @Test
    public void testWithBearerToken() {
        given()
            .auth().oauth2(token)
            .when()
            .get("https://api.example.com/protected")
            .then()
            .statusCode(200);
    }
    
    // Alternative: Using header
    @Test
    public void testWithAuthorizationHeader() {
        given()
            .header("Authorization", "Bearer " + token)
            .when()
            .get("https://api.example.com/protected")
            .then()
            .statusCode(200);
    }
}
```

### 3. OAuth 2.0 Authentication

```java
public class OAuth2Test {
    
    @Test
    public void testOAuth2() {
        given()
            .auth()
            .oauth2("YOUR_ACCESS_TOKEN")
            .when()
            .get("https://api.example.com/resource")
            .then()
            .statusCode(200);
    }
    
    // Get OAuth2 token
    public String getOAuth2Token() {
        return given()
            .formParam("grant_type", "client_credentials")
            .formParam("client_id", "your_client_id")
            .formParam("client_secret", "your_client_secret")
            .when()
            .post("https://oauth.example.com/token")
            .then()
            .statusCode(200)
            .extract()
            .jsonPath()
            .getString("access_token");
    }
}
```

### 4. API Key Authentication

```java
public class APIKeyTest {
    
    @Test
    public void testAPIKeyInHeader() {
        given()
            .header("X-API-Key", "your_api_key")
            .when()
            .get("https://api.example.com/data")
            .then()
            .statusCode(200);
    }
    
    @Test
    public void testAPIKeyInQueryParam() {
        given()
            .queryParam("api_key", "your_api_key")
            .when()
            .get("https://api.example.com/data")
            .then()
            .statusCode(200);
    }
}
```

---

## JSON/XML Parsing

### JSON Parsing with JsonPath

```java
import io.restassured.path.json.JsonPath;

public class JsonParsingTest {
    
    @Test
    public void testJsonParsing() {
        String jsonResponse = """
            {
                "id": 1,
                "name": "John Doe",
                "email": "john@example.com",
                "address": {
                    "city": "New York",
                    "zipcode": "10001"
                },
                "orders": [
                    {"id": 101, "amount": 250.50},
                    {"id": 102, "amount": 180.75}
                ]
            }
            """;
        
        JsonPath jp = new JsonPath(jsonResponse);
        
        // Simple values
        int id = jp.getInt("id");
        String name = jp.getString("name");
        String email = jp.getString("email");
        
        // Nested values
        String city = jp.getString("address.city");
        String zipcode = jp.getString("address.zipcode");
        
        // Array values
        int firstOrderId = jp.getInt("orders[0].id");
        double firstOrderAmount = jp.getDouble("orders[0].amount");
        
        // Get all order IDs
        List<Integer> orderIds = jp.getList("orders.id");
        
        // Get all order amounts
        List<Double> amounts = jp.getList("orders.amount");
        
        // Sum of all amounts
        double totalAmount = jp.getDouble("orders.amount.sum()");
        
        System.out.println("Total Amount: " + totalAmount);
    }
    
    @Test
    public void testComplexJsonParsing() {
        Response response = given()
            .when()
            .get("https://api.example.com/users");
        
        JsonPath jp = response.jsonPath();
        
        // Get user with specific ID
        String userName = jp.getString("find { it.id == 1 }.name");
        
        // Get all users with age > 30
        List<String> names = jp.getList("findAll { it.age > 30 }.name");
        
        // Count users in a city
        int count = jp.getInt("findAll { it.address.city == 'New York' }.size()");
        
        System.out.println("Users in New York: " + count);
    }
}
```

### XML Parsing with XmlPath

```java
import io.restassured.path.xml.XmlPath;

public class XmlParsingTest {
    
    @Test
    public void testXmlParsing() {
        String xmlResponse = """
            <user>
                <id>1</id>
                <name>John Doe</name>
                <email>john@example.com</email>
                <address>
                    <city>New York</city>
                    <zipcode>10001</zipcode>
                </address>
                <orders>
                    <order>
                        <id>101</id>
                        <amount>250.50</amount>
                    </order>
                    <order>
                        <id>102</id>
                        <amount>180.75</amount>
                    </order>
                </orders>
            </user>
            """;
        
        XmlPath xp = new XmlPath(xmlResponse);
        
        // Simple values
        int id = xp.getInt("user.id");
        String name = xp.getString("user.name");
        
        // Nested values
        String city = xp.getString("user.address.city");
        
        // Array values
        int firstOrderId = xp.getInt("user.orders.order[0].id");
        
        // Get all order IDs
        List<Integer> orderIds = xp.getList("user.orders.order.id");
        
        System.out.println("Order IDs: " + orderIds);
    }
}
```

---

## API Test Automation

### Complete API Test Framework

```java
// API Base Class
package api.base;

import io.restassured.RestAssured;
import io.restassured.specification.RequestSpecification;
import io.restassured.specification.ResponseSpecification;
import io.restassured.builder.*;
import org.testng.annotations.BeforeClass;

public class APIBaseTest {
    protected RequestSpecification requestSpec;
    protected ResponseSpecification responseSpec;
    
    @BeforeClass
    public void setupAPI() {
        RestAssured.baseURI = ConfigReader.getProperty("api.base.url");
        
        requestSpec = new RequestSpecBuilder()
            .setContentType(ContentType.JSON)
            .setAccept(ContentType.JSON)
            .addHeader("X-API-Key", ConfigReader.getProperty("api.key"))
            .build();
        
        responseSpec = new ResponseSpecBuilder()
            .expectContentType(ContentType.JSON)
            .expectResponseTime(lessThan(5000L))
            .build();
    }
    
    protected String getAuthToken() {
        // Implement token retrieval logic
        return "sample_token";
    }
}

// API Client
package api.clients;

import static io.restassured.RestAssured.*;

public class UserAPIClient extends APIBaseTest {
    private static final String USERS_ENDPOINT = "/users";
    
    public Response getAllUsers() {
        return given()
            .spec(requestSpec)
            .when()
            .get(USERS_ENDPOINT)
            .then()
            .spec(responseSpec)
            .extract()
            .response();
    }
    
    public Response getUserById(int userId) {
        return given()
            .spec(requestSpec)
            .pathParam("userId", userId)
            .when()
            .get(USERS_ENDPOINT + "/{userId}")
            .then()
            .spec(responseSpec)
            .extract()
            .response();
    }
    
    public Response createUser(User user) {
        return given()
            .spec(requestSpec)
            .body(user)
            .when()
            .post(USERS_ENDPOINT)
            .then()
            .extract()
            .response();
    }
    
    public Response updateUser(int userId, User user) {
        return given()
            .spec(requestSpec)
            .pathParam("userId", userId)
            .body(user)
            .when()
            .put(USERS_ENDPOINT + "/{userId}")
            .then()
            .spec(responseSpec)
            .extract()
            .response();
    }
    
    public Response deleteUser(int userId) {
        return given()
            .spec(requestSpec)
            .pathParam("userId", userId)
            .when()
            .delete(USERS_ENDPOINT + "/{userId}")
            .then()
            .extract()
            .response();
    }
}

// API Tests
package api.tests;

import org.testng.annotations.Test;
import api.clients.UserAPIClient;
import models.User;

public class UserAPITest extends APIBaseTest {
    private UserAPIClient userClient = new UserAPIClient();
    
    @Test(priority = 1)
    public void testGetAllUsers() {
        Response response = userClient.getAllUsers();
        
        Assert.assertEquals(response.getStatusCode(), 200);
        List<User> users = response.jsonPath().getList("", User.class);
        Assert.assertTrue(users.size() > 0);
    }
    
    @Test(priority = 2)
    public void testCreateUser() {
        User newUser = new User();
        newUser.setName("Test User");
        newUser.setEmail("test@example.com");
        newUser.setUsername("testuser");
        
        Response response = userClient.createUser(newUser);
        
        Assert.assertEquals(response.getStatusCode(), 201);
        User createdUser = response.as(User.class);
        Assert.assertEquals(createdUser.getName(), newUser.getName());
    }
    
    @Test(priority = 3)
    public void testGetUserById() {
        Response response = userClient.getUserById(1);
        
        Assert.assertEquals(response.getStatusCode(), 200);
        User user = response.as(User.class);
        Assert.assertEquals(user.getId(), 1);
    }
    
    @Test(priority = 4)
    public void testUpdateUser() {
        User updatedUser = new User();
        updatedUser.setName("Updated Name");
        updatedUser.setEmail("updated@example.com");
        
        Response response = userClient.updateUser(1, updatedUser);
        
        Assert.assertEquals(response.getStatusCode(), 200);
        User user = response.as(User.class);
        Assert.assertEquals(user.getName(), "Updated Name");
    }
    
    @Test(priority = 5)
    public void testDeleteUser() {
        Response response = userClient.deleteUser(1);
        
        Assert.assertEquals(response.getStatusCode(), 200);
    }
    
    @Test
    public void testInvalidUser() {
        Response response = userClient.getUserById(99999);
        
        Assert.assertEquals(response.getStatusCode(), 404);
    }
}
```

---

## Postman & Newman

### Postman Collections

```javascript
// Sample Postman Pre-request Script
pm.environment.set("timestamp", new Date().getTime());
pm.environment.set("random_email", `user_${Math.random()}@example.com`);

// Sample Postman Test Script
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

pm.test("Response time is less than 500ms", function () {
    pm.expect(pm.response.responseTime).to.be.below(500);
});

pm.test("User has required fields", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData).to.have.property('id');
    pm.expect(jsonData).to.have.property('name');
    pm.expect(jsonData).to.have.property('email');
});

pm.test("Email format is valid", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.email).to.match(/^[\w-\.]+@([\w-]+\.)+[\w-]{2,4}$/);
});

// Save response data to environment
var jsonData = pm.response.json();
pm.environment.set("user_id", jsonData.id);
pm.environment.set("auth_token", jsonData.token);
```

### Newman (Postman CLI)

```bash
# Install Newman
npm install -g newman

# Run collection
newman run collection.json

# Run with environment
newman run collection.json -e environment.json

# Run with specific folder
newman run collection.json --folder "User Tests"

# Generate HTML report
npm install -g newman-reporter-htmlextra
newman run collection.json -r htmlextra --reporter-htmlextra-export report.html

# Run with iterations
newman run collection.json -n 10

# Run with delay
newman run collection.json --delay-request 1000

# Integration with Jenkins
newman run collection.json --reporters cli,json --reporter-json-export results.json
```

---

---

## 📝 Chapter Summary

### What You Learned

**1. REST API Fundamentals**
- ✅ HTTP Methods: GET, POST, PUT, PATCH, DELETE
- ✅ Status Codes: 2xx (success), 4xx (client error), 5xx (server error)
- ✅ REST principles and best practices

**2. RestAssured Framework**
- ✅ Given-When-Then syntax (readable tests)
- ✅ Request specifications for reusability
- ✅ Response validation with Hamcrest matchers
- ✅ JSON Path and XML Path

**3. Authentication**
- ✅ Basic Auth (username:password)
- ✅ Bearer Token (JWT most common)
- ✅ OAuth 2.0 (complex but secure)
- ✅ API Keys (simple but less secure)

**4. JSON/XML Parsing**
- ✅ Extract values from responses
- ✅ Validate nested structures
- ✅ Schema validation
- ✅ POJO serialization/deserialization

**5. API Test Automation**
- ✅ Complete CRUD test suite
- ✅ Chaining requests (use response in next request)
- ✅ Data-driven API tests
- ✅ Error handling and negative testing

---

### 🎯 Interview Quick Tips

**Most Asked Questions:**

1. **"REST vs SOAP - which is better?"**
   → "REST for most cases - simpler, faster, uses JSON. SOAP for enterprise apps needing strict contracts and security (banking, healthcare)."

2. **"How do you test APIs?"**
   → "RestAssured for Java automation. Test CRUD operations, validate status codes, response schema, headers. Include negative tests and boundary conditions."

3. **"What status codes do you commonly see?"**
   → "200 OK, 201 Created, 204 No Content, 400 Bad Request, 401 Unauthorized, 404 Not Found, 500 Server Error."

4. **"How do you handle authentication in API tests?"**
   → "Store tokens in variables, use RequestSpecification for reusability. Never hardcode credentials. Use environment variables."

5. **"API testing vs UI testing - which do you prefer?"**
   → "API testing 70%, UI 30%. APIs are faster, more stable, test business logic directly. UI tests only for user workflows."

---

### 💡 Best Practices Checklist

✅ **Use Given-When-Then** - Readable test structure  
✅ **Validate Everything** - Status code, headers, body, schema  
✅ **Negative Testing** - Invalid data, missing fields, wrong auth  
✅ **Environment Variables** - URLs, credentials externalized  
✅ **Request Specification** - Reuse common settings  
✅ **Log Request/Response** - Easy debugging  
✅ **Schema Validation** - Catch contract changes early  
✅ **Response Time Assertions** - Performance testing  
✅ **Chaining Tests** - Real user scenarios  
✅ **Data Cleanup** - Delete test data after tests  

---

### ⚠️ Common Mistakes to Avoid

❌ Not validating status codes  
❌ Only testing happy path  
❌ Hardcoding URLs and credentials  
❌ Not checking response schema  
❌ Ignoring performance (response time)  
❌ No negative testing  
❌ Not cleaning up test data  
❌ Testing only with Postman (no automation)  
❌ Mixing test data with production data  
❌ Not handling authentication properly  

---

### 🚀 Real-World Example

**Complete API Test Suite:**

```java
public class UserAPITest {
    private static RequestSpecification requestSpec;
    private static String userId;
    
    @BeforeClass
    public static void setup() {
        RestAssured.baseURI = "https://api.example.com";
        
        requestSpec = new RequestSpecBuilder()
            .setContentType(ContentType.JSON)
            .addHeader("Authorization", "Bearer " + getToken())
            .setRelaxedHTTPSValidation()
            .build();
    }
    
    @Test(priority = 1)
    public void testCreateUser() {
        User user = new User("John Doe", "john@example.com");
        
        userId = given()
            .spec(requestSpec)
            .body(user)
        .when()
            .post("/users")
        .then()
            .statusCode(201)
            .body("name", equalTo("John Doe"))
            .body("email", equalTo("john@example.com"))
            .time(lessThan(2000L))
        .extract()
            .path("id");
    }
    
    @Test(priority = 2, dependsOnMethods = "testCreateUser")
    public void testGetUser() {
        given()
            .spec(requestSpec)
        .when()
            .get("/users/" + userId)
        .then()
            .statusCode(200)
            .body("id", equalTo(userId))
            .body("name", equalTo("John Doe"));
    }
    
    @Test(priority = 3, dependsOnMethods = "testGetUser")
    public void testUpdateUser() {
        User updatedUser = new User("John Updated", "john@example.com");
        
        given()
            .spec(requestSpec)
            .body(updatedUser)
        .when()
            .put("/users/" + userId)
        .then()
            .statusCode(200)
            .body("name", equalTo("John Updated"));
    }
    
    @Test(priority = 4, dependsOnMethods = "testUpdateUser")
    public void testDeleteUser() {
        given()
            .spec(requestSpec)
        .when()
            .delete("/users/" + userId)
        .then()
            .statusCode(204);
        
        // Verify deletion
        given()
            .spec(requestSpec)
        .when()
            .get("/users/" + userId)
        .then()
            .statusCode(404);
    }
    
    @Test
    public void testNegativeScenarios() {
        // Test 1: Create user with invalid email
        User invalidUser = new User("Test", "invalid-email");
        
        given()
            .spec(requestSpec)
            .body(invalidUser)
        .when()
            .post("/users")
        .then()
            .statusCode(400)
            .body("error", containsString("Invalid email"));
        
        // Test 2: Unauthorized access
        given()
            .contentType(ContentType.JSON)
            // No auth header
        .when()
            .get("/users")
        .then()
            .statusCode(401);
        
        // Test 3: Non-existent resource
        given()
            .spec(requestSpec)
        .when()
            .get("/users/99999")
        .then()
            .statusCode(404);
    }
}
```

---

### 📊 Testing Pyramid

```
       /\
      /UI\ ← 10% (E2E user workflows)
     /────\
    / API  \ ← 70% (Business logic, integrations)
   /────────\
  /   Unit   \ ← 20% (Individual functions)
 /____________\
```

**Focus on API layer:**
- Faster execution (milliseconds vs seconds)
- More stable (no browser issues)
- Better coverage (test all scenarios easily)
- Earlier feedback (test during development)

---

### 💰 API Testing ROI

**Metrics to Track:**
- **Test Execution Time**: API tests 10x faster than UI
- **Stability**: API tests 90%+ stable vs 70% for UI
- **Coverage**: Can test 100s of scenarios easily
- **Defect Detection**: Find bugs earlier (during API development)

**Example:**
- UI Test: 30 seconds per test, 70% stable
- API Test: 3 seconds per test, 95% stable
- **Result**: 10x faster, 25% more reliable!

---

### 🔧 Tools Comparison

| Tool | Language | Best For | Learning Curve |
|------|----------|----------|----------------|
| **RestAssured** | Java | Automation frameworks | Medium |
| **Postman** | GUI/JavaScript | Manual testing, exploration | Easy |
| **Karate** | DSL | BDD-style API tests | Easy |
| **Rest-Assured** | Python | Python projects | Medium |
| **SoapUI** | GUI | SOAP services, Enterprise | Medium |

**Recommendation for Java SDETs:** RestAssured (industry standard)

---

### 📚 What's Next?

Now that you know API testing, you're ready for:
- **Database Testing** (Validate data saved correctly)
- **Performance Testing** (Load test your APIs)
- **Microservices Testing** (Test service interactions)
- **Contract Testing** (Pact for microservices)

**Practice Tip:** Pick a public API (GitHub, OpenWeather) and write complete CRUD tests!

---

**Next:** [Database & SQL](06-database-sql.md)


