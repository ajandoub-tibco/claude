# Generate EBX Documentation from Code

Generate documentation in the `ebx-core-doc` repository based on source code from the `ebx-core-app` repository.

- **Arguments:** `<subject> -- <entry-points>`
  - `<subject>`: The documentation topic (e.g. "AI Assistant", "MCP Server", "REST Toolkit")
  - `<entry-points>`: Comma-separated list of fully-qualified Java class names or package names from `ebx-core-app-b1`
- **Usage:** `/ebx-doc-generate AI Assistant -- com.orchestranetworks.ui.assistant, com.orchestranetworks.assistant`

---

## Step 1 — Parse arguments

Split `$ARGUMENTS` on the `--` delimiter:
- Everything before `--` is the **subject** (trim whitespace).
- Everything after `--` is a comma-separated list of **entry points** (classes or packages).

If no `--` is found or either side is empty, ask the user:
> Please provide both a subject and entry points separated by `--`.
> Example: `/ebx-doc-generate MCP Server -- com.orchestranetworks.mcp`

---

## Step 2 — Discover and read source code

For each entry point:

1. **If it looks like a package** (no uppercase segments or ends with `.*`):
   - Use Glob to find all `.java` files under the matching directory tree in `ebx-core-app-b1`.
     Pattern: `**/<package-as-path>/**/*.java` (convert dots to `/`).
   - Exclude `*Test.java`, `*Tests.java`, and files under `src/test/`.

2. **If it looks like a class** (last segment starts with an uppercase letter):
   - Use Glob to find the exact file: `**/<class-as-path>.java`.

3. Read every discovered file in full. For each file, extract:
   - **Public API surface**: public/protected classes, interfaces, enums, methods, constructors, constants.
   - **Javadoc comments**: descriptions, `@param`, `@return`, `@throws`, `@since`, `@deprecated`.
   - **Class hierarchy**: extends, implements, inner classes.
   - **Behavioral semantics**: what the code does, how classes interact, configuration, lifecycle.
   - **Deprecations and removals**: any `@Deprecated` annotations or methods that replace older API.

4. Build a **mental model** of the feature: its purpose, how a user or developer would use it, configuration steps, and extension points.

---

## Step 3 — Determine documentation placement

Based on the subject and the code analysis, decide where the documentation belongs in the `ebx-core-doc` structure.

1. Read the master index: `../ebx-core-doc/src/main/resources-filtered/docIndexDefinition.xml`

2. Identify the correct **guide** and **section**:
   - `user` guide — end-user features (UI, data management, workflows)
   - `ref` guide — technical references (built-in services, XPath, persistence)
   - `admin` guide — administration and configuration (installation, directory, scheduling, AI assistant)
   - `dev` guide — developer topics (data model, API, data services, REST, scripting)
   - `migration` guide — migration and backward compatibility

3. Identify the correct **directory** under `../ebx-core-doc/src/main/resources-docbook/common/` by examining which existing directories and files relate to the subject. Common directories:
   - `engine/` — admin features (repository, workflow, scheduling, AI assistant)
   - `models/` — data model concepts (types, constraints, triggers)
   - `references/` — technical references (built-in, permissions, XPath)
   - `data_services/` — REST and SOAP services
   - `user_interface/` — UI customization and user services
   - `installation/` — configuration and properties
   - `rest_toolkit/` — REST toolkit

4. Decide whether to:
   - **Create a new `.xdb` file** if the subject is not yet documented.
   - **Update an existing `.xdb` file** if there is already a chapter covering this subject.

5. Read any existing `.xdb` file that will be updated to understand its current content and style.

---

## Step 4 — Generate the XDB documentation

Write the documentation as a DocBook XML 5.0 `.xdb` file following these exact conventions:

### File template (new file)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE chapter PUBLIC "-//OASIS//DTD DocBook XML V5.0//EN" "../../../dtd/docbook.dtd">
<chapter audience="advanced" xmlns="http://docbook.org/ns/docbook"
 xmlns:xlink="http://www.w3.org/1999/xlink" xml:id="UNIQUE_ID">
 <sect1>
  <title>CHAPTER TITLE</title>
  <!-- content here -->
 </sect1>
