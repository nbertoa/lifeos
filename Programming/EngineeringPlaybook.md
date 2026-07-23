# Engineering Playbook

## Purpose

This playbook integrates the repository's engineering principles into a practical approach for moving a software change from an unclear problem to a maintainable result. It is a decision framework, an entry point to deeper documents, and a guide for implementation and review that adapts to context.

It is not mandatory bureaucracy, a substitute for technical judgment, a complete development methodology, or a restatement of every Programming document. Engineering quality comes from a repeatable decision process, not isolated techniques.

## Engineering Priorities

Use this default priority order:

1. Correctness
2. Clear responsibility and ownership
3. Simplicity
4. Maintainability
5. Observability and diagnosability
6. Appropriate extensibility
7. Performance when evidence makes it relevant
8. Style and polish

Priorities may shift for real constraints, but convenience and elegance should not outrank correctness. Make the right behavior clear, make failure visible, make ownership understandable, preserve local reasoning, avoid unnecessary generality, demand evidence in proportion to risk, and make trade-offs explicit rather than hiding complexity.

## The Engineering Cycle

```text
Understand
    ↓
Constrain
    ↓
Design
    ↓
Implement
    ↓
Verify
    ↓
Review
    ↓
Integrate
    ↓
Learn
```

The cycle is lightweight and not strictly linear. New evidence may require returning to an earlier stage. That is iteration, not failure: a design discovered to violate a constraint should be reconsidered before more code is built on it.

## Understand the Problem

Before writing code, clarify the desired outcome, current behavior, affected users or systems, known constraints, success criteria, failure cases, scope, uncertainty, and available evidence. Separate requirement, assumption, implementation idea, and preference. Solving the first proposed implementation instead of the underlying problem creates expensive rework.

Use [Problem Solving](ProblemSolving.md) to reduce uncertainty and explore approaches. A small prototype or focused experiment can be better than a broad implementation when it answers a consequential question. The output of this stage is not certainty; it is a sufficiently clear problem and an explicit list of remaining risks.

## Define Constraints and Contracts

Identify inputs, outputs, preconditions, postconditions, invariants, ownership, lifecycle, expected failure, unacceptable failure, integration boundaries, and compatibility requirements. Constraints reduce the design space and make trade-offs visible.

Ask whether invalid states can be prevented through API and type design rather than repeatedly detected later. Use [Defensive Programming](DefensiveProgramming.md) for failure classification and validation boundaries, [API Design](APIDesign.md) for public contracts and ownership, and [Architecture](Architecture.md) for responsibility and coupling. This playbook does not replace their detailed guidance.

## Choose the Simplest Sound Design

Simple means minimizing unnecessary concepts and dependencies, not merely minimizing lines. Prefer cohesive responsibilities, narrow APIs, explicit data flow, composition, local reasoning, known project patterns, and direct solutions to current requirements.

Challenge speculative abstraction, premature frameworks, global state, unclear manager objects, configuration added to avoid making a decision, extension points without a current axis of variation, inheritance for convenience, and broad refactoring unrelated to the problem.

> The goal is not the smallest design. It is the simplest design that is correct, clear, and appropriately resilient.

## Implement in Small Coherent Steps

Implementation should proceed in increments that are understandable, verifiable, reviewable, reversible where practical, and cohesive. Preserve a buildable state, separate unrelated concerns, make intent visible, use existing conventions, minimize temporary invalid states, review the diff continuously, and update assumptions as evidence changes.

Do not require artificial micro-commits: a coherent change may span several files. [Development Workflow](DevelopmentWorkflow.md) covers practical coordination, feedback, and delivery, while [Estimation](Estimation.md) helps make scope and uncertainty explicit. Smallness is useful when it reduces review and rollback risk, not when it fragments one decision into meaningless pieces.

## Protect Invariants

During implementation, decide deliberately how assertions, validation, early returns, errors, defaults, fallback behavior, partial mutation, and recovery should work. Ask what must always be true, which failures are expected, which indicate a programming defect, whether execution can continue safely, where validation belongs, and whether failure leaves a valid state.

Early returns are useful when they keep valid control flow clear; they are dangerous when they silently hide a broken invariant. Do not use defensive code to compensate for unclear ownership or architecture. Refer to [Defensive Programming](DefensiveProgramming.md) for the detailed decision framework.

## Verify Behavior

Select verification according to risk. Evidence may include compilation, static analysis, targeted execution, assertions, automated tests where relevant, manual verification, logs, screenshots, traces, packaged builds, network scenarios, and engine tools. The important question is what can regress and what evidence demonstrates the behavior that matters.

Verification should consider failure paths and relevant lifecycle or integration contexts. “It compiled” is not sufficient evidence for a behavioral change. When unexpected behavior appears, use [Debugging](Debugging.md) to preserve the reproduction, gather evidence, and investigate the earliest divergence rather than applying random fixes.

