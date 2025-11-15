
# 📘 **Parking Lot Management System – Low Level Design (LLD)**

A clean, modular, scalable Parking Lot system built using core OOP principles.
Supports:

* Multi-floor parking
* Location-aware spot assignment
* Proximity-based parking logic (Manhattan Distance)
* Dynamic ticketing
* Time-based cost calculation
* Multiple payment methods
* Separate entry & exit gates
* Two-wheeler and four-wheeler spots

---

# 🏗️ **System Overview**

This Parking Lot LLD is divided into the following modules:

1. **Vehicle System**
2. **Location System (Manhattan Distance)**
3. **Parking Spot System**
4. **Ticket System**
5. **Payment System**
6. **Entry Gate System**
7. **Exit Gate System**
8. **Proximity Logic**

All modules work together to allow smooth entry → parking → exit → billing.

---

# 🚗 **1. Vehicle System**

```java
public enum VehicleType {
    TWO_WHEELER,
    FOUR_WHEELER
}

public interface Vehicle {
    String getVehicleNo();
    VehicleType getVehicleType();
}

public class Car implements Vehicle {
    private String vehicleNo;

    public Car(String vehicleNo) {
        this.vehicleNo = vehicleNo;
    }

    public String getVehicleNo() { return vehicleNo; }
    public VehicleType getVehicleType() { return VehicleType.FOUR_WHEELER; }
}

public class Bike implements Vehicle {
    private String vehicleNo;

    public Bike(String vehicleNo) {
        this.vehicleNo = vehicleNo;
    }

    public String getVehicleNo() { return vehicleNo; }
    public VehicleType getVehicleType() { return VehicleType.TWO_WHEELER; }
}
```

---

# 📍 **2. Location System (Manhattan Distance)**

Manhattan distance ensures the **closest** parking spot is chosen.

Formula:

```
|Δfloor| + |Δx| + |Δy|
```

```java
public class Location {
    private int floor;
    private int x;
    private int y;

    public Location(int floor, int x, int y) {
        this.floor = floor;
        this.x = x;
        this.y = y;
    }

    public int distanceTo(Location other) {
        return Math.abs(floor - other.floor)
                + Math.abs(x - other.x)
                + Math.abs(y - other.y);
    }
}
```

---

# 🅿️ **3. Parking Spot System**

Two types of spots, each with a price per hour.

```java
public interface ParkingSpot {
    int getId();
    boolean isEmpty();
    int getPrice();
    Location getLocation();
    Vehicle getVehicle();
    void parkVehicle(Vehicle v);
    void removeVehicle();
}
```

### Two-wheeler spot

```java
public class TwoWheelerParkingSpot implements ParkingSpot {
    private int id;
    private Vehicle vehicle;
    private Location location;
    private final int price = 20;

    public TwoWheelerParkingSpot(int id, Location location) {
        this.id = id;
        this.location = location;
    }

    public int getId() { return id; }
    public boolean isEmpty() { return vehicle == null; }
    public int getPrice() { return price; }
    public Location getLocation() { return location; }
    public Vehicle getVehicle() { return vehicle; }

    public void parkVehicle(Vehicle v) { this.vehicle = v; }
    public void removeVehicle() { this.vehicle = null; }
}
```

### Four-wheeler spot

```java
public class FourWheelerParkingSpot implements ParkingSpot {
    private int id;
    private Vehicle vehicle;
    private Location location;
    private final int price = 40;

    public FourWheelerParkingSpot(int id, Location location) {
        this.id = id;
        this.location = location;
    }

    public int getId() { return id; }
    public boolean isEmpty() { return vehicle == null; }
    public int getPrice() { return price; }
    public Location getLocation() { return location; }
    public Vehicle getVehicle() { return vehicle; }

    public void parkVehicle(Vehicle v) { this.vehicle = v; }
    public void removeVehicle() { this.vehicle = null; }
}
```

---

# 🎫 **4. Ticket System**

Ticket stores entry time, exit time, cost, vehicle, and spot.

```java
import java.time.LocalDateTime;

public class Ticket {
    private int ticketId;
    private LocalDateTime entryTime;
    private LocalDateTime exitTime;
    private ParkingSpot parkingSpot;
    private Vehicle vehicle;
    private int totalCost;

    public Ticket(int ticketId, Vehicle vehicle, ParkingSpot spot) {
        this.ticketId = ticketId;
        this.vehicle = vehicle;
        this.parkingSpot = spot;
        this.entryTime = LocalDateTime.now();
    }

    public void setExitTime(LocalDateTime exitTime) {
        this.exitTime = exitTime;
    }

    public void setTotalCost(int cost) {
        this.totalCost = cost;
    }

    public int getTicketId() { return ticketId; }
    public LocalDateTime getEntryTime() { return entryTime; }
    public LocalDateTime getExitTime() { return exitTime; }
    public int getTotalCost() { return totalCost; }
    public ParkingSpot getParkingSpot() { return parkingSpot; }
    public Vehicle getVehicle() { return vehicle; }
}
```

