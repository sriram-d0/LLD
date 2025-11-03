
## 🎬 Movie Ticket Booking System 

### 🔧 Purpose
A modular, concurrency-safe Java application that simulates an online movie ticket booking platform. It handles:
- Movie and theatre management
- Show scheduling
- Seat selection
- Payment processing
- Thread-safe booking to prevent double-booking

---







### 🧩 Core Entities

| Class         | Role                                                                 |
|--------------|----------------------------------------------------------------------|
| `User`        | Represents a customer                                                |
| `City`        | Location context for theatres and movies                            |
| `Movie`       | Metadata: name, duration, genre, language                           |
| `Theatre`     | Physical venue with screens and shows                               |
| `Screen`      | Contains seats; linked to shows                                     |
| `Seat`        | Individual seat with category and booking status                    |
| `Show`        | Scheduled movie on a screen at a time; tracks booked seat IDs       |

---

### 💳 Payment System

| Class / Enum       | Role                                                                 |
|--------------------|----------------------------------------------------------------------|
| `PaymentMethod`     | Enum: CREDIT_CARD, DEBIT_CARD, UPI, etc.                            |
| `PaymentStatus`     | Enum: INITIATED, SUCCESS, FAILED, CANCELLED                         |
| `Payment`           | Handles transaction logic, timestamps, and status                   |
| `PaymentGateway`    | Simulates external payment success/failure                          |

---

### 🎟️ Booking System

| Class / Enum       | Role                                                                 |
|--------------------|----------------------------------------------------------------------|
| `BookingStatus`     | Enum: PENDING, CONFIRMED, CANCELLED                                 |
| `Booking`           | Links user, show, seats, and payment                                |
| `BookingService`    | Core logic: validates seat availability, processes payment, confirms booking |
| `synchronized` block | Ensures thread-safe booking (no double-booking)                    |

---

### 🎬 Controllers

| Controller         | Role                                                                 |
|--------------------|----------------------------------------------------------------------|
| `MovieController`   | Manages movies per city; search by name                             |
| `TheatreController` | Manages theatres and shows per city                                 |

---

### 🔁 Booking Flow Summary

1. **User selects movie + city**
2. **System fetches theatres and shows**
3. **User selects show + seats**
4. **BookingService checks seat availability**
5. **Payment is processed**
6. **Booking is confirmed if payment succeeds**





---
## 📂 Folder Structure (Suggested)

```
src/
├── model/
│   ├── User.java
│   ├── Movie.java
│   ├── City.java
│   ├── Seat.java
│   ├── Screen.java
│   ├── Show.java
│   ├── Theatre.java
│   ├── Booking.java
│   ├── Payment.java
│   └── enums/
│       ├── SeatCategory.java
│       ├── PaymentMethod.java
│       ├── PaymentStatus.java
│       └── BookingStatus.java
├── controller/
│   ├── MovieController.java
│   └── TheatreController.java
├── service/
│   ├── BookingService.java
│   └── PaymentGateway.java
```
---
## Codes

### ✅ `User.java`
```java
public class User {
    private int id;
    private String name;

    public User(int id, String name) {
        this.id = id;
        this.name = name;
    }

    // Getters and setters
}
```

---

### ✅ `City.java`
```java
public class City {
    private int id;
    private String name;

    public City(int id, String name) {
        this.id = id;
        this.name = name;
    }

    // Getters and setters
}
```

---

### ✅ `Movie.java`
```java
public class Movie {
    private int id;
    private String movieName;
    private int duration;
    private String genre;
    private String language;

    public Movie(int id, String movieName, int duration, String genre, String language) {
        this.id = id;
        this.movieName = movieName;
        this.duration = duration;
        this.genre = genre;
        this.language = language;
    }

    // Getters and setters
}
```

---

### ✅ `SeatCategory.java`
```java
public enum SeatCategory {
    PLATINUM, DIAMOND, GOLD, SILVER
}
```

---

### ✅ `Seat.java`
```java
public class Seat {
    private int id;
    private int row;
    private SeatCategory category;
    private boolean isBooked;

    public Seat(int id, int row, SeatCategory category) {
        this.id = id;
        this.row = row;
        this.category = category;
        this.isBooked = false;
    }

    // Getters and setters
}
```

---

### ✅ `Screen.java`
```java
import java.util.List;

public class Screen {
    private int id;
    private List<Seat> seats;

    public Screen(int id, List<Seat> seats) {
        this.id = id;
        this.seats = seats;
    }

    // Getters and setters
}
```

---

### ✅ `Show.java`
```java
import java.time.LocalDateTime;
import java.util.Set;
import java.util.concurrent.ConcurrentHashMap;

public class Show {
    private int id;
    private Movie movie;
    private Screen screen;
    private LocalDateTime startTime;
    private Set<Integer> bookedSeatIds = ConcurrentHashMap.newKeySet();

    public Show(int id, Movie movie, Screen screen, LocalDateTime startTime) {
        this.id = id;
        this.movie = movie;
        this.screen = screen;
        this.startTime = startTime;
    }

    public boolean isSeatAvailable(int seatId) {
        return !bookedSeatIds.contains(seatId);
    }

    public void bookSeats(List<Integer> seatIds) {
        bookedSeatIds.addAll(seatIds);
    }

    // Getters and setters
}
```

---

