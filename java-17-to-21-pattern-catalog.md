# Java 17 to 21 Pattern Catalog

**Catalog version:** 0.3.0  
**Last reviewed:** 2026-07-22  
**Status:** Initial modernization pattern catalog pending organizational Java SME approval

## Purpose

This catalog defines actionable pattern families, detection heuristics, evidence requirements, default classifications, safety classes, implementation approval, pattern-specific consequences, and post-change validation expectations for the `/java-17-to-21` skill. Use it together with `java-17-to-21-release-delta`; the release delta defines completeness, while this catalog defines actionable findings.

## Pattern catalog template

Each pattern entry should be interpreted with these fields:

- **Pattern family:** Configuration, dependency compatibility, runtime risk, language modernization, API modernization, concurrency modernization, delivery/runtime alignment, or design-sensitive.
- **Pattern name:** Concise identifier such as `virtual-thread-candidate`, `threadlocal-risk`, or `switch-pattern-candidate`.
- **Intent:** Why the pattern exists and what migration or modernization question it helps answer.
- **Detection heuristics:** Source constructs, configuration keys, dependency names, command evidence, or file patterns to match.
- **Evidence to capture:** File path, line number, matched text, surrounding context, and whether the evidence is static or validated.
- **Why it matters:** Migration risk, modernization value, or runtime concern associated with the pattern.
- **Default classification:** REQUIRED, ADVISORY, UNRESOLVED, MODERNIZATION_CANDIDATE, or EXPERIMENTAL.
- **Safety class:** Safe Mechanical, Review Required, Design Sensitive, or Experimental.
- **Implementation approval:** Every pattern is implementable after the warning and approval required by its safety class. State any additional pattern-specific opt-in or command permission.
- **Validation required after change:** Compile, unit tests, integration tests, startup checks, runtime checks, warning review, or performance/concurrency validation.
- **Non-goals / cautions:** What should not be inferred from a match. Use these cautions plus the safety-class policy to state concrete failure consequences and rollback requirements before approval.

When a pattern relates to a JDK change, also capture its release/JEP provenance and distinguish final, preview, incubator, deprecated, and earlier-language-modernization features.

## Configuration patterns

### java-release-still-17

- **Pattern family:** Configuration
- **Intent:** Detect repositories still configured to emit Java 17 bytecode when the migration target is Java 21.
- **Detection heuristics:** `maven.compiler.release`, compiler `<release>`, `<source>`, `<target>`, `java.version`, Gradle `JavaLanguageVersion.of(17)`, `sourceCompatibility = 17`, `targetCompatibility = 17`, `options.release = 17`.
- **Evidence to capture:** Build file path, line number, property/plugin block, inherited source if known.
- **Why it matters:** The project is not configured for Java 21 output.
- **Default classification:** REQUIRED
- **Safety class:** Safe Mechanical for local release-property edits; Review Required when inherited or convention-managed.
- **Auto-apply policy:** implementable after exact approval; inherited or convention-managed values require Review Required warnings and coordinated validation.
- **Validation required after change:** Clean compile/build and test on JDK 21.
- **Non-goals / cautions:** Do not change inherited parent, convention plugin, or enterprise baseline values without explicit approval.

### wrapper-runtime-mismatch

- **Pattern family:** Configuration
- **Intent:** Detect Maven/Gradle wrapper or runtime incompatibility risks.
- **Detection heuristics:** `gradle-wrapper.properties`, Maven wrapper files, CI image/tool versions, plugin versions.
- **Evidence to capture:** Wrapper version, distribution URL, CI/runtime Java version, file path and line.
- **Why it matters:** Builds can fail before source compatibility is tested.
- **Default classification:** REQUIRED when below known support level; UNRESOLVED when support cannot be established.
- **Safety class:** Review Required
- **Auto-apply policy:** implementable after Review Required warning and exact approval; disclose distribution downloads, checksum/provenance, plugin compatibility, and rollback consequences.
- **Validation required after change:** Wrapper version command and clean build on JDK 21.
- **Non-goals / cautions:** Do not recommend uncontrolled jumps to the newest major version.

## Dependency and runtime risk patterns

### dependency-compatibility-unresolved

