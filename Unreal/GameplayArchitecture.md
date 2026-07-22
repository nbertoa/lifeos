# Gameplay Architecture

## Purpose

Gameplay architecture should support iteration without turning the Character class into a container for every mechanic.

The goal is to divide responsibilities clearly, reuse proven patterns, and design for the actual needs of the project.

For single-player prototypes, I do not add multiplayer complexity unless the project requires it.

---

## Design for the Real Project

I avoid preparing systems for hypothetical requirements that are not part of the current project.

If a prototype is single-player, I do not design the gameplay architecture around networking in advance.

Unnecessary preparation increases complexity before it provides value.

The architecture should solve the current problem while remaining understandable and extensible.

---

## Use Proven Architectural Patterns

When starting a new gameplay system, I reuse patterns that worked well in previous projects.

Previous experience helps reduce uncertainty and avoid repeating architectural mistakes.

This does not mean copying every previous solution unchanged.

It means beginning with a tested foundation and adapting it to the needs of the current project.

---

## Start With Gameplay Ability System

I now consider Gameplay Ability System from the beginning of gameplay development.

GAS provides a structured foundation for:

- Abilities.
- Attributes.
- Costs.
- Cooldowns.
- States.
- Restrictions.
- Gameplay events.
- Visual and audiovisual feedback.

Using it early helps avoid moving interconnected gameplay logic into an improvised architecture later.

---

## Use Enhanced Input

Enhanced Input is the standard input foundation for my Unreal projects.

Input should remain configurable and should connect cleanly with gameplay systems.

When appropriate, inputs can be represented through Gameplay Tags so the input layer and the ability layer share a
consistent language.

---

## Keep the Character Focused

I do not want the Character class to own every gameplay responsibility.

The Character should act primarily as an integration point between systems.

It may coordinate required Unreal functionality, but it should not become a monolithic class containing:

- Every gameplay action.
- Every persistent system.
- Every state transition.
- Every visual response.
- Every rule in the game.

A growing Character class is a signal that responsibilities may need to be redistributed.

---

## Use Components for Persistent Systems

I use components for systems that maintain their own persistent state and can exist as independent responsibilities.

Examples include:

- Inventory.
- Health.
- Stamina.

These systems usually:

- Store data.
- Expose operations.
- Maintain state over time.
- Communicate with several gameplay features.
- Benefit from being separated from the Character.

Components are not simply a solution for reducing file size.

They represent distinct systems with their own responsibilities.

---

## Use GAS for Gameplay Capabilities

I use Gameplay Ability System for actions and capabilities such as:

- Dash.
- Wall running.
- Interaction.
- Movement-related behavior.
- Targeting.

GAS is not limited to spectacular or combat-focused abilities.

It can provide the framework for organizing many kinds of character behavior.

The important distinction is that abilities represent executable gameplay capabilities, while components often maintain
persistent systems and data.

---

## Divide GAS Responsibilities

A Gameplay Ability should not contain every part of a mechanic.

I try to use each part of GAS according to its intended responsibility.

### Gameplay Abilities

Gameplay Abilities define activatable actions and their execution flow.

They coordinate the behavior of a mechanic without becoming a container for unrelated responsibilities.

### Gameplay Effects

Gameplay Effects represent:

- Costs.
- Cooldowns.
- Attribute changes.
- Temporary states.
- Persistent states.
- Gameplay modifiers.

Gameplay state changes should be expressed through effects when possible instead of being implemented through scattered
custom logic.

### Gameplay Cues

Gameplay Cues represent visual and audiovisual feedback.

Examples include:

- Particles.
- Sounds.
- Camera feedback.
- Impact presentation.
- Ability activation effects.
- Persistent status feedback.

Presentation should remain separate from the core gameplay rules.

### Gameplay Tags

Gameplay Tags provide a shared language across the architecture.

I use them wherever practical.

---

## Use Gameplay Tags Consistently

Gameplay Tags can represent:

- Character states.
- Abilities.
- Ability categories.
- Input actions.
- Blocking rules.
- Damage types.
- Teams.
- Factions.
- Gameplay events.
- Targeting states.
- Cooldowns.
- Permissions.
- Restrictions.
- Relationships between systems.

When a gameplay concept must be identified, queried, blocked, allowed, or communicated, I first consider representing it
with a tag.

Tags reduce the need for direct dependencies between systems and make gameplay rules more declarative.

---

## Prefer Declarative Rules

A system is easier to extend when its rules are represented through data, effects, and tags instead of being embedded in
large conditional blocks.

For example:

- Tags can determine whether an ability is allowed.
- Effects can apply costs and cooldowns.
- Cues can provide feedback.
- Abilities can focus on action flow.

This separation makes mechanics easier to understand, modify, and combine.

---

## Support Configuration Outside C++

A well-designed mechanic should expose appropriate configuration without requiring a C++ change for every variation.

Blueprints, assets, tags, effects, and data should allow designers to adjust behavior where appropriate.

Examples include:

- Costs.
- Cooldowns.
- Durations.
- Assets.
- Visual feedback.
- Ability classes.
- Tags.
- Tunable values.

C++ should define the stable foundation, while project-specific variation should remain configurable.

---

## Make Variants Easy to Create

A strong architecture should make it easy to add a variation of an existing mechanic.

Adding a new dash, interaction type, movement ability, or targeting mode should not require duplicating the entire
original implementation.

Variants should reuse the common foundation and modify only the behavior or configuration that differs.

Ease of variation is a practical measure of architectural quality.

---

## Review Integration Quality

Before considering a mechanic well integrated, I verify that:

- There is no unnecessary duplicated logic.
- The Character class has not grown excessively.
- Gameplay Tags are used consistently.
- Costs, cooldowns, and states are represented through Gameplay Effects.
- Visual and audiovisual feedback is implemented through Gameplay Cues.
- Relevant configuration can be changed without modifying C++.
- A new variant can be added without rewriting the system.

A mechanic is not complete simply because it works.

It should also fit cleanly into the architecture.

---

## Avoid Monolithic Abilities

Gameplay Abilities should remain focused.

A large ability that owns activation rules, movement, attributes, effects, feedback, targeting, and every special case
becomes difficult to understand and extend.

The ability should coordinate the mechanic while delegating responsibilities to:

- Effects.
- Cues.
- Tags.
- Components.
- Reusable tasks.
- Shared systems.

Working inside GAS does not automatically produce good architecture.

Responsibilities still need to be divided deliberately.

---

## What Success Looks Like

A successful gameplay architecture:

- Solves the real requirements of the project.
- Avoids unnecessary multiplayer complexity.
- Uses proven patterns from previous work.
- Introduces GAS early when it is appropriate.
- Uses Enhanced Input as a configurable foundation.
- Keeps the Character class focused.
- Uses components for persistent independent systems.
- Uses abilities for gameplay capabilities.
- Represents costs, cooldowns, and states through Gameplay Effects.
- Represents feedback through Gameplay Cues.
- Uses Gameplay Tags as a consistent shared language.
- Avoids duplicated logic.
- Supports configuration without frequent C++ changes.
- Makes new gameplay variants easy to implement.