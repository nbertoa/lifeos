# LifeOS Evolution

## Purpose

This document defines how LifeOS changes over time. Its purpose is to reduce unnecessary growth, duplicated knowledge, and architectural drift while preserving knowledge that remains valuable.

It establishes a shared process for evaluating change before effort is spent, so contributors can improve the repository without treating every idea as a file or structural change.

The [Knowledge Principles](KnowledgePrinciples.md) define what belongs in LifeOS. The [AI Engineering Collaboration](../AI/EngineeringCollaboration.md) guide defines how AI systems should assist with work. This document defines the process used to turn an idea into a deliberate repository change.

## Evolution Cycle

The default workflow is:

```text
Idea

↓

Discussion

↓

Decision

↓

Implementation

↓

Review

↓

Commit
```

Discussion is not implementation. Approval of an idea is not approval to modify files. Implementation begins only after the direction, intended outcome, and allowed scope are clear.

The cycle may be brief for a small correction and more deliberate for a durable change. It should not be skipped simply because an idea appears obvious. A short decision about scope often prevents a much larger correction later.

## Before Creating Anything

Before creating a document, directory, or other repository structure, answer:

- Why should this exist?
- What future decision or action will it improve?
- Does an existing document already cover the topic?
- Is this durable?
- Is this knowledge or only temporary research?
- Is the repository becoming larger without becoming better?

Creating a document should be the exception, not the default outcome. A useful outcome may be no change, a discussion, a short external reference, or an update to an existing document. New structure is justified only when it represents a distinct and durable concept that will remain useful over time.

## Implementation Rules

Prefer updating existing documents, improving clarity, making focused commits, recording explicit architectural decisions, and choosing reversible changes when possible. Keep each change small enough to understand, review, and correct.

Avoid creating documents merely because a topic exists, documenting for completeness, expanding scope during implementation, unrelated cleanup, large mixed commits, and speculative architecture. A change should solve the agreed problem without becoming an opportunity to reorganize everything nearby.

When a change affects navigation, links, or repository structure, update the smallest relevant index and validate the result. Preserve existing behavior and content unless the change is intentional. Do not assume that approval to discuss or implement also grants approval to commit, push, or remove content.

## Repository Reviews

Review the repository after meaningful growth, before a broad expansion, or when an area begins to feel difficult to navigate. Reviews should look for:

- Duplicated knowledge.
- Obsolete information.
- Documents that became too large.
- Documents that are never revisited.
- Opportunities to merge.
- Architectural drift.
- Areas dominating the repository identity.

Growth is not success. Useful knowledge density is success. A smaller collection that is easy to find, trust, and apply is more valuable than a larger collection that is difficult to maintain.

Reviews should produce evidence-based recommendations, not automatic cleanup. They may identify content to keep, merge, reduce, remove, postpone, or leave unchanged. Removal is appropriate only when the repository is not the correct place for the information; it is not a judgment that the information itself lacks value.

## Scope Control

Surface scope risk early when any of these statements becomes true:

- "This started as one document."
- "This is becoming a framework."
- "This is turning into an encyclopedia."
- "Research is continuing without improving decisions."
- "One folder is becoming the identity of the repository."
- "Completeness is replacing usefulness."

AI should explicitly recommend stopping, reducing, merging, or postponing work when these signs appear. The same applies to human contributors: a request should be narrowed when its next step does not improve a real future decision or action.

Scope control is not resistance to improvement. It is the discipline of preserving the part of the work that matters and discarding expansion that does not.

## Decision Outcomes

Every discussion should end with one of these outcomes:

- **No change**
- **Update existing document**
- **Create new document**
- **Merge documents**
- **Remove obsolete content**
- **Postpone**
- **Needs more discussion**

"No change" is often the correct decision. It means the current state is sufficient, the evidence is weak, the scope is unclear, or the proposed change does not provide durable value. Treating no change as a valid outcome prevents busywork from becoming repository growth.

Record the reason for a non-obvious decision in the relevant document, issue, review, or change discussion. The goal is not bureaucracy; it is making later decisions easier to understand.

## Long-Term Philosophy

LifeOS should become smaller, clearer, and more valuable over time. The repository should accumulate judgment rather than documentation, and every change should make future work easier.

Adding content is not progress unless it improves future decisions. A durable repository evolves through deliberate refinement: keep what matters, improve what is unclear, and avoid preserving information that does not earn its maintenance cost.
