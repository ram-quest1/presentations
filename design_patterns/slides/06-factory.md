## Factory Design Pattern

**Goal:** Centralize object creation logic in one place, hiding the complexity from the caller.

> **When to use:** Object creation is complex, involves pooling/caching, or requires configuration.

--

### The Problem

```java
// Without Factory - creation logic scattered everywhere
public class UserService {
    public void saveUser(User user) {
        // Every class creates its own connection - messy!
        Connection conn = new Connection();
        conn.setUrl("jdbc:mysql://localhost:3306/mydb");
        conn.setUsername("admin");
        conn.setPassword("secret");
        conn.setTimeout(30000);
        conn.connect();
        
        // ... use connection
        
        conn.close(); // Connection destroyed - wasteful!
    }
}
```

**Problems:**

- Connection setup code duplicated everywhere
- Creating new connections is SLOW (network handshake)
- No reuse - connections destroyed after each use
- Configuration scattered across codebase

--

### Solution: Factory + Object Pool

```text
┌─────────────────────────────────────────────────────────┐
│                  ConnectionFactory                       │
│  ┌─────────────────────────────────────────────────┐    │
│  │              Connection Pool                     │    │
│  │  ┌──────┐  ┌──────┐  ┌──────┐  ┌──────┐        │    │
│  │  │ Conn │  │ Conn │  │ Conn │  │ Conn │        │    │
│  │  │  #1  │  │  #2  │  │  #3  │  │  #4  │        │    │
│  │  └──────┘  └──────┘  └──────┘  └──────┘        │    │
│  └─────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────┘
        │                                     ▲
        │ getConnection()                     │ releaseConnection()
        ▼                                     │
   ┌─────────┐       uses            ┌─────────────┐
   │ Service │ ───────────────────► │ Connection  │
   └─────────┘                       └─────────────┘
```

> Factory manages a **pool** of pre-created connections. Borrow one, use it, return it.

--

### Step 1: The Connection Class

```java
public class Connection {
    
    private String url;
    private boolean inUse;
    private long lastUsed;
    
    public Connection(String url) {
        this.url = url;
        this.inUse = false;
        this.lastUsed = System.currentTimeMillis();
        
        // Simulate slow connection setup
        System.out.println("Creating new connection to: " + url);
        try {
            Thread.sleep(100); // Pretend this takes time
        } catch (InterruptedException e) {}
    }
    
    public void connect() {
        System.out.println("Connected to: " + url);
    }
    
    public void executeQuery(String sql) {
        this.lastUsed = System.currentTimeMillis();
        System.out.println("Executing: " + sql);
    }
    
    public boolean isInUse() {
        return inUse;
    }
    
    public void setInUse(boolean inUse) {
        this.inUse = inUse;
    }
    
    public long getLastUsed() {
        return lastUsed;
    }
}
```

--

### Step 2: The Factory with Object Pool

```java
public class ConnectionFactory {
    
    // Configuration
    private static final String DB_URL = "jdbc:mysql://localhost:3306/mydb";
    private static final int POOL_SIZE = 5;
    
    // The pool of reusable connections
    private static List<Connection> connectionPool = new ArrayList<>();
    
    // Lock for thread safety
    private static final Object lock = new Object();
    
    // Initialize pool when class loads
    static {
        System.out.println("Initializing connection pool...");
        for (int i = 0; i < POOL_SIZE; i++) {
            connectionPool.add(new Connection(DB_URL));
        }
        System.out.println("Pool ready with " + POOL_SIZE + " connections");
    }
    
    // Private constructor - use static methods only
    private ConnectionFactory() {}
    
    // Factory method - get a connection from the pool
    public static Connection getConnection() {
        synchronized (lock) {
            for (Connection conn : connectionPool) {
                if (!conn.isInUse()) {
                    conn.setInUse(true);
                    System.out.println("Borrowed connection from pool");
                    return conn;
                }
            }
            // All connections busy - could wait, expand pool, or throw
            throw new RuntimeException("No available connections in pool!");
        }
    }
    
    // Return connection to the pool
    public static void releaseConnection(Connection conn) {
        synchronized (lock) {
            conn.setInUse(false);
            System.out.println("Connection returned to pool");
        }
    }
}
```

--

### Why Object Pooling?

```text
WITHOUT Pool (create/destroy each time):
──────────────────────────────────────────
Request 1: [====CREATE====][use][destroy]
Request 2:                   [====CREATE====][use][destroy]
Request 3:                                    [====CREATE====][use][destroy]

Total time: 3 × creation time + 3 × use time


WITH Pool (reuse connections):
──────────────────────────────────────────
Startup:   [==CREATE ALL==]
Request 1:                  [use]
Request 2:                       [use]
Request 3:                            [use]

Total time: 1 × creation time + 3 × use time  ✓ MUCH FASTER
```

> **Pool Benefit:** Pay the creation cost ONCE at startup, then reuse forever.

--

### Step 3: Enhanced Factory with Wait Queue

