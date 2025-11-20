
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





Got it, Sriram 👍. Right now your classes are cluttered with **all possible getters and setters**, even for fields that don’t need them. For a clean design, we should only keep accessors where they’re actually useful:

- **Entities like `User`, `City`, `Movie`** → keep getters for reading values, setters only if you expect updates.  
- **Immutable or controlled fields (`transactionId`, `timestamp`, `bookedSeatIds`)** → no setters, because they should never be changed externally.  
- **Enums (`SeatCategory`, `PaymentMethod`, etc.)** → no getters/setters at all.  
- **Collections (`List<Seat>`, `Set<Integer>`)** → usually only getters, not setters, since you manage them through methods (`bookSeats`, etc.).  

Here’s the **refined full codebase** with only the necessary getters/setters:

---

### `User.java`
```java
public class User {
    private int id;
    private String name;

    public User(int id, String name) {
        this.id = id;
        this.name = name;
    }

    public int getId() { return id; }
    public String getName() { return name; }
}
```

---

### `City.java`
```java
public class City {
    private int id;
    private String name;

    public City(int id, String name) {
        this.id = id;
        this.name = name;
    }

    public int getId() { return id; }
    public String getName() { return name; }
}
```

---

### `Movie.java`
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

    public int getId() { return id; }
    public String getMovieName() { return movieName; }
    public int getDuration() { return duration; }
    public String getGenre() { return genre; }
    public String getLanguage() { return language; }
}
```

---

### `SeatCategory.java`
```java
public enum SeatCategory {
    PLATINUM, DIAMOND, GOLD, SILVER
}
```

---

### `Seat.java`
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

    public int getId() { return id; }
    public int getRow() { return row; }
    public SeatCategory getCategory() { return category; }
    public boolean isBooked() { return isBooked; }

    public void setBooked(boolean booked) { this.isBooked = booked; }
    public Object getLock() { return lock; }
}
```

---

### `Screen.java`
```java
import java.util.List;

public class Screen {
    private int id;
    private List<Seat> seats;

    public Screen(int id, List<Seat> seats) {
        this.id = id;
        this.seats = seats;
    }

    public int getId() { return id; }
    public List<Seat> getSeats() { return seats; }
}
```

---

### `Show.java`
```java
import java.time.LocalDateTime;
import java.util.List;
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

    public int getId() { return id; }
    public Movie getMovie() { return movie; }
    public Screen getScreen() { return screen; }
    public LocalDateTime getStartTime() { return startTime; }
    public Set<Integer> getBookedSeatIds() { return bookedSeatIds; }
}
```

---

### `Theatre.java`
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

    public int getId() { return id; }
    public String getAddress() { return address; }
    public City getCity() { return city; }
    public List<Show> getShows() { return shows; }
    public List<Screen> getScreens() { return screens; }
}
```

---

### `PaymentMethod.java`
```java
public enum PaymentMethod {
    CREDIT_CARD, DEBIT_CARD, UPI, NET_BANKING, WALLET
}
```

---

### `PaymentStatus.java`
```java
public enum PaymentStatus {
    INITIATED, SUCCESS, FAILED, CANCELLED
}
```

---

### `Payment.java`
```java
import java.time.LocalDateTime;
import java.util.UUID;

public class Payment {
    private double amount;
    private PaymentStatus status;
    private PaymentMethod method;
    private final String transactionId;
    private final LocalDateTime timestamp;

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

    public double getAmount() { return amount; }
    public PaymentStatus getStatus() { return status; }
    public PaymentMethod getMethod() { return method; }
    public String getTransactionId() { return transactionId; }
    public LocalDateTime getTimestamp() { return timestamp; }
}
```

---

### `PaymentGateway.java`
```java
public class PaymentGateway {
    public static boolean charge(double amount, PaymentMethod method) {
        return Math.random() > 0.1; // Simulate 90% success
    }
}
```

---

### `BookingStatus.java`
```java
public enum BookingStatus {
    PENDING, CONFIRMED, CANCELLED
}
```

---

### `Booking.java`
```java
import java.util.List;

