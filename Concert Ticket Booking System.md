



# Online Concert System

## Flow Summary

- **[User](guide://action?prefill=Tell%20me%20more%20about%3A%20User)**: Has `id`, `name`, `UserType` (ADMIN/ORGANIZER/CUSTOMER), and `balance`.
- **[Concert](guide://action?prefill=Tell%20me%20more%20about%3A%20Concert)**: Organized by an organizer, contains metadata.
- **[Show](guide://action?prefill=Tell%20me%20more%20about%3A%20Show)**: Specific concert occurrence with venue, time, seat map, ticket prices.
- **[Seat](guide://action?prefill=Tell%20me%20more%20about%3A%20Seat)**: Belongs to a show, has category and availability.
- **[Booking](guide://action?prefill=Tell%20me%20more%20about%3A%20Booking)**: Created by customer, holds seats, status, tickets.
- **[Payment](guide://action?prefill=Tell%20me%20more%20about%3A%20Payment)**: Handles payment and settlement.
- **[ConcertSystem](guide://action?prefill=Tell%20me%20more%20about%3A%20ConcertSystem)**: Main driver holding users, concerts, shows, bookings.

---




## 1. User and Roles

```java
public enum UserType {
    ADMIN, ORGANIZER, CUSTOMER
}

public class User {
    private int id;
    private String name;
    private UserType userType;
    private double balance;

    public User(int id, String name, UserType userType, double balance) {
        this.id = id;
        this.name = name;
        this.userType = userType;
        this.balance = balance;
    }

    public boolean canCreateConcert() {
        return userType == UserType.ORGANIZER || userType == UserType.ADMIN;
    }

    public boolean canBookTickets() {
        return userType == UserType.CUSTOMER;
    }

    public void debit(double amount) {
        if (balance < amount) throw new IllegalStateException("Insufficient balance");
        balance -= amount;
    }

    public void credit(double amount) {
        balance += amount;
    }

    // Getters
    public int getId() { return id; }
    public String getName() { return name; }
    public UserType getUserType() { return userType; }
    public double getBalance() { return balance; }
}
```

---

## 2. Concert

```java
public class Concert {
    private int id;
    private String name;
    private String artist;
    private User organizer;

    public Concert(int id, String name, String artist, User organizer) {
        if (!organizer.canCreateConcert()) {
            throw new IllegalStateException("Only organizers/admin can create concerts");
        }
        this.id = id;
        this.name = name;
        this.artist = artist;
        this.organizer = organizer;
    }

    public int getId() { return id; }
    public String getName() { return name; }
    public String getArtist() { return artist; }
    public User getOrganizer() { return organizer; }
}
```

---

## 3. Seat

```java
public enum SeatCategory {
    VIP, REGULAR, BALCONY
}

public class Seat {
    private String id;
    private SeatCategory category;
    private boolean booked;

    public Seat(String id, SeatCategory category) {
        this.id = id;
        this.category = category;
        this.booked = false;
    }

    public boolean isBooked() { return booked; }
    public void book() { this.booked = true; }

    public String getId() { return id; }
    public SeatCategory getCategory() { return category; }
}
```

---

## 4. Show

```java
import java.time.LocalDateTime;
import java.util.*;

public class Show {
    private int id;
    private Concert concert;
    private LocalDateTime startTime;
    private Map<SeatCategory, Double> priceByCategory;
    private List<Seat> seats;

    public Show(int id, Concert concert, LocalDateTime startTime,
                Map<SeatCategory, Double> priceByCategory, List<Seat> seats) {
        this.id = id;
        this.concert = concert;
        this.startTime = startTime;
        this.priceByCategory = priceByCategory;
        this.seats = seats;
    }

    public List<Seat> getSeats() { return seats; }

    public List<Seat> getAvailableSeats(Set<String> lockedSeatIds) {
        List<Seat> available = new ArrayList<>();
        for (Seat s : seats) {
            if (!s.isBooked() && !lockedSeatIds.contains(s.getId())) {
                available.add(s);
            }
        }
        return available;
    }

    public double getPrice(SeatCategory category) {
        return priceByCategory.getOrDefault(category, 0.0);
    }

    public int getId() { return id; }
    public Concert getConcert() { return concert; }
    public LocalDateTime getStartTime() { return startTime; }
}
```

---

## 5. Seat Lock Manager (Double Locking Prevention)

```java
import java.util.*;
import java.util.concurrent.ConcurrentHashMap;

class SeatLock {
    private final String seatId;
    private final int showId;
    private final int userId;
    private final long expiryTime;

    public SeatLock(String seatId, int showId, int userId, long expiryTime) {
        this.seatId = seatId;
        this.showId = showId;
        this.userId = userId;
        this.expiryTime = expiryTime;
    }

    public boolean isExpired() {
        return System.currentTimeMillis() > expiryTime;
    }

    public String getSeatId() { return seatId; }
    public int getUserId() { return userId; }
}

public class SeatLockManager {
    private final long lockDurationMillis;
    private final Map<String, SeatLock> locks = new ConcurrentHashMap<>();

    public SeatLockManager(long lockDurationMillis) {
        this.lockDurationMillis = lockDurationMillis;
    }

    private String key(int showId, String seatId) {
        return showId + "#" + seatId;
    }

    public synchronized boolean lockSeats(int showId, List<String> seatIds, int userId) {
        // Check if any seat is already locked
        for (String seatId : seatIds) {
            SeatLock existing = locks.get(key(showId, seatId));
            if (existing != null && !existing.isExpired()) {
                return false; // seat already locked
            }
        }
        // Lock all seats
        long expiry = System.currentTimeMillis() + lockDurationMillis;
        for (String seatId : seatIds) {
            locks.put(key(showId, seatId), new SeatLock(seatId, showId, userId, expiry));
        }
        return true;
    }

    public synchronized void releaseSeats(int showId, List<String> seatIds, int userId) {
        for (String seatId : seatIds) {
            String k = key(showId, seatId);
            SeatLock lock = locks.get(k);
            if (lock != null && lock.getUserId() == userId) {
                locks.remove(k);
            }
        }
    }

    public synchronized Set<String> getLockedSeats(int showId) {
        Set<String> res = new HashSet<>();
        for (Map.Entry<String, SeatLock> e : locks.entrySet()) {
            if (e.getKey().startsWith(showId + "#") && !e.getValue().isExpired()) {
                res.add(e.getValue().getSeatId());
            }
        }
        return res;
    }
}
```

---

## 6. Booking

```java
import java.util.*;

public enum BookingStatus {
    PENDING, CONFIRMED, CANCELLED
}

public class Booking {
    private int id;
    private User customer;
    private Show show;
    private List<Seat> seats;
    private double totalAmount;
    private BookingStatus status;

    public Booking(int id, User customer, Show show, List<Seat> seats) {
        if (!customer.canBookTickets()) {
            throw new IllegalStateException("Only customers can book tickets");
        }
        this.id = id;
        this.customer = customer;
        this.show = show;
        this.seats = seats;
        this.totalAmount = calculateTotal();
        this.status = BookingStatus.PENDING;
    }

    private double calculateTotal() {
        double sum = 0;
        for (Seat s : seats) {
            sum += show.getPrice(s.getCategory());
        }
        return sum;
    }

    public void confirm() {
        for (Seat s : seats) {
            s.book();
        }
        this.status = BookingStatus.CONFIRMED;
    }

    public void cancel() {
        this.status = BookingStatus.CANCELLED;
    }

    public int getId() { return id; }
    public double getTotalAmount() { return totalAmount; }
    public BookingStatus getStatus() { return status; }
    public List<Seat> getSeats() { return seats; }
    public Show getShow() { return show; }
    public User getCustomer() { return customer; }
}
```

---

## 7. Payment

```java
public class Payment {
    public void processPayment(Booking booking) {
        if (booking.getStatus() != BookingStatus.PENDING) {
            throw new IllegalStateException("Booking must be pending before payment");
        }
        User customer = booking.getCustomer();
        customer.debit(booking.getTotalAmount());
        booking.confirm();
    }
}
```

---



## 8. ConcertSystem (continued)

```java
import java.util.*;
import java.util.stream.Collectors;

public class ConcertSystem {
    private List<User> users = new ArrayList<>();
    private List<Concert> concerts = new ArrayList<>();
    private List<Show> shows = new ArrayList<>();
    private List<Booking> bookings = new ArrayList<>();
    private SeatLockManager seatLockManager = new SeatLockManager(60_000); // 1 min lock

    public void addUser(User user) { users.add(user); }
    public void addConcert(Concert concert) { concerts.add(concert); }
    public void addShow(Show show) { shows.add(show); }

    public Booking createBooking(User customer, Show show, List<String> seatIds) {
        boolean locked = seatLockManager.lockSeats(show.getId(), seatIds, customer.getId());
        if (!locked) throw new IllegalStateException("Some seats already locked");

        // Map seatIds to actual Seat objects
        Map<String, Seat> seatMap = show.getSeats().stream()
                .collect(Collectors.toMap(Seat::getId, s -> s));
        List<Seat> selectedSeats = new ArrayList<>();
        for (String sid : seatIds) {
            Seat seat = seatMap.get(sid);
            if (seat == null || seat.isBooked()) {
                seatLockManager.releaseSeats(show.getId(), seatIds, customer.getId());
                throw new IllegalStateException("Seat not available: " + sid);
            }
            selectedSeats.add(seat);
        }

        Booking booking = new Booking(bookings.size() + 1, customer, show, selectedSeats);
        bookings.add(booking);
        return booking;
    }

    public void payForBooking(Booking booking) {
        Payment payment = new Payment();
        payment.processPayment(booking);
        // Release locks after payment
        List<String> seatIds = booking.getSeats().stream().map(Seat::getId).collect(Collectors.toList());
        seatLockManager.releaseSeats(booking.getShow().getId(), seatIds, booking.getCustomer().getId());
    }

    public List<Show> searchShowsByConcertName(String name) {
        List<Show> result = new ArrayList<>();
        for (Show s : shows) {
            if (s.getConcert().getName().equalsIgnoreCase(name)) {
                result.add(s);
            }
        }
        return result;
    }

    // Getters
    public List<User> getUsers() { return users; }
    public List<Concert> getConcerts() { return concerts; }
    public List<Show> getShows() { return shows; }
    public List<Booking> getBookings() { return bookings; }
}
```

---

## 9. Example Usage (Demo)

```java
import java.time.LocalDateTime;
import java.util.*;

public class Demo {
    public static void main(String[] args) {
        ConcertSystem system = new ConcertSystem();

        // Create users
        User organizer = new User(1, "Organizer1", UserType.ORGANIZER, 0);
        User customer = new User(2, "Customer1", UserType.CUSTOMER, 5000);
        system.addUser(organizer);
        system.addUser(customer);

        // Create concert
        Concert concert = new Concert(1, "Winter Fest", "Aria", organizer);
        system.addConcert(concert);

        // Create show with seats
        List<Seat> seats = new ArrayList<>();
        for (int i = 1; i <= 5; i++) {
            seats.add(new Seat("VIP-" + i, SeatCategory.VIP));
        }
        for (int i = 1; i <= 5; i++) {
            seats.add(new Seat("REG-" + i, SeatCategory.REGULAR));
        }
        Map<SeatCategory, Double> prices = new HashMap<>();
        prices.put(SeatCategory.VIP, 3000.0);
        prices.put(SeatCategory.REGULAR, 1500.0);

        Show show = new Show(1, concert, LocalDateTime.now().plusDays(1), prices, seats);
        system.addShow(show);

        // Customer searches shows
        List<Show> found = system.searchShowsByConcertName("Winter Fest");
        System.out.println("Shows found: " + found.size());

        // Customer selects seats
        List<String> selectedSeatIds = Arrays.asList("VIP-1", "VIP-2");

        // Create booking
        Booking booking = system.createBooking(customer, show, selectedSeatIds);
        System.out.println("Booking created with total: " + booking.getTotalAmount());

        // Pay for booking
        system.payForBooking(booking);
        System.out.println("Booking status after payment: " + booking.getStatus());
        System.out.println("Customer balance after payment: " + customer.getBalance());

        // Try double booking same seat
        try {
            User customer2 = new User(3, "Customer2", UserType.CUSTOMER, 5000);
            system.addUser(customer2);
            system.createBooking(customer2, show, Arrays.asList("VIP-1"));
        } catch (Exception e) {
            System.out.println("Double booking prevented: " + e.getMessage());
        }
    }
}
```

---




## Example Usage Flow

1. Organizer creates a concert and adds a show with seats and prices.  
2. Customer searches for shows and selects seats.  
3. Booking created → payment processed.  
4. Seats marked booked → booking confirmed.  


