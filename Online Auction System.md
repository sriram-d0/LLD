

# Online Auction System LLD

## Flow Summary

- **[User](guide://action?prefill=Tell%20me%20more%20about%3A%20User)**: Has `id`, `name`, `UserType` (SELLER/BUYER/ADMIN), and `balance`.
- **[Item](guide://action?prefill=Tell%20me%20more%20about%3A%20Item)**: Listed by seller, has metadata.
- **[Auction](guide://action?prefill=Tell%20me%20more%20about%3A%20Auction)**: Holds item, start/end time, reserve price, bids, status.
- **[Bid](guide://action?prefill=Tell%20me%20more%20about%3A%20Bid)**: Placed by buyer with amount and timestamp.
- **[Payment](guide://action?prefill=Tell%20me%20more%20about%3A%20Payment)**: Escrow and settlement logic.
- **[AuctionSystem](guide://action?prefill=Tell%20me%20more%20about%3A%20AuctionSystem)**: Main driver holding users, items, auctions, supports search and lifecycle.

---

## 1. User and Roles

```java
public enum UserType {
    ADMIN, SELLER, BUYER, GUEST
}

public class User {
    private int id;
    private String name;
    private UserType userType;
    private double balance;

    public User(int id, String name, UserType userType, double balance) {
        this.id = id;
        this.name = name;
        this.userType = userType;
        this.balance = balance;
    }

    // Permission checks
    public boolean canCreateAuction() {
        return userType == UserType.SELLER || userType == UserType.ADMIN;
    }

    public boolean canBid() {
        return userType == UserType.BUYER;
    }

    public boolean canCloseAuction() {
        return userType == UserType.SELLER || userType == UserType.ADMIN;
    }

    // Balance operations
    public void debit(double amount) {
        if (balance < amount) throw new IllegalStateException("Insufficient balance");
        balance -= amount;
    }

    public void credit(double amount) {
        balance += amount;
    }

    // Getters
    public int getId() { return id; }
    public String getName() { return name; }
    public UserType getUserType() { return userType; }
    public double getBalance() { return balance; }
}
```

---

## 2. Item

```java
public class Item {
    private int id;
    private String name;
    private String description;
    private User seller;

    public Item(int id, String name, String description, User seller) {
        this.id = id;
        this.name = name;
        this.description = description;
        this.seller = seller;
    }

    // Getters
    public int getId() { return id; }
    public String getName() { return name; }
    public String getDescription() { return description; }
    public User getSeller() { return seller; }
}
```

---

## 3. Bid

```java
import java.time.LocalDateTime;

public class Bid {
    private int id;
    private User bidder;
    private double amount;
    private LocalDateTime timestamp;

    public Bid(int id, User bidder, double amount) {
        if (!bidder.canBid()) {
            throw new IllegalStateException("Only buyers can place bids");
        }
        this.id = id;
        this.bidder = bidder;
        this.amount = amount;
        this.timestamp = LocalDateTime.now();
    }

    // Getters
    public int getId() { return id; }
    public User getBidder() { return bidder; }
    public double getAmount() { return amount; }
    public LocalDateTime getTimestamp() { return timestamp; }
}
```

---

## 4. Auction

```java
import java.time.LocalDateTime;
import java.util.*;

public enum AuctionStatus {
    CREATED, RUNNING, ENDED, FAILED, SETTLED
}

public class Auction {
    private int id;
    private Item item;
    private User seller;
    private LocalDateTime startTime;
    private LocalDateTime endTime;
    private double reservePrice;
    private AuctionStatus status;
    private List<Bid> bids;
    private Bid highestBid;

    public Auction(int id, Item item, User seller, LocalDateTime startTime, LocalDateTime endTime, double reservePrice) {
        if (!seller.canCreateAuction()) {
            throw new IllegalStateException("Only sellers/admin can create auctions");
        }
        this.id = id;
        this.item = item;
        this.seller = seller;
        this.startTime = startTime;
        this.endTime = endTime;
        this.reservePrice = reservePrice;
        this.status = AuctionStatus.CREATED;
        this.bids = new ArrayList<>();
    }

    public void startAuction() {
        if (status != AuctionStatus.CREATED) throw new IllegalStateException("Auction not in CREATED state");
        this.status = AuctionStatus.RUNNING;
    }

    public void placeBid(Bid bid) {
        if (status != AuctionStatus.RUNNING) throw new IllegalStateException("Auction not running");
        if (bid.getAmount() <= (highestBid == null ? reservePrice : highestBid.getAmount())) {
            throw new IllegalStateException("Bid too low");
        }
        bids.add(bid);
        highestBid = bid;
    }

    public void closeAuction(User user) {
        if (!user.canCloseAuction()) throw new IllegalStateException("No permission to close auction");
        this.status = AuctionStatus.ENDED;
        if (highestBid == null || highestBid.getAmount() < reservePrice) {
            this.status = AuctionStatus.FAILED;
        }
    }

    public Bid getHighestBid() { return highestBid; }
    public AuctionStatus getStatus() { return status; }
    public List<Bid> getBids() { return bids; }
}
```

---

## 5. Payment

```java
public class Payment {
    public void processPayment(Auction auction) {
        if (auction.getStatus() != AuctionStatus.ENDED) {
            throw new IllegalStateException("Auction must be ended before payment");
        }
        Bid winningBid = auction.getHighestBid();
        if (winningBid == null) {
            throw new IllegalStateException("No winning bid");
        }
        User buyer = winningBid.getBidder();
        User seller = auction.getHighestBid().getBidder(); // Correction: should be auction.getItem().getSeller()

        buyer.debit(winningBid.getAmount());
        auction.getItem().getSeller().credit(winningBid.getAmount());
        auction.closeAuction(auction.getItem().getSeller());
    }
}
```

---

## 6. AuctionSystem (Driver)

```java
import java.util.*;

public class AuctionSystem {
    private List<User> users;
    private List<Item> items;
    private List<Auction> auctions;

    public AuctionSystem() {
        this.users = new ArrayList<>();
        this.items = new ArrayList<>();
        this.auctions = new ArrayList<>();
    }

    public void addUser(User user) { users.add(user); }
    public void addItem(Item item) { items.add(item); }
    public void addAuction(Auction auction) { auctions.add(auction); }

    public List<Auction> searchAuctionsByItemName(String name) {
        List<Auction> result = new ArrayList<>();
        for (Auction a : auctions) {
            if (a.getStatus() == AuctionStatus.RUNNING && a.getItem().getName().equalsIgnoreCase(name)) {
                result.add(a);
            }
        }
        return result;
    }

    // Getters
    public List<User> getUsers() { return users; }
    public List<Item> getItems() { return items; }
    public List<Auction> getAuctions() { return auctions; }
}
```

---

## Example Usage Flow

1. Seller creates an item and auction.  
2. Auction starts → buyers place bids.  
3. Auction closes → highest valid bid wins.  
4. Payment processed → funds transferred from buyer to seller.  

---
