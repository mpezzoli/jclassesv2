# Configuration Guide

This guide covers all configuration options and customization capabilities available in jclassesv2.

## Table of Contents

1. [Configuration Overview](#configuration-overview)
2. [Property-Based Configuration](#property-based-configuration)
3. [Programmatic Configuration](#programmatic-configuration)
4. [Environment-Specific Configuration](#environment-specific-configuration)
5. [Performance Tuning](#performance-tuning)
6. [Security Configuration](#security-configuration)
7. [Logging Configuration](#logging-configuration)
8. [Advanced Configuration Patterns](#advanced-configuration-patterns)

## Configuration Overview

jclassesv2 supports multiple configuration approaches:

- **Properties files** (`.properties`, `.yaml`, `.json`)
- **System properties**
- **Environment variables**
- **Programmatic configuration**
- **Configuration builders**

### Configuration Precedence

Configuration values are resolved in the following order (highest to lowest priority):

1. Programmatic configuration
2. System properties
3. Environment variables
4. Configuration files
5. Default values

## Property-Based Configuration

### Core Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `jclassesv2.max.threads` | int | 4 | Maximum number of processing threads |
| `jclassesv2.timeout.seconds` | int | 30 | Default timeout in seconds |
| `jclassesv2.retry.attempts` | int | 3 | Number of retry attempts |
| `jclassesv2.logging.enabled` | boolean | true | Enable/disable logging |
| `jclassesv2.cache.size` | int | 1000 | Maximum cache size |
| `jclassesv2.cache.ttl.seconds` | int | 300 | Cache TTL in seconds |
| `jclassesv2.batch.size` | int | 100 | Default batch size for processing |
| `jclassesv2.async.enabled` | boolean | true | Enable asynchronous processing |

### Performance Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `jclassesv2.thread.pool.core.size` | int | 2 | Core thread pool size |
| `jclassesv2.thread.pool.max.size` | int | 10 | Maximum thread pool size |
| `jclassesv2.thread.pool.keep.alive.seconds` | int | 60 | Thread keep-alive time |
| `jclassesv2.queue.capacity` | int | 1000 | Processing queue capacity |
| `jclassesv2.memory.threshold.mb` | int | 512 | Memory usage threshold |
| `jclassesv2.gc.suggestion.enabled` | boolean | false | Enable GC suggestions |

### Monitoring Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `jclassesv2.metrics.enabled` | boolean | true | Enable metrics collection |
| `jclassesv2.metrics.interval.seconds` | int | 60 | Metrics collection interval |
| `jclassesv2.health.check.enabled` | boolean | true | Enable health checks |
| `jclassesv2.statistics.enabled` | boolean | true | Enable processing statistics |

### Example Properties File

```properties
# jclassesv2 Configuration - application.properties

# Core Configuration
jclassesv2.max.threads=8
jclassesv2.timeout.seconds=60
jclassesv2.retry.attempts=5
jclassesv2.logging.enabled=true

# Performance Configuration
jclassesv2.cache.size=2000
jclassesv2.cache.ttl.seconds=600
jclassesv2.batch.size=200
jclassesv2.async.enabled=true

# Thread Pool Configuration
jclassesv2.thread.pool.core.size=4
jclassesv2.thread.pool.max.size=16
jclassesv2.thread.pool.keep.alive.seconds=120
jclassesv2.queue.capacity=2000

# Memory Management
jclassesv2.memory.threshold.mb=1024
jclassesv2.gc.suggestion.enabled=true

# Monitoring
jclassesv2.metrics.enabled=true
jclassesv2.metrics.interval.seconds=30
jclassesv2.health.check.enabled=true
jclassesv2.statistics.enabled=true
```

### YAML Configuration

```yaml
# jclassesv2 Configuration - application.yml
jclassesv2:
  core:
    max-threads: 8
    timeout-seconds: 60
    retry-attempts: 5
    logging-enabled: true
    
  performance:
    cache:
      size: 2000
      ttl-seconds: 600
    batch-size: 200
    async-enabled: true
    
  thread-pool:
    core-size: 4
    max-size: 16
    keep-alive-seconds: 120
    queue-capacity: 2000
    
  memory:
    threshold-mb: 1024
    gc-suggestion-enabled: true
    
  monitoring:
    metrics:
      enabled: true
      interval-seconds: 30
    health-check-enabled: true
    statistics-enabled: true
```

## Programmatic Configuration

### Using Configuration Builder

```java
import com.jclassesv2.config.Configuration;
import java.util.concurrent.TimeUnit;

public class ProgrammaticConfig {
    
    public static Configuration createBasicConfiguration() {
        return Configuration.builder()
            .setMaxThreads(8)
            .setTimeout(60, TimeUnit.SECONDS)
            .setRetryAttempts(5)
            .enableLogging(true)
            .setCacheSize(2000)
            .build();
    }
    
    public static Configuration createHighPerformanceConfiguration() {
        return Configuration.builder()
            .setMaxThreads(Runtime.getRuntime().availableProcessors() * 2)
            .setTimeout(2, TimeUnit.MINUTES)
            .setRetryAttempts(3)
            .setBatchSize(500)
            .enableAsyncProcessing(true)
            .setCacheSize(5000)
            .setCacheTtl(10, TimeUnit.MINUTES)
            .setThreadPoolCoreSize(8)
            .setThreadPoolMaxSize(32)
            .setQueueCapacity(5000)
            .setMemoryThreshold(2048) // 2GB
            .enableGcSuggestion(true)
            .enableMetrics(true)
            .setMetricsInterval(30, TimeUnit.SECONDS)
            .enableHealthCheck(true)
            .build();
    }
    
    public static Configuration createDevelopmentConfiguration() {
        return Configuration.builder()
            .setMaxThreads(2)
            .setTimeout(30, TimeUnit.SECONDS)
            .setRetryAttempts(1)
            .enableLogging(true)
            .setLogLevel(LogLevel.DEBUG)
            .setCacheSize(100)
            .enableAsyncProcessing(false)
            .enableMetrics(false)
            .build();
    }
}
```

### Dynamic Configuration Updates

```java
import com.jclassesv2.config.ConfigurationManager;
import com.jclassesv2.config.ConfigurationListener;

public class DynamicConfiguration {
    
    public static void setupDynamicConfiguration() {
        ConfigurationManager configManager = ConfigurationManager.getInstance();
        
        // Register configuration change listener
        configManager.addConfigurationListener(new ConfigurationListener() {
            @Override
            public void onConfigurationChanged(String key, Object oldValue, Object newValue) {
                System.out.println("Configuration changed: " + key + 
                                 " from " + oldValue + " to " + newValue);
                
                // Handle specific configuration changes
                switch (key) {
                    case "jclassesv2.max.threads":
                        updateThreadPoolSize((Integer) newValue);
                        break;
                    case "jclassesv2.cache.size":
                        updateCacheSize((Integer) newValue);
                        break;
                    case "jclassesv2.logging.enabled":
                        updateLoggingState((Boolean) newValue);
                        break;
                }
            }
        });
        
        // Enable configuration file watching
        configManager.enableFileWatching("application.properties", 
                                        Duration.ofSeconds(30));
    }
    
    private static void updateThreadPoolSize(int newSize) {
        // Update thread pool configuration
        System.out.println("Updating thread pool size to: " + newSize);
    }
    
    private static void updateCacheSize(int newSize) {
        // Update cache configuration
        System.out.println("Updating cache size to: " + newSize);
    }
    
    private static void updateLoggingState(boolean enabled) {
        // Update logging configuration
        System.out.println("Updating logging enabled to: " + enabled);
    }
}
```

## Environment-Specific Configuration

### Development Environment

```properties
# config-development.properties
jclassesv2.max.threads=2
jclassesv2.timeout.seconds=30
jclassesv2.retry.attempts=1
jclassesv2.logging.enabled=true
jclassesv2.log.level=DEBUG
jclassesv2.cache.size=100
jclassesv2.async.enabled=false
jclassesv2.metrics.enabled=false
jclassesv2.statistics.enabled=true
```

### Testing Environment

```properties
# config-testing.properties
jclassesv2.max.threads=4
jclassesv2.timeout.seconds=45
jclassesv2.retry.attempts=2
jclassesv2.logging.enabled=true
jclassesv2.log.level=INFO
jclassesv2.cache.size=500
jclassesv2.async.enabled=true
jclassesv2.metrics.enabled=true
jclassesv2.statistics.enabled=true
```

### Production Environment

```properties
# config-production.properties
jclassesv2.max.threads=16
jclassesv2.timeout.seconds=120
jclassesv2.retry.attempts=5
jclassesv2.logging.enabled=true
jclassesv2.log.level=WARN
jclassesv2.cache.size=5000
jclassesv2.cache.ttl.seconds=1800
jclassesv2.async.enabled=true
jclassesv2.memory.threshold.mb=4096
jclassesv2.metrics.enabled=true
jclassesv2.health.check.enabled=true
```

### Environment-Based Configuration Loading

```java
import com.jclassesv2.config.ConfigurationManager;

public class EnvironmentConfiguration {
    
    public static void loadEnvironmentSpecificConfiguration() {
        String environment = System.getProperty("environment", "development");
        String configFile = "config-" + environment + ".properties";
        
        try {
            ConfigurationManager.loadConfiguration(configFile);
            System.out.println("Loaded configuration for environment: " + environment);
            
        } catch (ConfigurationException e) {
            System.err.println("Failed to load environment configuration, using defaults");
            loadDefaultConfiguration();
        }
    }
    
    private static void loadDefaultConfiguration() {
        Configuration defaultConfig = Configuration.builder()
            .setMaxThreads(4)
            .setTimeout(30, TimeUnit.SECONDS)
            .setRetryAttempts(3)
            .enableLogging(true)
            .build();
            
        ConfigurationManager.setConfiguration(defaultConfig);
    }
}
```

## Performance Tuning

### Thread Pool Configuration

```java
import com.jclassesv2.config.ThreadPoolConfiguration;

public class ThreadPoolTuning {
    
    public static Configuration optimizeForCpuIntensiveTasks() {
        int processors = Runtime.getRuntime().availableProcessors();
        
        return Configuration.builder()
            .setMaxThreads(processors) // CPU-bound: 1 thread per core
            .setThreadPoolCoreSize(processors / 2)
            .setThreadPoolMaxSize(processors)
            .setThreadPoolKeepAlive(30, TimeUnit.SECONDS)
            .setQueueCapacity(processors * 10)
            .build();
    }
    
    public static Configuration optimizeForIoIntensiveTasks() {
        int processors = Runtime.getRuntime().availableProcessors();
        
        return Configuration.builder()
            .setMaxThreads(processors * 4) // IO-bound: more threads
            .setThreadPoolCoreSize(processors)
            .setThreadPoolMaxSize(processors * 4)
            .setThreadPoolKeepAlive(60, TimeUnit.SECONDS)
            .setQueueCapacity(processors * 50)
            .build();
    }
    
    public static Configuration optimizeForMixedWorkload() {
        int processors = Runtime.getRuntime().availableProcessors();
        
        return Configuration.builder()
            .setMaxThreads(processors * 2) // Balanced approach
            .setThreadPoolCoreSize(processors)
            .setThreadPoolMaxSize(processors * 3)
            .setThreadPoolKeepAlive(45, TimeUnit.SECONDS)
            .setQueueCapacity(processors * 25)
            .build();
    }
}
```

### Memory Optimization

```java
import com.jclassesv2.config.MemoryConfiguration;

public class MemoryTuning {
    
    public static Configuration optimizeForLowMemory() {
        return Configuration.builder()
            .setCacheSize(100)
            .setCacheTtl(60, TimeUnit.SECONDS)
            .setBatchSize(50)
            .setMemoryThreshold(256) // 256MB
            .enableGcSuggestion(true)
            .setQueueCapacity(100)
            .build();
    }
    
    public static Configuration optimizeForHighMemory() {
        return Configuration.builder()
            .setCacheSize(10000)
            .setCacheTtl(30, TimeUnit.MINUTES)
            .setBatchSize(1000)
            .setMemoryThreshold(8192) // 8GB
            .enableGcSuggestion(false)
            .setQueueCapacity(10000)
            .build();
    }
    
    public static Configuration adaptiveMemoryConfiguration() {
        long maxMemory = Runtime.getRuntime().maxMemory() / (1024 * 1024); // MB
        
        Configuration.Builder builder = Configuration.builder();
        
        if (maxMemory < 512) {
            // Low memory environment
            builder.setCacheSize(100)
                   .setBatchSize(25)
                   .setMemoryThreshold(128);
        } else if (maxMemory < 2048) {
            // Medium memory environment
            builder.setCacheSize(1000)
                   .setBatchSize(100)
                   .setMemoryThreshold(512);
        } else {
            // High memory environment
            builder.setCacheSize(5000)
                   .setBatchSize(500)
                   .setMemoryThreshold(2048);
        }
        
        return builder.build();
    }
}
```

## Security Configuration

### SSL/TLS Configuration

```java
import com.jclassesv2.config.SecurityConfiguration;

public class SecurityConfig {
    
    public static Configuration createSecureConfiguration() {
        return Configuration.builder()
            .enableSsl(true)
            .setSslProtocol("TLSv1.3")
            .setKeystorePath("/path/to/keystore.jks")
            .setKeystorePassword("keystorePassword")
            .setTruststorePath("/path/to/truststore.jks")
            .setTruststorePassword("truststorePassword")
            .enableHostnameVerification(true)
            .build();
    }
    
    public static Configuration createMutualTlsConfiguration() {
        return Configuration.builder()
            .enableSsl(true)
            .enableClientAuth(true)
            .setSslProtocol("TLSv1.3")
            .setKeystorePath("/path/to/server-keystore.jks")
            .setKeystorePassword("serverKeystorePassword")
            .setClientKeystorePath("/path/to/client-keystore.jks")
            .setClientKeystorePassword("clientKeystorePassword")
            .setTruststorePath("/path/to/truststore.jks")
            .setTruststorePassword("truststorePassword")
            .build();
    }
}
```

### Authentication Configuration

```properties
# Security properties
jclassesv2.security.authentication.enabled=true
jclassesv2.security.authentication.type=JWT
jclassesv2.security.jwt.secret=your-secret-key
jclassesv2.security.jwt.expiration.minutes=60
jclassesv2.security.authorization.enabled=true
jclassesv2.security.ssl.enabled=true
jclassesv2.security.ssl.protocol=TLSv1.3
```

## Logging Configuration

### Log4j2 Configuration

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="WARN">
    <Appenders>
        <Console name="Console" target="SYSTEM_OUT">
            <PatternLayout pattern="%d{HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n"/>
        </Console>
        
        <RollingFile name="FileAppender" fileName="logs/jclassesv2.log"
                     filePattern="logs/jclassesv2-%i.log">
            <PatternLayout pattern="%d{yyyy-MM-dd HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n"/>
            <Policies>
                <SizeBasedTriggeringPolicy size="10MB"/>
            </Policies>
            <DefaultRolloverStrategy max="10"/>
        </RollingFile>
    </Appenders>
    
    <Loggers>
        <Logger name="com.jclassesv2" level="INFO" additivity="false">
            <AppenderRef ref="Console"/>
            <AppenderRef ref="FileAppender"/>
        </Logger>
        
        <Root level="WARN">
            <AppenderRef ref="Console"/>
        </Root>
    </Loggers>
</Configuration>
```

### Programmatic Logging Configuration

```java
import com.jclassesv2.config.LoggingConfiguration;

public class LoggingConfig {
    
    public static Configuration createLoggingConfiguration() {
        return Configuration.builder()
            .enableLogging(true)
            .setLogLevel(LogLevel.INFO)
            .setLogFormat("%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger - %msg%n")
            .setLogFile("logs/jclassesv2.log")
            .setMaxLogFileSize("10MB")
            .setMaxLogFiles(10)
            .enableAsyncLogging(true)
            .setLogBufferSize(1000)
            .build();
    }
    
    public static void configureStructuredLogging() {
        LoggingConfiguration.builder()
            .enableStructuredLogging(true)
            .setLogFormat("json")
            .addStructuredField("application", "jclassesv2")
            .addStructuredField("version", "2.0.0")
            .addStructuredField("environment", System.getProperty("environment", "development"))
            .build();
    }
}
```

## Advanced Configuration Patterns

### Configuration Profiles

```java
import com.jclassesv2.config.ConfigurationProfile;

public class ConfigurationProfiles {
    
    @ConfigurationProfile("development")
    public static class DevelopmentProfile {
        @Bean
        public Configuration configuration() {
            return Configuration.builder()
                .setMaxThreads(2)
                .setTimeout(30, TimeUnit.SECONDS)
                .enableLogging(true)
                .setLogLevel(LogLevel.DEBUG)
                .build();
        }
    }
    
    @ConfigurationProfile("production")
    public static class ProductionProfile {
        @Bean
        public Configuration configuration() {
            return Configuration.builder()
                .setMaxThreads(16)
                .setTimeout(120, TimeUnit.SECONDS)
                .enableLogging(true)
                .setLogLevel(LogLevel.WARN)
                .enableMetrics(true)
                .build();
        }
    }
}
```

### Configuration Validation

```java
import com.jclassesv2.config.ConfigurationValidator;

public class ConfigurationValidation {
    
    public static void validateConfiguration(Configuration config) {
        ConfigurationValidator validator = new ConfigurationValidator();
        
        validator.addRule("maxThreads", value -> {
            int threads = (Integer) value;
            return threads > 0 && threads <= 100;
        }, "Max threads must be between 1 and 100");
        
        validator.addRule("timeout", value -> {
            Duration timeout = (Duration) value;
            return timeout.toSeconds() >= 1 && timeout.toSeconds() <= 3600;
        }, "Timeout must be between 1 second and 1 hour");
        
        validator.addRule("cacheSize", value -> {
            int size = (Integer) value;
            return size >= 0 && size <= 100000;
        }, "Cache size must be between 0 and 100000");
        
        ValidationResult result = validator.validate(config);
        
        if (!result.isValid()) {
            throw new ConfigurationException("Configuration validation failed: " + 
                                           result.getErrors());
        }
    }
}
```

### Configuration Migration

```java
import com.jclassesv2.config.ConfigurationMigration;

public class ConfigurationMigration {
    
    public static void migrateFromV1ToV2(String oldConfigFile, String newConfigFile) {
        ConfigurationMigration migration = new ConfigurationMigration();
        
        // Define migration rules
        migration.addMappingRule("old.thread.count", "jclassesv2.max.threads");
        migration.addMappingRule("old.timeout.ms", "jclassesv2.timeout.seconds", 
                                value -> ((Integer) value) / 1000);
        migration.addMappingRule("old.cache.enabled", "jclassesv2.cache.size", 
                                value -> ((Boolean) value) ? 1000 : 0);
        
        // Add default values for new properties
        migration.addDefaultValue("jclassesv2.async.enabled", true);
        migration.addDefaultValue("jclassesv2.metrics.enabled", true);
        
        // Perform migration
        try {
            migration.migrate(oldConfigFile, newConfigFile);
            System.out.println("Configuration migrated successfully");
        } catch (Exception e) {
            System.err.println("Configuration migration failed: " + e.getMessage());
        }
    }
}
```

### Configuration Templates

```java
import com.jclassesv2.config.ConfigurationTemplate;

public class ConfigurationTemplates {
    
    public static Configuration fromTemplate(String templateName, Map<String, Object> variables) {
        ConfigurationTemplate template = ConfigurationTemplate.load(templateName);
        
        return template.build(variables);
    }
    
    public static void createMicroserviceTemplate() {
        ConfigurationTemplate template = ConfigurationTemplate.builder()
            .addVariable("service.name", String.class, true)
            .addVariable("service.port", Integer.class, 8080)
            .addVariable("max.threads", Integer.class, 8)
            .setTemplate("""
                jclassesv2.service.name=${service.name}
                jclassesv2.service.port=${service.port}
                jclassesv2.max.threads=${max.threads}
                jclassesv2.metrics.enabled=true
                jclassesv2.health.check.enabled=true
                """)
            .build();
            
        template.save("microservice-template.properties");
    }
}
```

## Best Practices

### 1. Environment Separation
Always use separate configuration files for different environments:
```
config-development.properties
config-testing.properties
config-staging.properties
config-production.properties
```

### 2. Sensitive Information
Never store sensitive information in configuration files. Use environment variables or secure vaults:
```java
Configuration config = Configuration.builder()
    .setDatabasePassword(System.getenv("DB_PASSWORD"))
    .setApiKey(System.getenv("API_KEY"))
    .build();
```

### 3. Configuration Validation
Always validate configuration before using it:
```java
public void validateConfiguration(Configuration config) {
    if (config.getMaxThreads() <= 0) {
        throw new IllegalArgumentException("Max threads must be positive");
    }
    if (config.getTimeout().isNegative()) {
        throw new IllegalArgumentException("Timeout cannot be negative");
    }
}
```

### 4. Documentation
Document all configuration properties:
```properties
# Maximum number of threads for processing (1-100)
jclassesv2.max.threads=8

# Processing timeout in seconds (1-3600)
jclassesv2.timeout.seconds=60

# Cache size in number of entries (0-100000, 0 disables cache)
jclassesv2.cache.size=1000
```

### 5. Configuration Testing
Test configuration in different scenarios:
```java
@Test
public void testHighLoadConfiguration() {
    Configuration config = Configuration.builder()
        .setMaxThreads(32)
        .setTimeout(5, TimeUnit.MINUTES)
        .setCacheSize(10000)
        .build();
        
    // Test with high load
    assertThat(config.canHandleLoad(1000)).isTrue();
}
```

---

This configuration guide provides comprehensive coverage of all configuration options available in jclassesv2, from basic property files to advanced programmatic configuration patterns.