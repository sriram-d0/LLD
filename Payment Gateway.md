## Payment Gateway



### 🔄 Payment Gateway Flow Overview

1. **User Input via UI**
   - User enters: `userId`, `amount`, `currency`, `payment method`, and method-specific details (e.g., card number, UPI ID).
   - This data is wrapped into a `PaymentRequest`.

2. **Payment Gateway Receives Request**
   - `PaymentGateway.processPayment(request)` is called.
   - It selects the correct `PaymentStrategy` based on `PaymentMethod`.

3. **Strategy Executes Payment**
   - Strategy (e.g., `CreditCardStrategy`) uses `BankAPIClient` to simulate external payment.
   - A `PaymentResponse` is generated with `transactionId`, `success`, and `message`.

4. **Transaction Is Recorded**
   - `TransactionService.save()` stores a `TransactionRecord` containing both request and response.
   - This enables full transaction history and filtering by user.

5. **UI Displays Result**
   - Shows transaction ID, status, and message.
   - Offers options to view all transactions or filter by user.

---


### 📦 Payment Gateway Class Responsibilities

| **Class Name**             | **Primary Responsibility**                                                                 |
|---------------------------|---------------------------------------------------------------------------------------------|
| **PaymentRequest**         | Holds user ID, amount, currency, payment method, and metadata for a transaction            |
| **PaymentResponse**        | Contains transaction result: success status, transaction ID, and message                   |
| **PaymentMethod**          | Enum defining supported payment types: CREDIT_CARD, UPI, WALLET                            |
| **PaymentStrategy**        | Interface for implementing different payment method strategies                             |
| **CreditCardStrategy**     | Processes credit card payments using metadata like card number and CVV                    |
| **UPIStrategy**            | Processes UPI payments using UPI ID                                                        |
| **WalletStrategy**         | Processes wallet payments using wallet ID                                                  |
| **BankAPIClient**          | Simulates external bank API calls for charging cards, sending UPI, or using wallets        |
| **TransactionRecord**      | Combines `PaymentRequest` and `PaymentResponse` into a single transaction history entry    |
| **TransactionService**     | Stores and retrieves transaction history; supports filtering by user ID                   |
| **PaymentGateway**         | Orchestrates payment flow by selecting strategy and saving transaction records             |
| **PaymentUI**              | Console-based user interface for making payments and viewing transaction history           |

---



## 🧱 Core Classes

### `PaymentRequest.java`
```java
import java.util.Map;

public class PaymentRequest {
    private String userId;
    private double amount;
    private String currency;
    private PaymentMethod method;
    private Map<String, String> metadata;

    public PaymentRequest(String userId, double amount, String currency, PaymentMethod method, Map<String, String> metadata) {
        this.userId = userId;
        this.amount = amount;
        this.currency = currency;
        this.method = method;
        this.metadata = metadata;
    }

    public String getUserId() { return userId; }
    public double getAmount() { return amount; }
    public String getCurrency() { return currency; }
    public PaymentMethod getMethod() { return method; }
    public Map<String, String> getMetadata() { return metadata; }
}
```

---

### `PaymentResponse.java`
```java
public class PaymentResponse {
    private boolean success;
    private String transactionId;
    private String message;

    public PaymentResponse(boolean success, String transactionId, String message) {
        this.success = success;
        this.transactionId = transactionId;
        this.message = message;
    }

    public boolean isSuccess() { return success; }
    public String getTransactionId() { return transactionId; }
    public String getMessage() { return message; }
}
```

---

### `PaymentMethod.java`
```java
public enum PaymentMethod {
    CREDIT_CARD, UPI, WALLET
}
```

---

### `PaymentStrategy.java`
```java
public interface PaymentStrategy {
    PaymentResponse process(PaymentRequest request);
}
```

---

### `CreditCardStrategy.java`
```java
import java.util.UUID;

public class CreditCardStrategy implements PaymentStrategy {
    private BankAPIClient bankClient = new BankAPIClient();

    public PaymentResponse process(PaymentRequest request) {
        String cardNumber = request.getMetadata().get("cardNumber");
        String cvv = request.getMetadata().get("cvv");
        boolean approved = bankClient.chargeCard(cardNumber, cvv, request.getAmount());
        String txnId = UUID.randomUUID().toString();
        return new PaymentResponse(approved, txnId, approved ? "Payment successful" : "Declined");
    }
}
```

---

### `UPIStrategy.java`
```java
import java.util.UUID;

public class UPIStrategy implements PaymentStrategy {
    private BankAPIClient bankClient = new BankAPIClient();

    public PaymentResponse process(PaymentRequest request) {
        String upiId = request.getMetadata().get("upiId");
        boolean approved = bankClient.sendUPI(upiId, request.getAmount());
        String txnId = UUID.randomUUID().toString();
        return new PaymentResponse(approved, txnId, approved ? "UPI Payment successful" : "Failed");
    }
}
```

---

