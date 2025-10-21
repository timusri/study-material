# 15. Coding Challenges & Solutions

## Java Coding Challenges

### Challenge 1: Find Duplicate Characters in a String

**Problem:** Write a program to find all duplicate characters in a string along with their count.

**Solution:**

```java
public class DuplicateCharacters {
    
    // Method 1: Using HashMap
    public static void findDuplicatesUsingHashMap(String str) {
        Map<Character, Integer> charCount = new HashMap<>();
        
        // Count occurrences
        for (char c : str.toLowerCase().toCharArray()) {
            if (c != ' ') {  // Ignore spaces
                charCount.put(c, charCount.getOrDefault(c, 0) + 1);
            }
        }
        
        // Print duplicates
        System.out.println("Duplicate characters:");
        charCount.entrySet().stream()
            .filter(entry -> entry.getValue() > 1)
            .forEach(entry -> 
                System.out.println(entry.getKey() + " : " + entry.getValue()));
    }
    
    // Method 2: Using Java 8 Streams
    public static Map<Character, Long> findDuplicatesUsingStreams(String str) {
        return str.toLowerCase()
            .chars()
            .filter(c -> c != ' ')
            .mapToObj(c -> (char) c)
            .collect(Collectors.groupingBy(c -> c, Collectors.counting()))
            .entrySet()
            .stream()
            .filter(entry -> entry.getValue() > 1)
            .collect(Collectors.toMap(Map.Entry::getKey, Map.Entry::getValue));
    }
    
    // Method 3: Using Array (for ASCII characters only)
    public static void findDuplicatesUsingArray(String str) {
        int[] count = new int[256];  // ASCII characters
        
        for (char c : str.toLowerCase().toCharArray()) {
            if (c != ' ') {
                count[c]++;
            }
        }
        
        System.out.println("Duplicate characters:");
        for (int i = 0; i < count.length; i++) {
            if (count[i] > 1) {
                System.out.println((char) i + " : " + count[i]);
            }
        }
    }
    
    public static void main(String[] args) {
        String str = "programming";
        
        System.out.println("Method 1 - HashMap:");
        findDuplicatesUsingHashMap(str);
        
        System.out.println("\nMethod 2 - Streams:");
        Map<Character, Long> duplicates = findDuplicatesUsingStreams(str);
        duplicates.forEach((k, v) -> System.out.println(k + " : " + v));
        
        System.out.println("\nMethod 3 - Array:");
        findDuplicatesUsingArray(str);
    }
}
```

---

### Challenge 2: Reverse Words in a String

**Problem:** Reverse the words in a string without using built-in reverse methods.

**Solution:**

```java
public class ReverseWords {
    
    // Method 1: Using StringBuilder
    public static String reverseWords(String str) {
        String[] words = str.trim().split("\\s+");
        StringBuilder reversed = new StringBuilder();
        
        for (int i = words.length - 1; i >= 0; i--) {
            reversed.append(words[i]);
            if (i != 0) {
                reversed.append(" ");
            }
        }
        
        return reversed.toString();
    }
    
    // Method 2: Using Java 8 Streams
    public static String reverseWordsUsingStreams(String str) {
        return Arrays.stream(str.trim().split("\\s+"))
            .reduce((word1, word2) -> word2 + " " + word1)
            .orElse("");
    }
    
    // Method 3: Using Collections
    public static String reverseWordsUsingCollections(String str) {
        List<String> words = Arrays.asList(str.trim().split("\\s+"));
        Collections.reverse(words);
        return String.join(" ", words);
    }
    
    // Method 4: Reverse each character in the word as well
    public static String reverseWordsAndCharacters(String str) {
        String[] words = str.trim().split("\\s+");
        StringBuilder result = new StringBuilder();
        
        for (int i = words.length - 1; i >= 0; i--) {
            String word = words[i];
            for (int j = word.length() - 1; j >= 0; j--) {
                result.append(word.charAt(j));
            }
            if (i != 0) {
                result.append(" ");
            }
        }
        
        return result.toString();
    }
    
    public static void main(String[] args) {
        String str = "Hello World Java Programming";
        
        System.out.println("Original: " + str);
        System.out.println("Reversed words: " + reverseWords(str));
        System.out.println("Reversed (Streams): " + reverseWordsUsingStreams(str));
        System.out.println("Reversed (Collections): " + reverseWordsUsingCollections(str));
        System.out.println("Reversed words and chars: " + reverseWordsAndCharacters(str));
    }
}
```

