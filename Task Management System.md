

# Task Management System

---

### 🔄 Task Management Flow Overview

1. **User Input via UI**
   - User provides: `userId`, `projectId`, `title`, `description`, `priority`, `dueDate`, and optional tags.
   - This data is wrapped into a `TaskRequest`.

2. **Task Service Receives Request**
   - `TaskService.createTask(request)` is called.
   - A `Task` object is created with default status `OPEN`.

3. **Task Operations**
   - Users can update status, assign tasks, add comments, and manage tags.
   - Each operation is validated and recorded.

4. **Audit Logging**
   - `AuditService.record()` stores an `AuditLogEntry` for every change.
   - Provides full history of task modifications.

5. **UI Displays Result**
   - Shows task ID, status, assignee, due date, and comments.
   - Offers options to view all tasks, filter by project/user, or search by tags.

---

### 📦 Task Management Class Responsibilities

| **Class Name**        | **Primary Responsibility**                                                                 |
|------------------------|---------------------------------------------------------------------------------------------|
| **TaskRequest**        | Holds input data for creating a task (title, description, priority, due date, etc.)         |
| **Task**               | Represents a task entity with status, assignee, tags, comments                              |
| **Status**             | Enum defining task states: OPEN, IN_PROGRESS, BLOCKED, DONE                                 |
| **Priority**           | Enum defining task priority: LOW, MEDIUM, HIGH, CRITICAL                                    |
| **Comment**            | Represents a user comment with author and timestamp                                         |
| **AuditLogEntry**      | Records changes made to tasks (status updates, comments, assignments)                       |
| **TaskRepository**     | Stores tasks in memory; supports CRUD operations                                            |
| **AuditService**       | Stores and retrieves audit logs                                                             |
| **TaskService**        | Orchestrates task creation, updates, assignments, and audit logging                         |
| **TaskUI**             | Console-based user interface for creating tasks and viewing/updating them                   |

---

## 🧱 Core Classes

### `TaskRequest.java`
```java
import java.time.LocalDate;
import java.util.Set;

public class TaskRequest {
    private String userId;
    private String projectId;
    private String title;
    private String description;
    private Priority priority;
    private LocalDate dueDate;
    private Set<String> tags;

    public TaskRequest(String userId, String projectId, String title, String description,
                       Priority priority, LocalDate dueDate, Set<String> tags) {
        this.userId = userId;
        this.projectId = projectId;
        this.title = title;
        this.description = description;
        this.priority = priority;
        this.dueDate = dueDate;
        this.tags = tags;
    }

    public String getUserId() { return userId; }
    public String getProjectId() { return projectId; }
    public String getTitle() { return title; }
    public String getDescription() { return description; }
    public Priority getPriority() { return priority; }
    public LocalDate getDueDate() { return dueDate; }
    public Set<String> getTags() { return tags; }
}
```

---

### `Task.java`
```java
import java.time.LocalDate;
import java.time.Instant;
import java.util.*;

public class Task {
    private String id;
    private String title;
    private String description;
    private Status status;
    private Priority priority;
    private LocalDate dueDate;
    private String projectId;
    private String assigneeUserId;
    private String reporterUserId;
    private Set<String> tags = new HashSet<>();
    private List<Comment> comments = new ArrayList<>();
    private Instant createdAt;
    private Instant updatedAt;

    public Task(String id, String title, String description, Priority priority,
                LocalDate dueDate, String projectId, String reporterUserId) {
        this.id = id;
        this.title = title;
        this.description = description;
        this.status = Status.OPEN;
        this.priority = priority;
        this.dueDate = dueDate;
        this.projectId = projectId;
        this.reporterUserId = reporterUserId;
        this.createdAt = Instant.now();
        this.updatedAt = this.createdAt;
    }

    public String getId() { return id; }
    public String getTitle() { return title; }
    public String getDescription() { return description; }
    public Status getStatus() { return status; }
    public Priority getPriority() { return priority; }
    public LocalDate getDueDate() { return dueDate; }
    public String getProjectId() { return projectId; }
    public String getAssigneeUserId() { return assigneeUserId; }
    public String getReporterUserId() { return reporterUserId; }
    public Set<String> getTags() { return tags; }
    public List<Comment> getComments() { return comments; }
    public Instant getCreatedAt() { return createdAt; }
    public Instant getUpdatedAt() { return updatedAt; }

    public void setStatus(Status status) { this.status = status; touch(); }
    public void setAssigneeUserId(String assigneeUserId) { this.assigneeUserId = assigneeUserId; touch(); }
    public void addTag(String tag) { tags.add(tag); touch(); }
    public void addComment(Comment comment) { comments.add(comment); touch(); }

    private void touch() { this.updatedAt = Instant.now(); }
}
```

---

### `Status.java`
```java
public enum Status {
    OPEN, IN_PROGRESS, BLOCKED, DONE
}
```

---

