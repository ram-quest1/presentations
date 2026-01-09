# Getting Started with Gradle: Your First Java Command Line App

*A no-nonsense guide for beginners*

---

So you've landed your internship, and someone mentioned "Gradle" in a meeting. You nodded along, but inside you're wondering what on earth that is. Don't worry—we've all been there. This guide will walk you through creating your very first Java command line application using Gradle, step by step.

## What Even Is Gradle?

Think of Gradle as your project's personal assistant. It handles all the boring stuff:

- Compiling your Java code
- Managing dependencies (libraries your code needs)
- Running tests
- Packaging your app

Instead of manually running `javac` commands and keeping track of classpaths, Gradle does it all for you with simple commands.

Gradle is a build and dependency management system used for Java or JVM projects in general. Think of it as a superset of requirements.txt with additional stuff for building and assembling your compiled code.   

## Before We Start

Make sure you have these installed:

1. **Java JDK** (version 17 or higher recommended)
2. **Gradle** (version 8.0 or higher)

You can check by running:

```bash
java --version
gradle --version
```

If you see version numbers, you're good to go!

## Step 1: Create Your Project

Open your terminal and navigate to where you want your project to live. Then run:

```bash
mkdir my-first-app
cd my-first-app
gradle init
```

Gradle will ask you a series of questions. Here's what to pick:

```
Select type of build to generate:
  1: Application    <-- Pick this one (usually option 1 or 2)

Select implementation language:
  1: Java           <-- Pick Java

Select build script DSL:
  1: Kotlin
  2: Groovy         <-- Pick Groovy (it's simpler for beginners)

Select test framework:
  1: JUnit 4        <-- Pick JUnit 4 (simplest option)

Project name: my-first-app    <-- Just press Enter or type a name

Source package: com.example    <-- Press Enter or type your package name

Generate build using new APIs: no    <-- Pick no for now
```

That's it! Gradle just created a complete project structure for you.

## Step 2: Explore What Gradle Created

Run `ls` or `dir` and you'll see something like:

```
my-first-app/
├── app/
│   ├── build.gradle           # Build configuration for your app
│   └── src/
│       ├── main/
│       │   └── java/
│       │       └── com/example/
│       │           └── App.java    # Your main application code!
│       └── test/
│           └── java/
│               └── com/example/
│                   └── AppTest.java  # Tests go here
├── gradle/                    # Gradle wrapper files
├── gradlew                    # Gradle wrapper script (Linux/Mac)
├── gradlew.bat                # Gradle wrapper script (Windows)
└── settings.gradle            # Project settings
```

The important file is `app/src/main/java/com/example/App.java`. Let's look at it:

```java
package com.example;

public class App {
    public String getGreeting() {
        return "Hello World!";
    }

    public static void main(String[] args) {
        System.out.println(new App().getGreeting());
    }
}
```

It's just a simple Hello World program. Nothing scary!

## Step 3: Build Your Project

Building means compiling your Java code into something that can actually run. From your project root directory, run:

```bash
./gradlew build
```

> **Windows users**: Use `gradlew.bat build` instead.

> **Pro tip**: We use `./gradlew` instead of `gradle` because the "wrapper" ensures everyone on the team uses the same Gradle version. It's a best practice you'll see everywhere.

You should see output ending with:

```
BUILD SUCCESSFUL
```

Congrats! Your code compiled without errors.

## Step 4: Run Your Application

Now for the fun part—actually running it:

```bash
./gradlew run
```

You should see:

```
> Task :app:run
Hello World!

BUILD SUCCESSFUL
```

Your first Gradle-built Java app is running!

## Common Gradle Commands You'll Use Daily

Here's a quick reference card you can bookmark:

| Command | What It Does |
|---------|--------------|
| `./gradlew build` | Compiles code and runs tests |
| `./gradlew run` | Runs your application |
| `./gradlew clean` | Deletes all build outputs (start fresh) |
| `./gradlew test` | Runs only the tests |
| `./gradlew tasks` | Shows all available tasks |
| `./gradlew build --info` | Build with detailed output (great for debugging) |

## Let's Make It Actually Do Something

Let's modify the app to be a simple command line greeter. Open `app/src/main/java/com/example/App.java` and replace its contents with:

```java
package com.example;

public class App {
    
    public static void main(String[] args) {
        if (args.length == 0) {
            System.out.println("Usage: provide your name as an argument");
            System.out.println("Example: ./gradlew run --args='Alice'");
            return;
        }
        
        String name = args[0];
        System.out.println("=================================");
        System.out.println("  Welcome, " + name + "!");
        System.out.println("  Today is: " + java.time.LocalDate.now());
        System.out.println("  Have a great day!");
        System.out.println("=================================");
    }
}
```

Now run it with an argument:

```bash
./gradlew run --args='YourName'
```

Output:

```
> Task :app:run
=================================
  Welcome, YourName!
  Today is: 2025-01-09
  Have a great day!
=================================
```

You just built a command line app that accepts user input!

## Understanding build.gradle

The `app/build.gradle` file is where the magic happens. Here's what the important parts mean:

```groovy
plugins {
    id 'application'    // Tells Gradle this is a runnable app
}

repositories {
    mavenCentral()      // Where to download dependencies from
}

dependencies {
    // Your dependencies go here
    // Example: implementation 'com.google.guava:guava:32.1.1-jre'
}

application {
    mainClass = 'com.example.App'   // Which class has main()
}
```

If you ever need to add a library, you add it to the `dependencies` block.

## When Things Go Wrong

Here are solutions to common problems:

**"Permission denied" when running ./gradlew**
```bash
chmod +x gradlew
```

**"JAVA_HOME is not set"**
Set your Java path. On Mac/Linux add to your `.bashrc` or `.zshrc`:
```bash
export JAVA_HOME=$(/usr/libexec/java_home)
```

**"Could not find or load main class"**
Make sure your `mainClass` in `build.gradle` matches your actual package and class name.

**Build fails with no clear error**
Try cleaning and rebuilding:
```bash
./gradlew clean build --info
```

## Quick Recap

1. `gradle init` creates a project structure for you
2. Your code lives in `app/src/main/java/`
3. `./gradlew build` compiles everything
4. `./gradlew run` executes your app
5. Pass arguments with `--args='your arguments here'`

That's really all you need to get started. Gradle has a ton of features, but you don't need to learn them all at once. Start with these basics, and pick up more as you need them.

Now go build something cool!

---

*Have questions? Ask your team lead or check out the [official Gradle docs](https://docs.gradle.org/current/userguide/userguide.html).*