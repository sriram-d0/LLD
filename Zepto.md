## Inventory Management System

## 🧾 **Core Entities**
- **Product**: Basic item with `id` and `name`.
- **ProductCategory**: Groups products; has `categoryId`, `categoryName`, `price`, `stockCount`, and a list of `Product`s. Supports stock updates.
- **Inventory**: Holds multiple `ProductCategory` objects. Used by warehouses.
- **Address**: Contains `pincode` and `city`.
- **WareHouse**: Has an `Inventory` and `Address`. Manages product availability.
- **WareHouseController**: Manages all warehouses.

---

## 👤 **User & Cart**
- **User**: Has `id`, `name`, and a `Cart`.
- **Cart**: Maps `categoryId` to quantity. Supports add/remove operations.
- **UserController**: Manages all users.

---

## 🧠 **Strategy Pattern**
- **WareHouseSelectionStrategy**: Interface for selecting a warehouse.
  - `NearestWareHouseStrategy`: Chooses warehouse by matching city.
  - `CheapestWareHouseStrategy`: Chooses warehouse with lowest product price.

---

## 📦 **Order Flow**
- **Order**: Created by a user, tied to a warehouse and cart items. On placement, reduces stock.
- **Invoice**: Generated during order; calculates total cost.
- **Payment**: Linked to invoice; processes payment and updates status.

---

## 🚀 **App Entry**
- **App**: Coordinates `UserController` and `WareHouseController`. Runs order flow using a selected strategy.

---


## 🧱 Core Domain Classes

### `Product.java`
```java
public class Product {
    int id;
    String name;

    public Product(int id, String name) {
        this.id = id;
        this.name = name;
    }
}
```

### `ProductCategory.java`
```java
import java.util.*;

public class ProductCategory {
    int categoryId;
    String categoryName;
    List<Product> products = new ArrayList<>();
    double price;
    int stockCount;

    public ProductCategory(int categoryId, String categoryName, double price, int stockCount) {
        this.categoryId = categoryId;
        this.categoryName = categoryName;
        this.price = price;
        this.stockCount = stockCount;
    }

    public void addProduct(Product p) {
        products.add(p);
    }

    public void removeProduct(Product p) {
        products.remove(p);
    }

    public boolean reduceStock(int count) {
        if (stockCount >= count) {
            stockCount -= count;
            return true;
        }
        return false;
    }
}
```

### `Inventory.java`
```java
import java.util.*;

public class Inventory {
    List<ProductCategory> categories = new ArrayList<>();

    public void addCategory(ProductCategory pc) {
        categories.add(pc);
    }

    public ProductCategory getCategoryById(int id) {
        for (ProductCategory pc : categories) {
            if (pc.categoryId == id) return pc;
        }
        return null;
    }
}
```

### `Address.java`
```java
public class Address {
    int pincode;
    String city;

    public Address(int pincode, String city) {
        this.pincode = pincode;
        this.city = city;
    }
}
```

### `WareHouse.java`
```java
public class WareHouse {
    Inventory inventory;
    Address address;

    public WareHouse(Inventory inventory, Address address) {
        this.inventory = inventory;
        this.address = address;
    }
}
```

### `WareHouseController.java`
```java
import java.util.*;

public class WareHouseController {
    List<WareHouse> warehouses = new ArrayList<>();

    public void addWareHouse(WareHouse wh) {
        warehouses.add(wh);
    }

    public List<WareHouse> getWareHouses() {
        return warehouses;
    }
}
```

---

## 🛒 User and Cart

### `Cart.java`
```java
import java.util.*;

public class Cart {
    Map<Integer, Integer> categoryVsCount = new HashMap<>();

    public void addItem(int categoryId, int count) {
        categoryVsCount.put(categoryId, categoryVsCount.getOrDefault(categoryId, 0) + count);
    }

    public void removeItem(int categoryId) {
        categoryVsCount.remove(categoryId);
    }

    public Map<Integer, Integer> getItems() {
        return categoryVsCount;
    }
}
```

### `User.java`
```java
public class User {
    int id;
    String name;
    Cart cart = new Cart();

    public User(int id, String name) {
        this.id = id;
        this.name = name;
    }
}
```

