# Problem Solving

## Purpose

Problem solving is the decision process used when the right problem, constraint, or approach is uncertain. Its objective is to reduce uncertainty until the next engineering decision is justified. It applies before and between implementation steps; it is not a delivery workflow or a debugging procedure.

[Engineering Playbook](EngineeringPlaybook.md) explains how this work fits the broader engineering cycle. [Debugging](Debugging.md) owns investigation after a failure or unexpected behavior is observed.

## Define the Actual Problem

Separate the desired outcome from the first proposed solution. Clarify current behavior, affected users or systems, constraints, success criteria, risks, and what is explicitly out of scope. Distinguish confirmed facts from assumptions, preferences, and implementation ideas.

A poorly framed problem invites an elegant solution to the wrong question. A useful problem statement makes the decision that must be made visible.

## Reduce Uncertainty Deliberately

Seek the smallest useful evidence: relevant existing knowledge, official documentation where exact behavior matters, a focused prototype, an experiment, a comparison of alternatives, or a conversation that clarifies a constraint. AI can accelerate exploration, but its reasoning and recommendations still require validation.

Do not collect research merely because it might be useful later. Stop when additional information no longer improves a real decision. Preserve only durable conclusions and reasoning that will be useful again.

## Decompose Without Losing the Decision

Break a large problem into smaller decisions that can be understood and validated independently. Decomposition should reveal responsibility, dependency, ordering, and risk; it should not turn one coherent decision into disconnected tasks.

Start with the part that has the highest uncertainty or that constrains the remaining options. A focused prototype can validate direction, but it should not become production architecture without reassessing its assumptions.

## Compare Options by Constraints

Evaluate options against correctness, ownership, maintainability, integration cost, evidence, risk, and the current project stage. Reuse proven patterns when they fit the real constraints, but do not copy an implementation mechanically because it worked elsewhere.

The best option is rarely the most general or technically impressive one. It is the approach that satisfies the actual requirement with the clearest trade-offs and the least unnecessary complexity.

## Make the Next Decision Explicit

Record the chosen direction, the evidence supporting it, the trade-offs accepted, and the uncertainty that remains. If evidence is insufficient, postpone the decision or run a focused experiment rather than disguising uncertainty as confidence.

Once direction is clear, move into the implementation cycle. Revisit the problem only when new evidence changes the constraints or invalidates the decision.

## Long-Term Standard

Good problem solving does not promise immediate answers. It turns uncertainty into explicit questions, gathers evidence proportionate to the decision, and chooses the next step deliberately.