</chapter>
```

### Formatting rules

- **Sections**: Use `<sect1>` for the main title, `<sect2>` for major sections, `<sect3>` for subsections.
- **Paragraphs**: `<para>` for text. Use `role="note"` for notes, `role="attention"` for warnings, `role="seealso"` for see-also references.
- **Code**: `<code>` for inline code (class names, method names, property keys, paths).
- **Emphasis**: `<emphasis role="bold">` for bold, `<emphasis role="strong">` for strong labels in definition lists.
- **Definition lists**: Use `<itemizedlist role="dl">` with `<listitem role="dt">` (term) and `<listitem role="dd">` (definition) pairs.
- **Ordered steps**: Use `<orderedlist>` with `<listitem>` children.
- **Bulleted lists**: Use `<itemizedlist>` with `<listitem>` children.
- **Java API links**: `<link javadoc="com/orchestranetworks/path/ClassName.html"/>` or with method: `<link javadoc="com/orchestranetworks/path/ClassName.html#methodName(param.Type)">display text</link>`.
- **Internal cross-references**: `<link xlink:href="../directory/filename.html#sectionId">link text</link>`.
- **Product name**: Always use `{ebx.product.name}` placeholder, never hardcode "EBX".
- **Tables**: Use `<table>` with `<tgroup>`, `<thead>`, `<tbody>`, `<row>`, `<entry>`.
- **Audience**: Set `audience="advanced"` for developer docs, `audience="advancedUnlessCloud"` for on-prem-only content. Omit for content shown everywhere.

### Content guidelines

- Write from the user/developer perspective — explain **what** the feature does and **how** to use it.
- Start with an introduction section explaining the purpose and typical use cases.
- Include configuration/setup steps as ordered lists.
- Document public API with Java API links (`<link javadoc="..."/>`).
- Include code examples or configuration snippets where helpful.
- End with notes, limitations, or see-also references.
- Match the tone and depth of existing EBX documentation (see `engine/admin_ai_assistant.xdb` or `models/constraints.xdb` as examples).

### Write the file

Write the `.xdb` file to the appropriate directory under `../ebx-core-doc/src/main/resources-docbook/common/`.

---

## Step 5 — Register in the index (if new file)

If you created a new `.xdb` file:

1. Edit `../ebx-core-doc/src/main/resources-filtered/docIndexDefinition.xml`.
2. Add a `<chapter>` element in the correct `<guide>` and `<section>`:
   ```xml
   <chapter localized="false" path="directory/filename.xdb" audience="advanced"/>
   ```
3. Place it logically near related chapters.

---

## Step 6 — Document new properties (if applicable)

Scan the source code discovered in Step 2 for EBX configuration properties. Look for:
- String literals matching the pattern `ebx.*` that are used as property keys (e.g. `EBXProperty.get("ebx.feature.enabled")`).
- Constants defined as property keys (fields with values like `"ebx.something.something"`).
- References to `System.getProperty(...)`, `EBXProperty`, `PropertiesHelper`, or similar property-reading utilities.
- The actual `ebx.properties` file at `ebx-core-app-b1/ebx-runner/src/main/ebx/ebx.properties` — search for properties related to the subject.

If new properties are found that are **not yet documented**:

### 6a — Create or update a `.properties` sample file

1. Read the existing property files in `../ebx-core-doc/src/main/resources-docbook/common/installation/ebx_properties/` to check if a file already covers the feature area.

2. **If a matching file exists** (e.g. `security_ebx.properties` for security properties), add the new properties to it following the existing comment style.

3. **If no matching file exists**, create a new `.properties` file using the naming convention `<feature>_ebx.properties` (e.g. `mcp_ebx.properties`, `assistant_ebx.properties`).

Follow this exact format for each property:

```properties
# Description of the property.
# Additional details or constraints if needed.
# Accepted values are:
#   - value1: explanation
#   - value2: explanation
# Default value is: defaultValue
#ebx.feature.property.name=defaultValue
```

Conventions:
- Properties are **commented out** with `#` prefix (showing default values).
- Group related properties under a `#####` section header comment.
- Document: purpose, accepted values, default value, and any dependencies on other properties.
- Use the EBX property naming convention: `ebx.<feature>.<aspect>.<detail>`.

### 6b — Reference from `properties.xdb`

1. Read `../ebx-core-doc/src/main/resources-docbook/common/installation/properties.xdb`.

2. Add a new `<sect2>` (or `<sect3>` if it belongs inside an existing `<sect2>`) in the appropriate location with:

```xml
<sect2 xml:id="feature_name_properties">
 <title>Configuring FEATURE NAME</title>
 <para>Description of what these properties configure and when to use them.</para>
 <para role="seealso"><link xlink:href="../directory/feature_doc.html"/></para>
 <programlisting role="properties" xlink:href="./ebx_properties/feature_ebx.properties"/>
</sect2>
```

If no new properties are found, skip this step and state why.

---

## Step 7 — Add to release notes (if applicable)

Determine whether the documented feature is **new or significantly changed** in the current version. Check:
- `@since` tags in the source code.
- Recent git commits touching the entry-point files: run `git log --oneline -20 -- <file-paths>` in the `ebx-core-app-b1` repo.
- Whether the feature already appears in the current release notes.

If a release note entry is warranted:

### 7a — Resolve the current version from Maven

1. Read `ebx-core-app-b1/.mvn/maven.config`.
2. Extract the version components:
   - `product.version.major` (e.g. `6`)
   - `product.version.minor` (e.g. `2`)
   - `product.version.service.pack` (e.g. `4`)
