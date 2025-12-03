# Hotel Management System

---

### 🔄 Hotel operations flow overview

1. Guest registers with basic details.
2. Admin adds rooms with type, price, and initial status AVAILABLE.
3. Guest requests a reservation for dates; system validates availability and creates a Reservation (CONFIRMED).
4. Guest checks in; room status moves to OCCUPIED.
5. Guest checks out; payment is processed; room returns to AVAILABLE; reservation closed.
6. Audit-like events can be logged or printed where useful (optional extension).

---

### 📦 Class responsibilities

| Class/Enum                  | Primary responsibility                                                                 |
|----------------------------|-----------------------------------------------------------------------------------------|
| HotelManagementSystem      | Singleton orchestrator for guests, rooms, reservations                                 |
| Guest                      | Represents a customer                                                                  |
| Room                       | Represents a hotel room with type, price, and status                                   |
| Reservation                | Links guest, room, dates, and status                                                   |
| Payment (interface)        | Contract to process payments                                                            |
| CreditCardPayment          | Implements payment via credit card                                                     |
| CashPayment                | Implements payment via cash                                                            |
| PaymentStatus (enum)       | INITIATED, SUCCESS, FAILED, CANCELLED                                                  |
| RoomType (enum)            | SINGLE, DOUBLE, DELUXE, SUITE                                                          |
| RoomStatus (enum)          | AVAILABLE, BOOKED, OCCUPIED                                                            |
| ReservationStatus (enum)   | CONFIRMED, CANCELLED                                                                   |
| PaymentGateway             | Simulates external payment success/failure                                             |

---

### 🧭 Booking and payment flow summary

1. Guest selects room and provides check-in/check-out dates.
2. System checks the room status; if AVAILABLE, creates a reservation and marks room BOOKED.
3. On check-in date, room transitions to OCCUPIED.
4. On check-out, payment is processed:
   - If SUCCESS: room becomes AVAILABLE; reservation treated as closed (set CANCELLED to indicate lifecycle end).
   - If FAILED: room stays OCCUPIED; retry or alternative payment required.


---

## 🧱 Core classes

### Guest.java
```java
public class Guest {
    private String id;
    private String name;
    private String email;
    private String phoneNumber;

    public Guest(String id, String name, String email, String phoneNumber) {
        this.id = id;
        this.name = name;
        this.email = email;
        this.phoneNumber = phoneNumber;
    }

    public String getId() { return id; }
    public void setId(String id) { this.id = id; }

    public String getName() { return name; }
    public void setName(String name) { this.name = name; }

    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }

    public String getPhoneNumber() { return phoneNumber; }
    public void setPhoneNumber(String phoneNumber) { this.phoneNumber = phoneNumber; }
}
```

---

### Room.java
```java
public class Room {
    private String id;
    private RoomType type;
    private double price;
    private RoomStatus status;

    private final Object lock = new Object();

    public Room(String id, RoomType type, double price) {
        this.id = id;
        this.type = type;
        this.price = price;
        this.status = RoomStatus.AVAILABLE;
    }

    public synchronized void checkIn() { this.status = RoomStatus.OCCUPIED; }
    public synchronized void book() { this.status = RoomStatus.BOOKED; }
    public synchronized void checkOut() { this.status = RoomStatus.AVAILABLE; }

    public String getId() { return id; }
    public void setId(String id) { this.id = id; }

    public RoomType getType() { return type; }
    public void setType(RoomType type) { this.type = type; }

    public double getPrice() { return price; }
    public void setPrice(double price) { this.price = price; }

    public RoomStatus getStatus() { return status; }
    public void setStatus(RoomStatus status) { this.status = status; }

    public Object getLock() { return lock; }
}
```

---

### Reservation.java
```java
import java.time.LocalDate;

public class Reservation {
    private String id;
    private Guest guest;
    private Room room;
    private LocalDate checkInDate;
    private LocalDate checkOutDate;
    private ReservationStatus status;

    public Reservation(String id, Guest guest, Room room,
                       LocalDate checkInDate, LocalDate checkOutDate) {
        this.id = id;
        this.guest = guest;
        this.room = room;
        this.checkInDate = checkInDate;
        this.checkOutDate = checkOutDate;
        this.status = ReservationStatus.CONFIRMED;
    }

    public synchronized void cancel() { this.status = ReservationStatus.CANCELLED; }

    public String getId() { return id; }
    public void setId(String id) { this.id = id; }

    public Guest getGuest() { return guest; }
    public void setGuest(Guest guest) { this.guest = guest; }

    public Room getRoom() { return room; }
    public void setRoom(Room room) { this.room = room; }

    public LocalDate getCheckInDate() { return checkInDate; }
    public void setCheckInDate(LocalDate checkInDate) { this.checkInDate = checkInDate; }

    public LocalDate getCheckOutDate() { return checkOutDate; }
    public void setCheckOutDate(LocalDate checkOutDate) { this.checkOutDate = checkOutDate; }

    public ReservationStatus getStatus() { return status; }
    public void setStatus(ReservationStatus status) { this.status = status; }
}
```

---

### RoomType.java
```java
public enum RoomType {
    SINGLE, DOUBLE, DELUXE, SUITE
}
```

---

### RoomStatus.java
```java
public enum RoomStatus {
    AVAILABLE, BOOKED, OCCUPIED
}
```

---

### ReservationStatus.java
```java
public enum ReservationStatus {
    CONFIRMED, CANCELLED
}
```

---

## 💳 Payment system

### Payment.java
```java
public interface Payment {
    PaymentStatus processPayment(double amount);
}
```

---

