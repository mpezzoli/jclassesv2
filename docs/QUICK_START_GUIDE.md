# jclassesv2 Quick Start Guide

## 5-Minute Setup

This guide will get you up and running with jclassesv2 in just a few minutes.

## Prerequisites

- Java 11 or later
- Maven 3.6+ or Gradle 6.0+

## Step 1: Add Dependency

### Maven
Add to your `pom.xml`:
```xml
<dependency>
    <groupId>com.jclassesv2</groupId>
    <artifactId>jclassesv2-core</artifactId>
    <version>2.0.0</version>
</dependency>
```

### Gradle
Add to your `build.gradle`:
```gradle
implementation 'com.jclassesv2:jclassesv2-core:2.0.0'
```

## Step 2: Your First Program

Create a new Java file `QuickStart.java`:

```java
import com.jclassesv2.core.DataProcessor;
import com.jclassesv2.util.CollectionUtils;
import com.jclassesv2.util.StringUtils;
import java.util.Arrays;
import java.util.List;

public class QuickStart {
    public static void main(String[] args) {
        System.out.println("🚀 Starting jclassesv2 Quick Demo");
        
        // 1. Basic Data Processing
        basicProcessing();
        
        // 2. Collection Utilities
        collectionDemo();
        
        // 3. String Utilities
        stringDemo();
        
        System.out.println("✅ Demo completed successfully!");
    }
    
    private static void basicProcessing() {
        System.out.println("\n📊 Basic Data Processing:");
        
        try (DataProcessor processor = new DataProcessor()) {
            List<String> fruits = Arrays.asList("apple", "banana", "cherry");
            
            // Set a custom processor function
            processor.setProcessor(item -> item.toUpperCase() + " 🍎");
            
            List<String> processed = processor.process(fruits);
            System.out.println("Original: " + fruits);
            System.out.println("Processed: " + processed);
            
        } catch (Exception e) {
            System.err.println("Processing failed: " + e.getMessage());
        }
    }
    
    private static void collectionDemo() {
        System.out.println("\n📦 Collection Utilities:");
        
        List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
        
        // Filter even numbers
        List<Integer> evens = CollectionUtils.filter(numbers, n -> n % 2 == 0);
        System.out.println("Even numbers: " + evens);
        
        // Transform to squared values
        List<Integer> squares = CollectionUtils.transform(evens, n -> n * n);
        System.out.println("Squares: " + squares);
        
        // Check if empty
        boolean isEmpty = CollectionUtils.isEmpty(squares);
        System.out.println("Is empty: " + isEmpty);
    }
    
    private static void stringDemo() {
        System.out.println("\n📝 String Utilities:");
        
        List<String> words = Arrays.asList("Hello", "World", "Java", "Programming");
        
        // Join with delimiter
        String sentence = StringUtils.join(words, " ");
        System.out.println("Joined: " + sentence);
        
        // Check if not empty
        boolean hasContent = StringUtils.isNotEmpty(sentence);
        System.out.println("Has content: " + hasContent);
        
        // Safe trim
        String trimmed = StringUtils.trim("  " + sentence + "  ");
        System.out.println("Trimmed: '" + trimmed + "'");
    }
}
```

## Step 3: Run Your Program

### Using Maven
```bash
mvn compile exec:java -Dexec.mainClass="QuickStart"
```

### Using Gradle
```bash
./gradlew run --args="QuickStart"
```

### Expected Output
```
🚀 Starting jclassesv2 Quick Demo

📊 Basic Data Processing:
Original: [apple, banana, cherry]
Processed: [APPLE 🍎, BANANA 🍎, CHERRY 🍎]

📦 Collection Utilities:
Even numbers: [2, 4, 6, 8, 10]
Squares: [4, 16, 36, 64, 100]
Is empty: false

📝 String Utilities:
Joined: Hello World Java Programming
Has content: true
Trimmed: 'Hello World Java Programming'

✅ Demo completed successfully!
```

## Next Steps

### 1. Explore Asynchronous Processing
```java
import java.util.concurrent.CompletableFuture;

public class AsyncExample {
    public static void main(String[] args) {
        try (DataProcessor processor = new DataProcessor()) {
            List<String> data = Arrays.asList("task1", "task2", "task3");
            
            CompletableFuture<List<String>> future = processor.processAsync(data);
            
            future.thenAccept(result -> {
                System.out.println("Async result: " + result);
            }).join();
        }
    }
}
```

