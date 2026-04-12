# Claude Code Commands

Custom slash commands for the EBX Core App project.

## Available Commands

### `/ebx-review`

Performs an exhaustive senior-engineer code review of the current git diff.

- **Arguments:** None
- **Usage:** `/ebx-review`
- **What it does:**
  1. Gathers the full `git diff` (staged + unstaged)
  2. Reads every changed file in full for context
  3. Checks code style and conventions against CLAUDE.md rules (copyright, parameter naming, `this.` prefix, logging, visibility, imports)
  4. Runs IntelliJ MCP diagnostics on each changed `.java` file (falls back to manual checklist if unavailable)
  5. Evaluates architecture, design patterns, security (OWASP Top 10), and performance
  6. Assesses unit test coverage and identifies missing tests
  7. Compiles changed modules and runs affected tests
  8. Produces a per-file findings table with severity levels (Critical / Major / Minor / Info) and a summary with top 3 issues and an overall verdict

---

### `/ebx-support`

Investigates a Jira support ticket, analyzes the codebase, and produces either a patch or a solution proposal.

- **Arguments:** Jira ticket key (e.g. `EBXRFA-2638`)
- **Usage:** `/ebx-support EBXRFA-2638`
- **What it does:**
  1. Fetches the full ticket from Jira via the Atlassian MCP tools (summary, description, customer version, stack traces, investigation history)
  2. Classifies the issue as one of: `code-bug`, `configuration`, `documentation`, `environment`, or `usage`
  3. Searches the EBX codebase for relevant code using keywords from the ticket (property keys, error messages, class names)
  4. **If code-bug:** implements a fix, runs diagnostics and compilation, generates a patch file at `support/<ticket>.patch`, then reverts the workspace
  5. **If not a code-bug:** writes a detailed solution proposal at `support/<ticket>.md` with root cause analysis, step-by-step resolution, and code references

---

### `/ebx-doc-generate`

Generates documentation in the `ebx-core-doc` repository from source code in the `ebx-core-app` repository.

- **Arguments:** `<subject> -- <entry-points>` (subject and comma-separated classes/packages separated by `--`)
- **Usage:** `/ebx-doc-generate MCP Server -- com.orchestranetworks.mcp`
- **What it does:**
  1. Discovers and reads all Java source files matching the entry points (classes or packages) from `ebx-core-app-b1`
  2. Extracts the public API surface, Javadoc, class hierarchy, and behavioral semantics
  3. Determines the correct chapter and section in the `ebx-core-doc` structure (user guide, dev guide, admin guide, etc.)
  4. Generates a DocBook XML 5.0 `.xdb` file following EBX documentation conventions and writes it to `ebx-core-doc`
  5. Registers the new chapter in `docIndexDefinition.xml` if it is a new file
  6. Detects new `ebx.*` configuration properties and documents them: creates/updates a `.properties` sample file under `installation/ebx_properties/` and adds a reference section in `properties.xdb`
  7. Resolves the current version from `.mvn/maven.config`, finds the matching `x.y.z` section in release notes (or creates one if it doesn't exist), and adds an entry — never duplicates an existing `x.y.z` section even if the RC/build suffix differs
  8. Documents backward-incompatible API changes in `releasenotes/6.2-newJavaAPI.xdb` and `migration/development_tasks.xdb` if applicable
  9. Outputs a summary table of all actions taken and any manual follow-up needed