- **Pattern family:** Dependency compatibility
- **Intent:** Track frameworks, plugins, annotation processors, bytecode libraries, agents, drivers, and native integrations whose Java 21 support is not proven locally.
- **Detection heuristics:** Direct versions, managed versions, BOMs, parent POMs, Gradle platforms, version catalogs, build plugins.
- **Evidence to capture:** Dependency or plugin coordinate, version source, management source, file path and line.
- **Why it matters:** Dependencies often cause migration failures before source code does.
- **Default classification:** UNRESOLVED
- **Safety class:** Review Required
- **Auto-apply policy:** implementable after Review Required warning and exact approval when an authoritative target version and concrete diff are available; disclose transitive changes and downloads.
- **Validation required after change:** Compile, tests, startup, and library-specific compatibility checks.
- **Non-goals / cautions:** Do not label a dependency incompatible solely because it is old.

### reflective-access-risk

- **Pattern family:** Runtime risk
- **Intent:** Detect strong encapsulation and internal API risk.
- **Detection heuristics:** `sun.*`, most `com.sun.*`, `jdk.internal.*`, `Unsafe`, `--add-opens`, `--add-exports`, `--illegal-access`.
- **Evidence to capture:** Matched package/API/flag, owning dependency or code path if available.
- **Why it matters:** Runtime-only failures can appear on newer JDKs.
- **Default classification:** UNRESOLVED or REQUIRED when validation proves failure.
- **Safety class:** Review Required
- **Auto-apply policy:** implementable after Review Required warning and exact approval; disclose runtime-access failure risk and retain rollback for flags or source changes.
- **Validation required after change:** JDK 21 tests, startup, `jdeps --jdk-internals`, and warning review.
- **Non-goals / cautions:** `--add-opens` alone does not prove the flag can be removed.

## Language modernization patterns

### instanceof-pattern-cleanup

- **Pattern family:** Language modernization
- **Intent:** Detect local `instanceof` plus cast cleanup opportunities.
- **Detection heuristics:** `if (x instanceof Type) { Type y = (Type) x; ... }`
- **Evidence to capture:** Source file, line range, variable names, local usage.
- **Why it matters:** Pattern variables reduce boilerplate without changing public contracts.
- **Release provenance:** Earlier language modernization; pattern matching for `instanceof` was final in Java 16.
- **Default classification:** MODERNIZATION_CANDIDATE
- **Safety class:** Safe Mechanical
- **Auto-apply policy:** implementable after Safe Mechanical warning and exact approval.
- **Validation required after change:** Compile and relevant unit tests.
- **Non-goals / cautions:** Do not alter control flow or variable scope assumptions.

### switch-pattern-candidate

- **Pattern family:** Language modernization
- **Intent:** Detect switch modernization opportunities.
- **Detection heuristics:** Type-dispatch chains that could use type or record patterns, guarded type cases, and null-handling branches. Report ordinary switch-expression cleanup separately as earlier language modernization.
- **Evidence to capture:** Source file, branch structure, involved types.
- **Why it matters:** Java 21 finalizes pattern matching for switch and can simplify type dispatch.
- **Release provenance:** JEP 441, final in Java 21.
- **Default classification:** MODERNIZATION_CANDIDATE
- **Safety class:** Review Required
- **Auto-apply policy:** implementable after Review Required warning and exact approval; disclose exhaustiveness, null, branch-order, and control-flow consequences.
- **Validation required after change:** Compile, unit tests, branch coverage review.
- **Non-goals / cautions:** Do not rewrite complex control flow without Review Required warnings, exact approval, branch-behavior consequences, rollback, and tests.

### record-candidate

- **Pattern family:** Language modernization
- **Intent:** Detect immutable data carriers that may be record candidates.
- **Detection heuristics:** Final fields, canonical constructor, accessors, `equals`, `hashCode`, `toString`.
- **Evidence to capture:** Source file, class signature, constructors, public API usage.
- **Why it matters:** Records can reduce boilerplate but change API shape and serialization behavior.
- **Release provenance:** Earlier language modernization; record classes were final in Java 16. This is distinct from Java 21 record patterns.
- **Default classification:** MODERNIZATION_CANDIDATE
- **Safety class:** Design Sensitive
- **Auto-apply policy:** implementable after Design Sensitive warning and exact approval; disclose public API, serialization, reflection, framework binding, constructor, and accessor consequences.
- **Validation required after change:** Compile, serialization/API compatibility tests, framework binding tests.
- **Non-goals / cautions:** Do not convert public API or framework-bound classes without Design Sensitive warning, downstream-impact disclosure, exact approval, and integration validation.

### sealed-hierarchy-candidate

