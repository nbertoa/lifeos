# Lifecycle and Ownership in Unreal Engine

## Research Metadata

- **Research date:** 2026-07-22
- **Documentation baseline:** Unreal Engine 5.8
- **Primary authority:** Epic Games documentation and Unreal Engine API documentation
- **Scope:** UObject, Actor, Component, world, player, and travel lifecycles
- **Status:** Living document

---

## Purpose

Unreal Engine objects do not all share the same creation, initialization, ownership, destruction, or persistence rules.

Correct lifecycle design requires understanding:

- How an object is created.
- What conceptually owns it.
- Which references keep it reachable.
- When it becomes safe to use.
- Which callbacks may occur.
- How it should release resources.
- Whether it survives Actor destruction, Pawn replacement, level unloading, map travel, or application shutdown.
- Whether it exists independently on the server and each client.
- Whether asynchronous operations can outlive it.

The central principle is:

> Object existence does not imply object readiness, and object reachability does not automatically imply correct
> conceptual ownership.

---

## 1. Distinguish Conceptual Ownership From Garbage Collection Reachability

### Recommendation

Define conceptual ownership independently from the references that keep a UObject reachable by garbage collection.

### Classification

**Inference based on Epic object-handling documentation**

### Why

Several relationships are often described casually as ownership:

- An object's `Outer`.
- An Actor owning a Component.
- A `UPROPERTY` referencing another UObject.
- A container holding object pointers.
- A world containing Actors.
- A Controller possessing a Pawn.

These relationships do not all mean the same thing.

A useful architectural distinction is:

#### Conceptual ownership

The object or system responsible for:

- Creating the object.
- Initializing it.
- Defining its valid lifetime.
- Coordinating its use.
- Releasing external resources.
- Removing references when it is no longer needed.

#### Reachability ownership

References recognized by Unreal's garbage collector that prevent a UObject from becoming unreachable.

An object can be reachable without having a clear conceptual owner.

A system can also conceptually own an object while failing to expose the reference correctly to garbage collection.

### Questions

For every dynamically created UObject, document:

- Who creates it?
- Who decides when it is no longer needed?
- What reference keeps it reachable?
- Is that reference strong or weak?
- What is its `Outer`?
- Does the object need access to a world?
- Does it hold resources requiring explicit cleanup?
- Can asynchronous callbacks reference it after its intended lifetime?

### Validation

- Remove all intended strong references and verify collection.
- Retain the intended owner and verify the object remains valid.
- Destroy the conceptual owner and verify no stale callbacks remain.
- Use garbage-collection debugging tools when lifetime behavior is unclear.

---

## 2. Create UObjects Through Unreal's Object System

### Recommendation

Create UObject-derived instances using Unreal-supported creation functions rather than raw C++ allocation.

### Classification

**Official requirement**

### Why

UObjects participate in systems including:

- Reflection.
- Naming.
- Object flags.
- Serialization.
- Archetypes.
- Default objects.
- Garbage collection.
- Editor duplication.
- Package ownership.

Raw allocation does not establish the Unreal object metadata required by these systems.

### Implementation Guidance

Use mechanisms appropriate to the context:

- `NewObject` for dynamically created UObject instances.
- `CreateDefaultSubobject` for default subobjects created in a class constructor.
- `DuplicateObject` when duplicating an existing UObject.
- `LoadObject` or supported asset-loading systems for assets.
- `SpawnActor` for Actors.
- `NewObject` followed by appropriate registration for dynamically created Components.

### Example

```cpp
UExampleRuntimeObject* RuntimeObject =
    NewObject<UExampleRuntimeObject>(Owner);
```

Do not use:

```cpp
UExampleRuntimeObject* RuntimeObject =
    new UExampleRuntimeObject();
```

### Avoid

- Allocating UObjects with `new`.
- Freeing UObjects with `delete`.
- Treating UObject construction like ordinary C++ object construction.
- Creating UObject instances during static initialization.
- Creating runtime instances through a class default object.

### Sources