## Review the Change

Self-review happens before external review. Review the final diff as if it were authored by someone else: does every changed part support the objective, is the code understandable without reconstructing hidden reasoning, are important assumptions encoded or documented, and is there sufficient evidence for approval?

Use [Self Review](SelfReview.md) to prepare work and [Code Review](CodeReview.md) to evaluate intent, correctness, scope, architecture, contracts, ownership, maintainability, verification, and risk. Distinguish defects from preferences. Review engineering decisions, not only code style.

## Integrate and Maintain

Integration includes clear commit history, focused changes, documentation when a decision needs preservation, removal of temporary diagnostics, explicit tracking of deferred work, monitoring high-risk changes where appropriate, and correction of stale knowledge. Merging code is not the end of engineering responsibility.

Post-integration evidence may reveal hidden assumptions, incomplete observability, future refactoring needs, documentation updates, or recurring failure patterns. Capture durable learning in the appropriate existing LifeOS document, not as a new document for every one-off detail. [Knowledge Principles](../About/KnowledgePrinciples.md) and [LifeOS Evolution](../About/Evolution.md) define those boundaries.

## Working with Unreal Engine

Apply the same process while accounting for engine lifecycle, UObject ownership, reflection, Blueprint/C++ boundaries, replication, editor and packaged differences, build configurations, module boundaries, asset and code interaction, data-driven design, and official Epic conventions.

Verify exact engine behavior through current official documentation, engine source, or reproducible experiments. Unreal-specific patterns do not replace general engineering judgment. Use the boundary that gives the project clear ownership, safe iteration, and maintainability rather than applying Blueprint or C++ mechanically.

## Warning Signs

- Implementation begins before the problem is understood, which turns assumptions into architecture.
- Every dependency is nullable, which often hides an undefined lifecycle.
- Repeated defensive checks conceal unclear ownership instead of fixing it.
- A new abstraction has only one hypothetical consumer, so it adds indirection without proven value.
- A change cannot explain failure behavior, leaving recovery accidental.
- Public APIs expand for convenience, creating unnecessary long-term coupling.
- An early return hides corrupted state rather than containing it.
- Correctness relies on undocumented call order, making integration fragile.
- Review feedback uses principles as slogans rather than consequences.
- Verification is only “it compiled” for a behavioral change.
- Debugging begins with random modifications, destroying evidence.
- Unrelated cleanup grows scope and makes review or rollback harder.
- Code works only because of engine behavior nobody verified.
- A document or abstraction is broader than the decision it preserves.

## Practical Checklist

### Understand

- What problem is being solved, what evidence defines current behavior, and what is outside scope?

### Constrain

- What must remain true, who owns the relevant data and objects, and which failures are expected?

### Design

- What is the simplest sound design, is responsibility cohesive, is the public surface minimal, and is the abstraction justified now?

### Implement

- Is the change coherent and reviewable, are contracts and ownership visible, and can failure leave invalid state?

### Verify

- What evidence demonstrates correctness, does it match risk, and have relevant lifecycle or integration contexts been checked?

### Review

- Does every changed part support the goal, are defects distinguished from preferences, and is the code safe to integrate?

### Learn

- Did the change reveal durable knowledge, should an existing document be updated, and can temporary investigation material be removed?

## Related Documents

- [Architecture](Architecture.md) — Responsibilities, coupling, and change-friendly structure.
- [API Design](APIDesign.md) — Narrow contracts, ownership, and public interfaces.
- [Code Review](CodeReview.md) — Evaluating change intent, risk, and integration readiness.
- [Debugging](Debugging.md) — Reproduction, evidence, root-cause analysis, and verification.
- [Defensive Programming](DefensiveProgramming.md) — Contracts, invalid states, and deliberate failure behavior.
- [Development Workflow](DevelopmentWorkflow.md) — Coordinating progress, feedback, and delivery.
- [Estimation](Estimation.md) — Scope, uncertainty, and transparent forecasting.
- [Problem Solving](ProblemSolving.md) — Reducing uncertainty before selecting a solution.
- [Self Review](SelfReview.md) — Author responsibility before sharing a change.

## Long-Term Standard

Understand the problem before selecting a solution. Make constraints, ownership, and failure behavior explicit. Prefer the simplest sound design, keep changes cohesive and evidence-driven, and protect invariants rather than hiding their violation. Verify behavior in proportion to risk and review engineering decisions, not only style. Use Unreal patterns deliberately, preserve durable learning without documenting everything, and treat good engineering as a repeatable process of reducing uncertainty, managing trade-offs, and leaving the next engineer with clearer decisions than the last. Repeat the cycle whenever new evidence changes the decision.
