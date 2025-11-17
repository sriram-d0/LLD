


# 🚗 Car Rental System 



### 📘Class Descriptions (Tabular Format)

| **Class Name**                                                      | **Role / Responsibility**                                                               | **Key Functions / Notes**                                                       |
| ------------------------------------------------------------------- | --------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------- |
| **Vehicle**                                                         | Represents a single vehicle in the system with its details, specs, pricing, and status. | Holds vehicle metadata → company, model, CC, seats, daily/hourly price, status. |
| **Location**                                                        | Stores address details used by both Stores and Reservations.                            | Contains address, city, state, pincode.                                         |
| **User**                                                            | Represents a customer using the rental system.                                          | Holds user ID, name, and driving license number.                                |
| **VehicleInventory**                                                | Manages all vehicles of a given store.                                                  | Add/remove vehicles, fetch by ID, list all vehicles.                            |
| **Reservation**                                                     | Represents a booking of a specific vehicle by a user for a time range.                  | Has reservation ID, user, vehicle, pickup/drop, booking dates, and status.      |
| **Store**                                                           | A physical rental branch where vehicles are stored and reservations are created.        | Holds inventory, location, reservations. Creates & completes reservations.      |
| **VehicleRentalSystem**                                             | Top-level system that manages users and stores.                                         | Add/lookup users, add/lookup stores, fetch lists.                               |
| **Enums (VehicleType, Status, ReservationType, ReservationStatus)** | Standardizes fixed values used across the system.                                       | Used for vehicle types, availability, reservation durations, and status.        |

---

# 🔄 **Rough Flow of the System**

(How the system behaves from start to end)

```
                    ┌────────────────────────────┐
                    │ VehicleRentalSystem (root) │
                    └──────────────┬─────────────┘
                                   │
             ┌─────────────────────┴────────────────────────┐
             │                                              │
      Add/Fetch Users                                Add/Fetch Stores
             │                                              │
             ▼                                              ▼
         ┌──────────┐                                ┌──────────┐
         │   User   │                                │  Store   │
         └──────────┘                                └─────┬────┘
                                                            │
                                       ┌────────────────────┴────────────────────┐
                                       │                                          │
                                   Inventory                                 Reservations
                                       │                                          │
                                       ▼                                          ▼
                            ┌──────────────────┐                        ┌──────────────────┐
                            │ VehicleInventory │                        │   Reservation    │
                            └───────┬──────────┘                        └──────────────────┘
                                    │
                              Holds Vehicles
                                    │
                                    ▼
                          ┌──────────────────┐
                          │    Vehicle       │
                          └──────────────────┘
```

---



# 🎯 **Ultra-short summary**

| System Part             | Purpose                                            |
| ----------------------- | -------------------------------------------------- |
| **VehicleRentalSystem** | Entry point → manages users + stores               |
| **Store**               | A rental branch → manages reservations + inventory |
| **VehicleInventory**    | Manages vehicles of one store                      |
| **Reservation**         | Actual booking object                              |
| **Vehicle**             | Car details                                        |
| **User**                | Customer info                                      |
| **Location**            | Shared address model                               |

---


# 🧩 Enums

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
````

---

# 🚗 Vehicle

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

# 📍 Location

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

# 👤 User

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

# 📦 VehicleInventory

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
        return vehicles.stream()
                       .filter(v -> v.id == id)
                       .findFirst()
                       .orElse(null);
    }

    public List<Vehicle> getAllVehicles() {
        return vehicles;
    }
}
```

---

# 📄 Reservation

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

# 🏬 Store (Constructor Injection)

```java
public class Store {
    int storeId;
    VehicleInventory vi;
    Location loc;
    List<Reservation> reservations = new ArrayList<>();

    public Store(int storeId, Location loc, VehicleInventory vi) {
        this.storeId = storeId;
        this.loc = loc;
        this.vi = vi;
    }

    public Reservation createReservation(Vehicle v, User u, Date from, Date to,
                                         Location pickup, Location drop, ReservationType rt) {
        int resId = reservations.size() + 1;
        Reservation r = new Reservation(resId, u, v, from, to, pickup, drop, rt);
        reservations.add(r);
        return r;
    }

    public void completeReservation(int id) {
        for (Reservation r : reservations) {
            if (r.resId == id) {
                r.updateStatus(ReservationStatus.CLOSED);
                break;
            }
        }
    }

    public List<Vehicle> getAvailableVehicles() {
        return vi.getAllVehicles()
                 .stream()
                 .filter(v -> v.st == Status.ACTIVE)
                 .collect(Collectors.toList());
    }
}
```

---

# 🏢 VehicleRentalSystem

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
        return users.stream()
                    .filter(u -> u.id == id)
                    .findFirst()
                    .orElse(null);
    }

    public Store getStoreById(int id) {
        return stores.stream()
                     .filter(s -> s.storeId == id)
                     .findFirst()
                     .orElse(null);
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

# ▶️ **Sample Usage – Main.java**

```java
import java.util.*;

public class Main {
    public static void main(String[] args) {

        VehicleRentalSystem system = new VehicleRentalSystem();

        // Users
        User u1 = new User(1, "AP12345", "Sriram");
        User u2 = new User(2, "TS99887", "Karthik");
        system.addUser(u1);
        system.addUser(u2);

        // Locations
        Location loc1 = new Location("MG Road", "Vijayawada", "AP", 520001);

        // Inventory
        VehicleInventory inv1 = new VehicleInventory();

        // Vehicles
        Vehicle v1 = new Vehicle(1, 5678, VehicleType.CAR, 12000, "Honda", "City",
                new Date(), 1498, 1800, 200, 5, Status.ACTIVE);

        Vehicle v2 = new Vehicle(2, 9999, VehicleType.SUV, 30000, "Toyota", "Fortuner",
                new Date(), 2755, 3500, 300, 7, Status.ACTIVE);

        Vehicle v3 = new Vehicle(3, 1234, VehicleType.CAR, 22000, "Hyundai", "i20",
                new Date(), 1200, 1500, 150, 5, Status.INACTIVE);

        inv1.addVehicle(v1);
        inv1.addVehicle(v2);
        inv1.addVehicle(v3);

        // Store with injected inventory
        Store store1 = new Store(101, loc1, inv1);
        system.addStore(store1);

        // Available Vehicles
        System.out.println("--- Available Vehicles at Store 1 ---");
        for (Vehicle v : store1.getAvailableVehicles()) {
            System.out.println(v.companyName + " " + v.modelName);
        }

        // Reservation Dates
        Calendar c = Calendar.getInstance();
        c.add(Calendar.DATE, 1);
        Date from = c.getTime();

        c.add(Calendar.DATE, 2);
        Date to = c.getTime();

        // Create Reservation
        Reservation res = store1.createReservation(
                v1, u1, from, to, loc1, loc1, ReservationType.DAILY
        );

        System.out.println("\nReservation Created: ID " + res.resId);

        // Complete Reservation
        store1.completeReservation(res.resId);
        System.out.println("After Completion: " + res.rs);
    }
}
```

---