- **Pattern family:** Language modernization
- **Intent:** Identify type hierarchies that may benefit from sealed classes or interfaces.
- **Detection heuristics:** Closed implementation sets, package-private constructors, discriminator logic.
- **Evidence to capture:** Hierarchy root, implementors, external extension risk.
- **Why it matters:** Sealed hierarchies can document and enforce domain boundaries.
- **Release provenance:** Earlier language modernization; sealed classes and interfaces were final in Java 17.
- **Default classification:** MODERNIZATION_CANDIDATE
- **Safety class:** Design Sensitive
- **Auto-apply policy:** implementable after Design Sensitive warning and exact approval; disclose extension, proxying, reflection, and downstream compilation consequences.
- **Validation required after change:** Compile, API compatibility, framework proxying/binding tests.
- **Non-goals / cautions:** Do not restrict extension points without Design Sensitive warning, downstream-impact disclosure, exact approval, and compatibility validation.

### text-block-candidate

- **Pattern family:** Language modernization
- **Intent:** Detect multiline string literals that could become text blocks.
- **Detection heuristics:** String concatenation with repeated newlines, SQL/JSON/XML literals.
- **Evidence to capture:** Source file, literal, escaping behavior.
- **Why it matters:** Text blocks improve readability.
- **Release provenance:** Earlier language modernization; text blocks were final in Java 15.
- **Default classification:** MODERNIZATION_CANDIDATE
- **Safety class:** Safe Mechanical when exact string semantics are preserved.
- **Auto-apply policy:** implementable after Safe Mechanical warning and exact approval with before/after string-semantics comparison.
- **Validation required after change:** Compile and string-sensitive tests.
- **Non-goals / cautions:** Do not alter whitespace, escaping, or trailing-newline semantics.

### record-pattern-candidate

- **Pattern family:** Language modernization
- **Intent:** Detect Java 21 record deconstruction opportunities independently of record-class conversion.
- **Detection heuristics:** Type tests followed by record component accessor calls, nested record traversal, or switch type cases over record classes.
- **Evidence to capture:** Record declaration, source file and line range, accessor chain, null assumptions, and exhaustiveness requirements.
- **Why it matters:** Record patterns can make data navigation declarative without converting existing classes into records.
- **Release provenance:** JEP 440, final in Java 21.
- **Default classification:** MODERNIZATION_CANDIDATE
- **Safety class:** Review Required
- **Auto-apply policy:** implementable after Review Required warning and exact approval; disclose null, nesting, exhaustiveness, and generic inference consequences.
- **Validation required after change:** Compile, unit tests, null-path tests, and switch exhaustiveness review.
- **Non-goals / cautions:** Do not confuse record patterns with record declarations or change a class into a record.

### preview-language-feature-use

- **Pattern family:** Language modernization
- **Intent:** Detect use or proposed adoption of Java 21 preview language features.
- **Detection heuristics:** `--enable-preview`, string-template syntax, unnamed patterns/variables, or unnamed classes and instance main methods.
- **Evidence to capture:** Build/runtime preview flags, source construct, owning module, and deployment compatibility.
- **Why it matters:** JDK 21 includes String Templates (JEP 430), Unnamed Patterns and Variables (JEP 443), and Unnamed Classes and Instance Main Methods (JEP 445) as previews, not permanent baseline features.
- **Release provenance:** JEPs 430, 443, and 445, preview in Java 21.
- **Default classification:** EXPERIMENTAL
- **Safety class:** Experimental
- **Auto-apply policy:** implementable after Experimental explicit opt-in and exact approval; include compile/runtime preview flags and future-JDK removal or change consequences.
- **Validation required after change:** Explicit opt-in, preview-enabled compile/test/runtime, and future-JDK portability review.
- **Non-goals / cautions:** Never introduce or recommend a preview feature as a Java 21 readiness requirement.

## API modernization patterns

### sequenced-collection-candidate

- **Pattern family:** API modernization
- **Intent:** Detect opportunities to use Java 21 sequenced collection APIs.
- **Detection heuristics:** Manual first/last/reverse helpers, ordered collection assumptions.
- **Evidence to capture:** Source file, collection type, ordering contract.
- **Why it matters:** Java 21 adds explicit ordered collection abstractions.
- **Default classification:** MODERNIZATION_CANDIDATE
- **Safety class:** Review Required
- **Auto-apply policy:** implementable after Review Required warning and exact approval; disclose encounter-order, mutability, view, and compatibility consequences.
- **Validation required after change:** Compile and behavioral tests for ordering.
- **Non-goals / cautions:** Do not assume every `List` or `LinkedHashMap` usage should change.

