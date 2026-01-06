## Decorator Design Pattern

**Goal:** Add new behavior to an object WITHOUT modifying its original class.

> **When to use:** You want to extend functionality dynamically, without creating subclasses for every combination.

--

### The Problem

```java
import java.util.ArrayList;
import java.util.List;

// We have a regular ArrayList
List<String> names = new ArrayList<>();
names.add("Alice");
names.add("Bob");

// We want to share this list but prevent modifications
// Problem: Anyone can still do this!
names.add("Charlie");    // Works - list modified!
names.remove(0);         // Works - "Alice" removed!
names.clear();           // Works - everything gone!
```

**We need:** A way to make the list "read-only" without changing ArrayList itself.

--

### Solution: Wrap It with a Decorator

```text
┌─────────────────────────────────────────┐
│         ImmutableListDecorator          │
│  ┌───────────────────────────────────┐  │
│  │         Original ArrayList        │  │
│  │    ["Alice", "Bob", "Charlie"]    │  │
│  └───────────────────────────────────┘  │
│                                         │
│  ✓ get(index)    → allowed (read)      │
│  ✓ size()        → allowed (read)      │
│  ✗ add(item)     → blocked (write)     │
│  ✗ remove(index) → blocked (write)     │
│  ✗ clear()       → blocked (write)     │
└─────────────────────────────────────────┘
```

> The decorator WRAPS the original list and intercepts method calls.

--

### The Decorator (Using AbstractList)

```java
import java.util.AbstractList;
import java.util.List;

public class ImmutableListDecorator<T> extends AbstractList<T> {
    
    // The wrapped list (we're decorating this)
    private List<T> wrappedList;
    
    public ImmutableListDecorator(List<T> listToWrap) {
        this.wrappedList = listToWrap;
    }
    
    // READ operations - delegate to wrapped list (allowed)
    
    @Override
    public T get(int index) {
        return wrappedList.get(index);
    }
    
    @Override
    public int size() {
        return wrappedList.size();
    }
    
    // WRITE operations - AbstractList throws UnsupportedOperationException
    // by default for add(), remove(), set() - we get this for free!
}
```

> **Why AbstractList?** It implements `List<T>` and blocks write operations by default. We only override what we need.

--

### How It Works

```text
Caller                    Decorator                   ArrayList
──────                    ─────────                   ─────────

get(0) ─────────────────► get(0) ─────────────────► returns "Alice"
       ◄──────────────── "Alice" ◄─────────────────

add("Dan") ─────────────► BLOCKED! 
       ◄──────────────── throws UnsupportedOperationException

size() ─────────────────► size() ─────────────────► returns 3
       ◄──────────────── 3 ◄───────────────────────
```

> Read operations pass through. Write operations are intercepted and blocked.

--

### Using the Decorator

```java
import java.util.ArrayList;
import java.util.List;

public class Application {
    
    public static void main(String[] args) {
        
        // Create a regular ArrayList and add items
        List<String> names = new ArrayList<>();
        names.add("Alice");
        names.add("Bob");
        names.add("Charlie");
        
        System.out.println("Original list: " + names);
        System.out.println("\n--- Making list immutable ---\n");
        
        // Wrap it with the decorator
        List<String> immutableNames = new ImmutableListDecorator<>(names);
        
        // Reading works fine
        System.out.println("First name: " + immutableNames.get(0));
        System.out.println("List size: " + immutableNames.size());
        System.out.println("Contains Bob? " + immutableNames.contains("Bob"));
        
        // Writing throws exception
        try {
            immutableNames.add("Dan");
        } catch (UnsupportedOperationException e) {
            System.out.println("Blocked: Cannot add to immutable list");
        }
        
        try {
            immutableNames.remove(0);
        } catch (UnsupportedOperationException e) {
            System.out.println("Blocked: Cannot remove from immutable list");
        }
    }
}
```

--

### Output

