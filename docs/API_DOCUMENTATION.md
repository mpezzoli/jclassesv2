# jclassesv2 API Documentation

## Table of Contents

1. [Overview](#overview)
2. [Getting Started](#getting-started)
3. [Core APIs](#core-apis)
4. [Utility Classes](#utility-classes)
5. [Data Models](#data-models)
6. [Exception Handling](#exception-handling)
7. [Configuration](#configuration)
8. [Examples](#examples)
9. [Best Practices](#best-practices)
10. [Troubleshooting](#troubleshooting)

## Overview

jclassesv2 is a comprehensive Java library providing essential utilities, data structures, and APIs for modern Java applications. This documentation covers all public APIs, methods, and components available in the library.

### Key Features

- **Type-safe collections and utilities**
- **Robust error handling mechanisms**
- **Configurable components**
- **Thread-safe operations**
- **Extensive logging and monitoring support**

### System Requirements

- Java 11 or higher
- Maven 3.6+ or Gradle 6.0+

## Getting Started

### Installation

#### Maven
```xml
<dependency>
    <groupId>com.jclassesv2</groupId>
    <artifactId>jclassesv2-core</artifactId>
    <version>2.0.0</version>
</dependency>
```

#### Gradle
```gradle
implementation 'com.jclassesv2:jclassesv2-core:2.0.0'
```

### Quick Start Example

```java
import com.jclassesv2.core.DataProcessor;
import com.jclassesv2.util.CollectionUtils;

public class QuickStart {
    public static void main(String[] args) {
        DataProcessor processor = new DataProcessor();
        List<String> data = Arrays.asList("item1", "item2", "item3");
        
        // Process data with built-in utilities
        List<String> processed = processor.process(data);
        
        // Use utility functions
        boolean isEmpty = CollectionUtils.isEmpty(processed);
        System.out.println("Processed " + processed.size() + " items");
    }
}
```

## Core APIs

### DataProcessor

The `DataProcessor` class provides core data processing functionality.

#### Class Declaration
```java
public class DataProcessor implements Processor<T>, AutoCloseable
```

#### Constructor

##### `DataProcessor()`
Creates a new DataProcessor with default configuration.

**Example:**
```java
DataProcessor processor = new DataProcessor();
```

##### `DataProcessor(ProcessorConfig config)`
Creates a new DataProcessor with custom configuration.

**Parameters:**
- `config` - Configuration object specifying processing behavior

**Example:**
```java
ProcessorConfig config = ProcessorConfig.builder()
    .maxThreads(4)
    .timeout(30, TimeUnit.SECONDS)
    .build();
DataProcessor processor = new DataProcessor(config);
```

#### Methods

##### `process(List<T> data)`
Processes a list of data items using the configured processing pipeline.

**Parameters:**
- `data` - List of items to process (cannot be null)

**Returns:**
- `List<T>` - Processed list of items

**Throws:**
- `ProcessingException` - If processing fails
- `IllegalArgumentException` - If data is null

**Example:**
```java
List<String> input = Arrays.asList("apple", "banana", "cherry");
List<String> output = processor.process(input);
```

##### `processAsync(List<T> data)`
Asynchronously processes a list of data items.

**Parameters:**
- `data` - List of items to process

**Returns:**
- `CompletableFuture<List<T>>` - Future containing processed results

**Example:**
```java
CompletableFuture<List<String>> future = processor.processAsync(input);
future.thenAccept(result -> {
    System.out.println("Processing completed: " + result.size() + " items");
});
```

##### `setProcessor(Function<T, T> processingFunction)`
Sets a custom processing function.

**Parameters:**
- `processingFunction` - Function to apply to each data item

**Example:**
```java
processor.setProcessor(item -> item.toString().toUpperCase());
```

##### `getStatistics()`
Returns processing statistics.

**Returns:**
- `ProcessingStats` - Object containing processing metrics

**Example:**
```java
ProcessingStats stats = processor.getStatistics();
System.out.println("Items processed: " + stats.getItemsProcessed());
System.out.println("Average processing time: " + stats.getAverageProcessingTime());
```

### ConfigurationManager

Manages application configuration and settings.

#### Class Declaration
```java
public final class ConfigurationManager
```

#### Static Methods

##### `getInstance()`
Returns the singleton instance of ConfigurationManager.

**Returns:**
- `ConfigurationManager` - Singleton instance

**Example:**
```java
ConfigurationManager config = ConfigurationManager.getInstance();
```

##### `loadConfiguration(String configPath)`
Loads configuration from a file.

**Parameters:**
- `configPath` - Path to configuration file

**Throws:**
- `ConfigurationException` - If configuration cannot be loaded
- `FileNotFoundException` - If configuration file doesn't exist

**Example:**
```java
ConfigurationManager.loadConfiguration("config/application.properties");
```

#### Instance Methods

##### `getProperty(String key)`
Gets a configuration property value.

**Parameters:**
- `key` - Property key

**Returns:**
- `String` - Property value or null if not found

**Example:**
```java
String dbUrl = config.getProperty("database.url");
```

##### `getProperty(String key, String defaultValue)`
Gets a configuration property value with a default.

**Parameters:**
- `key` - Property key
- `defaultValue` - Default value if property not found

**Returns:**
- `String` - Property value or default value

**Example:**
```java
String maxConnections = config.getProperty("database.max.connections", "10");
```

##### `setProperty(String key, String value)`
Sets a configuration property value.

**Parameters:**
- `key` - Property key
- `value` - Property value

**Example:**
```java
config.setProperty("cache.enabled", "true");
```

## Utility Classes

### CollectionUtils

Utility methods for working with collections.

#### Static Methods

##### `isEmpty(Collection<?> collection)`
Checks if a collection is null or empty.

**Parameters:**
- `collection` - Collection to check

**Returns:**
- `boolean` - true if collection is null or empty

**Example:**
```java
List<String> list = new ArrayList<>();
boolean empty = CollectionUtils.isEmpty(list); // returns true
```

##### `isNotEmpty(Collection<?> collection)`
Checks if a collection is not null and not empty.

**Parameters:**
- `collection` - Collection to check

**Returns:**
- `boolean` - true if collection is not null and not empty

**Example:**
```java
List<String> list = Arrays.asList("item1", "item2");
boolean notEmpty = CollectionUtils.isNotEmpty(list); // returns true
```

##### `nullSafeSize(Collection<?> collection)`
Returns the size of a collection, handling null collections safely.

**Parameters:**
- `collection` - Collection to get size of

**Returns:**
- `int` - Size of collection, or 0 if null

**Example:**
```java
List<String> list = null;
int size = CollectionUtils.nullSafeSize(list); // returns 0
```

##### `filter(Collection<T> collection, Predicate<T> predicate)`
Filters a collection based on a predicate.

**Parameters:**
- `collection` - Collection to filter
- `predicate` - Predicate to apply

**Returns:**
- `List<T>` - Filtered list

**Example:**
```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
List<Integer> evens = CollectionUtils.filter(numbers, n -> n % 2 == 0);
// Result: [2, 4]
```

##### `transform(Collection<T> collection, Function<T, R> mapper)`
Transforms a collection using a mapping function.

**Parameters:**
- `collection` - Collection to transform
- `mapper` - Mapping function

**Returns:**
- `List<R>` - Transformed list

**Example:**
```java
List<String> words = Arrays.asList("hello", "world");
List<String> uppercase = CollectionUtils.transform(words, String::toUpperCase);
// Result: ["HELLO", "WORLD"]
```

### StringUtils

Utility methods for string operations.

#### Static Methods

##### `isEmpty(String str)`
Checks if a string is null or empty.

**Parameters:**
- `str` - String to check

**Returns:**
- `boolean` - true if string is null or empty

**Example:**
```java
boolean empty = StringUtils.isEmpty(""); // returns true
boolean empty2 = StringUtils.isEmpty(null); // returns true
```

##### `isNotEmpty(String str)`
Checks if a string is not null and not empty.

**Parameters:**
- `str` - String to check

**Returns:**
- `boolean` - true if string is not null and not empty

**Example:**
```java
boolean notEmpty = StringUtils.isNotEmpty("hello"); // returns true
```

##### `trim(String str)`
Trims a string, handling null strings safely.

**Parameters:**
- `str` - String to trim

**Returns:**
- `String` - Trimmed string, or null if input was null

**Example:**
```java
String trimmed = StringUtils.trim("  hello  "); // returns "hello"
String nullResult = StringUtils.trim(null); // returns null
```

##### `join(Collection<String> strings, String delimiter)`
Joins a collection of strings with a delimiter.

**Parameters:**
- `strings` - Collection of strings to join
- `delimiter` - Delimiter to use

**Returns:**
- `String` - Joined string

**Example:**
```java
List<String> words = Arrays.asList("apple", "banana", "cherry");
String result = StringUtils.join(words, ", ");
// Result: "apple, banana, cherry"
```

### DateUtils

Utility methods for date and time operations.

#### Static Methods

##### `now()`
Returns the current date and time.

**Returns:**
- `LocalDateTime` - Current date and time

**Example:**
```java
LocalDateTime current = DateUtils.now();
```

##### `formatDate(LocalDateTime dateTime, String pattern)`
Formats a date using the specified pattern.

**Parameters:**
- `dateTime` - Date to format
- `pattern` - Format pattern

**Returns:**
- `String` - Formatted date string

**Example:**
```java
LocalDateTime date = LocalDateTime.now();
String formatted = DateUtils.formatDate(date, "yyyy-MM-dd HH:mm:ss");
```

##### `parseDate(String dateString, String pattern)`
Parses a date string using the specified pattern.

**Parameters:**
- `dateString` - Date string to parse
- `pattern` - Pattern to use for parsing

**Returns:**
- `LocalDateTime` - Parsed date

**Throws:**
- `DateTimeParseException` - If date string cannot be parsed

**Example:**
```java
LocalDateTime date = DateUtils.parseDate("2023-12-25 10:30:00", "yyyy-MM-dd HH:mm:ss");
```

## Data Models

### ProcessingResult

Represents the result of a data processing operation.

#### Class Declaration
```java
public class ProcessingResult<T>
```

#### Constructor

##### `ProcessingResult(List<T> data, ProcessingStatus status)`
Creates a new ProcessingResult.

**Parameters:**
- `data` - Processed data
- `status` - Processing status

#### Methods

##### `getData()`
Gets the processed data.

**Returns:**
- `List<T>` - Processed data list

##### `getStatus()`
Gets the processing status.

**Returns:**
- `ProcessingStatus` - Current processing status

##### `isSuccess()`
Checks if processing was successful.

**Returns:**
- `boolean` - true if processing was successful

##### `getErrorMessage()`
Gets error message if processing failed.

**Returns:**
- `String` - Error message, or null if no error

**Example:**
```java
ProcessingResult<String> result = processor.processWithResult(data);
if (result.isSuccess()) {
    List<String> processedData = result.getData();
    System.out.println("Successfully processed " + processedData.size() + " items");
} else {
    System.err.println("Processing failed: " + result.getErrorMessage());
}
```

### Configuration

Represents application configuration.

#### Class Declaration
```java
public class Configuration
```

#### Builder Pattern

##### `Configuration.Builder`
Builder for creating Configuration instances.

**Example:**
```java
Configuration config = Configuration.builder()
    .setMaxThreads(8)
    .setTimeout(60, TimeUnit.SECONDS)
    .setRetryAttempts(3)
    .enableLogging(true)
    .build();
```

#### Methods

##### `getMaxThreads()`
Gets the maximum number of threads.

**Returns:**
- `int` - Maximum number of threads

##### `getTimeout()`
Gets the timeout duration.

**Returns:**
- `Duration` - Timeout duration

##### `getRetryAttempts()`
Gets the number of retry attempts.

**Returns:**
- `int` - Number of retry attempts

##### `isLoggingEnabled()`
Checks if logging is enabled.

**Returns:**
- `boolean` - true if logging is enabled

## Exception Handling

### ProcessingException

Exception thrown when data processing fails.

#### Class Declaration
```java
public class ProcessingException extends Exception
```

#### Constructors

##### `ProcessingException(String message)`
Creates a new ProcessingException with a message.

##### `ProcessingException(String message, Throwable cause)`
Creates a new ProcessingException with a message and cause.

**Example:**
```java
try {
    processor.process(data);
} catch (ProcessingException e) {
    logger.error("Processing failed: " + e.getMessage(), e);
    // Handle the exception appropriately
}
```

### ConfigurationException

Exception thrown when configuration operations fail.

#### Class Declaration
```java
public class ConfigurationException extends RuntimeException
```

#### Constructors

##### `ConfigurationException(String message)`
Creates a new ConfigurationException with a message.

##### `ConfigurationException(String message, Throwable cause)`
Creates a new ConfigurationException with a message and cause.

**Example:**
```java
try {
    ConfigurationManager.loadConfiguration("invalid-path.properties");
} catch (ConfigurationException e) {
    logger.error("Failed to load configuration: " + e.getMessage(), e);
    // Use default configuration or handle gracefully
}
```

## Configuration

### Configuration Properties

The library supports configuration through properties files or system properties.

#### Core Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `jclassesv2.max.threads` | int | 4 | Maximum number of processing threads |
| `jclassesv2.timeout.seconds` | int | 30 | Default timeout in seconds |
| `jclassesv2.retry.attempts` | int | 3 | Number of retry attempts |
| `jclassesv2.logging.enabled` | boolean | true | Enable/disable logging |
| `jclassesv2.cache.size` | int | 1000 | Maximum cache size |

#### Example Configuration File

```properties
# jclassesv2 Configuration
jclassesv2.max.threads=8
jclassesv2.timeout.seconds=60
jclassesv2.retry.attempts=5
jclassesv2.logging.enabled=true
jclassesv2.cache.size=2000
```

### Programmatic Configuration

```java
Configuration config = Configuration.builder()
    .setMaxThreads(8)
    .setTimeout(60, TimeUnit.SECONDS)
    .setRetryAttempts(5)
    .enableLogging(true)
    .setCacheSize(2000)
    .build();

DataProcessor processor = new DataProcessor(config);
```

## Examples

### Basic Data Processing

```java
import com.jclassesv2.core.DataProcessor;
import com.jclassesv2.model.ProcessingResult;

public class BasicProcessingExample {
    public static void main(String[] args) {
        // Create processor with default configuration
        DataProcessor processor = new DataProcessor();
        
        // Prepare data
        List<String> data = Arrays.asList(
            "apple", "banana", "cherry", "date", "elderberry"
        );
        
        try {
            // Process data
            List<String> result = processor.process(data);
            
            // Display results
            System.out.println("Original: " + data);
            System.out.println("Processed: " + result);
            
        } catch (ProcessingException e) {
            System.err.println("Processing failed: " + e.getMessage());
        } finally {
            // Clean up resources
            processor.close();
        }
    }
}
```

### Asynchronous Processing

```java
import com.jclassesv2.core.DataProcessor;
import java.util.concurrent.CompletableFuture;

public class AsyncProcessingExample {
    public static void main(String[] args) {
        DataProcessor processor = new DataProcessor();
        
        List<String> data = Arrays.asList("item1", "item2", "item3");
        
        // Process asynchronously
        CompletableFuture<List<String>> future = processor.processAsync(data);
        
        // Handle completion
        future.thenAccept(result -> {
            System.out.println("Async processing completed: " + result);
        }).exceptionally(throwable -> {
            System.err.println("Async processing failed: " + throwable.getMessage());
            return null;
        });
        
        // Wait for completion (in real applications, avoid blocking)
        future.join();
        
        processor.close();
    }
}
```

### Custom Configuration

```java
import com.jclassesv2.core.DataProcessor;
import com.jclassesv2.config.Configuration;
import java.util.concurrent.TimeUnit;

public class CustomConfigExample {
    public static void main(String[] args) {
        // Create custom configuration
        Configuration config = Configuration.builder()
            .setMaxThreads(8)
            .setTimeout(2, TimeUnit.MINUTES)
            .setRetryAttempts(5)
            .enableLogging(true)
            .build();
        
        // Create processor with custom configuration
        DataProcessor processor = new DataProcessor(config);
        
        // Set custom processing function
        processor.setProcessor(item -> {
            // Custom processing logic
            return item.toString().toUpperCase().trim();
        });
        
        List<String> data = Arrays.asList(" hello ", " world ", " java ");
        
        try {
            List<String> result = processor.process(data);
            System.out.println("Result: " + result);
            // Output: [HELLO, WORLD, JAVA]
        } catch (ProcessingException e) {
            System.err.println("Processing failed: " + e.getMessage());
        } finally {
            processor.close();
        }
    }
}
```

### Using Utility Classes

```java
import com.jclassesv2.util.CollectionUtils;
import com.jclassesv2.util.StringUtils;

public class UtilityExample {
    public static void main(String[] args) {
        // Collection utilities
        List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
        
        // Filter even numbers
        List<Integer> evens = CollectionUtils.filter(numbers, n -> n % 2 == 0);
        System.out.println("Even numbers: " + evens);
        
        // Transform to strings
        List<String> strings = CollectionUtils.transform(evens, Object::toString);
        System.out.println("As strings: " + strings);
        
        // String utilities
        String joined = StringUtils.join(strings, " | ");
        System.out.println("Joined: " + joined);
        
        // Check if collections are empty
        boolean isEmpty = CollectionUtils.isEmpty(strings);
        System.out.println("Is empty: " + isEmpty);
    }
}
```

### Error Handling

```java
import com.jclassesv2.core.DataProcessor;
import com.jclassesv2.exception.ProcessingException;
import com.jclassesv2.exception.ConfigurationException;

public class ErrorHandlingExample {
    public static void main(String[] args) {
        try {
            // Load configuration
            ConfigurationManager.loadConfiguration("config.properties");
            
            DataProcessor processor = new DataProcessor();
            
            // Process data with error handling
            List<String> data = Arrays.asList("valid", null, "data");
            
            List<String> result = processor.process(data);
            System.out.println("Processing successful: " + result);
            
        } catch (ConfigurationException e) {
            System.err.println("Configuration error: " + e.getMessage());
            // Fall back to default configuration
            
        } catch (ProcessingException e) {
            System.err.println("Processing error: " + e.getMessage());
            // Handle processing failure
            
        } catch (Exception e) {
            System.err.println("Unexpected error: " + e.getMessage());
            // Handle unexpected errors
        }
    }
}
```

## Best Practices

### 1. Resource Management

Always close resources properly using try-with-resources or explicit close calls:

```java
// Preferred: try-with-resources
try (DataProcessor processor = new DataProcessor()) {
    List<String> result = processor.process(data);
    // processor is automatically closed
}

// Alternative: explicit close
DataProcessor processor = new DataProcessor();
try {
    List<String> result = processor.process(data);
} finally {
    processor.close();
}
```

### 2. Configuration Management

Use configuration files for environment-specific settings:

```java
// Load configuration based on environment
String env = System.getProperty("environment", "development");
String configFile = "config-" + env + ".properties";
ConfigurationManager.loadConfiguration(configFile);
```

### 3. Error Handling

Implement comprehensive error handling:

```java
try {
    processor.process(data);
} catch (ProcessingException e) {
    // Log the error
    logger.error("Processing failed for data: " + data, e);
    
    // Decide on recovery strategy
    if (e.isRetriable()) {
        // Retry with exponential backoff
        retryProcessor.process(data);
    } else {
        // Handle non-retriable errors
        handleFailure(data, e);
    }
}
```

### 4. Performance Optimization

Use asynchronous processing for better performance:

```java
// Process multiple batches concurrently
List<CompletableFuture<List<String>>> futures = dataBatches.stream()
    .map(processor::processAsync)
    .collect(Collectors.toList());

// Wait for all to complete
CompletableFuture<Void> allOf = CompletableFuture.allOf(
    futures.toArray(new CompletableFuture[0])
);

List<List<String>> results = allOf.thenApply(v ->
    futures.stream()
        .map(CompletableFuture::join)
        .collect(Collectors.toList())
).get();
```

### 5. Thread Safety

Ensure thread safety when sharing processor instances:

```java
// DataProcessor is thread-safe for processing operations
DataProcessor sharedProcessor = new DataProcessor();

// Multiple threads can safely call process methods
ExecutorService executor = Executors.newFixedThreadPool(4);
for (List<String> batch : dataBatches) {
    executor.submit(() -> {
        try {
            sharedProcessor.process(batch);
        } catch (ProcessingException e) {
            logger.error("Batch processing failed", e);
        }
    });
}
```

### 6. Memory Management

Be mindful of memory usage with large datasets:

```java
// Process data in chunks to avoid memory issues
int chunkSize = 1000;
for (int i = 0; i < largeDataset.size(); i += chunkSize) {
    int end = Math.min(i + chunkSize, largeDataset.size());
    List<String> chunk = largeDataset.subList(i, end);
    
    List<String> result = processor.process(chunk);
    // Process or store result immediately
    handleResult(result);
}
```

## Troubleshooting

### Common Issues

#### 1. OutOfMemoryError

**Problem:** Application runs out of memory during processing.

**Solutions:**
- Process data in smaller chunks
- Increase JVM heap size: `-Xmx2g`
- Use streaming processing for large datasets
- Check for memory leaks

```java
// Process in chunks
int chunkSize = 100;
for (int i = 0; i < data.size(); i += chunkSize) {
    List<String> chunk = data.subList(i, Math.min(i + chunkSize, data.size()));
    processor.process(chunk);
}
```

#### 2. ProcessingException: Timeout

**Problem:** Processing operations time out.

**Solutions:**
- Increase timeout configuration
- Optimize processing logic
- Use asynchronous processing
- Check system resources

```java
Configuration config = Configuration.builder()
    .setTimeout(5, TimeUnit.MINUTES) // Increase timeout
    .build();
```

#### 3. ConfigurationException: File Not Found

**Problem:** Configuration file cannot be loaded.

**Solutions:**
- Check file path and permissions
- Use classpath resources
- Provide default configuration
- Validate file exists before loading

```java
// Check if file exists
File configFile = new File("config.properties");
if (configFile.exists()) {
    ConfigurationManager.loadConfiguration("config.properties");
} else {
    // Use default configuration
    logger.warn("Config file not found, using defaults");
}
```

#### 4. Poor Performance

**Problem:** Processing is slower than expected.

**Solutions:**
- Use parallel processing
- Optimize algorithms
- Increase thread pool size
- Profile the application

```java
// Use parallel streams
List<String> result = data.parallelStream()
    .map(item -> processItem(item))
    .collect(Collectors.toList());
```

### Debugging Tips

#### Enable Debug Logging

```java
// Set system property for debug logging
System.setProperty("jclassesv2.logging.level", "DEBUG");

// Or in configuration file
jclassesv2.logging.level=DEBUG
```

#### Monitor Processing Statistics

```java
DataProcessor processor = new DataProcessor();
// ... processing operations ...

ProcessingStats stats = processor.getStatistics();
System.out.println("Items processed: " + stats.getItemsProcessed());
System.out.println("Success rate: " + stats.getSuccessRate());
System.out.println("Average time: " + stats.getAverageProcessingTime());
```

#### Use JVM Monitoring Tools

```bash
# Monitor memory usage
jstat -gc <pid>

# Generate heap dump for analysis
jmap -dump:format=b,file=heap.hprof <pid>

# Monitor thread activity
jstack <pid>
```

### Getting Help

If you encounter issues not covered in this documentation:

1. Check the [GitHub Issues](https://github.com/jclassesv2/jclassesv2/issues)
2. Review the [FAQ](https://github.com/jclassesv2/jclassesv2/wiki/FAQ)
3. Join our [Discord Community](https://discord.gg/jclassesv2)
4. Contact support: support@jclassesv2.com

---

**Last Updated:** December 2023
**Version:** 2.0.0
**Authors:** jclassesv2 Development Team