---

### Challenge 3: Find Second Highest Number in Array

**Problem:** Find the second highest number in an integer array.

**Solution:**

```java
public class SecondHighest {
    
    // Method 1: Using sorting
    public static int findSecondHighestUsingSorting(int[] arr) {
        if (arr == null || arr.length < 2) {
            throw new IllegalArgumentException("Array must have at least 2 elements");
        }
        
        Arrays.sort(arr);
        
        // Handle duplicates - find unique second highest
        for (int i = arr.length - 2; i >= 0; i--) {
            if (arr[i] != arr[arr.length - 1]) {
                return arr[i];
            }
        }
        
        throw new IllegalArgumentException("All elements are same");
    }
    
    // Method 2: Single pass (O(n) time complexity)
    public static int findSecondHighestSinglePass(int[] arr) {
        if (arr == null || arr.length < 2) {
            throw new IllegalArgumentException("Array must have at least 2 elements");
        }
        
        int highest = Integer.MIN_VALUE;
        int secondHighest = Integer.MIN_VALUE;
        
        for (int num : arr) {
            if (num > highest) {
                secondHighest = highest;
                highest = num;
            } else if (num > secondHighest && num != highest) {
                secondHighest = num;
            }
        }
        
        if (secondHighest == Integer.MIN_VALUE) {
            throw new IllegalArgumentException("No second highest element found");
        }
        
        return secondHighest;
    }
    
    // Method 3: Using Java 8 Streams
    public static int findSecondHighestUsingStreams(int[] arr) {
        return Arrays.stream(arr)
            .boxed()
            .distinct()
            .sorted(Comparator.reverseOrder())
            .skip(1)
            .findFirst()
            .orElseThrow(() -> new IllegalArgumentException("No second highest"));
    }
    
    // Method 4: Using TreeSet (handles duplicates automatically)
    public static int findSecondHighestUsingTreeSet(int[] arr) {
        if (arr == null || arr.length < 2) {
            throw new IllegalArgumentException("Array must have at least 2 elements");
        }
        
        TreeSet<Integer> set = new TreeSet<>();
        for (int num : arr) {
            set.add(num);
        }
        
        if (set.size() < 2) {
            throw new IllegalArgumentException("Not enough unique elements");
        }
        
        set.pollLast();  // Remove highest
        return set.last();  // Return second highest
    }
    
    public static void main(String[] args) {
        int[] arr = {12, 35, 1, 10, 34, 1, 35};
        
        System.out.println("Array: " + Arrays.toString(arr));
        System.out.println("Second Highest (Sorting): " + 
            findSecondHighestUsingSorting(arr));
        System.out.println("Second Highest (Single Pass): " + 
            findSecondHighestSinglePass(arr));
        System.out.println("Second Highest (Streams): " + 
            findSecondHighestUsingStreams(arr));
        System.out.println("Second Highest (TreeSet): " + 
            findSecondHighestUsingTreeSet(arr));
    }
}
```

---

### Challenge 4: Check if String is Palindrome

**Problem:** Check if a given string is a palindrome (ignoring spaces and case).

**Solution:**