### ✅ `Theatre.java`
```java
import java.util.List;

public class Theatre {
    private int id;
    private String address;
    private City city;
    private List<Show> shows;
    private List<Screen> screens;

    public Theatre(int id, String address, City city, List<Show> shows, List<Screen> screens) {
        this.id = id;
        this.address = address;
        this.city = city;
        this.shows = shows;
        this.screens = screens;
    }

    // Getters and setters
}
```

---

### ✅ `PaymentMethod.java`
```java
public enum PaymentMethod {
    CREDIT_CARD, DEBIT_CARD, UPI, NET_BANKING, WALLET
}
```

---

### ✅ `PaymentStatus.java`
```java
public enum PaymentStatus {
    INITIATED, SUCCESS, FAILED, CANCELLED
}
```

---

### ✅ `Payment.java`
```java
import java.time.LocalDateTime;
import java.util.UUID;

public class Payment {
    private int id;
    private double amount;
    private PaymentStatus status;
    private PaymentMethod method;
    private String transactionId;
    private LocalDateTime timestamp;

    public Payment(double amount, PaymentMethod method) {
        this.amount = amount;
        this.method = method;
        this.status = PaymentStatus.INITIATED;
        this.timestamp = LocalDateTime.now();
        this.transactionId = UUID.randomUUID().toString();
    }

    public void processPayment() {
        boolean success = PaymentGateway.charge(amount, method);
        this.status = success ? PaymentStatus.SUCCESS : PaymentStatus.FAILED;
    }

    // Getters and setters
}
```

---

### ✅ `PaymentGateway.java`
```java
public class PaymentGateway {
    public static boolean charge(double amount, PaymentMethod method) {
        return Math.random() > 0.1; // Simulate 90% success
    }
}
```

---

### ✅ `BookingStatus.java`
```java
public enum BookingStatus {
    PENDING, CONFIRMED, CANCELLED
}
```

---

### ✅ `Booking.java`
```java
import java.util.List;

public class Booking {
    private int id;
    private User user;
    private Show show;
    private List<Seat> seats;
    private Payment payment;
    private BookingStatus status;

    public Booking(User user, Show show, List<Seat> seats, Payment payment) {
        this.user = user;
        this.show = show;
        this.seats = seats;
        this.payment = payment;
        this.status = BookingStatus.PENDING;
    }

    public void confirm() {
        this.status = BookingStatus.CONFIRMED;
    }

    // Getters and setters
}
```

---

### ✅ `BookingService.java`
```java
import java.util.List;
import java.util.stream.Collectors;

public class BookingService {
    private final Object lock = new Object();

    public Booking bookSeats(User user, Show show, List<Integer> seatIds, PaymentMethod method) {
        synchronized (lock) {
            for (Integer seatId : seatIds) {
                if (!show.isSeatAvailable(seatId)) {
                    throw new RuntimeException("Seat " + seatId + " is already booked.");
                }
            }

            show.bookSeats(seatIds);
            List<Seat> bookedSeats = getSeatsByIds(show.getScreen(), seatIds);
            double amount = calculateAmount(bookedSeats);
            Payment payment = new Payment(amount, method);
            payment.processPayment();

            if (payment.getStatus() != PaymentStatus.SUCCESS) {
                throw new RuntimeException("Payment failed.");
            }

            Booking booking = new Booking(user, show, bookedSeats, payment);
            booking.confirm();
            return booking;
        }
    }

    private List<Seat> getSeatsByIds(Screen screen, List<Integer> seatIds) {
        return screen.getSeats().stream()
            .filter(seat -> seatIds.contains(seat.getId()))
            .collect(Collectors.toList());
    }

    private double calculateAmount(List<Seat> seats) {
        return seats.stream().mapToDouble(seat -> {
            switch (seat.getCategory()) {
                case PLATINUM: return 500;
                case DIAMOND: return 400;
                case GOLD: return 300;
                case SILVER: return 200;
                default: return 0;
            }
        }).sum();
    }
}
```

---

### ✅ `MovieController.java`
```java
import java.util.*;

public class MovieController {
    private Map<City, List<Movie>> cityVsMovies = new HashMap<>();
    private List<Movie> allMovies = new ArrayList<>();

    public void addMovie(Movie m, City c) {
        allMovies.add(m);
        cityVsMovies.computeIfAbsent(c, k -> new ArrayList<>()).add(m);
    }

    public Movie getMovieByName(String name) {
        return allMovies.stream()
            .filter(m -> m.getMovieName().equalsIgnoreCase(name))
            .findFirst()
            .orElse(null);
    }

    public List<Movie> getMoviesByCity(City c) {
        return cityVsMovies.getOrDefault(c, new ArrayList<>());
    }
}
```

---

### ✅ `TheatreController.java`
```java
import java.util.*;

public class TheatreController {
    private Map<City, List<Theatre>> cityVsTheatres = new HashMap<>();
    private List<Theatre> allTheatres = new ArrayList<>();
    private Map<Show, Screen> showsVsScreens = new HashMap<>();

    public void addTheatre(Theatre t, City c) {
        allTheatres.add(t);
        cityVsTheatres.computeIfAbsent(c, k -> new ArrayList<>()).add(t);
    }

    public List<Theatre> getTheatresByCity(City c) {
        return cityVsTheatres.getOrDefault(c, new ArrayList<>());
    }

    public List<Show> getShowsByMovie(Movie m, City c) {
        List<Theatre> theatres = getTheatresByCity(c);
        List<Show> result = new ArrayList<>();
        for (Theatre t : theatres) {
            for (Show s : t.getShows()) {
                if (s.getMovie().equals(m)) {
                    result.add(s);
                }
            }
        }
        return result;
    }
}
```

---


