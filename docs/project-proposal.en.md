# GUDGEON Project Proposal

**GUDGEON — Gudgeon Unified Debuggable Guest Execution Orchestrator for NIF**

- Status: Draft / Living Document
- Repository: `asopitech-labs/gudgeon-nif-execution-platform`
- Last updated: 2026-09-04
- License: Apache-2.0
- Japanese: [project-proposal.md](project-proposal.md)

## 1. Executive Summary

GUDGEON is an execution platform for programs represented in NIF (Nimony Intermediate Format). It brings interpreted execution, native compilation, debugging, instrumentation, and native interoperability under one execution model.

Traditional compiler infrastructure treats IR primarily as a temporary form on the way to native code. GUDGEON treats lowered program IR as an executable and observable asset. The same program model can run immediately in a VM, compile ahead of time, stop for inspection in a debugger, and emit data for profilers and tracers.

> Interpreted. Native. Observable.

## 2. Background and Problem

- Validating IR semantics often requires completing the native compilation pipeline.
- Separate semantic models for a compiler, VM, debugger, and instrumentation system tend to drift.
- Source locations, types, call history, and values can be lost during lowering.
- Calls from NIF to native code and from native hosts to NIF programs lack a unified contract.
- New optimization passes and backends need a deterministic reference implementation for validation.

GUDGEON addresses these problems by aligning execution, transformation, and observation around NIF.

## 3. Vision and Design Principles

GUDGEON aims to provide a consistent path to run a NIF program first, inspect it accurately, make it fast, and connect it to native systems.

1. **One semantic model** — VM and native code generation share program semantics.
2. **Debuggability by design** — Debug information survives from IR loading through execution.
3. **Observability as a first-class feature** — Tracing, profiling, coverage, and event hooks are standard capabilities.
4. **Native interoperability** — An explicit ABI supports calls in both directions.
5. **Incremental delivery** — Development starts with a minimal VM and expands in verifiable steps.
6. **Deterministic behavior** — Reproducibility is prioritized for testing and debugging.

## 4. Goals

- Load, verify, normalize, and execute NIF inputs.
- Build an interpreter/VM that serves as a reference implementation.
- Generate native code from the same IR and verify semantic agreement with the VM.
- Support breakpoints, stepping, stacks, variables, and expression evaluation.
- Integrate with existing editors through the Debug Adapter Protocol (DAP).
- Provide profiling, tracing, coverage, and instrumentation hooks.
- Provide safe, explicit FFI and embedding APIs.
- Maintain a conformance suite that detects differences between execution paths.

### Initial Non-goals

- Replacing the entire Nim or Nimony compiler.
- Supporting every CPU, operating system, and ABI at once.
- Implementing an advanced optimizing JIT in the early phases.
- Reimplementing a language server, package manager, or IDE.
- Matching VM and native performance from the beginning.

## 5. Target Users and Use Cases

### Target Users

- Nimony and NIF implementers and contributors
- Compiler, runtime, and debugger researchers
- Developers extending the Nim toolchain
- Native application developers embedding NIF programs
- Analysis-tool developers who need traces or coverage

### Primary Use Cases

- Load and execute a NIF file immediately to validate its semantics.
- Compare VM and native execution results.
- Debug a NIF-derived program from an editor.
- Record instruction, function, allocation, and external-call events.
- Call native libraries from a NIF program.
- Embed GUDGEON in a native application.
- Verify that an optimization pass preserves behavior.

## 6. Product Scope

### NIF Frontend

Load inputs, identify their version, verify syntax/types/references, preserve source locations, normalize to the execution representation, and report unsupported capabilities explicitly.

### Execution VM

Provide a reference model for calls, control flow, values, and memory. Support deterministic stepping, structured failures, cancellation, timeouts, and execution-state snapshots.

### Native Compilation

Provide an AOT path from the shared Execution IR. Share types, calling conventions, and debug metadata with the VM, and validate conformance through differential testing.

### Debugging

Support line, instruction, and function breakpoints; step in/over/out; continue and pause; stacks and variables; expression evaluation; failure stops; and a DAP adapter.

### Instrumentation

Expose events for instructions, basic blocks, functions, and external calls, together with profiling, tracing, coverage, and user-defined hooks.

### Native Interoperability

Provide an FFI for calls from NIF to native functions and an embedding API for calls from native hosts to NIF functions. Specify type conversion, ownership, lifetime, error, callback, and reentrancy rules.

## 7. Proposed Architecture

1. **Loader** — Reads NIF inputs and versions.
2. **Verifier** — Validates types, references, control flow, and ABI constraints.
3. **Execution IR** — A normalized representation shared by execution, compilation, debugging, and instrumentation.
4. **Runtime Services** — Provides memory, failures, external symbols, scheduling, and host APIs.
5. **VM Backend** — Interprets the Execution IR as the reference semantics.
6. **Native Backend** — Translates the same Execution IR to native code.
7. **Debug Service** — Provides execution control, state inspection, source mapping, and DAP.
8. **Instrumentation Service** — Provides events, profiling, tracing, and coverage.
9. **Embedding API / CLI** — Exposes the platform to tools and host applications.

