# API Design

## Purpose

API design governs how responsibilities, data, behavior, and assumptions are exposed to other code. An API is more than a public function: it includes classes, interfaces, constructors, data structures, events, delegates, configuration, callbacks, Blueprint nodes, module boundaries, serialized data, and service or command-line boundaries.

API quality affects correctness, maintainability, testability, discoverability, coupling, failure behavior, and the cost of future change. A poor API can make correct use difficult even when its implementation is sound. A good API makes valid usage obvious, invalid usage difficult, and ownership explicit.

## API Design Mindset

Design for the caller, not only for the current implementation. Expose intent rather than internal mechanics, make important assumptions visible, prefer narrow cohesive APIs, and do not expose flexibility without a real use case. Convenience is valuable only when it does not obscure behavior, lifetime, or failure.

An API should not require callers to know hidden implementation details or a precise undocumented call order. Important boundaries should make ownership, optionality, mutability, and failure understandable. Prevent invalid use by design where practical, then detect and communicate the remaining invalid use deliberately.

## Start with Responsibilities

API design begins by defining responsibility. Ask what the object owns, which decisions it is allowed to make, what information callers must provide, which details remain internal, which invariant it protects, and what happens when an operation cannot succeed.

Keep related behavior together and separate unrelated behavior. Command/query separation can clarify whether an operation changes state or merely reports it; separating policy from mechanism can keep decisions from becoming hidden implementation detail. These are useful tools, not a SOLID checklist. Their value is that callers can reason locally about one responsibility.

Avoid manager classes with undefined boundaries and utility APIs that accumulate arbitrary functionality. An API that cannot explain why its operations belong together is likely to become a dumping ground with unclear ownership and change cost.

## Public Surface Area

Every public symbol creates coupling, caller expectations, maintenance cost, misuse opportunities, and often compatibility obligations. Prefer the smallest surface that supports current requirements, private implementation by default, deliberate exposure, cohesive operations, and meaningful types.

Challenge internal containers exposed directly, public mutable state, convenience methods with no clear responsibility, getters and setters for every field, Blueprint exposure “just in case,” globally accessible helpers without ownership, and configuration with no demonstrated need. Public APIs should be treated as liabilities that must justify their existence. This does not mean making code artificially inaccessible or creating wrappers that add no boundary.

## Contracts

Contracts describe preconditions, postconditions, invariants, valid input ranges, side effects, failure guarantees, and relevant thread or lifecycle assumptions. Callers should not need to infer them from implementation details. Express a contract through types, names, assertions, comments, return types, access control, constructors, state machines, and validation at trust boundaries.

Ask whether the API prevents invalid calls, detects invalid calls, documents invalid calls, or silently creates undefined behavior. Prefer preventing invalid use over repeatedly detecting it. [Defensive Programming](DefensiveProgramming.md) is the canonical guide for detailed validation and failure mechanisms; API design should encode those decisions at the boundary.

## Required and Optional Dependencies

Required and optional dependencies must be distinguishable. A reference may represent a required non-null dependency where lifetime is guaranteed; a pointer may express absence, reseating, ownership, observation, or C API interoperability; an optional wrapper communicates meaningful absence. Constructors or explicit initialization APIs should establish mandatory dependencies rather than leaving values nullable forever.

Do not reduce this to “always use references.” Review whether a dependency is required, optional, owned, borrowed, observed, replaceable, or lazily available. Avoid boolean parameters that hide distinct modes, and avoid nullable values that are never valid. Repeated null checks are often compensation for an undefined lifecycle, not evidence of defensive design.

## Ownership and Lifetime

Make ownership explicit. Distinguish value ownership, unique ownership, shared ownership, borrowed references, and weak observation. The API should allow a caller to answer who creates an object, who destroys it, who may retain it, whether it can outlive the caller, whether a reference can disappear, who unregisters callbacks, whether cleanup is symmetrical, and whether shutdown is valid.

Avoid raw owning pointers without a clear policy, shared ownership by default, callbacks that retain owners indefinitely, references to unstable storage, APIs that retain arguments without saying so, and hidden global lifetime. Asynchronous work and object graphs deserve the same clarity as immediate calls: a callback contract is incomplete until its registration, execution, cancellation, and teardown rules are clear.

In Unreal, account for UObject garbage collection, `UPROPERTY` references, weak object references, Actor and Component lifecycle, delegates, latent actions, asynchronous callbacks, and editor-world versus runtime-world lifetime. These are constraints on API shape, not reasons to document every engine mechanism here.

## Inputs and Outputs

Parameters should communicate direction, ownership, mutability, optionality, scale and units, valid range, and failure semantics. Return the primary result where possible. Use output parameters only when they improve a coherent operation, not to simulate multiple unrelated returns. Prefer a parameter object when values form one concept, and named domain types when primitive combinations would be ambiguous.

Passing by value, const reference, mutable reference, or pointer in C++ depends on ownership transfer, copy cost, move semantics, nullability, and clarity. Do not apply a size-only rule. Avoid many positional parameters, boolean switches that radically change behavior, and results that depend on hidden mutable global state.

## Types and Invalid States

Use types to encode meaning. Enums are clearer than magic integers, domain types are safer than unrelated primitives, explicit results preserve failure meaning, and constrained construction can prevent partial validity. State machines can make lifecycle transitions explicit. Multiple booleans often permit contradictory combinations that a single tagged state would not.

> Invalid states should be unrepresentable where the design cost is reasonable.

