# Unreal Engine Architecture Best Practices

## Research Metadata

- **Research date:** 2026-07-22
- **Documentation baseline:** Unreal Engine 5.8
- **Primary authority:** Epic Games documentation
- **Scope:** Runtime gameplay architecture
- **Status:** Living document

---

## Purpose

Unreal Engine provides several object types and frameworks with different responsibilities, lifetimes, ownership models,
world participation, and networking behavior.

Good Unreal architecture begins by placing each responsibility in the object whose semantics match the problem.

The central architectural questions are:

1. Does the system need to exist in the world?
2. Does it need a transform?
3. Who owns it?
4. How long should it live?
5. Does it exist on the server, clients, or both?
6. Does it need replication?
7. Is it associated with the application, a game instance, a world, a local player, a remote player, or an individual
   Actor?
8. Does it represent state, behavior, configuration, presentation, or infrastructure?
9. Should it be independently reusable?
10. Does it require intentional Blueprint extension points?

The most important architectural rule is:

> Choose an Unreal type according to responsibility, ownership, lifetime, authority, and world participation—not
> convenience alone.

---

## 1. Model Ownership and Lifetime First

### Recommendation

Before selecting an Unreal class, define the responsibility's owner and required lifetime.

### Classification

**Epic pattern**

### Why

Many Unreal architecture problems are lifetime mismatches:

- State is lost because it was stored on a Pawn that gets replaced.
- A system persists too long because it was placed in `UGameInstance`.
- Global state leaks between worlds or Play In Editor sessions.
- Local-player behavior is incorrectly shared by every player.
- Runtime logic is stored in an asset intended to describe configuration.
- Network state is stored in an object that does not replicate.

Unreal provides object types with explicit lifecycle semantics. Architecture becomes more predictable when those
semantics are used deliberately.

### Questions to Answer

Before implementing a system, determine:

- Who conceptually owns the responsibility?
- What creates it?
- What destroys it?
- Should it survive Actor destruction?
- Should it survive Pawn replacement?
- Should it survive map travel?
- Is it scoped to one world?
- Is it scoped to one local player?
- Does it need to exist on a dedicated server?
- Does it need to replicate?
- Does it need to be saved?
- Is it runtime state or authored configuration?

### Validation

Destroy or replace the expected owner and verify that the system:

- persists when it should;
- is destroyed when it should;
- does not leave stale references;
- initializes exactly once per intended scope;
- behaves correctly during map travel;
- behaves correctly in multiplayer;
- behaves correctly with multiple Play In Editor worlds.

---

## 2. Choose the Smallest Appropriate Unreal Abstraction

### Recommendation

Use the least powerful Unreal type that satisfies the actual requirements.

### Classification

**Inference based on Epic object-model documentation**

### Why

More powerful engine types generally bring additional semantics and cost.

For example:

- An `AActor` participates in a `UWorld`.
- A `USceneComponent` owns transform hierarchy behavior.
- A `UPrimitiveComponent` adds rendering, collision, or scene-proxy responsibilities.
- A replicated Actor adds networking considerations.
- A ticking object adds recurring runtime work.

A system that only needs data and behavior may not need to be an Actor.

### Decision Direction

Prefer:

- a plain C++ type for internal non-reflected implementation details;
- a `USTRUCT` for reflected value-oriented data;
- a `UObject` for reflected, garbage-collected, non-spatial behavior;
- an `UActorComponent` for reusable behavior owned by an Actor;
- a `USceneComponent` when that behavior also needs a transform;
- a `UPrimitiveComponent` when it needs renderable or collidable scene representation;
- an `AActor` when it needs an independent identity in the world.

### Avoid

- Creating Actors only to obtain convenient access to world functions.
- Giving every helper object a transform.
- Enabling replication before establishing a networking requirement.
- Enabling Tick by default.
- Using globally accessible objects to avoid defining ownership.

---

## 3. Use `UObject` for Non-Spatial Engine Objects

### Recommendation

Use `UObject`-derived classes for reflected, garbage-collected objects that do not need an independent world presence or
transform.

### Classification

**Epic pattern**

### Why

`UObject` is the shared base of much of Unreal's object system and supports:

- reflection;
- garbage collection;
- serialization;
- metadata;
- editor integration;
- Blueprint exposure where supported;
- Unreal object identity.

It is suitable for services, runtime models, strategies, tasks, processors, and other objects that do not need to be
placed in a level.

### Apply When

Use a `UObject` when the responsibility:

- requires Unreal reflection;
- owns or references other Unreal objects;
- should participate in garbage collection;
- may be created dynamically;
- does not need a world transform;
- does not need Actor replication;
- does not need to exist independently in a level.

### Implementation Guidance

- Create UObjects using Unreal-supported creation functions such as `NewObject`.
- Do not allocate UObjects with raw `new`.
- Provide an appropriate `Outer`.
- Treat the `Outer` relationship as part of the object's ownership and naming context.
- Keep valid references visible to Unreal's garbage collector.
- Do not assume that an `Outer` automatically guarantees every desired lifetime relationship.
- Use weak references where ownership is not intended.
- Avoid performing world-dependent work in constructors.
- Treat constructors primarily as default initialization.

### Validation

- Run garbage collection while testing object lifetimes.
- Verify that objects are collected when no longer referenced.
- Verify that active objects remain referenced.
- Test editor reconstruction, duplication, and serialization where applicable.

### Sources

