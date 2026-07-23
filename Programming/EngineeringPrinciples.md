# Engineering Principles

## Purpose

These principles are Nicolás Bertoa's concise engineering manifesto. They guide decisions but are not universal laws. Project requirements, product constraints, team context, and verified evidence can justify a different trade-off.

The [Engineering Playbook](EngineeringPlaybook.md) turns these principles into a complete working cycle.

## 1. Correctness Comes First

Software must satisfy the real requirement and preserve valid state. A readable or elegant implementation is not successful when its behavior is wrong.

In practice, define expected behavior and meaningful failure cases before polishing the design. Verify behavior with evidence proportionate to risk. Correctness may compete with delivery pressure, but reducing scope is usually safer than knowingly weakening essential behavior.

## 2. Reduce Uncertainty Before Adding Complexity

Misunderstood requirements and hidden assumptions create more waste than a lack of abstraction.

Ask questions, inspect the existing system, separate facts from assumptions, and use focused experiments for consequential unknowns. Stop investigating when the next decision is justified; analysis should reduce risk, not postpone action indefinitely.

## 3. Prefer the Simplest Sound Design

Simplicity minimizes unnecessary concepts, dependencies, states, and mental overhead. It does not mean the fewest lines or the least robust implementation.

Choose direct solutions for current requirements. Avoid speculative frameworks, undefined manager objects, and configuration created only to avoid a decision. Add complexity when a real constraint requires it and its value can be explained.

## 4. Optimize for Readability and Local Reasoning

Code is maintained by people who should not need to reconstruct hidden intent.

Use clear names, cohesive functions, straightforward control flow, and comments that explain constraints or decisions. Avoid clever compression and hidden side effects. Sometimes a familiar idiom or performance-critical representation is less immediately simple; when so, make the reason discoverable.

## 5. Make Responsibility, Ownership, and Lifecycle Explicit

Clear responsibility and lifetime prevent entire classes of coupling, stale-reference, teardown, and partial-state defects.

Give components cohesive roles. Distinguish ownership from observation and required dependencies from optional ones. Define initialization, use, and cleanup. In Unreal Engine, account explicitly for UObject garbage collection, Actor and Component lifecycle, delegates, timers, async work, and editor versus runtime contexts.

## 6. Keep APIs Narrow and Contracts Visible

Every public operation creates coupling and a long-term expectation.

Expose intent rather than implementation mechanics. Make preconditions, outcomes, side effects, ownership, optionality, and failure behavior understandable. Prevent invalid states through types and construction when practical. Broader APIs are justified when real consumers or compatibility needs require them, not for hypothetical flexibility.

## 7. Separate Concerns Without Chasing Abstraction

Separation makes systems easier to understand and change when it follows real responsibilities.

Prefer composition for independently varying capabilities and inheritance for genuine specialization or framework contracts. Accept limited duplication when a shared abstraction would blur responsibilities or predict variation that has not appeared. Reuse should emerge from demonstrated needs.

## 8. Treat Failure Deliberately

Expected runtime errors, caller mistakes, and broken invariants require different responses.

Validate external input at the appropriate boundary. Use explicit errors for recoverable outcomes and assertions for programming defects when continuing would be unsafe. Do not log and continue with corrupted state, or use a silent early return to hide unclear ownership. Recovery is valid only when known-valid state is preserved or restored.

## 9. Measure Before Optimizing

Performance matters when it affects the product, platform constraints, iteration, memory, or user experience. Speculative optimization often trades clarity for no demonstrated gain.

Measure hot paths and regressions, then address the largest verified cause. Prefer algorithmic and structural improvements. Act earlier when a hard budget is known or an obvious frame-rate, latency, or memory regression appears.

## 10. Work in Small, Coherent Increments

Changes that can be understood, verified, reviewed, and corrected reduce integration risk and make feedback useful.

Keep related work together and unrelated work separate. Preserve a valid intermediate state where practical. Smallness is a means to clearer decisions, not a demand for artificial micro-commits.

## 11. Verify According to Risk

Compilation proves only part of correctness. Evidence should address what can actually regress.

Use tests, targeted execution, logs, assertions, static analysis, profiling, Unreal tools, PIE, standalone, packaged builds, or client/server scenarios as relevant. Report what was and was not verified. Low-risk documentation changes need different evidence from lifecycle-sensitive runtime code.

## 12. Review Engineering Decisions, Not Only Style

Review should reduce risk and strengthen shared understanding.

Evaluate intent, correctness, failure paths, ownership, lifecycle, architecture, contracts, maintainability, verification, and performance before style. Distinguish blocking defects from optional preferences and explain concrete consequences. Do not require theoretical perfection when a practical design is correct and appropriate.

## 13. Document Decisions That Will Matter Again

Durable knowledge prevents repeated reasoning and contradictory practice.

Record important constraints, trade-offs, contracts, and adopted lessons in the appropriate canonical document. Avoid raw research dumps, duplicated explanations, and documentation created for topic completeness. Update an existing source when it already owns the decision.

## 14. Share Knowledge and Invite Feedback

Teaching, mentoring, demonstrations, and review improve both the work and the people maintaining it.

Explain the reasoning behind important decisions, make progress and risk visible, and welcome evidence that challenges an assumption. Knowledge sharing should enable others rather than create dependency. Communication effort should remain proportionate; not every routine choice requires a presentation.

## 15. Use AI Without Outsourcing Judgment

AI can accelerate exploration, learning, implementation, and review, but it does not own the result.

Ask for alternatives and explanations, verify factual and technical claims, review generated code according to risk, and preserve understanding through each iteration. Do not repeatedly apply generated fixes until a symptom disappears without knowing why.

## Long-Term Standard

Build the right behavior with the simplest design that remains clear and resilient. Make responsibility, ownership, contracts, and failure visible. Measure material performance concerns, verify according to risk, communicate trade-offs, and preserve only the knowledge that improves future engineering decisions.
