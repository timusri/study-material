# 1. Core Java (Advanced Level)

## Table of Contents
- [Multi-threading and Concurrency](#multi-threading-and-concurrency)
- [Collections Framework](#collections-framework)
- [Java 8+ Features](#java-8-features)
- [Exception Handling](#exception-handling)
- [SOLID Principles & Design Patterns](#solid-principles--design-patterns)
- [Memory Management & Garbage Collection](#memory-management--garbage-collection)
- [Generics and Reflection](#generics-and-reflection)

---

## Multi-threading and Concurrency

### Concepts
- **Thread**: Lightweight process that runs concurrently
- **Concurrency**: Multiple threads making progress
- **Synchronization**: Controlling access to shared resources
- **Thread Safety**: Ensuring correct behavior in multi-threaded environments

### Creating Threads

```java
// Method 1: Extending Thread class
class MyThread extends Thread {
    @Override
    public void run() {
        System.out.println("Thread running: " + Thread.currentThread().getName());
    }
}

// Method 2: Implementing Runnable (Preferred)
class MyRunnable implements Runnable {
    @Override
    public void run() {
        System.out.println("Runnable running: " + Thread.currentThread().getName());
    }
}

// Method 3: Using Lambda (Java 8+)
public class ThreadExample {
    public static void main(String[] args) {
        // Using Thread
        Thread t1 = new MyThread();
        t1.start();
        
        // Using Runnable
        Thread t2 = new Thread(new MyRunnable());
        t2.start();
        
        // Using Lambda
        Thread t3 = new Thread(() -> {
            System.out.println("Lambda thread: " + Thread.currentThread().getName());
        });
        t3.start();
    }
}
```

### Thread Synchronization

```java
public class BankAccount {
    private int balance = 1000;
    
    // Synchronized method
    public synchronized void withdraw(int amount) {
        if (balance >= amount) {
            System.out.println(Thread.currentThread().getName() + " is withdrawing " + amount);
            balance -= amount;
            System.out.println("Balance after withdrawal: " + balance);
        } else {
            System.out.println("Insufficient balance");
        }
    }
    
    // Synchronized block (more granular control)
    public void deposit(int amount) {
        synchronized(this) {
            balance += amount;
            System.out.println("Deposited: " + amount + ", New balance: " + balance);
        }
    }
}
```

### Thread Pool Implementation

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.TimeUnit;

public class ThreadPoolExample {
    public static void main(String[] args) {
        // Create fixed thread pool
        ExecutorService executor = Executors.newFixedThreadPool(5);
        
        // Submit 10 tasks
        for (int i = 1; i <= 10; i++) {
            final int taskId = i;
            executor.submit(() -> {
                System.out.println("Task " + taskId + " is running on " + 
                                   Thread.currentThread().getName());
                try {
                    Thread.sleep(2000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            });
        }
        
        // Shutdown executor
        executor.shutdown();
        try {
            executor.awaitTermination(1, TimeUnit.MINUTES);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

### Custom Thread Pool

```java
import java.util.concurrent.*;

public class CustomThreadPool {
    private final BlockingQueue<Runnable> taskQueue;
    private final Thread[] workerThreads;
    private volatile boolean isShutdown = false;
    
    public CustomThreadPool(int poolSize, int queueSize) {
        taskQueue = new LinkedBlockingQueue<>(queueSize);
        workerThreads = new Thread[poolSize];
        
        for (int i = 0; i < poolSize; i++) {
            workerThreads[i] = new WorkerThread();
            workerThreads[i].start();
        }
    }
    
    public void submit(Runnable task) throws InterruptedException {
        if (isShutdown) {
            throw new IllegalStateException("ThreadPool is shutdown");
        }
        taskQueue.put(task);
    }
    
    public void shutdown() {
        isShutdown = true;
        for (Thread thread : workerThreads) {
            thread.interrupt();
        }
    }
    
    private class WorkerThread extends Thread {
        @Override
        public void run() {
            while (!isShutdown) {
                try {
                    Runnable task = taskQueue.take();
                    task.run();
                } catch (InterruptedException e) {
                    break;
                }
            }
        }
    }
}
```

### Producer-Consumer Pattern

```java
import java.util.LinkedList;
import java.util.Queue;

public class ProducerConsumer {
    private final Queue<Integer> queue = new LinkedList<>();
    private final int MAX_SIZE = 10;
    
    public void produce() throws InterruptedException {
        int value = 0;
        while (true) {
            synchronized (queue) {
                while (queue.size() == MAX_SIZE) {
                    queue.wait();
                }
                System.out.println("Producing: " + value);
                queue.add(value++);
                queue.notifyAll();
                Thread.sleep(100);
            }
        }
    }
    
    public void consume() throws InterruptedException {
        while (true) {
            synchronized (queue) {
                while (queue.isEmpty()) {
                    queue.wait();
                }
                int value = queue.poll();
                System.out.println("Consuming: " + value);
                queue.notifyAll();
                Thread.sleep(500);
            }
        }
    }
    
    public static void main(String[] args) {
        ProducerConsumer pc = new ProducerConsumer();
        
        Thread producer = new Thread(() -> {
            try {
                pc.produce();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
        
        Thread consumer = new Thread(() -> {
            try {
                pc.consume();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
        
        producer.start();
        consumer.start();
    }
}
```

---

## Collections Framework

### HashMap vs ConcurrentHashMap

```java
import java.util.*;
import java.util.concurrent.ConcurrentHashMap;

public class HashMapComparison {
    public static void main(String[] args) {
        // HashMap - Not thread-safe
        Map<String, String> hashMap = new HashMap<>();
        hashMap.put("key1", "value1");
        hashMap.put("key2", "value2");
        hashMap.put(null, "nullValue");  // Allows null key
        
        // ConcurrentHashMap - Thread-safe
        Map<String, String> concurrentMap = new ConcurrentHashMap<>();
        concurrentMap.put("key1", "value1");
        concurrentMap.put("key2", "value2");
        // concurrentMap.put(null, "value"); // Will throw NullPointerException
        
        // Performance comparison
        testPerformance();
    }
    
    public static void testPerformance() {
        Map<Integer, Integer> hashMap = new HashMap<>();
        Map<Integer, Integer> concurrentMap = new ConcurrentHashMap<>();
        
        // Single-threaded performance
        long start = System.currentTimeMillis();
        for (int i = 0; i < 1000000; i++) {
            hashMap.put(i, i);
        }
        System.out.println("HashMap: " + (System.currentTimeMillis() - start) + "ms");
        
        start = System.currentTimeMillis();
        for (int i = 0; i < 1000000; i++) {
            concurrentMap.put(i, i);
        }
        System.out.println("ConcurrentHashMap: " + (System.currentTimeMillis() - start) + "ms");
    }
}
```

### Collection Deep Dive

```java
import java.util.*;

public class CollectionsExamples {
    
    // ArrayList vs LinkedList
    public void listComparison() {
        // ArrayList - Fast random access, slow insertion/deletion
        List<String> arrayList = new ArrayList<>();
        arrayList.add("Element1");
        arrayList.add("Element2");
        System.out.println("Get element: " + arrayList.get(0)); // O(1)
        
        // LinkedList - Slow random access, fast insertion/deletion
        List<String> linkedList = new LinkedList<>();
        linkedList.add("Element1");
        linkedList.add("Element2");
        System.out.println("Get element: " + linkedList.get(0)); // O(n)
    }
    
    // HashSet vs TreeSet vs LinkedHashSet
    public void setComparison() {
        // HashSet - No order, O(1) operations
        Set<String> hashSet = new HashSet<>();
        hashSet.add("Banana");
        hashSet.add("Apple");
        hashSet.add("Cherry");
        System.out.println("HashSet: " + hashSet); // Random order
        
        // TreeSet - Sorted order, O(log n) operations
        Set<String> treeSet = new TreeSet<>();
        treeSet.add("Banana");
        treeSet.add("Apple");
        treeSet.add("Cherry");
        System.out.println("TreeSet: " + treeSet); // [Apple, Banana, Cherry]
        
        // LinkedHashSet - Insertion order, O(1) operations
        Set<String> linkedHashSet = new LinkedHashSet<>();
        linkedHashSet.add("Banana");
        linkedHashSet.add("Apple");
        linkedHashSet.add("Cherry");
        System.out.println("LinkedHashSet: " + linkedHashSet); // [Banana, Apple, Cherry]
    }
    
    // Custom Comparator
    public void customSorting() {
        List<Employee> employees = Arrays.asList(
            new Employee("John", 30, 50000),
            new Employee("Alice", 25, 60000),
            new Employee("Bob", 35, 55000)
        );
        
        // Sort by salary
        Collections.sort(employees, (e1, e2) -> 
            Double.compare(e1.getSalary(), e2.getSalary()));
        
        // Sort by multiple fields
        Collections.sort(employees, 
            Comparator.comparing(Employee::getAge)
                      .thenComparing(Employee::getName));
        
        employees.forEach(System.out::println);
    }
}

class Employee {
    private String name;
    private int age;
    private double salary;
    
    public Employee(String name, int age, double salary) {
        this.name = name;
        this.age = age;
        this.salary = salary;
    }
    
    // Getters
    public String getName() { return name; }
    public int getAge() { return age; }
    public double getSalary() { return salary; }
    
    @Override
    public String toString() {
        return "Employee{name='" + name + "', age=" + age + ", salary=" + salary + "}";
    }
}
```

---

## Java 8+ Features

### Lambda Expressions

```java
import java.util.*;
import java.util.function.*;

public class LambdaExamples {
    
    public static void main(String[] args) {
        // Basic Lambda
        Runnable r = () -> System.out.println("Hello Lambda");
        r.run();
        
        // Lambda with parameters
        BiFunction<Integer, Integer, Integer> add = (a, b) -> a + b;
        System.out.println("Sum: " + add.apply(5, 3));
        
        // Functional Interfaces
        demoFunctionalInterfaces();
        
        // Method References
        demoMethodReferences();
    }
    
    public static void demoFunctionalInterfaces() {
        // Predicate - Takes input, returns boolean
        Predicate<Integer> isEven = num -> num % 2 == 0;
        System.out.println("Is 4 even? " + isEven.test(4));
        
        // Consumer - Takes input, returns nothing
        Consumer<String> print = str -> System.out.println(str.toUpperCase());
        print.accept("hello");
        
        // Supplier - Takes nothing, returns output
        Supplier<Double> random = () -> Math.random();
        System.out.println("Random: " + random.get());
        
        // Function - Takes input, returns output
        Function<String, Integer> length = str -> str.length();
        System.out.println("Length: " + length.apply("Hello"));
    }
    
    public static void demoMethodReferences() {
        List<String> names = Arrays.asList("John", "Alice", "Bob");
        
        // Static method reference
        names.forEach(System.out::println);
        
        // Instance method reference
        names.stream()
             .map(String::toUpperCase)
             .forEach(System.out::println);
        
        // Constructor reference
        Supplier<ArrayList<String>> listSupplier = ArrayList::new;
        ArrayList<String> list = listSupplier.get();
    }
}
```

### Stream API

```java
import java.util.*;
import java.util.stream.*;

public class StreamExamples {
    
    public static void main(String[] args) {
        List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
        
        // Filter and Map
        List<Integer> evenSquares = numbers.stream()
            .filter(n -> n % 2 == 0)
            .map(n -> n * n)
            .collect(Collectors.toList());
        System.out.println("Even squares: " + evenSquares);
        
        // Reduce
        int sum = numbers.stream()
            .reduce(0, (a, b) -> a + b);
        System.out.println("Sum: " + sum);
        
        // Complex operations
        complexStreamOperations();
        
        // Parallel Streams
        parallelStreamExample();
    }
    
    public static void complexStreamOperations() {
        List<Employee> employees = Arrays.asList(
            new Employee("John", 30, 50000),
            new Employee("Alice", 25, 60000),
            new Employee("Bob", 35, 55000),
            new Employee("Charlie", 28, 52000)
        );
        
        // Group by age range
        Map<String, List<Employee>> groupedByAge = employees.stream()
            .collect(Collectors.groupingBy(e -> {
                if (e.getAge() < 30) return "Young";
                else return "Senior";
            }));
        
        // Average salary
        double avgSalary = employees.stream()
            .mapToDouble(Employee::getSalary)
            .average()
            .orElse(0.0);
        System.out.println("Average salary: " + avgSalary);
        
        // Find highest paid employee
        Optional<Employee> highestPaid = employees.stream()
            .max(Comparator.comparing(Employee::getSalary));
        highestPaid.ifPresent(e -> System.out.println("Highest paid: " + e));
        
        // Partition by salary > 55000
        Map<Boolean, List<Employee>> partitioned = employees.stream()
            .collect(Collectors.partitioningBy(e -> e.getSalary() > 55000));
    }
    
    public static void parallelStreamExample() {
        List<Integer> numbers = IntStream.rangeClosed(1, 1000)
            .boxed()
            .collect(Collectors.toList());
        
        // Sequential
        long start = System.currentTimeMillis();
        long sum = numbers.stream()
            .mapToLong(Integer::longValue)
            .sum();
        System.out.println("Sequential: " + (System.currentTimeMillis() - start) + "ms");
        
        // Parallel
        start = System.currentTimeMillis();
        sum = numbers.parallelStream()
            .mapToLong(Integer::longValue)
            .sum();
        System.out.println("Parallel: " + (System.currentTimeMillis() - start) + "ms");
    }
}
```

### Optional Class

```java
import java.util.Optional;

public class OptionalExample {
    
    public static void main(String[] args) {
        // Creating Optional
        Optional<String> optional = Optional.of("Hello");
        Optional<String> empty = Optional.empty();
        Optional<String> nullable = Optional.ofNullable(null);
        
        // Checking presence
        if (optional.isPresent()) {
            System.out.println(optional.get());
        }
        
        // Better way - ifPresent
        optional.ifPresent(System.out::println);
        
        // orElse
        String value = nullable.orElse("Default Value");
        System.out.println(value);
        
        // orElseGet (lazy evaluation)
        String value2 = nullable.orElseGet(() -> computeDefault());
        
        // orElseThrow
        try {
            String value3 = nullable.orElseThrow(() -> 
                new RuntimeException("Value not present"));
        } catch (RuntimeException e) {
            System.out.println(e.getMessage());
        }
        
        // map and flatMap
        Optional<Integer> length = optional.map(String::length);
        System.out.println("Length: " + length.orElse(0));
        
        // filter
        Optional<String> filtered = optional.filter(s -> s.length() > 3);
        filtered.ifPresent(System.out::println);
    }
    
    public static String computeDefault() {
        System.out.println("Computing default...");
        return "Default";
    }
    
    // Practical example in automation
    public Optional<String> getElementText(String locator) {
        try {
            // WebElement element = driver.findElement(By.xpath(locator));
            // return Optional.ofNullable(element.getText());
            return Optional.of("Sample Text");
        } catch (Exception e) {
            return Optional.empty();
        }
    }
}
```

---

## Exception Handling

### Custom Exceptions

```java
// Custom checked exception
class TestExecutionException extends Exception {
    private String testName;
    private String errorCode;
    
    public TestExecutionException(String message, String testName) {
        super(message);
        this.testName = testName;
    }
    
    public TestExecutionException(String message, Throwable cause, String testName) {
        super(message, cause);
        this.testName = testName;
    }
    
    public String getTestName() {
        return testName;
    }
}

// Custom unchecked exception
class AutomationFrameworkException extends RuntimeException {
    private String componentName;
    
    public AutomationFrameworkException(String message, String componentName) {
        super(message);
        this.componentName = componentName;
    }
    
    public String getComponentName() {
        return componentName;
    }
}

// Usage in automation framework
public class TestExecutor {
    
    public void executeTest(String testName) throws TestExecutionException {
        try {
            // Test execution logic
            if (testName == null || testName.isEmpty()) {
                throw new TestExecutionException(
                    "Test name cannot be null or empty", testName);
            }
            
            // Simulate test execution
            performTestSteps();
            
        } catch (Exception e) {
            throw new TestExecutionException(
                "Test execution failed: " + e.getMessage(), e, testName);
        }
    }
    
    private void performTestSteps() {
        // Test steps
    }
}
```

### Try-with-Resources

```java
import java.io.*;
import java.sql.*;

public class TryWithResourcesExample {
    
    // Automatic resource management
    public void readFile(String path) {
        try (BufferedReader reader = new BufferedReader(new FileReader(path))) {
            String line;
            while ((line = reader.readLine()) != null) {
                System.out.println(line);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        // reader.close() is called automatically
    }
    
    // Multiple resources
    public void copyFile(String source, String dest) {
        try (BufferedReader reader = new BufferedReader(new FileReader(source));
             BufferedWriter writer = new BufferedWriter(new FileWriter(dest))) {
            
            String line;
            while ((line = reader.readLine()) != null) {
                writer.write(line);
                writer.newLine();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    
    // Custom AutoCloseable resource
    static class TestResource implements AutoCloseable {
        private String name;
        
        public TestResource(String name) {
            this.name = name;
            System.out.println("Opening resource: " + name);
        }
        
        public void doWork() {
            System.out.println("Working with: " + name);
        }
        
        @Override
        public void close() {
            System.out.println("Closing resource: " + name);
        }
    }
    
    public void useCustomResource() {
        try (TestResource resource = new TestResource("MyResource")) {
            resource.doWork();
        }
        // close() is called automatically
    }
}
```

---

## SOLID Principles & Design Patterns

### SOLID Principles

```java
// 1. Single Responsibility Principle (SRP)
// Each class should have only one reason to change

// Bad Example
class UserManager {
    public void createUser(String name) { }
    public void saveToDatabase(String data) { }
    public void sendEmail(String email) { }
}

// Good Example
class UserService {
    public void createUser(String name) { }
}

class DatabaseService {
    public void save(String data) { }
}

class EmailService {
    public void sendEmail(String email) { }
}

// 2. Open/Closed Principle (OCP)
// Open for extension, closed for modification

interface ReportGenerator {
    void generate();
}

class PDFReport implements ReportGenerator {
    @Override
    public void generate() {
        System.out.println("Generating PDF report");
    }
}

class ExcelReport implements ReportGenerator {
    @Override
    public void generate() {
        System.out.println("Generating Excel report");
    }
}

// 3. Liskov Substitution Principle (LSP)
// Subtypes must be substitutable for their base types

class Bird {
    public void eat() {
        System.out.println("Eating");
    }
}

class FlyingBird extends Bird {
    public void fly() {
        System.out.println("Flying");
    }
}

class Sparrow extends FlyingBird {
    // Can fly
}

class Penguin extends Bird {
    // Cannot fly, so doesn't extend FlyingBird
}

// 4. Interface Segregation Principle (ISP)
// Clients should not be forced to depend on interfaces they don't use

// Bad Example
interface Worker {
    void work();
    void eat();
    void sleep();
}

// Good Example
interface Workable {
    void work();
}

interface Eatable {
    void eat();
}

interface Sleepable {
    void sleep();
}

class Robot implements Workable {
    @Override
    public void work() {
        System.out.println("Robot working");
    }
}

class Human implements Workable, Eatable, Sleepable {
    @Override
    public void work() {
        System.out.println("Human working");
    }
    
    @Override
    public void eat() {
        System.out.println("Human eating");
    }
    
    @Override
    public void sleep() {
        System.out.println("Human sleeping");
    }
}

// 5. Dependency Inversion Principle (DIP)
// Depend on abstractions, not concretions

interface Database {
    void save(String data);
}

class MySQLDatabase implements Database {
    @Override
    public void save(String data) {
        System.out.println("Saving to MySQL: " + data);
    }
}

class MongoDatabase implements Database {
    @Override
    public void save(String data) {
        System.out.println("Saving to MongoDB: " + data);
    }
}

class UserRepository {
    private Database database;
    
    // Dependency Injection
    public UserRepository(Database database) {
        this.database = database;
    }
    
    public void saveUser(String userData) {
        database.save(userData);
    }
}
```

### Design Patterns for Automation

```java
// 1. Singleton Pattern (WebDriver instance)
public class DriverManager {
    private static DriverManager instance;
    private WebDriver driver;
    
    private DriverManager() {
        // Initialize driver
        // driver = new ChromeDriver();
    }
    
    public static DriverManager getInstance() {
        if (instance == null) {
            synchronized (DriverManager.class) {
                if (instance == null) {
                    instance = new DriverManager();
                }
            }
        }
        return instance;
    }
    
    public WebDriver getDriver() {
        return driver;
    }
}

// 2. Factory Pattern (Browser Factory)
interface WebDriver {
    void navigate(String url);
}

class ChromeDriver implements WebDriver {
    @Override
    public void navigate(String url) {
        System.out.println("Navigating to " + url + " in Chrome");
    }
}

class FirefoxDriver implements WebDriver {
    @Override
    public void navigate(String url) {
        System.out.println("Navigating to " + url + " in Firefox");
    }
}

class BrowserFactory {
    public static WebDriver createDriver(String browserType) {
        switch (browserType.toLowerCase()) {
            case "chrome":
                return new ChromeDriver();
            case "firefox":
                return new FirefoxDriver();
            default:
                throw new IllegalArgumentException("Unknown browser: " + browserType);
        }
    }
}

// 3. Builder Pattern (Test Data Builder)
class User {
    private String username;
    private String password;
    private String email;
    private String firstName;
    private String lastName;
    private int age;
    
    private User(UserBuilder builder) {
        this.username = builder.username;
        this.password = builder.password;
        this.email = builder.email;
        this.firstName = builder.firstName;
        this.lastName = builder.lastName;
        this.age = builder.age;
    }
    
    public static class UserBuilder {
        private String username;
        private String password;
        private String email;
        private String firstName;
        private String lastName;
        private int age;
        
        public UserBuilder(String username, String password) {
            this.username = username;
            this.password = password;
        }
        
        public UserBuilder email(String email) {
            this.email = email;
            return this;
        }
        
        public UserBuilder firstName(String firstName) {
            this.firstName = firstName;
            return this;
        }
        
        public UserBuilder lastName(String lastName) {
            this.lastName = lastName;
            return this;
        }
        
        public UserBuilder age(int age) {
            this.age = age;
            return this;
        }
        
        public User build() {
            return new User(this);
        }
    }
}

// Usage
class TestDataExample {
    public void createUser() {
        User user = new User.UserBuilder("john_doe", "password123")
            .email("john@example.com")
            .firstName("John")
            .lastName("Doe")
            .age(30)
            .build();
    }
}

// 4. Strategy Pattern (Wait Strategies)
interface WaitStrategy {
    void waitForElement(String locator);
}

class ExplicitWaitStrategy implements WaitStrategy {
    @Override
    public void waitForElement(String locator) {
        System.out.println("Using explicit wait for: " + locator);
    }
}

class FluentWaitStrategy implements WaitStrategy {
    @Override
    public void waitForElement(String locator) {
        System.out.println("Using fluent wait for: " + locator);
    }
}

class ElementWaiter {
    private WaitStrategy strategy;
    
    public ElementWaiter(WaitStrategy strategy) {
        this.strategy = strategy;
    }
    
    public void setStrategy(WaitStrategy strategy) {
        this.strategy = strategy;
    }
    
    public void waitForElement(String locator) {
        strategy.waitForElement(locator);
    }
}

// 5. Observer Pattern (Test Listeners)
interface TestListener {
    void onTestStart(String testName);
    void onTestSuccess(String testName);
    void onTestFailure(String testName);
}

class TestReporter implements TestListener {
    @Override
    public void onTestStart(String testName) {
        System.out.println("Test started: " + testName);
    }
    
    @Override
    public void onTestSuccess(String testName) {
        System.out.println("Test passed: " + testName);
    }
    
    @Override
    public void onTestFailure(String testName) {
        System.out.println("Test failed: " + testName);
    }
}

class TestExecutorWithListeners {
    private List<TestListener> listeners = new ArrayList<>();
    
    public void addListener(TestListener listener) {
        listeners.add(listener);
    }
    
    public void executeTest(String testName) {
        notifyTestStart(testName);
        
        try {
            // Execute test
            notifyTestSuccess(testName);
        } catch (Exception e) {
            notifyTestFailure(testName);
        }
    }
    
    private void notifyTestStart(String testName) {
        for (TestListener listener : listeners) {
            listener.onTestStart(testName);
        }
    }
    
    private void notifyTestSuccess(String testName) {
        for (TestListener listener : listeners) {
            listener.onTestSuccess(testName);
        }
    }
    
    private void notifyTestFailure(String testName) {
        for (TestListener listener : listeners) {
            listener.onTestFailure(testName);
        }
    }
}
```

---

## Memory Management & Garbage Collection

### Understanding Memory

```java
public class MemoryExample {
    
    // Stack vs Heap
    public void stackHeapDemo() {
        int x = 10;  // Stack - primitive
        Integer y = 20;  // Heap - object reference in stack, object in heap
        String str = "Hello";  // String pool (special area in heap)
        
        Employee emp = new Employee("John", 30, 50000);  // Heap
    }
    
    // Memory Leak Example
    static class MemoryLeak {
        private static List<byte[]> list = new ArrayList<>();
        
        public void createMemoryLeak() {
            while (true) {
                byte[] array = new byte[1024 * 1024];  // 1MB
                list.add(array);
                // Objects are never removed, causing memory leak
            }
        }
    }
    
    // Preventing Memory Leaks
    static class GoodPractice {
        private List<byte[]> list = new ArrayList<>();
        
        public void properMemoryManagement() {
            byte[] array = new byte[1024 * 1024];
            list.add(array);
            
            // Clear when done
            list.clear();
            list = null;  // Help GC
        }
    }
}
```

### Garbage Collection

```java
public class GarbageCollectionExample {
    
    public static void main(String[] args) {
        // Creating objects
        String s1 = new String("Hello");
        String s2 = new String("World");
        
        // s1 is eligible for GC after this line
        s1 = null;
        
        // Request garbage collection (not guaranteed)
        System.gc();
        
        // Object with finalize method
        demoFinalize();
    }
    
    public static void demoFinalize() {
        TestObject obj = new TestObject("Test");
        obj = null;
        System.gc();
        
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
    
    static class TestObject {
        private String name;
        
        public TestObject(String name) {
            this.name = name;
            System.out.println("Object created: " + name);
        }
        
        @Override
        protected void finalize() throws Throwable {
            System.out.println("Object finalized: " + name);
            super.finalize();
        }
    }
}

// Best practices for automation
class AutomationMemoryManagement {
    private WebDriver driver;
    
    public void setUp() {
        driver = new ChromeDriver();
    }
    
    public void tearDown() {
        if (driver != null) {
            driver.quit();
            driver = null;  // Help GC
        }
    }
    
    // Use try-with-resources for closeable resources
    public void readTestData() {
        try (BufferedReader reader = new BufferedReader(
                new FileReader("testdata.csv"))) {
            // Read data
        } catch (IOException e) {
            e.printStackTrace();
        }
        // Reader automatically closed
    }
}
```

---

## Generics and Reflection

### Generics

```java
// Generic Class
class Box<T> {
    private T content;
    
    public void set(T content) {
        this.content = content;
    }
    
    public T get() {
        return content;
    }
}

// Generic Method
class Utils {
    public static <T> void printArray(T[] array) {
        for (T element : array) {
            System.out.print(element + " ");
        }
        System.out.println();
    }
    
    public static <T extends Comparable<T>> T findMax(T[] array) {
        T max = array[0];
        for (int i = 1; i < array.length; i++) {
            if (array[i].compareTo(max) > 0) {
                max = array[i];
            }
        }
        return max;
    }
}

// Bounded Type Parameters
class Repository<T extends BaseEntity> {
    private List<T> entities = new ArrayList<>();
    
    public void add(T entity) {
        entities.add(entity);
    }
    
    public T findById(int id) {
        return entities.stream()
            .filter(e -> e.getId() == id)
            .findFirst()
            .orElse(null);
    }
}

class BaseEntity {
    private int id;
    
    public int getId() {
        return id;
    }
}

// Wildcard usage
class WildcardExample {
    // Upper bounded wildcard
    public void processNumbers(List<? extends Number> list) {
        for (Number num : list) {
            System.out.println(num.doubleValue());
        }
    }
    
    // Lower bounded wildcard
    public void addIntegers(List<? super Integer> list) {
        list.add(1);
        list.add(2);
    }
    
    // Unbounded wildcard
    public void printList(List<?> list) {
        for (Object obj : list) {
            System.out.println(obj);
        }
    }
}
```

### Reflection

```java
import java.lang.reflect.*;

public class ReflectionExample {
    
    public static void main(String[] args) throws Exception {
        // Get Class object
        Class<?> clazz = Employee.class;
        
        // Get class name
        System.out.println("Class name: " + clazz.getName());
        
        // Get constructors
        Constructor<?>[] constructors = clazz.getConstructors();
        for (Constructor<?> constructor : constructors) {
            System.out.println("Constructor: " + constructor);
        }
        
        // Create instance
        Constructor<?> constructor = clazz.getConstructor(
            String.class, int.class, double.class);
        Employee emp = (Employee) constructor.newInstance("John", 30, 50000);
        
        // Get fields
        Field[] fields = clazz.getDeclaredFields();
        for (Field field : fields) {
            System.out.println("Field: " + field.getName());
        }
        
        // Access private field
        Field nameField = clazz.getDeclaredField("name");
        nameField.setAccessible(true);
        String name = (String) nameField.get(emp);
        System.out.println("Name: " + name);
        
        // Set private field
        nameField.set(emp, "Alice");
        
        // Get methods
        Method[] methods = clazz.getDeclaredMethods();
        for (Method method : methods) {
            System.out.println("Method: " + method.getName());
        }
        
        // Invoke method
        Method getNameMethod = clazz.getMethod("getName");
        String methodResult = (String) getNameMethod.invoke(emp);
        System.out.println("Method result: " + methodResult);
    }
}

// Practical use in automation - Page Factory
class PageFactoryExample {
    
    public static <T> T initElements(WebDriver driver, Class<T> pageClass) {
        try {
            T page = pageClass.getDeclaredConstructor().newInstance();
            
            Field[] fields = pageClass.getDeclaredFields();
            for (Field field : fields) {
                if (field.isAnnotationPresent(FindBy.class)) {
                    FindBy findBy = field.getAnnotation(FindBy.class);
                    field.setAccessible(true);
                    
                    // Create WebElement proxy and inject
                    WebElement element = createElementProxy(driver, findBy);
                    field.set(page, element);
                }
            }
            
            return page;
        } catch (Exception e) {
            throw new RuntimeException("Failed to initialize page", e);
        }
    }
    
    private static WebElement createElementProxy(WebDriver driver, FindBy findBy) {
        // Simplified - actual implementation would use Proxy
        return null;
    }
}

@interface FindBy {
    String id() default "";
    String name() default "";
    String xpath() default "";
}
```

---

## Practice Questions

1. **Write a program to print numbers from 1 to 100 using 3 threads**
2. **Implement a thread-safe Singleton with lazy initialization**
3. **Create a generic method to swap two array elements**
4. **Use Stream API to find employees with salary > 50000 and age < 30**
5. **Implement a custom ArrayList using Generics**
6. **Write a program demonstrating all SOLID principles**
7. **Create a custom exception hierarchy for test automation framework**
8. **Use Reflection to create a simple dependency injection container**

---

**Next:** [Selenium WebDriver](02-selenium-webdriver.md)

