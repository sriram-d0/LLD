

# 🚗 Parking Lot Management System – LLD 

A fully functional, object-oriented **Parking Lot Low-Level Design** written in Java.
Supports:

* Multi-floor parking
* Manhattan distance–based nearest spot allocation
* Two & four-wheeler support
* Ticket generation with automatic time tracking
* Payment processing (UPI / Debit Card)
* Clean and extensible architecture

---

## 📦 Features

### ✔ Vehicle System

Supports **Bike** and **Car**, both implementing a common `Vehicle` interface.

### ✔ Location System

Every spot and gate has a `Location(floor, x, y)`.
Distance is calculated using **Manhattan Distance**.

### ✔ Parking Spot System

Two implementations:

* `TwoWheelerParkingSpot`
* `FourWheelerParkingSpot`

Each spot contains:

* ID
* Location
* Price (20 or 40)
* Occupancy state

### ✔ Ticket System

A ticket stores:

* Entry time
* Exit time
* Vehicle
* Parking spot
* Total cost (calculated using hours * spot price)

### ✔ Payment System

Supports:

* `UPIPayment`
* `DebitCardPayment`

### ✔ Entry & Exit Gate System

* Entry assigns nearest free spot
* Exit calculates cost using time difference

### ✔ Parking Lot

Manages:

* All parking spots
* Entry & exit gates

---

# 📁 Full Code

Below is **complete Java code** for the entire LLD.

---

## 🚗 Vehicle System

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

## 📍 Location System

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

## 🅿️ Parking Spot System

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

## 🎫 Ticket System

```java
import java.time.Duration;
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

    public long getHours() {
        return Math.max(1, Duration.between(entryTime, exitTime).toHours());
    }

    public void calculateTotalCost() {
        this.totalCost = (int)(getHours() * parkingSpot.getPrice());
    }

    public int getTotalCost() {
        return totalCost;
    }

    public ParkingSpot getParkingSpot() { return parkingSpot; }
}
```

---

## 💳 Payment System

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

## 🚪 Entry Gate

```java
import java.util.List;

public interface EntryGate {
    ParkingSpot findParkingSpot(Vehicle v, List<ParkingSpot> spots, Location gateLocation);
    Ticket generateTicket(Vehicle v, ParkingSpot nearest);
}

public class MainEntryGate implements EntryGate {
    private int ticketCounter = 1;

    @Override
    public ParkingSpot findParkingSpot(Vehicle vehicle, List<ParkingSpot> spots, Location gateLocation) {
        ParkingSpot nearest = null;
        int min = Integer.MAX_VALUE;

        for (ParkingSpot spot : spots) {
            if (!spot.isEmpty() && spot.getVehicle() != null) continue;

            if (vehicle.getVehicleType() == VehicleType.TWO_WHEELER
                    && !(spot instanceof TwoWheelerParkingSpot)) continue;

            if (vehicle.getVehicleType() == VehicleType.FOUR_WHEELER
                    && !(spot instanceof FourWheelerParkingSpot)) continue;

            int d = spot.getLocation().distanceTo(gateLocation);
            if (d < min) {
                min = d;
                nearest = spot;
            }
        }
        return nearest;
    }

    @Override
    public Ticket generateTicket(Vehicle vehicle, ParkingSpot nearest) {
        nearest.parkVehicle(vehicle);
        return new Ticket(ticketCounter++, vehicle, nearest);
    }
}
```

---

## 🧾 Exit Gate

```java
import java.time.LocalDateTime;

public class MainExitGate {
    public int calculateCost(Ticket ticket) {
        ticket.setExitTime(LocalDateTime.now());
        ticket.calculateTotalCost();
        return ticket.getTotalCost();
    }

    public boolean processPayment(Payment method, int amount) {
        return method.pay(amount) == PaymentStatus.SUCCESS;
    }

    public void freeSpot(Ticket ticket) {
        ticket.getParkingSpot().removeVehicle();
    }
}
```

---

## 🅿️ Parking Lot

```java
import java.util.ArrayList;
import java.util.List;

public class ParkingLot {
    private List<ParkingSpot> spots = new ArrayList<>();
    private EntryGate entryGate = new MainEntryGate();
    private MainExitGate exitGate = new MainExitGate();
    private Location entryGateLocation = new Location(0,0,0);

    public void addSpot(ParkingSpot spot) {
        spots.add(spot);
    }

    public Ticket parkVehicle(Vehicle v) {
        ParkingSpot nearest = entryGate.findParkingSpot(v, spots, entryGateLocation);

        if (nearest == null) {
            System.out.println("No spot available!");
            return null;
        }

        return entryGate.generateTicket(v, nearest);
    }

    public void exitVehicle(Ticket ticket, Payment payment) {
        int amount = exitGate.calculateCost(ticket);
        boolean success = exitGate.processPayment(payment, amount);

        if (success) {
            exitGate.freeSpot(ticket);
            System.out.println("Payment Success! Amount = " + amount);
        } else {
            System.out.println("Payment Failed!");
        }
    }
}
```

---

# ▶ Sample Usage

```java
public class Main {
    public static void main(String[] args) throws InterruptedException {

        ParkingLot lot = new ParkingLot();

        lot.addSpot(new TwoWheelerParkingSpot(1, new Location(0, 1, 1)));
        lot.addSpot(new TwoWheelerParkingSpot(2, new Location(0, 2, 2)));
        lot.addSpot(new FourWheelerParkingSpot(3, new Location(0, 3, 3)));

        Vehicle v = new Car("AP39 1234");

        Ticket t = lot.parkVehicle(v);

        Thread.sleep(2000); // simulate time

        lot.exitVehicle(t, new UPIPayment());
    }
}
```

