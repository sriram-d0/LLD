

# Pub/Sub System

### Core idea
The pub/sub system is modeled as a set of classes that handle **topics, subscribers, publishers, and a broker**.  
The broker manages topics, publishers push messages into topics, and subscribers (observers) receive notifications when new messages are published.

---

### Key classes
- **[MessageBroker](guide://action?prefill=Tell%20me%20more%20about%3A%20MessageBroker)** → Manages topics and subscriptions.  
- **[Topic](guide://action?prefill=Tell%20me%20more%20about%3A%20Topic)** → Subject that holds subscribers and notifies them when a message is published.  
- **[Subscriber](guide://action?prefill=Tell%20me%20more%20about%3A%20Subscriber)** → Observer interface with `update()` method.  
- **[PrintSubscriber](guide://action?prefill=Tell%20me%20more%20about%3A%20PrintSubscriber)** → Concrete subscriber that prints messages.  
- **[Publisher](guide://action?prefill=Tell%20me%20more%20about%3A%20Publisher)** → Publishes messages into topics via the broker.  
- **[PubSubDemo](guide://action?prefill=Tell%20me%20more%20about%3A%20PubSubDemo)** → Demo class with `main()` method to run the system.

---

### Flow of usage
1. **Create topics** → Broker registers topics.  
2. **Subscribe** → Subscribers attach to topics.  
3. **Publish** → Publisher sends messages to topics.  
4. **Notify** → Topic notifies all subscribers.  
5. **Consume** → Subscribers process messages.

---

---

### Subscriber.java
```java
public interface Subscriber {
    void update(String topicName, String message);
}
```

---

### PrintSubscriber.java
```java
public class PrintSubscriber implements Subscriber {
    private final String name;

    public PrintSubscriber(String name) {
        this.name = name;
    }

    @Override
    public void update(String topicName, String message) {
        System.out.println("[" + name + "] received on " + topicName + ": " + message);
    }
}
```

---

### Topic.java
```java
import java.util.ArrayList;
import java.util.List;

public class Topic {
    private final String name;
    private final List<Subscriber> subscribers = new ArrayList<>();

    public Topic(String name) {
        this.name = name;
    }

    public String getName() { return name; }

    public void subscribe(Subscriber subscriber) {
        subscribers.add(subscriber);
    }

    public void unsubscribe(Subscriber subscriber) {
        subscribers.remove(subscriber);
    }

    public void publish(String message) {
        for (Subscriber s : subscribers) {
            s.update(name, message);
        }
    }
}
```

---

### MessageBroker.java
```java
import java.util.HashMap;
import java.util.Map;

public class MessageBroker {
    private final Map<String, Topic> topics = new HashMap<>();

    public Topic createTopic(String name) {
        Topic topic = new Topic(name);
        topics.put(name, topic);
        return topic;
    }

    public Topic getTopic(String name) {
        return topics.get(name);
    }
}
```

---

### Publisher.java
```java
public class Publisher {
    private final MessageBroker broker;

    public Publisher(MessageBroker broker) {
        this.broker = broker;
    }

    public void publish(String topicName, String message) {
        Topic topic = broker.getTopic(topicName);
        if (topic != null) {
            topic.publish(message);
        } else {
            System.out.println("Topic not found: " + topicName);
        }
    }
}
```

---

### PubSubDemo.java
```java
public class PubSubDemo {
    public static void main(String[] args) {
        MessageBroker broker = new MessageBroker();

        // Create topics
        broker.createTopic("orders");
        broker.createTopic("payments");

        // Subscribers
        Subscriber logger = new PrintSubscriber("Logger");
        Subscriber auditor = new PrintSubscriber("Auditor");

        // Subscribe
        broker.getTopic("orders").subscribe(logger);
        broker.getTopic("orders").subscribe(auditor);
        broker.getTopic("payments").subscribe(logger);

        // Publisher
        Publisher publisher = new Publisher(broker);

        // Publish messages
        publisher.publish("orders", "Order#123 created");
        publisher.publish("payments", "Payment of $500 received");
    }
}
```

---