### `UserController.java`
```java
import java.util.*;

public class UserController {
    List<User> users = new ArrayList<>();

    public void addUser(User u) {
        users.add(u);
    }

    public User getUserById(int id) {
        for (User u : users) {
            if (u.id == id) return u;
        }
        return null;
    }
}
```

---

## 🧠 Strategy Pattern

### `WareHouseSelectionStrategy.java`
```java
public interface WareHouseSelectionStrategy {
    WareHouse selectWareHouse(List<WareHouse> warehouses, Address userAddress);
}
```

### `NearestWareHouseStrategy.java`
```java
public class NearestWareHouseStrategy implements WareHouseSelectionStrategy {
    public WareHouse selectWareHouse(List<WareHouse> warehouses, Address userAddress) {
        for (WareHouse wh : warehouses) {
            if (wh.address.city.equals(userAddress.city)) return wh;
        }
        return warehouses.get(0); // fallback
    }
}
```

### `CheapestWareHouseStrategy.java`
```java
public class CheapestWareHouseStrategy implements WareHouseSelectionStrategy {
    public WareHouse selectWareHouse(List<WareHouse> warehouses, Address userAddress) {
        WareHouse cheapest = null;
        double minPrice = Double.MAX_VALUE;
        for (WareHouse wh : warehouses) {
            for (ProductCategory pc : wh.inventory.categories) {
                if (pc.price < minPrice) {
                    minPrice = pc.price;
                    cheapest = wh;
                }
            }
        }
        return cheapest;
    }
}
```

---

## 📦 Order Flow

### `Order.java`
```java
import java.util.*;

public class Order {
    int orderId;
    User user;
    WareHouse warehouse;
    Map<Integer, Integer> items;
    Invoice invoice;

    public Order(int orderId, User user, WareHouse warehouse, Map<Integer, Integer> items) {
        this.orderId = orderId;
        this.user = user;
        this.warehouse = warehouse;
        this.items = items;
        this.invoice = new Invoice(orderId, items, warehouse);
    }

    public boolean placeOrder() {
        for (Map.Entry<Integer, Integer> entry : items.entrySet()) {
            ProductCategory pc = warehouse.inventory.getCategoryById(entry.getKey());
            if (pc == null || !pc.reduceStock(entry.getValue())) {
                return false;
            }
        }
        return true;
    }
}
```

### `Invoice.java`
```java
import java.util.*;

public class Invoice {
    int invoiceId;
    Map<Integer, Integer> items;
    double totalAmount;

    public Invoice(int invoiceId, Map<Integer, Integer> items, WareHouse warehouse) {
        this.invoiceId = invoiceId;
        this.items = items;
        this.totalAmount = calculateTotal(warehouse);
    }

    private double calculateTotal(WareHouse warehouse) {
        double sum = 0;
        for (Map.Entry<Integer, Integer> entry : items.entrySet()) {
            ProductCategory pc = warehouse.inventory.getCategoryById(entry.getKey());
            if (pc != null) {
                sum += pc.price * entry.getValue();
            }
        }
        return sum;
    }
}
```

### `Payment.java`
```java
public class Payment {
    int paymentId;
    double amount;
    String status;

    public Payment(int paymentId, double amount) {
        this.paymentId = paymentId;
        this.amount = amount;
        this.status = "Pending";
    }

    public void processPayment() {
        this.status = "Completed";
    }
}
```

---

## 🚀 App Entry Point

### `App.java`
```java
public class App {
    UserController uc = new UserController();
    WareHouseController whc = new WareHouseController();

    public void runOrderFlow(int userId, WareHouseSelectionStrategy strategy) {
        User user = uc.getUserById(userId);
        WareHouse selectedWH = strategy.selectWareHouse(whc.getWareHouses(), new Address(500084, "Hyderabad"));
        Order order = new Order(1, user, selectedWH, user.cart.getItems());

        if (order.placeOrder()) {
            Payment payment = new Payment(1, order.invoice.totalAmount);
            payment.processPayment();
            System.out.println("Order placed and payment completed.");
        } else {
            System.out.println("Order failed due to insufficient stock.");
        }
    }
}
```

---