```text
Original list: [Alice, Bob, Charlie]

--- Making list immutable --

First name: Alice
List size: 3
Contains Bob? true
Blocked: Cannot add to immutable list
Blocked: Cannot remove from immutable list
```

--

### Real-World Use: Sharing Data Safely

```java
import java.util.ArrayList;
import java.util.List;

public class UserDatabase {
    
    private List<String> users;
    
    public UserDatabase() {
        this.users = new ArrayList<>();
        users.add("admin");
        users.add("guest");
    }
    
    // Internal use - can modify
    public void addUser(String name) {
        users.add(name);
        System.out.println("Added user: " + name);
    }
    
    // External use - return immutable view
    public List<String> getUsers() {
        return new ImmutableListDecorator<>(users);
    }
}
```

```java
// Usage
UserDatabase db = new UserDatabase();
db.addUser("alice");

List<String> userList = db.getUsers();
System.out.println("Users: " + userList);   // Works - [admin, guest, alice]
userList.get(0);                            // Works - "admin"
userList.add("hacker");                     // BLOCKED!
```

> **Safe sharing:** Internal code can modify, external code cannot.

--

### Decorator Pattern Structure

```text
          ┌──────────────────┐
          │    List<T>       │  ◄── Java's List Interface
          │   (interface)    │
          └────────┬─────────┘
                   │
         ┌─────────┴─────────┐
         │                   │
         ▼                   ▼
┌─────────────────┐   ┌──────────────────┐
│  AbstractList   │   │    ArrayList     │
│ (abstract class)│   │    (concrete)    │
└────────┬────────┘   └──────────────────┘
         │
         ▼
┌──────────────────────┐
│ImmutableListDecorator│
│     (decorator)      │
│                      │
│  - wrappedList ──────┼──────► List<T>
└──────────────────────┘
```

**Key Parts:**

1. **Interface** (`List<T>`) - contract both must follow
2. **AbstractList** - provides default implementations
3. **Decorator** - wraps any `List<T>` and modifies behavior

--

### Java's Built-in Decorator

```java
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

// Java already has this decorator built-in!
List<String> names = new ArrayList<>();
names.add("Alice");
names.add("Bob");

// Collections.unmodifiableList() does exactly what we built!
List<String> immutable = Collections.unmodifiableList(names);

immutable.get(0);     // Works - "Alice"
immutable.add("Dan"); // Throws UnsupportedOperationException
```

> **Good to know:** `Collections.unmodifiableList()`, `unmodifiableSet()`, `unmodifiableMap()` are all decorators!

--

### Decorator vs Inheritance

```java
// INHERITANCE approach - need many subclasses
class ImmutableArrayList extends ArrayList { }
class SynchronizedArrayList extends ArrayList { }
class LoggingArrayList extends ArrayList { }
// What about ImmutableSynchronizedLoggingArrayList? Messy!

// DECORATOR approach - stack them at runtime
List<String> list = new ArrayList<>();
list = new ImmutableListDecorator<>(list);     // add immutability
list = new SynchronizedListDecorator<>(list);  // add thread-safety  
list = new LoggingListDecorator<>(list);       // add logging

// Mix and match without creating new classes!
```

| Aspect | Inheritance | Decorator |
|--------|-------------|-----------|
| **When decided** | Compile time | Runtime |
| **Flexibility** | Fixed | Mix and match |
| **Class explosion** | One per combination | One per feature |

--

### Decorator Pattern Checklist

| Component | Purpose |
|-----------|---------|
| Common interface (`List<T>`) | Both original and decorator implement it |
| Wrapped object | The original object being decorated |
| Delegating methods | Pass-through calls for unchanged behavior |
| Modified methods | Intercept and change specific behavior |

**Remember:**

1. **Same interface** - decorator and original are interchangeable
2. **Wrap, don't extend** - composition over inheritance
3. **Delegate most calls** - only change what you need to
4. **Stackable** - decorators can wrap other decorators