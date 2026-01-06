## Observer Design Pattern

**Goal:** When one object changes, automatically notify all objects that depend on it.

> **When to use:** Event systems, notifications, data binding, pub/sub messaging.

--

### The Problem

```java
// Stock price changes - need to notify multiple systems
public class Stock {
    private double price;
    
    public void setPrice(double price) {
        this.price = price;
        
        // Manually notify everyone - messy!
        emailService.send("Price changed to " + price);
        mobileApp.push("Price changed to " + price);
        dashboard.update("Price changed to " + price);
        tradingBot.check("Price changed to " + price);
        // What if we add more? Edit this class every time?
    }
}
```

--

**Problems:**

- Stock class knows about ALL listeners (tight coupling)
- Adding new listener = modify Stock class
- Can't add/remove listeners at runtime

--

### Solution: Observer Pattern

```text
         ┌─────────────┐
         │    Stock    │  (Subject)
         │  price: 150 │
         └──────┬──────┘
                │ notifies
       ┌────────┼────────┐
       ▼        ▼        ▼
  ┌────────┐ ┌────────┐ ┌────────┐
  │Investor│ │  App   │ │  Bot   │  (Observers)
  └────────┘ └────────┘ └────────┘

Stock doesn't know WHO is listening - just notifies everyone registered.
```

--

### Step 1: The Observer Interface

```java
// Anyone who wants updates implements this
public interface StockObserver {
    
    void update(String stockName, double price);
}
```

--

### Step 2: The Subject (Stock)

```java
import java.util.ArrayList;
import java.util.List;

public class Stock {
    
    private String name;
    private double price;
    private List<StockObserver> observers = new ArrayList<>();
    
    public Stock(String name, double price) {
        this.name = name;
        this.price = price;
    }
    
    public void addObserver(StockObserver observer) {
        observers.add(observer);
    }
    
    public void removeObserver(StockObserver observer) {
        observers.remove(observer);
    }
    
    public void setPrice(double price) {
        this.price = price;
        notifyObservers();
    }
    
    private void notifyObservers() {
        for (StockObserver observer : observers) {
            observer.update(name, price);
        }
    }
    
    public double getPrice() {
        return price;
    }
}
```

--

### Step 3: Concrete Observers

```java
public class Investor implements StockObserver {
    
    private String name;
    
    public Investor(String name) {
        this.name = name;
    }
    
    public void update(String stockName, double price) {
        System.out.println(name + " notified: " + stockName + " is now $" + price);
    }
}

public class TradingBot implements StockObserver {
    
    private double buyThreshold;
    
    public TradingBot(double buyThreshold) {
        this.buyThreshold = buyThreshold;
    }
    
    public void update(String stockName, double price) {
        if (price < buyThreshold) {
            System.out.println("BOT: " + stockName + " below $" + buyThreshold + " - BUY!");
        } else {
            System.out.println("BOT: " + stockName + " at $" + price + " - holding");
        }
    }
}
```

--

### Using the Observer Pattern

```java
public class Application {
    
    public static void main(String[] args) {
        
        // Create stock
        Stock apple = new Stock("AAPL", 150.0);
        
        // Create observers
        Investor alice = new Investor("Alice");
        Investor bob = new Investor("Bob");
        TradingBot bot = new TradingBot(145.0);
        
        // Register observers
        apple.addObserver(alice);
        apple.addObserver(bob);
        apple.addObserver(bot);
        
        // Price changes - all observers notified automatically!
        System.out.println("--- Price drops to $144 ---");
        apple.setPrice(144.0);
        
        System.out.println("\n--- Price rises to $152 ---");
        apple.setPrice(152.0);
        
        // Bob unsubscribes
        System.out.println("\n--- Bob unsubscribes ---");
        apple.removeObserver(bob);
        
        System.out.println("\n--- Price drops to $140 ---");
        apple.setPrice(140.0);
    }
}
```

--

### Output

```text
--- Price drops to $144 --
Alice notified: AAPL is now $144.0
Bob notified: AAPL is now $144.0
BOT: AAPL below $145.0 - BUY!

--- Price rises to $152 --
Alice notified: AAPL is now $152.0
Bob notified: AAPL is now $152.0
BOT: AAPL at $152.0 - holding

--- Bob unsubscribes --

--- Price drops to $140 --
Alice notified: AAPL is now $140.0
BOT: AAPL below $145.0 - BUY!
```

> Bob stopped receiving updates after unsubscribing.

--

### Observer Pattern Structure

```text
      ┌────────────────┐
      │ StockObserver  │ ◄── Interface
      │  (interface)   │
      └───────┬────────┘
              │ implements
     ┌────────┴────────┐
     ▼                 ▼
┌──────────┐    ┌────────────┐
│ Investor │    │ TradingBot │
└──────────┘    └────────────┘

┌─────────────────────────────┐
│           Stock             │
│  - observers: List          │
│  + addObserver(observer)    │
│  + removeObserver(observer) │
│  + setPrice(price)          │ ──► notifies all observers
└─────────────────────────────┘
```

--

### Real-World Examples

```java
// Java Swing - button click listeners
button.addActionListener(e -> System.out.println("Clicked!"));

// Java PropertyChangeListener
bean.addPropertyChangeListener(e -> System.out.println("Changed!"));

// JavaScript - DOM events
element.addEventListener("click", () => console.log("Clicked!"));

// Android - LiveData observers
liveData.observe(owner, data -> updateUI(data));
```

> Observer pattern is EVERYWHERE in UI frameworks and event systems.

--

### Observer Pattern Checklist

| Component | Purpose |
|-----------|---------|
| Observer interface | Defines `update()` method |
| Concrete observers | React to notifications |
| Subject | Holds list, notifies on change |
| `addObserver()` | Subscribe to updates |
| `removeObserver()` | Unsubscribe from updates |

--

**Remember:**

1. **Loose coupling** - subject doesn't know observer types
2. **Dynamic** - add/remove observers at runtime
3. **One-to-many** - one subject, many observers
4. **Push vs Pull** - push data in update, or let observer pull
