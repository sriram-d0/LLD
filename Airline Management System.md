# Airline Management System
Alright, let’s stitch everything together into a **flow** and then show you a **sample usage in Java** so you can see how the classes interact in practice.  

---

## 🔄 Typical Booking Flow

1. **Search flights**  
   - User searches for flights between origin and destination.  
   - System returns available flights with fares.

2. **Create itinerary**  
   - User selects one or more flights (segments).  
   - An `Itinerary` object is created with chosen segments.

3. **Add passengers**  
   - User enters passenger details.  
   - `Passenger` objects are created.

4. **Create booking**  
   - A `Booking` object is created with itinerary + passengers.  
   - Status = `PENDING`.

5. **Payment**  
   - A `Payment` object is initiated and captured.  
   - If successful, booking status = `CONFIRMED`.

6. **Issue tickets**  
   - `Ticket` objects are generated for each passenger.  

7. **Check-in & boarding pass**  
   - Passengers check in, seats assigned.  
   - `BoardingPass` objects are created.




---
## 📦 Enums

```java
public enum CabinClass { ECONOMY, PREMIUM_ECONOMY, BUSINESS, FIRST }
public enum BookingStatus { PENDING, CONFIRMED, CANCELLED }
public enum PaymentStatus { INITIATED, AUTHORIZED, CAPTURED, FAILED, REFUNDED }
public enum FlightStatus { SCHEDULED, DELAYED, DEPARTED, ARRIVED, CANCELLED }
```

---

## 📦 Domain Classes

### Airport
```java
public class Airport {
    private String code;
    private String name;
    private String city;

    public Airport(String code, String name, String city) {
        this.code = code;
        this.name = name;
        this.city = city;
    }

    public String getCode() { return code; }
    public void setCode(String code) { this.code = code; }

    public String getName() { return name; }
    public void setName(String name) { this.name = name; }

    public String getCity() { return city; }
    public void setCity(String city) { this.city = city; }
}
```

---

### Route
```java
public class Route {
    private Airport origin;
    private Airport destination;

    public Route(Airport origin, Airport destination) {
        this.origin = origin;
        this.destination = destination;
    }

    public Airport getOrigin() { return origin; }
    public void setOrigin(Airport origin) { this.origin = origin; }

    public Airport getDestination() { return destination; }
    public void setDestination(Airport destination) { this.destination = destination; }
}
```

---

### Aircraft
```java
public class Aircraft {
    private String tailNumber;
    private String model;
    private int seatCapacity;

    public Aircraft(String tailNumber, String model, int seatCapacity) {
        this.tailNumber = tailNumber;
        this.model = model;
        this.seatCapacity = seatCapacity;
    }

    public String getTailNumber() { return tailNumber; }
    public void setTailNumber(String tailNumber) { this.tailNumber = tailNumber; }

    public String getModel() { return model; }
    public void setModel(String model) { this.model = model; }

    public int getSeatCapacity() { return seatCapacity; }
    public void setSeatCapacity(int seatCapacity) { this.seatCapacity = seatCapacity; }
}
```

---

### Flight
```java
import java.time.LocalDateTime;

public class Flight {
    private String flightNumber;
    private Route route;
    private LocalDateTime departureTime;
    private LocalDateTime arrivalTime;
    private Aircraft aircraft;
    private FlightStatus status;

    public Flight(String flightNumber, Route route, LocalDateTime departureTime,
                  LocalDateTime arrivalTime, Aircraft aircraft, FlightStatus status) {
        this.flightNumber = flightNumber;
        this.route = route;
        this.departureTime = departureTime;
        this.arrivalTime = arrivalTime;
        this.aircraft = aircraft;
        this.status = status;
    }

    public String getFlightNumber() { return flightNumber; }
    public void setFlightNumber(String flightNumber) { this.flightNumber = flightNumber; }

    public Route getRoute() { return route; }
    public void setRoute(Route route) { this.route = route; }

    public LocalDateTime getDepartureTime() { return departureTime; }
    public void setDepartureTime(LocalDateTime departureTime) { this.departureTime = departureTime; }

    public LocalDateTime getArrivalTime() { return arrivalTime; }
    public void setArrivalTime(LocalDateTime arrivalTime) { this.arrivalTime = arrivalTime; }

    public Aircraft getAircraft() { return aircraft; }
    public void setAircraft(Aircraft aircraft) { this.aircraft = aircraft; }

    public FlightStatus getStatus() { return status; }
    public void setStatus(FlightStatus status) { this.status = status; }
}
```

---

