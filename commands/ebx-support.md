Investigate and resolve the Jira support ticket **$ARGUMENTS**.

Follow every step below in order.

---

## 1. Fetch the ticket

Use the Atlassian MCP tools to retrieve the full issue details:

1. Get accessible resources via `mcp__atlassian__getAccessibleAtlassianResources` to obtain the `cloudId`.
2. Fetch the issue via `mcp__atlassian__getJiraIssue` with `responseContentFormat: "markdown"`.
3. Also fetch comments via `mcp__atlassian__addCommentToJiraIssue` is NOT needed — instead read all available fields from the issue response.

Record the following for use in later steps:
- **Summary** and **full description**
- **Customer version** of EBX
- **Reproduction steps** described by the customer or support
- **Error messages, stack traces, or log excerpts** included in the description or attachments
- **Investigation history** — what has already been tried

---

## 2. Classify the issue

Based on the ticket content, classify the issue into one of these categories:

| Category | Description |
|----------|-------------|
| **code-bug** | A defect in the EBX codebase that can be fixed with a code change |
| **configuration** | Misconfiguration of EBX properties, deployment descriptors, or environment setup |
| **documentation** | Missing, incorrect, or misleading documentation |
| **environment** | Issue with the customer's infrastructure (JDK, app server, OS, IdP, network, etc.) |
| **usage** | Customer is not following the correct workflow or API usage |

State your classification and reasoning clearly before proceeding.

---

## 3. Investigate the codebase

Regardless of classification, search the EBX codebase for relevant code:

1. Identify **keywords** from the ticket: feature names, property keys, class names, error messages, log messages.
2. Use `Grep` and `Glob` to locate the relevant source files. Search broadly — check:
   - Property keys mentioned in the ticket (e.g., `ebx.saml`, `saml2`, `sp-descriptor`)
   - Error messages or log strings from any attached logs
   - Class names or packages related to the feature area
   - Configuration loading and initialization code
3. **Read the relevant source files in full** to understand the logic, initialization flow, error handling, and edge cases.
4. If the feature spans multiple classes, trace the call chain from entry point (servlet init, module startup, property loading) through to the reported failure point.

---

## 4A. If code-bug — Create a patch

If you identified a bug in the code:

1. Determine the **root cause** — explain exactly which line(s) of code are wrong and why.
2. Implement the fix following all EBX coding standards from CLAUDE.md:
   - Copyright header with current year
   - `final` parameters with correct prefix naming (`aXxx` / `anXxx` / plural)
   - `this.` for instance fields
   - `VM.log` for logging
   - Most restrictive visibility
   - Modern Java style
3. Run diagnostics on every changed file:
   ```
   mcp__ide__getDiagnostics(uri: "file:///<absolute-path-to-project>/<path>")
   ```
4. Compile the changed modules:
   ```bash
   mvn test-compile -pl <module>
   ```
5. Run any existing related tests:
   ```bash
   mvn test -pl <module> -Dtest=<relevant-tests>
   ```
6. Generate the patch file:
   ```bash
   mkdir -p support
   git diff > support/$ARGUMENTS.patch
   ```
7. **Revert your working changes** after generating the patch so the workspace stays clean:
   ```bash
   git checkout -- .
   ```

---

## 4B. If not a code-bug — Write a solution proposal

If the issue is configuration, documentation, environment, or usage related:

Create a detailed markdown file at `support/$ARGUMENTS.md` with this structure:

```markdown
# $ARGUMENTS — <Issue Summary>

## Ticket Overview

- **Type:** Support Request for Assistance
- **Customer:** <customer name>
- **EBX Version:** <version>
- **Status:** <status>
- **Classification:** <your classification from Step 2>

## Problem Summary

<Clear, concise description of the problem in your own words>

## Root Cause Analysis

<Explain the root cause based on your investigation. Reference specific code, configuration, or documentation where relevant.>

## Proposed Solution

<Step-by-step instructions to resolve the issue. Be specific — include exact property names, file paths, configuration values, commands, etc.>

### Steps

1. <step>
2. <step>
3. ...

## Code References

<List the relevant source files you examined, with brief notes on what each does>

- `path/to/File.java` — <what it does and how it relates to the issue>

## Additional Notes

<Any caveats, related issues, or follow-up recommendations>
```

---

## 5. Final output

After completing either 4A or 4B, report to the user:

1. **Classification** and reasoning
2. **Root cause** summary
3. **What was produced**: either the patch file path or the markdown file path
4. **Key findings** from the code investigation