# Self Review

## Purpose

Self-review is the author-side check performed before requesting external review or presenting a change as ready. Its purpose is to catch misunderstandings, regressions, scope growth, and unclear reasoning while the author still has the most context.

It does not replace [Code Review](CodeReview.md), which provides an independent reviewer perspective, or [Development Workflow](DevelopmentWorkflow.md), which covers coordination and delivery. [Engineering Playbook](EngineeringPlaybook.md) places self-review within the broader engineering cycle. Commit and push are separate authorized stages; completing self-review does not authorize either action.

## Review the Intended Outcome

Compare the final change with the original objective. Confirm that the problem is solved, that explicitly excluded work remains excluded, and that no assumption has silently become a requirement. A working implementation is incomplete when it solves a different problem than the one requested.

## Inspect the Final Diff

Read the final diff as if it were written by someone else. Verify that every changed file supports the intended outcome, unrelated cleanup is absent, and temporary code, diagnostics, generated artifacts, or abandoned experiments have not been included accidentally.

Small diffs are easier to understand, but size alone does not determine quality. A cohesive change may require several files; the question is whether each one is necessary and reviewable.

## Verify Behavior and Failure Paths

Check the expected behavior, meaningful edge cases, and relevant failure paths. Choose evidence proportionate to risk: compilation, static checks, targeted execution, automated tests where appropriate, manual verification, logs, or an explicit reproduction. “It compiled” is not sufficient evidence for a behavioral change.

Confirm that invalid input, missing optional dependencies, and unsupported states have deliberate behavior. If an invariant was broken during implementation, resolve the cause rather than hiding it with an early return. Use [Defensive Programming](DefensiveProgramming.md) for the detailed contract and failure model.

## Review Clarity and Integration

Ask whether another engineer can understand the change without reconstructing hidden reasoning. Check names, responsibilities, control flow, ownership, lifecycle, comments, and public API changes. Ensure the implementation follows the surrounding architecture and does not introduce speculative abstractions or unnecessary coupling.

Review related call sites and integration boundaries when the change affects shared behavior. For Unreal work, include the relevant lifecycle, Blueprint/C++ boundary, replication, editor, or shipping-build context rather than assuming the local code is sufficient.

## Prepare for External Review

State the intended outcome, meaningful trade-offs, evidence collected, and remaining uncertainty clearly enough for a reviewer to evaluate the change. Resolve known blocking concerns before requesting approval; keep genuine non-blocking improvements separate from the current scope.

## What Success Looks Like

Before requesting external review, I can answer yes to these questions:

- Does the final diff solve the intended problem without unrelated changes?
- Is the behavior verified in proportion to risk, including meaningful failure paths?
- Are contracts, ownership, and important assumptions clear?
- Does the change fit the existing architecture and remain understandable?
- Have temporary diagnostics and abandoned work been removed?
- Can another reviewer understand the evidence and remaining uncertainty?

## Long-Term Standard

Self-review is the last opportunity for the author to make a change easier to trust before another person must reason about it. Review the diff, the behavior, and the assumptions—not only whether the code appears to work.