3. Build the **base version** as `{major}.{minor}` (e.g. `6.2`) — this determines which release notes file to use.
4. Build the **service pack version** as `{major}.{minor}.{service_pack}` (e.g. `6.2.4`) — this determines which `<sect3>` section to target.

### 7b — Find or create the version section in release notes

1. Read the release notes file matching the base version: `../ebx-core-doc/src/main/resources-docbook/common/releasenotes/{major}.{minor}.xdb` (e.g. `6.2.xdb`).

2. **Search for an existing `<sect3>` whose `<title>` starts with the service pack version** (e.g. `6.2.4`). The title may have a suffix like `RC12`, `RC14`, a build date, etc. — ignore the suffix. Match only on the `x.y.z` prefix.

   **IMPORTANT:** If a section already exists for this `x.y.z` version (e.g. `<title>6.2.4</title>` or `<title>6.2.4 RC12</title>`), **use it as-is**. Do NOT create a duplicate section with a different RC/build suffix. Multiple RC builds within the same service pack share one section.

3. **If no `<sect3>` exists for this `x.y.z` version**, create a new one as the **first `<sect3>`** inside the first `<sect2>` (so it appears at the top of the service packs list). Also update the `<sect1><title>` to reflect the new version. Use this template:

```xml
<sect3>
 <title>{major}.{minor}.{service_pack}</title>
 <para> <emphasis> <emphasis role="bold">{major}.{minor}.{service_pack}</emphasis> has been built on
      XXX 2026. Overview of this service pack: </emphasis> </para>
 <itemizedlist role="dl">
  <!-- entries go here -->
 </itemizedlist>
</sect3>
```

### 7c — Add the release note entry

Add a new `<listitem role="dt">` / `<listitem role="dd">` pair inside the `<itemizedlist role="dl">` of the target `<sect3>`:

```xml
<listitem role="dt">
 <para>FEATURE NAME</para>
</listitem>
<listitem role="dd">
 <para>Brief description of the new feature or change.
  See <link xlink:href="../directory/filename.html">Chapter title</link>
  for more information.</para>
</listitem>
```

If no release note is needed, skip this step and state why.

---

## Step 8 — Document backward compatibility (if broken)

Analyze the source code for backward-incompatible changes:
- **Removed public/protected methods or classes** (check `@Deprecated` annotations, git history for removals).
- **Changed method signatures** (parameter types, return types, thrown exceptions).
- **Changed default behaviors** that existing code depends on.
- **Removed or renamed configuration properties**.

If backward compatibility is broken:

1. Read the current API changes file: `../ebx-core-doc/src/main/resources-docbook/common/releasenotes/6.2-newJavaAPI.xdb`.

2. Add entries under the appropriate section (`Added elements`, `Removed elements`, `Changed elements`). For each change, add an entry in the correct package `<sect3>`:

```xml
<sect3 xml:id="com.orchestranetworks.package.name">
 <title>com.orchestranetworks.package.name</title>
 <itemizedlist>
  <listitem>
   <para>
    <emphasis role="strong">Class </emphasis>
    <link javadoc="com/orchestranetworks/path/ClassName.html">ClassName</link>
   </para>
   <para>Description of the class.</para>
  </listitem>
  <listitem>
   <para>
    <emphasis role="strong">Method </emphasis>
    <link javadoc="com/orchestranetworks/path/ClassName.html#methodName(param.types)">ClassName.methodName(ParamType)</link>
   </para>
   <para>Description of the method.</para>
  </listitem>
 </itemizedlist>
</sect3>
```

Use `Class`, `Interface`, `Enum`, `Method`, `Constructor`, or `Field` as the element type label.

3. If the breaking change requires migration steps, also update `../ebx-core-doc/src/main/resources-docbook/common/migration/development_tasks.xdb` with a migration note explaining what developers need to change in their code.

If no backward compatibility is broken, skip this step and state why.

---

## Step 9 — Summary

Output a summary table:

| Action | File | Status |
|--------|------|--------|
| Documentation | `path/to/file.xdb` | Created / Updated |
| Index registration | `docIndexDefinition.xml` | Updated / Not needed |
| Properties sample | `installation/ebx_properties/feature_ebx.properties` | Created / Updated / Not needed (reason) |
| Properties doc | `installation/properties.xdb` | Updated / Not needed (reason) |
| Release notes | `releasenotes/6.2.xdb` | Updated / Not needed (reason) |
| API changes | `releasenotes/6.2-newJavaAPI.xdb` | Updated / Not needed (reason) |
| Migration guide | `migration/development_tasks.xdb` | Updated / Not needed (reason) |

Then list any manual follow-up actions the user should take (e.g., adding screenshots, reviewing wording, building the docs with `mvn clean install` in `ebx-core-doc`).