public class Booking {
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

    public User getUser() { return user; }
    public Show getShow() { return show; }
    public List<Seat> getSeats() { return seats; }
    public Payment getPayment() { return payment; }
    public BookingStatus getStatus() { return status; }
}
```

---



###  `BookingService.java`
```java
import java.util.List;
import java.util.stream.Collectors;

public class BookingService {

    public Booking bookSeats(User user, Show show, List<Integer> seatIds, PaymentMethod method) {
        // Fetch seat objects
        List<Seat> seatsToBook = getSeatsByIds(show.getScreen(), seatIds);

        // Lock each seat individually
        for (Seat seat : seatsToBook) {
            synchronized (seat.getLock()) {
                if (seat.isBooked()) {
                    throw new RuntimeException("Seat " + seat.getId() + " is already booked.");
                }
            }
        }

        // Process payment
        double amount = calculateAmount(seatsToBook);
        Payment payment = new Payment(amount, method);
        payment.processPayment();

        if (payment.getStatus() != PaymentStatus.SUCCESS) {
            throw new RuntimeException("Payment failed.");
        }

        // Mark seats as booked only after payment succeeds
        for (Seat seat : seatsToBook) {
            synchronized (seat.getLock()) {
                seat.setBooked(true);
            }
        }

        Booking booking = new Booking(user, show, seatsToBook, payment);
        booking.confirm();
        return booking;
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

### `MovieController.java`
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

### `TheatreController.java`
```java
import java.util.*;

public class TheatreController {
    private Map<City, List<Theatre>> cityVsTheatres = new HashMap<>();
    private List<Theatre> allTheatres = new ArrayList<>();

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

### Sample Usage Code
Here’s a **demo `Main.java`** showing how everything works together:

```java
import java.time.LocalDateTime;
import java.util.*;

public class Main {
    public static void main(String[] args) {
        // Create city
        City hyderabad = new City(1, "Hyderabad");

        // Create movies
        Movie movie1 = new Movie(1, "Inception", 148, "Sci-Fi", "English");
        Movie movie2 = new Movie(2, "RRR", 180, "Action", "Telugu");

        // Movie controller
        MovieController movieController = new MovieController();
        movieController.addMovie(movie1, hyderabad);
        movieController.addMovie(movie2, hyderabad);

        // Create seats
        List<Seat> seats = new ArrayList<>();
        for (int i = 1; i <= 10; i++) {
            seats.add(new Seat(i, i / 5, SeatCategory.GOLD));
        }

        // Create screen
        Screen screen1 = new Screen(1, seats);

        // Create show
        Show show1 = new Show(1, movie1, screen1, LocalDateTime.now().plusHours(2));

        // Create theatre
        Theatre theatre1 = new Theatre(1, "Madhapur Road", hyderabad, Arrays.asList(show1), Arrays.asList(screen1));

        // Theatre controller
        TheatreController theatreController = new TheatreController();
        theatreController.addTheatre(theatre1, hyderabad);

        // Create user
        User user1 = new User(1, "Sriram");

        // Booking service
        BookingService bookingService = new BookingService();

        // Book seats
        try {
            Booking booking = bookingService.bookSeats(user1, show1, Arrays.asList(1, 2), PaymentMethod.UPI);
            System.out.println("Booking confirmed for " + booking.getUser().getName());
            System.out.println("Seats booked: " + booking.getSeats().stream().map(Seat::getId).toList());
            System.out.println("Payment status: " + booking.getPayment().getStatus());
        } catch (RuntimeException e) {
            System.out.println("Booking failed: " + e.getMessage());
        }

        // Query movies by city
        System.out.println("Movies in Hyderabad: ");
        for (Movie m : movieController.getMoviesByCity(hyderabad)) {
            System.out.println("- " + m.getMovieName());
        }

        // Query shows by movie
        System.out.println("Shows for Inception in Hyderabad: ");
        for (Show s : theatreController.getShowsByMovie(movie1, hyderabad)) {
            System.out.println("- Show ID: " + s.getId() + " at " + s.getStartTime());
        }
    }
}
```

---



