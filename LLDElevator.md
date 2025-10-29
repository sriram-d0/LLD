# 🛗 Elevator System Simulation

A modular and extensible Java-based simulation of an elevator control system. Designed to handle multiple elevators, floors, and user requests with clear separation of concerns and scalable architecture.

---


## 🚀 Features

- Multiple elevators and floors
- External and internal button request handling
- Directional movement and status tracking
- Door control and display updates
- Simple dispatch logic for request routing

---

## 🧠 Design Overview

### 🏢 Building
- Initializes floors and elevator controllers
- Registers dispatchers for external buttons

### 🛗 Elevator
- Moves between floors
- Tracks direction, status, and current floor
- Contains internal buttons, door, and display

### 🎮 ElevatorController
- Accepts and queues requests
- Controls elevator movement based on queued requests

### 🛎️ ExternalButton & Dispatch
- Allows users to request elevators from floors
- Dispatches requests to appropriate controllers

### 🔘 InternalButton & Dispatch
- Allows passengers to select destination floors
- Directly controls elevator movement

### 📺 Display
- Shows current floor and direction

### 🚪 Door
- Opens and closes based on elevator arrival

---



## 📁 Project Structure

```
src/
├── Building.java
├── Floor.java
├── Elevator.java
├── ElevatorController.java
├── Door.java
├── Display.java
├── ExternalButton.java
├── ExternalButtonDispatch.java
├── InternalButton.java
├── InternalButtonDispatch.java
├── Direction.java
├── Status.java
├── DoorStatus.java
└── ElevatorSystem.java
```

---

## 🧱 Enums

### `Direction.java`
```java
enum Direction {
    UP,
    DOWN
}
```

### `Status.java`
```java
enum Status {
    MOVING,
    IDLE
}
```

### `DoorStatus.java`
```java
enum DoorStatus {
    OPEN,
    CLOSED
}
```

---

## 🏢 Core Classes

### `Building.java`
Initializes floors and elevators.

```java
class Building {
    List<Floor> floors;
    List<ElevatorController> elevatorControllers;

    Building(int numFloors, int numElevators) {
        floors = new ArrayList<>();
        elevatorControllers = new ArrayList<>();
        for (int i = 0; i < numFloors; i++) floors.add(new Floor(i));
        for (int i = 0; i < numElevators; i++)
            elevatorControllers.add(new ElevatorController(new Elevator(i)));
    }
}
```

---

### `Floor.java`
Represents a floor with an external button.

```java
class Floor {
    int floorNum;
    ExternalButton externalButton;

    Floor(int floorNum) {
        this.floorNum = floorNum;
        this.externalButton = new ExternalButton(floorNum);
    }
}
```

---

### `Elevator.java`
Handles movement, direction, and internal controls.

```java
class Elevator {
    int liftId;
    Direction direction;
    Status status;
    Door door;
    Display display;
    InternalButton internalButton;
    int currentFloor;

    Elevator(int liftId) {
        this.liftId = liftId;
        this.status = Status.IDLE;
        this.direction = Direction.UP;
        this.door = new Door();
        this.display = new Display();
        this.internalButton = new InternalButton(this);
        this.currentFloor = 0;
    }

    void move(int targetFloor, Direction direction) {
        this.direction = direction;
        this.status = Status.MOVING;
        while (currentFloor != targetFloor)
            currentFloor += (direction == Direction.UP) ? 1 : -1;
        status = Status.IDLE;
        door.open();
    }
}
```

---

### `ElevatorController.java`
Controls elevator logic and request queue.

```java
class ElevatorController {
    Elevator elevator;
    Queue<Integer> requestQueue;

    ElevatorController(Elevator elevator) {
        this.elevator = elevator;
        this.requestQueue = new LinkedList<>();
    }

    void acceptNewRequest(int floor, Direction direction) {
        requestQueue.add(floor);
    }

    void controlCar() {
        while (!requestQueue.isEmpty()) {
            int nextFloor = requestQueue.poll();
            Direction dir = (nextFloor > elevator.currentFloor) ? Direction.UP : Direction.DOWN;
            elevator.door.close();
            elevator.move(nextFloor, dir);
        }
    }
}
```

