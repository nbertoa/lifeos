# C++ Architecture in Unreal Engine

## Research Metadata

- **Research date:** 2026-07-22
- **Documentation baseline:** Unreal Engine 5.8
- **Primary authority:** Epic Games documentation and Epic C++ Coding Standard
- **Scope:** Native-code responsibility, dependency structure, modules, reflection boundaries, ownership, composition, interfaces, Blueprint integration, lifecycle, networking, and architectural validation
- **Status:** Living document

---

## Purpose

C++ architecture in Unreal Engine is not independent from the engine's object model.

Native systems operate inside an architecture shaped by:

- modules and plugins;
- Unreal Build Tool;
- the reflection system;
- `UObject` ownership and garbage collection;
- Actors and Components;
- Gameplay Framework lifetimes and authority;
- Subsystems;
- asset references and loading;
- Blueprint extension;
- replication;
- Editor and cooked-build constraints.

Standard C++ design principles remain useful, but they must be applied through Unreal's runtime, build, serialization, and content systems.

The central principle is:

> Organize native code around explicit responsibilities, ownership, lifetime, dependency direction, and engine integration—not merely around convenient class access.

---

## 1. Design With Unreal's Architecture, Not Around It

### Recommendation

Select Unreal types and extension mechanisms according to the engine responsibility they represent.

### Classification

**Official Epic recommendation**

### Why

Epic describes the Gameplay Framework as a modular foundation whose classes are designed to work together. Unreal also provides specialized mechanisms for reflected objects, world objects, reusable Actor capabilities, managed-lifetime services, modules, and plugins.

Using an arbitrary globally accessible class where an Unreal lifecycle type is required can bypass:

- world context;
- replication;
- garbage collection;
- serialization;
- Editor integration;
- automatic initialization and shutdown;
- Blueprint exposure;
- asset loading rules.

### Apply When

Before creating a class, determine whether the responsibility is best represented as:

- a plain C++ type;
- a `UObject`;
- an `AActor`;
- a `UActorComponent` or `USceneComponent`;
- a Gameplay Framework class;
- a Subsystem;
- a module-level service;
- a plugin;
- a data asset or other data representation.

### Avoid

Avoid beginning with a preferred class type and forcing the responsibility into it.

### Sources

