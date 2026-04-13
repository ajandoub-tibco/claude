# claude

This repository holds the `CLAUDE.md` instructions and a set of custom slash commands for the EBX Core App project, designed to be used with Claude Code.

## Prerequisites

### Atlassian MCP Server

The `/ebx-support` command requires the Atlassian MCP server. Add it with:

```bash
claude mcp add --transport http atlassian https://mcp.atlassian.com/v1/mcp
```

### GitHub CLI

The GitHub MCP server is tricky to set up. Instead, install and authenticate the GitHub CLI (`gh`), which Claude Code can use directly:

```bash
gh auth login
```

## Commands

### `/ebx-review-before-commit`

Performs an exhaustive senior-engineer code review of the current git diff. Checks code style, EBX conventions, architecture, design patterns, security (OWASP Top 10), performance, and unit test coverage. Compiles changed modules and runs affected tests. Produces a per-file findings table with severity levels and an overall verdict.

### `/ebx-support <ticket>`

Investigates a Jira support ticket end-to-end. Fetches the ticket via Atlassian MCP, classifies the issue (code-bug, configuration, documentation, environment, or usage), and searches the EBX codebase for relevant code. For code bugs it produces a patch file; for other issues it writes a detailed solution proposal with root cause analysis and resolution steps.

### `/ebx-doc-generate <subject> -- <entry-points>`

Generates DocBook XML documentation in the `ebx-core-doc` repository from source code in the `ebx-core-app` repository. Discovers Java sources matching the entry points, extracts the public API surface, and writes an `.xdb` file in the correct guide section. Also handles index registration, property documentation, release notes, and backward-compatibility entries.

### `/ebx-add-container-property <pattern>`

Adds EBX properties to the Docker container configuration. Given a property pattern (e.g. `ebx.directory.saml2.*`), finds all matching properties in the codebase, adds them to `dockerfile-first-stage.sh` with proper container variable mappings and defaults, and documents the new environment variables in the `running_the_image.adoc` documentation.

### `/ebx-help`

Displays the list of available Claude Code commands with descriptions and usage examples.