- [Objects in Unreal Engine](https://dev.epicgames.com/documentation/unreal-engine/objects-in-unreal-engine)
- [Unreal Object Handling](https://dev.epicgames.com/documentation/unreal-engine/unreal-object-handling-in-unreal-engine)
- [Creating Objects in Unreal Engine](https://dev.epicgames.com/documentation/unreal-engine/creating-objects-in-unreal-engine)

---

## 4. Use Actors for Independent World Entities

### Recommendation

Use an `AActor` when an object needs an independent identity and lifecycle in a `UWorld`.

### Classification

**Official framework definition**

### Why

Actors are world objects that can:

- be spawned and destroyed;
- be placed in levels;
- own Components;
- participate in networking;
- receive world lifecycle callbacks;
- expose level-editable properties;
- represent independently addressable gameplay entities.

### Apply When

An Actor is appropriate when the object:

- must be placed or spawned in a world;
- needs an independent lifecycle;
- requires replication;
- needs level-specific configuration;
- must be discovered or referenced as a world entity;
- owns a collection of Components representing one entity.

### Avoid or Reconsider When

Do not default to an Actor when:

- the responsibility has no world identity;
- it exists only as part of another Actor;
- it is authored data;
- it is an application-wide or player-wide service;
- it does not need replication or world lifecycle;
- a Component or UObject expresses ownership more accurately.

### Implementation Guidance

- Keep the Actor responsible for coordinating its entity-level behavior.
- Delegate specialized responsibilities to suitable Components or objects.
- Avoid turning the Actor into a container for every system associated with it.
- Disable Tick unless recurring per-frame work is actually required.
- Treat spawning, initialization, possession, replication, and destruction as different lifecycle stages.
- Do not put essential runtime initialization exclusively in constructors.

### Sources

- [Actors in Unreal Engine](https://dev.epicgames.com/documentation/unreal-engine/actors-in-unreal-engine)
- [Gameplay Framework in Unreal Engine](https://dev.epicgames.com/documentation/unreal-engine/gameplay-framework-in-unreal-engine)

---

## 5. Use Components for Actor-Owned Capabilities

### Recommendation

Use Components for reusable responsibilities that belong to an Actor and share its ownership context.

### Classification

**Official recommendation and Epic pattern**

### Why

Components support composition.

They allow an Actor to be constructed from focused responsibilities instead of relying exclusively on inheritance or a
monolithic class.

Epic distinguishes several important Component categories.

| Type                  | Primary Responsibility                                                 |
|-----------------------|------------------------------------------------------------------------|
| `UActorComponent`     | Non-spatial behavior and state                                         |
| `USceneComponent`     | Behavior or representation requiring a transform                       |
| `UPrimitiveComponent` | Renderable, collidable, or otherwise scene-proxy-backed representation |

### Actor Components

Use `UActorComponent` for abstract, non-spatial responsibilities such as:

- inventory;
- attributes;
- interaction capabilities;
- state tracking;
- reusable Actor-owned services;
- gameplay coordination.

An Actor Component has no transform.

### Scene Components

Use `USceneComponent` when the responsibility:

- needs position, rotation, or scale;
- participates in an attachment hierarchy;
- acts as a transform anchor;
- has child Scene Components.

### Primitive Components

Use `UPrimitiveComponent`-derived types for responsibilities that participate in:

- rendering;
- collision;
- overlaps;
- scene queries;
- physical representation.

### Apply When

A Component is a good fit when:

- its lifecycle belongs to an Actor;
- the behavior may be reused by multiple Actor classes;
- it has a focused responsibility;
- it benefits from composition;
- it needs direct access to an owning Actor;
- it should be configurable as part of an Actor.

### Avoid or Reconsider When

A Component may not be appropriate when:

- it needs to exist independently of an Actor;
- it has application-, world-, or player-scoped lifetime;
- it is only static configuration data;
- it requires independent network ownership;
- it has no meaningful relationship with the owning Actor;
- extracting it only moves a monolithic design into another large class.

### Implementation Guidance

- Create stable built-in Components as default subobjects where appropriate.
- Use `SetupAttachment` during construction for unregistered Components.
- Use runtime attachment APIs for Components that are already registered.
- Keep Component responsibilities focused.
- Avoid circular dependencies between Components.
- Communicate through explicit interfaces, delegates, or owner-mediated APIs where that reduces coupling.
- Enable Component replication only when required.
- Remember that Component replication depends on the owning Actor's networking context.
- Avoid unnecessary Component Tick.

### Sources

- [Components in Unreal Engine](https://dev.epicgames.com/documentation/unreal-engine/components-in-unreal-engine)
- [Replicating Actor Components](https://dev.epicgames.com/documentation/unreal-engine/replicating-actor-components-in-unreal-engine)

---

## 6. Prefer Composition, but Do Not Reject Inheritance

### Recommendation

Use composition for separable capabilities and inheritance for true specialization.

### Classification

**Epic pattern**

### Why

Unreal's architecture supports both:

- inheritance through Actor, Pawn, Character, Controller, Component, and UObject class hierarchies;
- composition through Components and contained objects.

Composition is useful when capabilities vary independently.

Inheritance is useful when a type genuinely satisfies the semantic contract of its base class.

### Use Inheritance When

- The derived class is a true specialization of the base type.
- Engine integration depends on overriding framework callbacks.
- Shared behavior is stable across the hierarchy.
- Polymorphic use is intentional.
- The base class defines a meaningful contract.

### Use Composition When

- Features should be independently reusable.
- Capabilities may be combined in different ways.
- A large base class would accumulate unrelated responsibilities.
- Runtime or data-driven configuration determines included capabilities.
- Multiple Actor types need the same focused behavior.

### Avoid

- Deep inheritance trees built only to share small implementation details.
- Components that depend on exact concrete owner classes without a clear reason.
- Base classes that become repositories for unrelated optional systems.
- Inheritance selected solely because it is faster to implement initially.

---

## 7. Use Gameplay Framework Classes According to Their Semantics

### Recommendation

Place gameplay responsibilities in the Gameplay Framework class designed for the required authority, ownership, and
lifetime.

### Classification

**Official framework guidance**

### Framework Responsibility Matrix

| Class                           | Primary Scope                               | Network Presence                | Typical Responsibility                                   |
|---------------------------------|---------------------------------------------|---------------------------------|----------------------------------------------------------|
| `AGameModeBase` / `AGameMode`   | Current match or world rules                | Server only                     | Rules, spawning policy, match flow                       |
| `AGameStateBase` / `AGameState` | Shared match state                          | Server and replicated clients   | State relevant to all participants                       |
| `APlayerController`             | One player's control connection             | Server and owning client        | Input coordination, possession, player-specific commands |
| `APlayerState`                  | One participant's replicated identity/state | Server and all relevant clients | Score, team, persistent match identity                   |
| `APawn`                         | Possessable world representation            | Depends on replication          | Player- or AI-controlled avatar                          |
| `ACharacter`                    | Pawn with Character Movement conventions    | Depends on replication          | Humanoid-style character movement                        |
| `AHUD`                          | One player's HUD coordinator                | Local player context            | Canvas HUD and player presentation coordination          |
| `UGameInstance`                 | Running application instance                | Local process                   | State and services spanning map travel                   |
| `ULocalPlayer`                  | One local human player                      | Local process                   | Local-player-specific systems and context                |

---

## 8. Keep Authoritative Rules in Game Mode

### Recommendation

Use `GameMode` for authoritative rules and match orchestration that only the server needs.

### Classification

**Official framework guidance**

### Appropriate Responsibilities

- Selecting default framework classes.
- Player login and logout handling.
- Spawn selection.
- Restarting players.
- Match transitions.
- Win and loss evaluation.
- Server-authoritative rule enforcement.

### Important Constraint

`GameMode` exists only on the authoritative server in a networked game.

Clients must not depend on direct access to it.

### Avoid

- Storing information clients need to display directly in `GameMode`.
- Putting persistent application services in `GameMode`.
- Treating `GameMode` as a general global singleton.
- Placing player-specific state in `GameMode`.

### Use Instead

Put replicated shared state in `GameState`.

---

## 9. Put Replicated Shared Match State in Game State

### Recommendation

Use `GameState` for match-level information that clients need to observe.

### Classification

**Official framework guidance**

### Appropriate Responsibilities

- Match phase.
- Shared objectives.
- Team scores.
- Match timer.
- Global match status.
- Collection of Player States.
- Replicated information relevant to participants.

### Avoid

- Storing server-secret information that clients must not receive.
- Storing application-wide persistence.
- Using it as a replacement for every independent replicated Actor.
- Placing purely local UI state in it.

### Principle

> GameMode defines authoritative rules; GameState communicates shared match state.

---

## 10. Use Player Controller for Player Control and Coordination

### Recommendation

Use `PlayerController` for responsibilities associated with one player's control connection rather than the physical
avatar alone.

### Classification

**Official framework guidance**

### Appropriate Responsibilities

- Possession.
- Player-originated commands.
- Input routing and high-level coordination.
- Camera coordination where appropriate.
- Owning-client RPC entry points.
- Player-specific interaction with Pawns.
- Coordination of local player presentation.

### Why

A PlayerController and a Pawn have different lifetimes.

The controlled Pawn may:

- die;
- respawn;
- be replaced;
- switch vehicles;
- become a spectator;
- change forms.

The PlayerController represents the controlling player relationship across those changes.

### Avoid

- Putting avatar-specific physical behavior in the PlayerController.
- Assuming every PlayerController exists on every client.
- Storing shared replicated player identity exclusively in it.
- Coupling UI widgets directly to every gameplay implementation detail.

---

## 11. Use Player State for Replicated Participant State

### Recommendation

Store replicated participant information that should be visible beyond the owning client in `PlayerState`.

### Classification

**Official framework guidance**

### Appropriate Responsibilities

- Player name.
- Score.
- Team.
- Match statistics.
- Participant status.
- Information that should survive Pawn replacement.
- Replicated player identity during a match.

### Why

Player State is designed to represent a participant separately from the currently possessed Pawn.

### Avoid

- Placing Pawn-specific transient movement state in Player State.
- Putting local preferences in Player State.
- Storing secrets that should not replicate to other players.
- Treating it as permanent account storage.

---

## 12. Use Pawn and Character for the Controllable World Avatar

### Recommendation

Use `Pawn` for possessable world entities and `Character` when the built-in Character conventions match the movement
model.

### Classification

**Official framework definition**

### Pawn

A Pawn represents an Actor that may be possessed by a player or AI Controller.

Use it for:

- vehicles;
- creatures;
- cameras;
- board-game pieces;
- custom movement models;
- non-humanoid controllable entities.

### Character

Character extends Pawn with conventions centered around:

- capsule-based collision;
- skeletal mesh presentation;
- `CharacterMovementComponent`;
- common networked character movement behavior.

### Avoid

- Using Character for every controllable entity regardless of movement needs.
- Storing player identity or match persistence exclusively on the Pawn.
- Making the Pawn responsible for every system associated with the player.
- Assuming possession and Pawn initialization happen in one universal order in all network contexts.

---

## 13. Use Game Instance for Application-Lifetime Context Sparingly

### Recommendation

Use `GameInstance` for information and coordination that must span map travel within one running application instance.

### Classification

**Official framework guidance**

### Appropriate Responsibilities

- Application-session context.
- High-level travel coordination.
- Services that must survive ordinary map transitions.
- Access to Game Instance Subsystems.
- Process-local session state.

### Avoid

Do not use Game Instance as:

- a dumping ground for globally accessible variables;
- replicated shared state;
- a replacement for save data;
- a replacement for World-scoped systems;
- a home for Actor references that become invalid during travel;
- the owner of every manager in the project.

### Important Networking Constraint

Each process has its own Game Instance.

It is not replicated between server and clients.

### Principle

> Persistence across maps does not imply persistence across application launches, and local process state does not imply
> network-shared state.

---

## 14. Use Subsystems for Automatically Managed Services

### Recommendation

Use a Subsystem when the responsibility is a service whose lifetime exactly matches one of Unreal's supported Subsystem
scopes.

### Classification

**Official recommendation**

### Why

Epic describes Subsystems as automatically instanced classes with managed lifetimes. They provide extension points
without requiring complex modification or overriding of engine classes.

### Common Runtime Subsystem Types

| Subsystem                | Lifetime               |
|--------------------------|------------------------|
| `UEngineSubsystem`       | Engine lifetime        |
| `UGameInstanceSubsystem` | Game Instance lifetime |
| `UWorldSubsystem`        | World lifetime         |
| `ULocalPlayerSubsystem`  | Local Player lifetime  |

Editor-specific Subsystems also exist, such as `UEditorSubsystem`.

### Choose by Scope

#### Engine Subsystem

Use for truly engine-process-wide services.

Reconsider carefully in editor workflows because one engine process may host multiple worlds and Play In Editor
sessions.

#### Game Instance Subsystem

Use for services that:

- span map travel;
- belong to one running game instance;
- do not require Actor representation;
- do not replicate automatically.

Examples may include online-service coordination or application-session services.

#### World Subsystem

Use for services that:

- belong to one `UWorld`;
- should initialize and deinitialize with that world;
- coordinate world-specific systems;
- must not leak across independent worlds.

#### Local Player Subsystem

Use for services associated with one local player, such as:

- local input context;
- local UI coordination;
- local-player preferences;
- local-player-specific runtime services.

This is especially important for split-screen and multiple-local-player support.

### Avoid or Reconsider When

Do not use a Subsystem merely to create a globally accessible object.

A Subsystem is a poor fit when:

- the responsibility belongs to one Actor;
- it requires independent world representation;
- its lifetime does not match the selected scope;
- multiple instances are needed within that scope;
- it represents authored data;
- it needs Actor replication;
- explicit ownership would be clearer.

### Implementation Guidance

- Select the narrowest correct Subsystem lifetime.
- Keep the service focused.
- Avoid hidden dependencies between many Subsystems.
- Do not assume initialization order without defining it.
- Use initialization and deinitialization callbacks.
- Account for multiple worlds in editor and test environments.
- Avoid retaining stale world or Actor references.
- Do not assume a Subsystem is replicated because it exists on multiple machines.

### Sources

- [Programming Subsystems in Unreal Engine](https://dev.epicgames.com/documentation/unreal-engine/programming-subsystems-in-unreal-engine)
- [Gameplay Framework in Unreal Engine](https://dev.epicgames.com/documentation/unreal-engine/gameplay-framework-in-unreal-engine)

---

## 15. Distinguish Runtime State From Authored Data

### Recommendation

Store authored definitions in assets and mutable runtime state in runtime objects.

### Classification

**Epic pattern**

### Why

Unreal supports data-driven systems through:

- Data Assets;
- Primary Data Assets;
- Data Tables;
- Curves;
- configuration files;
- Developer Settings;
- Blueprint defaults;
- Gameplay Tags.

These mechanisms allow content to be authored and tuned separately from executable runtime state.

### Authored Data Examples

- Item definitions.
- Ability definitions.
- Character archetypes.
- Balance values.
- Loot tables.
- AI configuration.
- UI style definitions.
- Asset references.
- Content classification.

### Runtime State Examples

- Current health.
- Inventory instances.
- Active cooldowns.
- Current objective progress.
- Spawned entities.
- Match scores.
- Temporary selections.

### Avoid

- Mutating shared asset definitions as though they were per-instance state.
- Duplicating large definitions into every runtime object unnecessarily.
- Using runtime Actors as databases for authored content.
- Hard-coding frequently tuned content in foundational C++ without a reason.

### Sources

- [Data Assets in Unreal Engine](https://dev.epicgames.com/documentation/unreal-engine/data-assets-in-unreal-engine)
- [Asset Management in Unreal Engine](https://dev.epicgames.com/documentation/unreal-engine/asset-management-in-unreal-engine)

---

## 16. Design Asset References Deliberately

### Recommendation

Choose hard and soft references according to required loading behavior.

### Classification

**Official guidance**

### Hard References

A hard object or class reference can make the referenced asset part of the referencing object's load dependency chain.

Use when:

- the dependency is essential;
- it must be available immediately;
- loading both assets together is intentional;
- the memory and packaging consequences are understood.

### Soft References

Soft object and class references identify assets without requiring immediate loading.

Use when:

- content should load on demand;
- optional content should not inflate initial memory;
- large content sets are selected dynamically;
- asynchronous loading is required;
- dependency chains must be controlled.

### Primary Assets

Primary Assets have identities understood by the Asset Manager and can be managed directly.

Use where projects need deliberate:

- loading and unloading;
- asset discovery;
- bundles;
- cooking rules;
- chunking;
- content classification.

### Avoid

- Replacing every hard reference with a soft reference without a loading design.
- Calling synchronous loads in latency-sensitive paths without measurement.
- Assuming soft references eliminate all dependencies automatically.
- Creating custom asset-loading systems without evaluating Asset Manager support.

### Sources

- [Asset Management in Unreal Engine](https://dev.epicgames.com/documentation/unreal-engine/asset-management-in-unreal-engine)
- [Asynchronous Asset Loading](https://dev.epicgames.com/documentation/unreal-engine/asynchronous-asset-loading-in-unreal-engine)
- [Data Assets in Unreal Engine](https://dev.epicgames.com/documentation/unreal-engine/data-assets-in-unreal-engine)

---

## 17. Separate Rules, State, Commands, and Presentation

### Recommendation

Avoid placing authoritative rules, mutable state, player commands, and presentation in one class.

### Classification

**Inference from Epic framework separation**

### Architectural Categories

#### Rules

Determine what is permitted and how the game progresses.

Examples:

- GameMode rules.
- Ability activation requirements.
- Damage rules.
- Match-state transitions.

#### State

Represents the current result of those rules.

Examples:

- GameState.
- PlayerState.
- attributes;
- inventory state;
- active Gameplay Effects.

#### Commands

Represent requested actions.

Examples:

- input actions;
- server RPC requests;
- ability activation attempts;
- interaction requests.

#### Presentation

Communicates state and outcomes to the player.

Examples:

- UI;
- animation;
- audio;
- Niagara;
- Gameplay Cues;
- camera feedback.

### Why

This separation improves:

- multiplayer authority;
- testing;
- reuse;
- prediction;
- debugging;
- UI replacement;
- content iteration.

### Avoid

- Letting UI widgets authoritatively modify game outcomes.
- Letting animation state become the sole source of gameplay truth.
- Treating cosmetic effects as authoritative events.
- Mixing server-only rules with client-only presentation.
- Requiring presentation objects for headless server execution.

---

## 18. Make Network Authority and Ownership Explicit

### Recommendation

For networked functionality, define authority, Actor ownership, replication direction, and relevancy as part of the
architecture.

### Classification

**Official requirement and recommendation**

### Questions

For every networked feature, establish:

- Which machine may authoritatively change the state?
- Which Actor owns the RPC path?
- Which clients require the resulting state?
- Is the information persistent state or a transient event?
- Can the client predict the result?
- How is prediction reconciled?
- What prevents invalid client requests?
- What happens for late-joining clients?
- What is relevant to each connection?
- What update frequency is actually required?

### Principles

- The server holds authoritative gameplay state in Unreal's standard client-server model.
- Clients request actions rather than declaring authoritative results.
- Replicate state when late joiners or future relevance require the result.
- Use RPCs for transient communication where appropriate.
- Do not replicate data merely because it exists.
- Do not trust client-provided gameplay outcomes.
- Treat ownership and relevancy as functional requirements, not optimizations added later.

### Architectural Consequence

Objects selected for network responsibilities must support the required network semantics.

A plain UObject or Subsystem does not automatically provide Actor replication.

### Sources

- [Networking and Multiplayer in Unreal Engine](https://dev.epicgames.com/documentation/unreal-engine/networking-and-multiplayer-in-unreal-engine)
- [Replicating Actor Components](https://dev.epicgames.com/documentation/unreal-engine/replicating-actor-components-in-unreal-engine)

---

## 19. Expose Intentional C++ APIs to Blueprints

### Recommendation

Design Blueprint exposure as a stable API rather than exposing implementation details indiscriminately.

### Classification

**Official recommendation and Epic pattern**

### Why

Epic recommends combining Blueprint and C++ according to project needs.

A healthy boundary commonly places:

- stable foundations and invariants in C++;
- content configuration and project-specific extension in Blueprints.

### Implementation Guidance

- Expose only properties and operations that content creators need.
- Use clear categories and metadata.
- Prefer semantic functions over exposing internal fields.
- Protect invariants.
- Provide Blueprint events for intentional extension points.
- Use validation for editable data.
- Avoid making every internal function `BlueprintCallable`.
- Avoid Blueprint dependencies that force foundational C++ systems to know concrete content classes.
- Treat Blueprint APIs as production APIs that may need compatibility over time.

### Avoid

- Exposing mutable internal state solely for convenience.
- Requiring designers to reproduce foundational rules in every Blueprint.
- Creating giant Blueprint function libraries as substitutes for ownership.
- Depending on broad global searches to connect systems.

### Sources

- [Balancing Blueprint and C++](https://dev.epicgames.com/documentation/en-us/unreal-engine/balancing-blueprint-and-cplusplus?application_version=4.27)
- [Blueprints Visual Scripting](https://dev.epicgames.com/documentation/unreal-engine/blueprints-visual-scripting-in-unreal-engine)

---

## 20. Use Interfaces and Delegates to Reduce Concrete Coupling

### Recommendation

Use explicit contracts and event communication where systems do not require direct knowledge of concrete
implementations.

### Classification

**Epic language and engine pattern**

### Interfaces

Interfaces are useful when multiple unrelated object types must provide the same capability.

Examples:

- interactable;
- damageable;
- targetable;
- team agent;
- save participant.

Use interfaces for capability contracts, not as unstructured collections of unrelated functions.

### Delegates

Delegates are useful when one system publishes an event and should not own all consumers.

Examples:

- health changed;
- inventory updated;
- match phase changed;
- asynchronous request completed;
- target selected.

### Avoid

- Replacing every direct call with an event.
- Creating event chains whose execution is impossible to trace.
- Binding without clearly managing unbinding and lifetime.
- Broadcasting mutable internal implementation details.
- Using interfaces to hide fundamentally incorrect ownership.

### Principle

> Decoupling should make responsibilities clearer, not make execution invisible.

### Sources

- [Delegates in Unreal Engine](https://dev.epicgames.com/documentation/en-us/unreal-engine/delegates?application_version=4.27)

---

## 21. Avoid Unnecessary Tick

### Recommendation

Do not enable Tick unless the responsibility genuinely requires recurring frame-based execution.

### Classification

**Recurring Epic performance guidance**

### Alternatives

Consider:

- timers;
- delegates;
- latent actions;
- asynchronous completion callbacks;
- animation notifications;
- overlap events;
- state-change events;
- Gameplay Effects;
- scheduled subsystem work;
- lower-frequency updates;
- significance-based updates.

### Apply Tick When

- behavior genuinely changes continuously;
- interpolation requires frame updates;
- movement or simulation semantics require it;
- event-driven alternatives would be more complex or incorrect.

### Implementation Guidance

- Disable Tick by default.
- Enable it only during states that need it.
- Select an appropriate Tick group.
- Avoid repeated object discovery.
- Cache stable references carefully.
- Measure aggregate cost, not just one instance.
- Test worst-case instance counts.

### Avoid

- Empty Tick functions.
- Per-frame casts and global searches without measurement.
- Polling for state that could publish an event.
- Ticking dormant or inactive systems.
- Assuming Blueprint Tick is always unacceptable or C++ Tick is always inexpensive.

---

## 22. Design Initialization as a Lifecycle

### Recommendation

Treat initialization as multiple ordered phases rather than assuming construction means runtime readiness.

### Classification

**Epic engine pattern**

### Relevant Phases May Include

- constructor and default-object construction;
- object creation;
- component registration;
- Actor spawning;
- construction scripts;
- pre-initialization;
- component initialization;
- `BeginPlay`;
- possession;
- player-state availability;
- replicated-property delivery;
- local-player initialization;
- asynchronous asset completion;
- Game Feature activation.

### Why

Networked and modular systems may become ready incrementally.

An object may exist before all dependencies are available.

### Implementation Guidance

- Define explicit readiness requirements.
- Avoid accessing world state from constructors.
- Make initialization idempotent when callbacks may occur more than once.
- Separate dependency discovery from use.
- Handle delayed replicated dependencies.
- Clean up symmetrically.
- Consider explicit initialization states for complex modular features.
- Do not rely on accidental callback ordering.

### Epic Pattern: Initialization States

Epic's Modular Gameplay framework includes initialization-state coordination through the Game Framework Component
Manager. This is useful evidence that complex systems should model readiness explicitly instead of assuming all
dependencies exist immediately.

### Sources

- [Game Framework Component Manager](https://dev.epicgames.com/documentation/unreal-engine/game-framework-component-manager-in-unreal-engine)
- [Objects in Unreal Engine](https://dev.epicgames.com/documentation/unreal-engine/objects-in-unreal-engine)

---

## 23. Use Modules as Compilation and Dependency Boundaries

### Recommendation

Use Unreal modules to define meaningful code and dependency boundaries.

### Classification

**Official engine architecture**

### Why

Modules control:

- compilation boundaries;
- startup and shutdown;
- dependency declarations;
- public and private headers;
- runtime versus editor code;
- code organization;
- API visibility.

### Implementation Guidance

- Declare dependencies explicitly in module rules.
- Keep public APIs small.
- Place implementation details in private headers and source.
- Separate editor-only code from runtime modules.
- Avoid circular module dependencies.
- Organize modules around coherent responsibilities rather than arbitrary folder counts.
- Consider compile-time and reuse consequences.

### Avoid

- Creating many tiny modules without a boundary benefit.
- Putting all project code into one module after responsibilities become independently reusable.
- Making runtime modules depend on editor modules.
- Exposing private implementation through public headers.
- Using modules as replacements for runtime object architecture.

### Sources

- [Unreal Engine Modules](https://dev.epicgames.com/documentation/unreal-engine/unreal-engine-modules)

---

## 24. Use Plugins for Independently Reusable or Optional Features

### Recommendation

Use a Plugin when functionality has an independent product, reuse, distribution, optionality, or ownership boundary.

### Classification

**Epic engine pattern**

### Appropriate Cases

- reusable integrations;
- external SDK wrappers;
- engine extensions;
- optional project features;
- editor tooling;
- functionality shared across projects;
- content and code that should be enabled or disabled together.

### Avoid

- Turning every project folder into a Plugin.
- Using Plugins only to make architecture appear modular.
- Creating Plugin boundaries with uncontrolled dependencies on game-specific code.
- Combining editor and runtime dependencies carelessly.
- Assuming a Plugin automatically produces runtime modularity.

### Principle

> A Plugin is a packaging and dependency boundary; internal architecture still requires clear ownership and
> responsibilities.

---

## 25. Use Game Feature Plugins for Dynamically Activatable Gameplay Features When Justified

### Recommendation

Evaluate Game Feature Plugins when gameplay content and code must be independently registered, activated, deactivated,
or delivered.

### Classification

**Context-dependent Epic pattern**

### Apply When

They may be suitable for:

- modular game modes;
- optional feature sets;
- downloadable gameplay content;
- experiences assembled from feature packages;
- projects following architectures similar to Lyra;
- features that extend existing gameplay objects through supported registration mechanisms.

### Avoid or Reconsider When

Do not adopt Game Feature Plugins solely because Lyra uses them.

They introduce concepts such as:

- feature state;
- activation and deactivation;
- registration actions;
- dependency management;
- initialization coordination;
- dynamic extension of existing gameplay objects.

For small or fixed-scope projects, ordinary modules, Plugins, Components, and assets may be simpler.

### Sources

- [Game Framework Component Manager](https://dev.epicgames.com/documentation/unreal-engine/game-framework-component-manager-in-unreal-engine)

---

## 26. Keep Dependencies Directional

### Recommendation

Organize dependencies so foundational systems do not depend on concrete high-level content.

### Classification

**Inference from module, Blueprint, and sample architecture**

### Preferred Direction

A common dependency direction is:

1. engine and shared foundations;
2. project frameworks and interfaces;
3. gameplay systems;
4. feature implementations;
5. content-specific Blueprints and assets;
6. presentation.

Higher-level content may depend on stable foundations.

Foundations should not require direct references to every content implementation.

### Techniques

- interfaces;
- delegates;
- data assets;
- soft class references;
- Gameplay Tags;
- registries;
- Asset Manager;
- dependency injection through initialization;
- Blueprint extension events;
- modular feature registration.

### Avoid

- Circular module dependencies.
- Foundational C++ loading specific content assets unnecessarily.
- Cross-referencing Blueprint classes in both directions.
- Global registries with uncontrolled write access.
- Casting to concrete classes throughout unrelated systems.

---

## 27. Treat Global Access as a Warning Signal

### Recommendation

Use globally reachable access points only when the responsibility's scope is genuinely global or framework-scoped.

### Classification

**Inference**

### Why

Unreal provides convenient access to:

- Game Instance;
- World;
- Game State;
- Controllers;
- Subsystems;
- Asset Manager;
- static Blueprint libraries.

Convenient retrieval does not prove correct ownership.

### Risks

- Hidden dependencies.
- Difficult testing.
- Incorrect multi-world behavior.
- Stale references.
- Network-authority mistakes.
- Unclear initialization ordering.
- Systems usable only inside a fully running game.

### Guidance

Before using global lookup, ask:

- Is this object actually owned by the caller?
- Should it be passed explicitly?
- Is an interface more accurate?
- Does the selected global object's lifetime match?
- Will this work with multiple worlds and local players?
- Is the access valid on server, client, editor, and test contexts?

---

## 28. Separate Runtime and Editor Architecture

### Recommendation

Keep editor-only tools and dependencies outside runtime modules and runtime object paths.

### Classification

**Official module and build pattern**

### Why

Editor code may depend on systems unavailable in packaged builds.

Mixing these concerns can cause:

- packaging failures;
- unnecessary dependencies;
- increased build time;
- accidental runtime code paths;
- unclear module boundaries.

### Implementation Guidance

- Use editor modules for custom editors, asset actions, details panels, validators, and editor workflows.
- Guard editor-only data and code appropriately.
- Avoid runtime-module dependencies on editor modules.
- Test packaged builds regularly.
- Distinguish editor utility objects from runtime systems.

---

## 29. Validate Architecture in Real Engine Contexts

### Recommendation

Review architecture through runtime tests, not class diagrams alone.

### Classification

**Inference aligned with Epic testing guidance**

### Required Contexts May Include

- standalone;
- Play In Editor;
- multiple Play In Editor clients;
- listen server;
- dedicated server;
- seamless and non-seamless travel;
- respawn and Pawn replacement;
- split-screen;
- packaged development build;
- target hardware;
- save and load;
- hot reload or editor reconstruction where relevant;
- Game Feature activation and deactivation;
- asynchronous loading failure.

### Questions

- Does every system have one clear owner?
- Does it initialize at the correct time?
- Does it clean up?
- Does it survive only as long as intended?
- Does it function without presentation?
- Does it function on a dedicated server?
- Does replicated state reach all required clients?
- Can late joiners reconstruct the state?
- Are assets loaded deliberately?
- Are Blueprint APIs stable and constrained?
- Does the design support the expected variants?
- Are dependencies directional and understandable?

---

## Architecture Selection Guide

| Requirement                                | Preferred Starting Point         |
|--------------------------------------------|----------------------------------|
| Reflected value data                       | `USTRUCT`                        |
| Non-reflected internal implementation      | Plain C++ type                   |
| Reflected non-spatial runtime object       | `UObject`                        |
| Authored reusable definition               | Data Asset                       |
| Actor-owned non-spatial capability         | `UActorComponent`                |
| Actor-owned capability requiring transform | `USceneComponent`                |
| Renderable or collidable representation    | `UPrimitiveComponent` derivative |
| Independent world entity                   | `AActor`                         |
| Possessable entity                         | `APawn`                          |
| Humanoid Character Movement conventions    | `ACharacter`                     |
| Player control connection                  | `APlayerController`              |
| Replicated participant state               | `APlayerState`                   |
| Server-only match rules                    | `AGameModeBase` / `AGameMode`    |
| Replicated shared match state              | `AGameStateBase` / `AGameState`  |
| Map-persistent process-local context       | `UGameInstance`                  |
| Engine-lifetime service                    | `UEngineSubsystem`               |
| Game-instance-lifetime service             | `UGameInstanceSubsystem`         |
| World-lifetime service                     | `UWorldSubsystem`                |
| Local-player-lifetime service              | `ULocalPlayerSubsystem`          |
| Compilation and dependency boundary        | Module                           |
| Reusable or optional packaged feature      | Plugin                           |
| Dynamically activatable feature package    | Game Feature Plugin              |
| Managed asset identity and loading         | Primary Asset and Asset Manager  |

This table provides starting points, not automatic answers.

---

## Architecture Review Checklist

### Responsibility

- [ ] The responsibility can be explained in one concise statement.
- [ ] The class does not combine unrelated rules, state, commands, and presentation.
- [ ] The responsibility is located in an Unreal type whose semantics match it.

### Ownership

- [ ] The conceptual owner is explicit.
- [ ] Creation and destruction responsibility are explicit.
- [ ] References do not accidentally extend lifetime.
- [ ] No global object is used only to avoid passing a dependency.

### Lifetime

- [ ] The required lifetime is documented.
- [ ] Pawn replacement is handled where relevant.
- [ ] Map travel is handled where relevant.
- [ ] Multiple worlds and Play In Editor contexts are considered.
- [ ] Initialization and cleanup are symmetrical.

### World Participation

- [ ] The system is an Actor only if it needs independent world identity.
- [ ] It is a Scene Component only if it needs a transform.
- [ ] It is a Primitive Component only if it needs scene representation.
- [ ] Actor and Component Tick are disabled unless needed.

### Gameplay Framework

- [ ] Server-only rules are not placed in client-dependent objects.
- [ ] Shared replicated state is not kept only in GameMode.
- [ ] Player identity is not tied unnecessarily to one Pawn.
- [ ] PlayerController and Pawn responsibilities are separated.
- [ ] Game Instance is not used as a universal global manager.

### Networking

- [ ] Authority is explicit.
- [ ] Actor ownership is explicit.
- [ ] Replication targets are explicit.
- [ ] RPC direction and validation are explicit.
- [ ] Late joining is considered.
- [ ] Relevancy and update frequency are considered.
- [ ] Local-only objects are not assumed to replicate.

### Data and Assets

- [ ] Authored definitions are separate from mutable runtime state.
- [ ] Hard and soft references are intentional.
- [ ] Asset loading behavior is understood.
- [ ] Shared assets are not mutated as per-instance state.
- [ ] Primary Assets are used when managed identity and loading are required.

### Dependencies

- [ ] Dependencies flow in a clear direction.
- [ ] Foundational code does not depend unnecessarily on concrete content.
- [ ] Module dependencies are explicit.
- [ ] Runtime modules do not depend on editor modules.
- [ ] Interfaces and delegates improve clarity rather than obscure execution.

### Blueprint and C++

- [ ] Blueprint exposure is intentional.
- [ ] Important invariants remain protected.
- [ ] Designers can configure intended variations.
- [ ] Foundational behavior is not repeatedly duplicated in Blueprints.
- [ ] Blueprint APIs have meaningful names, categories, and validation.

### Validation

- [ ] The system has been tested in the required runtime contexts.
- [ ] Destruction and recreation have been tested.
- [ ] Travel has been tested where applicable.
- [ ] Multiplayer roles have been tested where applicable.
- [ ] Packaged builds have been tested.
- [ ] Performance has been measured at expected scale.

---

## Common Architectural Failure Modes

### God Character

The Character owns movement, inventory, attributes, interaction, combat, targeting, UI state, audio, and every gameplay
rule.

**Correction:** Separate persistent capabilities, gameplay actions, state, and presentation while keeping the Character
as an integration point.

### God Game Instance

Game Instance becomes a global container for every manager and variable.

**Correction:** Select Game Instance, World, Local Player, Actor, or asset scope according to actual lifetime and
ownership.

### Manager Actor Without World Semantics

An invisible Actor is spawned only to host globally accessible logic.

**Correction:** Evaluate UObject, Subsystem, framework class, or explicit ownership.

### Component Explosion

Every small function becomes a Component, producing many interdependent classes with unclear boundaries.

**Correction:** Extract coherent capabilities, not arbitrary pieces of code.

### Deep Inheritance Tree

Optional and independent features are encoded through increasingly specialized base classes.

**Correction:** Use composition for capabilities that vary independently.

### Asset as Runtime State

A shared Data Asset is modified to represent one player's mutable state.

**Correction:** Keep the asset as a definition and create a runtime instance or state container.

### Replication as an Afterthought

Single-player assumptions are embedded throughout the design and replication is added mechanically later.

**Correction:** When multiplayer is a real requirement, model authority, ownership, state, events, relevancy, and
prediction from the beginning.

### Premature Multiplayer Architecture

A small single-player prototype implements server authority, prediction, and replicated abstractions without a
multiplayer requirement.

**Correction:** Design for the real project while avoiding decisions that make future change unnecessarily difficult.

### Hidden Blueprint Dependencies

Blueprints directly cast to and load one another across many systems.

**Correction:** Create stable C++ or interface boundaries, intentional events, data-driven references, and controlled
loading.

### Subsystem as Singleton by Habit

Every service becomes a Subsystem because it is easy to retrieve.

**Correction:** Use Subsystems only when their managed lifetime matches the responsibility.

---

## Core Conclusions

Epic's object model does not prescribe one universal project architecture.

It provides types with different semantics.

The strongest architecture is the one that deliberately aligns:

- responsibility;
- ownership;
- lifetime;
- authority;
- replication;
- world participation;
- data loading;
- dependency direction;
- extension needs.

The most reusable decision rule is:

> First determine what owns the responsibility, how long it must live, where it must exist, and who may modify it. Then
> choose the Unreal abstraction that naturally provides those semantics.

---

## Primary Sources

- [Programming in the Unreal Engine Architecture](https://dev.epicgames.com/documentation/unreal-engine/programming-in-the-unreal-engine-architecture)
- [Gameplay Framework in Unreal Engine](https://dev.epicgames.com/documentation/unreal-engine/gameplay-framework-in-unreal-engine)
- [Gameplay Framework Quick Reference](https://dev.epicgames.com/documentation/unreal-engine/gameplay-framework-quick-reference-in-unreal-engine)
- [Actors in Unreal Engine](https://dev.epicgames.com/documentation/unreal-engine/actors-in-unreal-engine)
- [Components in Unreal Engine](https://dev.epicgames.com/documentation/unreal-engine/components-in-unreal-engine)
- [Objects in Unreal Engine](https://dev.epicgames.com/documentation/unreal-engine/objects-in-unreal-engine)
- [Unreal Object Handling](https://dev.epicgames.com/documentation/unreal-engine/unreal-object-handling-in-unreal-engine)
- [Programming Subsystems](https://dev.epicgames.com/documentation/unreal-engine/programming-subsystems-in-unreal-engine)
- [Unreal Engine Modules](https://dev.epicgames.com/documentation/unreal-engine/unreal-engine-modules)
- [Asset Management](https://dev.epicgames.com/documentation/unreal-engine/asset-management-in-unreal-engine)
- [Data Assets](https://dev.epicgames.com/documentation/unreal-engine/data-assets-in-unreal-engine)
- [Asynchronous Asset Loading](https://dev.epicgames.com/documentation/unreal-engine/asynchronous-asset-loading-in-unreal-engine)
- [Replicating Actor Components](https://dev.epicgames.com/documentation/unreal-engine/replicating-actor-components-in-unreal-engine)
- [Game Framework Component Manager](https://dev.epicgames.com/documentation/unreal-engine/game-framework-component-manager-in-unreal-engine)