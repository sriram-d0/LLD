# 🐍 Snake & Ladder – Low-Level Design (LLD)

This project implements a modular, object-oriented **Snake & Ladder** game using Java. It supports multiple players, configurable board size, snakes, ladders, and dice count.

---

## 📦 Class Structure

### 1. `Player`
Represents a player in the game.

| Field     | Type   | Description              |
|-----------|--------|--------------------------|
| `id`      | String | Unique identifier        |
| `curPos`  | int    | Current board position   |

---

### 2. `Dice`
Handles dice rolling logic.

| Field        | Type | Description              |
|--------------|------|--------------------------|
| `diceCount`  | int  | Number of dice used      |

| Method       | Return | Description             |
|--------------|--------|-------------------------|
| `rollDice()` | int    | Returns total dice roll |

---

### 3. `Jump`
Represents a snake or ladder.

| Field   | Type | Description                  |
|---------|------|------------------------------|
| `start` | int  | Start position of the jump   |
| `end`   | int  | End position of the jump     |

---

### 4. `Cell`
Represents a cell on the board.

| Field   | Type  | Description                 |
|---------|-------|-----------------------------|
| `jump`  | Jump  | Optional snake or ladder    |

---

### 5. `Board`
Represents the game board.

| Field     | Type       | Description                     |
|-----------|------------|---------------------------------|
| `cells`   | Cell[][]   | 2D grid of cells                |
| `rows`    | int        | Number of rows (default: 10)    |
| `cols`    | int        | Number of columns (default: 10) |

| Method              | Description                                  |
|---------------------|----------------------------------------------|
| `initializeCells()` | Initializes all cells                        |
| `addSnakesAndLadders()` | Randomly places snakes and ladders     |
| `getCell(pos)`      | Returns the cell at a given position         |
| `getCoordinates(pos)` | Converts linear position to 2D coordinates |
| `getSize()`         | Returns total number of cells                |

---

### 6. `Game`
Controls the game loop and player turns.

| Field        | Type         | Description                         |
|--------------|--------------|-------------------------------------|
| `board`      | Board        | Game board                          |
| `dice`       | Dice         | Dice object                         |
| `playerList` | Deque<Player>| Queue of players                    |
| `winner`     | Player       | Winner of the game (if any)         |

| Method         | Description                              |
|----------------|------------------------------------------|
| `addPlayers()` | Adds players to the game                 |
| `startGame()`  | Starts the game loop                     |

---


---

## 📚 Class-Level Documentation

### 🧍 `Player.java`
Represents a player in the game with a unique ID and current position.

```java
public class Player {
    private String id;
    private int curPos;

    public Player(String id) {
        this.id = id;
        this.curPos = 0;
    }

    public String getId() { return id; }
    public int getCurPos() { return curPos; }
    public void setCurPos(int curPos) { this.curPos = curPos; }
}
```

---

### 🎲 `Dice.java`
Handles dice rolling logic. Supports multiple dice.

```java
import java.util.Random;

public class Dice {
    private int diceCount;
    private Random rand;

    public Dice(int diceCount) {
        this.diceCount = diceCount;
        this.rand = new Random();
    }

    public int rollDice() {
        int total = 0;
        for (int i = 0; i < diceCount; i++) {
            total += rand.nextInt(6) + 1;
        }
        return total;
    }
}
```

---

### 🐍🪜 `Jump.java`
Represents a snake or ladder jump from one cell to another.

```java
public class Jump {
    private int start;
    private int end;

    public Jump(int start, int end) {
        this.start = start;
        this.end = end;
    }

    public int getStart() { return start; }
    public int getEnd() { return end; }
}
```

---

### 🧱 `Cell.java`
Represents a cell on the board. May contain a jump (snake or ladder).

```java
public class Cell {
    private Jump jump;

    public Cell() {
        this.jump = null;
    }

    public void setJump(Jump jump) {
        this.jump = jump;
    }

    public Jump getJump() {
        return jump;
    }
}
```

---

### 🗺️ `Board.java`
Represents the 10×10 game board. Initializes cells and randomly places snakes and ladders.

```java
import java.util.Random;

public class Board {
    private int rows;
    private int cols;
    private Cell[][] cells;

    public Board(int rows, int cols, int noOfSnakes, int noOfLadders) {
        this.rows = rows;
        this.cols = cols;
        this.cells = new Cell[rows][cols];
        initializeCells();
        addSnakesAndLadders(noOfSnakes, noOfLadders);
    }

    private void initializeCells() {
        for (int i = 0; i < rows; i++) {
            for (int j = 0; j < cols; j++) {
                cells[i][j] = new Cell();
            }
        }
    }

    private void addSnakesAndLadders(int noOfSnakes, int noOfLadders) {
        Random rand = new Random();
        int totalCells = rows * cols;

        for (int i = 0; i < noOfSnakes; i++) {
            int start = rand.nextInt(totalCells - 1) + 1;
            int end = rand.nextInt(start - 1) + 1;
            setJump(start, end);
        }

        for (int i = 0; i < noOfLadders; i++) {
            int start = rand.nextInt(totalCells - 1) + 1;
            int end = rand.nextInt(totalCells - start) + start + 1;
            setJump(start, end);
        }
    }

    private void setJump(int start, int end) {
        int[] coord = getCoordinates(start);
        cells[coord[0]][coord[1]].setJump(new Jump(start, end));
    }

    public Cell getCell(int position) {
        if (position <= 0 || position > rows * cols) return null;
        int[] coord = getCoordinates(position);
        return cells[coord[0]][coord[1]];
    }

    public int[] getCoordinates(int position) {
        int row = (position - 1) / cols;
        int col = (position - 1) % cols;
        return new int[]{row, col};
    }

    public int getSize() {
        return rows * cols;
    }
}
```