### PaymentStatus.java
```java
public enum PaymentStatus {
    INITIATED, SUCCESS, FAILED, CANCELLED
}
```

---

### PaymentGateway.java
```java
public class PaymentGateway {
    public static boolean charge(double amount, String channel) {
        // Simulate gateway variability; tweak as needed
        return Math.random() > 0.1;
    }
}
```

---

### CreditCardPayment.java
```java
public class CreditCardPayment implements Payment {
    @Override
    public PaymentStatus processPayment(double amount) {
        boolean ok = PaymentGateway.charge(amount, "CREDIT_CARD");
        return ok ? PaymentStatus.SUCCESS : PaymentStatus.FAILED;
    }
}
```

---

### CashPayment.java
```java
public class CashPayment implements Payment {
    @Override
    public PaymentStatus processPayment(double amount) {
        // Cash assumed successful upon check-out counter settlement
        return PaymentStatus.SUCCESS;
    }
}
```

---

## 🧠 Service

### HotelManagementSystem.java
```java
import java.time.LocalDate;
import java.util.*;

public class HotelManagementSystem {
    private static HotelManagementSystem instance;

    private final Map<String, Guest> guests = new HashMap<>();
    private final Map<String, Room> rooms = new HashMap<>();
    private final Map<String, Reservation> reservations = new HashMap<>();

    private HotelManagementSystem() {}

    public static synchronized HotelManagementSystem getInstance() {
        if (instance == null) {
            instance = new HotelManagementSystem();
        }
        return instance;
    }

    public void addGuest(Guest guest) {
        guests.put(guest.getId(), guest);
    }

    public Guest getGuest(String guestId) {
        return guests.get(guestId);
    }

    public void addRoom(Room room) {
        rooms.put(room.getId(), room);
    }

    public Room getRoom(String roomId) {
        return rooms.get(roomId);
    }

    public synchronized Reservation bookRoom(Guest guest, Room room,
                                             LocalDate checkInDate, LocalDate checkOutDate) {
        if (room == null) throw new IllegalArgumentException("Room cannot be null");
        if (guest == null) throw new IllegalArgumentException("Guest cannot be null");
        if (room.getStatus() != RoomStatus.AVAILABLE) {
            throw new IllegalStateException("Room not available");
        }

        String id = generateReservationId();
        Reservation reservation = new Reservation(id, guest, room, checkInDate, checkOutDate);
        reservations.put(id, reservation);
        room.book();
        return reservation;
    }

    public synchronized void cancelReservation(String reservationId) {
        Reservation res = reservations.get(reservationId);
        if (res == null) return;
        res.cancel();
        Room room = res.getRoom();
        if (room.getStatus() == RoomStatus.BOOKED) {
            room.setStatus(RoomStatus.AVAILABLE);
        }
    }

    public synchronized void checkIn(String reservationId) {
        Reservation res = reservations.get(reservationId);
        if (res == null) throw new NoSuchElementException("Reservation not found");
        Room room = res.getRoom();
        if (room.getStatus() != RoomStatus.BOOKED) {
            throw new IllegalStateException("Room must be BOOKED to check in");
        }
        room.checkIn();
    }

    public synchronized PaymentStatus checkOut(String reservationId, Payment payment) {
        Reservation res = reservations.get(reservationId);
        if (res == null) throw new NoSuchElementException("Reservation not found");

        Room room = res.getRoom();
        if (room.getStatus() != RoomStatus.OCCUPIED) {
            throw new IllegalStateException("Room must be OCCUPIED to check out");
        }

        double amount = room.getPrice(); // Simplified: flat price per reservation
        PaymentStatus status = payment.processPayment(amount);

        if (status == PaymentStatus.SUCCESS) {
            room.checkOut();
            res.setStatus(ReservationStatus.CANCELLED); // Lifecycle end marker
        }

        return status;
    }

    public String generateReservationId() {
        return UUID.randomUUID().toString();
    }

    public Reservation getReservation(String reservationId) {
        return reservations.get(reservationId);
    }

    public Collection<Reservation> listReservations() {
        return new ArrayList<>(reservations.values());
    }

    public Collection<Room> listRooms() {
        return new ArrayList<>(rooms.values());
    }

    public Collection<Guest> listGuests() {
        return new ArrayList<>(guests.values());
    }
}
```

---

## ▶️ Sample usage (Main.java)

```java
import java.time.LocalDate;

public class Main {
    public static void main(String[] args) {
        HotelManagementSystem hms = HotelManagementSystem.getInstance();

        // Add guest
        Guest guest = new Guest("G1", "Sriram", "sriram@example.com", "9876543210");
        hms.addGuest(guest);

        // Add rooms
        Room room101 = new Room("101", RoomType.DOUBLE, 2500.0);
        Room room102 = new Room("102", RoomType.SINGLE, 1800.0);
        hms.addRoom(room101);
        hms.addRoom(room102);

        // Book a room
        Reservation reservation = hms.bookRoom(
                guest, room101, LocalDate.now(), LocalDate.now().plusDays(2)
        );
        System.out.println("Reservation created: " + reservation.getId() + " | Status: " + reservation.getStatus());
        System.out.println("Room 101 status after booking: " + room101.getStatus());

        // Check-in
        hms.checkIn(reservation.getId());
        System.out.println("Room 101 status after check-in: " + room101.getStatus());

        // Check-out with payment
        Payment payment = new CreditCardPayment();
        var paymentStatus = hms.checkOut(reservation.getId(), payment);
        System.out.println("Payment status: " + paymentStatus);
        System.out.println("Room 101 status after check-out: " + room101.getStatus());
        System.out.println("Reservation status after check-out: " + reservation.getStatus());
    }
}
```

---