---

## 🧩 Components

### `Door.java`
```java
class Door {
    DoorStatus status = DoorStatus.CLOSED;

    void open() { status = DoorStatus.OPEN; }
    void close() { status = DoorStatus.CLOSED; }
}
```

### `Display.java`
```java
class Display {
    int currentFloor;
    Direction direction;

    void update(int floor, Direction direction) {
        this.currentFloor = floor;
        this.direction = direction;
    }
}
```

---

## 🔘 Buttons & Dispatchers

### `ExternalButton.java`
```java
class ExternalButton {
    int floorNum;
    ExternalButtonDispatch dispatch;

    ExternalButton(int floorNum) {
        this.floorNum = floorNum;
        this.dispatch = new ExternalButtonDispatch();
    }

    void pushButton(Direction direction) {
        dispatch.submitNewRequest(floorNum, direction);
    }
}
```

### `ExternalButtonDispatch.java`
```java
class ExternalButtonDispatch {
    List<ElevatorController> controllers = new ArrayList<>();

    void registerControllers(List<ElevatorController> controllers) {
        this.controllers = controllers;
    }

    void submitNewRequest(int floor, Direction direction) {
        controllers.get(0).acceptNewRequest(floor, direction); // Simplified
    }
}
```

### `InternalButton.java`
```java
class InternalButton {
    Elevator elevator;
    InternalButtonDispatch dispatch;

    InternalButton(Elevator elevator) {
        this.elevator = elevator;
        this.dispatch = new InternalButtonDispatch(elevator);
    }

    void pressButton(int targetFloor) {
        dispatch.submitInternalRequest(targetFloor);
    }
}
```

### `InternalButtonDispatch.java`
```java
class InternalButtonDispatch {
    Elevator elevator;

    InternalButtonDispatch(Elevator elevator) {
        this.elevator = elevator;
    }

    void submitInternalRequest(int targetFloor) {
        Direction dir = (targetFloor > elevator.currentFloor) ? Direction.UP : Direction.DOWN;
        elevator.door.close();
        elevator.move(targetFloor, dir);
    }
}
```

---

## 🧪 Main Simulation

### `ElevatorSystem.java`
```java
public class ElevatorSystem {
    public static void main(String[] args) {
        Building building = new Building(5, 2);

        for (Floor floor : building.floors)
            floor.externalButton.dispatch.registerControllers(building.elevatorControllers);

        building.floors.get(2).externalButton.pushButton(Direction.UP);
        building.elevatorControllers.get(0).elevator.internalButton.pressButton(4);

        for (ElevatorController ec : building.elevatorControllers)
            ec.controlCar();
    }
}
```

---

## 🛠️ How to Run

```bash
javac *.java
java ElevatorSystem
```

---

## 📌 Future Enhancements

- Smart elevator allocation (nearest car, load balancing)
- GUI simulation
- Emergency handling
- Logging and analytics

---

Let me know if you want this bundled into a downloadable file or turned into a GitHub project template.
## 🧪 Sample Usage

```java
public class ElevatorSystem {
    public static void main(String[] args) {
        Building building = new Building(5, 2);

        // Register controllers to dispatchers
        for (Floor floor : building.floors) {
            floor.externalButton.dispatch.registerControllers(building.elevatorControllers);
        }

        // Simulate external request
        building.floors.get(2).externalButton.pushButton(Direction.UP);

        // Simulate internal request
        ElevatorController controller = building.elevatorControllers.get(0);
        controller.elevator.internalButton.pressButton(4);

        // Process requests
        for (ElevatorController ec : building.elevatorControllers) {
            ec.controlCar();
        }
    }
}
