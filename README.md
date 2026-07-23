# fm-pack-java-modernization

An Anteroom pack for repeatable, evidence-based Java assessment and approval-gated end-to-end migration.

Version `0.3.0` provides `/java-17-to-21`, a workflow that inspects a local Java repository, proposes exact categorized diffs, explains risks and failure consequences, implements any user-approved safety category, requests command permission, validates the result, and produces a final report.

## Scope

Supported in this version:

- Java 17 to Java 21
- Maven and Gradle
- Gradle Groovy and Kotlin DSL
- Single-module and multi-module repositories
- Build, dependency, source, CI, container, and runtime configuration analysis
- Language, API, concurrency, and delivery/runtime modernization audit
- Complete JEP-level Java 17-to-21 release-delta coverage, including final, preview, incubator, and deprecated features
- Integrated proposal and diff preview inside `apply`
- Approval-gated implementation of Safe Mechanical, Review Required, Design Sensitive, and Experimental changes
- Dependency, framework, public API, concurrency, runtime, and preview/incubator changes when exact diffs, consequences, rollback, and validation are available
- Chat-only reports

Not supported in this version:

- Any file modification without explicit approval of the exact diff
- Silent or unapproved implementation
- Changes that cannot be expressed as a responsible repository-local diff because required evidence is unavailable
- Production deployment
- Organization-wide Java-version enforcement
- Android, Scala-only, or Kotlin-only modernization
- A guarantee of production compatibility without application-specific testing
- Preview/incubator feature adoption as a baseline migration requirement

## Usage

Start Anteroom from the root of the target repository:

```powershell
cd C:\path\to\java-repository
aroom web
```

Run a read-only assessment:

```text
/java-17-to-21
```

Run the assessment and request validation command approval:

```text
/java-17-to-21 validate
```

Run a modernization audit without changing files:

```text
/java-17-to-21 audit
```

Preview the complete apply proposal without changing files:

```text
/java-17-to-21 apply --preview
```

Run the end-to-end approval-gated migration workflow:

```text
/java-17-to-21 apply
```

The legacy `/java-17-to-21 plan` argument remains a read-only alias for `apply --preview`; planning is no longer a separate primary mode.

In `apply`, the skill inspects files, generates categorized exact diffs, and lets the user choose individual suggestions, categories, `all proposed changes`, or none. Every safety category is implementable after its warnings, failure consequences, rollback, and exact selected diff are approved.

The approval sequence has three distinct decisions:

1. **Selection:** choose which suggestions are in scope. This does not authorize edits.
2. **Edit approval:** review and approve the exact diff. This authorizes only those file edits.
3. **Execution permission:** after editing, allow the displayed build, test, application, or analysis command once or for the current Anteroom session. Repository commands can execute plugins, download dependencies, contact services, or have other side effects.

`Allow for Session` does not authorize unrelated work: the pack remains limited to the current repository and migration workflow and must explain materially different or expanded commands.

The workflow may stop after editing. If command execution is declined, the final report records the changes as applied but not runtime-validated.

## Output

The chat report includes:

- Overall readiness status
- Detected repository and Java configuration with file evidence
- Required changes
- Advisory recommendations
- Unresolved or high-risk items
- Modernization candidates
- Experimental items
- Suggested implementation groups by category and safety class
- Assumptions, warnings, and failure consequences
- Applied changes and rollback instructions
- Command permission scope
- JEP-level release-delta coverage and applicability
- Validation results
- Checks not performed
- Suggested next steps

Possible readiness statuses are `READY`, `CHANGES REQUIRED`, `BLOCKED`, `PARTIALLY VALIDATED`, and `UNSUPPORTED`.

Modernization progress is reported separately from readiness so optional feature adoption is not confused with Java 21 migration readiness.

## Install in the approved work environment

The onboarding script discovers this directory automatically when it is placed directly under `aroom-packs`.

Manual installation, when authorized in the work environment:

```powershell
aroom pack install <path-to-fm-pack-java-modernization>
aroom pack attach fm/java-modernization --project
aroom pack list
```

Run the project-scoped attach command from the root of the Java repository. A global attachment makes the pack instructions and skill active outside the repository being assessed and is not recommended for this pack.

If an earlier version was attached globally, detach it before attaching the updated pack to the project:

```powershell
aroom pack detach fm/java-modernization
aroom pack install <path-to-fm-pack-java-modernization>
cd <path-to-java-repository>
aroom pack attach fm/java-modernization --project
```

## Baseline and policy maintenance

The compatibility baseline is in `instructions/java-17-to-21-baseline.md`.

The modernization governance policy is in `instructions/java-17-to-21-modernization-policy.md`.

The pattern catalog is in `instructions/java-17-to-21-pattern-catalog.md`.

The complete JEP-level coverage inventory is in `instructions/java-17-to-21-release-delta.md`.

These files are intentionally versioned and sourced so normal skill runs do not require internet access. Review them periodically and before changing any exact compatibility recommendation, warning boundary, or implementation-approval rule.

## Validation evidence for the Jira story

In the work environment, capture:

1. Successful pack installation and project-scoped attachment.
2. A Maven Java 17 assessment.
3. A Gradle Java 17 assessment.
4. Required versus advisory findings in each report.
5. A modernization audit report with candidates separated from readiness.
6. An apply-preview report showing categorized diffs, warnings, failure consequences, rollback, and validation.
7. Approved implementation examples for each safety class, including dependency/framework or experimental work where representative fixtures permit it.
8. A post-change validation report with remaining risks after an approved apply run.
9. Approval-refusal scenarios proving that selection does not authorize edits and edit approval does not authorize command execution.
10. `Allow Once`, `Allow for Session`, and rejected-execution scenarios.
11. A release-delta report with a disposition for every JEP-level entry.
12. Java subject-matter-expert review of at least one report.

The `tests/fixtures` repositories provide small static-analysis examples. Run `python tests/validate_pack.py` only in an environment where Python execution is approved.
