# Logging Framework

## 🎯 Core Classes in the LLD

- **LogLevel**: Enum for levels (INFO, DEBUG, ERROR, etc.).  
- **LogRecord**: Immutable object holding a single log event.  
- **Formatter**: Interface for formatting log records.  
- **Filter**: Interface for filtering log records.  
- **Appender**: Interface for destinations (console, file, etc.).  
- **Logger**: Main API class used by developers.  
- **LogManager**: Singleton managing loggers and appenders.

---

## 🧩 Class-by-Class Skeleton

### LogLevel.java
```java
public enum LogLevel {
    TRACE, DEBUG, INFO, WARN, ERROR;
}
```

---

### LogRecord.java
```java
import java.time.Instant;

public class LogRecord {
    private final Instant timestamp;
    private final String loggerName;
    private final LogLevel level;
    private final String message;

    public LogRecord(String loggerName, LogLevel level, String message) {
        this.timestamp = Instant.now();
        this.loggerName = loggerName;
        this.level = level;
        this.message = message;
    }

    public Instant getTimestamp() { return timestamp; }
    public String getLoggerName() { return loggerName; }
    public LogLevel getLevel() { return level; }
    public String getMessage() { return message; }
}
```

---

### Formatter.java
```java
public interface Formatter {
    String format(LogRecord record);
}
```

#### SimpleFormatter.java
```java
public class SimpleFormatter implements Formatter {
    @Override
    public String format(LogRecord record) {
        return record.getTimestamp() + " [" + record.getLevel() + "] "
               + record.getLoggerName() + " - " + record.getMessage();
    }
}
```

---

### Filter.java
```java
public interface Filter {
    boolean test(LogRecord record);
}
```

#### LevelFilter.java
```java
public class LevelFilter implements Filter {
    private final LogLevel minLevel;

    public LevelFilter(LogLevel minLevel) {
        this.minLevel = minLevel;
    }

    @Override
    public boolean test(LogRecord record) {
        return record.getLevel().ordinal() >= minLevel.ordinal();
    }
}
```

---

### Appender.java
```java
public interface Appender {
    void append(LogRecord record);
}
```

#### ConsoleAppender.java
```java
public class ConsoleAppender implements Appender {
    private final Formatter formatter;

    public ConsoleAppender(Formatter formatter) {
        this.formatter = formatter;
    }

    @Override
    public void append(LogRecord record) {
        System.out.println(formatter.format(record));
    }
}
```

#### FileAppender.java
```java
import java.io.*;

public class FileAppender implements Appender {
    private final Formatter formatter;
    private final PrintWriter writer;

    public FileAppender(String filePath, Formatter formatter) throws IOException {
        this.formatter = formatter;
        this.writer = new PrintWriter(new FileWriter(filePath, true));
    }

    @Override
    public void append(LogRecord record) {
        writer.println(formatter.format(record));
        writer.flush();
    }
}
```

---

### Logger.java
```java
public class Logger {
    private final String name;
    private final LogManager manager;

    Logger(String name, LogManager manager) {
        this.name = name;
        this.manager = manager;
    }

    public void log(LogLevel level, String message) {
        LogRecord record = new LogRecord(name, level, message);
        manager.dispatch(record);
    }

    public void info(String msg) { log(LogLevel.INFO, msg); }
    public void debug(String msg) { log(LogLevel.DEBUG, msg); }
    public void error(String msg) { log(LogLevel.ERROR, msg); }
}
```

---

### LogManager.java
```java
import java.util.*;

public class LogManager {
    private static final LogManager INSTANCE = new LogManager();
    private final List<Appender> appenders = new ArrayList<>();

    private LogManager() {}

    public static LogManager getInstance() {
        return INSTANCE;
    }

    public Logger getLogger(String name) {
        return new Logger(name, this);
    }

    public void addAppender(Appender appender) {
        appenders.add(appender);
    }

    void dispatch(LogRecord record) {
        for (Appender appender : appenders) {
            appender.append(record);
        }
    }
}
```

---

## 🚀 Example Usage

```java
public class Demo {
    public static void main(String[] args) throws Exception {
        LogManager manager = LogManager.getInstance();
        manager.addAppender(new ConsoleAppender(new SimpleFormatter()));
        manager.addAppender(new FileAppender("app.log", new SimpleFormatter()));

        Logger logger = manager.getLogger("MyApp");
        logger.info("Application started");
        logger.debug("Debugging details here");
        logger.error("Something went wrong!");
    }
}
```

---