### stream-optional-cleanup

- **Pattern family:** API modernization
- **Intent:** Detect verbose stream or Optional idioms that could be simplified.
- **Detection heuristics:** Manual null checks around Optional, redundant stream collections, verbose first/last helpers.
- **Evidence to capture:** Source file, expression, expected behavior.
- **Why it matters:** Local cleanup can improve clarity after the migration target is established.
- **Default classification:** MODERNIZATION_CANDIDATE
- **Safety class:** Review Required
- **Auto-apply policy:** implementable after Review Required warning and exact approval; disclose stream laziness, ordering, parallelism, null behavior, and side-effect consequences.
- **Validation required after change:** Compile and affected unit tests.
- **Non-goals / cautions:** Where laziness, ordering, parallelism, or side effects matter, disclose the behavior change and tests before exact approval.

### javadoc-snippet-candidate

- **Pattern family:** API modernization
- **Intent:** Identify Javadoc examples that may benefit from the JDK 18 `@snippet` tag.
- **Detection heuristics:** Large `<pre>{@code ...}</pre>` blocks, copied example code, or external snippet files referenced manually.
- **Evidence to capture:** Documentation file and line range, source of the example, and current doclint behavior.
- **Why it matters:** `@snippet` can improve example maintainability and validation.
- **Release provenance:** JEP 413, final in JDK 18 tooling.
- **Default classification:** MODERNIZATION_CANDIDATE
- **Safety class:** Review Required
- **Auto-apply policy:** implementable after Review Required warning and exact approval; disclose doclint and generated-documentation consequences.
- **Validation required after change:** Javadoc generation and doclint.
- **Non-goals / cautions:** Do not change executable behavior or claim examples are tested unless a test verifies them.

### kem-api-candidate

- **Pattern family:** API modernization
- **Intent:** Identify relevant custom key-encapsulation code that may use the Java 21 KEM API.
- **Detection heuristics:** Cryptographic key encapsulation, provider-specific KEM abstractions, or protocol implementations using asymmetric wrapping for symmetric keys.
- **Evidence to capture:** Algorithm/provider, protocol boundary, compliance requirements, and existing tests.
- **Why it matters:** Java 21 provides a standard KEM API, but cryptographic changes require specialized review.
- **Release provenance:** JEP 452, final in Java 21.
- **Default classification:** MODERNIZATION_CANDIDATE
- **Safety class:** Design Sensitive
- **Auto-apply policy:** implementable after Design Sensitive warning, security-specific consequences, and exact approval.
- **Validation required after change:** Security review, provider interoperability, known-answer tests, and protocol integration tests.
- **Non-goals / cautions:** Do not redesign cryptography or select algorithms without Design Sensitive warning, security-review consequences, exact approval, interoperability tests, and rollback.

## Concurrency modernization patterns

### virtual-thread-candidate

- **Pattern family:** Concurrency modernization
- **Intent:** Identify areas that may benefit from virtual threads.
- **Detection heuristics:** `new Thread`, `Executors.newFixedThreadPool`, blocking I/O tasks, request-per-thread patterns.
- **Evidence to capture:** Executor ownership, task type, blocking behavior, framework involvement.
- **Why it matters:** Java 21 introduces major concurrency opportunities.
- **Default classification:** MODERNIZATION_CANDIDATE
- **Safety class:** Design Sensitive
- **Auto-apply policy:** implementable after Design Sensitive warning and exact approval; disclose throttling, ownership, pinning, context, integration, and performance consequences.
- **Validation required after change:** Integration/runtime tests, performance/concurrency validation, startup checks.
- **Non-goals / cautions:** Do not rewrite framework-managed executors or throttling designs without Design Sensitive warning, exact approval, and representative runtime validation.

### threadlocal-risk

