# Logging

Spring Boot provides logging capabilities using **SLF4J** (Simple Logging Facade for Java) and **Logback** (the default logging framework).

### **Content**

1. **Basic Setup**: How to configure logging in Spring Boot.
2. **Logging Levels**: How to control what gets logged.
3. **Customizing Log Files**: Setting up a file where logs will be stored.
4. **Custom Logging Messages**: How to log specific messages or data during your app’s execution.
5. **Additional Configuration**: Logging rotation, pattern formatting, etc.

---

### **Basic Logging Setup**

Spring Boot uses **SLF4J** with **Logback** as the default logging framework, so in most cases, you don't need to add any external dependencies. However, for flexibility, let’s ensure that you have the logging dependencies in your `pom.xml`.

```xml
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
</dependency>

<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
</dependency>
```

---

### **Basic Logging Example in Spring Boot**

You can log data from any class using the `Logger` interface. Here's an example of how to log data in a service or controller class:

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class AlertController {

    private static final Logger logger = LoggerFactory.getLogger(AlertController.class);

    @GetMapping("/alerts")
    public String getAlerts() {
        logger.info("Fetching the list of alerts");

        // Log different levels
        logger.debug("Debug level logging enabled");
        logger.warn("This is a warning");
        logger.error("This is an error log");

        return "Todo List";
    }
}

```

In this example:

- `logger.info()`, `logger.debug()`, `logger.warn()`, and `logger.error()` log messages at various levels.
- These logs will appear in your console by default.

---

### **Configuring Log Output to a File**

By default, Spring Boot logs only to the **console**. To configure logging to write to a **file**, you need to set this up in the `application.properties` or `application.yml` file.

### **Using `application.properties`**

In your `src/main/resources/application.properties`, you can define file-based logging like this:

```
logging.file.name=logs/myapp.log
logging.level.root=INFO

# Optional
logging.level.com.yourpackage=DEBUG
logging.file.max-size=10MB
logging.file.max-history=10
```

### **Using `application.yml`**

Alternatively, if you’re using YAML configuration (`src/main/resources/application.yml`), the setup will look like this:

```yaml
logging:
  file:
    name: logs/application.log
    max-size: 10MB
    max-history: 10
  level:
    root: INFO
    com.yourpackage: DEBUG
```

### **What This Does:**

- Logs will be written to a file located at `logs/`application`.log`.
- The logging level is set to `INFO`, but you can adjust it to `DEBUG`, `WARN`, `ERROR`, etc.
- **Log rotation** ensures your log file doesn't grow endlessly (configurable with `max-size` and `max-history`).

---

### **Custom Log Messages**

### **Example: Logging HTTP Request Details**

You can log useful information like request details in your controller:

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.web.bind.annotation.*;

import javax.servlet.http.HttpServletRequest;

@RestController
public class TodoController {

    private static final Logger logger = LoggerFactory.getLogger(TodoController.class);

    @PostMapping("/todos")
    public String createTodo(@RequestBody String todo, HttpServletRequest request) {
        // Log request details (e.g., IP address, headers, etc.)
        String clientIp = request.getRemoteAddr();
        logger.info("Received a request from IP: " + clientIp);
        logger.info("Todo content: " + todo);

        // Simulate some business logic
        logger.debug("Business logic for creating a todo executed");

        return "Todo created";
    }
}
```

This example shows how to:

- Log the **IP address** of the incoming request.
- Log the **payload** (`todo`).
- Log a **debug** message for business logic.

---

### **Advanced Logback Configuration (Optional)**

If you need more advanced control over your logging, you can create a **custom Logback configuration file** (`logback-spring.xml`). This file allows for customization of log formats, rolling policies, and different appenders (console, file, etc.).

### Example: `src/main/resources/logback-spring.xml`

```xml
<configuration>

    <!-- Define log format (timestamp, log level, logger, and message) -->
    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>logs/myapp.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!-- Logs will rotate daily, keeping up to 30 days of logs -->
            <fileNamePattern>logs/myapp-%d{yyyy-MM-dd}.log</fileNamePattern>
            <maxHistory>30</maxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <!-- Log to the console as well -->
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <!-- Define logging levels -->
    <root level="INFO">
        <appender-ref ref="CONSOLE" />
        <appender-ref ref="FILE" />
    </root>

    <!-- Optionally fine-tune logging levels for specific packages -->
    <logger name="com.yourpackage" level="DEBUG"/>

</configuration>

```

### **What This Does:**

- Logs will be written both to the **console** and **file**.
- The log file will **rotate daily**, and the application will keep logs for up to 30 days.
- The log format is customizable, showing a **timestamp**, **log level**, **thread name**, and **message**.

---

### **Logging Exceptions**

You can also log exceptions using `logger.error()` to capture stack traces:

```java
@GetMapping("/todo/{id}")
public String getTodoById(@PathVariable int id) {
    try {
        // Simulate fetching a todo (may throw an exception)
        if (id <= 0) {
            throw new IllegalArgumentException("Invalid todo ID");
        }
        return "Todo";
    } catch (IllegalArgumentException ex) {
        // Log the exception
        logger.error("Error fetching todo with ID " + id, ex);
        throw ex;  // Rethrow the exception if needed
    }
}

```

- `logger.error("message", ex)` captures the stack trace for the exception, which can be invaluable when debugging issues.

---

### **Using a Logging Facade (Optional)**

Spring Boot uses **SLF4J** as a facade for logging. If you're working on a bigger project and want to make your logging framework interchangeable (i.e., replace Logback with another framework in the future), stick to using **SLF4J** (`LoggerFactory.getLogger()`). This allows you to swap out logging implementations (like Log4J) without changing your logging code.