- [Creating Objects in Unreal Engine](https://dev.epicgames.com/documentation/unreal-engine/creating-objects-in-unreal-engine)
- [Objects in Unreal Engine](https://dev.epicgames.com/documentation/unreal-engine/objects-in-unreal-engine)

---

## 3. Understand the Class Default Object

### Recommendation

Treat the Class Default Object as a template containing class defaults, not as a normal runtime gameplay instance.

### Classification

**Official engine behavior**

### Why

Every reflected Unreal class has a Class Default Object, commonly called the CDO.

The CDO is used for purposes including:

- Default property values.
- Class archetype behavior.
- Object construction.
- Serialization comparisons.
- Blueprint class defaults.
- Editor workflows.

Constructors run while creating the CDO and may also run for other instances.

This means a UObject or Actor constructor can execute:

- Before gameplay begins.
- Without a valid gameplay world.
- During editor loading.
- During Blueprint compilation or reconstruction.
- For template objects rather than live runtime instances.

### Implementation Guidance

Use constructors primarily to:

- Assign deterministic default values.
- Create default subobjects.
- Configure default subobject properties.
- Establish class-level defaults.

Do not assume constructors have:

- A valid gameplay world.
- A local player.
- A GameMode.
- A network connection.
- Fully initialized Components.
- Loaded optional assets.
- Completed Blueprint property initialization.

### Avoid

- Starting timers in constructors.
- Querying world Actors in constructors.
- Executing gameplay logic in constructors.
- Binding to runtime services in constructors.
- Performing network-dependent initialization in constructors.
- Mutating shared assets through class defaults.

---

## 4. Use `Outer` Intentionally

### Recommendation

Assign an `Outer` that accurately represents the object's naming, containment, serialization, and contextual
relationship.

### Classification

**Official engine concept with context-dependent lifetime implications**

### Why

Every UObject exists within an Outer chain.

The Outer participates in:

- The object's full path and name.
- Package and containment relationships.
- Contextual lookup.
- Duplication behavior.
- Serialization behavior.
- Access to typed outer objects.
- Certain object-lifetime and garbage-collection relationships.

However, `Outer` should not be treated as a universal substitute for a strong reference.

### Apply When

Choose an Outer that reflects where the object logically belongs.

Examples:

- An Actor Component normally belongs to its owning Actor.
- A runtime helper object may belong to the system managing it.
- An asset belongs to a package.
- A subobject belongs to the object whose structure it forms part of.

### Important Distinction

The Outer answers:

> Within which Unreal object context does this object exist?

It does not by itself answer every question about:

- Runtime ownership.
- Reachability.
- Cleanup.
- Replication.
- Save persistence.
- Map-travel persistence.

### Avoid

- Passing `GetTransientPackage()` merely because the correct Outer is unclear.
- Choosing an extremely long-lived Outer to prevent collection.
- Assuming the Outer automatically acts like `std::unique_ptr`.
- Reparenting object concepts casually through inappropriate Outers.
- Using Outer chains as hidden service-location mechanisms.

### Validation

- Inspect the object's full path.
- Test duplication and editor reconstruction.
- Test garbage collection.
- Verify that package and save behavior match expectations.
- Verify that the Outer will not disappear earlier than the object can safely handle.

---

## 5. Use Strong UObject References for Intended Retention

### Recommendation

Store strong UObject references in forms visible to Unreal's garbage collector when the referencing object intends to
keep the target alive.

### Classification

**Official garbage-collection requirement**

### Typical Strong References

Within reflected UObject types, strong references commonly use:

```cpp
UPROPERTY()
TObjectPtr<UExampleObject> ExampleObject;
```

Containers containing UObject references should also be visible to reflection where required:

```cpp
UPROPERTY()
TArray<TObjectPtr<UExampleObject>> Objects;
```

### Why

Unreal's garbage collector determines which UObjects remain reachable by traversing known references from roots and
other reachable objects.

A raw pointer that is not reported to the collector may not protect a UObject from collection.

### Implementation Guidance

- Use `TObjectPtr` for UObject member references in supported engine code patterns.
- Mark reflected UObject references with the appropriate `UPROPERTY`.
- Use supported containers containing reflected object pointer types.
- Clear strong references when retention is no longer intended.
- Use `AddReferencedObjects` or `FGCObject` only for specialized non-UObject ownership scenarios.

### Avoid

- Assuming any raw pointer keeps a UObject alive.
- Keeping obsolete strong references indefinitely.
- Using `AddToRoot` as a routine ownership mechanism.
- Retaining whole object graphs accidentally through one long-lived manager.
- Holding strong references from persistent systems to world objects without cleanup.

---

## 6. Use Weak References for Observation Without Ownership

### Recommendation

Use weak object references when code needs to observe a UObject without extending its lifetime.

### Classification

**Official engine pattern**

### Typical Form

```cpp
TWeakObjectPtr<AActor> ObservedActor;
```

### Apply When

Weak references are appropriate for:

- Cached world-object lookups.
- Optional targets.
- Observer relationships.
- Asynchronous operations that must not retain the target.
- References from longer-lived systems to shorter-lived objects.
- Avoiding strong-reference cycles or unintended retention.

### Implementation Guidance

Validate before use:

```cpp
if (AActor* Actor = ObservedActor.Get())
{
    Actor->PerformAction();
}
```

Do not assume validity remains unchanged after:

- Latent operations.
- Async callbacks.
- Network updates.
- Level unloading.
- Actor destruction.
- Garbage collection.
- Deferred execution.

### Avoid

- Using a weak reference where the caller genuinely owns the object.
- Repeatedly dereferencing without checking validity.
- Treating a weak reference as thread-safe access to gameplay objects.
- Assuming weak references survive object replacement or seamless travel.
- Capturing a raw pointer in a callback after first validating a weak pointer.

---

## 7. Use Soft References for Asset Identity and Deferred Loading

### Recommendation

Use soft object or class references when an asset should be identified without requiring it to be loaded immediately.

### Classification

**Official asset-loading guidance**

### Typical Forms

```cpp
TSoftObjectPtr<UTexture2D> Icon;
TSoftClassPtr<AActor> ActorClass;
```

### Why

A soft reference contains an asset path and can refer to an asset that is not currently loaded.

It is therefore useful for:

- Optional content.
- Large configurable content libraries.
- Asynchronous loading.
- Avoiding unnecessary hard-reference dependency chains.
- Asset Manager workflows.
- Data-driven class selection.

### Important Distinction

A soft reference is not equivalent to a weak reference.

- A weak reference refers to a currently existing UObject without retaining it.
- A soft reference can identify an unloaded asset by path.
- A resolved soft reference does not automatically guarantee permanent retention.
- Loading policy must still be defined.

### Avoid

- Calling synchronous load operations during latency-sensitive gameplay without measurement.
- Replacing every hard reference with a soft reference.
- Dereferencing a soft reference before checking or loading it.
- Assuming soft references automatically manage unloading.
- Ignoring cooking rules for dynamically referenced content.

### Sources

- [Asynchronous Asset Loading](https://dev.epicgames.com/documentation/unreal-engine/asynchronous-asset-loading-in-unreal-engine)
- [Asset Management](https://dev.epicgames.com/documentation/unreal-engine/asset-management-in-unreal-engine)

---

## 8. Use Non-UObject Smart Pointers Only for Non-UObject Types

### Recommendation

Use standard Unreal smart pointers for ordinary C++ types, but not as a replacement for UObject reference types.

### Classification

**Official pointer-model distinction**

### Appropriate Types

For non-UObject data, Unreal provides types such as:

- `TUniquePtr`.
- `TSharedPtr`.
- `TSharedRef`.
- `TWeakPtr`.

These belong to a reference-counted or unique-ownership model separate from UObject garbage collection.

### Avoid

Do not place UObjects in:

- `TSharedPtr`.
- `TSharedRef`.
- `TUniquePtr`.

UObjects must use the Unreal object system and garbage-collection-aware pointer semantics.

### Principle

> Reference counting and UObject garbage collection are separate lifetime systems.

---

## 9. Avoid Routine Use of `AddToRoot`

### Recommendation

Do not use root-set retention as a default solution for object lifetime.

### Classification

**Epic pattern and inference from garbage-collection design**

### Why

Adding an object to the root set prevents normal garbage collection until it is removed.

This can be useful for narrowly controlled engine-level cases, but routine use creates risks:

- Memory leaks.
- Objects surviving beyond their valid world.
- Stale world references.
- Shutdown-order problems.
- Missing `RemoveFromRoot` calls.
- Hidden global lifetime.

### Prefer

- A correctly scoped owner.
- A reflected strong reference.
- A Subsystem with the intended lifetime.
- A managed asset-loading handle.
- Explicit non-UObject reference reporting where required.

### Apply Only When

- Root lifetime is truly intended.
- Removal is deterministic.
- The implementation owns both root addition and removal.
- Engine shutdown and editor workflows are considered.

---

## 10. Report UObject References Held by Non-UObject Owners

### Recommendation

When a non-UObject C++ object retains UObjects, use an Unreal-supported reference-reporting mechanism.

### Classification

**Official garbage-collection mechanism**

### Options

Depending on architecture, use:

- `FGCObject`.
- `AddReferencedObjects`.
- Ownership through a UObject that exposes `UPROPERTY` references.
- Another engine-supported collector integration.

### Why

The garbage collector cannot infer references stored in arbitrary native C++ structures.

### Avoid

- Hiding UObject pointers inside native classes without reporting them.
- Using `AddToRoot` instead of modeling proper retention.
- Assuming a `TSharedPtr` owner keeps a referenced UObject alive.
- Forgetting to unregister or destroy native holders before shutdown.

---

## 11. Treat Actor Construction and Gameplay Initialization Separately

### Recommendation

Do not place all Actor initialization in the constructor.

### Classification

**Official Actor lifecycle guidance**

### Why

Actor creation can follow different paths:

- Loading from a level.
- Play In Editor duplication.
- Deferred spawning.
- Immediate runtime spawning.
- Blueprint construction.
- Network creation.
- Editor reconstruction.

These paths share some callbacks but not necessarily identical context or timing.

### Constructor Responsibilities

The Actor constructor is appropriate for:

- Default values.
- Default Components.
- Default attachment hierarchy.
- Tick defaults.
- Replication defaults.
- Class-wide configuration.

### Runtime Initialization Responsibilities

Later callbacks are appropriate for:

- World-dependent setup.
- Binding to runtime services.
- Reading resolved player or game state.
- Starting timers.
- Registering gameplay events.
- Initializing based on dynamically assigned spawn data.
- Beginning active gameplay behavior.

### Sources

- [Unreal Engine Actor Lifecycle](https://dev.epicgames.com/documentation/unreal-engine/unreal-engine-actor-lifecycle)

---

## 12. Understand Major Actor Lifecycle Paths

### Recommendation

Account for the Actor's creation path when selecting initialization callbacks.

### Classification

**Official lifecycle documentation**

### Level-Loaded Actors

Actors saved into a level are loaded and initialized as part of world and level startup.

Relevant stages include:

- Loading and serialization.
- Component registration.
- Component initialization.
- `PostInitializeComponents`.
- `BeginPlay`.

### Spawned Actors

Runtime spawning includes stages such as:

- Actor allocation and construction.
- Component creation.
- Construction-script execution.
- Component registration.
- Spawn initialization.
- `PostInitializeComponents`.
- `BeginPlay`.

### Deferred Spawn

Deferred spawning allows properties and dependencies to be assigned before final construction completes.

Typical flow:

```cpp
AExampleActor* Actor = World->SpawnActorDeferred<AExampleActor>(
    ActorClass,
    Transform,
    Owner,
    Instigator);

Actor->ConfigureBeforeConstruction(Configuration);

UGameplayStatics::FinishSpawningActor(Actor, Transform);
```

### Apply Deferred Spawn When

- Construction depends on caller-provided data.
- Properties must be set before construction-script execution.
- Components or defaults need pre-finalization configuration.
- Immediate spawn would expose a partially configured Actor.

### Avoid

- Forgetting to finish deferred spawning.
- Publishing references to an Actor before spawn finalization.
- Starting behavior before required configuration exists.
- Assuming `BeginPlay` always occurs immediately after allocation.

---

## 13. Use `OnConstruction` for Construction-Time Representation Carefully

### Recommendation

Use Actor construction logic for authored or spawn-time setup that can safely run repeatedly.

### Classification

**Official Actor lifecycle mechanism**

### Why

Construction scripts and `OnConstruction` may execute:

- When an Actor is spawned.
- When properties change in the editor.
- During editor reconstruction.
- During Blueprint compilation-related reconstruction.
- More often than gameplay programmers initially expect.

### Appropriate Uses

- Updating editor-visible representation.
- Configuring default Components from editable properties.
- Building deterministic construction-time arrangements.
- Applying spawn-time configuration before gameplay begins.

### Requirements

Construction logic should be:

- Deterministic.
- Safe to rerun.
- Free of irreversible gameplay side effects.
- Careful about dynamically created Components.
- Efficient enough for editor use.

### Avoid

- Awarding gameplay rewards.
- Registering persistent global listeners repeatedly.
- Starting network requests.
- Saving runtime progress.
- Spawning uncontrolled permanent Actors.
- Assuming construction runs only once.

---

## 14. Register Dynamic Components Correctly

### Recommendation

Create, attach, register, initialize, and destroy runtime Components using the appropriate Component lifecycle.

### Classification

**Official Component behavior**

### Typical Runtime Creation

A dynamically created Component commonly requires:

```cpp
UExampleComponent* Component =
    NewObject<UExampleComponent>(OwnerActor);

Component->RegisterComponent();
```

For Scene Components, attachment and transform setup must also be handled deliberately.

Depending on context, additional ownership or instance-component registration may be needed for editor persistence or
serialization workflows.

### Component Lifecycle Concepts

A Component may pass through:

- Creation.
- Attachment.
- Registration.
- Initialization.
- Activation.
- `BeginPlay`.
- Deactivation.
- Unregistration.
- Destruction.

These states are related but not identical.

### Important Distinctions

#### Registered

The Component participates in its world-facing engine systems.

#### Initialized

The Component has executed initialization behavior when configured to do so.

#### Active

The Component is currently enabled according to its activation model.

#### Begun Play

The owning Actor and Component have entered gameplay.

### Avoid

- Assuming `NewObject` alone makes a Component operational.
- Attaching registered Scene Components using construction-only APIs.
- Registering the same Component repeatedly.
- Leaving dynamic Components registered after their purpose ends.
- Depending on `BeginPlay` for Components created after the Actor already began play without testing that path.
- Treating activation as creation or destruction.

### Sources

- [Components in Unreal Engine](https://dev.epicgames.com/documentation/unreal-engine/components-in-unreal-engine)
- [UActorComponent API](https://dev.epicgames.com/documentation/unreal-engine/API/Runtime/Engine/UActorComponent)

---

## 15. Use Component Initialization Callbacks According to Their Meaning

### Recommendation

Select Component callbacks according to whether work requires registration, initialization, gameplay start, or
activation.

### Classification

**Official engine lifecycle**

### Common Callbacks

| Callback                 | Intended Context                                                          |
|--------------------------|---------------------------------------------------------------------------|
| `OnRegister`             | Component has been registered and may create or update world-facing state |
| `InitializeComponent`    | Gameplay initialization for Components opting into initialization         |
| `BeginPlay`              | Owning Actor has entered gameplay                                         |
| `OnComponentActivated`   | Component becomes active                                                  |
| `OnComponentDeactivated` | Component becomes inactive                                                |
| `OnUnregister`           | Component leaves registration                                             |
| `OnComponentDestroyed`   | Component is being destroyed                                              |

### Guidance

Use:

- `OnRegister` for registration-dependent setup that can safely repeat.
- `InitializeComponent` for one-time gameplay-session initialization where supported.
- `BeginPlay` for behavior requiring gameplay to have started.
- Activation callbacks for temporary enabled and disabled states.
- Unregistration and destruction callbacks for symmetrical cleanup.

### Avoid

- Binding the same delegate every time `OnRegister` runs without guarding or unbinding.
- Treating `OnUnregister` as permanent object destruction.
- Starting permanent gameplay logic in editor registration.
- Assuming registration occurs only once.
- Forgetting that dynamic Components can be created after Actor `BeginPlay`.

---

## 16. Use `PostInitializeComponents` for Fully Initialized Actor Components

### Recommendation

Use `PostInitializeComponents` when Actor logic requires its Components to have completed initialization.

### Classification

**Official Actor lifecycle mechanism**

### Apply When

The Actor needs to:

- Establish relationships among initialized Components.
- Cache required Component interfaces.
- Validate Component configuration.
- Complete setup before gameplay begins.
- Execute setup in gameplay and selected preview contexts.

### Avoid

- Assuming all replicated references are already available.
- Assuming possession has occurred.
- Assuming all asynchronous content is loaded.
- Starting player-specific logic before player context exists.
- Using it as a substitute for explicit dependency readiness.

---

## 17. Use `BeginPlay` for Gameplay Start, Not Universal Readiness

### Recommendation

Use `BeginPlay` for behavior that should start when an Actor enters gameplay, but do not assume every external
dependency is ready at that moment.

### Classification

**Official lifecycle behavior**

### Appropriate Uses

- Starting gameplay timers.
- Enabling active gameplay behavior.
- Registering with world systems.
- Beginning AI or simulation logic.
- Performing setup requiring an active gameplay world.

### Readiness Limitations

At `BeginPlay`, an Actor may still be waiting for:

- Replicated properties.
- A PlayerState.
- Possession-related dependencies.
- Async-loaded assets.
- Online-service responses.
- Feature-plugin activation.
- Other independently initialized systems.

Networking and child-Actor conditions can also affect when `BeginPlay` occurs.

### Guidance

- Make readiness requirements explicit.
- Use `OnRep` functions for replicated dependency arrival.
- Use possession callbacks for possession-dependent setup.
- Use delegates for asynchronous completion.
- Use initialization-state systems for complex modular dependencies.
- Make repeated readiness evaluation idempotent.

### Principle

> `BeginPlay` means gameplay has begun for this object; it does not mean the entire game has completed every form of
> initialization.

---

## 18. Separate Possession From Pawn Initialization

### Recommendation

Treat Pawn creation, controller assignment, possession, PlayerState availability, and local input setup as distinct
lifecycle events.

### Classification

**Official Gameplay Framework behavior**

### Why

A Pawn can:

- Exist without a Controller.
- Be possessed after `BeginPlay`.
- Change Controllers.
- Be respawned.
- Be replaced.
- Be controlled by AI or a player.
- Receive replicated Controller information at different times on clients.
- Have PlayerState relationships become available later.

### Relevant Hooks

Depending on authority and role, architecture may use:

- `PossessedBy`.
- `UnPossessed`.
- Controller possession callbacks.
- `OnRep_Controller`.
- `OnRep_PlayerState`.
- Pawn restart callbacks.
- Local-player initialization callbacks.

### Avoid

- Initializing all player-dependent systems only in Pawn `BeginPlay`.
- Assuming a PlayerController exists on every client.
- Storing participant persistence only on the Pawn.
- Binding input without handling possession changes.
- Retaining stale Controller, Pawn, or PlayerState references after replacement.

### Validation

Test:

- Initial spawn.
- Death and respawn.
- Spectating.
- Pawn swapping.
- AI possession.
- Listen server.
- Dedicated server.
- Owning and non-owning clients.
- Seamless travel.

---

## 19. React to Replicated Data in `OnRep` Functions

### Recommendation

Use replication notifications when client-side behavior depends on the arrival or change of replicated state.

### Classification

**Official replication mechanism**

### Why

Replicated properties may not be available during construction, initialization, or even early gameplay callbacks.

`OnRep` functions provide an intentional point to respond when clients receive a replicated value.

### Implementation Guidance

- Keep the replicated property authoritative on the server.
- Make the `OnRep` handler safe for the client's current lifecycle state.
- Consider whether the server needs to call equivalent local handling explicitly.
- Avoid assuming related replicated properties arrive in one guaranteed order.
- Make handlers idempotent where practical.
- Separate state application from presentation where useful.

### Avoid

- Polling every frame for replicated data.
- Treating `BeginPlay` as proof that replication is complete.
- Triggering irreversible behavior from an `OnRep` without considering repeated updates.
- Depending on replication notification for server-only execution.

---

## 20. Model Multi-System Readiness Explicitly

### Recommendation

Use explicit readiness states when a feature depends on several asynchronously initialized systems.

### Classification

**Epic pattern demonstrated by Modular Gameplay**

### Why

Complex features may require combinations such as:

- Pawn spawned.
- Controller assigned.
- PlayerState replicated.
- Ability System initialized.
- Input Component available.
- Game Feature activated.
- Required assets loaded.
- Team data assigned.
- UI layer created.

A single lifecycle callback cannot accurately represent every combination.

### Possible State Model

```text
Spawned
    ↓
DataAvailable
    ↓
Initialized
    ↓
GameplayReady
```

### Requirements

- Define prerequisites for each state.
- Allow state checks to run repeatedly.
- Advance only when all prerequisites are satisfied.
- Prevent duplicate initialization.
- Handle dependency removal or feature deactivation.
- Log blocked transitions during development.

### Apply When

- Multiple modular Components initialize independently.
- Network replication affects readiness.
- Game Feature Plugins add runtime capabilities.
- Pawn and player systems must coordinate.
- Late activation is supported.

### Avoid

- Creating a complex state machine for a simple isolated object.
- Advancing states through timing assumptions.
- Hiding missing dependencies by delaying arbitrary numbers of frames.
- Treating initialization states as substitutes for clear ownership.

### Sources

- [Game Framework Component Manager](https://dev.epicgames.com/documentation/unreal-engine/game-framework-component-manager-in-unreal-engine)

---

## 21. Clean Up Symmetrically

### Recommendation

Every runtime registration, binding, timer, asynchronous operation, and external resource should have a corresponding
cleanup strategy.

### Classification

**Epic engine pattern**

### Resources Requiring Cleanup May Include

- Delegate bindings.
- Timers.
- Async callbacks.
- Streamable handles.
- Online requests.
- Input mapping contexts.
- Gameplay message listeners.
- Subsystem registrations.
- Component registrations.
- Dynamically spawned Actors.
- Native resources.
- File or socket handles.
- External SDK callbacks.

### Symmetry Examples

| Setup                    | Cleanup                          |
|--------------------------|----------------------------------|
| Bind delegate            | Unbind delegate                  |
| Start timer              | Clear timer                      |
| Register listener        | Unregister listener              |
| Add input context        | Remove input context             |
| Retain async handle      | Release or cancel handle         |
| Allocate native resource | Release native resource          |
| Spawn owned Actor        | Destroy it or transfer ownership |
| Register Component       | Unregister or destroy Component  |

### Avoid

- Assuming garbage collection cancels external operations.
- Allowing callbacks to execute against destroyed objects.
- Binding lambdas that capture raw object pointers without lifetime safeguards.
- Clearing resources only in destructors when earlier lifecycle callbacks are required.
- Performing cleanup in only one destruction path.

---

## 22. Use `EndPlay` for Actor Gameplay Teardown

### Recommendation

Use `EndPlay` for Actor cleanup that must occur whenever the Actor stops participating in gameplay.

### Classification

**Official Actor lifecycle guidance**

### Why

An Actor can end play for several reasons, including:

- Explicit destruction.
- Level transition.
- Level streaming removal.
- End of Play In Editor.
- Application shutdown.
- World cleanup.

`EndPlay` is therefore generally broader than handling only explicit `Destroy` calls.

### Appropriate Cleanup

- Clear timers.
- Unbind runtime delegates.
- Cancel or invalidate async requests.
- Remove registrations from systems.
- Release transient gameplay resources.
- Notify owned non-Actor services.
- Clear references held by longer-lived objects.

### Use the Reason

Inspect the `EEndPlayReason` when behavior legitimately differs according to why play is ending.

### Avoid

- Relying exclusively on `Destroyed` for all teardown.
- Starting new persistent work during teardown.
- Calling into already torn-down global systems without validity checks.
- Assuming callback order across unrelated Actors.
- Expecting garbage collection immediately after `EndPlay`.

### Sources

- [Unreal Engine Actor Lifecycle](https://dev.epicgames.com/documentation/unreal-engine/unreal-engine-actor-lifecycle)

---

## 23. Distinguish Actor Destruction From Garbage Collection

### Recommendation

Treat `AActor::Destroy` as the beginning of Actor removal, not equivalent to immediate C++ memory deletion.

### Classification

**Official Actor lifecycle behavior**

### Why

Actor destruction is deferred through Unreal's world and object lifecycle.

After destruction begins:

- The Actor should no longer be treated as usable gameplay state.
- References may still physically exist temporarily.
- End-play and destruction callbacks occur.
- Garbage collection later reclaims unreachable UObject memory.

### Implementation Guidance

- Use validity checks before acting on cached references.
- Remove references from long-lived owners.
- Do not manually delete Actors.
- Do not continue scheduling gameplay work after destruction begins.
- Use weak references for observational caches.
- Design callbacks to tolerate the target disappearing.

### Avoid

- Assuming a non-null raw pointer is safe.
- Calling methods on an Actor after initiating destruction.
- Waiting for C++ destruction to perform gameplay teardown.
- Retaining destroyed Actors through unintended strong references.

---

## 24. Use `IsValid` According to Context

### Recommendation

Validate UObject references before use when their lifetime is not guaranteed by the caller's current scope.

### Classification

**Official object-handling pattern**

### Why

A pointer can be non-null while its UObject is no longer valid for gameplay use.

Typical checks include:

```cpp
if (IsValid(Object))
{
    Object->PerformAction();
}
```

For weak pointers:

```cpp
if (UExampleObject* Object = WeakObject.Get())
{
    Object->PerformAction();
}
```

### Important Limitation

Validity checks are not a complete ownership model.

A check performed now does not guarantee validity after:

- Deferred execution.
- Awaiting asynchronous work.
- A future frame.
- A network callback.
- Level unloading.
- Another system destroying the object.

### Avoid

- Adding validity checks everywhere instead of fixing lifetime design.
- Assuming `IsValid` provides thread synchronization.
- Checking once and storing an unsafe raw pointer for later.
- Silently ignoring invalid critical dependencies without logging or recovery.

---

## 25. Use Destructors and UObject Teardown Hooks Correctly

### Recommendation

Release resources through the lifecycle hook appropriate to the type of resource and object.

### Classification

**Official engine pattern**

### UObject Teardown Concepts

Depending on object type and responsibility, cleanup may involve:

- Explicit shutdown methods.
- Owner lifecycle callbacks.
- `BeginDestroy`.
- `FinishDestroy`.
- Native C++ destructor.
- Garbage-collector reference reporting.
- Subsystem `Deinitialize`.
- Component unregistration or destruction.
- Actor `EndPlay`.

### Guidance

Use higher-level explicit teardown for gameplay relationships and external operations.

Use lower-level destruction hooks for resources that truly belong to final object destruction.

### Why

Garbage collection timing is not a deterministic gameplay scheduling mechanism.

Waiting for final object destruction may be too late to:

- Unregister from live systems.
- Cancel callbacks safely.
- Release scarce runtime resources promptly.
- Notify related gameplay objects.
- Remove UI or input state.

### Avoid

- Putting gameplay notifications in C++ destructors.
- Calling arbitrary UObjects during late destruction.
- Assuming world systems remain valid during final teardown.
- Depending on immediate garbage collection.
- Performing expensive blocking work during destruction.

---

## 26. Do Not Retain Stale World References Across Travel

### Recommendation

Long-lived objects must clear or reacquire references to world-scoped objects when worlds change.

### Classification

**Inference based on Unreal travel and world lifecycles**

### Why

Objects such as Game Instance and Game Instance Subsystems may survive map transitions while these do not necessarily
survive:

- World.
- GameMode.
- GameState.
- Level Actors.
- World Subsystems.
- World-owned Components.
- Timers belonging to the old world.
- UI tied to old world state.

### Implementation Guidance

Long-lived systems should:

- Avoid permanent strong references to world Actors.
- Listen for appropriate world or travel events when necessary.
- Release old-world registrations.
- Reacquire world-scoped dependencies.
- Distinguish current world from previous world.
- Handle travel failure.
- Avoid using cached world pointers after teardown.

### Avoid

- Storing arbitrary Actor pointers in Game Instance indefinitely.
- Assuming the same GameState survives travel.
- Treating map persistence as application persistence.
- Using delayed callbacks from the old world after the new world loads.

---

## 27. Treat Seamless Travel as an Explicit Persistence Policy

### Recommendation

Persist Actors across seamless travel only when their continuity is intentionally required.

### Classification

**Official travel mechanism**

### Why

Seamless travel can move selected Actors into the destination world without disconnecting clients.

Epic provides GameMode hooks for specifying additional Actors that should travel.

PlayerControllers and associated player framework state have specialized seamless-travel behavior.

### Apply When

- The player connection must continue through map transition.
- Match or session continuity requires selected framework state.
- A specific Actor has been designed to survive the transition.
- References and world-dependent resources can be safely reinitialized.

### Requirements

A traveling Actor must handle:

- A changing world context.
- Destination GameMode and GameState changes.
- World-owned dependency replacement.
- Component and subsystem re-registration as required.
- References to non-traveling Actors becoming invalid.
- Client and server transition behavior.

### Avoid

- Persisting large object graphs for convenience.
- Carrying destination-incompatible Actors.
- Assuming every reference contained by a traveling Actor remains valid.
- Using seamless travel as a replacement for explicit save or session state.
- Preserving Actors that can be reconstructed more safely.

### Sources

- [AGameModeBase API](https://dev.epicgames.com/documentation/unreal-engine/API/Runtime/Engine/AGameModeBase)
- [Travelling in Multiplayer](https://dev.epicgames.com/documentation/unreal-engine/travelling-in-multiplayer-in-unreal-engine)

---

## 28. Distinguish Travel Persistence From Saved Persistence

### Recommendation

Use separate mechanisms for map-travel continuity and persistence across application runs.

### Classification

**Official framework distinction**

### Travel Persistence

May involve:

- Game Instance.
- Game Instance Subsystems.
- Seamless-travel Actors.
- PlayerState copying or travel behavior.
- Session services.

### Saved Persistence

May involve:

- `USaveGame`.
- Platform save APIs.
- Backend services.
- Account services.
- Databases.
- Versioned serialized data.

### Avoid

- Treating Game Instance as permanent saved storage.
- Treating SaveGame objects as live authoritative runtime systems.
- Saving direct world-object pointers.
- Assuming seamless travel persists application state after shutdown.
- Persisting implementation details without a schema or version strategy.

---

## 29. Scope Subsystem Initialization and Deinitialization Correctly

### Recommendation

Use each Subsystem's initialization and deinitialization lifecycle as the boundary for its scoped resources.

### Classification

**Official Subsystem behavior**

### Guidance

A Subsystem should:

- Acquire scoped dependencies during or after initialization.
- Avoid using dependencies before their owning scope is ready.
- Register listeners deliberately.
- Release listeners and external resources during deinitialization.
- Avoid retaining objects from a destroyed scope.
- Account for editor environments containing several instances of its scope.

### Scope Examples

#### Engine Subsystem

Must consider:

- Editor lifetime.
- Multiple worlds.
- Application shutdown.
- Engine-wide tooling.

#### Game Instance Subsystem

Must consider:

- Map travel.
- Per-process networking.
- Session transitions.
- Old-world cleanup.

#### World Subsystem

Must consider:

- World type.
- Streaming and world teardown.
- Dedicated-server applicability.
- Editor preview worlds.

#### Local Player Subsystem

Must consider:

- Multiple local players.
- Local-player removal.
- Input and UI teardown.
- Client-only operation.

### Avoid

- Assuming one global instance in editor.
- Accessing world state before a valid world is available.
- Keeping stale Actor references after scope deinitialization.
- Omitting cleanup because Unreal destroys the Subsystem automatically.

### Sources

- [Programming Subsystems](https://dev.epicgames.com/documentation/unreal-engine/programming-subsystems-in-unreal-engine)

---

## 30. Make Delegate Binding Lifetime-Safe

### Recommendation

Select delegate-binding forms and unbinding strategies according to the lifetimes of publisher and subscriber.

### Classification

**Official delegate system with Epic usage patterns**

### Questions

Before binding, determine:

- Which object owns the binding?
- Which object may disappear first?
- Is the delegate dynamic or native?
- Does the binding retain the target?
- Can the event fire during teardown?
- Can registration happen more than once?
- Is explicit unbinding required?
- Can the callback be queued or delayed?

### Guidance

- Store handles for native delegate bindings when explicit removal is needed.
- Use UObject-aware binding forms for UObject targets.
- Avoid duplicate registration.
- Unbind during the subscriber's teardown.
- Keep callback work safe if the publisher is itself shutting down.
- Use weak captures for asynchronous lambdas when lifetime is not guaranteed.

### Dangerous Pattern

```cpp
Publisher->OnCompleted.AddLambda([this]()
{
    PerformAction();
});
```

This is unsafe when the callback can outlive `this`.

### Safer Direction

Capture a weak UObject reference and validate it at callback time, or use a UObject-aware delegate binding supported by
the delegate type.

### Avoid

- Capturing raw UObject pointers in long-lived callbacks.
- Losing delegate handles.
- Binding every time a Component registers without unbinding.
- Assuming dynamic delegates remove every possible native callback automatically.
- Broadcasting from partially destroyed objects without a clear contract.

---

## 31. Make Timers Lifetime-Safe

### Recommendation

Clear or invalidate timers when their target or owning gameplay context ends.

### Classification

**Epic runtime pattern**

### Guidance

- Store timer handles when cancellation is required.
- Clear timers during Actor or Component teardown.
- Avoid raw lambda captures that can outlive the target.
- Prefer UObject-bound timer delegates when suitable.
- Verify timer ownership when worlds transition.
- Treat timers as world-scoped.

### Avoid

- Assuming a timer automatically models gameplay ownership.
- Scheduling callbacks for an Actor that may be destroyed.
- Retaining Game Instance state through a timer belonging to an old World.
- Using timers to guess when another system will be initialized.

---

## 32. Make Asynchronous Work Lifetime-Safe

### Recommendation

Every asynchronous operation must define what happens when its requester, owner, world, or destination context
disappears.

### Classification

**Inference aligned with Epic asynchronous systems**

### Operations Include

- Asset loads.
- HTTP requests.
- Online-subsystem calls.
- AI or external-service calls.
- Async tasks.
- Background calculations.
- Save operations.
- Platform dialogs.
- Streaming operations.
- Delayed gameplay tasks.

### Questions

- Can the operation be cancelled?
- Who owns the cancellation handle?
- What happens if the caller is destroyed?
- Is callback execution guaranteed on the game thread?
- Does the result belong to the same world that requested it?
- Is the result still relevant after travel?
- Can requests complete out of order?
- How are duplicate or stale responses detected?

### Implementation Guidance

- Capture weak references where appropriate.
- Use request IDs or generation counters.
- Cancel work during teardown when supported.
- Ignore stale responses deliberately.
- Marshal UObject interaction to the game thread.
- Hold loaded assets through a documented retention policy.
- Log failures and cancellation separately.

### Avoid

- Capturing raw Actor pointers.
- Applying results to a replacement Pawn unintentionally.
- Treating callback arrival as proof the original context still exists.
- Blocking the game thread while waiting.
- Allowing old-world operations to mutate new-world state.

---

## 33. Do Not Assume Destruction Order Between Independent Objects

### Recommendation

Design cleanup so it does not depend on an undocumented destruction order among unrelated Actors, Components,
Subsystems, or services.

### Classification

**Inference from engine lifecycle complexity**

### Why

World teardown, editor shutdown, map travel, networking, and garbage collection can produce different relative teardown
sequences.

### Guidance

- Validate dependencies before use during cleanup.
- Let each object release its own registrations.
- Avoid requiring another object to remain fully operational during final teardown.
- Use explicit shutdown orchestration when order matters.
- Separate runtime shutdown from final garbage collection.
- Keep cleanup idempotent.

### Avoid

- Calling deeply into other systems from destructors.
- Assuming GameState still exists during every Actor's `EndPlay`.
- Assuming a Subsystem is available during final object destruction.
- Broadcasting required cleanup events after listeners may already be gone.

---

## 34. Make Cleanup Idempotent

### Recommendation

Cleanup functions should tolerate being called when some or all resources have already been released.

### Classification

**Defensive Epic-compatible engineering practice**

### Why

Teardown may be initiated through several paths:

- Explicit shutdown.
- Actor destruction.
- Level transition.
- Component deactivation.
- Feature deactivation.
- Request cancellation.
- World cleanup.
- Application shutdown.

### Implementation Guidance

- Check handles before clearing them.
- Reset handles after removal.
- Clear references after release.
- Track registration state.
- Guard against duplicate delegate removal where required.
- Avoid performing the same external side effect twice.

### Example

```cpp
void UExampleComponent::Shutdown()
{
    if (bIsRegisteredWithService)
    {
        Service->Unregister(this);
        bIsRegisteredWithService = false;
    }

    GetWorld()->GetTimerManager().ClearTimer(UpdateTimerHandle);
    ObservedActor.Reset();
}
```

---

## 35. Test Lifecycle Under Failure Conditions

### Recommendation

Test lifecycle behavior when initialization or teardown is interrupted.

### Classification

**Inference aligned with Epic testing practices**

### Required Failure Cases May Include

- Required asset fails to load.
- Actor is destroyed before async completion.
- Component is removed dynamically.
- Player disconnects during initialization.
- Travel begins while requests are active.
- Game Feature deactivates during gameplay.
- Server rejects a requested action.
- Pawn is replaced before setup finishes.
- World shuts down during a timer callback.
- Online service becomes unavailable.
- Save operation fails.
- Object is garbage-collected after references are removed.

### Why

Many lifecycle bugs occur outside the successful initialization path.

---

## Pointer Selection Guide

| Requirement                                     | Preferred Starting Point                                     |
|-------------------------------------------------|--------------------------------------------------------------|
| Strong UObject member reference                 | `UPROPERTY()` with `TObjectPtr<T>`                           |
| Non-owning reference to an existing UObject     | `TWeakObjectPtr<T>`                                          |
| Reference to a potentially unloaded asset       | `TSoftObjectPtr<T>`                                          |
| Reference to a potentially unloaded class asset | `TSoftClassPtr<T>`                                           |
| Non-UObject unique ownership                    | `TUniquePtr<T>`                                              |
| Non-UObject shared ownership                    | `TSharedPtr<T>` / `TSharedRef<T>`                            |
| Non-owning reference-counted native pointer     | `TWeakPtr<T>`                                                |
| Native owner reporting UObject references to GC | `FGCObject` or supported collector integration               |
| Default UObject subobject                       | `CreateDefaultSubobject<T>`                                  |
| Dynamic UObject                                 | `NewObject<T>`                                               |
| Dynamic Actor                                   | `SpawnActor<T>`                                              |
| Dynamic Component                               | `NewObject<T>` plus correct registration and ownership setup |

This table provides starting points rather than universal answers.

---

## Lifecycle Placement Guide

| Work                                              | Typical Earliest Appropriate Phase  |
|---------------------------------------------------|-------------------------------------|
| Assign class defaults                             | Constructor                         |
| Create default Components                         | Constructor                         |
| Define default attachments                        | Constructor                         |
| Apply repeatable construction-time representation | `OnConstruction`                    |
| Configure deferred-spawn properties               | Before `FinishSpawning`             |
| Work requiring registered Component state         | `OnRegister` or later               |
| Work requiring initialized Actor Components       | `PostInitializeComponents` or later |
| Start active gameplay behavior                    | `BeginPlay`                         |
| Initialize Controller-dependent Pawn behavior     | Possession callbacks                |
| React to replicated dependency arrival            | `OnRep`                             |
| React to asynchronously loaded content            | Completion callback                 |
| Stop gameplay participation                       | `EndPlay`                           |
| Deinitialize scoped service                       | Subsystem `Deinitialize`            |
| Release final native resources                    | Appropriate destruction hook        |

---

## Lifecycle Review Checklist

### Creation

- [ ] UObject instances use Unreal-supported creation functions.
- [ ] Actors use spawning APIs.
- [ ] Default subobjects use `CreateDefaultSubobject`.
- [ ] Dynamic Components are registered correctly.
- [ ] Deferred spawn is used when pre-construction configuration is required.
- [ ] Constructors contain only safe default initialization.

### Ownership

- [ ] The conceptual owner is documented.
- [ ] The reference keeping the object reachable is documented.
- [ ] The `Outer` is intentional.
- [ ] Strong references are visible to garbage collection.
- [ ] Weak references are used where retention is not intended.
- [ ] Non-UObject owners report UObject references correctly.

### Initialization

- [ ] Initialization prerequisites are explicit.
- [ ] `BeginPlay` is not treated as universal readiness.
- [ ] Possession-dependent setup handles possession changes.
- [ ] Replicated dependencies use appropriate notifications.
- [ ] Async dependencies use completion callbacks.
- [ ] Initialization is idempotent where callbacks may repeat.
- [ ] Construction scripts are safe to rerun.

### Components

- [ ] Component registration is distinct from creation.
- [ ] Activation is distinct from registration.
- [ ] Registration callbacks tolerate repeated execution.
- [ ] Dynamic Components handle Actors that already began play.
- [ ] Scene Component attachments use the correct lifecycle API.
- [ ] Component cleanup is symmetrical.

### References

- [ ] `TObjectPtr` or another appropriate strong form is used for retained UObjects.
- [ ] Weak object references are validated at use time.
- [ ] Soft references have a defined loading and retention policy.
- [ ] UObjects are not stored in reference-counted smart pointers.
- [ ] Root-set retention is avoided unless explicitly justified.
- [ ] Long-lived systems do not retain obsolete world objects.

### Gameplay Framework

- [ ] Pawn initialization does not assume immediate possession.
- [ ] PlayerState availability is handled independently.
- [ ] Respawn and Pawn replacement are tested.
- [ ] Local-player systems are scoped to the correct local player.
- [ ] Dedicated-server execution does not require local-only objects.

### Cleanup

- [ ] Every delegate binding has an unbinding strategy.
- [ ] Every timer has a cancellation strategy.
- [ ] Async work handles requester destruction.
- [ ] External resources are explicitly released.
- [ ] `EndPlay` handles all relevant end-play reasons.
- [ ] Cleanup is idempotent.
- [ ] Final destruction does not contain ordinary gameplay orchestration.

### Travel

- [ ] Map-travel persistence requirements are explicit.
- [ ] Long-lived systems clear old-world references.
- [ ] Seamless-travel Actors are intentionally selected.
- [ ] Traveling Actors reacquire destination-world dependencies.
- [ ] Travel persistence and SaveGame persistence are separated.
- [ ] Active async work is cancelled or invalidated during travel.

### Networking

- [ ] Replicated properties are not assumed to exist during construction.
- [ ] Client setup reacts through replication callbacks where required.
- [ ] Server and client initialization paths are tested independently.
- [ ] Late joiners receive reconstructible state.
- [ ] Disconnect and reconnect behavior is tested.
- [ ] Client callbacks do not use destroyed or replaced Pawns.

### Testing

- [ ] Object collection has been tested after releasing references.
- [ ] Actor destruction during initialization has been tested.
- [ ] Component removal has been tested.
- [ ] World teardown has been tested.
- [ ] Seamless and non-seamless travel have been tested where applicable.
- [ ] Play In Editor with multiple worlds has been tested.
- [ ] Packaged-build lifecycle has been tested.
- [ ] Initialization failure and cancellation paths have been tested.

---

## Common Lifecycle Failure Modes

### Gameplay Logic in Constructors

The constructor queries the world, registers runtime listeners, or starts gameplay.

**Correction:** Restrict constructors to defaults and default subobjects; move runtime work into an appropriate
lifecycle phase.

### `BeginPlay` as Universal Initialization

A feature assumes Controllers, PlayerStates, replicated data, assets, and online services are all ready at `BeginPlay`.

**Correction:** Model each dependency through its actual readiness event.

### Raw Pointer as Ownership

A class stores a raw UObject pointer and assumes the target cannot be collected.

**Correction:** Use a GC-visible strong reference when retention is intended or a validated weak reference when it is
not.

### `Outer` as Lifetime Guarantee

A UObject receives a long-lived Outer, but no clear reference or cleanup policy exists.

**Correction:** Define conceptual ownership, reachability, and cleanup independently.

### Rooting Everything

Objects are added to the root set to prevent mysterious collection.

**Correction:** Fix reference reporting and scope ownership.

### Dynamic Component Without Registration

A Component is created with `NewObject`, but never becomes registered or operational.

**Correction:** Follow the complete runtime Component lifecycle.

### Duplicate Delegate Bindings

A registration callback runs repeatedly and binds the same handler each time.

**Correction:** Track registration, remove existing bindings, or use idempotent binding logic.

### Timer After Destruction

A timer callback accesses an Actor after its gameplay lifetime ended.

**Correction:** Clear the timer and use lifetime-safe bindings.

### Async Callback Into Old World

A request started before travel completes afterward and mutates destination-world state incorrectly.

**Correction:** Cancel, invalidate, or associate responses with a specific world or request generation.

### Game Instance Retains Old Actors

A persistent manager holds strong references to Actors from a destroyed world.

**Correction:** Clear world-scoped references during travel and reacquire destination dependencies.

### Pawn-Only Player State

Important participant data disappears when the Pawn dies.

**Correction:** Store information according to Pawn, PlayerController, or PlayerState lifetime.

### Cleanup Only in Destructor

Gameplay registrations remain active until garbage collection eventually destroys the object.

**Correction:** Perform gameplay teardown through explicit lifecycle callbacks such as `EndPlay`, `Deinitialize`, or
owner shutdown.

### Arbitrary Delay Initialization

Code waits one or more frames and assumes dependencies will then exist.

**Correction:** Listen for the actual possession, replication, loading, or feature-readiness event.

---

## Core Conclusions

Unreal lifecycle architecture should be based on explicit answers to five questions:

1. Who conceptually owns the object?
2. Which reference keeps it reachable?
3. At what lifecycle phase is it ready?
4. Which event ends its useful lifetime?
5. What must be released before final destruction?

The strongest general rule is:

> Create objects through the Unreal system, retain them through intentional references, initialize them only when their
> real dependencies are ready, and clean them up when their gameplay scope ends rather than waiting for garbage
> collection.

---

## Primary Sources

- [Objects in Unreal Engine](https://dev.epicgames.com/documentation/unreal-engine/objects-in-unreal-engine)
- [Creating Objects in Unreal Engine](https://dev.epicgames.com/documentation/unreal-engine/creating-objects-in-unreal-engine)
- [Unreal Object Handling](https://dev.epicgames.com/documentation/unreal-engine/unreal-object-handling-in-unreal-engine)
- [Unreal Engine Actor Lifecycle](https://dev.epicgames.com/documentation/unreal-engine/unreal-engine-actor-lifecycle)
- [Actors in Unreal Engine](https://dev.epicgames.com/documentation/unreal-engine/actors-in-unreal-engine)
- [Components in Unreal Engine](https://dev.epicgames.com/documentation/unreal-engine/components-in-unreal-engine)
- [UActorComponent API](https://dev.epicgames.com/documentation/unreal-engine/API/Runtime/Engine/UActorComponent)
- [Programming Subsystems](https://dev.epicgames.com/documentation/unreal-engine/programming-subsystems-in-unreal-engine)
- [Asynchronous Asset Loading](https://dev.epicgames.com/documentation/unreal-engine/asynchronous-asset-loading-in-unreal-engine)
- [Asset Management](https://dev.epicgames.com/documentation/unreal-engine/asset-management-in-unreal-engine)
- [Game Framework Component Manager](https://dev.epicgames.com/documentation/unreal-engine/game-framework-component-manager-in-unreal-engine)
- [Game Mode and Game State](https://dev.epicgames.com/documentation/unreal-engine/game-mode-and-game-state-in-unreal-engine)
- [AGameModeBase API](https://dev.epicgames.com/documentation/unreal-engine/API/Runtime/Engine/AGameModeBase)
- [Travelling in Multiplayer](https://dev.epicgames.com/documentation/unreal-engine/travelling-in-multiplayer-in-unreal-engine)
- [Delegates in Unreal Engine](https://dev.epicgames.com/documentation/en-us/unreal-engine/delegates?application_version=4.27)