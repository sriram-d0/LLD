


# 🚗 Car Rental System – Low-Level Design (LLD)

## 📖 Overview

This project presents a comprehensive Low-Level Design (LLD) for a modular Car Rental System built in Java. It models the core entities and workflows required to manage vehicle rentals across multiple store locations. The system supports:

- Vehicle inventory management
- User registration and lookup
- Reservation creation and lifecycle tracking
- Store-level operations and vehicle assignment

Each class is designed with clear responsibilities and extensible structure, using enums for state management and encapsulating CRUD operations where appropriate. The design emphasizes clarity, modularity, and real-world applicability, making it suitable for both backend implementation and future API integration.

---

## 📦 Project Structure

```plaintext
com.carrental
├── enums
│   ├── VehicleType.java
│   ├── Status.java
│   ├── ReservationType.java
│   └── ReservationStatus.java
├── models
│   ├── Vehicle.java
│   ├── Location.java
│   ├── User.java
│   ├── Reservation.java
│   ├── VehicleInventory.java
│   ├── Store.java
│   └── VehicleRentalSystem.java
└── README.md
```

---

## 📜 Enums

```java
// VehicleType.java
public enum VehicleType {
    CAR, SUV, TRUCK, VAN
}

// Status.java
public enum Status {
    ACTIVE, INACTIVE
}

// ReservationType.java
public enum ReservationType {
    DAILY, HOURLY
}

// ReservationStatus.java
public enum ReservationStatus {
    SCHEDULED, ONGOING, CLOSED, CANCELLED
}
```

---

## 🚗 Vehicle

```java
public class Vehicle {
    int id;
    int vehNo;
    VehicleType type;
    int kmDriven;
    String companyName;
    String modelName;
    Date manfacDate;
    int cc;
    int dailyRentalCost;
    int hourlyRentalCost;
    int noOfSeats;
    Status st;

    public Vehicle(int id, int vehNo, VehicleType type, int kmDriven, String companyName,
                   String modelName, Date manfacDate, int cc, int dailyRentalCost,
                   int hourlyRentalCost, int noOfSeats, Status st) {
        this.id = id;
        this.vehNo = vehNo;
        this.type = type;
        this.kmDriven = kmDriven;
        this.companyName = companyName;
        this.modelName = modelName;
        this.manfacDate = manfacDate;
        this.cc = cc;
        this.dailyRentalCost = dailyRentalCost;
        this.hourlyRentalCost = hourlyRentalCost;
        this.noOfSeats = noOfSeats;
        this.st = st;
    }
}
```

---

## 📍 Location

```java
public class Location {
    String address;
    String city;
    String state;
    int pinCode;

    public Location(String address, String city, String state, int pinCode) {
        this.address = address;
        this.city = city;
        this.state = state;
        this.pinCode = pinCode;
    }
}
```

---

## 👤 User

```java
public class User {
    int id;
    String licence;
    String name;

    public User(int id, String licence, String name) {
        this.id = id;
        this.licence = licence;
        this.name = name;
    }
}
```

---

## 📦 VehicleInventory

```java
public class VehicleInventory {
    List<Vehicle> vehicles = new ArrayList<>();

    public void addVehicle(Vehicle v) {
        vehicles.add(v);
    }

    public void removeVehicle(int id) {
        vehicles.removeIf(v -> v.id == id);
    }

    public Vehicle getVehicleById(int id) {
        return vehicles.stream().filter(v -> v.id == id).findFirst().orElse(null);
    }

    public List<Vehicle> getAllVehicles() {
        return vehicles;
    }
}
```

---

## 📄 Reservation

```java
public class Reservation {
    int resId;
    User user;
    Vehicle vehicle;
    Date bookingDate;
    Date bookedFrom;
    Date bookedTo;
    Location pickup;
    Location drop;
    ReservationType rt;
    ReservationStatus rs;

    public Reservation(int resId, User user, Vehicle vehicle, Date bookedFrom, Date bookedTo,
                       Location pickup, Location drop, ReservationType rt) {
        this.resId = resId;
        this.user = user;
        this.vehicle = vehicle;
        this.bookingDate = new Date();
        this.bookedFrom = bookedFrom;
        this.bookedTo = bookedTo;
        this.pickup = pickup;
        this.drop = drop;
        this.rt = rt;
        this.rs = ReservationStatus.SCHEDULED;
    }

    public void updateStatus(ReservationStatus status) {
        this.rs = status;
    }
}
```

