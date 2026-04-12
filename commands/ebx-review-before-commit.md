Perform an exhaustive senior-engineer code review of the current git diff. Follow every section below in order and report findings in the format specified at the end.

---

## 1. Scope — Gather the diff

Run `git diff HEAD` (unstaged + staged) and `git diff --cached` (staged only). Identify every changed file. For each file, read the full file (not just the diff) so you can evaluate context, surrounding code, and module-level impact.

---

## 2. Code Style & Conventions (CLAUDE.md)

Enforce every rule from the project's CLAUDE.md. In particular check:

- **Copyright header** — must be `Copyright © 2000-<current year>. Cloud Software Group, Inc. All Rights Reserved.`
- **Parameter naming** — all parameters must be `final`; consonant-sound prefix `aXxx`, vowel-sound prefix `anXxx`, collections/arrays/varargs use plural nouns with no prefix
- **Explicit `this.`** — all instance field access must use `this.` prefix
- **Logging** — only `VM.log.kernel*()` is allowed; no Log4j, SLF4J, `java.util.logging`, `System.out`, `System.err`, `LoggingCategory.getKernel()`
- **Visibility** — most restrictive possible; fields must be `private` with getters/setters where needed
- **Modern Java style** — streams over manual loops, `String.join()` / `Collectors.joining()`, `isEmpty()` not `size() == 0`, text blocks for multiline strings, `var` for obvious local types
- **Imports** — no unused imports, no wildcard imports
- **`@Override`** — present on every overridden method

---

## 3. Code Diagnostics

For every changed `.java` file, run IntelliJ MCP diagnostics:

```
mcp__ide__getDiagnostics(uri: "file:///<absolute-path-to-project>/<path>")
```

If the tool times out or is unavailable, perform the manual diagnostics checklist from CLAUDE.md:

- Compilation & import checks (all symbols resolve, correct method signatures)
- Null safety (no unguarded dereferences, use `Optional` where appropriate)
- Resource management (try-with-resources for all `Closeable`/`AutoCloseable`)
- Type safety (no raw types, no unchecked casts without justification)
- Cognitive complexity (max 16 per method)
- SonarQube rules: S1192, S2259, S1135, S1066, S3776, S1854, S112, S106, S2095, S1452
- IDE warnings: missing `@Override`, `equals`/`hashCode` pairing, non-static inner classes, fields that can be `final`, redundant modifiers, unused variables

---

## 4. Architecture & Design

Evaluate the changes from an architectural perspective:

- **Responsibility** — does each class/method have a single, clear responsibility? Are concerns properly separated?
- **Dependencies** — are dependency directions correct? No circular dependencies? No inappropriate coupling between modules?
- **API surface** — are new public APIs minimal, well-named, and consistent with existing patterns? Could anything be package-private instead?
- **Abstraction level** — are abstractions at the right level? Not too granular (premature), not too coarse (god classes)?
- **EBX patterns** — are EBX APIs (`SessionContext`, `McpContext`, `Adaptation`, `SchemaNode`, `Request`, etc.) used idiomatically? No reinvention of existing EBX facilities?
- **Extensibility** — are extension points (interfaces, abstract classes) used where behavior may vary, and concrete classes used where it will not?
- **Module boundaries** — do changes respect the module structure (`ebx-core` vs WARs vs test modules)? No leaking of internal implementation across module boundaries?

---

## 5. Design Patterns

Identify design patterns in use and evaluate their appropriateness:

- Are well-known patterns (Builder, Factory, Strategy, Observer, Decorator, etc.) applied correctly, or are there anti-patterns?
- Could a pattern simplify complex conditional logic, object creation, or cross-cutting concerns?
- Are there missed opportunities: e.g., a switch on type that should be polymorphism, manual wiring that should be a registry, repeated instantiation that should be a factory?
- Is there over-engineering: unnecessary patterns applied where straightforward code would suffice?

---

## 6. Security

Check for vulnerabilities (OWASP Top 10 and beyond):

- **Injection** — SQL/XPath/LDAP injection via unsanitized input in queries or predicates
- **Authentication & authorization** — are access checks present and correct? Is `McpAccessManager` or EBX permission model used properly?
- **Sensitive data exposure** — passwords, tokens, or secrets logged, hardcoded, or returned in API responses?
- **Input validation** — are external inputs (HTTP parameters, JSON payloads, user-provided paths) validated and sanitized at system boundaries?
- **Resource exhaustion** — unbounded collections, missing pagination limits, uncontrolled recursion, missing timeouts?
- **Deserialization** — is untrusted data deserialized safely?
- **Path traversal** — are file paths constructed from user input without sanitization?
- **Error handling** — do error responses leak stack traces, internal paths, or implementation details?

---

## 7. Performance

Evaluate performance characteristics of the changed code:

- **Algorithmic complexity** — are there O(n^2) or worse patterns that could be O(n) or O(n log n)?
- **Database / adaptation access** — are there N+1 query patterns? Unnecessary full-table scans? Missing use of `RequestResult` pagination?
- **Memory** — are large collections loaded entirely when streaming or pagination would suffice? Are objects retained longer than needed?
- **Concurrency** — is shared mutable state properly synchronized? Are there race conditions or deadlock risks? Is thread-safety documented?
- **Caching** — are expensive computations repeated unnecessarily? Would caching help without introducing stale-data risks?
- **String operations** — repeated concatenation in loops instead of `StringBuilder` or `Collectors.joining()`?
- **I/O** — are streams, connections, and readers closed promptly? Are blocking calls on critical paths?

---

## 8. Unit Test Coverage

Evaluate test quality for the changed code:

- **Coverage** — does every new public method have corresponding test cases? Are edge cases covered (null inputs, empty collections, boundary values, error paths)?
- **Test isolation** — do tests depend on external state, ordering, or other tests? Are mocks/stubs used appropriately?
- **Assertions** — are assertions specific and meaningful (not just `assertNotNull`)? Do they verify behavior, not implementation?
- **Naming** — do test method names clearly describe the scenario and expected outcome?
- **Missing tests** — identify specific methods or branches that lack test coverage and suggest what tests to add
- **Test code quality** — do tests follow the same coding standards (parameter naming relaxed, but structure and readability still expected)?

---

## 9. Compilation verification

Compile all changed modules to verify no build errors:

```bash
mvn test-compile -pl <changed-modules>
```

If tests were added or changed, run them:

```bash
mvn test -pl <module> -Dtest=<TestClass>
```

---

## Output Format

For each changed file, produce a findings table:

```
## [FileName.java]

| # | Category | Location | Severity | Finding |
|---|----------|----------|----------|---------|
| 1 | Style | Line 12 | Minor | Missing `final` on parameter `name`, should be `final String aName` |
| 2 | Security | Line 45 | Critical | XPath predicate built from unsanitized user input — injection risk |
| 3 | Performance | Lines 78-92 | Major | N+1 adaptation lookups inside loop — batch with a single `RequestResult` |
| 4 | Architecture | Class level | Major | Helper class has 14 public methods — split by concern |
| 5 | Tests | — | Major | No test coverage for `resolveTable()` error path |
```

Severity levels:
- **Critical** — security vulnerability, data loss risk, or crash
- **Major** — bug, significant design flaw, or missing test coverage for critical path
- **Minor** — style violation, naming issue, or minor improvement opportunity
- **Info** — suggestion or observation, no action required

After all file tables, provide a **Summary** section with:
- Total finding count by severity
- Top 3 most impactful issues to address first
- Overall assessment (approve / approve with minor fixes / request changes)