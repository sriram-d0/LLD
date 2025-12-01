# Vending Machine

### Core idea
The vending machine is modeled as a set of classes that handle **items, slots, inventory, payments, transactions, and receipts**. The machine moves through states (idle → selection → payment → dispensing → finish).

---

### Key classes
- **[Item](guide://action?prefill=Tell%20me%20more%20about%3A%20Item)** → Represents a product with `id`, `name`, and `price`.
- **[Slot](guide://action?prefill=Tell%20me%20more%20about%3A%20Slot)** → Holds an item and its quantity (like a shelf position).
- **[Inventory](guide://action?prefill=Tell%20me%20more%20about%3A%20Inventory)** → Manages all slots, checks stock, and dispenses items.
- **[PaymentMethod](guide://action?prefill=Tell%20me%20more%20about%3A%20PaymentMethod)** → Interface for different payment types (card, cash, coins).
- **[CardPayment](guide://action?prefill=Tell%20me%20more%20about%3A%20CardPayment)** → Simple card payment implementation.
- **[Transaction](guide://action?prefill=Tell%20me%20more%20about%3A%20Transaction)** → Tracks selection, total price, amount paid, change, and status.
- **[Receipt](guide://action?prefill=Tell%20me%20more%20about%3A%20Receipt)** → Prints out transaction details.
- **[VendingMachine](guide://action?prefill=Tell%20me%20more%20about%3A%20VendingMachine)** → The main controller: handles selection, payment, dispensing, and resets state.

---

### Flow of usage
1. **Select item** → User chooses a slot and quantity.  
2. **Get amount due** → Machine calculates total price.  
3. **Pay** → User pays (card/cash/coins).  
4. **Dispense** → Machine reduces inventory, calculates change.  
5. **Finish** → Receipt is generated, machine resets to idle.

---

### States
- **Idle** → Waiting for user.  
- **Selection** → Item chosen.  
- **Payment Pending** → Waiting for money.  
- **Dispensing** → Giving product.  
- **Returning Change** → Giving back extra money.  
- **Out of Service** → Disabled.

---


### MachineState.java
```java
public enum MachineState {
    IDLE, SELECTION, PAYMENT_PENDING, DISPENSING, RETURNING_CHANGE, OUT_OF_SERVICE
}
```

---

### Item.java
```java
public class Item {
    private String id;
    private String name;
    private double price;

    public Item(String id, String name, double price) {
        this.id = id;
        this.name = name;
        this.price = price;
    }

    public String getId() { return id; }
    public String getName() { return name; }
    public double getPrice() { return price; }
}
```

---

### Slot.java
```java
public class Slot {
    private String slotId;
    private Item item;
    private int quantity;

    public Slot(String slotId) {
        this.slotId = slotId;
        this.quantity = 0;
    }

    public String getSlotId() { return slotId; }
    public Item getItem() { return item; }
    public int getQuantity() { return quantity; }

    public void setItem(Item item) { this.item = item; }
    public void restock(int count) { quantity += count; }
    public void decrement(int count) { quantity -= count; }
}
```

---

### Inventory.java
```java
import java.util.*;

public class Inventory {
    private Map<String, Slot> slots = new HashMap<>();

    public void addSlot(Slot slot) { slots.put(slot.getSlotId(), slot); }
    public Slot getSlot(String slotId) { return slots.get(slotId); }
    public boolean hasStock(String slotId, int qty) {
        Slot s = getSlot(slotId);
        return s != null && s.getItem() != null && s.getQuantity() >= qty;
    }
    public void dispense(String slotId, int qty) { getSlot(slotId).decrement(qty); }
}
```

---

### PaymentMethod.java
```java
public interface PaymentMethod {
    boolean authorize(double amount);
    boolean capture(double amount);
    boolean refund(double amount);
}
```

---

### CardPayment.java
```java
public class CardPayment implements PaymentMethod {
    public boolean authorize(double amount) { return amount > 0; }
    public boolean capture(double amount) { return true; }
    public boolean refund(double amount) { return true; }
}
```

---

### Transaction.java
```java
public class Transaction {
    private String slotId;
    private int quantity;
    private double totalPrice;
    private double paid;
    private double change;
    private String status;

    public void select(String slotId, int qty, double pricePerItem) {
        this.slotId = slotId;
        this.quantity = qty;
        this.totalPrice = pricePerItem * qty;
        this.status = "INITIATED";
    }

    public void authorize(double paid) {
        this.paid = paid;
        this.status = "AUTHORIZED";
    }

    public void captured() { this.status = "CAPTURED"; }

    public void dispensed(double change) {
        this.change = change;
        this.status = "DISPENSED";
    }

    public String getStatus() { return status; }
    public double getTotalPrice() { return totalPrice; }
    public double getPaid() { return paid; }
    public double getChange() { return change; }
}
```

---

### Receipt.java
```java
public class Receipt {
    private Transaction tx;

    public Receipt(Transaction tx) { this.tx = tx; }

    public String toString() {
        return "Receipt: slot=" + tx.getStatus() +
               ", total=" + tx.getTotalPrice() +
               ", paid=" + tx.getPaid() +
               ", change=" + tx.getChange();
    }
}
```

---

### VendingMachine.java
```java
public class VendingMachine {
    private Inventory inventory = new Inventory();
    private MachineState state = MachineState.IDLE;
    private Transaction currentTx;

    public void addSlot(Slot slot) { inventory.addSlot(slot); }

    public void select(String slotId, int qty) {
        Slot slot = inventory.getSlot(slotId);
        if (slot != null && inventory.hasStock(slotId, qty)) {
            currentTx = new Transaction();
            currentTx.select(slotId, qty, slot.getItem().getPrice());
            state = MachineState.SELECTION;
        }
    }

    public double getAmountDue() { return currentTx.getTotalPrice(); }

    public void pay(double amount) {
        if (amount >= currentTx.getTotalPrice()) {
            currentTx.authorize(amount);
            currentTx.captured();
            double change = amount - currentTx.getTotalPrice();
            inventory.dispense(currentTx.slotId, currentTx.quantity);
            currentTx.dispensed(change);
            state = MachineState.DISPENSING;
        }
    }

    public Receipt finish() {
        Receipt r = new Receipt(currentTx);
        state = MachineState.IDLE;
        currentTx = null;
        return r;
    }
}
```

---
