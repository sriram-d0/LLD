


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

