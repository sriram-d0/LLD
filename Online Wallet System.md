# Digital wallet system 





## Design overview

- **Entities:** `User`, `Wallet`, `Transaction`
- **Transaction types:** `ADD_FUNDS`, `PAYMENT`, `TRANSFER`, `REFUND`
- **Status:** `PENDING`, `SUCCESS`, `FAILED`, `REFUNDED`
- **Repositories:** In-memory stores for users, wallets, and transactions
- **Service layer:** `WalletService` for operations and validation
- **Security/permissions (simple):** Only the wallet owner can initiate transfers or payments from their wallet
- **Concurrency:** Synchronized balance updates in `Wallet`

---

## Core models

```java
package wallet;

import java.math.BigDecimal;
import java.util.Objects;
import java.util.UUID;

public class User {
    private final String userId;
    private final String name;
    private final String email;

    public User(String name, String email) {
        this.userId = UUID.randomUUID().toString();
        this.name = Objects.requireNonNull(name);
        this.email = Objects.requireNonNull(email);
    }

    public String getUserId() { return userId; }
    public String getName() { return name; }
    public String getEmail() { return email; }

    @Override
    public String toString() {
        return "User{id=" + userId + ", name=" + name + ", email=" + email + "}";
    }
}
```

```java
package wallet;

import java.math.BigDecimal;
import java.util.UUID;

public class Wallet {
    private final String walletId;
    private final String userId;
    private BigDecimal balance;

    public Wallet(String userId) {
        this.walletId = UUID.randomUUID().toString();
        this.userId = userId;
        this.balance = BigDecimal.ZERO;
    }

    public String getWalletId() { return walletId; }
    public String getUserId() { return userId; }

    public synchronized BigDecimal getBalance() { return balance; }

    public synchronized void credit(BigDecimal amount) {
        validateAmount(amount);
        balance = balance.add(amount);
    }

    public synchronized void debit(BigDecimal amount) {
        validateAmount(amount);
        if (balance.compareTo(amount) < 0) {
            throw new InsufficientBalanceException("Insufficient funds for wallet " + walletId);
        }
        balance = balance.subtract(amount);
    }

    private void validateAmount(BigDecimal amount) {
        if (amount == null || amount.compareTo(BigDecimal.ZERO) <= 0) {
            throw new IllegalArgumentException("Amount must be positive");
        }
    }

    @Override
    public String toString() {
        return "Wallet{id=" + walletId + ", userId=" + userId + ", balance=" + balance + "}";
    }
}
```

```java
package wallet;

public enum TransactionType {
    ADD_FUNDS, PAYMENT, TRANSFER, REFUND
}
```

```java
package wallet;

public enum TransactionStatus {
    PENDING, SUCCESS, FAILED, REFUNDED
}
```

```java
package wallet;

import java.math.BigDecimal;
import java.time.Instant;
import java.util.UUID;

public class Transaction {
    private final String txnId;
    private final TransactionType type;
    private final String fromWalletId; // nullable for ADD_FUNDS
    private final String toWalletId;   // nullable for PAYMENT
    private final BigDecimal amount;
    private final Instant createdAt;
    private TransactionStatus status;
    private String reference; // e.g., merchant ref or refund reason

    public Transaction(TransactionType type, String fromWalletId, String toWalletId,
                       BigDecimal amount, String reference) {
        this.txnId = UUID.randomUUID().toString();
        this.type = type;
        this.fromWalletId = fromWalletId;
        this.toWalletId = toWalletId;
        this.amount = amount;
        this.createdAt = Instant.now();
        this.status = TransactionStatus.PENDING;
        this.reference = reference;
    }

    public String getTxnId() { return txnId; }
    public TransactionType getType() { return type; }
    public String getFromWalletId() { return fromWalletId; }
    public String getToWalletId() { return toWalletId; }
    public BigDecimal getAmount() { return amount; }
    public Instant getCreatedAt() { return createdAt; }
    public TransactionStatus getStatus() { return status; }
    public String getReference() { return reference; }

    public void markSuccess() { this.status = TransactionStatus.SUCCESS; }
    public void markFailed() { this.status = TransactionStatus.FAILED; }
    public void markRefunded() { this.status = TransactionStatus.REFUNDED; }

    @Override
    public String toString() {
        return "Txn{id=" + txnId + ", type=" + type + ", from=" + fromWalletId +
            ", to=" + toWalletId + ", amount=" + amount + ", status=" + status +
            ", ref=" + reference + ", at=" + createdAt + "}";
    }
}
```

---

## Exceptions and repositories

```java
package wallet;

public class NotFoundException extends RuntimeException {
    public NotFoundException(String msg) { super(msg); }
}
```

```java
package wallet;

public class PermissionException extends RuntimeException {
    public PermissionException(String msg) { super(msg); }
}
```