- **Pattern family:** Concurrency modernization
- **Intent:** Detect `ThreadLocal` and `InheritableThreadLocal` usage that may interact badly with concurrency model changes.
- **Detection heuristics:** `ThreadLocal`, `InheritableThreadLocal`, request/security/logging context holders.
- **Evidence to capture:** Source file, owning context, lifecycle, cleanup behavior.
- **Why it matters:** Context propagation assumptions can break with executor or virtual-thread changes.
- **Default classification:** UNRESOLVED or MODERNIZATION_CANDIDATE
- **Safety class:** Design Sensitive
- **Auto-apply policy:** implementable after Design Sensitive warning and exact approval; disclose context loss/leakage, lifecycle, security, tracing, logging, and transaction consequences.
- **Validation required after change:** Integration tests for request, security, tracing, logging, and transaction context.
- **Non-goals / cautions:** Do not modify context propagation without Design Sensitive warning, exact approval, and context-specific integration validation.

### pinning-risk

- **Pattern family:** Concurrency modernization
- **Intent:** Identify pinning-sensitive synchronized or monitor-heavy regions around blocking work.
- **Detection heuristics:** `synchronized` blocks/methods, blocking I/O, waits, locks around external calls.
- **Evidence to capture:** Source file, lock region, blocking call evidence.
- **Why it matters:** Pinning can reduce benefits of virtual threads.
- **Default classification:** MODERNIZATION_CANDIDATE
- **Safety class:** Design Sensitive
- **Auto-apply policy:** implementable after Design Sensitive warning and exact approval; disclose deadlock, race, ordering, throughput, and correctness consequences.
- **Validation required after change:** Concurrency tests and runtime profiling.
- **Non-goals / cautions:** Do not change locking semantics without Design Sensitive warning, exact approval, rollback, concurrency tests, and profiling.

### structured-concurrency-trial

- **Pattern family:** Concurrency modernization
- **Intent:** Identify fan-out/fan-in logic that could be trialed with structured concurrency.
- **Detection heuristics:** sibling `CompletableFuture` orchestration, manual thread joins, coordinated subtasks.
- **Evidence to capture:** Source file, cancellation/error handling, task ownership.
- **Why it matters:** Structured concurrency can simplify lifecycle management but remains trial-oriented.
- **Default classification:** EXPERIMENTAL
- **Safety class:** Experimental
- **Auto-apply policy:** implementable after Experimental explicit opt-in and exact approval; include preview flags plus cancellation, error propagation, observability, and future-JDK consequences.
- **Validation required after change:** Explicit opt-in prototype, integration tests, cancellation/error-path validation.
- **Non-goals / cautions:** Never present structured concurrency as a baseline migration requirement.

### scoped-value-trial

- **Pattern family:** Concurrency modernization
- **Intent:** Relate `ThreadLocal` context usage to optional scoped-value experiments.
- **Detection heuristics:** Read-mostly `ThreadLocal` values with bounded call lifetimes, especially around virtual-thread candidates.
- **Evidence to capture:** Context ownership, mutation, inheritance, cleanup, framework integration, and call boundaries.
- **Why it matters:** Scoped Values were previewed in Java 21 as an alternative for some immutable context-sharing cases.
- **Release provenance:** JEP 446, preview in Java 21.
- **Default classification:** EXPERIMENTAL
- **Safety class:** Experimental
- **Auto-apply policy:** implementable after Experimental explicit opt-in and exact approval; include preview flags plus context semantics, framework support, and future-JDK consequences.
- **Validation required after change:** Explicit prototype opt-in and context, security, tracing, transaction, and concurrency tests.
- **Non-goals / cautions:** Do not replace `ThreadLocal` without Experimental opt-in and context-specific validation; do not assume scoped values fit mutable context.

### foreign-function-memory-use

- **Pattern family:** Concurrency modernization
- **Intent:** Detect existing or proposed use of the Java 21 preview Foreign Function & Memory API.
- **Detection heuristics:** `java.lang.foreign`, `Arena`, `MemorySegment`, linker calls, native bindings, or preview flags.
- **Evidence to capture:** Native library, lifecycle/ownership, platform targets, preview flags, and JNI/JNA alternatives.
- **Why it matters:** The API was still preview in Java 21 and is platform- and lifecycle-sensitive.
- **Release provenance:** JEP 442, third preview in Java 21.
- **Default classification:** EXPERIMENTAL
- **Safety class:** Experimental
- **Auto-apply policy:** implementable after Experimental explicit opt-in and exact approval; include preview flags plus native memory, lifecycle, ABI, platform, and future-JDK consequences.
- **Validation required after change:** Native integration tests, memory-lifetime tests, platform matrix, and explicit preview opt-in.
- **Non-goals / cautions:** Do not replace JNI/JNA without Experimental opt-in, exact approval, native-lifecycle analysis, and platform validation.