---

## 🏬 Store

```java
public class Store {
    int storeId;
    VehicleInventory vi = new VehicleInventory();
    Location loc;
    List<Reservation> reservations = new ArrayList<>();

    public Store(int storeId, Location loc) {
        this.storeId = storeId;
        this.loc = loc;
    }

    public void setVehicles(List<Vehicle> vehicles) {
        for (Vehicle v : vehicles) {
            vi.addVehicle(v);
        }
    }

    public Reservation createReservation(Vehicle v, User u, Date from, Date to,
                                         Location pickup, Location drop, ReservationType rt) {
        int resId = reservations.size() + 1;
        Reservation r = new Reservation(resId, u, v, from, to, pickup, drop, rt);
        reservations.add(r);
        return r;
    }

    public void completeReservation(int reservationId) {
        for (Reservation r : reservations) {
            if (r.resId == reservationId) {
                r.updateStatus(ReservationStatus.CLOSED);
                break;
            }
        }
    }

    public List<Vehicle> getAvailableVehicles() {
        return vi.getAllVehicles().stream()
                 .filter(v -> v.st == Status.ACTIVE)
                 .collect(Collectors.toList());
    }
}
```

---

## 🏢 VehicleRentalSystem

```java
public class VehicleRentalSystem {
    List<User> users = new ArrayList<>();
    List<Store> stores = new ArrayList<>();

    public void addUser(User u) {
        users.add(u);
    }

    public void addStore(Store s) {
        stores.add(s);
    }

    public User getUserById(int id) {
        return users.stream().filter(u -> u.id == id).findFirst().orElse(null);
    }

    public Store getStoreById(int id) {
        return stores.stream().filter(s -> s.storeId == id).findFirst().orElse(null);
    }

    public List<Store> getAllStores() {
        return stores;
    }

    public List<User> getAllUsers() {
        return users;
    }
}
```

---

Here’s your Snake & Ladder LLD revision guide formatted as a clean, professional README section — perfect for quick reference and revision:

---

# 🧠 Snake & Ladder LLD – Quick Revision Guide

## 🎮 Game Flow
- Players take turns rolling dice.
- They move forward by the dice value.
- If they land on a cell with a snake or ladder, they jump to the target cell.
- First player to reach cell 100 wins.

---

## 🧱 Board Structure
- 10×10 grid = 100 cells.
- Each cell may contain a `Jump` (snake or ladder).
- Board randomly places snakes and ladders during initialization.

---

## 🧩 Class Summary

### 1. `Player`
- **Fields**: `id`, `curPos`
- Tracks player identity and current position.

### 2. `Dice`
- **Field**: `diceCount`
- **Method**: `rollDice()` → returns total dice value.

### 3. `Jump`
- **Fields**: `start`, `end`
- Represents a snake (`start > end`) or ladder (`start < end`).

### 4. `Cell`
- **Field**: `Jump j`
- Each cell may contain a jump.

### 5. `Board`
- **Fields**: `Cell[][] cells`, `rows`, `cols`
- **Methods**:
  - `initializeCells()` → fills grid with empty cells.
  - `addSnakesAndLadders()` → randomly places jumps.
  - `getCell(pos)` → returns cell at linear position.
  - `getCoordinates(pos)` → converts linear to grid coordinates.
  - `getSize()` → returns total cells (100).

### 6. `Game`
- **Fields**: `Board`, `Dice`, `Deque<Player>`, `Player winner`
- **Methods**:
  - `addPlayers()` → adds players to queue.
  - `startGame()` → runs game loop until someone wins.

---

## 🧪 Sample Setup

```java
Game game = new Game(10, 10, 5, 5, 1); // 10x10 board, 5 snakes, 5 ladders, 1 dice
game.addPlayers(Arrays.asList("Alice", "Bob"));
game.startGame();
```

---

## ✅ Key Concepts to Remember
- `Jump` is used for both snakes and ladders.
- Board uses a 2D grid but maps positions linearly (1–100).
- Game uses a queue to rotate player turns.
- Dice rolls are skipped if they overshoot cell 100.
- Winner is declared when `curPos == 100`.

---

