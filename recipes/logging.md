# Logging

- [Writing Log Messages](#writing-log-messages)
  - [Logging to the console](#logging-to-the-console)
  - [Logging to a file](#logging-to-a-file)
- [Logging Levels](#logging-levels)
- [Additional Configuration](#additional-configuration-optional)
- [Extra](#extra)
  - [Using a Logging Facade](#using-a-logging-facade)

Spring Boot provides logging capabilities using [**SLF4J**](https://www.slf4j.org/) (Simple Logging Facade for Java) and [**Logback**](https://logback.qos.ch/) (the default logging framework).

## Writing Log Messages

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

### Logging to the console

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
- These logs will appear in your **console by default**.

### Logging to a File

By default, Spring Boot logs only to the **console**. To configure logging to write to a **file**, you need to set this up in the `application.properties` or `application.yml` file.

#### Using `application.properties`

In your `src/main/resources/application.properties`, you can define file-based logging like this:

```
logging.file.name=logs/myapp.log
logging.level.root=INFO
```

#### Using `application.yml`

Alternatively, if you’re using YAML configuration (`src/main/resources/application.yml`), the setup will look like this:

```yaml
logging:
  file:
    name: logs/application.log
  level:
    root: INFO
```

**Optionaly** you can add configuration like

```
logging.level.com.yourpackage=DEBUG
logging.file.max-size=10MB
logging.file.max-history=10
```

Here is the full list for propoties avalable for loggin -

**What This Does:**

- Logs will be written to a file located at `logs/`application`.log`.
- The logging level is set to `INFO`, but you can adjust it to `DEBUG`, `WARN`, `ERROR`, etc.
- **Log rotation** ensures your log file doesn't grow endlessly (configurable with `max-size` and `max-history`).

## Logging Levels

Logging levels allow you to control the verbosity of your logs. Here are the commonly used logging levels:

| Level | Description                                                                  |
| ----- | ---------------------------------------------------------------------------- |
| TRACE | The most detailed level of logging, typically used for debugging purposes.   |
| DEBUG | Detailed information useful for debugging the application.                   |
| INFO  | General information about the application's progress.                        |
| WARN  | Indicates a potential issue that may cause problems in the future.           |
| ERROR | Indicates a serious error that may prevent the application from functioning. |

You can set the desired logging level in your application's configuration file (`application.properties` or `application.yml`) using the `logging.level` property. For example, to set the logging level to `DEBUG`, you can add the following line to your `application.properties` or `application.yml` file:

```
logging.level.root=DEBUG
```

This will enable logging at the `DEBUG` level for all classes in your application. You can also specify the logging level for specific packages or classes by using their respective names in the configuration file.

## Additional Configuration (Optional)

If you need more control over your logging, you can create a **custom Logback configuration file** (`logback-spring.xml`). This file allows for customization of log formats, rolling policies, and different appenders (console, file, etc.).

`src/main/resources/logback-spring.xml`

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

**What This Does:**

- Logs will be written both to the **console** and **file**.
- The log file will **rotate daily**, and the application will keep logs for up to 30 days.
- The log format is customizable, showing a **timestamp**, **log level**, **thread name**, and **message**.

## Extra

### Using a Logging Facade

Spring Boot uses **SLF4J** as a facade for logging. If you're working on a bigger project and want to make your logging framework interchangeable (i.e., replace Logback with another framework in the future), stick to using **SLF4J** (`LoggerFactory.getLogger()`). This allows you to swap out logging implementations (like Log4J) without changing your logging code.