### `Priority.java`
```java
public enum Priority {
    LOW, MEDIUM, HIGH, CRITICAL
}
```

---

### `Comment.java`
```java
import java.time.Instant;

public class Comment {
    private String authorUserId;
    private String text;
    private Instant createdAt;

    public Comment(String authorUserId, String text) {
        this.authorUserId = authorUserId;
        this.text = text;
        this.createdAt = Instant.now();
    }

    public String getAuthorUserId() { return authorUserId; }
    public String getText() { return text; }
    public Instant getCreatedAt() { return createdAt; }
}
```

---

### `AuditLogEntry.java`
```java
import java.time.Instant;

public class AuditLogEntry {
    private String entityId;
    private String actorUserId;
    private String action;
    private String details;
    private Instant timestamp;

    public AuditLogEntry(String entityId, String actorUserId, String action, String details) {
        this.entityId = entityId;
        this.actorUserId = actorUserId;
        this.action = action;
        this.details = details;
        this.timestamp = Instant.now();
    }

    public String getEntityId() { return entityId; }
    public String getActorUserId() { return actorUserId; }
    public String getAction() { return action; }
    public String getDetails() { return details; }
    public Instant getTimestamp() { return timestamp; }
}
```

---

### `TaskRepository.java`
```java
import java.util.*;

public class TaskRepository {
    private Map<String, Task> tasks = new HashMap<>();

    public Task save(Task task) {
        tasks.put(task.getId(), task);
        return task;
    }

    public Optional<Task> findById(String id) {
        return Optional.ofNullable(tasks.get(id));
    }

    public List<Task> findAll() {
        return new ArrayList<>(tasks.values());
    }

    public void delete(String id) {
        tasks.remove(id);
    }
}
```

---

### `AuditService.java`
```java
import java.util.*;

public class AuditService {
    private List<AuditLogEntry> logs = new ArrayList<>();

    public void record(AuditLogEntry entry) {
        logs.add(entry);
    }

    public List<AuditLogEntry> getAll() {
        return logs;
    }
}
```

---


## 🧱 `TaskService.java`

```java
import java.time.LocalDate;
import java.util.*;

public class TaskService {
    private TaskRepository repo = new TaskRepository();
    private AuditService audit = new AuditService();

    // Create a new task
    public Task createTask(TaskRequest request) {
        String id = UUID.randomUUID().toString();
        Task task = new Task(id, request.getTitle(), request.getDescription(),
                request.getPriority(), request.getDueDate(),
                request.getProjectId(), request.getUserId());

        if (request.getTags() != null) {
            for (String tag : request.getTags()) {
                task.addTag(tag);
            }
        }

        repo.save(task);
        audit.record(new AuditLogEntry(id, request.getUserId(), "TASK_CREATED", request.getTitle()));
        return task;
    }

    // Assign a task
    public void assignTask(String taskId, String assigneeUserId, String actorUserId) {
        Task task = repo.findById(taskId).orElseThrow(() -> new NoSuchElementException("Task not found"));
        task.setAssigneeUserId(assigneeUserId);
        repo.save(task);
        audit.record(new AuditLogEntry(taskId, actorUserId, "TASK_ASSIGNED", assigneeUserId));
    }

    // Update status
    public void updateStatus(String taskId, Status newStatus, String actorUserId) {
        Task task = repo.findById(taskId).orElseThrow(() -> new NoSuchElementException("Task not found"));
        task.setStatus(newStatus);
        repo.save(task);
        audit.record(new AuditLogEntry(taskId, actorUserId, "STATUS_CHANGED", newStatus.name()));
    }

    // Update priority
    public void updatePriority(String taskId, Priority newPriority, String actorUserId) {
        Task task = repo.findById(taskId).orElseThrow(() -> new NoSuchElementException("Task not found"));
        task.setPriority(newPriority);
        repo.save(task);
        audit.record(new AuditLogEntry(taskId, actorUserId, "PRIORITY_CHANGED", newPriority.name()));
    }

    // Update due date
    public void updateDueDate(String taskId, LocalDate dueDate, String actorUserId) {
        Task task = repo.findById(taskId).orElseThrow(() -> new NoSuchElementException("Task not found"));
        task.setDueDate(dueDate);
        repo.save(task);
        audit.record(new AuditLogEntry(taskId, actorUserId, "DUEDATE_CHANGED", String.valueOf(dueDate)));
    }

    // Add a comment
    public void addComment(String taskId, String userId, String text) {
        Task task = repo.findById(taskId).orElseThrow(() -> new NoSuchElementException("Task not found"));
        Comment comment = new Comment(userId, text);
        task.addComment(comment);
        repo.save(task);
        audit.record(new AuditLogEntry(taskId, userId, "COMMENT_ADDED", text));
    }

    // Add a tag
    public void addTag(String taskId, String tag, String actorUserId) {
        Task task = repo.findById(taskId).orElseThrow(() -> new NoSuchElementException("Task not found"));
        task.addTag(tag);
        repo.save(task);
        audit.record(new AuditLogEntry(taskId, actorUserId, "TAG_ADDED", tag));
    }

    // Delete a task
    public void deleteTask(String taskId, String actorUserId) {
        Task task = repo.findById(taskId).orElseThrow(() -> new NoSuchElementException("Task not found"));
        repo.delete(taskId);
        audit.record(new AuditLogEntry(taskId, actorUserId, "TASK_DELETED", task.getTitle()));
    }

    // Retrieve a task
    public Optional<Task> getTask(String id) {
        return repo.findById(id);
    }

    // List all tasks
    public List<Task> listAll() {
        return repo.findAll();
    }

    // View audit logs
    public List<AuditLogEntry> getAuditLogs() {
        return audit.getAll();
    }
}
```

