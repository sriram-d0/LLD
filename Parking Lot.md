# 🚗 Parking Lot Management System

A **modular and extensible Parking Lot Management System** designed to manage multi-floor parking with entry/exit gates, vehicle tracking, ticketing, and payment processing.  
It supports proximity-based parking allocation and multiple vehicle types.

---

## 📦 Features

- 🏢 Multi-floor parking with location-aware spot assignment  
- 🚪 Entry and exit gate logic with ticket generation and cost calculation  
- 🚗 Supports Two-Wheeler and Four-Wheeler vehicles  
- 💳 Modular payment system (Debit Card, Credit Card, UPI)  
- ⚙️ Extensible architecture using interfaces and enums  
- 📍 Proximity-based parking spot selection  

---

## 🧱 Architecture Overview

| Interface | Purpose |
|------------|----------|
| `Vehicle` | Represents any vehicle (Car, Bike, Auto) |
| `ParkingSpot` | Abstracts a parking spot with location, price, and occupancy |
| `Payment` | Handles payment logic for different methods |
| `EntryGate` | Manages entry operations: spot allocation, ticketing |
| `ExitGate` | Manages exit operations: cost calculation, payment |

---

## 📁 Java Code

```java
// ===========================
// 🚗 Vehicle System
// ===========================

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

// ===========================
// 📍 Location System
// ===========================

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
        return Math.abs(floor - other.floor) + Math.abs(x - other.x) + Math.abs(y - other.y);
    }
}

// ===========================
// 🅿️ Parking Spot System
// ===========================

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

// ===========================
// 🎫 Ticket System
// ===========================

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

    // Getters omitted for brevity
}

// ===========================
// 💳 Payment System
// ===========================

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

// ===========================
// 🚪 Entry Gate System
// ===========================

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
        if (vehicle.getVehicleType() == VehicleType.TWO_WHEELER && spot instanceof TwoWheelerParkingSpot) return true;
        if (vehicle.getVehicleType() == VehicleType.FOUR_WHEELER && spot instanceof FourWheelerParkingSpot) return true;
        return false;
    }

    @Override
    public void updateParkingSpot(ParkingSpot spot, boolean isOccupied) {
        if (isOccupied) {
            spot.parkVehicle(spot.getVehicle());
        } else {
            spot.removeVehicle();
        }
    }

    @Override
    public Ticket generateTicket(Vehicle vehicle, ParkingSpot spot) {
        return new Ticket(ticketCounter++, vehicle, spot);
    }
}

// ===========================
// 🧾 Exit Gate System
// ===========================

public interface ExitGate {
    int calculateCost(Ticket ticket);
    boolean processPayment(Payment payment, int amount);
    void updateParkingSpot(ParkingSpot spot, boolean isEmpty);
}

// ===========================
// 🧠 Proximity Logic
// ===========================

// Parking spots are assigned based on the shortest Manhattan distance
// to the entry gate. This ensures vehicles are parked as close as possible.

int distance = spot.getLocation().distanceTo(entryGateLocation);