---

### 🎮 `Game.java`
Controls the game loop, player turns, dice rolls, and win condition.

```java
import java.util.*;

public class Game {
    private Board board;
    private Dice dice;
    private Deque<Player> playerList;
    private Player winner;

    public Game(int rows, int cols, int noOfSnakes, int noOfLadders, int diceCount) {
        this.board = new Board(rows, cols, noOfSnakes, noOfLadders);
        this.dice = new Dice(diceCount);
        this.playerList = new LinkedList<>();
    }

    public void addPlayers(List<String> playerIds) {
        for (String id : playerIds) {
            playerList.add(new Player(id));
        }
    }

    public void startGame() {
        while (winner == null) {
            Player currentPlayer = playerList.poll();
            int diceRoll = dice.rollDice();
            int newPos = currentPlayer.getCurPos() + diceRoll;

            if (newPos > board.getSize()) {
                playerList.add(currentPlayer);
                continue;
            }

            Cell cell = board.getCell(newPos);
            if (cell != null && cell.getJump() != null) {
                Jump jump = cell.getJump();
                System.out.println(currentPlayer.getId() + " hit a jump from " + jump.getStart() + " to " + jump.getEnd());
                newPos = jump.getEnd();
            }

            currentPlayer.setCurPos(newPos);
            System.out.println(currentPlayer.getId() + " rolled " + diceRoll + " and moved to " + newPos);

            if (newPos == board.getSize()) {
                winner = currentPlayer;
                System.out.println("🎉 Winner is: " + winner.getId());
            } else {
                playerList.add(currentPlayer);
            }
        }
    }
}
```

---

### 🚀 `Main.java`
Entry point to initialize and run the game.

```java
import java.util.Arrays;

public class Main {
    public static void main(String[] args) {
        Game game = new Game(10, 10, 5, 5, 1); // 10x10 board, 5 snakes, 5 ladders, 1 dice
        game.addPlayers(Arrays.asList("Alice", "Bob"));
        game.startGame();
    }
}
```


---

## 🧠 Snake & Ladder LLD – Quick Revision Guide

### 🎮 Game Flow
- Players take turns rolling dice.
- They move forward by the dice value.
- If they land on a cell with a **snake or ladder**, they jump to the target cell.
- First player to reach **cell 100** wins.

---

### 🧱 Board Structure
- **10×10 grid** = 100 cells.
- Each cell may contain a `Jump` (snake or ladder).
- Board randomly places snakes and ladders during initialization.

---

### 🧩 Class Summary

#### 1. `Player`
- Fields: `id`, `curPos`
- Tracks player identity and current position.

#### 2. `Dice`
- Field: `diceCount`
- Method: `rollDice()` → returns total dice value.

#### 3. `Jump`
- Fields: `start`, `end`
- Represents a snake (start > end) or ladder (start < end).

#### 4. `Cell`
- Field: `Jump j`
- Each cell may contain a jump.

#### 5. `Board`
- Fields: `Cell[][] cells`, `rows`, `cols`
- Methods:
  - `initializeCells()` → fills grid with empty cells.
  - `addSnakesAndLadders()` → randomly places jumps.
  - `getCell(pos)` → returns cell at linear position.
  - `getCoordinates(pos)` → converts linear to grid coordinates.
  - `getSize()` → returns total cells (100).

#### 6. `Game`
- Fields: `Board`, `Dice`, `Deque<Player>`, `Player winner`
- Methods:
  - `addPlayers()` → adds players to queue.
  - `startGame()` → runs game loop until someone wins.

---

### 🧪 Sample Setup
```java
Game game = new Game(10, 10, 5, 5, 1); // 10x10 board, 5 snakes, 5 ladders, 1 dice
game.addPlayers(Arrays.asList("Alice", "Bob"));
game.startGame();
```

---

### ✅ Key Concepts to Remember
- `Jump` is used for both snakes and ladders.
- `Board` uses 2D grid but maps positions linearly (1–100).
- `Game` uses a queue to rotate player turns.
- Dice rolls can be skipped if they overshoot cell 100.
- Winner is declared when `curPos == 100`.

---



