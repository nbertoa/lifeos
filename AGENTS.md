# LifeOS AI Collaboration Guide

## Purpose

LifeOS is my personal source of truth for durable knowledge, engineering judgment, decisions, and AI collaboration. It should preserve useful judgment and decisions, not become encyclopedic documentation. Use this file as the practical entry point; follow the linked documents for the detail needed by the task.

## Instruction Hierarchy

Apply instructions in this order:

1. Explicit instructions from the user in the current task.
2. Concrete project requirements and constraints.
3. A more specific local `AGENTS.md`.
4. This `AGENTS.md`.
5. Linked LifeOS documents.
6. General conventions or assistant preferences.

Surface material contradictions instead of resolving them silently.

## Collaboration Workflow

Use this default cycle, returning to an earlier stage when new evidence changes the decision:

```text
Understand
→ Constrain
→ Design
→ Implement
→ Verify
→ Review
→ Integrate
→ Learn
```

- Discussion does not authorize file changes; authorization to modify does not authorize commit or push.
- Before implementation, understand the problem, expected result, scope, and risks. Investigate material uncertainty or state the assumption being made.
- Keep changes small, cohesive, and reviewable. Do not expand scope through unsolicited refactors or cleanup.
- Inspect the complete final diff before finishing.

## Engineering Priorities

Default order:

1. Correctness.
2. Clear responsibility and ownership.
3. Simplicity.
4. Maintainability.
5. Observability and diagnosability.
6. Appropriate extensibility.
7. Performance when evidence makes it relevant.
8. Style and polish.

Simplicity minimizes concepts, dependencies, and difficult-to-reason-about states; it does not merely minimize lines of code.

## Design Principles

- Keep responsibilities cohesive and clear; make ownership and lifecycle explicit.
- Prefer low coupling, small APIs, and clear contracts.
- Prefer composition to inheritance unless specialization or a framework contract justifies inheritance.
- Avoid global state and “manager” objects without a concrete responsibility.
- Avoid speculative abstractions and extension points without a real need or axis of variation.
- Accept limited duplication when a premature abstraction would be worse.
- Use SOLID and Clean Code as reasoning tools, not dogma.
- Prioritize readability and safe change.

## Defensive Programming

Classify conditions before selecting a mechanism:

- **Preconditions** are caller obligations; **postconditions** are operation guarantees; **invariants** must hold throughout a valid object or subsystem lifetime.
- **Expected runtime errors** are normal, recoverable outcomes; **programming defects** break an internal contract.
- **Recoverable failures** preserve or restore known-valid state; **non-recoverable failures** cannot safely continue.

Prevent invalid states through design where practical. Checks must not compensate for undefined ownership or lifecycle, and not every pointer is automatically nullable. Deliberately choose assertion, validation, explicit error, early return, fallback, or failure propagation.

An early return is appropriate when it keeps the valid path clear or contains an expected failure. It is wrong when it hides a broken invariant or leaves partial state. Never continue silently after state is no longer safe. Assertion and log messages must provide useful diagnostic context.

## Unreal Engine Guidance

For Unreal work, explicitly consider engine lifecycle; `UObject` ownership and garbage collection; reflection; `UPROPERTY`, `UFUNCTION`, and Blueprint exposure; module boundaries; editor versus packaged behavior; build configurations; replication and authority; assets versus code; delegates and timers; async work; game-thread performance; and current Epic Games guidance.

Use C++ for systems, contracts, complex logic, performance-critical work, and architectural boundaries. Use Blueprint for composition, configuration, presentation, and gameplay iteration where appropriate. Do not choose either dogmatically: protect important boundaries and invariants, while giving the project the clearest ownership and iteration model.

Verify exact Unreal behavior through current official documentation, engine source, or reproducible experiments. Never invent APIs or assume engine behavior.

### Unreal Defensive Programming

- Use `check` or `checkf` for invariants and programming defects when continuing is unsafe.
- Use `ensure` or `ensureMsgf` for a defect when continuing can be safe and diagnostic value is useful.
- Use normal validation and explicit errors for external input and expected failures.
- Do not use `check` for legitimate user, network, optional-asset, or external-data conditions.
- Choose `IsValid`, `nullptr` checks, weak pointers, or another mechanism based on reference type and lifecycle; do not apply `IsValid` mechanically.
- Account for Development, Debug, and Shipping behavior.
- Do not let a silent fallback turn a deterministic defect into corrupt, difficult-to-diagnose behavior.

## Verification

Match evidence to risk. Relevant verification can include compilation, static analysis, automated or functional tests, targeted execution, logs, assertions, Unreal Automation Tests, PIE, Standalone, client/server scenarios, packaged builds, and profiling or traces. “It compiles” alone does not establish behavioral correctness.

Report precisely what was verified, what could not be verified, remaining risks, and assumptions. Never claim to have run a check that was not run.

## Code Review Standard

Review critically, in this order:

1. Correctness.
2. Crashes, corruption, and undefined behavior.
3. Ownership and lifecycle.
4. Invariants and failure handling.
5. Architecture and responsibilities.
6. API design and coupling.
7. Concurrency, replication, and performance.
8. Maintainability and readability.
9. Style.

Distinguish defects from preferences. Each finding must explain its concrete consequence. Review in context, including edge cases, partial states, and error paths; avoid dogmatic or purely stylistic comments. Do not approve a change merely because it looks clean.

## Repository Changes

Before creating a LifeOS file, ask what future decision it improves, whether the knowledge is durable, personally adopted or validated, whether an existing document can be updated, and whether it will remain useful in five years. “No change” is valid.

Use Markdown and English for LifeOS documentation. Do not add placeholders, taxonomies, frameworks, or documents merely to complete a topic.

## Git Policy

Work directly on `main` unless instructed otherwise. Unless explicitly authorized, do not create branches, commit, push, open pull requests, modify or remove unrelated content, or disturb existing user work. Preserve the working tree and show or summarize the final diff.

## Document Routing

| Task | Consult |
| --- | --- |
| General engineering process | [Programming/EngineeringPlaybook.md](Programming/EngineeringPlaybook.md) |
| Architecture | [Programming/Architecture.md](Programming/Architecture.md) |
| APIs and contracts | [Programming/APIDesign.md](Programming/APIDesign.md) |
| Defensive programming | [Programming/DefensiveProgramming.md](Programming/DefensiveProgramming.md) |
| Debugging | [Programming/Debugging.md](Programming/Debugging.md) |
| Code review | [Programming/CodeReview.md](Programming/CodeReview.md) |
| Self-review | [Programming/SelfReview.md](Programming/SelfReview.md) |
| Workflow | [Programming/DevelopmentWorkflow.md](Programming/DevelopmentWorkflow.md) |
| Unreal C++ versus Blueprint | [Unreal/BlueprintsVsCpp.md](Unreal/BlueprintsVsCpp.md) |
| Gameplay architecture | [Unreal/GameplayArchitecture.md](Unreal/GameplayArchitecture.md) |
| AI collaboration | [AI/EngineeringCollaboration.md](AI/EngineeringCollaboration.md) |
| LifeOS evolution | [About/Evolution.md](About/Evolution.md) |
| Knowledge inclusion | [About/KnowledgePrinciples.md](About/KnowledgePrinciples.md) |

## Final Standard

Understand before implementing. Make contracts, ownership, lifecycle, invariants, and failure behavior explicit. Choose the simplest design that is correct and resilient. Verify according to risk. Review engineering decisions, not only style. Preserve durable knowledge without documenting everything.
