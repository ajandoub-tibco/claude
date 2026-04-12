# EBX Core App — Claude Code Guidelines

## Project Overview

This is an EBX (TIBCO/Cloud Software Group) Java monorepo with two Maven profiles active by default:

**Core JARs (`jars` profile):**
`ebx-ui-resources`, `ebx-core`, `ebx-workflow-core`, `ebx-dma-core`, `ebx-dataservices-core`, `ebx-staging-core`, `ebx-staging-cli`

**WARs & Other (`other` profile):**
`ebx-war`, `ebx-root-war`, `ebx-dma-war`, `ebx-ui-war`, `ebx-manager-war`, `ebx-hub-war`, `ebx-ide-war`, `ebx-authentication-war`, `ebx-dataservices-war`, `ebx-mcp-war`, `ebx-assistant-war`, `ebx-dev`, `ebx-runner`, `ebx-test-core`, `ebx-staging-integration-test-core`

## Build & Test

### Full Project

```bash
# Build everything (all modules, all profiles)
mvn clean install

# Build without tests
mvn clean install -DskipTests

# Only compile (no tests, no package)
mvn clean compile

# Run all tests across all modules
mvn test
```

### Single Module

Use `-pl <module>` to target a specific module. Replace `<module>` with any module name (e.g., `ebx-core`, `ebx-mcp-war`, `ebx-workflow-core`, `ebx-dataservices-core`, etc.):

```bash
# Build a single module
mvn clean install -pl <module>

# Build a single module without tests
mvn clean install -pl <module> -DskipTests

# Compile a single module only
mvn clean compile -pl <module>

# Run all tests in a single module
mvn test -pl <module>

# Run a specific test class in a module
mvn test -pl <module> -Dtest=MyTestClass

# Run a specific test method
mvn test -pl <module> -Dtest=MyTestClass#testMethodName

# Run multiple test classes
mvn test -pl <module> -Dtest="TestClassA,TestClassB"

# Run tests matching a pattern
mvn test -pl <module> -Dtest="Mcp*Test"
```

### Multiple Modules

```bash
# Build specific modules together
mvn clean install -pl ebx-core,ebx-mcp-war

# Build a module and all modules it depends on
mvn clean install -pl <module> -am

# Build a module and all modules that depend on it
mvn clean install -pl <module> -amd
```

### Useful Flags

```bash
# Skip compilation of tests (faster than -DskipTests)
-Dmaven.test.skip=true

# Resume build from a specific module (after a failure)
mvn clean install -rf :<module-artifactId>

# Offline mode (use local repo only)
mvn clean install -o

# Show full stack trace on errors
mvn clean install -e

# Debug output
mvn clean install -X
```

## Generated Tests — Must Compile and Pass

When you generate or modify test files, you **must** compile and run them to verify they work:

1. **Compile the test** to catch syntax or import errors:
   ```bash
   mvn test-compile -pl <module>
   ```
2. **Run the test** to verify it passes:
   ```bash
   mvn test -pl <module> -Dtest=MyGeneratedTest
   ```
3. If the test **fails to compile or run**, fix it immediately — do not leave broken tests behind.
4. Never consider a test generation task complete until the test is **green**.

## Code Diagnostics — Mandatory After Every Code Change

After writing or modifying any `.java` file, you **must** run diagnostics before considering the task complete.

### Step 1: Try IntelliJ MCP Diagnostics

Use the `mcp__ide__getDiagnostics` tool to get real-time diagnostics from the IDE:

```
mcp__ide__getDiagnostics(uri: "file:///C:/Workspace/ebx-core-app-b1/path/to/File.java")
```

- Run this on **every file you created or modified**.
- If IntelliJ returns issues, fix them all before moving on.
- If the tool **times out or is unavailable**, proceed to Step 2.

### Step 2: Fallback — Manual Diagnostics (IntelliJ/SonarQube-style)

When IntelliJ MCP is unavailable, perform the following checks manually on every changed file. These mirror what IntelliJ inspections and SonarQube would flag.

