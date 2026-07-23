# Gameplay Architecture

## Purpose

Gameplay architecture should support iteration without turning the Character class into a container for every mechanic.

The goal is to make deliberate decisions about responsibilities, ownership, configuration, and extension points for the
actual project. This document records the patterns I intend to apply, not a complete description of engine systems.

For single-player prototypes, I do not add multiplayer complexity unless the project requires it.

---

## Design for the Real Project

I avoid preparing systems for hypothetical requirements that are not part of the current project.

If a prototype is single-player, I do not design around networking in advance. I begin with patterns that have proven
useful, then adapt them to the requirements rather than copying them mechanically. The architecture should solve the
current problem while remaining understandable and extensible.

---

## Divide Responsibilities Deliberately

I keep the Character focused on integration with the framework rather than using it as the owner of every mechanic.
Persistent systems with their own state and operations, such as inventory, health, or stamina, belong in components when
that separation reflects their actual responsibility.

I use Gameplay Ability System when it provides a useful foundation for executable gameplay capabilities and states. The
ability should coordinate an action; it should not become a container for unrelated rules, movement, presentation,
targeting, and special cases. Costs, cooldowns, state changes, and presentation should have clear homes instead of being
implemented as scattered custom logic.

Gameplay Tags are a shared vocabulary for concepts that need to be queried, communicated, allowed, or blocked. They are
valuable when they reduce direct dependencies, but they require consistent naming and should not become an ungoverned
replacement for clear design.

## Support Safe Configuration and Variation

A well-designed mechanic should expose appropriate configuration without requiring a C++ change for every variation.

Blueprints, assets, tags, effects, and data should allow project-specific variation where appropriate, while C++ defines
stable foundations and protects important invariants. A new variant should reuse the common foundation and modify only
the behavior or configuration that differs. Ease of variation is a practical measure of architectural quality.

---

## Review Integration Quality

Before considering a mechanic well integrated, I verify that:

- Responsibilities are explicit and there is no unnecessary duplicated logic.
- The Character and individual abilities remain focused.
- State, presentation, and configuration are not accidentally coupled.
- Tags and configuration are used consistently.
- Important invariants remain protected.
- A meaningful variant can be added without rewriting the original mechanic.

A mechanic is not complete simply because it works. It should also fit cleanly into the architecture.

---

## What Success Looks Like

A successful gameplay architecture:

- Solves the real requirements of the project.
- Keeps responsibilities, ownership, and variation clear.
- Avoids unnecessary complexity and speculative preparation.
- Supports configuration without weakening the stable foundation.
- Makes future variants easier to implement and review.
