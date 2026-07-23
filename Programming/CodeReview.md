# Code Review

## Purpose

Code review reduces risk and improves the quality of engineering decisions before a change becomes part of a codebase. It evaluates both the resulting code and the reasoning, assumptions, and trade-offs behind the change. A review should make integration safer and leave the shared understanding of the system stronger.

Review is not primarily about personal preference, stylistic dominance, proving technical superiority, finding the largest number of comments, or rewriting the author's solution into the reviewer's preferred solution. The author remains responsible for understanding and defending the change; review adds an independent perspective but does not transfer that ownership.

## Review Mindset

Review the change, not the person. Prefer correctness over agreement, challenge assumptions respectfully but directly, and distinguish defects from preferences. Explain why a concern matters rather than relying on a slogan or an unexplained request.

A reviewer should be skeptical without becoming adversarial. Do not approve unclear code merely because it appears to work, but do not block useful work for speculative perfection. Preserve context before proposing alternatives, and do not manufacture issues to make a review appear thorough.

> A strong review is critical about risk and generous about intent.

## Review Order

Review in an intentional order:

1. Understand the problem and intended outcome.
2. Confirm scope.
3. Evaluate correctness.
4. Evaluate architecture and boundaries.
5. Evaluate contracts, ownership, and failure behavior.
6. Evaluate readability and maintainability.
7. Evaluate tests and validation.
8. Evaluate performance where it is relevant.
9. Review style and polish last.

Style-first reviews waste attention and can hide deeper problems. A reviewer should inspect surrounding code, relevant call sites, and established conventions when necessary rather than judging isolated lines without context.

## Correctness

Correctness includes more than the happy path. Review expected behavior, edge cases, failure behavior, state transitions, invalid inputs, concurrency assumptions where relevant, ordering dependencies, lifecycle assumptions, partial failure, consistency with existing behavior, and unintended side effects.

Distinguish “I cannot prove this is correct” from “I have evidence this is incorrect.” Both may require action, but they are different claims. The first may call for context, a test, simplification, an explicit invariant, or inspection of call sites. The second should identify the failing behavior and its consequence. Do not approve meaningful-risk code by intuition alone.

When logic is difficult to establish, ask for evidence that matches the uncertainty. A clear test, reproduction, assertion, diagram, or smaller implementation is more valuable than a confident explanation that cannot be checked.

## Scope and Intent

Review whether the change solves the stated problem without unnecessary expansion. Look for unrelated cleanup, opportunistic refactoring, speculative abstraction, hidden behavior changes, new responsibilities, unrelated renames, dependency growth, and documentation expansion beyond the decision being made.

A small diff is not automatically well scoped, and a larger diff is not automatically excessive. The relevant question is: does every changed part support the intended outcome? Split changes when independent concerns make review harder or rollback riskier. Do not demand artificial splitting when the work is cohesive and the parts must change together.

## Design and Architecture

Evaluate responsibility boundaries, cohesion, coupling, dependency direction, abstraction level, extension points, data flow, ownership, public API growth, and consistency with the surrounding architecture. Prefer the simplest design that satisfies current requirements while leaving reasonable room for known change.

Challenge abstractions created for one hypothetical future case, local changes made globally configurable without evidence, unnecessarily broad interfaces, leaked implementation details, objects with unclear ownership, violations of module or layer boundaries, and duplicated business or gameplay rules. Do not reject a practical design solely because a more elegant theoretical design exists.

Architecture review should identify the decision that the code makes. If responsibilities or dependencies cannot be explained clearly, the reviewer should ask whether the change has introduced accidental complexity rather than merely requesting more comments.

## Readability and Simplicity

Readability is not measured by line count alone. Evaluate whether intent is visible, names explain responsibility, control flow is easy to follow, nesting is justified, important decisions are obscured by cleverness, comments explain why rather than restate what, functions are cohesive, the happy path is visible, and the code can be changed safely later.

Prefer explicit intent, guard clauses where appropriate, narrow functions, stable names, straightforward control flow, and local reasoning. Challenge compressed expressions, hidden side effects, ambiguous booleans, overly generic names, unnecessary indirection, and abstraction layers without clear value. Duplication is sometimes preferable to the wrong abstraction.

## Contracts and Defensive Programming

Use [Defensive Programming](DefensiveProgramming.md) as the detailed guide for contracts and failure behavior. During review, evaluate preconditions, postconditions, invariants, expected versus unexpected failure, validation boundaries, early returns, assertions, error propagation, and object validity after failure.

Ask whether the failure is expected, whether the selected mechanism matches it, whether an early return hides a broken invariant, whether execution can continue safely, whether validation is duplicated across layers, and whether design can prevent invalid state instead. Flag silent null returns for required dependencies, logging followed by corrupted-state execution, `check` for external input, `ensure` as routine control flow, generic `false` where a reason affects the caller, and partial mutation before validation.

## Ownership and Lifetime

Review who creates an object, who owns it, who may destroy it, and who only observes it. Make sure references that may become stale, callbacks that may outlive their owners, and asynchronous work that captures state have explicit lifetime rules. Teardown should be symmetrical, and optional and required dependencies should be distinguishable.

