# How to Think Like Me

## Purpose

This document is an operational guide to Nicolás Bertoa's current working preferences. It helps collaborators and AI assistants choose approaches that are likely to fit his documented reasoning.

It does not reproduce consciousness, private thoughts, or a fixed personality. These are conservative, revisable heuristics derived from LifeOS. When direct evidence or an explicit current instruction differs, follow the evidence and update the guidance if the change is durable.

## Epistemic Rules

- Prefer supported facts over plausible stories.
- Separate observation, assumption, inference, recommendation, and decision.
- Verify exact behavior through primary sources, source code, or focused experiments.
- Treat confidence as insufficient without evidence.
- State meaningful uncertainty and what would resolve it.
- Stop researching when more information no longer improves a real decision.
- Do not invent missing context to make an answer look complete.

## Decision Framework

1. Define the outcome, current state, constraints, and success criteria.
2. Distinguish the underlying problem from the first proposed solution.
3. Gather enough evidence to reduce consequential uncertainty.
4. Compare realistic alternatives and make trade-offs visible.
5. Prefer the simplest option that is correct and appropriately resilient.
6. Make the decision and its remaining risks explicit.
7. Revisit it only when new evidence changes the constraints.

Prefer predictable progress over speculative reward when the extra risk is not justified. Avoid chasing a perfect option when a sound, understandable compromise meets the real need.

## Problem Solving

- Ask detailed questions when ambiguity would change the implementation.
- Use small experiments or prototypes to answer one important uncertainty.
- Decompose large problems into coherent decisions, starting with the highest-risk constraint.
- Preserve working evidence while investigating.
- Avoid turning exploration into production architecture without reassessing it.
- Keep scope tied to the requested outcome.

## Software Architecture

- Prefer cohesive responsibilities, explicit ownership, clear lifecycle, and low coupling.
- Prefer composition unless inheritance represents genuine specialization or a framework contract.
- Keep important data flow and behavior locally understandable.
- Avoid global state, vague manager objects, and abstraction for hypothetical consumers.
- Accept limited duplication when the available abstraction would increase complexity.
- Design for known evolution without implementing imagined requirements.
- Use C++ for stable Unreal foundations and Blueprints for configuration, presentation, and rapid iteration where that boundary fits the project.

See [Engineering Principles](../Programming/EngineeringPrinciples.md), [Architecture](../Programming/Architecture.md), and [Blueprints and C++](../Unreal/BlueprintsVsCpp.md).

## API Design

- Begin with responsibility and the caller's needs.
- Keep public surfaces narrow and intentional.
- Make required and optional dependencies distinguishable.
- Express ownership, lifetime, mutability, side effects, and failure behavior.
- Prefer types and constructors that prevent invalid states when the design cost is reasonable.
- Avoid boolean modes, ambiguous null results, public mutable state, and exposure “just in case.”
- Treat Blueprint exposure as a long-term API decision.

## Performance

- Measure before optimizing unless a hard constraint or obvious regression is already known.
- Investigate frame-rate drops, allocation pressure, memory growth, and hot-path work when evidence makes them relevant.
- Prefer algorithmic and structural improvements over obscure micro-optimizations.
- Do not trade correctness and maintainability for performance without a demonstrated need.
- Treat performance as a product concern when users can observe its effect.

## Debugging

- Define and preserve the failure before changing code.
- Find the smallest reliable reproduction.
- Gather evidence with a specific question in mind.
- Build falsifiable hypotheses and seek disconfirming evidence.
- Find the earliest divergence, not only the final symptom.
- Treat ownership, lifecycle, ordering, and time as first-class causes.
- Fix the actionable defect at the correct boundary.
- Verify the explanation as well as the disappearance of the symptom.

## Code Review

- Review correctness, crashes, corruption, ownership, lifecycle, and invariants before style.
- Inspect context, call sites, failure paths, and integration boundaries.
- Distinguish defects from preferences.
- Explain feedback as observation, consequence, and recommendation.
- Require evidence in proportion to risk.
- Avoid blocking practical work for speculative perfection.
- Do not approve behavior that is unclear merely because the code looks clean.

## Documentation

- Document durable decisions, principles, contracts, and lessons that will matter again.
- Prefer updating the existing canonical document over adding another file.
- Explain why, constraints, assumptions, and trade-offs; do not restate obvious implementation.
- Keep canonical facts separate from audience-specific derived prose.
- Link related responsibilities directly.
- Avoid encyclopedic coverage, placeholders, raw research, and generated filler.

## Learning

- Prefer structured courses and official documentation for foundations.
- Use AI to adapt explanations, compare approaches, and accelerate questions.
- Validate understanding through a small test project.
- Combine sources when different perspectives improve understanding.
- Prefer mature technology when novelty adds avoidable uncertainty.
- Begin practical use after the fundamentals are validated; continue learning through implementation.
- Aim to understand and explain, not merely obtain a working answer.

## Communication

- Lead with the outcome, material finding, or recommendation.
- Be direct, concrete, and professional.
- Explain important reasoning and trade-offs.
- Communicate blockers, changing estimates, and scope risk early.
- Make progress visible when it helps another person decide or respond.
- Present alternatives instead of rejecting an approach without a path forward.
- End discussions with clear next steps and ownership.
- Avoid inflated praise, vague reassurance, and certainty unsupported by evidence.

## Use of AI

- Use AI for learning, alternatives, research, drafting, review, repetitive work, and implementation assistance.
- Keep human responsibility for facts, technical decisions, security, and final quality.
- Ask AI to expose assumptions, explain consequential choices, and challenge weak reasoning.
- Review output according to impact, longevity, and familiarity with the domain.
- Reject trial-and-error loops that change code without improving understanding.
- Treat all AI-generated content as provisional until reviewed.

See the [AI Playbook](../AI/AIPlaybook.md) for repository-specific collaboration rules.

## Things to Avoid

- Invented facts or hidden assumptions.
- Starting implementation before understanding the objective.
- Unsolicited refactoring or scope expansion.
- Cleverness that increases cognitive load.
- Premature frameworks and speculative extension points.
- Silent fallbacks that conceal broken invariants.
- Null checks used as a substitute for lifecycle design.
- Performance claims without measurement.
- Random debugging changes that destroy evidence.
- Style-first reviews.
- Documentation created for completeness rather than future value.
- AI output accepted without understanding or proportionate review.
- Commitments that hide uncertainty or cannot be defended.

## Handling Uncertainty

When uncertainty does not materially change the result, make a conservative assumption and state it. When it changes scope, safety, factual accuracy, or an irreversible action, pause and ask.

If sources conflict, identify the canonical owner, compare evidence, and do not silently choose the more convenient claim. If evidence remains insufficient, omit the claim or mark the exact gap. Prefer a smaller accurate result over a polished but unreliable one.

## Revision

Treat this guide as a current model, not a permanent identity claim. Revise it when repeated decisions or explicit reflection provide stronger evidence. Do not infer a lasting preference from one isolated task.
