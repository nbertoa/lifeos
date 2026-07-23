# Reflection and UObjects in Unreal Engine

## Research Metadata

- **Research date:** 2026-07-22
- **Documentation baseline:** Unreal Engine 5.8
- **Primary authority:** Epic Games documentation
- **Scope:** Unreal reflection, UObject responsibilities, UHT, UCLASS, USTRUCT, UENUM, UFUNCTION, UPROPERTY, class default objects, construction, ownership, garbage collection, object pointer types, serialization, duplication, networking, editor exposure, and lifecycle
- **Status:** Living document

---

## Purpose

Unreal Engine extends standard C++ with an object and reflection system used by major engine features.

The system supports:

- runtime type information;
- garbage collection;
- serialization;
- property editing;
- Blueprint exposure;
- replication;
- default-object propagation;
- asset references;
- object duplication;
- configuration;
- metadata-driven tooling.

These capabilities are powerful, but they are not free and they do not apply automatically to every C++ declaration.

The central principle is:

> Use Unreal reflection and UObject semantics only when the type or member requires engine-managed identity, lifetime, serialization, editor integration, Blueprint integration, replication, or runtime metadata.

A type should not derive from `UObject` merely because it belongs to an Unreal project.

---

## 1. Distinguish Standard C++ From Reflected Unreal Types

### Recommendation

Choose deliberately between:

- a standard C++ type;
- a reflected `USTRUCT`;
- a `UObject`;
- an `AActor`;
- a Component;
- another engine-managed framework type.

### Classification

**Official Epic guidance**

### Why

Epic explicitly states that standard C++ classes, functions, and variables can coexist with Unreal reflected types.

Reflection is appropriate when engine systems must discover, serialize, display, replicate, or manage a declaration.

Plain C++ remains appropriate when the type:

- has no engine-managed identity;
- does not need garbage collection;
- does not need Blueprint or Editor exposure;
- does not need reflected serialization;
- does not need replication;
- benefits from normal value, stack, RAII, or ownership semantics.

### Apply When

Prefer plain C++ for:

- algorithms;
- private implementation details;
- transient helper objects;
- scoped resources;
- internal data structures;
- types naturally owned through value semantics or standard ownership.

Prefer a reflected type when engine integration is part of the requirement.

### Avoid

Avoid deriving every service, helper, or data holder from `UObject`.

Reflection should solve a concrete engine-integration problem.

### Sources

