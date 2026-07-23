# Java 17 to 21 Release Delta

**Delta version:** 0.3.0  
**Last reviewed:** 2026-07-22  
**Status:** Complete JEP-level inventory pending organizational Java SME approval

## Purpose

Use this inventory to ensure that a Java 17 to 21 assessment considers every JEP integrated since JDK 17, not only common source-modernization examples. The inventory follows the OpenJDK consolidated JDK 17-to-21 list, which omits superseded intermediate previews and retains the final Java 21 state.

This is a **JEP-level completeness boundary**. It does not claim to enumerate every added method, resolved bug, provider-specific change, or security patch. Use the Oracle migration guide, release notes, API removals/deprecations, `jdeprscan`, and application-specific testing for those surfaces.

## Required coverage behavior

For every entry below, report one coverage disposition:

- `APPLICABLE_FINDING`: repository evidence makes the item relevant and a finding was created.
- `APPLICABLE_NO_FINDING`: the item was checked and no risk or opportunity was detected.
- `NOT_APPLICABLE`: the repository does not use the affected platform, API, tool, or behavior.
- `UNRESOLVED`: available evidence cannot establish applicability.
- `NOT_CHECKED`: the check was outside the approved or available validation surface; explain why.

Preview and incubator entries are never Java 21 readiness requirements merely because they exist. Detect existing use and optional trial opportunities, classify them `EXPERIMENTAL`, and allow implementation only after explicit opt-in, failure consequences, exact-diff approval, and preview/incubator-aware validation.

## Final additions since JDK 17

| JEP | Release area | Change | Default assessment treatment |
| --- | --- | --- | --- |
| 400 | JDK 18 / I/O | UTF-8 by Default | Check implicit default-charset dependencies and runtime parity. |
| 408 | JDK 18 / Tooling | Simple Web Server | Usually not applicable; identify only repository scripts or documentation that could intentionally use `jwebserver`. Never recommend it for production serving. |
| 413 | JDK 18 / Javadoc | Code Snippets in Java API Documentation | Detect legacy embedded Javadoc examples as optional documentation modernization. |
| 416 | JDK 18 / Reflection | Reimplement Core Reflection with Method Handles | Review reflection-heavy frameworks, serialization, agents, and performance/behavior assumptions. |
| 418 | JDK 18 / Networking | Internet-Address Resolution SPI | Review custom DNS/host-resolution code or resolver-provider requirements. |
| 422 | JDK 19 / HotSpot | Linux/RISC-V Port | Check only when delivery targets Linux RISC-V or native artifacts constrain architecture. |
| 431 | JDK 21 / Collections | Sequenced Collections | Detect ordered-collection helpers and API-modernization candidates. |
| 439 | JDK 21 / GC | Generational ZGC | Review ZGC flags, performance requirements, container memory limits, and runtime validation needs. |
| 440 | JDK 21 / Language | Record Patterns | Detect record deconstruction opportunities separately from record-class conversion. |
| 441 | JDK 21 / Language | Pattern Matching for `switch` | Detect type-dispatch and pattern-switch opportunities separately from older switch expressions. |
| 444 | JDK 21 / Concurrency | Virtual Threads | Audit executor ownership, blocking work, throttling, pinning, and context propagation. |
| 451 | JDK 21 / Serviceability | Prepare to Disallow Dynamic Agent Loading | Detect agent attachment and validate warnings and startup configuration. |
| 452 | JDK 21 / Cryptography | Key Encapsulation Mechanism API | Report only when cryptographic key encapsulation or provider integration is relevant; require security review. |

## Preview and incubator state in JDK 21

| JEP | Feature | Required treatment |
| --- | --- | --- |
| 430 | String Templates (Preview) | Detect `--enable-preview` and source use; implement only after `EXPERIMENTAL` opt-in and preview/future-portability warning. |
| 442 | Foreign Function & Memory API (Third Preview) | Detect preview API/native-memory use; implement only after `EXPERIMENTAL` opt-in and native-lifecycle/platform warning. |
| 443 | Unnamed Patterns and Variables (Preview) | Detect existing preview use or optional trials; implement only after `EXPERIMENTAL` opt-in and portability warning. |
| 445 | Unnamed Classes and Instance Main Methods (Preview) | Usually not applicable to production repositories; implement only after `EXPERIMENTAL` opt-in and supportability warning. |
| 446 | Scoped Values (Preview) | Relate to `ThreadLocal`/context-propagation audits; implement only after `EXPERIMENTAL` opt-in and context/framework warning. |
| 448 | Vector API (Sixth Incubator) | Detect incubator module/API use and performance-sensitive opportunities; implement only after `EXPERIMENTAL` opt-in and benchmark/architecture warning. |
| 453 | Structured Concurrency (Preview) | Detect fan-out/fan-in trials; implement only after `EXPERIMENTAL` opt-in and cancellation/error/future-portability warning. |

## Deprecations

| JEP | Change | Default assessment treatment |
| --- | --- | --- |
| 421 | Deprecate Finalization for Removal | Detect custom finalizers and finalization dependencies; require lifecycle review. |
| 449 | Deprecate the Windows 32-bit x86 Port for Removal | Check delivery architecture only when Windows x86-32 is relevant. |

## Earlier language features that remain useful modernization targets

The pattern catalog may also suggest features finalized before or in JDK 17 because Java 17 code can still contain older idioms. Label them `EARLIER_LANGUAGE_MODERNIZATION`, not as new JDK 18-21 additions:

- switch expressions: final in Java 14
- text blocks: final in Java 15
- pattern matching for `instanceof`: final in Java 16
- record classes: final in Java 16
- sealed classes and interfaces: final in Java 17

## Authoritative sources

- OpenJDK JEPs integrated since JDK 17: https://openjdk.org/projects/jdk/21/jeps-since-jdk-17
- OpenJDK JDK 21 feature list: https://openjdk.org/projects/jdk/21/
- Oracle JDK 21 Migration Guide, Significant Changes: https://docs.oracle.com/en/java/javase/21/migrate/significant-changes-jdk-release.html
- Oracle JDK 21 Migration Guide, Removed Tools and Components: https://docs.oracle.com/en/java/javase/21/migrate/removed-tools-and-components.html
- Oracle JDK 21 Migration Guide, Removed APIs: https://docs.oracle.com/en/java/javase/21/migrate/removed-apis.html
