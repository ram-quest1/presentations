## Proxy Design Pattern

**Goal:** Control access to an object by placing a "middleman" in front of it.

> **When to use:** Lazy loading, access control, logging, or validating before accessing the real object.

--

### The Problem

```java
// Database connection is EXPENSIVE to create
public class UserService {
    
    // Connection created immediately - even if never used!
    private Connection connection = new Connection("jdbc:mysql://localhost:3306/db");
    
    public void doSomething() {
        // Maybe we never even call a query...
        // But connection was already created - wasteful!
    }
    
    public void getUser(int id) {
        connection.executeQuery("SELECT * FROM users WHERE id = " + id);
    }
}
```

--

**Problems:**

- Connection created even if never used (slow startup)
- No way to check if connection is still alive before using
- No logging of what queries are being run
- No access control (anyone can run any query)

--

### Solution: Put a Proxy in Front

```text
┌──────────┐      ┌───────────────────┐      ┌────────────────┐
│  Client  │ ───► │  ConnectionProxy  │ ───► │ Real Connection │
└──────────┘      └───────────────────┘      └────────────────┘
                          │
                          │ Can do BEFORE calling real object:
                          │ • Create connection only when needed (lazy)
                          │ • Check if connection is alive
                          │ • Log the query
                          │ • Check user permissions
                          │
                          │ Can do AFTER calling real object:
                          │ • Log the result
                          │ • Cache the result
                          │ • Transform the result
```

> Proxy has the SAME interface as the real object - client doesn't know the difference.

--

### Step 1: Define the Interface

```java
// Both real connection and proxy implement this
public interface DatabaseConnection {
    
    void connect();
    
    String executeQuery(String sql);
    
    void close();
    
    boolean isConnected();
}
```

--

### Step 2: The Real Connection

```java
public class RealDatabaseConnection implements DatabaseConnection {
    
    private String url;
    private boolean connected;
    
    public RealDatabaseConnection(String url) {
        this.url = url;
        this.connected = false;
    }
    
    public void connect() {
        // Simulate slow connection (network handshake, authentication, etc.)
        System.out.println("Connecting to " + url + "...");
        try {
            Thread.sleep(1000); // Pretend this takes 1 second
        } catch (InterruptedException e) {}
        
        this.connected = true;
        System.out.println("Connected!");
    }
    
    public String executeQuery(String sql) {
        if (!connected) {
            throw new RuntimeException("Not connected!");
        }
        System.out.println("Executing: " + sql);
        return "Result for: " + sql;
    }
    
    public void close() {
        System.out.println("Connection closed");
        this.connected = false;
    }
    
    public boolean isConnected() {
        return connected;
    }
}
```

--

### Step 3: The Proxy (Lazy Initialization)

```java
public class LazyConnectionProxy implements DatabaseConnection {
    
    private String url;
    private RealDatabaseConnection realConnection;  // Created later!
    
    public LazyConnectionProxy(String url) {
        this.url = url;
        this.realConnection = null;  // NOT created yet
        System.out.println("Proxy created (no connection yet)");
    }
    
    // Create real connection only when first needed
    private void ensureConnected() {
        if (realConnection == null) {
            System.out.println("First use - creating real connection now...");
            realConnection = new RealDatabaseConnection(url);
            realConnection.connect();
        }
    }
    
    public void connect() {
        ensureConnected();
    }
    
    public String executeQuery(String sql) {
        ensureConnected();  // Lazy! Connect only if needed
        return realConnection.executeQuery(sql);
    }
    
    public void close() {
        if (realConnection != null) {
            realConnection.close();
            realConnection = null;
        }
    }
    
    public boolean isConnected() {
        return realConnection != null && realConnection.isConnected();
    }
}
```

--

### How Lazy Proxy Works

```text
WITHOUT Proxy (eager):
──────────────────────────────────────
new Connection()  →  [====CONNECT====]  →  ready
                     (1 second wasted even if never used)


WITH Proxy (lazy):
──────────────────────────────────────
new Proxy()  →  ready instantly!

... time passes, maybe never used ...

executeQuery()  →  [====CONNECT====]  →  execute  →  result
                   (connects only when actually needed)
```

> **Lazy Loading:** Delay expensive operations until absolutely necessary.

--

### Using the Lazy Proxy

```java
public class Application {
    
    public static void main(String[] args) {
        System.out.println("=== Application Starting ===\n");
        
        // Create proxy - NO connection made yet!
        DatabaseConnection db = new LazyConnectionProxy("jdbc:mysql://localhost/mydb");
        
        System.out.println("\nDoing other startup tasks...");
        System.out.println("Loading config...");
        System.out.println("Starting web server...\n");
        
        // NOW we need the database - connection happens here
        String result = db.executeQuery("SELECT * FROM users");
        System.out.println("Got: " + result);
        
        // Already connected - no delay
        result = db.executeQuery("SELECT * FROM orders");
        System.out.println("Got: " + result);
        
        db.close();
    }
}
```

--

### Output

