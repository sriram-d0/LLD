


## 🧩 ATM 


## 🧠 ATMContext & State Pattern
- `ATMContext` holds the current state and account info.
- States (`IdleState`, `CardInsertedState`, `AuthenticatedState`) define ATM behavior at each step.
- Each state receives `ATMContext` to:
  - Access account data
  - Transition to the next state using `context.setState(...)`

---

## 🔗 Chain of Responsibility Pattern
- Used for handling operations: `WithdrawHandler`, `DepositHandler`, `BalanceEnquiryHandler`
- Each handler checks if it can process the request; if not, passes to the next.
- Ensures clean separation of logic for each operation.

---

## 🔄 Flow Summary
1. **IdleState** → User inserts card → moves to `CardInsertedState`
2. **CardInsertedState** → User enters PIN → if correct → `AuthenticatedState`
3. **AuthenticatedState** → User selects one operation → handled via chain → resets to `IdleState`

---


## 🧱 Core Classes

### 1. `Account.java`
```java
public class Account {
    private String cardNumber;
    private String pin;
    private double balance;

    public Account(String cardNumber, String pin, double balance) {
        this.cardNumber = cardNumber;
        this.pin = pin;
        this.balance = balance;
    }

    public boolean authenticate(String inputPin) {
        return this.pin.equals(inputPin);
    }

    public double getBalance() {
        return balance;
    }

    public void deposit(double amount) {
        balance += amount;
    }

    public boolean withdraw(double amount) {
        if (amount <= balance) {
            balance -= amount;
            return true;
        }
        return false;
    }

    public String getCardNumber() {
        return cardNumber;
    }
}
```

---

### 2. `ATMContext.java`
```java
public class ATMContext {
    private ATMState currentState;
    private Account currentAccount;

    public ATMContext() {
        this.currentState = new IdleState();
    }

    public void setState(ATMState state) {
        this.currentState = state;
    }

    public void insertCard(Account account) {
        this.currentAccount = account;
        currentState.insertCard(this);
    }

    public void enterPin(String pin) {
        currentState.enterPin(this, pin);
    }

    public void selectOperation(String operation) {
        currentState.selectOperation(this, operation);
    }

    public Account getAccount() {
        return currentAccount;
    }
}
```

---

### 3. `ATMState.java`
```java
public interface ATMState {
    void insertCard(ATMContext context);
    void enterPin(ATMContext context, String pin);
    void selectOperation(ATMContext context, String operation);
}
```

---

### 4. `IdleState.java`
```java
public class IdleState implements ATMState {
    public void insertCard(ATMContext context) {
        System.out.println("Card inserted.");
        context.setState(new CardInsertedState());
    }

    public void enterPin(ATMContext context, String pin) {
        System.out.println("Insert card first.");
    }

    public void selectOperation(ATMContext context, String operation) {
        System.out.println("Insert card and authenticate first.");
    }
}
```

---

### 5. `CardInsertedState.java`
```java
public class CardInsertedState implements ATMState {
    public void insertCard(ATMContext context) {
        System.out.println("Card already inserted.");
    }

    public void enterPin(ATMContext context, String pin) {
        if (context.getAccount().authenticate(pin)) {
            System.out.println("PIN correct.");
            context.setState(new AuthenticatedState());
        } else {
            System.out.println("Incorrect PIN. Try again.");
        }
    }

    public void selectOperation(ATMContext context, String operation) {
        System.out.println("Authenticate first.");
    }
}
```

---

### 6. `AuthenticatedState.java`
```java
public class AuthenticatedState implements ATMState {
    public void insertCard(ATMContext context) {
        System.out.println("Card already inserted.");
    }

    public void enterPin(ATMContext context, String pin) {
        System.out.println("Already authenticated.");
    }

    public void selectOperation(ATMContext context, String operation) {
        OperationHandler handler = new WithdrawHandler();
        handler.setNext(new DepositHandler())
               .setNext(new BalanceEnquiryHandler());

        handler.handle(context, operation);
        context.setState(new IdleState()); // Reset after one operation
    }
}
```

---

## 🔗 Chain of Responsibility

### 7. `OperationHandler.java`
```java
public abstract class OperationHandler {
    protected OperationHandler next;

    public OperationHandler setNext(OperationHandler nextHandler) {
        this.next = nextHandler;
        return nextHandler;
    }

    public void handle(ATMContext context, String operation) {
        if (canHandle(operation)) {
            process(context);
        } else if (next != null) {
            next.handle(context, operation);
        } else {
            System.out.println("Invalid operation.");
        }
    }

    protected abstract boolean canHandle(String operation);
    protected abstract void process(ATMContext context);
}
```

---

### 8. `WithdrawHandler.java`
```java
import java.util.Scanner;

public class WithdrawHandler extends OperationHandler {
    protected boolean canHandle(String operation) {
        return operation.equalsIgnoreCase("withdraw");
    }

    protected void process(ATMContext context) {
        Scanner sc = new Scanner(System.in);
        System.out.print("Enter amount to withdraw: ");
        double amount = sc.nextDouble();
        if (context.getAccount().withdraw(amount)) {
            System.out.println("Withdrawal successful. New balance: " + context.getAccount().getBalance());
        } else {
            System.out.println("Insufficient balance.");
        }
    }
}
```

---

### 9. `DepositHandler.java`
```java
import java.util.Scanner;

public class DepositHandler extends OperationHandler {
    protected boolean canHandle(String operation) {
        return operation.equalsIgnoreCase("deposit");
    }

    protected void process(ATMContext context) {
        Scanner sc = new Scanner(System.in);
        System.out.print("Enter amount to deposit: ");
        double amount = sc.nextDouble();
        context.getAccount().deposit(amount);
        System.out.println("Deposit successful. New balance: " + context.getAccount().getBalance());
    }
}
```

---

### 10. `BalanceEnquiryHandler.java`
```java
public class BalanceEnquiryHandler extends OperationHandler {
    protected boolean canHandle(String operation) {
        return operation.equalsIgnoreCase("balance");
    }

    protected void process(ATMContext context) {
        System.out.println("Current balance: " + context.getAccount().getBalance());
    }
}
```

---

## 🚀 Demo Runner

### 11. `ATMRunner.java`
```java
import java.util.Scanner;

public class ATMRunner {
    public static void main(String[] args) {
        Account account = new Account("1234567890", "4321", 1000.0);
        ATMContext atm = new ATMContext();
        Scanner sc = new Scanner(System.in);

        atm.insertCard(account);

        System.out.print("Enter PIN: ");
        String pin = sc.nextLine();
        atm.enterPin(pin);

        System.out.println("Select operation: withdraw / deposit / balance");
        String operation = sc.nextLine();
        atm.selectOperation(operation);
    }
}
```

