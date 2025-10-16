
# 🚗 Parking Lot Management System

This project is a modular and extensible Parking Lot Management System designed to handle multi-floor parking with entry/exit gates, vehicle tracking, ticketing, and payment processing. It supports proximity-based parking allocation and multiple vehicle types.

---

## 📦 Features

- Multi-floor parking with location-aware spot assignment
- Entry and exit gate logic with ticket generation and cost calculation
- Support for Two-Wheeler and Four-Wheeler vehicles
- Modular payment system (Debit Card, Credit Card, UPI)
- Extensible design using interfaces and enums
- Proximity-based parking spot selection

---

## 🧱 Architecture Overview

### 🔹 Core Interfaces

| Interface      | Purpose |
|----------------|---------|
| `Vehicle`      | Represents any vehicle (Car, Bike, Auto) |
| `ParkingSpot`  | Abstracts a parking spot with location, price, and occupancy |
| `Payment`      | Handles payment logic for different methods |
| `EntryGate`    | Manages entry operations: spot allocation, ticketing |
| `ExitGate`     | Manages exit operations: cost calculation, payment |

---

### 🔹 Key Classes

#### 🚘 Vehicle Types
- `Car`, `Bike`, `Auto` implement `Vehicle`
- `VehicleType` enum: `TWO_WHEELER`, `FOUR_WHEELER`

#### 🅿️ Parking Spots
- `TwoWheelerParkingSpot`, `FourWheelerParkingSpot` implement `ParkingSpot`
- Each has `id`, `location`, `price`, and occupancy status

#### 📍 Location
- `Location` class with `floor`, `x`, `y`
- Method `distanceTo(Location other)` for proximity calculation

#### 🎫 Ticket
- Tracks `entryTime`, `exitTime`, `parkingSpot`, `vehicle`, and `totalCost`

#### 💳 Payment Methods
- `DebitCardPayment`, `CreditCardPayment`, `UPIPayment` implement `Payment`
- `PaymentStatus` enum: `SUCCESS`, `FAILED`

#### 🚪 Gates
- `MainEntryGate` implements `EntryGate`
  - Finds nearest available spot based on vehicle type and gate location
  - Generates ticket
- `ExitGate` interface (implementation pending)
  - Calculates cost and processes payment

---

## 🧠 Proximity Logic

Parking spots are assigned based on the shortest Manhattan distance to the entry gate:

```java
int distance = spot.getLocation().distanceTo(entryGateLocation);