---

## 🧱 `TaskUI.java`

```java
import java.time.LocalDate;
import java.util.*;

public class TaskUI {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        TaskService service = new TaskService();

        while (true) {
            System.out.println("\n1. Create Task\n2. Assign Task\n3. Update Status\n4. Add Comment\n5. Add Tag\n6. View All Tasks\n7. View Audit Logs\n8. Delete Task\n9. Exit");
            int choice = Integer.parseInt(scanner.nextLine());

            if (choice == 1) {
                System.out.println("Enter User ID:");
                String userId = scanner.nextLine();

                System.out.println("Enter Project ID:");
                String projectId = scanner.nextLine();

                System.out.println("Enter Title:");
                String title = scanner.nextLine();

                System.out.println("Enter Description:");
                String description = scanner.nextLine();

                System.out.println("Enter Priority (LOW/MEDIUM/HIGH/CRITICAL):");
                Priority priority = Priority.valueOf(scanner.nextLine().toUpperCase());

                System.out.println("Enter Due Date (YYYY-MM-DD):");
                LocalDate dueDate = LocalDate.parse(scanner.nextLine());

                Set<String> tags = new HashSet<>();
                System.out.println("Enter tags (comma separated):");
                String tagLine = scanner.nextLine();
                if (!tagLine.isEmpty()) {
                    tags.addAll(Arrays.asList(tagLine.split(",")));
                }

                TaskRequest request = new TaskRequest(userId, projectId, title, description, priority, dueDate, tags);
                Task task = service.createTask(request);
                System.out.println("Task created with ID: " + task.getId());

            } else if (choice == 2) {
                System.out.println("Enter Task ID:");
                String taskId = scanner.nextLine();
                System.out.println("Enter Assignee User ID:");
                String assignee = scanner.nextLine();
                System.out.println("Enter Actor User ID:");
                String actor = scanner.nextLine();
                service.assignTask(taskId, assignee, actor);
                System.out.println("Task assigned.");

            } else if (choice == 3) {
                System.out.println("Enter Task ID:");
                String taskId = scanner.nextLine();
                System.out.println("Enter New Status (OPEN/IN_PROGRESS/BLOCKED/DONE):");
                Status status = Status.valueOf(scanner.nextLine().toUpperCase());
                System.out.println("Enter Actor User ID:");
                String actor = scanner.nextLine();
                service.updateStatus(taskId, status, actor);
                System.out.println("Status updated.");

            } else if (choice == 4) {
                System.out.println("Enter Task ID:");
                String taskId = scanner.nextLine();
                System.out.println("Enter User ID:");
                String userId = scanner.nextLine();
                System.out.println("Enter Comment:");
                String text = scanner.nextLine();
                service.addComment(taskId, userId, text);
                System.out.println("Comment added.");

            } else if (choice == 5) {
                System.out.println("Enter Task ID:");
                String taskId = scanner.nextLine();
                System.out.println("Enter Tag:");
                String tag = scanner.nextLine();
                System.out.println("Enter Actor User ID:");
                String actor = scanner.nextLine();
                service.addTag(taskId, tag, actor);
                System.out.println("Tag added.");

            } else if (choice == 6) {
                List<Task> tasks = service.listAll();
                for (Task t : tasks) {
                    System.out.println(t.getId() + " | " + t.getTitle() + " | " + t.getStatus() + " | Assignee: " + t.getAssigneeUserId());
                }

            } else if (choice == 7) {
                List<AuditLogEntry> logs = service.getAuditLogs();
                for (AuditLogEntry log : logs) {
                    System.out.println(log.getEntityId() + " | " + log.getAction() + " | " + log.getDetails() + " | by " + log.getActorUserId() + " at " + log.getTimestamp());
                }

            } else if (choice == 8) {
                System.out.println("Enter Task ID:");
                String taskId = scanner.nextLine();
                System.out.println("Enter Actor User ID:");
                String actor = scanner.nextLine();
                service.deleteTask(taskId, actor);
                System.out.println("Task deleted.");

            } else if (choice == 9) {
                break;
            }
        }

        scanner.close();
    }
}
```

---