For example, `void SetState(bool Enabled, bool Paused, bool Destroyed)` permits combinations that may have no valid interpretation. A named state or transitions that only allow valid combinations moves the decision into the model. This principle should not justify type proliferation or ceremony when a simple representation remains clear and safe.

## Mutability and Const-Correctness

Mutability is a design decision. Prefer immutable inputs, narrow command methods, visible state changes, and const-correct APIs. `const` is not merely syntax: it communicates and enforces intent. Avoid non-const accessors that expose internal storage, getters that permit uncontrolled mutation, modifiable references to containers, and `const_cast` used to bypass an unclear design.

Logical caching can be compatible with a read-only API when behavior remains logically const and thread assumptions are understood. The caller should still be able to tell whether an operation observes state or changes it.

## Naming and Discoverability

Names should reveal intent, responsibility, side effects, units, failure expectations, lifecycle stage, and ownership when important. A caller should be able to predict behavior without reading the implementation. Use consistent vocabulary and symmetry for related operations; do not use different names for the same concept or aliases that blur boundaries.

Challenge vague names such as `Handle`, `Process`, `Do`, `Update`, `Manage`, or `Execute` when context does not make the responsibility clear. Choose verbs deliberately for queries and commands, and use clear positive boolean names. For Unreal-facing APIs, follow Epic naming and reflection conventions where they apply.

## Error Communication

Error behavior is part of the contract. Choose explicit result objects, status enums, optional results, diagnostic errors, exceptions where the project permits them, assertions for programmer errors, logs for observability, or callbacks and delegates for asynchronous completion according to the failure category.

Distinguish expected operational failure, invalid caller usage, violated invariants, unavailable resources, partial completion, and asynchronous cancellation. Avoid plain `bool` when callers need a reason, null as an ambiguous catch-all, logging without communicating failure, success after partial failure, exceptions in a codebase that does not support them, and repeated translation of the same error at every layer.

## Abstraction and Extensibility

Abstract around real variation. An abstraction is justified when multiple real implementations exist, a stable concept has emerged, a meaningful boundary is protected, substitution or isolation improves, or harmful coupling is removed. Select composition, interfaces, policies, callbacks, data-driven configuration, templates, or inheritance according to the actual axis of change.

Avoid abstractions because a future implementation might appear, a pattern exists, dependency injection is considered universally desirable, every class is expected to have an interface, or duplication appears once. Duplication is often cheaper than the wrong abstraction. Inheritance is not a default code-reuse tool.

## Compatibility and Evolution

APIs evolve, but change cost differs by boundary. Consider source and binary compatibility where applicable, serialized data, public Blueprint APIs, save-game data, networking, plugin interfaces, module boundaries, deprecation, and migration paths. Internal APIs should be easier to change than public or serialized boundaries.

Prefer additive changes when compatibility matters, explicit deprecation, and removal of unused APIs where no promise exists. Do not preserve permanent complexity for hypothetical compatibility. Verify version-sensitive engine behavior before making commitments around it.

## Blueprint-Facing APIs

Blueprint and C++ interfaces may need different shapes because they serve different usage models. Design Blueprint-facing functions, properties, events, delegates, enums, structs, and configuration for clear node names, safe defaults, minimal pin count, meaningful categories, explicit authority expectations, world-context clarity, useful metadata, predictable failure, and limited surface area.

Do not expose implementation details, excessive BlueprintCallable functions, mutable public properties without validation, long nodes with many pins, or ambiguous pure functions with hidden cost. Blueprint exposure is API design, not annotation. Expose what supports intentional configuration or extension; protect invariants in the stable foundation.

## Common Failure Modes

- Unclear ownership causes stale references, leaks, or teardown bugs.
- Broad mutable state lets callers violate invariants without a boundary.
- Optional parameters that are actually required hide invalid usage.
- Booleans used as modes permit unclear or contradictory calls.
- Public APIs that mirror implementation details couple callers to refactors.
- Abstractions without a present need add indirection without protection.
- Inconsistent failure signaling forces callers to guess how to recover.
- Hidden side effects make local reasoning unreliable.
- Internal references with unstable lifetime create delayed failures.
- Required call order that is neither encoded nor documented creates fragile integration.
- Configuration used instead of making a design decision creates unsupported combinations.
- Exposing everything to Blueprint turns implementation detail into a long-term promise.

## API Review Checklist

### Responsibility

- Does the API have a clear responsibility and expose intent rather than mechanics?
- Is any operation unrelated to its main abstraction?

### Surface and Contracts

- Is every public element necessary, and can behavior remain private?
- Are preconditions, outcomes, failure behavior, and lifecycle assumptions clear?
- Are invalid states prevented or deliberately detected?

### Ownership and Types

- Who owns each object, and can borrowed references outlive their targets?
- Are callbacks and cleanup symmetrical?
- Do types communicate meaning and optionality without hiding domain concepts in booleans or primitives?

### Mutation and Evolution

- Is mutability narrow, intentional, and const-correct?
- Is the abstraction justified by current requirements?
- Is compatibility relevant, and is the public API larger than necessary for hypothetical future use?

### Unreal

- Is Blueprint exposure deliberate?
- Are UObject lifetime, lifecycle, reflection, replication, editor, or shipping implications understood?

## Long-Term Standard

Begin with responsibility. Keep public surfaces narrow. Make contracts and failure behavior explicit. Distinguish ownership from observation, use types to communicate meaning, and prevent invalid states where reasonable. Treat mutability as a design decision and abstract only around real variation. Design for callers without leaking implementation. Every API creates long-term coupling; the best API is often the smallest one that clearly supports the required behavior.