```java
package wallet;

public class InsufficientBalanceException extends RuntimeException {
    public InsufficientBalanceException(String msg) { super(msg); }
}
```

```java
package wallet;

import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;

public class UserRepository {
    private final Map<String, User> users = new ConcurrentHashMap<>();

    public User save(User user) {
        users.put(user.getUserId(), user);
        return user;
    }

    public User findById(String userId) {
        User u = users.get(userId);
        if (u == null) throw new NotFoundException("User not found: " + userId);
        return u;
    }
}
```

```java
package wallet;

import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;

public class WalletRepository {
    private final Map<String, Wallet> wallets = new ConcurrentHashMap<>();

    public Wallet save(Wallet wallet) {
        wallets.put(wallet.getWalletId(), wallet);
        return wallet;
    }

    public Wallet findById(String walletId) {
        Wallet w = wallets.get(walletId);
        if (w == null) throw new NotFoundException("Wallet not found: " + walletId);
        return w;
    }

    public Wallet findByUserId(String userId) {
        return wallets.values().stream()
            .filter(w -> w.getUserId().equals(userId))
            .findFirst()
            .orElseThrow(() -> new NotFoundException("Wallet not found for user: " + userId));
    }
}
```

```java
package wallet;

import java.util.ArrayList;
import java.util.List;
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;

public class TransactionRepository {
    private final Map<String, Transaction> txns = new ConcurrentHashMap<>();

    public Transaction save(Transaction txn) {
        txns.put(txn.getTxnId(), txn);
        return txn;
    }

    public Transaction findById(String txnId) {
        Transaction t = txns.get(txnId);
        if (t == null) throw new NotFoundException("Transaction not found: " + txnId);
        return t;
    }

    public List<Transaction> findByWallet(String walletId) {
        List<Transaction> list = new ArrayList<>();
        for (Transaction t : txns.values()) {
            if (walletId.equals(t.getFromWalletId()) || walletId.equals(t.getToWalletId())) {
                list.add(t);
            }
        }
        return list;
    }
}
```

---

## Service layer

```java
package wallet;

import java.math.BigDecimal;
import java.util.Objects;

public class WalletService {
    private final UserRepository userRepo;
    private final WalletRepository walletRepo;
    private final TransactionRepository txnRepo;

    public WalletService(UserRepository userRepo, WalletRepository walletRepo, TransactionRepository txnRepo) {
        this.userRepo = userRepo;
        this.walletRepo = walletRepo;
        this.txnRepo = txnRepo;
    }

    // Create user + wallet
    public Wallet createUserWithWallet(String name, String email) {
        User user = userRepo.save(new User(name, email));
        Wallet wallet = walletRepo.save(new Wallet(user.getUserId()));
        return wallet;
    }

    // Add funds (e.g., from bank/card)
    public Transaction addFunds(String userId, BigDecimal amount, String reference) {
        Objects.requireNonNull(userId);
        userRepo.findById(userId); // ensure user exists
        Wallet wallet = walletRepo.findByUserId(userId);

        Transaction txn = new Transaction(TransactionType.ADD_FUNDS, null, wallet.getWalletId(), amount, reference);
        txnRepo.save(txn);

        try {
            wallet.credit(amount);
            txn.markSuccess();
        } catch (RuntimeException e) {
            txn.markFailed();
            throw e;
        }
        return txn;
    }

    // Pay a merchant (merchant is represented as a wallet too)
    public Transaction pay(String payerUserId, String merchantWalletId, BigDecimal amount, String reference) {
        User payer = userRepo.findById(payerUserId);
        Wallet payerWallet = walletRepo.findByUserId(payer.getUserId());
        Wallet merchantWallet = walletRepo.findById(merchantWalletId);

        enforceOwner(payer.getUserId(), payerWallet);

        Transaction txn = new Transaction(TransactionType.PAYMENT, payerWallet.getWalletId(), merchantWallet.getWalletId(), amount, reference);
        txnRepo.save(txn);

        try {
            // Order matters: debit first, then credit
            payerWallet.debit(amount);
            merchantWallet.credit(amount);
            txn.markSuccess();
        } catch (RuntimeException e) {
            txn.markFailed();
            throw e;
        }
        return txn;
    }

    // Transfer between user wallets
    public Transaction transfer(String senderUserId, String receiverUserId, BigDecimal amount, String reference) {
        User sender = userRepo.findById(senderUserId);
        User receiver = userRepo.findById(receiverUserId);

        Wallet from = walletRepo.findByUserId(sender.getUserId());
        Wallet to = walletRepo.findByUserId(receiver.getUserId());

        enforceOwner(sender.getUserId(), from);

        // Simple deadlock avoidance: lock ordering by walletId
        Wallet firstLock = from.getWalletId().compareTo(to.getWalletId()) < 0 ? from : to;
        Wallet secondLock = firstLock == from ? to : from;

        Transaction txn = new Transaction(TransactionType.TRANSFER, from.getWalletId(), to.getWalletId(), amount, reference);
        txnRepo.save(txn);

        synchronized (firstLock) {
            synchronized (secondLock) {
                try {
                    from.debit(amount);
                    to.credit(amount);
                    txn.markSuccess();
                } catch (RuntimeException e) {
                    txn.markFailed();
                    throw e;
                }
            }
        }
        return txn;
    }

    // Refund a previous payment (basic: reverse to original payer)
    public Transaction refund(String originalTxnId, String merchantUserId, String reason) {
        Transaction original = txnRepo.findById(originalTxnId);
        if (original.getType() != TransactionType.PAYMENT || original.getStatus() != TransactionStatus.SUCCESS) {
            throw new IllegalArgumentException("Only successful PAYMENT can be refunded");
        }

        User merchant = userRepo.findById(merchantUserId);
        Wallet merchantWallet = walletRepo.findByUserId(merchant.getUserId());
        Wallet payerWallet = walletRepo.findById(original.getFromWalletId());

        enforceOwner(merchant.getUserId(), merchantWallet);

        BigDecimal amount = original.getAmount();
        Transaction refund = new Transaction(TransactionType.REFUND, merchantWallet.getWalletId(), payerWallet.getWalletId(), amount, reason);
        txnRepo.save(refund);

        try {
            merchantWallet.debit(amount);
            payerWallet.credit(amount);
            refund.markSuccess();
            original.markRefunded();
        } catch (RuntimeException e) {
            refund.markFailed();
            throw e;
        }
        return refund;
    }

    // Helpers
    private void enforceOwner(String userId, Wallet wallet) {
        if (!wallet.getUserId().equals(userId)) {
            throw new PermissionException("User " + userId + " does not own wallet " + wallet.getWalletId());
        }
    }

    // Query helpers
    public Wallet getWalletByUser(String userId) { return walletRepo.findByUserId(userId); }
    public User getUser(String userId) { return userRepo.findById(userId); }
}
```