```java
public class ConnectionFactory {
    
    private static final String DB_URL = "jdbc:mysql://localhost:3306/mydb";
    private static final int POOL_SIZE = 5;
    private static final long MAX_WAIT_MS = 5000; // Wait up to 5 seconds
    
    private static List<Connection> connectionPool = new ArrayList<>();
    private static final Object lock = new Object();
    
    static {
        for (int i = 0; i < POOL_SIZE; i++) {
            connectionPool.add(new Connection(DB_URL));
        }
    }
    
    private ConnectionFactory() {}
    
    public static Connection getConnection() {
        synchronized (lock) {
            long startTime = System.currentTimeMillis();
            
            while (true) {
                // Try to find an available connection
                for (Connection conn : connectionPool) {
                    if (!conn.isInUse()) {
                        conn.setInUse(true);
                        return conn;
                    }
                }
                
                // No connection available - wait or timeout
                long elapsed = System.currentTimeMillis() - startTime;
                if (elapsed >= MAX_WAIT_MS) {
                    throw new RuntimeException(
                        "Timeout waiting for connection after " + MAX_WAIT_MS + "ms"
                    );
                }
                
                try {
                    // Wait and let other threads release connections
                    lock.wait(MAX_WAIT_MS - elapsed);
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                    throw new RuntimeException("Interrupted while waiting for connection");
                }
            }
        }
    }
    
    public static void releaseConnection(Connection conn) {
        if (conn == null) return;
        
        synchronized (lock) {
            conn.setInUse(false);
            lock.notify(); // Wake up one waiting thread
        }
    }
}
```

--

### Using the Factory

```java
public class UserService {
    
    public void saveUser(User user) {
        Connection conn = null;
        try {
            // Get connection from factory - simple!
            conn = ConnectionFactory.getConnection();
            
            conn.executeQuery("INSERT INTO users VALUES ('" + user.getName() + "')");
            
        } finally {
            // ALWAYS release back to pool
            ConnectionFactory.releaseConnection(conn);
        }
    }
}

public class OrderService {
    
    public void createOrder(Order order) {
        Connection conn = null;
        try {
            // Same simple call - factory handles everything
            conn = ConnectionFactory.getConnection();
            
            conn.executeQuery("INSERT INTO orders VALUES (" + order.getId() + ")");
            
        } finally {
            ConnectionFactory.releaseConnection(conn);
        }
    }
}
```

> **Notice:** Services don't know about pool size, URLs, or connection setup. Factory handles it all!

--

### Try-With-Resources Pattern

```java
// Make Connection implement AutoCloseable for cleaner code
public class Connection implements AutoCloseable {
    
    // ... existing code ...
    
    @Override
    public void close() {
        // Return to pool instead of destroying
        ConnectionFactory.releaseConnection(this);
    }
}

// Now services can use try-with-resources
public class UserService {
    
    public void saveUser(User user) {
        try (Connection conn = ConnectionFactory.getConnection()) {
            conn.executeQuery("INSERT INTO users VALUES ('" + user.getName() + "')");
        }
        // Automatically returned to pool when block exits!
    }
}
```

--

### Complete Example: Running It

```java
public class Application {
    
    public static void main(String[] args) {
        System.out.println("=== Application Starting ===\n");
        
        // Pool created automatically (static block)
        
        UserService userService = new UserService();
        OrderService orderService = new OrderService();
        
        // These reuse pooled connections - fast!
        userService.saveUser(new User("Alice"));
        userService.saveUser(new User("Bob"));
        orderService.createOrder(new Order(1001));
        userService.saveUser(new User("Charlie"));
        
        System.out.println("\n=== All Done ===");
    }
}
```

**Output:**

```text
=== Application Starting ===

Initializing connection pool...
Creating new connection to: jdbc:mysql://localhost:3306/mydb
Creating new connection to: jdbc:mysql://localhost:3306/mydb
Creating new connection to: jdbc:mysql://localhost:3306/mydb
Creating new connection to: jdbc:mysql://localhost:3306/mydb
Creating new connection to: jdbc:mysql://localhost:3306/mydb
Pool ready with 5 connections

Borrowed connection from pool
Executing: INSERT INTO users VALUES ('Alice')
Connection returned to pool

Borrowed connection from pool
Executing: INSERT INTO users VALUES ('Bob')
Connection returned to pool

... (reuses same connections)

=== All Done ===
```

--

### Factory vs Singleton

| Aspect | Singleton | Factory |
|--------|-----------|---------|
| **Purpose** | One instance of a class | Centralized object creation |
| **Creates** | Itself (one time) | Multiple objects |
| **Returns** | Always same instance | May return different instances |
| **Use case** | Logger, Config | Connection pool, Object pool |

> 💡 **Pro tip:** They work great together! A Factory can BE a Singleton.

--

### Factory Pattern Checklist

| Component | Purpose |
|-----------|---------|
| `private` constructor | Prevents direct instantiation |
| `static` factory method | Single point for getting objects |
| Object pool (optional) | Reuses expensive objects |
| `synchronized` blocks | Thread-safe pool access |
| `releaseConnection()` | Returns objects to pool |

--

**Remember:**

1. **Centralize creation** - all object setup in one place
2. **Hide complexity** - callers don't know about pooling
3. **Always release** - use try-finally or try-with-resources
4. **Thread safety** - synchronize pool access
5. **Configure once** - URL, pool size, etc. in factory only
