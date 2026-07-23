---
name: java-17-to-21
description: Assess, audit, implement, and validate Java 17 to 21 migration and modernization changes on a local Maven or Gradle repository, with complete JEP-level coverage, risk-specific warnings, exact-diff approval for every safety category, and Anteroom execution permission
---

# Java 17 to 21 migration

Assess the current local Java repository and produce an evidence-based Java 17 to Java 21 migration report. Follow the `java-17-to-21-baseline`, `java-17-to-21-release-delta`, `java-17-to-21-modernization-policy`, and `java-17-to-21-pattern-catalog` instructions supplied by this pack.

Invocation arguments: {args}

Interpret an argument matching `validate`, `audit`, or `apply` as the corresponding mode. Treat `apply preview`, `apply --preview`, or the legacy `plan` argument as read-only apply preview. Otherwise use `assessment` mode. Arguments may also identify a local repository path, but never inspect a different repository unless the user explicitly supplied it.

## Usage

```text
/java-17-to-21
/java-17-to-21 validate
/java-17-to-21 audit
/java-17-to-21 apply
/java-17-to-21 apply --preview
```

- `assessment` (default) performs static, read-only discovery and does not execute repository code.
- `validate` performs the same assessment, proposes exact commands, and executes them only after explicit approval.
- `audit` performs static discovery plus modernization pattern analysis and categorized reporting. No file modification.
- `apply` performs discovery, categorized diff proposal, consequence disclosure, user selection, exact-diff approval, implementation across any approved safety category, command permission, validation, and final reporting.
- `apply --preview` performs the proposal phase without file modification or repository-code execution. The legacy `plan` argument is an alias for this preview.

## Operating contract

- Use `glob_files`, `grep`, and `read_file` for static discovery and evidence collection.
- Use `bash` only when shell execution is necessary. Follow the active Anteroom safety policy even for read-only shell commands.
- Use `ask_user` to obtain explicit approval before Maven, Gradle, wrapper, application, test, `jdeps`, or `jdeprscan` execution.
- Treat wrappers and build scripts as repository code. Approval for one command does not authorize materially different commands.
- Treat scope selection, exact-diff approval, and repository-code execution permission as sequential stages inside the single `apply` workflow. Selecting suggestions does not authorize edits, and approving edits does not authorize Maven, Gradle, wrapper, application, test, or analysis commands.
- Tool-use rules by mode:
  - `assessment`, `validate`, `audit`, and apply preview: do not use `write_file` or `edit_file`. Static discovery and, in `validate`, approved command execution only.
  - `apply`: `write_file` and `edit_file` are permitted for `Safe Mechanical`, `Review Required`, `Design Sensitive`, and `Experimental` changes only when (1) the proposal includes the required warnings and failure consequences, (2) the user selected the suggestion/category or selected `all proposed changes`, (3) the exact combined diff has been shown and approved via `ask_user`, and (4) every changed file is inside the current repository.
- `save_memory` and `run_agent` are prohibited in all modes.
- Do not install a JDK, build tool, or scanner. Dependency/plugin changes require categorized consequences and exact-diff approval; related downloads require execution permission. Do not commit, push, deploy, or update external systems unless the user separately requests that additional workflow.
- Do not send repository content to an online service merely to perform the assessment.
- Redact secrets and sensitive configuration values.
- Mark facts that cannot be resolved as `UNRESOLVED`; never guess effective values or compatibility.

## Workflow

### Step 1: Establish repository context

1. Use the current workspace unless an explicit local path was provided.
2. Discover Java source, build files, wrapper files, CI files, Dockerfiles, and nested modules with `glob_files`.
3. Use `grep` to locate Java-version properties, compiler settings, toolchains, plugins, dependencies, JVM flags, and risk patterns.
4. Use `read_file` to inspect relevant files and capture file-and-line evidence.
5. Detect Maven, Gradle, mixed builds, nested independent builds, and unsupported build systems.
6. Return `UNSUPPORTED` when no applicable Java repository exists or the repository is outside the baseline scope.

Do not assume the installed Java version is the repository's configured source, target, release, or runtime version.

### Step 2: Discover Java and build configuration

Inspect all applicable modules.

For Maven, inspect:

