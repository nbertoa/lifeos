# LifeOS

## Purpose

LifeOS is Nicolás Bertoa's personal source of truth for durable knowledge, engineering judgment, decisions, working practices, and long-term direction.

It exists to make important context easier to find, trust, reuse, and revise. Memory, disconnected notes, public profiles, and AI conversations are poor places to maintain facts and principles that must remain consistent over time. LifeOS gives that information an intentional home.

LifeOS is not a complete record of a life or an attempt to reproduce a person. It is a maintained knowledge system whose value is measured by whether it improves future judgment, decisions, or actions.

## Scope

LifeOS stores:

- Stable identity, professional history, values, and direction.
- Personally adopted engineering principles and working methods.
- Durable conclusions from learning and experience.
- Decision frameworks that are likely to be reused.
- Guidance for human and AI collaboration.
- Selected project knowledge when it has lasting value.

LifeOS should not store:

- Raw research dumps or copied vendor documentation.
- Temporary task notes, chat transcripts, or unreviewed brainstorming.
- Secrets, credentials, private personal data, or confidential employer and client information.
- Claims that cannot be supported.
- Broad topic coverage created for completeness.
- Every generated artifact merely because it might become useful.

The [Knowledge Principles](KnowledgePrinciples.md) provide the detailed inclusion test.

## Core Principles

### Preserve Judgment, Not Volume

Information belongs when it captures a fact, decision, principle, or tested practice that will matter again. Repository growth is not success; useful knowledge density is.

### Prefer Durable Knowledge

Durable knowledge remains useful beyond the immediate task. Temporary information helps with current execution but loses value quickly.

A decision criterion, verified career date, or adopted debugging method may be durable. A work-in-progress checklist, raw search result, or transient product detail usually is not. Temporary material may support a decision, but only its durable conclusion should enter LifeOS.

### Maintain a Canonical Source of Truth

Each important fact or concept should have one authoritative home. Other documents may summarize or present it for a particular audience, but they should link back instead of becoming competing sources.

Canonical does not mean infallible or permanent. It means that corrections begin there and derived material is reconciled afterward.

### Make Relationships Explicit

Documents should connect through meaningful repository-relative links. A link should clarify responsibility: where detail lives, which source governs a fact, or what framework should be applied next. Navigation lists alone do not replace conceptual links.

### Preserve Evidence and Uncertainty

Separate supported fact, adopted preference, inference, proposal, and unknown. Omit unsupported claims. When an uncertainty matters and cannot be resolved, label it narrowly and identify what would confirm it.

## Information Architecture

Top-level directories represent durable areas of life and work. Their `README.md` files provide local navigation. Focused documents own specific concepts, while root documents explain the repository as a whole.

Documents have one of three practical roles:

- **Canonical:** authoritative for a defined set of facts or decisions.
- **Derived:** adapted from canonical material for an audience or use case.
- **Index:** navigation and brief context for an area.

A document should state its role when confusion could create factual drift. Not every document needs metadata or a formal type label.

## Canonical Documents

The main foundation is:

- [Profile](Profile.md) — Concise identity and current direction.
- [Biography](Biography.md) — Narrative professional development.
- [Professional Journey](../Career/ProfessionalJourney.md) — Career facts, dates, roles, company relationships, and credentials.
- [Identity](Identity.md) — Mission and long-term professional direction.
- [Philosophy](../Personal/Philosophy.md) — Broader personal principles.
- [How to Think Like Me](HowToThinkLikeMe.md) — Operational reasoning and collaboration preferences.
- [Engineering Principles](../Programming/EngineeringPrinciples.md) — Concise engineering manifesto.
- [AI Playbook](../AI/AIPlaybook.md) — Rules for AI-assisted work in LifeOS.
- [Knowledge Principles](KnowledgePrinciples.md) — Inclusion and exclusion criteria.
- [LifeOS Evolution](Evolution.md) — Change and maintenance process.

These documents are canonical only for their stated responsibility. For example, the biography explains continuity, but the professional journey governs exact career facts.

