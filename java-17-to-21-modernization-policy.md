# Java 17 to 21 Modernization Policy

**Policy version:** 0.3.0  
**Last reviewed:** 2026-07-22  
**Status:** Draft implementation policy pending organizational Java SME and platform approval

## Purpose

Define how the Java 17 to 21 skill proposes, warns about, implements, validates, and reports migration changes. Every safety category is implementable in `apply` mode after the user reviews the category-specific consequences and approves the exact selected diff.

Approval accepts disclosed risk; it does not make a risky change safe or prove production compatibility. Readiness remains governed by the baseline, and optional modernization remains separate from readiness.

## User-facing modes

- **assessment:** Static readiness discovery and reporting. No repository code execution or file modification.
- **validate:** Static assessment followed by approved build, test, application, and analysis commands. No file modification.
- **audit:** Static assessment plus complete release-delta and modernization reporting. No file modification.
- **apply:** Discover, propose categorized diffs, show consequences, let the user select some or all changes, approve the exact combined diff, implement it, request command permission, validate, and produce the final report.

`plan` is not a separate primary mode. Planning is an internal phase of `apply`. Use `apply --preview` when the user wants the proposal and diffs without file modification or command execution. Treat the legacy `plan` argument as an alias for `apply --preview`.

## Apply state machine

Use these states in order:

1. `DISCOVERED`: collect static evidence and release-delta coverage.
2. `PROPOSED`: generate categorized changes, exact candidate diffs, warnings, failure consequences, rollback, and validation requirements.
3. `SELECTED`: let the user select suggestion IDs, categories, `all proposed changes`, or `none`. Selection defines scope; it does not authorize edits.
4. `EDIT_APPROVED`: present the exact combined diff for the selected scope and obtain explicit approval.
5. `APPLIED`: write only the approved diff and produce a revertible change list.
6. `EXECUTION_PERMISSION`: present exact commands and invoke Anteroom's permission flow. The user may choose `Allow Once`, `Allow for Session`, or reject execution.
7. `VALIDATED`: execute only approved commands and record the results.
8. `REPORTED`: produce the final readiness, implementation, validation, consequence, and rollback report.

The workflow may stop at any state. If edits were applied but command execution was rejected, retain the approved edits, mark runtime validation `NOT RUN`, and report no status better than `PARTIALLY VALIDATED`.

## Required proposal fields

Every proposed change must include:

- stable suggestion identifier
- category and pattern
- classification and safety class
- file and line evidence
- evidence type and confidence
- exact proposed diff
- implementation scope and affected modules
- assumptions
- expected benefit
- consequences if the assumptions are wrong or validation fails
- rollback procedure
- required validation commands/checks
- preview/incubator flags and portability impact when applicable

## Safety classes and required warnings

All four classes are implementable after exact approval. The class controls warning strength and validation, not eligibility.

### Safe Mechanical

Narrow, local changes with low expected semantic risk.

Required warning:

- compilation or focused tests may still fail
- formatting, string, or configuration semantics may differ if the detected context was incomplete
- the exact diff should be easy to revert

These may be batch-selected and approved together.

### Review Required

Changes dependent on local intent or behavior, including ordering, null handling, serialization, control flow, configuration inheritance, or dependency compatibility.

Required warning:

- the change may compile while altering application behavior
- tests may need updates or may not cover the affected assumption
- dependency or plugin changes may introduce transitive changes, downloads, or new configuration requirements
- failure requires reverting the change or completing follow-up remediation

These may be selected individually, by category, or through `all proposed changes`, but their warning and consequences must appear in the final combined approval.

### Design Sensitive

Changes that may affect public APIs, concurrency, framework integration, lifecycle, security/transaction/request/logging context, runtime flags, native integration, or production behavior.

Required warning:

- downstream consumers or framework-managed behavior may break
- compilation and unit tests may pass while integration, performance, deadlock, data consistency, or context-propagation failures remain
- the change may require coordinated changes across modules or services
- rollback and representative integration/runtime validation are mandatory

These are implementable only after the exact blast radius, known unknowns, failure consequences, rollback, and required integration/runtime validation are displayed.

### Experimental

Preview, incubating, or trial-oriented language/API/runtime work.

Required warning:

- preview/incubator flags may be required at compile and runtime
- the API or language feature may change or disappear in later JDKs
- IDE, build, library, runtime, or vendor support may be incomplete
- production supportability and future upgrades may become harder
- failure may require removing the feature and restoring the previous implementation

These are implementable only after explicit opt-in. Selecting `all proposed changes` counts as opt-in only when the combined approval clearly states that experimental changes and their flags are included.

## Selection and edit approval

Present all proposed changes grouped by classification and safety class. Let the user choose:

- one or more suggestion IDs
- one or more categories
- `all proposed changes`
- `none`

After selection, present one exact combined diff containing only the selected changes. Repeat category warnings and aggregate consequences, affected modules, validation, and rollback. Ask for explicit approval of that exact diff before writing files.

Approval of a category does not authorize unlisted future findings. Approval of a diff authorizes only that displayed edit set.

## Command execution permission

After edits, show every proposed command with:

- exact command and working directory
- purpose
- expected file/process/network side effects
- dependency-download possibility
- affected suggestion IDs
- stop conditions

Then invoke Anteroom's execution permission flow:

- **Allow Once:** authorize the displayed execution request once.
- **Allow for Session:** allow execution for the current Anteroom session. Even when the tool permission persists, this pack must remain inside the current repository and migration workflow and must explain materially different or expanded commands before running them.
- **Reject:** do not execute commands; report validation as not run or blocked.

Do not request `Allow Always` as part of the normal migration workflow. A persistent configuration change is outside this pack unless the user explicitly requests it.

## Execution and failure behavior

- Apply changes in reversible batches where dependencies between edits allow it.
- Record a before/after change list and rollback instructions for every batch.
- Stop the affected batch when an approved command fails.
- Classify the failure as migration-related, implementation-related, pre-existing, environmental, permission-related, or unresolved.
- Do not silently broaden the diff or command set to repair a failure.
- Propose the repair as a new categorized change with its own consequences and approval.
- Never commit, push, deploy, or update external systems unless the user separately and explicitly requests that additional workflow.
- Never modify files outside the current repository.
- Redact secrets and sensitive values from proposals and reports.

## Reporting requirements

The final report must separate:

- readiness findings
- modernization findings
- selected and rejected changes
- warnings and accepted consequences by safety class
- exact changes applied
- commands and permission scope used
- validation results and failures
- rollback instructions
- remaining findings and checks not performed

A repository may be `READY` for Java 21 while optional modernization remains, and may be `PARTIALLY VALIDATED` after approved changes when command execution or required runtime validation was not completed.