- POM modules, parents, profiles, properties, plugin management, and dependency management
- `maven.compiler.release`, compiler `<release>`, `<source>`, `<target>`, and `java.version`
- Maven Wrapper and `.mvn/` configuration
- Toolchains, compiler, Surefire, Failsafe, Javadoc, JaCoCo, packaging, and framework plugins

For Gradle, inspect:

- Groovy/Kotlin build and settings files, `gradle.properties`, version catalogs, and convention plugins
- Java toolchains, source/target compatibility, `options.release`, and test launchers
- Root, subproject, `buildSrc`, and included-build overrides
- Wrapper distribution version and relevant plugin versions

For delivery and runtime, inspect:

- CI configuration, Dockerfiles, buildpacks, deployment manifests, and startup scripts
- `.java-version`, `.sdkmanrc`, `.tool-versions`, toolchain declarations, and committed IDE settings
- JVM environment variables and flags without exposing secret values

Report contradictions separately. Do not run Maven or Gradle to calculate effective configuration in assessment mode.

### Step 3: Analyze migration areas

Create findings across the pattern families defined in `java-17-to-21-pattern-catalog`:

- Configuration (compiler/release, wrapper, plugins)
- Dependency and framework compatibility
- Runtime risk (internal JDK APIs, module access, deprecated/removed APIs, charset/locale, TLS/crypto, JNI/native)
- Language modernization (`instanceof` patterns, switch/pattern matching, records, sealed types, text blocks)
- API modernization (sequenced collections, stream/Optional cleanup)
- Concurrency modernization (virtual-thread candidates, `ThreadLocal` risk, pinning risk, structured concurrency trials)
- Delivery/runtime alignment (CI, container, packaging, deployment JDK)
- Validation coverage
- Design-sensitive patterns (public API changes, framework-managed executors, context propagation)

Refer to the pattern catalog for detection heuristics, evidence requirements, default classification, safety class, implementation approval, failure consequences, and required post-change validation per pattern.

Walk every entry in `java-17-to-21-release-delta`. For each JEP-level item record `APPLICABLE_FINDING`, `APPLICABLE_NO_FINDING`, `NOT_APPLICABLE`, `UNRESOLVED`, or `NOT_CHECKED`. Do not claim complete Java 17-to-21 update coverage unless every delta entry has a disposition.

Classify each finding according to the pack baseline:

- `REQUIRED`: repository or approved command evidence proves a migration change is needed.
- `ADVISORY`: a resilience or maintainability recommendation not proven to block Java 21.
- `UNRESOLVED`: available evidence cannot establish compatibility.
- `MODERNIZATION_CANDIDATE`: an optional source-level opportunity to adopt newer Java language, API, or concurrency features. Never a Java 21 readiness blocker.
- `EXPERIMENTAL`: a preview, incubating, or design-sensitive opportunity that requires explicit opt-in and separate validation. Never a required migration step.

In addition to classification, tag each finding with a safety class from the modernization policy: `Safe Mechanical`, `Review Required`, `Design Sensitive`, or `Experimental`.

For each finding also record:

- related release and JEP, when applicable
- evidence type: `STATIC` or `VALIDATED`
- confidence: `HIGH`, `MEDIUM`, or `LOW`
- applicability and release-delta disposition

Do not label a dependency incompatible solely because it is old. Exact version recommendations require an authoritative dated source or approved organizational baseline.

Treat `--add-opens` as evidence of reflective access to a strongly encapsulated package. It does not by itself prove use of an internal JDK API or that the flag can be removed.

Treat an unavailable JDK 21 or build tool as a blocked validation prerequisite, not as a required repository change.

### Step 4: Produce the assessment report

Start with a concise executive summary, followed by detailed evidence:

