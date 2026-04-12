# Add EBX Container Property Skill

## Purpose

Given a base EBX property pattern (e.g., `ebx.directory.saml2.*`), this skill:

1. Finds all matching properties in the EBX codebase
2. Adds missing ones to `../ebx-ce\core\docker\resources\dockerfile-first-stage.sh` using the existing two-block convention
3. Documents the new Docker environment variables in `../ebx-core-doc\src\main\resources-asciidoc\advanced_unless_cloud\common\ece\running_the_image.adoc`

**Arguments:** `$ARGUMENTS` is the property base (e.g., `ebx.directory.saml2.*` or `ebx.directory.oidc.*`)

---

## Step 1 — Find matching properties

Search for all properties matching `$ARGUMENTS` in:
- `../ebx.properties` (primary source — contains commented and active properties)
- Any `*.properties` files under `../ebx-core-doc\src\main\resources-docbook\common\installation\ebx_properties\`

Collect every property key matching the pattern (strip the trailing `.*` to get the prefix, then find all lines like `#?<prefix>.<suffix>=`).

---

## Step 2 — Check which properties are already handled

Read `dockerfile-first-stage.sh` and collect all property names already passed to `set_property` in the **mapping section** (first block, before the comment `# These properties can be overridden using environment variables when running the container.`).

Identify which found properties are **not yet present** in the script. Only those need to be added.

---

## Step 3 — Derive names for each new property

For each new EBX property `ebx.directory.X.Y.Z` (or any `ebx.<prefix>.<rest>`):

### 3a — Container variable name
- Strip the `ebx.directory.` prefix (or `ebx.` for non-directory properties)
- Lowercase the rest
- If the remaining part contains **camelCase** segments (e.g., `hostName`, `bindDnOrUser`), split them into dot-separated lowercase words (e.g., `host.name`, `bind.dn.or.user`). When in doubt, keep the original casing lowercased without splitting.
- Prepend `ebx.container.`

Examples:
| EBX property | Container variable |
|---|---|
| `ebx.directory.saml2.default.sp.base.url` | `ebx.container.saml2.default.sp.base.url` |
| `ebx.directory.oidc.default.client.id` | `ebx.container.oidc.default.client.id` |
| `ebx.directory.ldap.default.hostName` | `ebx.container.ldap.default.hostname` |
| `ebx.directory.ldap.default.bindDnOrUser` | `ebx.container.ldap.default.bind.dn.or.user` |

### 3b — Docker environment variable name
- Take the container variable, strip `ebx.container.` prefix
- Replace all `.` with `_`
- Uppercase the result
- Prepend `EBX_`

Examples:
| Container variable | Docker env var |
|---|---|
| `ebx.container.saml2.default.sp.base.url` | `EBX_SAML2_DEFAULT_SP_BASE_URL` |
| `ebx.container.oidc.default.client.id` | `EBX_OIDC_DEFAULT_CLIENT_ID` |

### 3c — Default value
Use `""` (empty string) as default unless the EBX property has an obvious boolean default:
- If the property contains `enabled` and is a flag → default `"false"`
- If the property contains `signed` and is a boolean → default `"true"`
- Otherwise → `""`

---

## Step 4 — Edit dockerfile-first-stage.sh

The script has two distinct blocks. You **must** insert into both.

### Block 1 — Property-to-container-var mapping (around lines 93–274)

Find the group of `set_property` lines for the same property family (e.g., the `saml2` block ends at the last `ebx.directory.saml2.*` line). Insert the new lines **after** the last existing line of that family group, or before the next unrelated group if the family doesn't exist yet.

Format:
```bash
set_property "ebx.directory.saml2.default.new.field" '${ebx.container.saml2.default.new.field}'
```

Use single quotes around the container variable reference so `${}` is not expanded by bash.

### Block 2 — Container variable defaults (around lines 277–445)

Find the comment `# These properties can be overridden using environment variables when running the container.` — defaults go **after** this line, grouped with the same property family.

Format:
```bash
set_property "ebx.container.saml2.default.new.field" ""
```

**Insert both blocks atomically** — read the file, insert both changes, write back.

---

## Step 5 — Edit running_the_image.adoc

File: `../ebx-core-doc\src\main\resources-asciidoc\advanced_unless_cloud\common\ece\running_the_image.adoc`

### 5a — Find or create the section

Look for an existing section whose anchor ID corresponds to the property family:
- `saml2` → `[[saml2_connectivity]]`
- `oidc` → `[[oidc_connectivity]]`
- `bearer` → `[[bearer_connectivity]]`
- `scim` → `[[scim_connectivity]]`
- `ldap` → `[[ldap_connectivity]]`
- `synchronized` → `[[synchronized_directory]]`

If no matching section exists, create one at the end of the `== Environment variables` chapter, using this template:

```asciidoc
[[<family>_connectivity]]
=== <Family> connectivity

The {ebx.product.name} <description> can be configured through the following environment variables:

[cols="8,6"]
|===
|Name |{ebx.product.fullname} main configuration file equivalent

|===

See link:../installation/properties.html[{ebx.product.name} configuration] for more information.
```

### 5b — Add table rows

For each new property, add a row to the table in the matching section:

```asciidoc
|EBX_SAML2_DEFAULT_NEW_FIELD
|ebx.directory.saml2.default.new.field

```

Insert **before** the `|===` closing line of the table. Maintain alphabetical or logical ordering consistent with adjacent rows.

---

## Step 6 — Report to the user

After making all changes, produce a compact summary:

```
Added N properties matching '<pattern>':

  dockerfile-first-stage.sh:
    set_property "ebx.directory.saml2.default.X"  →  ${ebx.container.saml2.default.X}   [default: ""]
    ...

  running_the_image.adoc  (section [[saml2_connectivity]]):
    EBX_SAML2_DEFAULT_X  →  ebx.directory.saml2.default.X
    ...
```

If a property was already present in the script, mention it as skipped.

---

## Important conventions to preserve

- Single quotes around `'${ebx.container.*}'` in mapping lines (prevents bash expansion)
- Double quotes around default values: `"false"`, `""`, `"true"`
- The `set_property` function handles commented-out, duplicate, and missing properties — no need to verify if the property exists in `ebx-default.properties`
- Do **not** modify any other part of `dockerfile-first-stage.sh` (the `unzip`, `mkdir`, `cp` sections must remain untouched)
- Do **not** add the property to `ebx-container.properties` (that file stays minimal)
- AsciiDoc tables use `|` at the start of each cell with a blank line between rows