```java
public class PalindromeChecker {
    
    // Method 1: Using two pointers
    public static boolean isPalindrome(String str) {
        if (str == null || str.isEmpty()) {
            return true;
        }
        
        str = str.toLowerCase().replaceAll("\\s+", "");
        int left = 0;
        int right = str.length() - 1;
        
        while (left < right) {
            if (str.charAt(left) != str.charAt(right)) {
                return false;
            }
            left++;
            right--;
        }
        
        return true;
    }
    
    // Method 2: Using StringBuilder
    public static boolean isPalindromeUsingStringBuilder(String str) {
        if (str == null || str.isEmpty()) {
            return true;
        }
        
        str = str.toLowerCase().replaceAll("\\s+", "");
        String reversed = new StringBuilder(str).reverse().toString();
        return str.equals(reversed);
    }
    
    // Method 3: Using recursion
    public static boolean isPalindromeRecursive(String str) {
        str = str.toLowerCase().replaceAll("\\s+", "");
        return isPalindromeRecursiveHelper(str, 0, str.length() - 1);
    }
    
    private static boolean isPalindromeRecursiveHelper(String str, int left, int right) {
        if (left >= right) {
            return true;
        }
        
        if (str.charAt(left) != str.charAt(right)) {
            return false;
        }
        
        return isPalindromeRecursiveHelper(str, left + 1, right - 1);
    }
    
    // Method 4: Using Streams
    public static boolean isPalindromeUsingStreams(String str) {
        str = str.toLowerCase().replaceAll("\\s+", "");
        String reversed = str.chars()
            .mapToObj(c -> (char) c)
            .reduce("", (s, c) -> c + s, (s1, s2) -> s2 + s1);
        return str.equals(reversed);
    }
    
    public static void main(String[] args) {
        String[] testStrings = {
            "racecar",
            "A man a plan a canal Panama",
            "hello",
            "Madam",
            "Was it a car or a cat I saw"
        };
        
        for (String test : testStrings) {
            System.out.println("\"" + test + "\" is palindrome? " + 
                isPalindrome(test));
        }
    }
}
```

---

## Selenium Coding Challenges

### Challenge 5: Implement Dynamic Wait Utility

**Problem:** Create a comprehensive wait utility class that handles various wait scenarios.

**Solution:**

```java
public class DynamicWaitUtility {
    private WebDriver driver;
    private WebDriverWait wait;
    private int defaultTimeout;
    
    public DynamicWaitUtility(WebDriver driver, int timeoutInSeconds) {
        this.driver = driver;
        this.defaultTimeout = timeoutInSeconds;
        this.wait = new WebDriverWait(driver, Duration.ofSeconds(timeoutInSeconds));
    }
    
    // Wait for element to be visible
    public WebElement waitForVisibility(By locator) {
        return wait.until(ExpectedConditions.visibilityOfElementLocated(locator));
    }
    
    // Wait for element to be clickable
    public WebElement waitForClickable(By locator) {
        return wait.until(ExpectedConditions.elementToBeClickable(locator));
    }
    
    // Wait for element to be present
    public WebElement waitForPresence(By locator) {
        return wait.until(ExpectedConditions.presenceOfElementLocated(locator));
    }
    
    // Wait for element to be invisible
    public boolean waitForInvisibility(By locator) {
        return wait.until(ExpectedConditions.invisibilityOfElementLocated(locator));
    }
    
    // Wait for text to be present in element
    public boolean waitForTextPresent(By locator, String text) {
        return wait.until(ExpectedConditions.textToBePresentInElementLocated(
            locator, text));
    }
    
    // Wait for attribute value
    public boolean waitForAttributeValue(By locator, String attribute, String value) {
        return wait.until(driver -> {
            try {
                WebElement element = driver.findElement(locator);
                return element.getAttribute(attribute).equals(value);
            } catch (Exception e) {
                return false;
            }
        });
    }
    
    // Wait for page load complete
    public void waitForPageLoad() {
        wait.until(driver -> {
            JavascriptExecutor js = (JavascriptExecutor) driver;
            return js.executeScript("return document.readyState").equals("complete");
        });
    }
    
    // Wait for AJAX/jQuery to complete
    public void waitForAjaxComplete() {
        wait.until(driver -> {
            JavascriptExecutor js = (JavascriptExecutor) driver;
            return (Boolean) js.executeScript("return jQuery.active == 0");
        });
    }
    
    // Wait for element to stop moving (for animated elements)
    public WebElement waitForElementStable(By locator) {
        return wait.until(driver -> {
            WebElement element = driver.findElement(locator);
            Point location1 = element.getLocation();
            
            try {
                Thread.sleep(100);
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
    
    // Wait for element count
    public boolean waitForElementCount(By locator, int expectedCount) {
        return wait.until(driver -> {
            List<WebElement> elements = driver.findElements(locator);
            return elements.size() == expectedCount;
        });
    }
    
    // Wait for URL to contain
    public boolean waitForUrlContains(String urlFragment) {
        return wait.until(ExpectedConditions.urlContains(urlFragment));
    }
    
    // Wait for alert present
    public Alert waitForAlert() {
        return wait.until(ExpectedConditions.alertIsPresent());
    }
    
    // Wait for frame and switch
    public void waitAndSwitchToFrame(By locator) {
        wait.until(ExpectedConditions.frameToBeAvailableAndSwitchToIt(locator));
    }
    
    // Custom wait with polling
    public WebElement waitWithPolling(By locator, int timeoutSeconds, int pollingMillis) {
        Wait<WebDriver> fluentWait = new FluentWait<>(driver)
            .withTimeout(Duration.ofSeconds(timeoutSeconds))
            .pollingEvery(Duration.ofMillis(pollingMillis))
            .ignoring(NoSuchElementException.class)
            .ignoring(StaleElementReferenceException.class);
        
        return fluentWait.until(driver -> driver.findElement(locator));
    }
    
    // Wait with retry on stale element
    public WebElement waitWithStaleRetry(By locator, int maxRetries) {
        int attempts = 0;
        while (attempts < maxRetries) {
            try {
                WebElement element = wait.until(
                    ExpectedConditions.presenceOfElementLocated(locator));
                // Try to interact to ensure it's not stale
                element.isDisplayed();
                return element;
            } catch (StaleElementReferenceException e) {
                attempts++;
                if (attempts >= maxRetries) {
                    throw e;
                }
            }
        }
        throw new RuntimeException("Element not found after " + maxRetries + " attempts");
    }
    
    // Wait for element to be in viewport
    public boolean waitForElementInViewport(By locator) {
        return wait.until(driver -> {
            WebElement element = driver.findElement(locator);
            JavascriptExecutor js = (JavascriptExecutor) driver;
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
        });
    }
    
    // Wait for number of windows
    public boolean waitForNumberOfWindows(int expectedWindows) {
        return wait.until(ExpectedConditions.numberOfWindowsToBe(expectedWindows));
    }
    
    // Wait for element to be selected
    public boolean waitForElementSelected(By locator) {
        return wait.until(ExpectedConditions.elementToBeSelected(locator));
    }
}

// Usage Example
public class WaitUtilityTest {
    
    @Test
    public void testDynamicWait() {
        WebDriver driver = new ChromeDriver();
        DynamicWaitUtility waitUtil = new DynamicWaitUtility(driver, 20);
        
        driver.get("https://example.com");
        
        // Wait for element to be clickable
        WebElement button = waitUtil.waitForClickable(By.id("submitButton"));
        button.click();
        
        // Wait for page load
        waitUtil.waitForPageLoad();
        
        // Wait for specific text
        waitUtil.waitForTextPresent(By.id("message"), "Success");
        
        // Wait for element to stop moving
        WebElement animatedElement = waitUtil.waitForElementStable(
            By.className("animated"));
        
        driver.quit();
    }
}
```