### Passenger
```java
public class Passenger {
    private String firstName;
    private String lastName;
    private String email;
    private String phone;

    public Passenger(String firstName, String lastName, String email, String phone) {
        this.firstName = firstName;
        this.lastName = lastName;
        this.email = email;
        this.phone = phone;
    }

    public String getFirstName() { return firstName; }
    public void setFirstName(String firstName) { this.firstName = firstName; }

    public String getLastName() { return lastName; }
    public void setLastName(String lastName) { this.lastName = lastName; }

    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }

    public String getPhone() { return phone; }
    public void setPhone(String phone) { this.phone = phone; }
}
```

---

### Fare
```java
public class Fare {
    private CabinClass cabinClass;
    private String fareCode;
    private String currency;
    private long basePriceMinor;

    public Fare(CabinClass cabinClass, String fareCode, String currency, long basePriceMinor) {
        this.cabinClass = cabinClass;
        this.fareCode = fareCode;
        this.currency = currency;
        this.basePriceMinor = basePriceMinor;
    }

    public CabinClass getCabinClass() { return cabinClass; }
    public void setCabinClass(CabinClass cabinClass) { this.cabinClass = cabinClass; }

    public String getFareCode() { return fareCode; }
    public void setFareCode(String fareCode) { this.fareCode = fareCode; }

    public String getCurrency() { return currency; }
    public void setCurrency(String currency) { this.currency = currency; }

    public long getBasePriceMinor() { return basePriceMinor; }
    public void setBasePriceMinor(long basePriceMinor) { this.basePriceMinor = basePriceMinor; }
}
```

---

### Booking
```java
import java.util.List;
import java.util.UUID;

public class Booking {
    private UUID id;
    private Flight flight;
    private List<Passenger> passengers;
    private BookingStatus status;

    public Booking(UUID id, Flight flight, List<Passenger> passengers, BookingStatus status) {
        this.id = id;
        this.flight = flight;
        this.passengers = passengers;
        this.status = status;
    }

    public UUID getId() { return id; }
    public void setId(UUID id) { this.id = id; }

    public Flight getFlight() { return flight; }
    public void setFlight(Flight flight) { this.flight = flight; }

    public List<Passenger> getPassengers() { return passengers; }
    public void setPassengers(List<Passenger> passengers) { this.passengers = passengers; }

    public BookingStatus getStatus() { return status; }
    public void setStatus(BookingStatus status) { this.status = status; }
}
```

---

### Payment
```java
public class Payment {
    private String paymentId;
    private long amountMinor;
    private String currency;
    private PaymentStatus status;

    public Payment(String paymentId, long amountMinor, String currency, PaymentStatus status) {
        this.paymentId = paymentId;
        this.amountMinor = amountMinor;
        this.currency = currency;
        this.status = status;
    }

    public String getPaymentId() { return paymentId; }
    public void setPaymentId(String paymentId) { this.paymentId = paymentId; }

    public long getAmountMinor() { return amountMinor; }
    public void setAmountMinor(long amountMinor) { this.amountMinor = amountMinor; }

    public String getCurrency() { return currency; }
    public void setCurrency(String currency) { this.currency = currency; }

    public PaymentStatus getStatus() { return status; }
    public void setStatus(PaymentStatus status) { this.status = status; }
}
```

---

### Ticket
```java
public class Ticket {
    private String ticketNumber;
    private Passenger passenger;
    private Flight flight;
    private String seatNumber;

    public Ticket(String ticketNumber, Passenger passenger, Flight flight, String seatNumber) {
        this.ticketNumber = ticketNumber;
        this.passenger = passenger;
        this.flight = flight;
        this.seatNumber = seatNumber;
    }

    public String getTicketNumber() { return ticketNumber; }
    public void setTicketNumber(String ticketNumber) { this.ticketNumber = ticketNumber; }

    public Passenger getPassenger() { return passenger; }
    public void setPassenger(Passenger passenger) { this.passenger = passenger; }

    public Flight getFlight() { return flight; }
    public void setFlight(Flight flight) { this.flight = flight; }

    public String getSeatNumber() { return seatNumber; }
    public void setSeatNumber(String seatNumber) { this.seatNumber = seatNumber; }
}
```

---



## BoardingPass 

```java
public class BoardingPass {
    private String bpId;
    private Passenger passenger;
    private Flight flight;
    private String seatNumber;
    private String gate;

    public BoardingPass(String bpId, Passenger passenger, Flight flight, String seatNumber, String gate) {
        this.bpId = bpId;
        this.passenger = passenger;
        this.flight = flight;
        this.seatNumber = seatNumber;
        this.gate = gate;
    }

    public String getBpId() { return bpId; }
    public void setBpId(String bpId) { this.bpId = bpId; }

    public Passenger getPassenger() { return passenger; }
    public void setPassenger(Passenger passenger) { this.passenger = passenger; }

    public Flight getFlight() { return flight; }
    public void setFlight(Flight flight) { this.flight = flight; }

    public String getSeatNumber() { return seatNumber; }
    public void setSeatNumber(String seatNumber) { this.seatNumber = seatNumber; }

    public String getGate() { return gate; }
    public void setGate(String gate) { this.gate = gate; }
}
```