Repeated null checks often indicate unclear ownership or lifecycle design. For Unreal code, consider UObject lifetime, garbage collection, weak references, delegates, Actor and Component lifecycle, initialization order, editor versus runtime ownership, and latent or asynchronous callbacks. This is not a substitute for a complete lifecycle guide; it is a review prompt to make lifetime assumptions visible.

## Change Risk

Risk is not measured only by diff size. Consider affected callers, public API changes, serialization or persistence, save-game compatibility, networking, concurrency, initialization order, engine lifecycle, editor behavior, data migration, rollback difficulty, and observability if the change fails.

High-risk changes require stronger evidence, not merely more reviewer confidence. Useful evidence may include targeted tests, logs, profiling, screenshots, reproduction steps, before-and-after behavior, explicit manual verification, or staged rollout where available. The evidence should address the actual risk rather than produce activity without information.

## Testing and Verification

Verification should match the risk and behavior changed. Do not require tests mechanically for every edit. Ask what can regress, what behavior changed, what evidence demonstrates correctness, whether edge cases and failure paths are represented, whether a test observes behavior rather than implementation details, whether it would fail for the intended bug, and whether manual verification is sufficient for this risk.

Unit tests, integration tests, functional tests, manual verification, static analysis, compile validation, and runtime assertions each provide different evidence. Do not approve a change merely because tests pass when those tests do not cover the meaningful risk. Conversely, do not reject a low-risk, well-observed change solely because a particular test type is impractical.

## Performance

Performance is contextual. Review it when a change affects hot paths, per-frame execution, loops over large collections, allocations, serialization, networking, asynchronous work, loading, memory lifetime, or editor iteration time. Do not request micro-optimizations without evidence.

Ask for measurement when the concern is material. Prefer algorithmic improvements, fewer unnecessary operations, appropriate caching, reduced allocation, and clear data access patterns. Do not trade correctness and maintainability for speculative performance.

## Unreal Engine Considerations

For Unreal changes, review correct use of engine lifecycle; Actor versus Component responsibility; UObject ownership and validity; Blueprint/C++ boundaries; reflection exposure; replication and authority; gameplay tags; configuration versus hard-coded behavior; module dependencies; editor-only versus runtime code; and shipping-build behavior.

Evaluate Blueprint-facing APIs for clear names, safe defaults, limited surface area, meaningful categories, explicit authority or ownership expectations, and resistance to misuse. Do not demand Blueprint exposure just in case, and do not assume C++ is automatically superior to Blueprint or vice versa. Prefer the boundary that gives the project the clearest ownership, iteration, and maintenance model. When exact engine behavior matters, verify it against current Unreal Engine source or official Epic documentation.

## Review Feedback

Make the severity of feedback clear:

- **Blocking** — must be resolved before approval because correctness, safety, architecture, or meaningful maintainability is at risk.
- **Question** — context or reasoning is unclear.
- **Suggestion** — an improvement worth considering but not required.
- **Nit** — minor polish with no meaningful risk.
- **Praise** — a specific strong decision worth reinforcing.

Use the pattern `Observation → Risk → Recommendation`. For example: “This callback captures the owning object strongly. If the asynchronous operation outlives the owner, the lifetime becomes unclear. Could this use a weak reference and validate at execution time?” Avoid vague comments such as “This feels wrong,” “Not clean,” or “Use SOLID.” Explain the engineering consequence.

## Approval Standard

Approval means the reviewer understands the intent, no known blocking issue remains, the design is appropriate for the current requirement, meaningful risks are addressed, verification is proportionate, remaining suggestions are genuinely non-blocking, and the reviewer is comfortable with the code becoming part of the codebase.

Approval does not mean the code is perfect, the reviewer would have written it identically, no future refactor will be needed, or every suggestion must be implemented. Do not approve with unresolved blocking concerns, and do not withhold approval because of optional improvements.

## Review Checklist

### Intent and Scope

- Do I understand the problem, and does every changed part support it?
- Is unrelated work included?

### Correctness

- Are normal, edge, and failure paths correct?
- Are state transitions and lifecycle assumptions valid?
- Can partial failure leave invalid state?

### Design

- Are responsibilities and dependencies clear?
- Is the abstraction justified now, and is the public surface no broader than necessary?

### Defensive Programming

- Are contracts explicit, and are expected and unexpected failures handled differently?
- Can the code continue safely after a detected error?

### Ownership

- Are ownership and lifetime clear, and can references become stale?
- Are optional and required dependencies distinguishable?

### Maintainability

- Is intent easy to understand, and is control flow unnecessarily complex?
- Are names and comments useful?

### Verification and Integration

- What evidence supports correctness, and does it match the risk?
- Does the change follow repository architecture?
- Are Unreal lifecycle, replication, reflection, or shipping-build implications relevant?

## Long-Term Standard

Review the engineering decision, not only the diff. Correctness and risk come before style. Distinguish defects from preferences, demand evidence in proportion to risk, and prefer clear contracts and ownership. Do not block practical work for speculative perfection or approve code you do not understand. Explain consequences, not slogans. Approval means the change is safe and appropriate to integrate, not perfect. A good review leaves both the code and the shared engineering judgment stronger.
