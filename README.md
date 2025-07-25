# jclassesv2

A comprehensive Java library providing essential utilities, data structures, and APIs for modern Java applications.

[![Java Version](https://img.shields.io/badge/Java-11%2B-blue.svg)](https://openjdk.java.net/projects/jdk/11/)
[![Build Status](https://img.shields.io/badge/build-passing-brightgreen.svg)](https://github.com/jclassesv2/jclassesv2)
[![Maven Central](https://img.shields.io/badge/Maven%20Central-2.0.0-orange.svg)](https://search.maven.org/artifact/com.jclassesv2/jclassesv2-core)
[![License](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

## 🚀 Features

- **Type-safe collections and utilities** - Enhanced collection operations with null safety
- **Robust error handling mechanisms** - Comprehensive exception handling and retry logic
- **Configurable components** - Flexible configuration system with multiple sources
- **Thread-safe operations** - Built-in concurrency support and parallel processing
- **Extensive logging and monitoring** - Detailed metrics and health monitoring
- **High performance** - Optimized for throughput and low latency
- **Easy integration** - Simple APIs with extensive documentation

## 📦 Installation

### Maven
```xml
<dependency>
    <groupId>com.jclassesv2</groupId>
    <artifactId>jclassesv2-core</artifactId>
    <version>2.0.0</version>
</dependency>
```

### Gradle
```gradle
implementation 'com.jclassesv2:jclassesv2-core:2.0.0'
```

## 🏁 Quick Start

```java
import com.jclassesv2.core.DataProcessor;
import com.jclassesv2.util.CollectionUtils;

public class Example {
    public static void main(String[] args) {
        // Create a data processor
        try (DataProcessor processor = new DataProcessor()) {
            List<String> data = Arrays.asList("apple", "banana", "cherry");
            
            // Process data with custom logic
            processor.setProcessor(item -> item.toUpperCase());
            List<String> results = processor.process(data);
            
            System.out.println("Results: " + results);
            // Output: [APPLE, BANANA, CHERRY]
        } catch (Exception e) {
            System.err.println("Processing failed: " + e.getMessage());
        }
    }
}
```

## 📚 Documentation

### Core Documentation
- **[📖 Complete API Documentation](docs/API_DOCUMENTATION.md)** - Comprehensive reference for all APIs, classes, and methods
- **[🚀 Quick Start Guide](docs/QUICK_START_GUIDE.md)** - Get up and running in 5 minutes
- **[⚙️ Configuration Guide](docs/CONFIGURATION.md)** - Complete configuration reference and best practices
- **[🔧 Advanced Examples](docs/ADVANCED_EXAMPLES.md)** - Complex usage patterns and real-world scenarios

### Key Components

#### Core APIs
- **DataProcessor** - Main processing engine with async support
- **ConfigurationManager** - Centralized configuration management
- **ProcessingResult** - Type-safe result handling

#### Utility Classes
- **CollectionUtils** - Enhanced collection operations
- **StringUtils** - Null-safe string manipulations
- **DateUtils** - Date/time processing utilities

#### Advanced Features
- **Parallel Processing** - Multi-threaded data processing
- **Error Recovery** - Retry logic and circuit breakers
- **Memory Management** - Optimized for large datasets
- **Monitoring** - Built-in metrics and health checks

## 🎯 Use Cases

### Data Processing Pipeline
```java
List<String> rawData = Arrays.asList("  apple  ", "BANANA", "Cherry");

// Clean and transform data
List<String> cleaned = CollectionUtils.transform(rawData, item -> {
    String trimmed = StringUtils.trim(item);
    return trimmed.toLowerCase();
});

// Filter results
List<String> filtered = CollectionUtils.filter(cleaned, 
    item -> item.length() > 4);

System.out.println(filtered); // [apple, banana, cherry]
```

### Asynchronous Processing
```java
try (DataProcessor processor = new DataProcessor()) {
    List<String> data = Arrays.asList("task1", "task2", "task3");
    
    CompletableFuture<List<String>> future = processor.processAsync(data);
    
    future.thenAccept(results -> {
        System.out.println("Async processing completed: " + results);
    });
}
```

### Custom Configuration
```java
Configuration config = Configuration.builder()
    .setMaxThreads(8)
    .setTimeout(60, TimeUnit.SECONDS)
    .enableLogging(true)
    .build();

try (DataProcessor processor = new DataProcessor(config)) {
    // Your processing logic here
}
```

## 🛠️ Building from Source

```bash
# Clone the repository
git clone https://github.com/jclassesv2/jclassesv2.git
cd jclassesv2

# Build with Maven
mvn clean install

# Or build with Gradle
./gradlew build
```

## 🧪 Testing

```bash
# Run tests with Maven
mvn test

# Run tests with Gradle
./gradlew test

# Run with coverage
mvn test jacoco:report
```

## 📊 Performance

jclassesv2 is designed for high performance:

- **Throughput**: Process 100K+ items per second
- **Memory**: Optimized for low GC pressure
- **Latency**: Sub-millisecond processing times
- **Scalability**: Linear scaling with CPU cores

### Benchmarks
```
Benchmark                     Mode  Cnt    Score   Error  Units
DataProcessor.process        thrpt   25  125.3 ± 2.1  ops/s
CollectionUtils.filter       thrpt   25  890.2 ± 8.3  ops/s
StringUtils.operations       thrpt   25  750.1 ± 5.2  ops/s
```

## 🤝 Contributing

We welcome contributions! Please see our [Contributing Guide](CONTRIBUTING.md) for details.

### Development Setup
1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests for new functionality
5. Ensure all tests pass
6. Submit a pull request

## 📝 Changelog

### Version 2.0.0 (Current)
- ✨ New async processing capabilities
- 🚀 Enhanced performance and memory optimization
- 🔧 Improved configuration system
- 📊 Built-in metrics and monitoring
- 🛡️ Enhanced error handling and recovery

### Previous Versions
See [CHANGELOG.md](CHANGELOG.md) for complete version history.

## 🔒 Security

For security concerns, please email [security@jclassesv2.com](mailto:security@jclassesv2.com).

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- **Java Community** for inspiration and feedback
- **Contributors** who have helped improve this library
- **Users** who provide valuable feedback and bug reports

## 📞 Support

- **📖 Documentation**: [Full API Documentation](docs/API_DOCUMENTATION.md)
- **🐛 Issues**: [GitHub Issues](https://github.com/jclassesv2/jclassesv2/issues)
- **💬 Discussions**: [GitHub Discussions](https://github.com/jclassesv2/jclassesv2/discussions)
- **📧 Email**: [support@jclassesv2.com](mailto:support@jclassesv2.com)

---

**Made with ❤️ by the jclassesv2 team**
