---
**Author**: [skdishansachin](https://github.com/skdishansachin)
**Created**: 2024/10/6
**Last Updated**: N/A
**Contributors**: N/A
---

# Logging

- [Writing Log Messages](#writing-log-messages)
  - [Logging to the console](#logging-to-the-console)
  - [Logging to a file](#logging-to-a-file)
- [Logging Levels](#logging-levels)
- [Additional Configuration](#additional-configuration-optional)
- [Using a Logging Facade](#using-a-logging-facade)

Spring Boot provides logging capabilities using [**SLF4J**](https://www.slf4j.org/) (Simple Logging Facade for Java) and [**Logback**](https://logback.qos.ch/) (the default logging framework).

## Writing Log Messages

Spring Boot uses **SLF4J** with **Logback** as the default logging framework, so in most cases, you don't need to add any external dependencies. However, to ensure flexibility, make sure the following dependencies are in your `pom.xml`:

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

### Logging to the console

You can log messages from any class using the `Logger` interface. Here's an example of how to log data in a service or controller class:

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
- These logs will appear in your **console by default**.

### Logging to a File

By default, Spring Boot logs only to the **console**. To configure logging to write to a **file**, you can set this up in the `application.properties` or `application.yml` file.

#### Using `application.properties`

In your `src/main/resources/application.properties`, configure file-based logging like this:

```properties
logging.file.name=logs/application.log
logging.level.root=INFO
```

#### Using `application.yml`

Alternatively, if you're using YAML configuration (`src/main/resources/application.yml`), the setup would look like this:

```yaml
logging:
  file:
    name: logs/application.log
  level:
    root: INFO
```

**Optionally**, you can add more configurations like:

```properties
logging.level.com.yourpackage=DEBUG
logging.file.max-size=10MB
logging.file.max-history=10
```

Here is the full list of properties available for logging:

- `logging.file.name`: Specifies the name and location of the log file.
- `logging.file.max-size`: Maximum size of the log file before rolling over.
- `logging.file.max-history`: How many rolled-over log files to retain.

**What This Does:**

- Logs will be written to a file located at `logs/application.log`.
- The logging level is set to `INFO`, but you can adjust it to `DEBUG`, `WARN`, `ERROR`, etc.
- **Log rotation** ensures your log file doesn't grow endlessly (configurable with `max-size` and `max-history`).

## Logging Levels

Logging levels allow you to control the verbosity of your logs. Here are the commonly used logging levels:

| Level | Description                                                                  |
| ----- | ---------------------------------------------------------------------------- |
| TRACE | The most detailed level, typically used for fine-grained debugging.          |
| DEBUG | Provides debugging information, useful for development troubleshooting.      |
| INFO  | Logs important but general application events or statuses.                   |
| WARN  | Indicates a potential problem that isn't breaking, but might need attention. |
| ERROR | Logs serious issues that might cause the application to malfunction.         |

Logging levels are hierarchical. For example, if you set the logging level to `INFO`, it will capture all `INFO`, `WARN`, and `ERROR` logs, but ignore `DEBUG` and `TRACE`.

To set the logging level in your configuration file, add:

```properties
logging.level.root=DEBUG
```

This will apply the `DEBUG` level globally across your app. You can also adjust the logging level for specific packages:

```properties
logging.level.com.yourpackage=DEBUG
```

## Additional Configuration (Optional)

If you need more control over your logging, you can create a **custom Logback configuration file** (`logback-spring.xml`). This file allows for customization of log formats, rolling policies, and different appenders (console, file, etc.).

Place this file under `src/main/resources/logback-spring.xml`:

```xml
<configuration>

    <!-- Define log format (timestamp, log level, logger, and message) -->
    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>logs/application.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!-- Logs will rotate daily, keeping up to 30 days of logs -->
            <fileNamePattern>logs/application-%d{yyyy-MM-dd}.log</fileNamePattern>
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

**What This Does:**

- Logs will be written both to the **console** and a **file**.
- The log file will **rotate daily**, and the application will keep logs for up to 30 days.
- The log format is customizable, showing a **timestamp**, **log level**, **thread name**, and **message**.

## Using a Logging Facade

Spring Boot uses **SLF4J** as a facade for logging. This is useful in large projects because it decouples your application from a specific logging framework like **Logback** or **Log4J**. With SLF4J, you can switch to another logging framework without changing your logging code.

By sticking to SLF4J in your code (e.g., `LoggerFactory.getLogger()`), you keep the flexibility to swap in a different logging implementation simply by changing dependencies or configurations. This makes your codebase more modular and adaptable for future needs.