The VM and Native Backend share contracts defined by Runtime Services and the Execution IR. Debugging and instrumentation connect through a common event and state model rather than backend-specific implementations.

## 8. Delivery Plan

### Phase 0 — Foundations

Define the target NIF subset, Execution IR, value model, repository layout, CI, testing conventions, diagnostics, and fixtures.

**Exit criterion:** The system can load a minimal input and emit stable verification results.

### Phase 1 — VM MVP

Implement integers, booleans, basic control flow, function calls, a CLI, deterministic execution tests, and structured diagnostics.

**Exit criterion:** Representative small programs run reproducibly in the VM.

### Phase 2 — Debugging and Instrumentation

Implement stops, stepping, stack/variable inspection, a minimal DAP adapter, function/instruction events, basic profiling, and coverage.

**Exit criterion:** A representative program can be debugged from an editor and produce an execution trace.

### Phase 3 — Native Compilation

Generate native code from the Execution IR, automate conformance tests against the VM, and add source mapping and native debug information.

**Exit criterion:** VM and native execution produce the same observable results for the supported subset.

### Phase 4 — Native Interoperability and Hardening

Stabilize FFI and embedding APIs, finalize ownership/error/callback/reentrancy contracts, and improve performance, security, compatibility, and diagnostics.

**Exit criterion:** Native hosts and NIF programs can call each other, and the public API compatibility policy is defined.

## 9. Deliverables

- GUDGEON CLI, NIF loader/verifier, Execution IR specification, and reference VM
- Native Backend, DAP adapter, and Instrumentation API
- FFI/Embedding API and conformance test suite
- Examples, integrations, and architecture/ABI/debugging/operations documentation

## 10. Quality Strategy

Combine unit, fixture, differential, property-based, fuzz, and integration tests. Continuously measure startup time, execution time, memory use, and instrumentation overhead. Compare observable events in addition to final VM and native results.

## 11. Success Metrics

- Instruction and type coverage for the defined NIF subset
- VM/native conformance-test pass rate
- Percentage of diagnostics that include a source location and cause
- Success rate of core DAP scenarios
- Instrumentation overhead when disabled and enabled
- Number of crashes, undefined behaviors, and nondeterministic failures
- Number of successful external integrations

Numeric targets will be fixed after Phase 0 establishes a baseline.

## 12. Risks and Mitigations

| Risk | Impact | Mitigation |
| --- | --- | --- |
| NIF specification changes | Loader and IR compatibility breaks | Identify input versions and isolate conversion layers |
| VM/native semantic drift | Loss of confidence in correctness | Use a shared Execution IR and differential tests |
| Lost debug metadata | Debugging becomes impractical | Preserve source locations and types from the beginning |
| FFI ownership mismatch | Crashes and leaks | Fix ABI, ownership, and error boundaries in docs and tests |
| Excessive scope | Delayed MVP | Define a NIF subset and exit criteria for every phase |
| Instrumentation overhead | Production use is discouraged | Provide a fast disabled path and graduated instrumentation levels |

## 13. Governance

- Maintain this proposal as the version-controlled source of project intent.
- Record important design decisions as Architecture Decision Records (ADRs).
- Track NIF support in a capability matrix.
- Define versioning and compatibility policies for public APIs and file formats.
- Complete milestones only when their exit criteria are met.
- Review both language versions whenever the proposal changes to prevent semantic drift.

## 14. Initial Backlog

1. Select the target NIF version and minimal subset.
2. Define how NIF fixtures are collected and licensed.
3. Specify the Execution IR for values, types, functions, and control flow.
4. Define the loader/verifier diagnostic format.
5. Implement the VM's minimal instruction set and execution loop.
6. Build a conformance harness that compares VM results with expected results.
7. Design a shared model for debug and instrumentation events.
8. Select the DAP requests required for the MVP.
9. Evaluate a minimal proof of concept for candidate Native Backends.
10. Design the ownership model for the FFI/Embedding API.
11. Select CI platforms and define the support policy.
12. Document security boundaries and handling of untrusted NIF inputs.

## 15. Open Questions

- Which NIF version and instruction subset should be fixed first?
- Where is the boundary between preserving NIF and normalizing to the Execution IR?
- Which technology should be the first Native Backend?
- How should GC, references, and ownership agree between VM and native execution?
- In which phase should async execution, threading, and exceptions be introduced?
- How should DAP map Nim sources, NIF, and generated code?
- What stability and external compatibility should instrumentation events guarantee?
- If untrusted inputs are supported, which isolation model should be used?

## 16. Definition of the First Public Release

- The supported NIF subset is documented.
- Representative programs run in the VM.
- Errors are structured and expose their input location and cause.
- Minimal debugging and execution tracing are available.
- Automated tests demonstrate VM/Native Backend conformance.
- CLI or embedding API examples are reproducible.
- Licensing, security, compatibility, and contribution policies are published.

## 17. Conclusion

GUDGEON makes NIF the center of execution, validation, observation, and integration rather than treating it as a temporary compiler artifact. A shared model for the VM, native compilation, debugging, instrumentation, and FFI improves both experimentation speed and correctness across the Nimony ecosystem.

This proposal will evolve with implementation and design decisions, with every change preserved in Git history.
