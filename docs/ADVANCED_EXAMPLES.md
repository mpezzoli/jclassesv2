# Advanced Examples

This document provides comprehensive examples for advanced usage patterns of the jclassesv2 library.

## Table of Contents

1. [Complex Data Processing Pipelines](#complex-data-processing-pipelines)
2. [Parallel Processing](#parallel-processing)
3. [Custom Exception Handling](#custom-exception-handling)
4. [Configuration Management](#configuration-management)
5. [Performance Optimization](#performance-optimization)
6. [Real-World Use Cases](#real-world-use-cases)
7. [Integration Patterns](#integration-patterns)

## Complex Data Processing Pipelines

### Multi-Stage Processing Pipeline

```java
import com.jclassesv2.core.DataProcessor;
import com.jclassesv2.util.CollectionUtils;
import com.jclassesv2.util.StringUtils;
import com.jclassesv2.model.ProcessingResult;
import java.util.List;
import java.util.stream.Collectors;

public class MultiStagePipeline {
    
    public static class DataItem {
        private String value;
        private int priority;
        private boolean processed;
        
        public DataItem(String value, int priority) {
            this.value = value;
            this.priority = priority;
            this.processed = false;
        }
        
        // Getters and setters
        public String getValue() { return value; }
        public void setValue(String value) { this.value = value; }
        public int getPriority() { return priority; }
        public void setPriority(int priority) { this.priority = priority; }
        public boolean isProcessed() { return processed; }
        public void setProcessed(boolean processed) { this.processed = processed; }
        
        @Override
        public String toString() {
            return String.format("DataItem{value='%s', priority=%d, processed=%s}", 
                value, priority, processed);
        }
    }
    
    public static void main(String[] args) {
        // Stage 1: Data Validation and Cleaning
        List<String> rawData = Arrays.asList(
            "  APPLE  ", "banana", "", "CHERRY", null, "date", "elderberry"
        );
        
        List<String> cleanedData = validateAndClean(rawData);
        System.out.println("Stage 1 - Cleaned data: " + cleanedData);
        
        // Stage 2: Transform to DataItems
        List<DataItem> dataItems = transformToDataItems(cleanedData);
        System.out.println("Stage 2 - Data items: " + dataItems);
        
        // Stage 3: Priority-based processing
        List<DataItem> processedItems = processByPriority(dataItems);
        System.out.println("Stage 3 - Processed items: " + processedItems);
        
        // Stage 4: Final aggregation
        ProcessingResult<DataItem> finalResult = aggregateResults(processedItems);
        System.out.println("Stage 4 - Final result: " + finalResult);
    }
    
    private static List<String> validateAndClean(List<String> rawData) {
        return rawData.stream()
            .filter(item -> !StringUtils.isEmpty(item))  // Remove null/empty
            .map(StringUtils::trim)                       // Trim whitespace
            .map(String::toLowerCase)                     // Normalize case
            .filter(item -> item.length() >= 3)          // Min length validation
            .collect(Collectors.toList());
    }
    
    private static List<DataItem> transformToDataItems(List<String> data) {
        return CollectionUtils.transform(data, item -> {
            int priority = item.length(); // Priority based on length
            return new DataItem(item, priority);
        });
    }
    
    private static List<DataItem> processByPriority(List<DataItem> items) {
        // Sort by priority (descending)
        List<DataItem> sortedItems = items.stream()
            .sorted((a, b) -> Integer.compare(b.getPriority(), a.getPriority()))
            .collect(Collectors.toList());
        
        try (DataProcessor processor = new DataProcessor()) {
            // Process high priority items first
            processor.setProcessor(item -> {
                DataItem dataItem = (DataItem) item;
                dataItem.setValue(dataItem.getValue().toUpperCase());
                dataItem.setProcessed(true);
                return dataItem;
            });
            
            return processor.process(sortedItems);
            
        } catch (Exception e) {
            System.err.println("Priority processing failed: " + e.getMessage());
            return items;
        }
    }
    
    private static ProcessingResult<DataItem> aggregateResults(List<DataItem> items) {
        // Calculate statistics
        long processedCount = items.stream()
            .mapToLong(item -> item.isProcessed() ? 1 : 0)
            .sum();
        
        double averagePriority = items.stream()
            .mapToInt(DataItem::getPriority)
            .average()
            .orElse(0.0);
        
        System.out.println("Statistics:");
        System.out.println("  Processed items: " + processedCount + "/" + items.size());
        System.out.println("  Average priority: " + String.format("%.2f", averagePriority));
        
        ProcessingStatus status = processedCount == items.size() ? 
            ProcessingStatus.SUCCESS : ProcessingStatus.PARTIAL_SUCCESS;
            
        return new ProcessingResult<>(items, status);
    }
}
```

### Conditional Processing Pipeline

```java
import com.jclassesv2.core.DataProcessor;
import com.jclassesv2.config.Configuration;
import java.util.function.Predicate;
import java.util.concurrent.TimeUnit;

public class ConditionalProcessing {
    
    public static class ProcessingContext {
        private final String environment;
        private final boolean debugMode;
        private final int maxRetries;
        
        public ProcessingContext(String environment, boolean debugMode, int maxRetries) {
            this.environment = environment;
            this.debugMode = debugMode;
            this.maxRetries = maxRetries;
        }
        
        // Getters
        public String getEnvironment() { return environment; }
        public boolean isDebugMode() { return debugMode; }
        public int getMaxRetries() { return maxRetries; }
    }
    
    public static void main(String[] args) {
        ProcessingContext context = new ProcessingContext("production", false, 3);
        
        List<String> data = Arrays.asList(
            "critical_data", "normal_data", "low_priority_data"
        );
        
        processWithConditions(data, context);
    }
    
    private static void processWithConditions(List<String> data, ProcessingContext context) {
        // Configure based on environment
        Configuration config = Configuration.builder()
            .setMaxThreads(context.getEnvironment().equals("production") ? 8 : 2)
            .setTimeout(context.getEnvironment().equals("production") ? 60 : 30, TimeUnit.SECONDS)
            .setRetryAttempts(context.getMaxRetries())
            .enableLogging(context.isDebugMode())
            .build();
        
        try (DataProcessor processor = new DataProcessor(config)) {
            
            // Define processing rules based on data characteristics
            processor.setProcessor(item -> {
                String dataItem = (String) item;
                
                if (dataItem.startsWith("critical_")) {
                    return processCriticalData(dataItem, context);
                } else if (dataItem.startsWith("normal_")) {
                    return processNormalData(dataItem, context);
                } else {
                    return processLowPriorityData(dataItem, context);
                }
            });
            
            // Apply conditional filtering
            List<String> filteredData = CollectionUtils.filter(data, 
                createDataFilter(context));
            
            if (context.isDebugMode()) {
                System.out.println("Processing " + filteredData.size() + " items");
                System.out.println("Configuration: " + config);
            }
            
            List<String> results = processor.process(filteredData);
            
            // Post-processing based on environment
            if (context.getEnvironment().equals("production")) {
                validateResults(results);
            }
            
            System.out.println("Results: " + results);
            
        } catch (Exception e) {
            handleProcessingError(e, context);
        }
    }
    
    private static String processCriticalData(String data, ProcessingContext context) {
        // High priority processing with validation
        String processed = data.toUpperCase() + "_CRITICAL_PROCESSED";
        
        if (context.isDebugMode()) {
            System.out.println("Critical processing: " + data + " -> " + processed);
        }
        
        return processed;
    }
    
    private static String processNormalData(String data, ProcessingContext context) {
        // Standard processing
        return data + "_processed";
    }
    
    private static String processLowPriorityData(String data, ProcessingContext context) {
        // Minimal processing for low priority items
        return context.getEnvironment().equals("production") ? 
            data + "_prod" : data + "_dev";
    }
    
    private static Predicate<String> createDataFilter(ProcessingContext context) {
        return data -> {
            // In production, skip low priority data if under load
            if (context.getEnvironment().equals("production") && 
                data.startsWith("low_priority_")) {
                return Runtime.getRuntime().freeMemory() > 100_000_000; // 100MB threshold
            }
            return true;
        };
    }
    
    private static void validateResults(List<String> results) {
        // Production validation
        boolean allValid = results.stream()
            .allMatch(result -> result != null && !result.trim().isEmpty());
        
        if (!allValid) {
            throw new RuntimeException("Production validation failed");
        }
        
        System.out.println("Production validation passed");
    }
    
    private static void handleProcessingError(Exception e, ProcessingContext context) {
        if (context.getEnvironment().equals("production")) {
            // Log to monitoring system
            System.err.println("PRODUCTION ERROR: " + e.getMessage());
            // Send alert, etc.
        } else {
            // Development error handling
            System.err.println("Development error: " + e.getMessage());
            e.printStackTrace();
        }
    }
}
```

## Parallel Processing

### Concurrent Data Processing

```java
import com.jclassesv2.core.DataProcessor;
import java.util.concurrent.*;
import java.util.stream.IntStream;

public class ConcurrentProcessing {
    
    public static void main(String[] args) {
        // Large dataset simulation
        List<String> largeDataset = generateLargeDataset(10000);
        
        // Example 1: Parallel batch processing
        parallelBatchProcessing(largeDataset);
        
        // Example 2: Fork-Join processing
        forkJoinProcessing(largeDataset);
        
        // Example 3: Producer-Consumer pattern
        producerConsumerProcessing(largeDataset);
    }
    
    private static List<String> generateLargeDataset(int size) {
        return IntStream.range(0, size)
            .mapToObj(i -> "data_item_" + i)
            .collect(Collectors.toList());
    }
    
    private static void parallelBatchProcessing(List<String> dataset) {
        System.out.println("\n=== Parallel Batch Processing ===");
        
        int batchSize = 1000;
        int numThreads = Runtime.getRuntime().availableProcessors();
        ExecutorService executor = Executors.newFixedThreadPool(numThreads);
        
        List<Future<List<String>>> futures = new ArrayList<>();
        
        try (DataProcessor processor = new DataProcessor()) {
            processor.setProcessor(item -> item.toString().toUpperCase());
            
            // Split into batches and submit for parallel processing
            for (int i = 0; i < dataset.size(); i += batchSize) {
                final int start = i;
                final int end = Math.min(i + batchSize, dataset.size());
                
                Future<List<String>> future = executor.submit(() -> {
                    List<String> batch = dataset.subList(start, end);
                    try {
                        List<String> result = processor.process(batch);
                        System.out.println("Batch " + (start / batchSize + 1) + 
                                         " processed by " + Thread.currentThread().getName());
                        return result;
                    } catch (Exception e) {
                        System.err.println("Batch processing failed: " + e.getMessage());
                        return Collections.emptyList();
                    }
                });
                
                futures.add(future);
            }
            
            // Collect results
            List<String> allResults = new ArrayList<>();
            for (Future<List<String>> future : futures) {
                try {
                    allResults.addAll(future.get(30, TimeUnit.SECONDS));
                } catch (Exception e) {
                    System.err.println("Failed to get batch result: " + e.getMessage());
                }
            }
            
            System.out.println("Parallel processing completed. Total results: " + 
                             allResults.size());
            
        } catch (Exception e) {
            System.err.println("Parallel batch processing failed: " + e.getMessage());
        } finally {
            executor.shutdown();
        }
    }
    
    private static void forkJoinProcessing(List<String> dataset) {
        System.out.println("\n=== Fork-Join Processing ===");
        
        ForkJoinPool forkJoinPool = new ForkJoinPool();
        
        try {
            DataProcessingTask task = new DataProcessingTask(dataset, 0, dataset.size());
            List<String> results = forkJoinPool.invoke(task);
            
            System.out.println("Fork-Join processing completed. Results: " + 
                             results.size());
            
        } finally {
            forkJoinPool.shutdown();
        }
    }
    
    private static class DataProcessingTask extends RecursiveTask<List<String>> {
        private static final int THRESHOLD = 500;
        private final List<String> data;
        private final int start;
        private final int end;
        
        public DataProcessingTask(List<String> data, int start, int end) {
            this.data = data;
            this.start = start;
            this.end = end;
        }
        
        @Override
        protected List<String> compute() {
            if (end - start <= THRESHOLD) {
                // Process directly
                return processSequentially(data.subList(start, end));
            } else {
                // Split into subtasks
                int middle = (start + end) / 2;
                
                DataProcessingTask leftTask = new DataProcessingTask(data, start, middle);
                DataProcessingTask rightTask = new DataProcessingTask(data, middle, end);
                
                leftTask.fork();
                List<String> rightResult = rightTask.compute();
                List<String> leftResult = leftTask.join();
                
                // Combine results
                List<String> combined = new ArrayList<>(leftResult);
                combined.addAll(rightResult);
                return combined;
            }
        }
        
        private List<String> processSequentially(List<String> batch) {
            try (DataProcessor processor = new DataProcessor()) {
                processor.setProcessor(item -> item.toString() + "_fork_join");
                return processor.process(batch);
            } catch (Exception e) {
                System.err.println("Sequential processing failed: " + e.getMessage());
                return Collections.emptyList();
            }
        }
    }
    
    private static void producerConsumerProcessing(List<String> dataset) {
        System.out.println("\n=== Producer-Consumer Processing ===");
        
        BlockingQueue<String> inputQueue = new LinkedBlockingQueue<>(1000);
        BlockingQueue<String> outputQueue = new LinkedBlockingQueue<>(1000);
        
        // Producer thread
        Thread producer = new Thread(() -> {
            try {
                for (String item : dataset) {
                    inputQueue.put(item);
                }
                // Signal end of data
                inputQueue.put("END_OF_DATA");
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                System.err.println("Producer interrupted");
            }
        });
        
        // Consumer threads
        int numConsumers = 4;
        List<Thread> consumers = new ArrayList<>();
        
        for (int i = 0; i < numConsumers; i++) {
            final int consumerId = i;
            Thread consumer = new Thread(() -> {
                try (DataProcessor processor = new DataProcessor()) {
                    processor.setProcessor(item -> "consumer_" + consumerId + "_" + item);
                    
                    while (true) {
                        String item = inputQueue.take();
                        if ("END_OF_DATA".equals(item)) {
                            inputQueue.put(item); // Put it back for other consumers
                            break;
                        }
                        
                        try {
                            List<String> result = processor.process(Arrays.asList(item));
                            for (String processed : result) {
                                outputQueue.put(processed);
                            }
                        } catch (Exception e) {
                            System.err.println("Consumer " + consumerId + 
                                             " processing failed: " + e.getMessage());
                        }
                    }
                    
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                    System.err.println("Consumer " + consumerId + " interrupted");
                } catch (Exception e) {
                    System.err.println("Consumer " + consumerId + " failed: " + e.getMessage());
                }
            });
            
            consumers.add(consumer);
        }
        
        // Start all threads
        producer.start();
        consumers.forEach(Thread::start);
        
        // Collect results
        List<String> results = new ArrayList<>();
        int expectedResults = dataset.size();
        
        try {
            while (results.size() < expectedResults) {
                String result = outputQueue.poll(5, TimeUnit.SECONDS);
                if (result != null) {
                    results.add(result);
                } else {
                    System.err.println("Timeout waiting for results");
                    break;
                }
            }
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            System.err.println("Result collection interrupted");
        }
        
        // Wait for completion
        try {
            producer.join();
            for (Thread consumer : consumers) {
                consumer.join();
            }
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            System.err.println("Thread join interrupted");
        }
        
        System.out.println("Producer-Consumer processing completed. Results: " + 
                         results.size());
    }
}
```

## Custom Exception Handling

### Comprehensive Error Recovery

```java
import com.jclassesv2.core.DataProcessor;
import com.jclassesv2.exception.ProcessingException;
import com.jclassesv2.exception.ConfigurationException;

public class AdvancedErrorHandling {
    
    private static final Logger logger = LoggerFactory.getLogger(AdvancedErrorHandling.class);
    
    public static class RetryableProcessor {
        private final DataProcessor processor;
        private final int maxRetries;
        private final long retryDelayMs;
        
        public RetryableProcessor(DataProcessor processor, int maxRetries, long retryDelayMs) {
            this.processor = processor;
            this.maxRetries = maxRetries;
            this.retryDelayMs = retryDelayMs;
        }
        
        public List<String> processWithRetry(List<String> data) throws ProcessingException {
            Exception lastException = null;
            
            for (int attempt = 1; attempt <= maxRetries; attempt++) {
                try {
                    return processor.process(data);
                    
                } catch (ProcessingException e) {
                    lastException = e;
                    
                    if (isRetriable(e) && attempt < maxRetries) {
                        logger.warn("Processing attempt {} failed, retrying in {}ms", 
                                  attempt, retryDelayMs, e);
                        
                        try {
                            Thread.sleep(retryDelayMs * attempt); // Exponential backoff
                        } catch (InterruptedException ie) {
                            Thread.currentThread().interrupt();
                            throw new ProcessingException("Retry interrupted", ie);
                        }
                    } else {
                        logger.error("Processing failed after {} attempts", attempt, e);
                        break;
                    }
                }
            }
            
            throw new ProcessingException("Processing failed after " + maxRetries + 
                                        " attempts", lastException);
        }
        
        private boolean isRetriable(ProcessingException e) {
            // Define retry logic based on exception characteristics
            String message = e.getMessage().toLowerCase();
            return message.contains("timeout") || 
                   message.contains("connection") ||
                   message.contains("temporary");
        }
    }
    
    public static class FallbackProcessor {
        private final DataProcessor primaryProcessor;
        private final DataProcessor fallbackProcessor;
        
        public FallbackProcessor(DataProcessor primary, DataProcessor fallback) {
            this.primaryProcessor = primary;
            this.fallbackProcessor = fallback;
        }
        
        public List<String> processWithFallback(List<String> data) {
            try {
                return primaryProcessor.process(data);
                
            } catch (ProcessingException e) {
                logger.warn("Primary processing failed, using fallback", e);
                
                try {
                    return fallbackProcessor.process(data);
                    
                } catch (ProcessingException fallbackException) {
                    logger.error("Fallback processing also failed", fallbackException);
                    
                    // Return partial results or default processing
                    return data.stream()
                        .map(item -> "fallback_" + item)
                        .collect(Collectors.toList());
                }
            }
        }
    }
    
    public static void main(String[] args) {
        demonstrateRetryLogic();
        demonstrateFallbackLogic();
        demonstrateCircuitBreaker();
    }
    
    private static void demonstrateRetryLogic() {
        System.out.println("\n=== Retry Logic Demonstration ===");
        
        try (DataProcessor processor = new DataProcessor()) {
            // Simulate unreliable processor
            processor.setProcessor(item -> {
                if (Math.random() < 0.7) { // 70% failure rate
                    throw new RuntimeException("Simulated timeout error");
                }
                return item.toString().toUpperCase();
            });
            
            RetryableProcessor retryProcessor = new RetryableProcessor(processor, 3, 1000);
            
            List<String> data = Arrays.asList("item1", "item2", "item3");
            
            try {
                List<String> results = retryProcessor.processWithRetry(data);
                System.out.println("Retry processing succeeded: " + results);
            } catch (ProcessingException e) {
                System.err.println("Retry processing failed: " + e.getMessage());
            }
        }
    }
    
    private static void demonstrateFallbackLogic() {
        System.out.println("\n=== Fallback Logic Demonstration ===");
        
        try (DataProcessor primary = new DataProcessor();
             DataProcessor fallback = new DataProcessor()) {
            
            // Primary processor that always fails
            primary.setProcessor(item -> {
                throw new RuntimeException("Primary processor failure");
            });
            
            // Fallback processor
            fallback.setProcessor(item -> "fallback_" + item);
            
            FallbackProcessor fallbackProcessor = new FallbackProcessor(primary, fallback);
            
            List<String> data = Arrays.asList("item1", "item2", "item3");
            List<String> results = fallbackProcessor.processWithFallback(data);
            
            System.out.println("Fallback processing results: " + results);
        }
    }
    
    private static void demonstrateCircuitBreaker() {
        System.out.println("\n=== Circuit Breaker Demonstration ===");
        
        CircuitBreakerProcessor cbProcessor = new CircuitBreakerProcessor();
        
        List<String> data = Arrays.asList("item1", "item2", "item3");
        
        // Simulate multiple requests
        for (int i = 1; i <= 10; i++) {
            try {
                List<String> results = cbProcessor.process(data);
                System.out.println("Request " + i + " succeeded: " + results);
            } catch (Exception e) {
                System.err.println("Request " + i + " failed: " + e.getMessage());
            }
            
            try {
                Thread.sleep(500); // Small delay between requests
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                break;
            }
        }
    }
    
    private static class CircuitBreakerProcessor {
        private enum State { CLOSED, OPEN, HALF_OPEN }
        
        private State state = State.CLOSED;
        private int failureCount = 0;
        private long lastFailureTime = 0;
        private final int failureThreshold = 3;
        private final long timeoutMs = 5000; // 5 seconds
        
        public List<String> process(List<String> data) throws ProcessingException {
            if (state == State.OPEN) {
                if (System.currentTimeMillis() - lastFailureTime >= timeoutMs) {
                    state = State.HALF_OPEN;
                } else {
                    throw new ProcessingException("Circuit breaker is OPEN");
                }
            }
            
            try (DataProcessor processor = new DataProcessor()) {
                // Simulate unreliable service
                processor.setProcessor(item -> {
                    if (Math.random() < 0.6) { // 60% failure rate
                        throw new RuntimeException("Service unavailable");
                    }
                    return item.toString().toUpperCase();
                });
                
                List<String> results = processor.process(data);
                
                // Success - reset circuit breaker
                if (state == State.HALF_OPEN) {
                    state = State.CLOSED;
                    failureCount = 0;
                }
                
                return results;
                
            } catch (Exception e) {
                recordFailure();
                throw new ProcessingException("Processing failed", e);
            }
        }
        
        private void recordFailure() {
            failureCount++;
            lastFailureTime = System.currentTimeMillis();
            
            if (failureCount >= failureThreshold) {
                state = State.OPEN;
                System.out.println("Circuit breaker opened due to " + failureCount + " failures");
            }
        }
    }
}
```

## Real-World Use Cases

### File Processing System

```java
import com.jclassesv2.core.DataProcessor;
import java.nio.file.*;
import java.io.IOException;
import java.util.stream.Stream;

public class FileProcessingSystem {
    
    public static class FileProcessor {
        private final DataProcessor processor;
        private final Path inputDirectory;
        private final Path outputDirectory;
        private final Path errorDirectory;
        
        public FileProcessor(Path inputDir, Path outputDir, Path errorDir) {
            this.processor = new DataProcessor();
            this.inputDirectory = inputDir;
            this.outputDirectory = outputDir;
            this.errorDirectory = errorDir;
            
            // Configure processor for file content processing
            processor.setProcessor(this::processFileContent);
        }
        
        public void processFiles() {
            try (Stream<Path> files = Files.walk(inputDirectory)) {
                files.filter(Files::isRegularFile)
                     .filter(path -> path.toString().endsWith(".txt"))
                     .forEach(this::processFile);
                     
            } catch (IOException e) {
                System.err.println("Error walking directory: " + e.getMessage());
            }
        }
        
        private void processFile(Path filePath) {
            try {
                System.out.println("Processing file: " + filePath.getFileName());
                
                // Read file content
                List<String> lines = Files.readAllLines(filePath);
                
                // Process content
                List<String> processedLines = processor.process(lines);
                
                // Write processed content
                Path outputFile = outputDirectory.resolve(filePath.getFileName());
                Files.write(outputFile, processedLines);
                
                // Move original file to processed directory
                Path processedFile = inputDirectory.resolve("processed")
                    .resolve(filePath.getFileName());
                Files.createDirectories(processedFile.getParent());
                Files.move(filePath, processedFile);
                
                System.out.println("Successfully processed: " + filePath.getFileName());
                
            } catch (Exception e) {
                handleFileError(filePath, e);
            }
        }
        
        private String processFileContent(Object line) {
            String content = (String) line;
            
            // Example processing: clean and transform content
            content = content.trim();
            
            // Remove empty lines
            if (content.isEmpty()) {
                return null;
            }
            
            // Convert to uppercase and add timestamp
            return content.toUpperCase() + " [Processed: " + 
                   java.time.LocalDateTime.now() + "]";
        }
        
        private void handleFileError(Path filePath, Exception e) {
            try {
                System.err.println("Error processing file " + filePath.getFileName() + 
                                 ": " + e.getMessage());
                
                // Move file to error directory
                Path errorFile = errorDirectory.resolve(filePath.getFileName());
                Files.createDirectories(errorFile.getParent());
                Files.move(filePath, errorFile);
                
                // Create error log
                Path errorLog = errorDirectory.resolve(filePath.getFileName() + ".error");
                Files.write(errorLog, Arrays.asList(
                    "Error processing file: " + filePath.getFileName(),
                    "Error message: " + e.getMessage(),
                    "Timestamp: " + java.time.LocalDateTime.now(),
                    "Stack trace:",
                    java.util.Arrays.toString(e.getStackTrace())
                ));
                
            } catch (IOException ioException) {
                System.err.println("Failed to handle file error: " + ioException.getMessage());
            }
        }
        
        public void close() {
            processor.close();
        }
    }
    
    public static void main(String[] args) {
        // Setup directories
        Path inputDir = Paths.get("input");
        Path outputDir = Paths.get("output");
        Path errorDir = Paths.get("error");
        
        try {
            Files.createDirectories(inputDir);
            Files.createDirectories(outputDir);
            Files.createDirectories(errorDir);
            
            // Create sample files for testing
            createSampleFiles(inputDir);
            
            // Process files
            FileProcessor fileProcessor = new FileProcessor(inputDir, outputDir, errorDir);
            fileProcessor.processFiles();
            fileProcessor.close();
            
            System.out.println("File processing completed");
            
        } catch (Exception e) {
            System.err.println("File processing system failed: " + e.getMessage());
        }
    }
    
    private static void createSampleFiles(Path inputDir) throws IOException {
        // Sample file 1
        Files.write(inputDir.resolve("sample1.txt"), Arrays.asList(
            "Hello World",
            "This is a test file",
            "",
            "Processing with jclassesv2"
        ));
        
        // Sample file 2
        Files.write(inputDir.resolve("sample2.txt"), Arrays.asList(
            "Java Programming",
            "Advanced Examples",
            "File Processing"
        ));
    }
}
```

### Web API Integration

```java
import com.jclassesv2.core.DataProcessor;
import com.jclassesv2.util.CollectionUtils;
import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;
import java.net.URI;
import java.time.Duration;
import java.util.concurrent.CompletableFuture;

public class WebApiIntegration {
    
    public static class ApiDataProcessor {
        private final DataProcessor processor;
        private final HttpClient httpClient;
        private final String baseUrl;
        
        public ApiDataProcessor(String baseUrl) {
            this.processor = new DataProcessor();
            this.httpClient = HttpClient.newBuilder()
                .timeout(Duration.ofSeconds(30))
                .build();
            this.baseUrl = baseUrl;
            
            // Configure processor for API data transformation
            processor.setProcessor(this::transformApiData);
        }
        
        public CompletableFuture<List<String>> processApiData(List<String> endpoints) {
            return CompletableFuture.supplyAsync(() -> {
                try {
                    // Fetch data from multiple API endpoints
                    List<String> apiResponses = fetchApiData(endpoints);
                    
                    // Process the fetched data
                    return processor.process(apiResponses);
                    
                } catch (Exception e) {
                    throw new RuntimeException("API processing failed", e);
                }
            });
        }
        
        private List<String> fetchApiData(List<String> endpoints) {
            List<CompletableFuture<String>> futures = endpoints.stream()
                .map(this::fetchFromEndpoint)
                .collect(Collectors.toList());
            
            // Wait for all API calls to complete
            CompletableFuture<Void> allOf = CompletableFuture.allOf(
                futures.toArray(new CompletableFuture[0])
            );
            
            return allOf.thenApply(v -> 
                futures.stream()
                    .map(CompletableFuture::join)
                    .filter(response -> response != null)
                    .collect(Collectors.toList())
            ).join();
        }
        
        private CompletableFuture<String> fetchFromEndpoint(String endpoint) {
            HttpRequest request = HttpRequest.newBuilder()
                .uri(URI.create(baseUrl + endpoint))
                .header("Accept", "application/json")
                .GET()
                .build();
            
            return httpClient.sendAsync(request, HttpResponse.BodyHandlers.ofString())
                .thenApply(HttpResponse::body)
                .exceptionally(throwable -> {
                    System.err.println("API call failed for " + endpoint + ": " + 
                                     throwable.getMessage());
                    return null;
                });
        }
        
        private String transformApiData(Object apiResponse) {
            String response = (String) apiResponse;
            
            // Example: Extract relevant data from JSON response
            // In real implementation, you'd use a JSON library
            if (response.contains("\"status\":\"success\"")) {
                // Extract and transform data
                return "PROCESSED: " + response.length() + " chars";
            } else {
                return "ERROR: Invalid response";
            }
        }
        
        public void close() {
            processor.close();
        }
    }
    
    public static void main(String[] args) {
        String apiBaseUrl = "https://jsonplaceholder.typicode.com";
        
        List<String> endpoints = Arrays.asList(
            "/posts/1",
            "/posts/2", 
            "/users/1",
            "/albums/1"
        );
        
        ApiDataProcessor apiProcessor = new ApiDataProcessor(apiBaseUrl);
        
        try {
            CompletableFuture<List<String>> futureResults = 
                apiProcessor.processApiData(endpoints);
            
            // Handle results asynchronously
            futureResults.thenAccept(results -> {
                System.out.println("API processing completed:");
                results.forEach(result -> System.out.println("  " + result));
            }).exceptionally(throwable -> {
                System.err.println("API processing failed: " + throwable.getMessage());
                return null;
            });
            
            // Wait for completion (in real apps, you might not want to block)
            futureResults.join();
            
        } finally {
            apiProcessor.close();
        }
    }
}
```

## Performance Optimization

### Memory-Efficient Processing

```java
import com.jclassesv2.core.DataProcessor;
import java.lang.management.ManagementFactory;
import java.lang.management.MemoryMXBean;

public class MemoryOptimizedProcessing {
    
    private static final MemoryMXBean memoryBean = ManagementFactory.getMemoryMXBean();
    
    public static class MemoryAwareProcessor {
        private final DataProcessor processor;
        private final long maxMemoryThreshold;
        private final int chunkSize;
        
        public MemoryAwareProcessor(long maxMemoryThresholdMB, int chunkSize) {
            this.processor = new DataProcessor();
            this.maxMemoryThreshold = maxMemoryThresholdMB * 1024 * 1024; // Convert to bytes
            this.chunkSize = chunkSize;
        }
        
        public List<String> processLargeDataset(List<String> dataset) {
            List<String> results = new ArrayList<>();
            
            for (int i = 0; i < dataset.size(); i += chunkSize) {
                // Check memory usage before processing each chunk
                if (getUsedMemory() > maxMemoryThreshold) {
                    System.gc(); // Suggest garbage collection
                    
                    if (getUsedMemory() > maxMemoryThreshold) {
                        System.err.println("Memory threshold exceeded, reducing chunk size");
                        return processWithReducedChunks(dataset.subList(i, dataset.size()), 
                                                      results);
                    }
                }
                
                int end = Math.min(i + chunkSize, dataset.size());
                List<String> chunk = dataset.subList(i, end);
                
                try {
                    List<String> chunkResults = processor.process(chunk);
                    results.addAll(chunkResults);
                    
                    System.out.println("Processed chunk " + (i / chunkSize + 1) + 
                                     ", Memory usage: " + (getUsedMemory() / 1024 / 1024) + " MB");
                    
                } catch (Exception e) {
                    System.err.println("Chunk processing failed: " + e.getMessage());
                }
            }
            
            return results;
        }
        
        private List<String> processWithReducedChunks(List<String> remainingData, 
                                                     List<String> existingResults) {
            int reducedChunkSize = chunkSize / 2;
            
            if (reducedChunkSize < 10) {
                System.err.println("Cannot reduce chunk size further, processing stopped");
                return existingResults;
            }
            
            System.out.println("Reducing chunk size to: " + reducedChunkSize);
            
            MemoryAwareProcessor reducedProcessor = 
                new MemoryAwareProcessor(maxMemoryThreshold / 1024 / 1024, reducedChunkSize);
            
            List<String> remainingResults = reducedProcessor.processLargeDataset(remainingData);
            existingResults.addAll(remainingResults);
            
            return existingResults;
        }
        
        private long getUsedMemory() {
            return memoryBean.getHeapMemoryUsage().getUsed();
        }
        
        public void close() {
            processor.close();
        }
    }
    
    public static void main(String[] args) {
        // Generate large dataset
        List<String> largeDataset = generateLargeDataset(50000);
        
        System.out.println("Starting memory-optimized processing of " + 
                         largeDataset.size() + " items");
        
        MemoryAwareProcessor memoryProcessor = new MemoryAwareProcessor(100, 1000); // 100MB threshold
        
        try {
            long startTime = System.currentTimeMillis();
            List<String> results = memoryProcessor.processLargeDataset(largeDataset);
            long endTime = System.currentTimeMillis();
            
            System.out.println("Processing completed:");
            System.out.println("  Input items: " + largeDataset.size());
            System.out.println("  Output items: " + results.size());
            System.out.println("  Processing time: " + (endTime - startTime) + " ms");
            System.out.println("  Final memory usage: " + 
                             (memoryBean.getHeapMemoryUsage().getUsed() / 1024 / 1024) + " MB");
            
        } finally {
            memoryProcessor.close();
        }
    }
    
    private static List<String> generateLargeDataset(int size) {
        return IntStream.range(0, size)
            .mapToObj(i -> "Large data item with some content " + i + 
                          " that takes up memory space")
            .collect(Collectors.toList());
    }
}
```

---

These advanced examples demonstrate sophisticated usage patterns of the jclassesv2 library, including complex data processing pipelines, parallel processing strategies, comprehensive error handling, and real-world integration scenarios. Each example includes detailed explanations and can be adapted for specific use cases.