#### 2.1 Compilation & Import Checks
- All referenced classes must exist and be imported
- No unused imports
- All method calls must match their signatures (correct parameter count and types)
- No unresolved symbols or types

#### 2.2 Null Safety
- No potential `NullPointerException` — add null checks or use `Optional`
- Never dereference a value that could be null without checking first
- Use `Optional` for nullable return values

#### 2.3 Resource Management
- All `Closeable`/`AutoCloseable` resources must use try-with-resources
- No unclosed streams, connections, or readers

#### 2.4 Type Safety
- No raw types (`List` instead of `List<String>`) — always use generics
- No unchecked casts without `@SuppressWarnings("unchecked")` and a comment explaining why

#### 2.5 Cognitive Complexity (max 16)
- No method should exceed a cognitive complexity of **16**
- Flatten with guard clauses and early returns
- Extract sub-logic into private helper methods
- Use streams instead of nested loops with conditionals

#### 2.6 Common SonarQube Rules

| Rule ID | Description | Fix |
|---------|-------------|-----|
| S1192 | Duplicated string literals (3+) | Extract to `private static final String` constant |
| S2259 | Null pointer dereference | Add null checks or `Optional` |
| S1135 | TODO/FIXME left in code | Resolve or remove |
| S1066 | Collapsible `if` statements | Merge conditions |
| S3776 | Cognitive complexity too high | Refactor (see 2.5) |
| S1854 | Useless assignment to local variable | Remove dead assignments |
| S112 | Generic exceptions (`Exception`, `Throwable`) thrown | Use specific exception types |
| S106 | `System.out` / `System.err` usage | Replace with `VM.log` |
| S2095 | Resources not closed | Use try-with-resources |
| S1452 | Wildcard return types (`? extends`) | Use concrete types |

#### 2.7 IDE Warnings

| Warning | Fix |
|---------|-----|
| Missing `@Override` | Add annotation on all overridden methods |
| `equals()` without `hashCode()` | Implement both |
| Non-static inner class that could be static | Make it `static` |
| Field can be `final` | Add `final` |
| Redundant modifiers (`public` in interface) | Remove |
| Unused local variables | Remove or prefix with `_` |

## EBX Coding Standards — Always Enforced

### Copyright Header

Every `.java` file must start with the correct copyright. The year must match the **current year**.

```java
/*
 * Copyright © 2000-2026. Cloud Software Group, Inc. All Rights Reserved.
 */
```

### Method Parameter Naming

All method parameters must be `final` and follow prefix naming:

| Type | Prefix | Example |
|------|--------|---------|
| Consonant sound | `a` + PascalCase | `aName`, `aCount`, `aServer` |
| Vowel sound | `an` + PascalCase | `anEvent`, `anIndex`, `anObject` |
| Collection / array / varargs | Plural noun, no prefix | `servers`, `names`, `items` |

```java
// Correct
public void register(final String aName, final int aCount, final List<String> items) { }
public void handle(final Event anEvent, final Object anObject) { }
```

### Logging — Use VM.log Only

Never use Log4j, SLF4J, `java.util.logging`, or `LoggingCategory.getKernel()`.

```java
import com.onwbp.boot.VM;

VM.log.kernelInfo("message");
VM.log.kernelError("error message", exception);
VM.log.kernelWarn("warning message");
VM.log.kernelDebug("debug message");
```

### Explicit `this.` for Instance Fields

All instance field access must use `this.` prefix:

```java
// Correct
this.name = aName;
return this.name + "_label";
```

### Visibility

Apply the most restrictive visibility possible:
- `private` for internal helpers and fields
- `protected` only for intentional inheritance
- `public` only for API surface
- Entity/model fields must be `private` with getters/setters

### Modern Java Style

- Prefer streams over manual loops for filtering/mapping
- Use `String.join()` or `Collectors.joining()` for concatenation
- Use `isEmpty()` instead of `size() == 0` or `length() == 0`
- Use text blocks for multiline strings
- Use `var` for local variables with obvious types