```text
Java 17 -> 21 Migration Assessment

Executive Summary
Overall status: <CHANGES REQUIRED | BLOCKED | PARTIALLY VALIDATED | UNSUPPORTED | READY>
Mode: <assessment | validate | audit | apply | apply preview>
Top required changes: <count and short list>
Validation blockers: <count and short list>
Highest risks: <short list>
Modernization progress: <count of candidates, none, or not audited>

Repository: <path>
Build: <Maven | Gradle | Mixed> <wrapper version or unresolved>
Configured Java: <value(s) with evidence>
Installed Java: <value, not checked, or unavailable>

Configuration Evidence
- <file:line/property -> detected value>

Required Changes
R1. <title>
    Pattern: <catalog pattern name>
    Release/JEP: <release and JEP, or not applicable>
    Evidence: <file:line/property/command>
    Evidence type: <STATIC | VALIDATED>
    Confidence: <HIGH | MEDIUM | LOW>
    Why: <Java 21 impact>
    Recommendation: <specific change or decision>
    Safety class: <Safe Mechanical | Review Required | Design Sensitive | Experimental>
    Implementation: <implementable after exact approval | BLOCKED pending evidence>
    Assumptions: <assumptions used to prepare the change>
    Failure consequences: <what may break or require rollback>
    Rollback: <how to revert>
    Validate: <check>

Advisory Recommendations
A1. <same structure>

Unresolved / High Risk
U1. <same structure plus missing evidence>

Modernization Candidates
M1. <title>
    Pattern: <catalog pattern name>
    Release/JEP: <release and JEP, or EARLIER_LANGUAGE_MODERNIZATION>
    Evidence: <file:line>
    Evidence type: <STATIC | VALIDATED>
    Confidence: <HIGH | MEDIUM | LOW>
    Why: <modernization value>
    Safety class: <Safe Mechanical | Review Required | Design Sensitive>
    Implementation: <implementable after exact approval>
    Assumptions: <assumptions used to prepare the change>
    Failure consequences: <what may break or require rollback>
    Rollback: <how to revert>
    Validate: <post-change check>

Experimental
E1. <same structure plus preview/incubator flags, supportability consequences, and explicit opt-in requirement>

Release-Delta Coverage
- <JEP>: <APPLICABLE_FINDING | APPLICABLE_NO_FINDING | NOT_APPLICABLE | UNRESOLVED | NOT_CHECKED> - <evidence or reason>

Implementation
- Selected: <IDs/categories/all/none>
- Rejected or deferred: <IDs and reason>
- Applied changes: <exact files and summary, or none>
- Accepted warnings/consequences: <by safety class>
- Rollback: <commands or edit list>
- Execution permission: <Allow Once | Allow for Session | Rejected | Not requested>

Validation
- <check>: NOT RUN | PASS | FAIL | BLOCKED | NOT APPLICABLE

Checks Not Performed
- <check and reason>

Suggested Next Steps
1. <ordered action>
```

For an empty finding section, write `None identified`. Do not paste long raw output; preserve only concise, relevant errors or warnings.

Static assessment cannot return `READY`. Use `CHANGES REQUIRED` when concrete required changes exist, `BLOCKED` when meaningful static assessment cannot proceed, `UNSUPPORTED` when outside scope, and otherwise `PARTIALLY VALIDATED`.

Report "Modernization progress" in a separate section from readiness status. A repository may be `READY` for Java 21 and still have outstanding modernization candidates.

### Step 4b: Modernization audit (audit mode)

In `audit` mode, walk the pattern catalog's modernization families (language, API, concurrency) against the repository. For each match, record:

- suggestion identifier (`M1`, `M2`, ...)
- pattern name and family from the catalog
- file path and line evidence, with surrounding context
- default classification (typically `MODERNIZATION_CANDIDATE` or `EXPERIMENTAL`)
- safety class
- category-specific warning and approval requirements from the catalog
- why it matters and expected impact
- assumptions, failure consequences, and rollback
- required post-change validation

Group results by classification and safety class. Do not present modernization findings as required changes.

### Step 4c: Apply: propose, select, implement, and validate

In `apply` mode follow the modernization policy state machine:

1. Generate proposed changes for every applicable finding that can be expressed as repository edits, including dependency, framework, public API, concurrency, runtime, and experimental changes when evidence is sufficient.
2. Group proposals by classification and safety class. For each proposal show its exact candidate diff, assumptions, affected modules, expected benefit, failure consequences, rollback, and required validation.
3. Let the user select individual suggestion IDs, one or more categories, `all proposed changes`, or `none`. Selection defines scope only.
4. Build one exact combined diff containing only the selected changes. Repeat and aggregate all applicable `Safe Mechanical`, `Review Required`, `Design Sensitive`, and `Experimental` warnings. If experimental changes are included, explicitly list preview/incubator flags and future-portability consequences.
5. Ask via `ask_user` for approval of the exact combined diff. Do not modify files if the response is refused or ambiguous.
6. After approval, use `edit_file` or `write_file` to apply only that diff. Prefer reversible batches and record a before/after change list with rollback instructions.
7. Continue directly to Step 5 to request execution permission, validate, and produce the final report.

