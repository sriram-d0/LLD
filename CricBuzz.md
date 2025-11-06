
## 🏏 CricBuzz

### 1. **Match**
- Represents a cricket match.
- Contains:
  - `Team teamA`, `teamB`
  - `Date dt`, `String venue`
  - `Innings[] inn`
  - `MatchType tp` (ODI, T20, Test)
  - `Team tossWon`
- Method: `startMatch()` initializes the match and innings.

### 2. **Team**
- Represents a cricket team.
- Contains:
  - `String name`
  - `Queue<Player> battingOrder`
  - `List<Player> allPlayers`
  - `PlayerBattingController pc`
  - `PlayerBowlingController bc`

### 3. **Player**
- Represents a cricketer.
- Contains:
  - `String name`, `int age`, `String address`
  - `PlayerType pt` (enum: BATTER, BOWLER, WICKET_KEEPER, CAPTAIN)

---

## 📊 Scorecards

### 4. **BattingScoreCard**
- Tracks:
  - `totalRuns`, `totalBallsPlayed`, `noOfFours`, `noOfSixes`, `strikeRate`
- Method: `updateStats()` updates stats per ball.

### 5. **BowlingScoreCard**
- Tracks:
  - `totalOvers`, `runsGiven`, `wickets`, `dotBalls`, `extras`, `economy`
- Method: `updateStats()` updates stats per ball.

---

## 🎮 Controllers

### 6. **PlayerBattingController**
- Manages batting order.
- Contains:
  - `Queue<Player> yetToPlay`
  - `Player striker`, `nonStriker`
- Method: `rotateStrike()`

### 7. **PlayerBowlingController**
- Manages bowling rotation.
- Contains:
  - `Deque<Player> bowlers`
  - `Map<Player, Integer> oversVsBowlers`
  - `Player currentBowler`
- Method: `nextBowler()`

---

## 🧩 Match Format

### 8. **MatchType Interface**
- Methods:
  - `noOfOvers()`
  - `maxOversForBowler()`
- Implementations:
  - `ODI`, `T20`, `Test`

---

## 🏁 Game Flow

### 9. **Innings**
- Contains:
  - `Team batting`, `bowling`
  - `List<Over> overs`
- Method: `startInnings()`

### 10. **Over**
- Contains:
  - `int overNumber`
  - `List<Ball> balls`
- Method: `startOver()`

### 11. **Ball**
- Contains:
  - `int ballNumber`
  - `BallType bt` (DOT, NO_BALL, WIDE, NORMAL)
  - `RunType rt` (ZERO, SINGLE, DOUBLE, FOUR, SIX)
  - `Player bowledBy`, `battedBy`

---

### 11. **Match Controller**
- Manage toss and innings setup
- Trigger overs and ball deliveries
- Rotate strike and bowlers
- Track match progress

---





### 🏗️ Core Class Skeletons



---

## 🏏 Enums
```java
enum PlayerType { BATTER, BOWLER, WICKET_KEEPER, CAPTAIN }
enum BallType { DOT, NO_BALL, WIDE, NORMAL }
enum RunType { ZERO, SINGLE, DOUBLE, THREE, FOUR, SIX }
```

---

## 👤 Player
```java
class Player {
    String name;
    int age;
    String address;
    PlayerType pt;

    public Player(String name, int age, String address, PlayerType pt) {
        this.name = name;
        this.age = age;
        this.address = address;
        this.pt = pt;
    }
}
```

---

## 📊 Scorecards
```java
class BattingScoreCard {
    int totalRuns, totalBallsPlayed, noOfFours, noOfSixes;
    float strikeRate;

    public void updateStats(int runs, boolean isBoundary, boolean isSix) {
        totalRuns += runs;
        totalBallsPlayed++;
        if (isBoundary) noOfFours++;
        if (isSix) noOfSixes++;
        strikeRate = (totalBallsPlayed == 0) ? 0 : (float) totalRuns * 100 / totalBallsPlayed;
    }
}

class BowlingScoreCard {
    int totalOvers, runsGiven, wickets, dotBalls, extras;
    float economy;

    public void updateStats(int runs, boolean isDot, boolean isExtra) {
        runsGiven += runs;
        if (isDot) dotBalls++;
        if (isExtra) extras++;
        economy = (totalOvers == 0) ? 0 : (float) runsGiven / totalOvers;
    }
}
```

---

## 🏏 Team
```java
class Team {
    String name;
    Queue<Player> battingOrder;
    List<Player> allPlayers;
    PlayerBattingController pc;
    PlayerBowlingController bc;

    public Team(String name, Queue<Player> battingOrder, List<Player> allPlayers) {
        this.name = name;
        this.battingOrder = battingOrder;
        this.allPlayers = allPlayers;
        this.pc = new PlayerBattingController(battingOrder);
        this.bc = new PlayerBowlingController(allPlayers);
    }
}
```

---