---

### Challenge 6: Handle Dynamic Web Table

**Problem:** Create a robust utility to handle dynamic web tables with sorting, searching, and pagination.

**Solution:**

```java
public class DynamicWebTableHandler {
    private WebDriver driver;
    private By tableLocator;
    private DynamicWaitUtility waitUtil;
    
    public DynamicWebTableHandler(WebDriver driver, By tableLocator) {
        this.driver = driver;
        this.tableLocator = tableLocator;
        this.waitUtil = new DynamicWaitUtility(driver, 20);
    }
    
    // Get all table headers
    public List<String> getHeaders() {
        WebElement table = driver.findElement(tableLocator);
        List<WebElement> headerElements = table.findElements(
            By.xpath(".//thead//th | .//tr[1]//th"));
        
        return headerElements.stream()
            .map(WebElement::getText)
            .collect(Collectors.toList());
    }
    
    // Get row count
    public int getRowCount() {
        WebElement table = driver.findElement(tableLocator);
        List<WebElement> rows = table.findElements(
            By.xpath(".//tbody//tr | .//tr[position()>1]"));
        return rows.size();
    }
    
    // Get column count
    public int getColumnCount() {
        return getHeaders().size();
    }
    
    // Get cell value by row and column index
    public String getCellValue(int row, int column) {
        WebElement table = driver.findElement(tableLocator);
        WebElement cell = table.findElement(
            By.xpath(".//tbody//tr[" + row + "]//td[" + column + "]"));
        return cell.getText();
    }
    
    // Get cell value by row and column name
    public String getCellValueByColumnName(int row, String columnName) {
        List<String> headers = getHeaders();
        int columnIndex = headers.indexOf(columnName) + 1;
        
        if (columnIndex == 0) {
            throw new IllegalArgumentException("Column not found: " + columnName);
        }
        
        return getCellValue(row, columnIndex);
    }
    
    // Get entire row data
    public List<String> getRowData(int row) {
        List<String> rowData = new ArrayList<>();
        int columnCount = getColumnCount();
        
        for (int col = 1; col <= columnCount; col++) {
            rowData.add(getCellValue(row, col));
        }
        
        return rowData;
    }
    
    // Get entire column data
    public List<String> getColumnData(String columnName) {
        List<String> columnData = new ArrayList<>();
        int rowCount = getRowCount();
        
        for (int row = 1; row <= rowCount; row++) {
            columnData.add(getCellValueByColumnName(row, columnName));
        }
        
        return columnData;
    }
    
    // Get all table data as List of Maps
    public List<Map<String, String>> getAllTableData() {
        List<Map<String, String>> tableData = new ArrayList<>();
        List<String> headers = getHeaders();
        int rowCount = getRowCount();
        
        for (int row = 1; row <= rowCount; row++) {
            Map<String, String> rowMap = new HashMap<>();
            
            for (String header : headers) {
                String value = getCellValueByColumnName(row, header);
                rowMap.put(header, value);
            }
            
            tableData.add(rowMap);
        }
        
        return tableData;
    }
    
    // Search for value in specific column
    public List<Integer> searchInColumn(String columnName, String searchValue) {
        List<Integer> matchingRows = new ArrayList<>();
        int rowCount = getRowCount();
        
        for (int row = 1; row <= rowCount; row++) {
            String cellValue = getCellValueByColumnName(row, columnName);
            if (cellValue.equals(searchValue)) {
                matchingRows.add(row);
            }
        }
        
        return matchingRows;
    }
    
    // Search with multiple criteria
    public List<Integer> searchWithCriteria(Map<String, String> criteria) {
        List<Integer> matchingRows = new ArrayList<>();
        int rowCount = getRowCount();
        
        for (int row = 1; row <= rowCount; row++) {
            boolean matches = true;
            
            for (Map.Entry<String, String> criterion : criteria.entrySet()) {
                String columnName = criterion.getKey();
                String expectedValue = criterion.getValue();
                String actualValue = getCellValueByColumnName(row, columnName);
                
                if (!actualValue.equals(expectedValue)) {
                    matches = false;
                    break;
                }
            }
            
            if (matches) {
                matchingRows.add(row);
            }
        }
        
        return matchingRows;
    }
    
    // Click element in specific cell
    public void clickElementInCell(int row, String columnName, By elementLocator) {
        List<String> headers = getHeaders();
        int columnIndex = headers.indexOf(columnName) + 1;
        
        WebElement table = driver.findElement(tableLocator);
        WebElement cell = table.findElement(
            By.xpath(".//tbody//tr[" + row + "]//td[" + columnIndex + "]"));
        
        WebElement element = cell.findElement(elementLocator);
        waitUtil.waitForClickable(elementLocator);
        element.click();
    }
    
    // Sort by column
    public void sortByColumn(String columnName) {
        List<String> headers = getHeaders();
        int columnIndex = headers.indexOf(columnName) + 1;
        
        if (columnIndex == 0) {
            throw new IllegalArgumentException("Column not found: " + columnName);
        }
        
        WebElement table = driver.findElement(tableLocator);
        WebElement header = table.findElement(
            By.xpath(".//thead//th[" + columnIndex + "]"));
        
        header.click();
        
        // Wait for table to update
        try {
            Thread.sleep(1000);  // Or use proper wait
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
    
    // Check if table is sorted
    public boolean isSortedAscending(String columnName) {
        List<String> columnData = getColumnData(columnName);
        
        for (int i = 0; i < columnData.size() - 1; i++) {
            if (columnData.get(i).compareTo(columnData.get(i + 1)) > 0) {
                return false;
            }
        }
        
        return true;
    }
    
    // Handle pagination
    public boolean hasNextPage() {
        try {
            WebElement nextButton = driver.findElement(
                By.xpath("//a[contains(text(), 'Next')] | //button[contains(text(), 'Next')]"));
            return nextButton.isEnabled();
        } catch (NoSuchElementException e) {
            return false;
        }
    }
    
    public void goToNextPage() {
        if (hasNextPage()) {
            WebElement nextButton = driver.findElement(
                By.xpath("//a[contains(text(), 'Next')] | //button[contains(text(), 'Next')]"));
            nextButton.click();
            waitUtil.waitForPageLoad();
        }
    }
    
    public boolean hasPreviousPage() {
        try {
            WebElement prevButton = driver.findElement(
                By.xpath("//a[contains(text(), 'Previous')] | //button[contains(text(), 'Previous')]"));
            return prevButton.isEnabled();
        } catch (NoSuchElementException e) {
            return false;
        }
    }
    
    public void goToPreviousPage() {
        if (hasPreviousPage()) {
            WebElement prevButton = driver.findElement(
                By.xpath("//a[contains(text(), 'Previous')] | //button[contains(text(), 'Previous')]"));
            prevButton.click();
            waitUtil.waitForPageLoad();
        }
    }
    
    // Get data from all pages
    public List<Map<String, String>> getAllDataFromAllPages() {
        List<Map<String, String>> allData = new ArrayList<>();
        
        do {
            allData.addAll(getAllTableData());
        } while (hasNextPage() && (goToNextPage() != null || true));
        
        return allData;
    }
}

// Usage Example
@Test
public void testWebTableOperations() {
    WebDriver driver = new ChromeDriver();
    driver.get("https://example.com/table");
    
    DynamicWebTableHandler table = new DynamicWebTableHandler(
        driver, By.id("dataTable"));
    
    // Get all headers
    List<String> headers = table.getHeaders();
    System.out.println("Headers: " + headers);
    
    // Get specific cell value
    String value = table.getCellValueByColumnName(1, "Name");
    System.out.println("First row name: " + value);
    
    // Search for user
    List<Integer> rows = table.searchInColumn("Name", "John Doe");
    System.out.println("John Doe found in rows: " + rows);
    
    // Search with multiple criteria
    Map<String, String> criteria = new HashMap<>();
    criteria.put("Name", "John Doe");
    criteria.put("Status", "Active");
    List<Integer> matchingRows = table.searchWithCriteria(criteria);
    
    // Click edit button in first matching row
    if (!matchingRows.isEmpty()) {
        table.clickElementInCell(matchingRows.get(0), "Actions", 
            By.xpath(".//button[text()='Edit']"));
    }
    
    // Sort by name
    table.sortByColumn("Name");
    
    // Verify sorted
    boolean isSorted = table.isSortedAscending("Name");
    Assert.assertTrue(isSorted, "Table should be sorted");
    
    // Get all data from all pages
    List<Map<String, String>> allData = table.getAllDataFromAllPages();
    System.out.println("Total records: " + allData.size());
    
    driver.quit();
}
```

