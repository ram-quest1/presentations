# Using PMD and SpotBugs in a Gradle Project

A practical guide to integrating static code analysis tools into your Java Gradle project.

---

## Overview

**PMD** analyzes source code for common programming flaws like unused variables, empty catch blocks, and unnecessary object creation.

**SpotBugs** (successor to FindBugs) analyzes bytecode to detect potential bugs such as null pointer dereferences, infinite loops, and resource leaks.

---

## Setup

### Adding Plugins to `build.gradle`

```groovy
plugins {
    id 'java'
    id 'pmd'
    id 'com.github.spotbugs' version '6.0.7'
}

// PMD Configuration
pmd {
    consoleOutput = true
    toolVersion = "7.0.0"
    rulesMinimumPriority = 5
    ruleSets = [
        'category/java/errorprone.xml',
        'category/java/bestpractices.xml'
    ]
}

// SpotBugs Configuration
spotbugs {
    ignoreFailures = true
    showStackTraces = true
    showProgress = true
    effort = 'max'
    reportLevel = 'low'
}

// Generate HTML reports for SpotBugs
tasks.withType(com.github.spotbugs.snom.SpotBugsTask) {
    reports {
        html {
            required = true
            outputLocation = file("${buildDir}/reports/spotbugs/${name}.html")
        }
        xml {
            required = false
        }
    }
}
```

---

## Running the Tools

### Execute PMD

```bash
# Run PMD on main source set
./gradlew pmdMain

# Run PMD on test source set
./gradlew pmdTest

# Run all PMD tasks
./gradlew pmd
```

### Execute SpotBugs

```bash
# Run SpotBugs on main source set
./gradlew spotbugsMain

# Run SpotBugs on test source set
./gradlew spotbugsTest

# Run all SpotBugs tasks
./gradlew spotbugs
```

### Run All Analysis Together

```bash
# Run both PMD and SpotBugs
./gradlew pmd spotbugs

# Run as part of the check task
./gradlew check
```

---

## Report Locations

After running the analysis tools, reports are generated in the `build/reports` directory:

| Tool     | Report Location                          |
|----------|------------------------------------------|
| PMD      | `build/reports/pmd/main.html`            |
| PMD Test | `build/reports/pmd/test.html`            |
| SpotBugs | `build/reports/spotbugs/main.html`       |
| SpotBugs Test | `build/reports/spotbugs/test.html`  |

### Viewing Reports

```bash
# Linux
xdg-open build/reports/pmd/main.html
xdg-open build/reports/spotbugs/main.html

# macOS
open build/reports/pmd/main.html
open build/reports/spotbugs/main.html

# Windows
start build/reports/pmd/main.html
start build/reports/spotbugs/main.html
```

---

## Integrating with CI/CD

Add to your CI pipeline configuration:

```yaml
# Example GitHub Actions step
- name: Run Static Analysis
  run: ./gradlew pmd spotbugs --continue

- name: Upload Reports
  uses: actions/upload-artifact@v3
  if: always()
  with:
    name: analysis-reports
    path: |
      build/reports/pmd/
      build/reports/spotbugs/
```

---

## Common Issues and Solutions

### Issue: SpotBugs task fails with "No classes found"

Ensure you run `compileJava` first or use the proper task dependencies:

```bash
./gradlew compileJava spotbugsMain
```

### Issue: PMD rules not found

Make sure you're using valid ruleset paths for your PMD version. PMD 7.x uses different paths than 6.x:

```groovy
// PMD 7.x
ruleSets = ['category/java/bestpractices.xml']

// PMD 6.x (legacy)
ruleSets = ['java-basic', 'java-braces']
```

### Issue: Out of memory during analysis

Increase heap size for the analysis tasks:

```groovy
tasks.withType(Pmd) {
    jvmArgs = ['-Xmx1g']
}

tasks.withType(com.github.spotbugs.snom.SpotBugsTask) {
    jvmArgs = ['-Xmx1g']
}
```

---

## Quick Reference

| Command | Description |
|---------|-------------|
| `./gradlew pmdMain` | Run PMD on main sources |
| `./gradlew spotbugsMain` | Run SpotBugs on main sources |
| `./gradlew check` | Run all verification tasks |
| `./gradlew pmd spotbugs --continue` | Run both, continue on failure |

---

## Additional Resources

- [PMD Documentation](https://pmd.github.io/)
- [PMD Rule Reference](https://pmd.github.io/latest/pmd_rules_java.html)
- [SpotBugs Documentation](https://spotbugs.readthedocs.io/)
- [SpotBugs Gradle Plugin](https://github.com/spotbugs/spotbugs-gradle-plugin)
- [SpotBugs Bug Descriptions](https://spotbugs.readthedocs.io/en/latest/bugDescriptions.html)