```text
=== Application Starting ===

Proxy created (no connection yet)

Doing other startup tasks...
Loading config...
Starting web server...

First use - creating real connection now...
Connecting to jdbc:mysql://localhost/mydb...
Connected!
Executing: SELECT * FROM users
Got: Result for: SELECT * FROM users

Executing: SELECT * FROM orders
Got: Result for: SELECT * FROM orders

Connection closed
```

> Notice: Connection created AFTER "Starting web server" - only when first query runs.

--

### Proxy with Logging

```java
public class LoggingConnectionProxy implements DatabaseConnection {
    
    private DatabaseConnection realConnection;
    
    public LoggingConnectionProxy(DatabaseConnection connection) {
        this.realConnection = connection;
    }
    
    public void connect() {
        System.out.println("[LOG] Connecting...");
        realConnection.connect();
        System.out.println("[LOG] Connected successfully");
    }
    
    public String executeQuery(String sql) {
        long start = System.currentTimeMillis();
        System.out.println("[LOG] Query: " + sql);
        
        String result = realConnection.executeQuery(sql);
        
        long elapsed = System.currentTimeMillis() - start;
        System.out.println("[LOG] Completed in " + elapsed + "ms");
        
        return result;
    }
    
    public void close() {
        System.out.println("[LOG] Closing connection");
        realConnection.close();
    }
    
    public boolean isConnected() {
        return realConnection.isConnected();
    }
}
```

--

### Proxy with Access Control

```java
public class SecureConnectionProxy implements DatabaseConnection {
    
    private DatabaseConnection realConnection;
    private String currentUser;
    
    public SecureConnectionProxy(DatabaseConnection connection, String user) {
        this.realConnection = connection;
        this.currentUser = user;
    }
    
    public void connect() {
        realConnection.connect();
    }
    
    public String executeQuery(String sql) {
        // Block dangerous operations for non-admin users
        String sqlUpper = sql.toUpperCase();
        
        if (!currentUser.equals("admin")) {
            if (sqlUpper.contains("DROP") || sqlUpper.contains("DELETE")) {
                throw new SecurityException(
                    "User '" + currentUser + "' cannot run destructive queries!"
                );
            }
        }
        
        return realConnection.executeQuery(sql);
    }
    
    public void close() {
        realConnection.close();
    }
    
    public boolean isConnected() {
        return realConnection.isConnected();
    }
}
```

--

### Stacking Proxies

```java
public class Application {
    
    public static void main(String[] args) {
        
        // Start with lazy connection
        DatabaseConnection db = new LazyConnectionProxy("jdbc:mysql://localhost/mydb");
        
        // Wrap with logging
        db = new LoggingConnectionProxy(db);
        
        // Wrap with security
        db = new SecureConnectionProxy(db, "guest");
        
        // Now we have: Security → Logging → Lazy → Real Connection
        
        db.executeQuery("SELECT * FROM users");  // Works!
        db.executeQuery("DROP TABLE users");     // BLOCKED!
    }
}
```

--

### Proxy Pattern Structure

```text
         ┌────────────────────┐
         │ DatabaseConnection │  ◄── Common Interface
         │    (interface)     │
         └─────────┬──────────┘
                   │
       ┌───────────┴───────────┐
       │                       │
       ▼                       ▼
┌──────────────────┐   ┌─────────────────┐
│RealDatabaseConn  │   │ ConnectionProxy │
│   (real object)  │   │    (proxy)      │
└──────────────────┘   └────────┬────────┘
                                │
                                │ wraps
                                ▼
                       ┌────────────────────┐
                       │ DatabaseConnection │
                       └────────────────────┘
```

--

### Proxy vs Decorator

| Aspect | Proxy | Decorator |
|--------|-------|-----------|
| **Purpose** | Control access | Add behavior |
| **Creates object?** | Often creates/manages it | Receives existing object |
| **Focus** | When/if to access | What happens during access |
| **Examples** | Lazy load, security, caching | Logging, formatting, filtering |

> **Similar structure, different intent.** Proxy controls IF you access. Decorator changes WHAT happens.

--

### Types of Proxies

| Type | Purpose | Example |
|------|---------|---------|
| **Virtual Proxy** | Lazy initialization | Create DB connection on first query |
| **Protection Proxy** | Access control | Check user permissions |
| **Logging Proxy** | Track operations | Log all queries with timing |
| **Caching Proxy** | Cache results | Store query results, skip DB |
| **Remote Proxy** | Hide network calls | Make remote service look local |

--

### Proxy Pattern Checklist

| Component | Purpose |
|-----------|---------|
| Common interface | Proxy and real object implement same interface |
| Real object | The actual object that does the work |
| Proxy | Controls access to real object |
| Client | Uses interface, doesn't know if proxy or real |

--

**Remember:**

1. **Same interface** - client can't tell proxy from real object
2. **Control access** - decide when/if to call real object
3. **Add behavior** - logging, caching, validation before/after
4. **Stackable** - wrap proxy with another proxy