### vector-api-use

- **Pattern family:** Concurrency modernization
- **Intent:** Detect use or performance-sensitive opportunities for the incubating Vector API.
- **Detection heuristics:** `jdk.incubator.vector`, explicit incubator modules, or measured scalar numeric hotspots.
- **Evidence to capture:** Module flags, supported architectures, benchmark evidence, and fallback behavior.
- **Why it matters:** The Vector API remained incubating in Java 21 and should be driven by measurement.
- **Release provenance:** JEP 448, sixth incubator in Java 21.
- **Default classification:** EXPERIMENTAL
- **Safety class:** Experimental
- **Auto-apply policy:** implementable after Experimental explicit opt-in and exact approval; include incubator modules plus numerical, architecture, benchmark, and future-JDK consequences.
- **Validation required after change:** JMH or equivalent benchmarks, numerical correctness, architecture coverage, and fallback tests.
- **Non-goals / cautions:** Do not propose vectorization without a measured hotspot.

## Platform and runtime delta patterns

### reflection-behavior-risk

- **Pattern family:** Runtime risk
- **Intent:** Identify reflection-heavy behavior that may be affected by the JDK 18 core-reflection reimplementation.
- **Detection heuristics:** Intensive `java.lang.reflect`/method-handle use, generated accessors, serialization frameworks, agents, and reflection performance assumptions.
- **Evidence to capture:** API/framework, code path, encapsulation flags, and Java 21 runtime evidence.
- **Why it matters:** JEP 416 changed the reflection implementation while preserving API contracts; edge behavior and performance still require validation.
- **Release provenance:** JEP 416, final in JDK 18.
- **Default classification:** UNRESOLVED
- **Safety class:** Review Required
- **Auto-apply policy:** implementable after Review Required warning and exact approval; disclose reflection access, serialization, framework, startup, and performance consequences.
- **Validation required after change:** Framework tests, startup, serialization, reflection access, and performance checks where relevant.
- **Non-goals / cautions:** Do not infer incompatibility from reflection use alone.

### resolver-spi-candidate

- **Pattern family:** Runtime risk
- **Intent:** Identify custom hostname/address-resolution integrations affected by or suitable for the InetAddress resolver SPI.
- **Detection heuristics:** Custom DNS libraries, hosts-file overrides, `InetAddress` wrappers, or provider configuration.
- **Evidence to capture:** Resolver ownership, caching, security policy, deployment configuration, and failure behavior.
- **Why it matters:** JDK 18 added a standard resolver SPI, but network semantics are environment-sensitive.
- **Release provenance:** JEP 418, final in JDK 18.
- **Default classification:** MODERNIZATION_CANDIDATE or UNRESOLVED
- **Safety class:** Design Sensitive
- **Auto-apply policy:** implementable after Design Sensitive warning and exact approval; disclose DNS, caching, failover, security, and deployment consequences.
- **Validation required after change:** DNS, caching, failover, security, and deployment integration tests.
- **Non-goals / cautions:** Do not replace production name resolution without Design Sensitive warning, exact approval, rollback, and environment-specific validation.

### generational-zgc-candidate

- **Pattern family:** Runtime risk
- **Intent:** Detect ZGC use or workloads that may warrant a Generational ZGC evaluation.
- **Detection heuristics:** `-XX:+UseZGC`, latency-sensitive workloads, allocation-heavy services, and GC tuning files.
- **Evidence to capture:** JVM flags, heap/container limits, latency objectives, GC logs, and baseline performance.
- **Why it matters:** Java 21 added Generational ZGC, but collector selection and tuning require measurement.
- **Release provenance:** JEP 439, final in JDK 21.
- **Default classification:** MODERNIZATION_CANDIDATE
- **Safety class:** Design Sensitive
- **Auto-apply policy:** implementable after Design Sensitive warning and exact approval; disclose latency, throughput, memory, container, operational, and rollback consequences.
- **Validation required after change:** Representative load tests, GC-log analysis, latency/throughput comparison, and rollback plan.
- **Non-goals / cautions:** Do not change garbage collectors or production JVM flags without Design Sensitive warning, exact approval, performance evidence, and rollback.

### platform-architecture-risk