---

## Sample usage and flow

```java
package wallet;

import java.math.BigDecimal;
import java.util.List;

public class Demo {
    public static void main(String[] args) {
        // Setup
        UserRepository userRepo = new UserRepository();
        WalletRepository walletRepo = new WalletRepository();
        TransactionRepository txnRepo = new TransactionRepository();
        WalletService service = new WalletService(userRepo, walletRepo, txnRepo);

        // Create users and wallets
        Wallet sriramWallet = service.createUserWithWallet("Sriram", "sriram@example.com");
        Wallet aliceWallet = service.createUserWithWallet("Alice", "alice@example.com");
        Wallet merchantWallet = service.createUserWithWallet("Cafe Mocha", "merchant@cafemocha.com");

        String sriramUserId = service.getUser(sriramWallet.getUserId()).getUserId(); // same as stored
        String aliceUserId = service.getUser(aliceWallet.getUserId()).getUserId();
        String merchantUserId = service.getUser(merchantWallet.getUserId()).getUserId();

        System.out.println("Initial balances:");
        System.out.println("Sriram: " + sriramWallet.getBalance());
        System.out.println("Alice: " + aliceWallet.getBalance());
        System.out.println("Merchant: " + merchantWallet.getBalance());

        // Add funds to Sriram's wallet
        service.addFunds(sriramUserId, new BigDecimal("1000.00"), "Bank UPI top-up");
        System.out.println("\nAfter top-up:");
        System.out.println("Sriram: " + sriramWallet.getBalance());

        // Pay merchant for coffee
        var paymentTxn = service.pay(sriramUserId, merchantWallet.getWalletId(), new BigDecimal("150.00"), "Latte x2");
        System.out.println("\nPayment txn: " + paymentTxn);
        System.out.println("Balances after payment:");
        System.out.println("Sriram: " + sriramWallet.getBalance());
        System.out.println("Merchant: " + merchantWallet.getBalance());

        // Transfer money to Alice
        var transferTxn = service.transfer(sriramUserId, aliceUserId, new BigDecimal("200.00"), "Split lunch");
        System.out.println("\nTransfer txn: " + transferTxn);
        System.out.println("Balances after transfer:");
        System.out.println("Sriram: " + sriramWallet.getBalance());
        System.out.println("Alice: " + aliceWallet.getBalance());

        // Refund the coffee (merchant initiates)
        var refundTxn = service.refund(paymentTxn.getTxnId(), merchantUserId, "Order canceled");
        System.out.println("\nRefund txn: " + refundTxn);
        System.out.println("Balances after refund:");
        System.out.println("Sriram: " + sriramWallet.getBalance());
        System.out.println("Merchant: " + merchantWallet.getBalance());

        // View Sriram's transaction history
        List<Transaction> history = txnRepo.findByWallet(sriramWallet.getWalletId());
        System.out.println("\nSriram's transactions:");
        history.forEach(System.out::println);
    }
}
```

---