### `WalletStrategy.java`
```java
import java.util.UUID;

public class WalletStrategy implements PaymentStrategy {
    private BankAPIClient bankClient = new BankAPIClient();

    public PaymentResponse process(PaymentRequest request) {
        String walletId = request.getMetadata().get("walletId");
        boolean approved = bankClient.useWallet(walletId, request.getAmount());
        String txnId = UUID.randomUUID().toString();
        return new PaymentResponse(approved, txnId, approved ? "Wallet Payment successful" : "Failed");
    }
}
```

---

### `BankAPIClient.java`
```java
public class BankAPIClient {
    public boolean chargeCard(String cardNumber, String cvv, double amount) {
        return true;
    }

    public boolean sendUPI(String upiId, double amount) {
        return true;
    }

    public boolean useWallet(String walletId, double amount) {
        return true;
    }
}
```

---

### `TransactionRecord.java`
```java
public class TransactionRecord {
    private PaymentRequest request;
    private PaymentResponse response;

    public TransactionRecord(PaymentRequest request, PaymentResponse response) {
        this.request = request;
        this.response = response;
    }

    public PaymentRequest getRequest() { return request; }
    public PaymentResponse getResponse() { return response; }
}
```

---

### `TransactionService.java`
```java
import java.util.ArrayList;
import java.util.List;

public class TransactionService {
    private List<TransactionRecord> history = new ArrayList<>();

    public void save(TransactionRecord record) {
        history.add(record);
    }

    public List<TransactionRecord> getAll() {
        return history;
    }

    public List<TransactionRecord> getByUser(String userId) {
        List<TransactionRecord> result = new ArrayList<>();
        for (TransactionRecord record : history) {
            if (record.getRequest().getUserId().equals(userId)) {
                result.add(record);
            }
        }
        return result;
    }
}
```

---

### `PaymentGateway.java`
```java
import java.util.HashMap;
import java.util.Map;

public class PaymentGateway {
    private Map<PaymentMethod, PaymentStrategy> strategies = new HashMap<>();
    private TransactionService transactionService = new TransactionService();

    public PaymentGateway() {
        strategies.put(PaymentMethod.CREDIT_CARD, new CreditCardStrategy());
        strategies.put(PaymentMethod.UPI, new UPIStrategy());
        strategies.put(PaymentMethod.WALLET, new WalletStrategy());
    }

    public PaymentResponse processPayment(PaymentRequest request) {
        PaymentStrategy strategy = strategies.get(request.getMethod());
        PaymentResponse response = strategy.process(request);
        transactionService.save(new TransactionRecord(request, response));
        return response;
    }

    public TransactionService getTransactionService() {
        return transactionService;
    }
}
```

---

### `PaymentUI.java`
```java
import java.util.*;

public class PaymentUI {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        PaymentGateway gateway = new PaymentGateway();

        while (true) {
            System.out.println("1. Make Payment\n2. View All Transactions\n3. View Transactions by User\n4. Exit");
            int choice = Integer.parseInt(scanner.nextLine());

            if (choice == 1) {
                System.out.println("Enter User ID:");
                String userId = scanner.nextLine();

                System.out.println("Enter Amount:");
                double amount = Double.parseDouble(scanner.nextLine());

                System.out.println("Enter Currency:");
                String currency = scanner.nextLine();

                System.out.println("Enter Payment Method (CREDIT_CARD / UPI / WALLET):");
                PaymentMethod method = PaymentMethod.valueOf(scanner.nextLine().toUpperCase());

                Map<String, String> metadata = new HashMap<>();
                switch (method) {
                    case CREDIT_CARD:
                        System.out.println("Enter Card Number:");
                        metadata.put("cardNumber", scanner.nextLine());
                        System.out.println("Enter CVV:");
                        metadata.put("cvv", scanner.nextLine());
                        break;
                    case UPI:
                        System.out.println("Enter UPI ID:");
                        metadata.put("upiId", scanner.nextLine());
                        break;
                    case WALLET:
                        System.out.println("Enter Wallet ID:");
                        metadata.put("walletId", scanner.nextLine());
                        break;
                }

                PaymentRequest request = new PaymentRequest(userId, amount, currency, method, metadata);
                PaymentResponse response = gateway.processPayment(request);

                System.out.println("Transaction ID: " + response.getTransactionId());
                System.out.println("Status: " + (response.isSuccess() ? "Success" : "Failure"));
                System.out.println("Message: " + response.getMessage());

            } else if (choice == 2) {
                List<TransactionRecord> all = gateway.getTransactionService().getAll();
                for (TransactionRecord record : all) {
                    System.out.println(record.getRequest().getUserId() + " | " +
                            record.getResponse().getTransactionId() + " | " +
                            record.getResponse().getMessage());
                }

            } else if (choice == 3) {
                System.out.println("Enter User ID:");
                String userId = scanner.nextLine();
                List<TransactionRecord> userTxns = gateway.getTransactionService().getByUser(userId);
                for (TransactionRecord record : userTxns) {
                    System.out.println(record.getRequest().getUserId() + " | " +
                            record.getResponse().getTransactionId() + " | " +
                            record.getResponse().getMessage());
                }

            } else if (choice == 4) {
                break;
            }
        }

        scanner.close();
    }
}
```