- **Pattern family:** Delivery/runtime alignment
- **Intent:** Cover the new Linux/RISC-V port and deprecation of the Windows 32-bit x86 port.
- **Detection heuristics:** Architecture-specific images, native libraries, JNI/JNA artifacts, Windows x86 runners, or Linux RISC-V deployment targets.
- **Evidence to capture:** Target OS/architecture matrix, container base images, native artifacts, CI runners, and vendor support.
- **Why it matters:** JDK 19 added Linux/RISC-V support and JDK 21 deprecated Windows x86-32 for removal.
- **Release provenance:** JEP 422 and JEP 449.
- **Default classification:** UNRESOLVED or REQUIRED when an approved target is unsupported.
- **Safety class:** Design Sensitive
- **Auto-apply policy:** implementable after Design Sensitive warning and exact approval; disclose native artifact, target availability, packaging, CI, and deployment consequences.
- **Validation required after change:** Target-platform build, packaging, startup, and native integration tests.
- **Non-goals / cautions:** Do not infer target architectures from the developer workstation alone.

### simple-web-server-use

- **Pattern family:** Delivery/runtime alignment
- **Intent:** Detect intentional use or inappropriate production use of the JDK simple web server.
- **Detection heuristics:** `jwebserver`, `com.sun.net.httpserver.SimpleFileServer`, or repository scripts/documentation invoking them.
- **Evidence to capture:** Script or source location, environment, network binding, and intended purpose.
- **Why it matters:** JEP 408 provides a testing/prototyping tool, not a production server recommendation.
- **Release provenance:** JEP 408, final in JDK 18 tooling/API.
- **Default classification:** ADVISORY or UNRESOLVED
- **Safety class:** Review Required
- **Auto-apply policy:** implementable after Review Required warning and exact approval; disclose network exposure and non-production suitability consequences.
- **Validation required after change:** Script/documentation review and environment-specific smoke test.
- **Non-goals / cautions:** Never recommend the simple web server for production serving.

## Design-sensitive patterns

### public-api-change-risk

- **Pattern family:** Design-sensitive
- **Intent:** Flag candidates that would alter external contracts.
- **Detection heuristics:** Public classes/methods, exported packages, DTOs, framework-bound APIs.
- **Evidence to capture:** Public signature, consumers if known, serialization or reflection use.
- **Why it matters:** API changes can break callers even when source compiles.
- **Default classification:** EXPERIMENTAL or MODERNIZATION_CANDIDATE
- **Safety class:** Design Sensitive
- **Auto-apply policy:** implementable after Design Sensitive warning and exact approval; disclose binary/source compatibility, serialization, reflection, and downstream consumer consequences.
- **Validation required after change:** API compatibility and downstream integration testing.
- **Non-goals / cautions:** Do not change public API signatures without Design Sensitive warning, downstream-consumer impact, exact approval, compatibility checks, and rollback.

### context-propagation-risk

- **Pattern family:** Design-sensitive
- **Intent:** Detect transaction, security, request, tracing, or logging context propagation assumptions.
- **Detection heuristics:** ThreadLocal context holders, framework context APIs, MDC, security context, transactions.
- **Evidence to capture:** Source file, context type, lifecycle and cleanup.
- **Why it matters:** Context propagation can change under executor or concurrency modernization.
- **Default classification:** UNRESOLVED
- **Safety class:** Design Sensitive
- **Auto-apply policy:** implementable after Design Sensitive warning and exact approval; disclose transaction, security, request, tracing, logging, lifecycle, and data-consistency consequences.
- **Validation required after change:** Integration and runtime context validation.
- **Non-goals / cautions:** Do not modify context propagation without Design Sensitive warning, exact approval, and transaction/security/request/tracing/logging validation.

### framework-executor-risk

- **Pattern family:** Design-sensitive
- **Intent:** Detect framework-managed execution boundaries.
- **Detection heuristics:** Spring executors, scheduled tasks, async annotations, reactive schedulers, container-managed pools.
- **Evidence to capture:** Framework configuration, executor ownership, lifecycle.
- **Why it matters:** Framework-managed concurrency changes can alter runtime behavior.
- **Default classification:** UNRESOLVED or MODERNIZATION_CANDIDATE
- **Safety class:** Design Sensitive
- **Auto-apply policy:** implementable after Design Sensitive warning and exact approval; disclose framework lifecycle, scheduler, backpressure, reactive-boundary, and runtime consequences.
- **Validation required after change:** Framework integration and startup/runtime tests.
- **Non-goals / cautions:** Do not rewrite framework-managed executors or reactive boundaries without Design Sensitive warning, exact approval, framework-specific validation, and rollback.
