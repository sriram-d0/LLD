

## 💸 Splitwise LLD in Java

A modular, extensible Low-Level Design (LLD) of a Splitwise-like expense-sharing system implemented in Java. This project supports friend management, group-based expense tracking, and multiple splitting strategies (equal, unequal, percentage).

---

## 📦 Features

- ✅ Add and manage friends
- ✅ Create and manage groups
- ✅ Add/remove users in groups
- ✅ Add expenses inside or outside groups
- ✅ Split expenses equally, unequally, or by percentage
- ✅ Maintain per-user balance sheets

---

## 🧱 Object-Oriented Design

### Core Classes

| Class         | Responsibility |
|---------------|----------------|
| `User`        | Represents a user and their balance sheet |
| `Group`       | Represents a group of users and their expenses |
| `Expense`     | Represents an expense and applies the split logic |
| `Splitwise`   | Main orchestrator for users, groups, and expenses |
| `SplitStrategy` | Interface for different splitting strategies |
| `EqualSplit`  | Splits expense equally |
| `UnequalSplit`| Splits expense by fixed amounts |
| `PercentSplit`| Splits expense by percentage |

---




### Project Structure

```
src/
├── model/
│   ├── User.java
│   ├── Group.java
│   ├── Expense.java
│   └── Splitwise.java
├── strategy/
│   ├── SplitStrategy.java
│   ├── EqualSplit.java
│   ├── UnequalSplit.java
│   └── PercentSplit.java
└── Main.java
```
---

## 🧩 Core Classes and Interfaces

### 1. `User.java`
```java
public class User {
    private String userId;
    private String name;
    private Map<String, Double> balanceSheet; // friendId -> balance

    public User(String userId, String name) {
        this.userId = userId;
        this.name = name;
        this.balanceSheet = new HashMap<>();
    }

    public String getUserId() { return userId; }
    public String getName() { return name; }

    public Map<String, Double> getBalanceSheet() { return balanceSheet; }

    public void updateBalance(String friendId, double amount) {
        balanceSheet.put(friendId, balanceSheet.getOrDefault(friendId, 0.0) + amount);
    }
}
```

---

### 2. `Group.java`
```java
public class Group {
    private String groupId;
    private String name;
    private List<User> members;
    private List<Expense> expenses;

    public Group(String groupId, String name) {
        this.groupId = groupId;
        this.name = name;
        this.members = new ArrayList<>();
        this.expenses = new ArrayList<>();
    }

    public void addMember(User user) {
        if (!members.contains(user)) members.add(user);
    }

    public void removeMember(User user) {
        members.remove(user);
    }

    public List<User> getMembers() { return members; }
    public List<Expense> getExpenses() { return expenses; }

    public void addExpense(Expense expense) {
        expenses.add(expense);
    }
}
```

---

### 3. `SplitStrategy.java` (Interface)
```java
public interface SplitStrategy {
    Map<User, Double> calculateSplits(double totalAmount, List<User> users, List<Double> values);
}
```

---

### 4. `EqualSplit.java`
```java
public class EqualSplit implements SplitStrategy {
    public Map<User, Double> calculateSplits(double totalAmount, List<User> users, List<Double> values) {
        Map<User, Double> splits = new HashMap<>();
        double splitAmount = totalAmount / users.size();
        for (User user : users) {
            splits.put(user, splitAmount);
        }
        return splits;
    }
}
```

---

### 5. `UnequalSplit.java`
```java
public class UnequalSplit implements SplitStrategy {
    public Map<User, Double> calculateSplits(double totalAmount, List<User> users, List<Double> values) {
        Map<User, Double> splits = new HashMap<>();
        for (int i = 0; i < users.size(); i++) {
            splits.put(users.get(i), values.get(i));
        }
        return splits;
    }
}
```

---