---

## Seat
```java
public class Seat {
    private String seatNumber;
    private CabinClass cabinClass;
    private boolean isExitRow;
    private boolean isExtraLegroom;

    public Seat(String seatNumber, CabinClass cabinClass, boolean isExitRow, boolean isExtraLegroom) {
        this.seatNumber = seatNumber;
        this.cabinClass = cabinClass;
        this.isExitRow = isExitRow;
        this.isExtraLegroom = isExtraLegroom;
    }

    public String getSeatNumber() { return seatNumber; }
    public void setSeatNumber(String seatNumber) { this.seatNumber = seatNumber; }

    public CabinClass getCabinClass() { return cabinClass; }
    public void setCabinClass(CabinClass cabinClass) { this.cabinClass = cabinClass; }

    public boolean isExitRow() { return isExitRow; }
    public void setExitRow(boolean exitRow) { isExitRow = exitRow; }

    public boolean isExtraLegroom() { return isExtraLegroom; }
    public void setExtraLegroom(boolean extraLegroom) { isExtraLegroom = extraLegroom; }
}
```

---

## Itinerary and FlightSegment
```java
import java.util.List;
import java.util.UUID;

public class Itinerary {
    private UUID id;
    private List<FlightSegment> segments;

    public Itinerary(UUID id, List<FlightSegment> segments) {
        this.id = id;
        this.segments = segments;
    }

    public UUID getId() { return id; }
    public void setId(UUID id) { this.id = id; }

    public List<FlightSegment> getSegments() { return segments; }
    public void setSegments(List<FlightSegment> segments) { this.segments = segments; }
}

public class FlightSegment {
    private Flight flight;
    private CabinClass cabinClass;
    private Fare fare;

    public FlightSegment(Flight flight, CabinClass cabinClass, Fare fare) {
        this.flight = flight;
        this.cabinClass = cabinClass;
        this.fare = fare;
    }

    public Flight getFlight() { return flight; }
    public void setFlight(Flight flight) { this.flight = flight; }

    public CabinClass getCabinClass() { return cabinClass; }
    public void setCabinClass(CabinClass cabinClass) { this.cabinClass = cabinClass; }

    public Fare getFare() { return fare; }
    public void setFare(Fare fare) { this.fare = fare; }
}
```

---

## 🧑‍💻 Sample Usage in Java

Here’s a simple **main method** that simulates the flow:

```java
import java.time.LocalDateTime;
import java.util.*;

public class AirlineSystemDemo {
    public static void main(String[] args) {
        // Step 1: Create airports and route
        Airport hyd = new Airport("HYD", "Rajiv Gandhi Intl", "Hyderabad");
        Airport del = new Airport("DEL", "Indira Gandhi Intl", "Delhi");
        Route route = new Route(hyd, del);

        // Step 2: Create aircraft and flight
        Aircraft aircraft = new Aircraft("TN-123", "Airbus A320", 180);
        Flight flight = new Flight("AI-560", route,
                LocalDateTime.of(2025, 12, 20, 9, 0),
                LocalDateTime.of(2025, 12, 20, 11, 0),
                aircraft, FlightStatus.SCHEDULED);

        // Step 3: Create fare and segment
        Fare fare = new Fare(CabinClass.ECONOMY, "Y", "INR", 5000);
        FlightSegment segment = new FlightSegment(flight, CabinClass.ECONOMY, fare);
        Itinerary itinerary = new Itinerary(UUID.randomUUID(), List.of(segment));

        // Step 4: Add passenger
        Passenger passenger = new Passenger("Uday", "Dutta", "uday@example.com", "9876543210");

        // Step 5: Create booking
        Booking booking = new Booking(UUID.randomUUID(), flight,
                List.of(passenger), BookingStatus.PENDING);
        System.out.println("Booking created: " + booking.getId() + " Status: " + booking.getStatus());

        // Step 6: Payment
        Payment payment = new Payment("PAY123", fare.getBasePriceMinor(), "INR", PaymentStatus.CAPTURED);
        booking.setStatus(BookingStatus.CONFIRMED);
        System.out.println("Payment successful. Booking confirmed!");

        // Step 7: Issue ticket
        Ticket ticket = new Ticket("TICK123", passenger, flight, "12A");
        System.out.println("Ticket issued: " + ticket.getTicketNumber() + " Seat: " + ticket.getSeatNumber());

        // Step 8: Boarding pass
        BoardingPass bp = new BoardingPass("BP123", passenger, flight, "12A", "Gate 5");
        System.out.println("Boarding pass generated for " + bp.getPassenger().getFirstName() +
                " at " + bp.getGate());
    }
}
```