---

## Practice Problems

### Additional Coding Challenges

**Java Challenges:**
1. Find the first non-repeated character in a string
2. Implement custom ArrayList class
3. Find all pairs in array that sum to target
4. Implement LRU Cache
5. Check if two strings are anagrams

**Selenium Challenges:**
1. Handle infinite scroll (like Instagram/Twitter feed)
2. Implement screenshot on failure with retry
3. Create custom annotation for test priority
4. Implement parallel test execution framework
5. Handle cookie banner and GDPR popups

**API Testing Challenges:**
1. Implement request/response logging
2. Create token refresh mechanism
3. Implement rate limit handling
4. Create API performance benchmarking
5. Implement request chaining (output of one API as input to another)

---

**Congratulations! You've completed the comprehensive SDET interview preparation guide.**

## Summary

This guide covered:
1. ✅ Core Java (Advanced Level)
2. ✅ Selenium WebDriver (Expert Level)
3. ✅ Test Automation Frameworks
4. ✅ API Testing & Web Services
5. ✅ Interview Q&A (200+ questions)
6. ✅ Coding Challenges & Solutions
7. ✅ System Design for Test Automation
8. ✅ Real-world scenarios

**Next Steps:**
1. Practice coding challenges daily (LeetCode/HackerRank)
2. Build a sample framework and host on GitHub
3. Prepare 2-3 detailed project discussions
4. Practice explaining your solutions
5. Mock interviews with peers
6. Stay updated with latest tools/technologies

**Good luck with your interviews! 🚀**