## Derived Documents

Derived documents transform canonical information for a context without becoming new factual authorities. A résumé, portfolio biography, cover letter, public profile, prompt context, or project-specific checklist may be derived.

Derived documents should:

- Identify or link to the relevant canonical source.
- Preserve exact facts and important distinctions.
- Add presentation and selection, not unsupported claims.
- Be regenerated or reviewed after canonical facts change.
- Avoid being copied back into canonical documents without verification.

[Resume](Resume.md) is currently a reserved derived document rather than a duplicated career record.

## Knowledge Lifecycle

```text
Observation or source
→ Verification
→ Durable conclusion
→ Canonical placement
→ Review and use
→ Revision, consolidation, or removal
```

Before adding knowledge, determine what decision it improves, whether it is supported, whether it is durable, and whether an existing document already owns it. During review, compare it with related documents and preserve material uncertainty. After acceptance, link it from the smallest relevant index.

When knowledge changes:

1. Update the canonical source.
2. Identify affected derived documents and summaries.
3. Resolve contradictions deliberately.
4. Update navigation when responsibility or location changed.
5. Remove obsolete duplication when safe.

Knowledge may be consolidated or removed when it becomes stale, redundant, too broad, or no longer worth maintaining. Git history preserves prior text; the active repository should represent current adopted understanding.

## Contradictions

Do not resolve material contradictions by selecting the most polished or recent-looking statement.

1. Identify which document owns the fact or decision.
2. Compare the supporting evidence and document purpose.
3. Prefer the canonical source when it is current and supported.
4. If the canonical source may be wrong, preserve the conflict and request verification.
5. Correct the canonical source first, then reconcile derived documents and links.

Conflicting dates, employers, titles, credentials, beliefs, or technical claims require explicit review. Silence is preferable to invented resolution.

## AI Usage Rules

AI assistants should begin with [Profile](Profile.md), then read the canonical document for the task. They should use repository content before external assumptions, distinguish facts from inferences, and treat generated content as provisional until reviewed.

AI may summarize, compare, draft, implement, review, research, and help maintain links. It must not:

- Invent personal or career facts.
- turn an isolated statement into a stronger personality claim.
- Treat a derived document as more authoritative than its source.
- Expose secrets or confidential employer or client information.
- Expand the repository merely to complete a taxonomy.
- Claim validation that did not occur.

The [AI Playbook](../AI/AIPlaybook.md) defines the complete workflow.

## Maintenance Rules

- Update an existing document before creating a new one when responsibility already exists.
- Give every new document a distinct durable purpose.
- Keep indexes accurate and links repository-relative.
- Use clear English unless an area intentionally establishes another language.
- Prefer direct, restrained language over promotional claims.
- Keep changes cohesive, reviewable, and limited to the agreed scope.
- Verify names, dates, relationships, links, and technical claims.
- Review generated prose for repetition and false certainty.
- Review code and commands according to their risk, not their author.
- Record durable lessons in the most relevant existing document.
- Use [LifeOS Evolution](Evolution.md) when changing the structure or responsibilities of the system.

## Contribution Principles

A contribution should answer:

- What future decision or action will this improve?
- What existing source supports it?
- Which document owns it?
- Is it durable enough to maintain?
- Does it contradict or duplicate existing material?
- What verification is proportionate?

Discussion, implementation, commit, and publication are separate actions unless the current task authorizes them together. Human review remains responsible for accepting personal claims and AI-generated material.

## Non-Goals

LifeOS is not:

- A digital clone, artificial mind, or complete model of Nicolás.
- An encyclopedia of programming, Unreal Engine, AI, faith, or any external domain.
- A replacement for authoritative vendor documentation or source code.
- A raw archive of every thought, conversation, task, or activity.
- A public résumé, portfolio, or social profile, although those may derive from it.
- A system that removes the need for judgment, verification, privacy, or revision.

Its intended outcome is smaller and more practical: trustworthy context and reusable judgment for future work.
