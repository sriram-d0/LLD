## 🧠 Tic Tac Toe LLD 

### 🎯 Objective
Build a modular, extensible Tic Tac Toe game in Java using clean object-oriented design and enums for piece symbols.

---

### 🧩 Core Components

| Class / Enum     | Role                                                                 |
|------------------|----------------------------------------------------------------------|
| `PieceType`      | Enum for symbols: `X`, `O`                                           |
| `PlayingPiece`   | Wraps `PieceType`, provides `getSymbol()`                            |
| `Player`         | Holds player name and their `PlayingPiece`                           |
| `Board`          | NxN grid, handles move placement, win check, draw check, display     |
| `Game`           | Controls game flow, player turns, win/draw logic                     |

---

### 🔄 Game Flow

1. Initialize `Board` and two `Player` objects.
2. Loop through turns:
   - Current player makes a move.
   - Board places piece if valid.
   - Check for win or draw.
   - Switch player if game continues.


---

### 📦 Folder Structure

```
src/
├── enums/          → PieceType.java
├── models/         → PlayingPiece.java, Player.java, Board.java
└── Game.java       → Main game controller
```

---



### 🧪 Key Methods to Remember

- `Board.placePiece(row, col, piece)` → Places a symbol.
- `Board.checkWin(piece)` → Checks win condition.
- `Board.isFull()` → Checks for draw.
- `Game.playTurn(row, col)` → Executes a turn.

---



## 🧩 Class Overview

### `PieceType` (Enum)
Defines the two valid symbols used in the game:
```java
public enum PieceType {
    X, O;
}
```

### `PlayingPiece`
Wraps the enum and provides symbol access:
```java
public class PlayingPiece {
    private final PieceType type;

    public PlayingPiece(PieceType type) {
        this.type = type;
    }

    public PieceType getType() {
        return type;
    }

    public char getSymbol() {
        return type.name().charAt(0); // Returns 'X' or 'O'
    }
}
```

### `Player`
Represents a player with a name and a piece:
```java
public class Player {
    private final String name;
    private final PlayingPiece piece;

    public Player(String name, PlayingPiece piece) {
        this.name = name;
        this.piece = piece;
    }

    public String getName() {
        return name;
    }

    public PlayingPiece getPiece() {
        return piece;
    }
}
```

### `Board`
Handles the game grid, move placement, win checks, and board display:
```java
public class Board {
    private final int size;
    private final PlayingPiece[][] grid;

    public Board(int size) {
        this.size = size;
        this.grid = new PlayingPiece[size][size];
    }

    public boolean placePiece(int row, int col, PlayingPiece piece) {
        if (grid[row][col] == null) {
            grid[row][col] = piece;
            return true;
        }
        return false;
    }

    public boolean checkWin(PlayingPiece piece) {
        // Check rows, columns, diagonals
        // Implementation omitted for brevity
    }

    public boolean isFull() {
        for (int i = 0; i < size; i++)
            for (int j = 0; j < size; j++)
                if (grid[i][j] == null) return false;
        return true;
    }

    public void printBoard() {
        // Display current board state
    }
}
```

### `Game`
Controls the game loop and player turns:
```java
public class Game {
    private final Board board;
    private final Player[] players;
    private int currentPlayerIndex;

    public Game(Player player1, Player player2, int boardSize) {
        this.board = new Board(boardSize);
        this.players = new Player[]{player1, player2};
        this.currentPlayerIndex = 0;
    }

    public void playTurn(int row, int col) {
        Player currentPlayer = players[currentPlayerIndex];
        if (board.placePiece(row, col, currentPlayer.getPiece())) {
            board.printBoard();
            if (board.checkWin(currentPlayer.getPiece())) {
                System.out.println(currentPlayer.getName() + " wins!");
            } else if (board.isFull()) {
                System.out.println("It's a draw!");
            } else {
                currentPlayerIndex = (currentPlayerIndex + 1) % 2;
            }
        } else {
            System.out.println("Invalid move. Try again.");
        }
    }
}
```

---

## 🧠 Design Principles

- ✅ Encapsulation: Each class handles its own logic.
- ✅ Enum Safety: Prevents invalid symbols.
- ✅ Extensibility: Easy to scale to NxN boards or add AI.
- ✅ Separation of Concerns: Game flow, board logic, and player identity are decoupled.

---

## 🛠️ Future Enhancements

- Add AI player using Minimax algorithm.
- Support for NxN boards and dynamic win conditions.
- GUI using JavaFX or Swing.
- Persistent score tracking.

---