### 6. `PercentSplit.java`
```java
public class PercentSplit implements SplitStrategy {
    public Map<User, Double> calculateSplits(double totalAmount, List<User> users, List<Double> percentages) {
        Map<User, Double> splits = new HashMap<>();
        for (int i = 0; i < users.size(); i++) {
            splits.put(users.get(i), totalAmount * percentages.get(i) / 100.0);
        }
        return splits;
    }
}
```

---

### 7. `Expense.java`
```java
public class Expense {
    private String expenseId;
    private User paidBy;
    private double amount;
    private List<User> involvedUsers;
    private SplitStrategy splitStrategy;
    private List<Double> values; // used for unequal or percent splits

    public Expense(String expenseId, User paidBy, double amount, List<User> involvedUsers,
                   SplitStrategy splitStrategy, List<Double> values) {
        this.expenseId = expenseId;
        this.paidBy = paidBy;
        this.amount = amount;
        this.involvedUsers = involvedUsers;
        this.splitStrategy = splitStrategy;
        this.values = values;
        applySplit();
    }

    private void applySplit() {
        Map<User, Double> splits = splitStrategy.calculateSplits(amount, involvedUsers, values);
        for (Map.Entry<User, Double> entry : splits.entrySet()) {
            User user = entry.getKey();
            double share = entry.getValue();
            if (!user.getUserId().equals(paidBy.getUserId())) {
                user.updateBalance(paidBy.getUserId(), -share);
                paidBy.updateBalance(user.getUserId(), share);
            }
        }
    }
}
```

---

### 8. `Splitwise.java`
```java
public class Splitwise {
    private Map<String, User> users;
    private Map<String, Group> groups;

    public Splitwise() {
        users = new HashMap<>();
        groups = new HashMap<>();
    }

    public void addUser(String userId, String name) {
        users.put(userId, new User(userId, name));
    }

    public void addFriend(String userId1, String userId2) {
        users.get(userId1).updateBalance(userId2, 0.0);
        users.get(userId2).updateBalance(userId1, 0.0);
    }

    public void createGroup(String groupId, String name) {
        groups.put(groupId, new Group(groupId, name));
    }

    public void addUserToGroup(String groupId, String userId) {
        Group group = groups.get(groupId);
        User user = users.get(userId);
        group.addMember(user);
    }

    public void addExpense(String expenseId, String paidById, double amount, List<String> involvedUserIds,
                           SplitStrategy strategy, List<Double> values, String groupId) {
        List<User> involvedUsers = new ArrayList<>();
        for (String id : involvedUserIds) {
            involvedUsers.add(users.get(id));
        }
        Expense expense = new Expense(expenseId, users.get(paidById), amount, involvedUsers, strategy, values);
        if (groupId != null && groups.containsKey(groupId)) {
            groups.get(groupId).addExpense(expense);
        }
    }

    public void showBalanceSheet(String userId) {
        User user = users.get(userId);
        System.out.println("Balance Sheet for " + user.getName() + ":");
        for (Map.Entry<String, Double> entry : user.getBalanceSheet().entrySet()) {
            String friendId = entry.getKey();
            double balance = entry.getValue();
            if (balance > 0) {
                System.out.println("Gets ₹" + balance + " from " + users.get(friendId).getName());
            } else if (balance < 0) {
                System.out.println("Owes ₹" + (-balance) + " to " + users.get(friendId).getName());
            }
        }
    }
}
```

---

## ✅ Example Usage
```java
Splitwise app = new Splitwise();
app.addUser("u1", "Sriram");
app.addUser("u2", "Rahul");
app.addUser("u3", "Priya");

app.addFriend("u1", "u2");
app.addFriend("u1", "u3");

app.createGroup("g1", "Trip");
app.addUserToGroup("g1", "u1");
app.addUserToGroup("g1", "u2");
app.addUserToGroup("g1", "u3");

List<String> members = Arrays.asList("u1", "u2", "u3");
SplitStrategy strategy = new EqualSplit();
app.addExpense("e1", "u1", 3000, members, strategy, null, "g1");

app.showBalanceSheet("u1");
app.showBalanceSheet("u2");
app.showBalanceSheet("u3");
```






---