## 🎮 Controllers
```java
class PlayerBattingController {
    Queue<Player> yetToPlay;
    Player striker, nonStriker;

    public PlayerBattingController(Queue<Player> players) {
        this.yetToPlay = players;
        striker = yetToPlay.poll();
        nonStriker = yetToPlay.poll();
    }

    public void rotateStrike() {
        Player temp = striker;
        striker = nonStriker;
        nonStriker = temp;
    }
}

class PlayerBowlingController {
    Deque<Player> bowlers;
    Map<Player, Integer> oversVsBowlers = new HashMap<>();
    Player currentBowler;

    public PlayerBowlingController(List<Player> bowlersList) {
        this.bowlers = new ArrayDeque<>(bowlersList);
        this.currentBowler = bowlers.peek();
    }

    public void nextBowler() {
        bowlers.addLast(bowlers.removeFirst());
        currentBowler = bowlers.peek();
    }
}
```

---

## 🧩 Match Types
```java
interface MatchType {
    int noOfOvers();
    int maxOversForBowler();
}

class ODI implements MatchType {
    public int noOfOvers() { return 50; }
    public int maxOversForBowler() { return 10; }
}

class T20 implements MatchType {
    public int noOfOvers() { return 20; }
    public int maxOversForBowler() { return 4; }
}

class Test implements MatchType {
    public int noOfOvers() { return 90; }
    public int maxOversForBowler() { return Integer.MAX_VALUE; }
}
```

---

## 🎯 Ball and Over
```java
class Ball {
    int ballNumber;
    BallType bt;
    RunType rt;
    Player bowledBy;
    Player battedBy;

    public Ball(int ballNumber, BallType bt, RunType rt, Player bowledBy, Player battedBy) {
        this.ballNumber = ballNumber;
        this.bt = bt;
        this.rt = rt;
        this.bowledBy = bowledBy;
        this.battedBy = battedBy;
    }
}

class Over {
    int overNumber;
    List<Ball> balls = new ArrayList<>();

    public void startOver() {
        System.out.println("Starting Over " + overNumber);
    }
}
```

---

## 🏁 Innings with Real-Time Over Tracking
```java
class Innings {
    Team batting, bowling;
    List<Over> overs = new ArrayList<>();
    int currentOverNumber = 0;

    public void startInnings() {
        System.out.println("Innings started for " + batting.name);
        startNextOver();
    }

    public void startNextOver() {
        Over over = new Over();
        over.overNumber = ++currentOverNumber;
        over.startOver();
        overs.add(over);
    }
}
```

---

## 🏆 Match
```java
class Match {
    Team teamA, teamB;
    Date dt;
    String venue;
    Innings[] innings = new Innings[2];
    MatchType type;
    Team tossWon;

    public Match(Team teamA, Team teamB, MatchType type, String venue) {
        this.teamA = teamA;
        this.teamB = teamB;
        this.type = type;
        this.venue = venue;
        this.dt = new Date();
    }

    public void startMatch() {
        tossWon = teamA; // simulate toss
        innings[0] = new Innings();
        innings[0].batting = tossWon;
        innings[0].bowling = (tossWon == teamA) ? teamB : teamA;
        innings[0].startInnings();
    }
}
```

---




## 🏗️ `MatchController` Class
```java
class MatchController {
    Match match;
    int ballsPerOver = 6;

    public MatchController(Match match) {
        this.match = match;
    }

    public void startMatch() {
        System.out.println("Match started at " + match.venue + " on " + match.dt);
        match.tossWon = match.teamA; // simulate toss
        System.out.println("Toss won by: " + match.tossWon.name);

        // First Innings
        match.innings[0] = new Innings();
        match.innings[0].batting = match.tossWon;
        match.innings[0].bowling = (match.tossWon == match.teamA) ? match.teamB : match.teamA;
        match.innings[0].startInnings();

        simulateInnings(match.innings[0], match.type.noOfOvers());
    }

    private void simulateInnings(Innings innings, int totalOvers) {
        for (int i = 0; i < totalOvers; i++) {
            innings.startNextOver();
            Over currentOver = innings.overs.get(i);
            simulateOver(currentOver, innings.batting.pc, innings.bowling.bc);
        }
    }

    private void simulateOver(Over over, PlayerBattingController battingController, PlayerBowlingController bowlingController) {
        Player bowler = bowlingController.currentBowler;
        System.out.println("Bowler: " + bowler.name);

        for (int i = 1; i <= ballsPerOver; i++) {
            Ball ball = new Ball(i, BallType.NORMAL, RunType.SINGLE, bowler, battingController.striker);
            over.balls.add(ball);
            System.out.println("Ball " + i + ": " + ball.rt + " by " + ball.battedBy.name);

            // Update scorecards (optional logic)
            boolean isBoundary = ball.rt == RunType.FOUR;
            boolean isSix = ball.rt == RunType.SIX;
            int runs = getRunValue(ball.rt);
            battingController.strikerScore.updateStats(runs, isBoundary, isSix);

            // Rotate strike if odd run
            if (runs % 2 == 1) battingController.rotateStrike();
        }

        bowlingController.nextBowler();
    }

    private int getRunValue(RunType rt) {
        switch (rt) {
            case SINGLE: return 1;
            case DOUBLE: return 2;
            case THREE: return 3;
            case FOUR: return 4;
            case SIX: return 6;
            default: return 0;
        }
    }
}
```

---

