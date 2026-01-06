## Singleton Design Pattern

**Goal:** Ensure a class has only ONE instance throughout the application.

> **When to use:** Database connections, configuration managers, logging, caches.

--

### The Problem

```java
// Without Singleton - anyone can create multiple instances!
DatabaseConnection db1 = new DatabaseConnection();
DatabaseConnection db2 = new DatabaseConnection();
// db1 and db2 are DIFFERENT objects - wastes resources!
```

--

### Step 1: Naive Approach (NOT Thread-Safe)

```java
public class DatabaseConnection {
    
    // 1. Private static variable to hold the single instance
    private static DatabaseConnection instance;
    
    // 2. Private constructor - prevents "new DatabaseConnection()"
    private DatabaseConnection() {
        System.out.println("Connecting to database...");
    }
    
    // 3. Public method to get the instance
    public static DatabaseConnection getInstance() {
        if (instance == null) {
            instance = new DatabaseConnection();
        }
        return instance;
    }
    
    public void query(String sql) {
        System.out.println("Executing: " + sql);
    }
}
```

⚠️ **Problem:** Two threads can enter `if (instance == null)` simultaneously!

--

### The Race Condition Problem

```text
Thread A                         Thread B
────────                         ────────
if (instance == null) ──────────► if (instance == null)
    ↓ TRUE                            ↓ TRUE (same time!)
instance = new DB()              instance = new DB()
    ↓                                 ↓
Returns instance #1              Returns instance #2

❌ TWO instances created! Singleton violated.
```

> **Race Condition:** When multiple threads access shared data and the result depends on timing.

--

### Step 2: Synchronized Method (Thread-Safe but Slow)

```java
public static synchronized DatabaseConnection getInstance() {
    if (instance == null) {
        instance = new DatabaseConnection();
    }
    return instance;
}
```

✅ **Thread-safe:** `synchronized` locks the method - only one thread at a time.

❌ **Performance issue:** EVERY call to `getInstance()` acquires a lock, even after the instance exists!

```java
// These calls don't need locking - instance already exists!
DatabaseConnection db = DatabaseConnection.getInstance();  // waits for lock
DatabaseConnection db2 = DatabaseConnection.getInstance(); // waits for lock
```

--

### Step 3: Double-Checked Locking (Best Practice)

```java
public class DatabaseConnection {
    
    // volatile: ensures all threads see the updated value
    private static volatile DatabaseConnection instance;
    
    private String connectionUrl;
    
    private DatabaseConnection() {
        this.connectionUrl = "jdbc:mysql://localhost:3306/mydb";
        System.out.println("Connected to: " + connectionUrl);
    }
    
    public static DatabaseConnection getInstance() {
        // First check (no locking) - fast path
        if (instance == null) {
            // Lock only when instance might need creation
            synchronized (DatabaseConnection.class) {
                // Second check (with lock) - another thread may have created it
                if (instance == null) {
                    instance = new DatabaseConnection();
                }
            }
        }
        return instance;
    }
}
```

--

### Why Double-Checked Locking Works

```text
Thread A                              Thread B
────────                              ────────
if (instance == null) ✓
    synchronized(lock) ──────────────► if (instance == null) ✓
        if (instance == null) ✓           synchronized(lock) 🔒 WAITING...
            instance = new DB()               │
        }                                     │
    } ◄─── releases lock ─────────────────────┘
                                          if (instance == null) ✗ FALSE!
                                          // Skips creation
                                      }
return instance                       return instance (same one!)
```

**Key Points:**

- `volatile` prevents instruction reordering
- First `if` is fast - no lock needed after creation
- `synchronized` block only entered once (during creation)
- Second `if` catches threads that waited at the lock

--

### Complete Example: Logger Singleton

```java
public class Logger {
    
    private static volatile Logger instance;
    private java.io.PrintWriter writer;
    
    private Logger() {
        try {
            writer = new java.io.PrintWriter("app.log");
        } catch (Exception e) {
            writer = new java.io.PrintWriter(System.out);
        }
    }
    
    public static Logger getInstance() {
        if (instance == null) {
            synchronized (Logger.class) {
                if (instance == null) {
                    instance = new Logger();
                }
            }
        }
        return instance;
    }
    
    public void log(String message) {
        writer.println("[" + System.currentTimeMillis() + "] " + message);
        writer.flush();
    }
}
```

--

### Singleton Checklist

| Component | Purpose |
|--------|------|
| `private static volatile instance` | Holds the single instance, visible across threads |
| `private` constructor | Prevents `new ClassName()` from outside |
| `public static getInstance()` | Single access point to get the instance |
| Double-checked locking | Thread-safe without constant locking overhead |

**Remember:**

1. **Private constructor** - blocks external instantiation
2. **volatile keyword** - ensures visibility across threads
3. **Double null check** - performance + thread safety
4. **synchronized on class** - not on instance (it doesn't exist yet!)

> 💡 **Interview tip:** Always mention thread-safety when discussing Singleton!