- [Programming with C++](https://dev.epicgames.com/documentation/unreal-engine/programming-with-cplusplus-in-unreal-engine)
- [Gameplay Framework](https://dev.epicgames.com/documentation/unreal-engine/gameplay-framework-in-unreal-engine)
- [Programming Subsystems](https://dev.epicgames.com/documentation/unreal-engine/programming-subsystems-in-unreal-engine)

---

## 2. Assign Every Responsibility an Explicit Owner

### Recommendation

Place state and behavior on the object whose lifetime, authority, and semantic responsibility match them.

### Classification

**Inference supported by Epic's framework and lifecycle documentation**

### Why

Many Unreal architecture failures are ownership failures rather than syntax failures.

A responsibility should have clear answers to:

- Who creates it?
- Who destroys it?
- Who may mutate it?
- Who observes it?
- Does it exist once per engine, game instance, world, local player, player, Actor, or component instance?
- Does it exist on the server, clients, Editor, or all of them?
- Does it survive map travel?
- Does it replicate?
- Does it own assets or only reference them?

### Apply When

Make ownership decisions before exposing accessors or adding global lookup paths.

### Avoid

Avoid storing state in an object merely because it is easy to retrieve from many places.

Avoid using `UGameInstance` as a general dumping ground for unrelated systems solely because it survives map loads.

### Validation

For each major system, document:

```text
Owner:
Lifetime:
Authority:
Creation:
Shutdown:
Mutation policy:
Replication policy:
Asset-loading policy:
```

---

## 3. Prefer Cohesive Responsibilities

### Recommendation

Keep classes and modules focused on cohesive responsibilities and split methods or types when they accumulate unrelated reasons to change.

### Classification

**Official Epic coding-standard guidance combined with general engineering inference**

### Why

Epic's coding standard directs developers to split methods into sub-methods where possible and emphasizes maintainability, readability, encapsulation, and minimizing dependency distance.

In Unreal, oversized classes often become especially costly because they also accumulate:

- reflected APIs;
- serialized properties;
- Blueprint dependencies;
- replication state;
- lifecycle callbacks;
- asset references;
- component coordination;
- Editor-facing configuration.

### Apply When

Split a responsibility when it has distinct:

- ownership;
- lifetime;
- authority;
- data model;
- test boundary;
- module dependency;
- content-authoring workflow.

### Avoid or Reconsider When

Do not fragment one cohesive behavior into many indirections solely to make classes smaller.

### Sources

- [Epic C++ Coding Standard](https://dev.epicgames.com/documentation/unreal-engine/epic-cplusplus-coding-standard-for-unreal-engine)

---

## 4. Enforce Encapsulation

### Recommendation

Keep class members private unless they are intentionally part of the public or protected contract.

### Classification

**Official Epic requirement/recommendation**

### Epic Direction

Epic's coding standard states that class members should almost always be private unless they belong to the public or protected interface. When derived classes need access, Epic recommends private storage with protected accessors.

Epic also recommends using `final` when a class is not designed for inheritance.

### Why

Encapsulation preserves the ability to:

- validate mutation;
- add side effects;
- change representation;
- protect invariants;
- control replication-aware updates;
- refactor without breaking consumers;
- narrow Blueprint exposure;
- prevent subclasses from depending on storage details.

### Apply When

Use:

- private data by default;
- narrow public operations;
- read-only accessors for observation;
- protected extension APIs only where subclass customization is intentional;
- `final` for non-extensible types or overrides.

### Avoid

Avoid public mutable fields as a substitute for API design.

Avoid protected data solely to make subclass implementation convenient.

### Sources

- [Epic C++ Coding Standard](https://dev.epicgames.com/documentation/unreal-engine/epic-cplusplus-coding-standard-for-unreal-engine)

---

## 5. Separate Public Contract From Private Implementation

### Recommendation

Expose the smallest stable contract required by consumers and keep implementation-only types and dependencies private.

### Classification

**Epic pattern and module-system guidance**

### Why

Unreal modules can enforce separation and hide internal implementation. A narrow public surface reduces:

- compile dependencies;
- transitive includes;
- ABI exposure;
- Blueprint-facing complexity;
- plugin coupling;
- migration cost.

### Apply When

For each public declaration, ask:

- Does another module actually need this type?
- Can the dependency be represented through an interface or forward declaration?
- Is the type an implementation detail?
- Is the reflected declaration intentionally part of a content contract?
- Will changing it force unrelated recompilation or asset migration?

### Avoid

Avoid placing every header in `Public`.

Avoid exporting internal helper types merely to solve an include problem.

### Validation

A consumer module should compile using only the documented public headers and declared module dependencies.

### Sources

- [Unreal Engine Modules](https://dev.epicgames.com/documentation/unreal-engine/unreal-engine-modules)
- [Gameplay Modules](https://dev.epicgames.com/documentation/unreal-engine/gameplay-modules-in-unreal-engine)

---

## 6. Organize Code Into Modules Around Stable Boundaries

### Recommendation

Use modules to encapsulate meaningful runtime features, editor tools, or libraries when the separation provides concrete architectural or build value.

### Classification

**Official Epic recommendation with context-dependent tradeoffs**

### Epic Direction

Epic identifies modules as the basic building blocks of Unreal's software architecture. Modules encapsulate functionality and compile as separate units.

Epic also warns that splitting gameplay into many DLLs may create more trouble than it is worth. Multiple gameplay modules can improve link times and code iteration, but increase export, interface, and dependency-management work.

### Apply When

A module boundary is useful when a capability:

- has a coherent public API;
- can hide meaningful private implementation;
- has a distinct runtime/editor dependency set;
- is reusable;
- needs independent loading behavior;
- benefits from reduced compilation scope;
- may become a plugin;
- should be excluded from some targets.

### Avoid or Reconsider When

Do not create a module for every folder or conceptual noun.

Avoid modules that:

- are mutually dependent;
- expose most of their implementation;
- contain only trivial forwarding code;
- require constant cross-module changes;
- exist without a stable responsibility boundary.

### Validation

Inspect the module dependency graph for cycles and unnecessary public dependencies.

### Sources

- [Unreal Engine Modules](https://dev.epicgames.com/documentation/unreal-engine/unreal-engine-modules)
- [Gameplay Modules](https://dev.epicgames.com/documentation/unreal-engine/gameplay-modules-in-unreal-engine)
- [Unreal Engine C++ API Reference](https://dev.epicgames.com/documentation/unreal-engine/API)

---

## 7. Keep Module Dependencies Acyclic Where Practical

### Recommendation

Prefer one-directional module dependencies and avoid cross-dependent gameplay modules.

### Classification

**Official Epic guidance**

### Why

Epic states that cross-dependent modules are supported but not ideal. They can harm compile times and cause static-initialization problems.

Cycles also indicate that:

- responsibilities may be incorrectly divided;
- shared contracts are in the wrong module;
- implementation details are leaking;
- ownership direction is unclear.

### Apply When

Resolve cycles by considering:

- moving shared contracts into a lower-level module;
- dependency inversion through interfaces;
- delegates or messages;
- a coordinating higher-level module;
- data-based communication;
- combining modules when the separation is artificial.

### Avoid

Avoid creating a generic shared module that becomes an ungoverned dumping ground.

### Sources

- [Gameplay Modules](https://dev.epicgames.com/documentation/unreal-engine/gameplay-modules-in-unreal-engine)

---

## 8. Separate Runtime and Editor Dependencies

### Recommendation

Keep editor-only functionality out of runtime modules and shipping dependency paths.

### Classification

**Official Unreal build architecture requirement and Epic pattern**

### Why

Runtime code may be compiled for targets where Editor modules and APIs are unavailable. Mixing editor-only dependencies into runtime code can:

- break non-editor targets;
- increase build size and compile time;
- make plugins incorrectly loadable;
- introduce unavailable UObject classes or assets in cooked builds.

### Apply When

Use separate editor modules for:

- custom asset editors;
- details customizations;
- editor commands;
- placement tools;
- content-generation utilities;
- validation UI;
- editor-only import workflows.

Keep runtime-facing contracts in runtime modules when both sides require them.

### Avoid

Avoid relying only on preprocessor guards when the dependency itself belongs in a separate module.

### Validation

Compile and package non-editor targets regularly.

---

## 9. Use the Reflection System Intentionally

### Recommendation

Add reflection only when an Unreal Engine system requires or benefits from it.

### Classification

**Official Epic guidance**

### Why

Epic documents that standard C++ classes, functions, and variables may be used normally. Reflection macros make Unreal aware of declarations and enable capabilities such as:

- garbage-collection tracking;
- serialization;
- Editor exposure;
- Blueprint exposure;
- replication;
- metadata-driven tooling;
- runtime type information within Unreal's type system.

Reflection also expands the contract and build complexity of a declaration.

### Apply When

Use `UCLASS`, `USTRUCT`, `UENUM`, `UFUNCTION`, and `UPROPERTY` when the declaration participates in a relevant engine system.

Use plain C++ for implementation details that do not require those capabilities.

### Avoid

Avoid reflecting every helper type by default.

Avoid omitting required reflection from UObject references that must be tracked by garbage collection.

### Sources

- [Programming with C++](https://dev.epicgames.com/documentation/unreal-engine/programming-with-cplusplus-in-unreal-engine)
- [Gameplay Architecture](https://dev.epicgames.com/documentation/unreal-engine/programming-with-cpp-in-unreal-engine)

---

## 10. Choose Plain C++ Types for Non-Reflected Implementation Details

### Recommendation

Use plain C++ types for algorithms, private value objects, policies, and implementation details that do not need Unreal reflection or UObject lifetime behavior.

### Classification

**Inference supported by Epic's explicit support for standard C++ types**

### Why

Plain types can provide:

- simpler construction and destruction;
- fewer reflection constraints;
- easier isolated tests;
- no garbage-collection participation;
- lower conceptual overhead;
- clearer value semantics.

### Apply When

Plain C++ is often suitable for:

- private algorithms;
- immutable or value-style domain types;
- calculation helpers;
- internal strategies;
- parsers;
- test doubles;
- non-serialized caches.

### Avoid or Reconsider When

Use a reflected Unreal type when the object must participate in:

- Editor or Blueprint workflows;
- serialization;
- replication;
- garbage-collected references;
- Unreal object identity;
- asset references;
- engine-managed instancing.

---

## 11. Prefer Composition for Reusable Actor Capabilities

### Recommendation

Use Actor Components for reusable capabilities that belong to Actors and benefit from Actor ownership, lifecycle, replication, or Editor composition.

### Classification

**Official Epic framework pattern**

### Why

Epic describes Actor Components as the building blocks of Actors. Components allow capabilities to be reused without forcing unrelated Actors into a deep inheritance hierarchy.

### Apply When

A Component is suitable when a capability:

- belongs to an Actor;
- has per-Actor state;
- follows Actor lifecycle;
- may replicate with its owner;
- should be composed in the Editor or Blueprint;
- is reusable across different Actor classes.

### Avoid or Reconsider When

Do not use a Component when the responsibility:

- does not belong to an Actor;
- is stateless utility behavior;
- exists once per world, game instance, or local player;
- has no meaningful component lifecycle;
- is only a workaround for poor class decomposition.

### Sources

- [Gameplay Framework](https://dev.epicgames.com/documentation/unreal-engine/gameplay-framework-in-unreal-engine)

---

## 12. Use Inheritance Only for Genuine Substitutability and Extension

### Recommendation

Create an inheritance relationship only when the derived type is intentionally substitutable for the base and the base defines a stable extension contract.

### Classification

**Inference combined with Epic encapsulation guidance**

### Why

Inheritance creates coupling to:

- base lifecycle;
- protected API;
- virtual contracts;
- reflected class hierarchy;
- serialized defaults;
- Blueprint subclass behavior.

Deep hierarchies can make initialization and state ownership difficult to reason about.

### Apply When

Inheritance is appropriate when:

- Unreal requires a framework base type;
- consumers operate on the base abstraction;
- virtual extension is intentional;
- derived types preserve the base contract;
- common state truly belongs to every derived instance.

### Avoid

Avoid inheritance solely to reuse implementation.

Prefer composition, helper types, or delegated policies when the relationship is capability reuse rather than semantic specialization.

---

## 13. Design Virtual Functions as Explicit Contracts

### Recommendation

Make functions virtual only when replacement or extension by subclasses is intentionally supported.

### Classification

**Epic coding-standard pattern and inference**

### Why

A virtual function is an architectural extension point. It increases the behaviors that subclasses may alter and can weaken invariants when its contract is unclear.

### Apply When

For each virtual function, define:

- whether overriding is optional or required;
- whether the parent implementation must be called;
- valid lifecycle phases;
- authority expectations;
- allowed side effects;
- thread assumptions;
- failure behavior.

Use `final` for classes or overrides that are not designed for further derivation.

### Avoid

Avoid virtualizing functions preemptively.

Avoid placing mandatory invariant-preserving work only in an overrideable function when subclasses can accidentally omit it.

### Sources

- [Epic C++ Coding Standard](https://dev.epicgames.com/documentation/unreal-engine/epic-cplusplus-coding-standard-for-unreal-engine)

---

## 14. Use Interfaces for Shared Capabilities Without Shared Ownership

### Recommendation

Use interfaces when unrelated Unreal types must expose a common capability without sharing implementation state or a concrete base class.

### Classification

**Official Unreal feature guidance with context-dependent use**

### Why

Interfaces can reduce dependence on concrete types and support communication across otherwise unrelated classes.

### Apply When

An interface is appropriate when:

- several unrelated classes provide the same semantic capability;
- callers should depend on behavior rather than implementation;
- no shared storage is required;
- the capability may be implemented in C++ or Blueprint, according to the chosen interface design.

### Avoid or Reconsider When

Do not create an interface for every class boundary.

An interface may be unnecessary when:

- there is only one stable implementation;
- direct ownership is clearer;
- a delegate better represents notification;
- the behavior belongs to a Component;
- data is the real shared contract.

---

## 15. Use Delegates for Notification, Not Hidden Command Ownership

### Recommendation

Use delegates when one object publishes an event and does not need concrete knowledge of its observers.

### Classification

**Official Epic feature guidance and inference**

### Why

Epic describes delegates as type-safe references to functions that can be called later without the caller knowing the receiver's concrete type.

Delegates are useful for decoupled notification, but they can obscure control flow if used as a general command bus.

### Apply When

Use delegates for:

- state-change notification;
- completion callbacks;
- optional observation;
- UI reactions;
- content-facing events;
- one-to-many broadcasts.

### Avoid

Avoid delegates when:

- exactly one owner must perform the operation;
- the caller requires a return value from a clear service;
- execution order is critical but unspecified;
- the event replaces an explicit dependency;
- lifetime and unbinding responsibilities are unclear.

### Validation

Document who binds, who unbinds, when broadcast is legal, and whether listeners may mutate the publisher.

### Sources

- [Programming with C++](https://dev.epicgames.com/documentation/unreal-engine/programming-with-cplusplus-in-unreal-engine)

---

## 16. Use Subsystems for Managed-Lifetime Services

### Recommendation

Use a Subsystem when a service naturally belongs to one of Unreal's supported managed lifetimes and does not need a physical world presence.

### Classification

**Official Epic recommendation**

### Epic Direction

Epic describes Subsystems as automatically instanced classes with managed lifetimes. Epic lists benefits including modularity, avoiding unnecessary engine-class overrides, and avoiding adding APIs to already busy classes.

### Apply When

Choose a Subsystem according to ownership:

- `UEngineSubsystem`: engine lifetime;
- `UEditorSubsystem`: editor lifetime;
- `UGameInstanceSubsystem`: game-instance lifetime;
- `UWorldSubsystem`: world lifetime;
- `ULocalPlayerSubsystem`: local-player lifetime.

### Avoid

Do not use a Subsystem merely to obtain global-style access.

Avoid a Subsystem when the service:

- belongs to one Actor;
- needs transform or world representation;
- should replicate as an Actor;
- has a shorter explicit owner;
- is stateless utility code;
- should be configured as content data.

### Validation

Verify initialization, deinitialization, world type, multiplayer multiplicity, map travel, and PIE behavior.

### Sources

- [Programming Subsystems](https://dev.epicgames.com/documentation/unreal-engine/programming-subsystems-in-unreal-engine)

---

## 17. Avoid Convenience Globals That Hide Context

### Recommendation

Prefer explicit context and ownership over global access patterns that conceal which world, player, or runtime instance is being used.

### Classification

**Inference**

### Why

Unreal may run multiple worlds or instances in:

- PIE;
- multiplayer;
- Editor previews;
- automation tests;
- server travel;
- commandlets.

A global accessor can silently choose the wrong context and makes dependencies harder to test.

### Apply When

Pass or derive the narrowest valid context:

- owning object;
- world context object;
- game instance;
- local player;
- Actor owner;
- explicit service interface.

### Avoid or Reconsider When

Engine-level globals are appropriate only for responsibilities that genuinely have engine-wide semantics and whose lifecycle permits access.

---

## 18. Make Lifetime Assumptions Explicit

### Recommendation

Design APIs around the actual lifetime guarantees of their objects and references.

### Classification

**Official Unreal object-model requirement and inference**

### Why

Unreal objects may be:

- class-default objects;
- editor preview instances;
- pending destruction;
- garbage collected;
- unloaded assets;
- replicated later than their consumers expect;
- recreated across map travel;
- absent on a client or server.

### Apply When

For every stored reference, define:

- ownership or observation;
- strong, weak, soft, or raw-reference semantics;
- whether garbage collection must track it;
- whether it may be unloaded;
- when validity must be checked;
- whether it is safe across async work or latent callbacks.

### Avoid

Avoid treating pointer non-nullness as proof that the full semantic precondition is satisfied.

Avoid storing world-owned objects in longer-lived owners without explicit reset behavior.

---

## 19. Separate Configuration, Runtime State, and Derived State

### Recommendation

Represent authored configuration, mutable runtime state, and computed or cached state as distinct concepts.

### Classification

**Inference supported by Unreal's serialization and content workflows**

### Why

These categories have different requirements:

- Configuration is authored, versioned, and often serialized in assets or defaults.
- Runtime state changes during play and may replicate or persist.
- Derived state is recalculated or cached and may not belong in serialization.

Mixing them can produce:

- invalid save data;
- accidental default mutation;
- stale caches;
- unnecessary replication;
- hidden dependencies on construction order.

### Apply When

Use clear naming, access policy, and serialization specifiers to communicate each category.

### Avoid

Avoid using editable properties as both authored configuration and mutable authoritative runtime state without an explicit policy.

---

## 20. Prefer Data-Driven Variation Over Class Proliferation

### Recommendation

Use data for variations that differ primarily by values rather than behavior.

### Classification

**Epic pattern and context-dependent guidance**

### Why

Creating a subclass for every value combination can generate:

- deep content hierarchies;
- duplicated defaults;
- difficult bulk edits;
- class-loading dependencies;
- unclear distinction between behavior and configuration.

### Apply When

Use data assets, tables, curves, config, tags, or structured properties when variants share behavior and differ in authored values.

Use subclasses when variants genuinely replace or extend behavior through a meaningful type contract.

### Avoid

Avoid converting every behavior difference into data-driven conditionals inside one universal class.

Data-driven design still requires validation and clear schemas.

---

## 21. Keep Blueprint as an Intentional Extension Layer

### Recommendation

Expose stable native contracts to Blueprint rather than exposing implementation details indiscriminately.

### Classification

**Official Epic recommendation**

### Why

Epic recommends using C++ as a foundation and building Blueprint classes on top for many mixed projects.

A designed boundary allows:

- programmers to protect invariants;
- designers to iterate on content;
- complex or high-frequency work to remain native;
- Blueprint graphs to operate at a semantic level;
- native implementation to evolve behind a stable contract.

### Apply When

Expose:

- configuration schemas;
- semantic operations;
- read-only observable state;
- intentional events and override hooks;
- focused utility nodes.

### Avoid

Avoid exposing every native member as `BlueprintReadWrite` or `BlueprintCallable`.

Avoid foundational native code depending directly on every concrete Blueprint child.

### Sources

- [Coding in Unreal Engine: Blueprint vs. C++](https://dev.epicgames.com/documentation/unreal-engine/coding-in-unreal-engine-blueprint-vs-cplusplus)
- [Exposing C++ to Blueprints](https://dev.epicgames.com/documentation/unreal-engine/exposing-cplusplus-to-blueprints-visual-scripting-in-unreal-engine)

---

## 22. Preserve Mandatory Native Invariants

### Recommendation

Keep required validation and state transitions in non-optional native code, with narrower extension hooks around them.

### Classification

**Inference**

### Why

Subclass and Blueprint overrides may omit parent calls or perform actions in an unexpected order.

### Preferred Pattern

```cpp
void UEquipmentComponent::EquipItem(const FEquipmentRequest& Request)
{
    if (!CanEquip(Request))
    {
        return;
    }

    CommitEquipmentChange(Request);
    OnItemEquipped(Request);
}
```

The public operation preserves required behavior. `OnItemEquipped` is the optional customization point.

### Apply When

Use this structure when:

- authority must be checked;
- state changes must be atomic;
- replication or notification must occur;
- cleanup is mandatory;
- a Blueprint override is optional.

### Avoid

Avoid placing core correctness entirely in an override that consumers may replace or forget to call.

---

## 23. Keep Networking Responsibilities Explicit

### Recommendation

Place authoritative state and network operations according to Unreal's server-authoritative model and ownership rules.

### Classification

**Official Unreal networking requirement**

### Why

Architecture that ignores authority often works in standalone play but fails in multiplayer.

### Apply When

For every networked responsibility, define:

- authoritative owner;
- replicated state;
- RPC caller and receiver;
- Actor ownership requirement;
- relevancy;
- prediction policy;
- cosmetic versus gameplay behavior;
- late-join behavior.

### Avoid

Avoid allowing client-side presentation code to become the source of authoritative gameplay state.

Avoid using RPCs as a substitute for a coherent replicated state model.

---

## 24. Avoid Hidden Asset-Loading Dependencies

### Recommendation

Design native dependencies with awareness of hard references, soft references, asset loading, cooking, and package inclusion.

### Classification

**Official Unreal asset-system guidance combined with inference**

### Why

A class reference or reflected property can cause an asset and its dependency chain to load or cook. Architectural coupling therefore affects memory, startup, transitions, and package size.

### Apply When

Decide explicitly whether a dependency should be:

- a hard object/class reference;
- a soft object/class reference;
- a Primary Asset identifier;
- loaded synchronously;
- loaded asynchronously;
- injected by configuration;
- discovered through Asset Manager rules.

### Avoid

Avoid native constructors or globally initialized code that forces content loading without an explicit requirement.

Avoid direct references from foundational systems to large content hierarchies when deferred loading is intended.

---

## 25. Minimize Header and Build Dependencies

### Recommendation

Keep include and module dependencies as narrow as practical.

### Classification

**Epic pattern and build-system guidance**

### Why

Unnecessary dependencies increase:

- compile time;
- rebuild scope;
- coupling;
- public API leakage;
- risk of circular dependencies.

### Apply When

Use:

- forward declarations where valid;
- private dependencies for implementation-only use;
- focused headers;
- implementation files for expensive includes;
- narrow public module interfaces.

### Avoid

Avoid broad umbrella headers in public APIs unless they are intentionally stable aggregation points.

Avoid moving dependencies to public scope solely to fix a local compile error.

### Validation

Clean-build modules after dependency changes and test non-unity builds where practical to reveal missing direct includes.

---

## 26. Avoid Static Initialization That Depends on Engine State

### Recommendation

Do not rely on C++ static initialization order for behavior that requires modules, UObjects, worlds, assets, or other engine systems to be ready.

### Classification

**Official Epic module warning combined with inference**

### Why

Epic notes that cross-dependent modules can cause problems with static initialization. More broadly, native static initialization occurs outside many Unreal lifecycle guarantees.

### Apply When

Initialize engine-dependent systems through explicit lifecycle points such as:

- module startup;
- Subsystem initialization;
- object initialization;
- world lifecycle delegates;
- Begin Play where appropriate.

### Avoid

Avoid static constructors that:

- load assets;
- access worlds;
- register against unavailable systems;
- assume module order;
- create UObjects without a supported lifecycle.

### Sources

- [Gameplay Modules](https://dev.epicgames.com/documentation/unreal-engine/gameplay-modules-in-unreal-engine)

---

## 27. Make Initialization and Shutdown Symmetric

### Recommendation

For every registration, binding, allocation, or external relationship, define the corresponding unregistration, unbinding, release, or shutdown behavior.

### Classification

**Inference supported by Unreal lifecycle systems**

### Why

Unreal Editor sessions, PIE, hot reload/live coding, world teardown, module unload, and travel can expose lifecycle leaks that remain hidden in a single standalone run.

### Apply When

Pair:

- delegate bind/unbind;
- module startup/shutdown;
- Subsystem initialize/deinitialize;
- registration/unregistration;
- async request/cancellation;
- resource acquisition/release.

### Avoid

Avoid assuming process exit is the only shutdown path.

### Validation

Run repeated PIE sessions, map travel, module/plugin reload where supported, and automated tests that create and destroy the system multiple times.

---

## 28. Design for Testability Through Explicit Dependencies

### Recommendation

Make important dependencies explicit enough that system behavior can be exercised without relying on uncontrolled global state or full game startup.

### Classification

**Inference**

### Why

Explicit dependencies improve:

- deterministic tests;
- replacement with test implementations;
- understanding of side effects;
- isolation from unrelated worlds or services;
- failure-path testing.

### Apply When

Consider:

- constructor or initializer input for plain C++ helpers;
- interfaces;
- explicit service access passed from an owner;
- data-driven inputs;
- test-specific worlds or fixtures;
- small pure calculation types.

### Avoid

Avoid adding abstraction layers solely to enable hypothetical tests.

Focus on dependencies that are expensive, variable, stateful, or external to the responsibility under test.

---

## 29. Use Defensive Programming According to Contract

### Recommendation

Distinguish programmer errors, violated invariants, recoverable runtime conditions, and external invalid input.

### Classification

**Official Unreal facilities with context-dependent architectural guidance**

### Why

Architecture should define whether a failure means:

- execution cannot correctly continue;
- a development invariant was violated;
- the condition should be reported once but runtime may continue;
- the caller supplied a normal invalid request;
- an asset or network input requires validation.

### Apply When

Use the appropriate mechanism:

- compile-time constraints and `static_assert`;
- `check`/`checkf` for non-recoverable development invariants;
- `ensure`/`ensureMsgf` for reportable unexpected conditions when continuation is possible;
- explicit return values for expected failure;
- logs with actionable context;
- early returns to stop invalid execution paths;
- Editor validation for authoring errors.

### Avoid

Avoid using assertions as routine runtime input validation.

Avoid logging an invariant violation and then continuing into invalid state.

Avoid silently accepting invalid external or content data.

---

## 30. Prefer Early Returns for Failed Preconditions

### Recommendation

Exit a function early when a required precondition is not satisfied and no meaningful work should continue.

### Classification

**Epic-compatible coding pattern and inference**

### Why

Early returns can:

- keep the valid path less nested;
- place checks near the dependency they protect;
- prevent partially executed state changes;
- make failure paths explicit.

### Apply When

Use early returns for:

- invalid object context;
- unsupported authority or role;
- unavailable required data;
- rejected commands;
- failed initialization prerequisites;
- no-op conditions.

### Avoid

Avoid scattering cleanup obligations across many exits. Use RAII, scoped guards, or a structured cleanup strategy when resources must be released.

Avoid replacing meaningful error propagation with silent returns.

---

## 31. Keep Control Flow and Dependency Distance Small

### Recommendation

Place dependent values and checks close to their use and decompose long control flow into named cohesive operations.

### Classification

**Official Epic coding-standard guidance**

### Why

Epic explicitly recommends minimizing dependency distance and splitting methods into sub-methods where possible.

### Apply When

Prefer code in which:

- prerequisites appear immediately before the operation they protect;
- temporary variables have narrow scope;
- method names describe meaningful steps;
- nested control flow is limited;
- state changes are grouped coherently.

### Avoid

Avoid extracting trivial one-line methods when doing so obscures rather than clarifies behavior.

### Sources

- [Epic C++ Coding Standard](https://dev.epicgames.com/documentation/unreal-engine/epic-cplusplus-coding-standard-for-unreal-engine)

---

## 32. Use Standard Library and Unreal Idioms Consistently

### Recommendation

Choose Unreal or standard-library facilities according to Epic's current guidance and avoid mixing incompatible idioms within one API.

### Classification

**Official Epic guidance**

### Epic Direction

Epic's current coding standard permits many standard-library facilities and recommends choosing the option with superior results, while valuing consistency. Epic specifically advises avoiding mixed Unreal and standard-library idioms in the same API and generally avoiding standard containers and strings except for interoperability.

Unreal Engine 5.8 documentation states that the engine compiles with C++20 by default and requires at least C++20.

### Apply When

Evaluate:

- engine integration;
- allocator requirements;
- reflection and serialization;
- API consistency;
- platform portability;
- interoperability;
- established local convention.

### Avoid

Avoid assuming all standard-library use is forbidden.

Avoid introducing standard containers into reflected or established Unreal-style APIs without a concrete interoperability reason.

### Sources

- [Epic C++ Coding Standard](https://dev.epicgames.com/documentation/unreal-engine/epic-cplusplus-coding-standard-for-unreal-engine)

---

## 33. Treat Architecture as Version- and Context-Dependent

### Recommendation

Revalidate architectural choices when engine version, platform, scale, networking model, or content workflow changes.

### Classification

**Context-dependent guidance**

### Why

Unreal systems evolve. A mechanism may become production-ready, deprecated, replaced, or better integrated in a newer version.

### Apply When

Record for significant architecture decisions:

- engine version;
- feature maturity;
- target platforms;
- expected player and Actor scale;
- server topology;
- asset-loading constraints;
- team ownership;
- Editor workflow;
- migration assumptions.

### Avoid

Avoid treating a sample-project pattern or historical engine pattern as a universal current rule.

---

## 34. Architecture Decision Checklist

### Responsibility

- What single cohesive responsibility is being introduced?
- Is it behavior, data, orchestration, presentation, or infrastructure?
- Which object semantically owns it?

### Lifetime

- Is it engine-, editor-, game-instance-, world-, local-player-, player-, Actor-, or operation-scoped?
- What creates and destroys it?
- What happens during travel, PIE restart, and shutdown?

### Authority and Networking

- Where does authoritative state live?
- Which instances exist on server and clients?
- What replicates, and why?
- Are RPC ownership and relevancy understood?

### Type Choice

- Does it need reflection?
- Does it need UObject identity or garbage collection?
- Does it need world presence?
- Does it belong on an Actor Component?
- Does a Subsystem lifetime fit?
- Would a plain C++ type be simpler?

### Dependency Direction

- Which module owns the public contract?
- Are dependencies acyclic?
- Are runtime and editor dependencies separated?
- Are implementation details private?
- Are asset dependencies deliberate?

### Extension

- Is inheritance truly required?
- Is composition more appropriate?
- Is a virtual function an intentional contract?
- Would an interface or delegate better represent the relationship?
- Are Blueprint hooks optional or required?

### State

- Is configuration separated from runtime and derived state?
- Who may mutate state?
- Are invariants protected?
- Is failure explicit?

### Build and Maintenance

- Are public headers narrow?
- Are module dependencies minimal?
- Does the design create unnecessary rebuild scope?
- Can the implementation evolve without asset or API breakage?

### Validation

- Can the responsibility be tested in isolation or a focused fixture?
- Are lifecycle and failure paths tested?
- Are non-editor targets compiled?
- Are multiplayer roles tested where relevant?
- Are Blueprint consumers compiled and validated after API changes?

---

## Summary

Strong Unreal C++ architecture is built from explicit decisions about:

- responsibility;
- ownership;
- lifetime;
- authority;
- reflection;
- dependency direction;
- module boundaries;
- composition and inheritance;
- Blueprint extension;
- asset loading;
- failure contracts;
- validation.

Epic's documentation provides specialized architecture mechanisms rather than one universal service or object type. Actors, Components, Gameplay Framework classes, Subsystems, modules, plugins, UObjects, and plain C++ types are complementary tools with different semantics.

The preferred architectural direction is to:

- work with Unreal's object and build systems;
- keep contracts narrow;
- enforce encapsulation;
- assign state to the correct lifetime and authority;
- use modules for meaningful boundaries without over-fragmentation;
- keep runtime and editor code separated;
- prefer composition for reusable Actor capabilities;
- introduce inheritance and virtual extension deliberately;
- expose intentional Blueprint APIs;
- preserve native invariants;
- make asset and network dependencies explicit;
- validate lifecycle, packaging, and multiplayer behavior continuously.

Architecture is successful when consumers can understand what owns a responsibility, when it exists, who may change it, and what it depends on—without relying on hidden global state or incidental access paths.

---

## Primary Sources

- [Programming with C++ in Unreal Engine](https://dev.epicgames.com/documentation/unreal-engine/programming-with-cplusplus-in-unreal-engine)
- [Gameplay Architecture](https://dev.epicgames.com/documentation/unreal-engine/programming-with-cpp-in-unreal-engine)
- [Gameplay Framework](https://dev.epicgames.com/documentation/unreal-engine/gameplay-framework-in-unreal-engine)
- [Gameplay Classes](https://dev.epicgames.com/documentation/unreal-engine/gameplay-classes-in-unreal-engine)
- [Epic C++ Coding Standard](https://dev.epicgames.com/documentation/unreal-engine/epic-cplusplus-coding-standard-for-unreal-engine)
- [Unreal Engine Modules](https://dev.epicgames.com/documentation/unreal-engine/unreal-engine-modules)
- [Gameplay Modules](https://dev.epicgames.com/documentation/unreal-engine/gameplay-modules-in-unreal-engine)
- [Programming Subsystems](https://dev.epicgames.com/documentation/unreal-engine/programming-subsystems-in-unreal-engine)
- [Unreal Engine C++ API Reference](https://dev.epicgames.com/documentation/unreal-engine/API)
- [Coding in Unreal Engine: Blueprint vs. C++](https://dev.epicgames.com/documentation/unreal-engine/coding-in-unreal-engine-blueprint-vs-cplusplus)
- [Exposing C++ to Blueprints](https://dev.epicgames.com/documentation/unreal-engine/exposing-cplusplus-to-blueprints-visual-scripting-in-unreal-engine)