In `apply --preview` or legacy `plan`, stop after step 4 and do not request edit approval, modify files, or execute repository code.

### Step 5: Prepare validation commands

In `validate` mode and after approved edits in `apply` mode, select commands from repository documentation and the operating system. Prefer wrappers. Show every proposed command, working directory, purpose, expected file/process/network side effects, dependency-download possibility, affected suggestion IDs, and stop conditions before execution.

```text
Proposed commands (execute repository code):
1. <JDK/build-tool version command>
2. <clean verification/build command>
3. <project-specific tests or startup command>
4. <jdeps command against built artifacts, if applicable>
5. <jdeprscan command against built artifacts, if applicable>

These commands may execute build plugins and download dependencies.
Proceed?
```

Use `ask_user` to confirm the proposed command scope, then invoke `bash` so Anteroom can present `Allow Once`, `Allow for Session`, or rejection according to the active safety configuration. Do not request `Allow Always` during the normal workflow. Do not treat the invocation word `validate` as permission to execute commands.

`Allow for Session` does not broaden the pack's task: remain inside the current repository and migration workflow, and explain materially different or expanded commands before running them even when the tool permission persists. If the user rejects execution after edits, retain the approved edits, mark runtime validation `NOT RUN`, and report the resulting status as no better than `PARTIALLY VALIDATED`.

Default proposals, only when repository documentation does not define commands:

- Maven Unix-like: `./mvnw clean verify`
- Maven Windows: `.\mvnw.cmd clean verify`
- Gradle Unix-like: `./gradlew clean build --no-daemon`
- Gradle Windows: `.\gradlew.bat clean build --no-daemon`

If no wrapper exists, report it and propose system Maven or Gradle only when already installed. Never install it.

Run `jdeps --jdk-internals` and `jdeprscan --for-removal` only after successful artifact creation and only against relevant artifacts. Do not invent a classpath or module path.

### Step 6: Execute approved validation

After approval:

1. Confirm `java -version` and `javac -version` report JDK 21.
2. Run only the approved commands with `bash`, in order.
3. Stop and ask before expanding the command set.
4. Record command, working directory, exit status, and concise relevant output.
5. Distinguish migration failures from pre-existing failures, missing credentials/network, unavailable tooling, and sandbox limits.
6. Do not fix failures automatically.
7. Stop the affected batch on failure. Propose any repair as a new categorized diff with consequences and approval.

If JDK 21 is unavailable, mark JDK-dependent checks `BLOCKED`. Continue reporting static repository findings; do not create a required-change finding merely because the local environment lacks Java.

### Step 7: Produce the final validation report

Reprint the full report and update each validation item to `PASS`, `FAIL`, `BLOCKED`, or `NOT APPLICABLE` with evidence.

- Return `READY` only when the complete baseline readiness gate passes.
- Return `CHANGES REQUIRED` when concrete repository changes remain.
- Return `BLOCKED` when a prerequisite prevents meaningful assessment or validation and no more specific status applies.
- Return `PARTIALLY VALIDATED` when static analysis succeeded but required runtime evidence remains incomplete.

Always report remaining unresolved and high-risk items, even when all approved commands pass.

For `apply`, also report selected and rejected suggestions, accepted warnings and consequences, exact changes applied, permission scope, stopped batches, repair proposals, and rollback instructions.

## Guidelines

- Prefer evidence over generic migration advice.
- Keep required, advisory, unresolved, validation-blocker, modernization, and experimental items distinct.
- Treat all safety classes as implementable after their required warnings, explicit selection, and exact-diff approval; never describe approval as proof of safety.
- Explain recommendations in plain language.
- Do not claim transitive dependency compatibility from direct versions alone.
- Do not assume successful compilation proves runtime, integration, performance, security, charset, or deployment compatibility.
- Do not present modernization candidates or experimental items as Java 21 readiness blockers. Report modernization progress separately from readiness status.
- Require developer and Java SME review of generated guidance and results.