### 2. Custom Configuration
```java
import com.jclassesv2.config.Configuration;
import java.util.concurrent.TimeUnit;

public class ConfigExample {
    public static void main(String[] args) {
        Configuration config = Configuration.builder()
            .setMaxThreads(8)
            .setTimeout(30, TimeUnit.SECONDS)
            .enableLogging(true)
            .build();
            
        try (DataProcessor processor = new DataProcessor(config)) {
            // Your processing logic here
        }
    }
}
```

### 3. Error Handling
```java
import com.jclassesv2.exception.ProcessingException;

public class ErrorHandlingExample {
    public static void main(String[] args) {
        try (DataProcessor processor = new DataProcessor()) {
            List<String> data = Arrays.asList("valid", null, "data");
            List<String> result = processor.process(data);
            
        } catch (ProcessingException e) {
            System.err.println("Processing failed: " + e.getMessage());
            // Handle the error appropriately
            
        } catch (Exception e) {
            System.err.println("Unexpected error: " + e.getMessage());
        }
    }
}
```

## Common Use Cases

### Data Transformation Pipeline
```java
public class DataPipeline {
    public static void main(String[] args) {
        List<String> rawData = Arrays.asList("  apple  ", "BANANA", "Cherry");
        
        // Create a processing pipeline
        List<String> cleaned = CollectionUtils.transform(rawData, item -> {
            String trimmed = StringUtils.trim(item);
            return trimmed.toLowerCase();
        });
        
        // Filter out items that are too short
        List<String> filtered = CollectionUtils.filter(cleaned, 
            item -> item.length() > 4);
        
        System.out.println("Cleaned and filtered: " + filtered);
        // Output: [apple, banana, cherry]
    }
}
```

### Batch Processing
```java
public class BatchProcessor {
    public static void main(String[] args) {
        // Simulate large dataset
        List<String> largeDataset = generateLargeDataset(10000);
        
        try (DataProcessor processor = new DataProcessor()) {
            // Process in chunks to manage memory
            int chunkSize = 1000;
            
            for (int i = 0; i < largeDataset.size(); i += chunkSize) {
                int end = Math.min(i + chunkSize, largeDataset.size());
                List<String> chunk = largeDataset.subList(i, end);
                
                List<String> processed = processor.process(chunk);
                System.out.println("Processed chunk " + (i/chunkSize + 1) + 
                                 ": " + processed.size() + " items");
            }
        } catch (ProcessingException e) {
            System.err.println("Batch processing failed: " + e.getMessage());
        }
    }
    
    private static List<String> generateLargeDataset(int size) {
        return IntStream.range(0, size)
            .mapToObj(i -> "item_" + i)
            .collect(Collectors.toList());
    }
}
```

## Troubleshooting

### Issue: Dependencies not found
**Solution:** Make sure you've added the correct dependency to your build file and refreshed your project.

### Issue: ClassNotFoundException
**Solution:** Ensure you're using Java 11 or later and that all dependencies are properly resolved.

### Issue: Processing takes too long
**Solution:** Use asynchronous processing or adjust the configuration:
```java
Configuration config = Configuration.builder()
    .setMaxThreads(Runtime.getRuntime().availableProcessors())
    .setTimeout(60, TimeUnit.SECONDS)
    .build();
```

## What's Next?

1. **Read the full [API Documentation](API_DOCUMENTATION.md)** for comprehensive coverage
2. **Check out [Advanced Examples](ADVANCED_EXAMPLES.md)** for complex scenarios
3. **Review [Best Practices](BEST_PRACTICES.md)** for production usage
4. **Explore [Configuration Guide](CONFIGURATION.md)** for customization options

## Getting Help

- 📖 [Full Documentation](API_DOCUMENTATION.md)
- 🐛 [Report Issues](https://github.com/jclassesv2/jclassesv2/issues)
- 💬 [Community Discord](https://discord.gg/jclassesv2)
- 📧 [Email Support](mailto:support@jclassesv2.com)

---
**Happy Coding! 🎉**