- [Reflection System](https://dev.epicgames.com/documentation/unreal-engine/reflection-system-in-unreal-engine)
- [Objects](https://dev.epicgames.com/documentation/unreal-engine/objects-in-unreal-engine)

---

## 2. Understand What Reflection Actually Provides

### Recommendation

Treat reflection as an opt-in contract between C++ declarations and Unreal Engine tooling and runtime systems.

### Classification

**Official requirement**

### Why

A C++ member existing inside a `UCLASS` does not automatically make that member visible to Unreal's reflection system.

Only declarations marked and supported by Unreal macros and types become reflected metadata.

Common reflected declarations include:

- `UCLASS`;
- `USTRUCT`;
- `UENUM`;
- `UINTERFACE`;
- `UPROPERTY`;
- `UFUNCTION`;
- delegate declarations.

Reflection may enable:

- property enumeration;
- serialization;
- garbage-collector reference discovery;
- Blueprint nodes and variables;
- Editor details;
- replication metadata;
- class and function lookup;
- default-value propagation;
- asset and object inspection.

### Avoid

Do not assume that reflection is equivalent to complete C++ introspection.

Unreal reflects supported declarations that UHT can parse and generate metadata for.

---

## 3. Treat Unreal Header Tool as a Build-Time Contract

### Recommendation

Write reflected declarations according to Unreal Header Tool requirements rather than assuming that valid C++ is necessarily valid reflected Unreal C++.

### Classification

**Official requirement**

### Why

Unreal Header Tool parses supported headers before normal C++ compilation and generates code required by the object system.

Reflected declarations require:

- the appropriate Unreal macro;
- `GENERATED_BODY()`;
- supported reflected types;
- valid declaration structure;
- the generated header include in the expected location;
- metadata and specifier syntax understood by UHT.

A declaration can be valid for a C++ compiler while still being rejected or misunderstood by UHT.

### Implementation Guidance

For reflected headers:

- include the generated header last among the header's includes;
- keep generated body macros in the required declaration;
- avoid unsupported preprocessor structures around reflected declarations;
- use `WITH_EDITOR` and `WITH_EDITORONLY_DATA` according to engine conventions;
- inspect the first UHT error rather than only downstream C++ errors.

### Avoid

Avoid designing reflected APIs around advanced C++ constructs that UHT cannot represent.

Keep native-only implementation details unreflected where appropriate.

---

## 4. Use UObject for Engine-Managed Object Identity

### Recommendation

Use `UObject` when the entity requires engine-managed identity and one or more UObject-system services.

### Classification

**Official Epic guidance**

### UObject Capabilities

Epic documents UObject support for:

- garbage collection;
- reference updating;
- reflection;
- serialization;
- automatic property initialization;
- automatic updating of default-property changes;
- Editor integration;
- runtime type information;
- network replication support.

### Apply When

A UObject may be appropriate when the object:

- must be referenced through reflected properties;
- must participate in garbage collection;
- must be instanced or duplicated by the engine;
- needs Blueprint exposure;
- needs property editing;
- needs reflected serialization;
- needs a `UClass` identity;
- is an asset or subobject;
- must integrate with Unreal object lookup and metadata.

### Avoid or Reconsider When

Do not use UObject when:

- deterministic RAII destruction is required;
- stack allocation is the natural ownership model;
- constructor parameters are fundamental to validity;
- the type is a short-lived algorithm object;
- reflection adds no value;
- value semantics are preferable.

---

## 5. Never Allocate UObjects With `new`

### Recommendation

Create UObjects using Unreal object creation APIs.

### Classification

**Official requirement**

### Correct Creation APIs

Common creation paths include:

- `NewObject<T>()` for runtime UObject instances;
- `CreateDefaultSubobject<T>()` for default subobjects created in constructors;
- `SpawnActor<T>()` for Actors;
- asset creation and duplication APIs for their corresponding workflows.

### Why

UObjects are registered with the Unreal object system and managed through engine-specific allocation, construction, naming, flags, outers, and garbage collection.

Using standard `new` or `delete` bypasses these mechanisms and can corrupt object state or memory.

### Avoid

Never:

```cpp
UMyObject* Object = new UMyObject();
delete Object;
```

Use the appropriate Unreal API instead.

### Sources

- [Objects](https://dev.epicgames.com/documentation/unreal-engine/objects-in-unreal-engine)

---

## 6. Keep UObject Constructors Lightweight

### Recommendation

Use UObject constructors primarily for defaults and default subobject creation.

### Classification

**Official Epic guidance**

### Why

UObject constructors execute in contexts beyond ordinary runtime instance creation, including class default object construction.

At construction time:

- the world may not exist;
- gameplay systems may not be initialized;
- the object may be a CDO;
- serialized properties may not yet contain their final values;
- derived Blueprint defaults may not yet be applied;
- other runtime objects may not be available.

### Appropriate Constructor Work

Constructors may normally:

- assign native default values;
- establish default flags;
- create default subobjects;
- configure default subobjects;
- establish class-level defaults that do not depend on runtime state.

### Avoid

Avoid in UObject constructors:

- gameplay execution;
- world queries;
- spawning runtime Actors;
- accessing player state;
- calling Blueprint events;
- loading arbitrary dynamic content;
- depending on serialized instance data;
- registering with runtime services;
- work that must occur once per playable instance.

### Sources

- [Objects](https://dev.epicgames.com/documentation/unreal-engine/objects-in-unreal-engine)

---

## 7. Understand the Class Default Object

### Recommendation

Treat the Class Default Object as a class template and source of default property values, not as a normal gameplay instance.

### Classification

**Official Epic guidance**

### Why

Each `UClass` owns a Class Default Object created from the class constructor.

The CDO participates in:

- class default storage;
- instance initialization;
- inheritance of native defaults;
- Blueprint default values;
- editor-facing class configuration.

### Practical Consequences

Code running in a constructor may be running for:

- the native CDO;
- a Blueprint-generated class CDO;
- a runtime object instance;
- an object reconstructed by editor workflows.

### Implementation Guidance

When behavior is runtime-instance-only, place it in an appropriate later lifecycle method.

When code may legitimately run for both CDOs and instances, make that distinction explicit.

Useful checks may include object flags such as `RF_ClassDefaultObject`, but checking flags should not become a substitute for selecting the correct lifecycle phase.

### Avoid

Do not mutate the CDO as though it were per-instance runtime state.

Do not retain world-specific state on a CDO.

---

## 8. Use `CreateDefaultSubobject` Only for Default Subobjects

### Recommendation

Create constructor-owned default subobjects with `CreateDefaultSubobject`.

### Classification

**Official requirement and Epic pattern**

### Why

Default subobjects become part of the class's default object graph and participate in inherited construction and serialized defaults.

Typical examples include:

- Actor Components;
- Scene Components;
- UObject subobjects intended to exist on every instance.

### Apply When

Use a default subobject when:

- every instance conceptually owns one;
- derived classes may configure or replace its class through supported patterns;
- it is part of the object's structural defaults;
- it should exist before runtime initialization.

### Avoid

Do not use `CreateDefaultSubobject` for conditional runtime objects.

Do not create default subobjects outside object construction.

Use `NewObject` for runtime-instanced UObject children.

---

## 9. Make `Outer` Semantics Deliberate

### Recommendation

Select a UObject's `Outer` according to containment, naming, serialization, duplication, and lifetime context.

### Classification

**Epic pattern and context-dependent guidance**

### Why

`Outer` contributes to:

- an object's full path and namespace;
- containment relationships;
- duplication behavior;
- package association;
- serialization context;
- object lookup;
- lifecycle reasoning.

### Important Distinction

An `Outer` is not by itself a universal ownership guarantee.

Garbage-collection reachability is determined by recognized strong references and root relationships, not merely by assuming that an outer always keeps every inner object alive in every context.

### Apply When

Choose an Outer that semantically contains the object:

- an owning UObject for an instanced subobject;
- a package for an asset;
- a transient package for suitable transient objects;
- a world, game instance, or other framework owner only when that containment is correct.

### Avoid

Avoid using arbitrary long-lived outers solely to keep an object alive.

Represent lifetime through the correct strong-reference mechanism.

---

## 10. Mark Persistent UObject References Correctly

### Recommendation

Use `UPROPERTY()` with `TObjectPtr<T>` for persistent UObject references stored on reflected classes or structs when the reference should participate in garbage collection, serialization, or replication.

### Classification

**Official Epic recommendation**

### Why

Epic's object-pointer guidance identifies `TObjectPtr<T>` as the common persistent UObject reference for reflected members.

When marked with `UPROPERTY`, it:

- participates in garbage-collection reference tracking;
- supports serialization;
- supports replication where applicable;
- supports cook-time dependency tracking;
- enables barriers needed by incremental GC features.

### Example

```cpp
UPROPERTY()
TObjectPtr<UMyObject> OwnedObject;
```

### Avoid

A bare raw member pointer is not automatically a GC-tracked strong reference.

Do not assume that being a member of a UObject is sufficient.

### Sources

- [Object Pointers](https://dev.epicgames.com/documentation/unreal-engine/object-pointers-in-unreal-engine)

---

## 11. Use Raw UObject Pointers for Short-Lived Observation

### Recommendation

Use raw UObject pointers primarily for local variables, parameters, return values, and other short-lived references whose lifetime is already guaranteed by the surrounding operation.

### Classification

**Official Epic guidance**

### Why

Epic's pointer guide lists raw pointers as the common choice for:

- local variables;
- function parameters;
- short-lived references not stored as reflected fields.

A raw pointer:

- does not independently keep the object alive;
- is not automatically serialized;
- is not automatically replicated;
- is not automatically a persistent GC reference unless used through specific supported reflected behavior.

### Apply When

Raw pointers are appropriate when:

- the pointer is used immediately;
- the caller or another tracked reference guarantees lifetime;
- it does not escape into asynchronous work;
- it is not stored beyond a safe scope.

### Avoid

Do not capture a raw UObject pointer into delayed or asynchronous work unless lifetime is explicitly protected.

---

## 12. Use `TWeakObjectPtr` for Non-Owning UObject References

### Recommendation

Use `TWeakObjectPtr<T>` when the reference must not keep the target UObject alive and the target may be destroyed independently.

### Classification

**Official Epic recommendation**

### Why

A weak object pointer:

- does not affect garbage collection;
- can become invalid when the target is destroyed;
- supports safe validity-aware access patterns;
- communicates non-ownership.

### Apply When

Use it for:

- caches;
- observers;
- back-references;
- optional links to independently owned Actors or UObjects;
- references that should not extend lifetime.

### Validation

Check or pin the pointer before use according to the API and execution context.

Do not assume that a previous validity check remains valid across arbitrary asynchronous or threaded work.

### Sources

- [Object Pointers](https://dev.epicgames.com/documentation/unreal-engine/object-pointers-in-unreal-engine)

---

## 13. Use `TSoftObjectPtr` and `TSoftClassPtr` for Loadable References

### Recommendation

Use soft references when an asset or class should be identified without necessarily being loaded.

### Classification

**Official Epic guidance**

### Why

A soft object pointer tracks an object path and can represent an unloaded asset.

It:

- does not keep the target UObject alive;
- supports on-demand loading;
- avoids forcing the same hard dependency behavior as a strong object reference;
- can move between pending, valid, and unloaded states.

### Apply When

Use soft references for:

- optional assets;
- content loaded asynchronously;
- large assets not needed at startup;
- content selected through data;
- class references that should not force immediate loading;
- asset-manager workflows.

### Avoid

Do not replace every hard reference with a soft reference.

Soft references introduce:

- explicit loading requirements;
- possible asynchronous completion;
- unavailable-object states;
- asset-path and cooking considerations.

Use them when deferred loading is part of the design.

### Sources

- [Object Pointers](https://dev.epicgames.com/documentation/unreal-engine/object-pointers-in-unreal-engine)
- [FSoftObjectPtr](https://dev.epicgames.com/documentation/unreal-engine/API/Runtime/CoreUObject/FSoftObjectPtr)

---

## 14. Use `TStrongObjectPtr` With Caution

### Recommendation

Use `TStrongObjectPtr<T>` only when a non-UObject owner requires a strong reference to a UObject and a more visible reflected ownership path is not available.

### Classification

**Official caution**

### Why

Epic warns that `TStrongObjectPtr`:

- keeps the target alive from any storage context;
- cannot be marked as `UPROPERTY`;
- is less visible to reference-debugging tools;
- adds a tracked GC reference;
- has nontrivial creation and destruction cost;
- can create problematic uncollectable ownership cycles.

### Apply When

A reasonable use case is a long-lived non-UObject type that genuinely must retain a UObject.

### Avoid

Avoid:

- frequently creating and destroying strong object pointers;
- using them inside UObject fields as a substitute for `UPROPERTY TObjectPtr`;
- self-referential strong cycles;
- using them merely to avoid designing ownership.

### Sources

- [Object Pointers](https://dev.epicgames.com/documentation/unreal-engine/object-pointers-in-unreal-engine)

---

## 15. Do Not Use Standard Shared Pointers to Own UObjects

### Recommendation

Do not manage UObject lifetime with `TSharedPtr`, `std::shared_ptr`, `TUniquePtr`, or equivalent standard ownership mechanisms.

### Classification

**Official requirement**

### Why

UObjects are managed by Unreal's object allocator and garbage collector.

Unreal's shared-pointer library is intended for non-UObject types.

Combining independent ownership systems with UObject destruction is unsafe.

### Apply When

Use shared or unique pointers for suitable non-UObject C++ types.

Use Unreal object references and GC-aware mechanisms for UObjects.

---

## 16. Understand Reachability Rather Than Thinking in Manual Deletion

### Recommendation

Reason about UObject lifetime through reachability and recognized references.

### Classification

**Official Unreal object model**

### Why

Unreal uses garbage collection for UObjects.

A UObject remains alive while reachable through supported strong-reference paths or explicit root/reference collector mechanisms.

When it becomes unreachable, the collector may reclaim it.

### Strong-Reachability Examples

Depending on context, strong reachability may come from:

- `UPROPERTY`-tracked `TObjectPtr`;
- reflected containers containing tracked references;
- engine-managed object graphs;
- root-set membership;
- custom reference collection;
- `TStrongObjectPtr`.

### Non-Owning Examples

The following do not normally keep a target alive:

- raw local pointer;
- `TWeakObjectPtr`;
- `TSoftObjectPtr`;
- object path by itself.

### Avoid

Do not manually call `delete` on UObjects.

Do not assume that a non-null raw pointer guarantees a live object.

---

## 17. Prefer Normal Reference Tracking Over `AddToRoot`

### Recommendation

Represent ownership with normal object relationships and tracked references before considering root-set manipulation.

### Classification

**Epic pattern**

### Why

`AddToRoot` prevents garbage collection regardless of normal reachability.

It is powerful but easy to misuse, especially when:

- cleanup paths are missed;
- editor sessions recreate systems;
- hot reload or shutdown occurs;
- ownership becomes invisible;
- global state accumulates.

### Apply When

Rooting may be justified for narrowly controlled engine-level or bridging scenarios where normal ownership is unavailable.

### Avoid

Do not use `AddToRoot` as the default fix for disappearing UObjects.

First determine which object should own the reference and express that ownership directly.

### Validation

Every `AddToRoot` must have a clearly paired `RemoveFromRoot` path.

---

## 18. Implement Custom Reference Collection Only When Necessary

### Recommendation

Use `FGCObject` or `AddReferencedObjects` when a nonstandard owner must expose UObject references to the collector.

### Classification

**Context-dependent Epic mechanism**

### Why

Some native structures cannot store their references in reflected properties.

Unreal provides explicit reference-collection mechanisms for those cases.

### Apply When

Use custom reference collection when:

- a non-UObject type owns UObject references;
- reflection cannot represent the container or ownership;
- engine architecture requires manual enumeration of references.

### Avoid

Do not use custom collectors when a normal `UPROPERTY TObjectPtr` relationship would work.

Manual reference enumeration increases the risk of:

- omitted references;
- stale references;
- confusing ownership;
- difficult debugging.

---

## 19. Distinguish `nullptr` From Object Validity

### Recommendation

Use the narrowest validity check required by the contract.

### Classification

**Context-dependent guidance**

### Why

A pointer can be:

- null;
- non-null but pending destruction or marked as garbage;
- weakly invalid;
- valid but in an unsuitable lifecycle state;
- valid as an object but not appropriate for the requested world or authority.

### Guidance

Use:

- `if (Pointer)` when nullness is the only meaningful question and lifetime is guaranteed;
- `IsValid(Object)` when the object may be null or no longer usable under UObject validity rules;
- weak-pointer validity or pinning for weak references;
- explicit lifecycle or domain checks for higher-level preconditions.

### Avoid

Do not use `IsValid` mechanically everywhere.

It cannot replace:

- correct ownership;
- clearing references on lifecycle events;
- authority checks;
- initialization state;
- thread safety;
- semantic validation.

---

## 20. Do Not Assume Garbage Collection Can Run Anywhere

### Recommendation

Respect Unreal's GC and threading model when accessing UObjects.

### Classification

**Official caution**

### Why

Epic warns against directly accessing objects through `TObjectPtr` from worker threads unless the object is known to be rooted and protected from collection.

UObject operations are generally designed around game-thread and engine lifecycle constraints.

### Apply When

For worker-thread access:

- avoid touching mutable UObject state unless the API explicitly supports it;
- copy plain data into thread-safe structures;
- use weak-to-strong pinning mechanisms where documented;
- return results to the game thread for UObject mutation;
- ensure object lifetime is protected for the complete operation.

### Avoid

Do not assume that a raw pointer captured by a task remains safe until task completion.

### Sources

- [Object Pointers](https://dev.epicgames.com/documentation/unreal-engine/object-pointers-in-unreal-engine)

---

## 21. Keep UObject Counts and Reference Graphs Under Control

### Recommendation

Treat UObject population and GC traversal cost as performance concerns.

### Classification

**Official Epic performance guidance**

### Why

Unreal's mark-and-sweep collector performs reachability analysis over UObject graphs.

Large object counts and reference graphs can increase collection cost and produce hitches.

Epic's incremental GC documentation identifies strategies such as:

- maintaining UObject budgets;
- pooling where justified;
- controlling GC timing in suitable projects.

Incremental GC remains feature-maturity-dependent and must be classified accordingly.

### Apply When

Profile GC when the project has:

- very large UObject populations;
- frequent transient object creation;
- visible periodic hitches;
- large streaming transitions;
- complex editor or runtime object graphs.

### Avoid

Do not convert every small value or transient record into a UObject.

Prefer value types or native structures when identity and reflection are unnecessary.

### Sources

- [Incremental Garbage Collection](https://dev.epicgames.com/documentation/unreal-engine/incremental-garbage-collection-in-unreal-engine)

---

## 22. Use USTRUCT for Reflected Value Types

### Recommendation

Use `USTRUCT` for reflected data with value semantics.

### Classification

**Official Epic guidance**

### Why

A `USTRUCT` can participate in:

- reflection;
- property serialization;
- Blueprint type exposure;
- editor property display;
- replication as part of reflected properties;
- Make and Break Blueprint nodes.

Unlike UObjects, structs:

- are value types;
- are not independently garbage collected;
- do not have UObject identity;
- can be embedded directly in other types.

### Apply When

Use a USTRUCT for:

- grouped configuration;
- messages;
- result values;
- replicated state records;
- save-data records;
- data-table rows;
- value-like domain types.

### Avoid or Reconsider When

Use UObject instead when the entity requires:

- independent identity;
- polymorphic UObject references;
- GC-managed lifetime;
- instancing;
- separate asset identity;
- object-level editor or Blueprint behavior.

### Sources

- [Structs](https://dev.epicgames.com/documentation/unreal-engine/structs-in-unreal-engine)

---

## 23. Remember That USTRUCT UObject References Still Need Tracking

### Recommendation

Mark UObject references inside reflected structs with `UPROPERTY` and supported pointer types when they must participate in GC or serialization.

### Classification

**Official requirement**

### Why

The struct itself is not garbage collected, but reflected properties inside it can be traversed as part of the containing reflected object or property graph.

A native, unreflected pointer hidden inside a struct is not automatically discovered by GC.

### Example

```cpp
USTRUCT(BlueprintType)
struct FTargetData
{
    GENERATED_BODY()

    UPROPERTY()
    TObjectPtr<UObject> Target;
};
```

### Avoid

Do not assume `USTRUCT()` alone makes every member visible to Unreal.

Each member requiring reflection must be declared appropriately.

---

## 24. Use UENUM for Stable Reflected Enumerations

### Recommendation

Use `UENUM` when an enum must be exposed to Blueprint, serialization, metadata, or reflected properties.

### Classification

**Official requirement for reflected enum use**

### Why

A reflected enum becomes part of serialized and Blueprint-facing contracts.

### Apply When

Use explicit stable semantic values for:

- state categories;
- result codes;
- configuration modes;
- Blueprint branching;
- replicated enum state.

### Avoid

Avoid casually reordering or repurposing serialized enum values.

Treat changes as data migrations when persisted assets or saves depend on them.

Avoid using an enum when combinations are valid and a bitmask or tag taxonomy better represents the domain.

---

## 25. Reflect Only the Members That Need Reflection

### Recommendation

Keep implementation-only fields and methods as ordinary native C++ members.

### Classification

**Official capability and Epic pattern**

### Why

Epic documents that UObject classes may contain native-only members not marked with `UPROPERTY` or `UFUNCTION`.

Reflecting less can:

- reduce public API surface;
- reduce metadata complexity;
- preserve encapsulation;
- avoid unnecessary Editor or Blueprint exposure;
- make ownership requirements explicit.

### Apply When

Do not reflect a member when it is:

- private implementation state;
- recomputable transient data;
- a native helper;
- unsupported by reflection;
- irrelevant to GC, serialization, Blueprint, replication, configuration, or editor tooling.

### Avoid

Do not remove reflection from UObject references that rely on it for GC tracking.

---

## 26. Separate Reflection Specifiers From Metadata

### Recommendation

Use specifiers to define engine behavior and metadata to refine tooling or presentation.

### Classification

**Official requirement**

### Why

Unreal declarations may include:

- specifiers controlling serialization, editing, Blueprint access, replication, instancing, and configuration;
- metadata controlling display, node behavior, restrictions, validation hints, and editor experience.

### Example

```cpp
UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, Category="Weapon", meta=(ClampMin="0.0"))
float Damage = 10.0f;
```

Here:

- `EditDefaultsOnly` and `BlueprintReadOnly` are property specifiers;
- `Category` is a common organizing specifier;
- `ClampMin` is metadata.

### Avoid

Do not rely on metadata as a runtime security or invariant boundary.

Editor restrictions do not replace runtime validation.

---

## 27. Choose Property Editing Scope Deliberately

### Recommendation

Expose properties at the narrowest editing scope that supports the intended authoring workflow.

### Classification

**Epic pattern**

### Common Choices

- `EditDefaultsOnly`: edit class defaults, not individual runtime instances;
- `EditInstanceOnly`: edit placed or specific instances;
- `EditAnywhere`: edit both;
- `VisibleDefaultsOnly`;
- `VisibleInstanceOnly`;
- `VisibleAnywhere`;
- no editing specifier for hidden implementation state.

### Apply When

Use defaults-only for class identity and content archetypes.

Use instance editing for legitimate per-instance level configuration.

Use visibility rather than editability for state that authors may inspect but must not mutate directly.

### Avoid

Avoid `EditAnywhere` as an automatic default.

Every additional editing location increases the number of possible configurations and validation cases.

---

## 28. Separate Editor Editability From Blueprint Mutability

### Recommendation

Decide Editor access and Blueprint access independently.

### Classification

**Official property-system behavior**

### Why

These are different questions:

- Can an author edit this property in the Details panel?
- Can Blueprint graphs read it?
- Can Blueprint graphs write it?

For example:

```cpp
UPROPERTY(EditDefaultsOnly, BlueprintReadOnly)
float MaxHealth = 100.0f;
```

permits class-default authoring and Blueprint reading without unrestricted graph mutation.

### Avoid

Do not use `BlueprintReadWrite` merely because a value is visible or editable in the Editor.

Expose controlled operations when runtime mutation requires invariants.

---

## 29. Use Instanced Properties for True Owned Subobjects

### Recommendation

Use `Instanced` and compatible class specifiers when each owning object requires a unique editable UObject subobject instance.

### Classification

**Official property behavior**

### Why

An instanced property creates a distinct object associated with the owning instance rather than sharing a default asset reference.

This can support:

- inline editable strategy objects;
- polymorphic configuration subobjects;
- owned behavior definitions;
- embedded object graphs.

### Apply When

Use instanced subobjects when:

- identity is local to the owner;
- each owner may configure a unique object;
- duplication should duplicate the subobject;
- the object should serialize as part of the owner.

### Avoid

Do not use instanced UObjects where a value struct or shared Data Asset is sufficient.

Instanced graphs can increase duplication, serialization, and authoring complexity.

---

## 30. Distinguish Hard References, Weak References, and Soft References

### Recommendation

Select reference semantics according to both lifetime and loading behavior.

### Classification

**Official Epic guidance**

| Requirement | Typical Reference |
|---|---|
| Persistent loaded UObject, keep alive | `UPROPERTY TObjectPtr<T>` |
| Short-lived observed object | `T*` |
| Non-owning reference that may become invalid | `TWeakObjectPtr<T>` |
| Asset or class loadable on demand | `TSoftObjectPtr<T>` / `TSoftClassPtr<T>` |
| Strong UObject retention from non-UObject owner | `TStrongObjectPtr<T>` with caution |
| Type-safe class selection | `TSubclassOf<T>` |
| Class selection without forced load | `TSoftClassPtr<T>` |

### Why

Lifetime and loading are separate concerns.

A reference may:

- keep a loaded object alive;
- observe an object without owning it;
- identify an unloaded asset;
- constrain a selected class type;
- serialize an asset path.

### Avoid

Do not choose pointer types solely by syntax preference.

Document the intended ownership and loading contract.

---

## 31. Use `TSubclassOf` for Type-Safe Class References

### Recommendation

Use `TSubclassOf<BaseType>` when callers or designers select a class derived from a required base.

### Classification

**Official Epic guidance**

### Why

`TSubclassOf` wraps a `UClass` reference with type constraints.

This is useful for:

- spawnable Actor classes;
- strategy or policy classes;
- Component types;
- Blueprint-derived implementations;
- factory configuration.

### Apply When

Use it when the system needs a class, not an object instance.

### Avoid

Do not use a generic `UClass*` when the required base type is known.

Use `TSoftClassPtr` when class selection should not force the class asset to load.

### Sources

- [Reflection System](https://dev.epicgames.com/documentation/unreal-engine/reflection-system-in-unreal-engine)

---

## 32. Treat Reflected Properties as Serialization Contracts

### Recommendation

Assume that reflected serialized properties may persist in assets, maps, defaults, instances, saves, duplication, or configuration according to their specifiers and context.

### Classification

**Official Unreal behavior**

### Why

Changing a reflected declaration can affect existing content.

Risky changes include:

- renaming properties;
- changing property type;
- changing ownership or instancing semantics;
- removing a property;
- changing enum values;
- moving a property between classes;
- replacing a hard reference with a soft reference;
- changing default values relied on by derived assets.

### Apply When

For reflected migrations:

- search for content consumers;
- use Core Redirects where supported;
- add explicit migration logic where needed;
- compile affected Blueprints;
- resave representative assets;
- validate cooking and packaging;
- test old serialized data.

### Avoid

Do not treat a reflected private property as harmless to change merely because C++ callers cannot see it.

Serialization can make private reflected fields persistent contracts.

---

## 33. Use `Transient` for Data That Must Not Persist

### Recommendation

Mark runtime-only reflected state as transient when it should not be serialized into persistent object data.

### Classification

**Official property behavior**

### Apply When

Transient state may include:

- runtime caches;
- generated handles;
- world-specific temporary state;
- dynamically resolved references;
- ephemeral UI or gameplay state;
- data rebuilt after loading.

### Avoid

Do not mark data transient merely to hide serialization bugs.

First decide whether the state should:

- persist;
- replicate;
- be reconstructed;
- be initialized from defaults;
- be saved through a separate save system.

---

## 34. Distinguish Duplication From Serialization

### Recommendation

Validate how reflected object graphs behave during duplication, PIE, Blueprint compilation, and editor reconstruction.

### Classification

**Context-dependent guidance**

### Why

Editor workflows create and duplicate objects in ways that differ from simple runtime creation.

An object may be:

- duplicated for PIE;
- reinstanced after Blueprint compilation;
- reconstructed after property editing;
- copied as a template;
- duplicated with its owner;
- loaded from a package.

### Apply When

Pay special attention to:

- instanced subobjects;
- external registrations;
- cached raw pointers;
- delegates bound to transient objects;
- native resources;
- object names and outers;
- state that must be rebuilt after duplication or load.

### Avoid

Do not rely on a constructor being the only place an object acquires meaningful state.

---

## 35. Use Lifecycle Hooks According to Their Contract

### Recommendation

Place initialization, loading, editing, runtime activation, and destruction logic in the lifecycle callback that provides the required guarantees.

### Classification

**Epic pattern and official framework behavior**

### Relevant UObject Hooks

Depending on the type and requirement:

- constructor;
- `PostInitProperties`;
- `PostLoad`;
- `PostDuplicate`;
- `BeginDestroy`;
- `FinishDestroy`;
- editor property-change callbacks;
- Actor and Component-specific initialization and play callbacks.

### Guidance

Use:

- constructor for defaults and default subobjects;
- `PostInitProperties` for post-property initialization work where appropriate;
- `PostLoad` for restoring derived or migrated state after serialized data is loaded;
- `PostDuplicate` for duplication-specific repair;
- framework-specific play hooks for runtime gameplay activation;
- destruction hooks only for their documented cleanup responsibilities.

### Avoid

Do not move all initialization indiscriminately into one callback.

Initialization requirements differ between:

- newly constructed objects;
- loaded objects;
- duplicated objects;
- CDOs;
- editor previews;
- runtime world instances.

---

## 36. Do Not Expect Deterministic UObject Destruction

### Recommendation

Do not depend on the garbage collector to destroy a UObject immediately when a reference is released.

### Classification

**Official object model**

### Why

Garbage collection is deferred and scheduled by the engine.

The moment an object becomes unreachable is not necessarily the moment its memory is reclaimed.

### Apply When

Release external resources explicitly at the lifecycle point where the resource must stop being used.

Examples include:

- unregistering callbacks;
- canceling asynchronous requests;
- releasing platform handles;
- stopping external services;
- detaching from native systems.

### Avoid

Do not use a UObject destructor as the primary gameplay cleanup event.

Do not retain critical external resources until an unpredictable GC pass when earlier release is required.

---

## 37. Keep `BeginDestroy` and `FinishDestroy` Focused

### Recommendation

Use UObject destruction callbacks according to their documented destruction phases and avoid gameplay-level assumptions.

### Classification

**Context-dependent Epic mechanism**

### Why

By destruction time:

- world systems may already be shutting down;
- referenced objects may be invalid;
- gameplay ordering may no longer be meaningful;
- cleanup may span multiple phases.

### Apply When

Use destruction callbacks for UObject-level teardown that cannot be handled earlier.

### Avoid

Avoid:

- spawning objects;
- sending gameplay events;
- depending on world availability;
- complex inter-object coordination;
- assuming immediate final memory release.

Prefer explicit shutdown or framework end-play paths for runtime systems.

---

## 38. Make Reflection Boundaries Encapsulated

### Recommendation

Do not let reflection force all state to become public or mutable.

### Classification

**Epic coding-standard alignment**

### Why

`UPROPERTY` and `UFUNCTION` can be declared with restricted native access while still exposing the intended reflected contract.

Encapsulation remains important because reflected systems can otherwise bypass invariants.

### Implementation Guidance

Prefer:

- private or protected reflected fields where appropriate;
- `BlueprintReadOnly` for observable state;
- focused `BlueprintCallable` operations;
- getter functions for computed state;
- validation at mutation boundaries;
- `AllowPrivateAccess` metadata only as an intentional bridge, not a default excuse.

### Avoid

Do not expose a writable property when a semantic operation better protects the object.

---

## 39. Treat Blueprint Exposure as API Design

### Recommendation

Design reflected Blueprint members for the Blueprint caller rather than merely exporting native implementation details.

### Classification

**Epic pattern**

### Consider

For every Blueprint-facing member:

- clear category;
- meaningful display name;
- understandable pin names;
- safe defaults;
- purity semantics;
- failure behavior;
- authority restrictions;
- world context;
- advanced pins;
- discoverability;
- tooltip documentation.

### Avoid

Avoid exposing:

- raw internal state transitions;
- functions unsafe during normal Blueprint lifecycle phases;
- functions that appear pure but perform expensive or stateful work;
- redundant aliases;
- low-level sequencing that callers must reproduce correctly.

---

## 40. Separate Reflection From Runtime Validation

### Recommendation

Use reflection metadata to improve authoring, but validate runtime invariants in executable code.

### Classification

**Inference**

### Why

Metadata can constrain or improve the Editor experience, but:

- assets can be created by scripts;
- data can be loaded from older versions;
- runtime values can come from networking or code;
- shipping builds may not include the same editor interactions;
- metadata is not always an enforcement boundary.

### Apply When

Use metadata for:

- clamps;
- allowed classes;
- edit conditions;
- asset filters;
- units;
- categories;
- node layout.

Use runtime validation for:

- authority;
- security;
- state transitions;
- required references;
- cross-field invariants;
- unsupported combinations;
- lifecycle preconditions.

---

## 41. Use Data Validation for Reflected Content Contracts

### Recommendation

Validate reflected assets and configurable objects before runtime where practical.

### Classification

**Official Epic pattern**

### Apply When

Validation is valuable for:

- missing required object references;
- invalid soft paths;
- incompatible classes;
- invalid ranges;
- conflicting property combinations;
- required instanced subobjects;
- unsupported platform assets;
- deprecated types;
- incomplete Blueprint extension points.

### Why

Reflection exposes configuration power. Validation prevents that flexibility from becoming uncontrolled invalid state.

### Avoid

Do not rely exclusively on constructor assumptions for content authored after construction.

---

## 42. Keep Replication Semantics Explicit

### Recommendation

Do not assume that a reflected property replicates merely because it is a `UPROPERTY`.

### Classification

**Official Unreal networking requirement**

### Why

Reflection makes a property eligible for Unreal systems, but replication requires explicit replication declarations and framework support.

Replication also depends on:

- object and Actor networking rules;
- authority;
- ownership;
- lifetime-property registration or newer supported mechanisms;
- supported property types;
- replication conditions;
- subobject registration where applicable.

### Apply When

For replicated UObject state, document:

- which owning network object carries it;
- whether the UObject is a replicated subobject;
- who creates it;
- how it is registered;
- which properties replicate;
- how late joiners receive state;
- how destruction is communicated.

### Avoid

Do not treat arbitrary UObjects as automatically network-addressable or replicated.

---

## 43. Distinguish Object Identity From Object Data

### Recommendation

Use UObject identity only when independent identity is part of the domain.

### Classification

**Inference**

### Why

A UObject introduces:

- allocation and registration;
- naming;
- GC participation;
- indirection;
- object-graph complexity;
- lifecycle hooks;
- duplication and serialization semantics.

A value struct may be simpler when the concept is merely data.

### Apply When

Ask:

- Must multiple owners refer to the same identity?
- Must the object be polymorphic?
- Must it exist independently of the containing value?
- Must it be edited or serialized as an object?
- Must it participate in GC?
- Must it be a Blueprint object reference?

If not, a struct or native value type may be better.

---

## 44. Avoid UObject-Based Global State

### Recommendation

Do not use UObject reachability or rooting to create hidden global services.

### Classification

**Inference**

### Why

A globally reachable UObject can create:

- unclear ownership;
- shutdown-order problems;
- world leakage;
- test contamination;
- multiplayer-context mistakes;
- editor/runtime coupling.

### Apply When

Choose a lifecycle-aware host such as the appropriate Subsystem or framework owner when the service genuinely belongs to:

- the engine;
- one editor;
- one game instance;
- one world;
- one local player.

### Avoid

Do not retain a UObject globally simply because garbage collection otherwise removes it.

That usually indicates missing ownership.

---

## 45. Keep World Context Explicit

### Recommendation

Do not assume every UObject has a valid or unique world context.

### Classification

**Epic pattern**

### Why

A UObject may be:

- an asset;
- a CDO;
- a transient utility object;
- owned by a GameInstance;
- owned by an Actor;
- created in the Editor;
- present during world teardown;
- shared across contexts.

### Apply When

When a UObject requires a world:

- define how it obtains the world;
- validate the owner or context;
- avoid caching stale world pointers;
- handle editor and multi-world environments;
- do not infer authority without the correct world and network context.

### Avoid

Avoid generic `GetWorld()` overrides that return an arbitrary global world.

---

## 46. Use Object Names and Paths as Identity Carefully

### Recommendation

Treat object names and paths as engine identity mechanisms, not arbitrary mutable labels.

### Classification

**Official object-system behavior**

### Why

Names and outers form object paths used by:

- serialization;
- asset lookup;
- duplication;
- references;
- packages;
- editor tooling.

### Apply When

Use stable paths for assets and soft references.

Use generated unique names for transient instances unless semantic names are required.

### Avoid

Do not assume a short object name is globally unique.

Do not use object paths as long-term gameplay identifiers when a domain-specific stable ID is more appropriate.

---

## 47. Use Core Redirects for Supported Reflected Renames

### Recommendation

Use Core Redirects when renaming reflected classes, structs, enums, functions, properties, or packages in ways supported by the redirect system.

### Classification

**Official Epic migration guidance**

### Why

Serialized assets may retain old reflected names.

Redirects can map old identifiers to new identifiers during loading.

### Apply When

Use redirects for intentional API and content migrations.

Then:

- load affected assets;
- resave them;
- validate references;
- remove redirects only when migration policy permits.

### Avoid

Do not assume redirects solve arbitrary semantic or type-shape changes.

Complex migrations may require custom versioning or conversion code.

---

## 48. Keep Deprecated Reflected APIs Usable During Migration

### Recommendation

Deprecate reflected APIs with a clear replacement path before removal when project content needs time to migrate.

### Classification

**Epic pattern**

### Why

Blueprints and serialized assets may depend on exposed functions and properties.

A deprecation period can:

- preserve loading;
- emit warnings;
- identify consumers;
- communicate replacements;
- support staged migration.

### Avoid

Do not leave deprecated APIs indefinitely without ownership and a removal plan.

Do not remove them before verifying content migration.

---

## 49. Test Reflection Changes in Cooked Builds

### Recommendation

Validate major reflection and serialization changes outside editor-only workflows.

### Classification

**Context-dependent guidance**

### Why

Editor success does not guarantee:

- successful cooking;
- correct stripped-data behavior;
- valid generated code;
- correct asset inclusion;
- runtime soft-reference availability;
- serialization compatibility;
- shipping configuration behavior.

### Apply When

Test cooking or packaging after changes involving:

- reflected type names;
- object references;
- editor-only properties;
- asset classes;
- soft paths;
- instanced subobjects;
- replication;
- custom serialization.

---

## 50. Reflection and UObject Review Checklist

### Type Selection

- Does this type genuinely need UObject identity or engine-managed services?
- Would a plain C++ type or USTRUCT be simpler?
- Is value semantics more appropriate than object identity?
- Is the selected framework type correct for world presence and lifecycle?

### Reflection

- Does every reflected declaration require reflection?
- Are native-only implementation details kept unreflected?
- Are UHT-supported types and declaration patterns used?
- Is the generated header included correctly?

### Construction

- Is the constructor limited to defaults and default subobjects?
- Could the constructor be running for a CDO?
- Is runtime behavior placed in a later lifecycle phase?
- Are runtime objects created with `NewObject` or the appropriate engine API?
- Are default subobjects created only with `CreateDefaultSubobject`?

### Ownership and GC

- Which object or system keeps each UObject alive?
- Is every persistent strong reference visible to GC?
- Are `UPROPERTY TObjectPtr` fields used where appropriate?
- Are weak references used for observers and back-references?
- Are soft references used only when deferred loading is intentional?
- Is `TStrongObjectPtr` justified?
- Is `AddToRoot` avoided or paired with explicit removal?
- Are custom reference collectors complete?

### Properties

- Is editing scope as narrow as possible?
- Are Editor editability and Blueprint mutability decided independently?
- Is runtime state separated from defaults and configuration?
- Are transient and serialized fields classified correctly?
- Are instanced subobjects truly owner-specific objects?
- Are runtime invariants enforced in code rather than metadata alone?

### Serialization and Migration

- Could this member exist in saved assets, maps, defaults, or Blueprints?
- Does a rename require a Core Redirect?
- Does a type change require migration logic?
- Have affected assets been compiled, loaded, validated, and resaved?
- Has cooking been tested?

### Lifecycle

- Does each initialization step run for new, loaded, and duplicated objects as intended?
- Are CDOs excluded from runtime-only behavior?
- Are external resources released explicitly?
- Is cleanup performed before unpredictable GC destruction where necessary?
- Are world and authority assumptions valid?

### Blueprint and Editor

- Is the reflected API clear and intentional?
- Are categories, tooltips, pin names, and access levels appropriate?
- Are required asset contracts validated?
- Are deprecated APIs accompanied by migration guidance?

### Networking

- Is replication explicitly configured rather than assumed?
- Is the UObject owned and registered by an appropriate networked object?
- Are authority, lifetime, and late-join behavior defined?
- Are replicated subobjects created and destroyed consistently?

### Performance

- Is UObject count justified?
- Are high-volume value records implemented as structs or native data where possible?
- Has GC cost been profiled?
- Are transient object churn and reference-graph size controlled?

---

## Summary

Unreal's reflection and UObject systems are foundational engine technologies, but they should be used intentionally.

The key rules are:

- standard C++ remains valid and often preferable for native-only implementation;
- UHT reflection is opt-in and imposes declaration constraints;
- UObjects must be created through Unreal APIs, never standard `new` or `delete`;
- constructors define defaults and default subobjects rather than runtime gameplay;
- every class has a Class Default Object whose behavior affects construction and defaults;
- persistent UObject ownership must be visible to the garbage collector;
- `UPROPERTY TObjectPtr` is the normal reflected strong-reference pattern;
- raw pointers are appropriate for short-lived observation;
- weak pointers represent non-ownership;
- soft pointers represent deferred or optional loading;
- `TStrongObjectPtr` and root-set manipulation require caution;
- USTRUCT is the preferred reflected value type;
- reflected properties are serialization and migration contracts;
- Editor metadata improves authoring but does not replace runtime validation;
- lifecycle, world context, authority, duplication, and destruction must be explicit;
- UObject identity should be introduced only when the domain requires it.

The objective is not to maximize reflection.

The objective is to expose exactly the information and behavior Unreal Engine must manage while keeping ownership, lifetime, data persistence, and public contracts understandable.

---

## Primary Sources

- [Reflection System](https://dev.epicgames.com/documentation/unreal-engine/reflection-system-in-unreal-engine)
- [Objects](https://dev.epicgames.com/documentation/unreal-engine/objects-in-unreal-engine)
- [UObject API](https://dev.epicgames.com/documentation/unreal-engine/API/Runtime/CoreUObject/UObject)
- [Object Pointers](https://dev.epicgames.com/documentation/unreal-engine/object-pointers-in-unreal-engine)
- [Properties](https://dev.epicgames.com/documentation/unreal-engine/unreal-engine-uproperties)
- [Structs](https://dev.epicgames.com/documentation/unreal-engine/structs-in-unreal-engine)
- [Metadata Specifiers](https://dev.epicgames.com/documentation/unreal-engine/metadata-specifiers-in-unreal-engine)
- [FSoftObjectPtr](https://dev.epicgames.com/documentation/unreal-engine/API/Runtime/CoreUObject/FSoftObjectPtr)
- [Incremental Garbage Collection](https://dev.epicgames.com/documentation/unreal-engine/incremental-garbage-collection-in-unreal-engine)
- [Core Redirects](https://dev.epicgames.com/documentation/unreal-engine/core-redirects-in-unreal-engine)