---

# 💳 **5. Payment System**

```java
public enum PaymentStatus {
    SUCCESS,
    FAILED
}

public interface Payment {
    PaymentStatus pay(int amount);
}

public class DebitCardPayment implements Payment {
    public PaymentStatus pay(int amount) {
        return PaymentStatus.SUCCESS;
    }
}

public class UPIPayment implements Payment {
    public PaymentStatus pay(int amount) {
        return PaymentStatus.SUCCESS;
    }
}
```

---

# 🚪 **6. Entry Gate (Proximity-based Spot Selection)**

Entry gate picks the **nearest suitable available spot**.

```java
import java.util.List;

public interface EntryGate {
    ParkingSpot findParkingSpot(Vehicle vehicle, List<ParkingSpot> spots, Location gateLocation);
    void updateParkingSpot(ParkingSpot spot, boolean isOccupied);
    Ticket generateTicket(Vehicle vehicle, ParkingSpot spot);
}

public class MainEntryGate implements EntryGate {
    private int ticketCounter = 1;

    @Override
    public ParkingSpot findParkingSpot(Vehicle vehicle, List<ParkingSpot> spots, Location gateLocation) {
        ParkingSpot nearest = null;
        int minDistance = Integer.MAX_VALUE;

        for (ParkingSpot spot : spots) {
            if (spot.isEmpty() && matchesVehicleType(spot, vehicle)) {
                int distance = spot.getLocation().distanceTo(gateLocation);
                if (distance < minDistance) {
                    minDistance = distance;
                    nearest = spot;
                }
            }
        }

        return nearest;
    }

    private boolean matchesVehicleType(ParkingSpot spot, Vehicle vehicle) {
        if (vehicle.getVehicleType() == VehicleType.TWO_WHEELER &&
            spot instanceof TwoWheelerParkingSpot) return true;

        if (vehicle.getVehicleType() == VehicleType.FOUR_WHEELER &&
            spot instanceof FourWheelerParkingSpot) return true;

        return false;
    }

    @Override
    public Ticket generateTicket(Vehicle vehicle, ParkingSpot spot) {
        return new Ticket(ticketCounter++, vehicle, spot);
    }

    @Override
    public void updateParkingSpot(ParkingSpot spot, boolean isOccupied) {
        if (!isOccupied) spot.removeVehicle();
    }
}
```

---

# 🧾 **7. Exit Gate (Dynamic Hourly Billing)**

Uses `Duration.between(entry, exit)`.

```java
import java.time.Duration;
import java.time.LocalDateTime;

public class MainExitGate implements ExitGate {

    @Override
    public int calculateCost(Ticket ticket) {
        LocalDateTime entry = ticket.getEntryTime();
        LocalDateTime exit = ticket.getExitTime();

        if (exit == null) {
            exit = LocalDateTime.now();
            ticket.setExitTime(exit);
        }

        long minutes = Duration.between(entry, exit).toMinutes();
        long hours = Math.max(1, (minutes + 59) / 60);  // ceil(minutes/60)

        int pricePerHour = ticket.getParkingSpot().getPrice();
        return (int)(hours * pricePerHour);
    }

    @Override
    public boolean processPayment(Payment payment, int amount) {
        return payment.pay(amount) == PaymentStatus.SUCCESS;
    }

    @Override
    public void updateParkingSpot(ParkingSpot spot, boolean isEmpty) {
        if (isEmpty) {
            spot.removeVehicle();
        }
    }
}
```

---

# 🧠 **8. Proximity Logic**

```java
int distance = spot.getLocation().distanceTo(entryGateLocation);
```

Nearest empty spot → chosen for parking.

---

# 🎯 **Sample Parking Flow**

1️⃣ A bike comes to entry gate
2️⃣ Entry gate finds nearest 2-wheeler spot
3️⃣ Create ticket with entry time
4️⃣ When vehicle exits, exit time is recorded
5️⃣ Duration calculated → cost = hours × price/hour
6️⃣ Payment processed
7️⃣ Spot becomes empty again


