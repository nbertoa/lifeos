# Memory Management in Unreal Engine

## Research Metadata

- **Research date:** 2026-07-22
- **Documentation baseline:** Unreal Engine 5.8
- **Primary authority:** Epic Games documentation
- **Scope:** Native C++ ownership, Unreal smart pointers, UObject garbage collection, containers, allocation behavior, RAII, pooling, asset memory, memory budgets, profiling, leaks, fragmentation, asynchronous lifetime, and review practices
- **Status:** Living document

---

## Purpose

Memory management in Unreal Engine is not one system.

A typical project combines:

- standard C++ automatic storage;
- value semantics;
- explicit native allocation;
- Unreal's smart pointer library;
- UObject garbage collection;
- engine containers and allocators;
- resource lifetime managed by engine subsystems;
- streamed assets and packages;
- CPU and GPU memory;
- platform-specific memory limits.

Correct code begins by identifying which lifetime system owns each resource.

The central principle is:

> Every allocation, object, asset, and external resource must have one explicit lifetime model, one understandable owner, and one verifiable release path.

Do not mix ownership systems merely because their pointer syntax looks similar.

---

## 1. Classify the Managed Resource First

### Recommendation

Before selecting a pointer or container, classify what is being managed.

### Classification

**Inference**

### Common Categories

1. **Value data**
   - numbers;
   - structs;
   - strings;
   - containers;
   - small domain records.

2. **Native non-UObject allocation**
   - algorithm objects;
   - service implementation details;
   - platform resources wrapped in C++;
   - Slate or tools data;
   - native shared state.

3. **UObject**
   - reflected object identity;
   - assets;
   - Actors;
   - Components;
   - engine-managed subobjects.

4. **External resource**
   - file handle;
   - socket;
   - OS object;
   - third-party library allocation;
   - GPU or RHI resource;
   - database or service connection.

5. **Asset dependency**
   - loaded UObject asset;
   - soft reference;
   - streamable asset;
   - package or bulk data.

### Why

Each category has different ownership and release rules.

A correct smart pointer for a native object can be invalid for a UObject.

A valid UObject reference does not automatically release an external resource at the required time.

---

## 2. Prefer Automatic Storage and Value Semantics

### Recommendation

Prefer values and scoped objects when independent heap identity is unnecessary.

### Classification

**General C++ best practice aligned with Epic coding patterns**

### Why

Automatic storage and value semantics provide:

- deterministic lifetime;
- simpler ownership;
- fewer allocations;
- reduced fragmentation;
- fewer failure paths;
- easier testing;
- easier reasoning;
- better data locality.

### Apply When

Prefer values for:

- small records;
- temporary calculations;
- function-local state;
- configuration snapshots;
- result objects;
- arrays and maps owned directly by a containing object;
- types that do not require polymorphic identity.

### Avoid

Do not allocate a small object merely to share syntax with other pointer-based systems.

Do not introduce UObject identity where a `USTRUCT` or native struct is sufficient.

---

## 3. Use RAII for Native and External Resources

### Recommendation

Wrap native and external resources in objects whose destructors release them.

### Classification

**General C++ requirement and Epic-compatible pattern**

### Why

RAII ties release to scope and ownership.

It protects against leaks caused by:

- early returns;
- failed initialization;
- exceptions in third-party code;
- multiple shutdown paths;
- refactoring;
- partial construction.

### Apply When

Use RAII wrappers for:

- memory allocated outside UObject GC;
- files;
- locks;
- sockets;
- handles;
- temporary registrations;
- third-party resources;
- scoped tracing or state changes.

### Implementation Guidance

The wrapper should:

- acquire or receive the resource;
- represent invalid or empty state clearly;
- release exactly once;
- support move semantics when ownership transfers;
- prohibit copying unless shared ownership is intentional.

### Avoid

Avoid manual pairs spread across multiple functions:

```cpp
Open();
...
Close();
```

Prefer an owner whose destructor guarantees cleanup.

---

## 4. Use `TUniquePtr` for Exclusive Native Ownership

### Recommendation

Use `TUniquePtr<T>` when exactly one native owner controls a non-UObject allocation.

### Classification

**Official Epic smart-pointer capability**

### Why

Unique ownership:

- expresses a single owner;
- avoids reference-counting overhead;
- supports deterministic destruction;
- makes transfer explicit through move semantics;
- is the simplest heap ownership model.

### Apply When

Use `TUniquePtr` for:

- private implementation objects;
- pimpl state;
- native strategy instances;
- parser or compiler state;
- platform abstractions;
- optional large native members;
- ownership returned from factories.

### Example

```cpp
TUniquePtr<FWorkerState> WorkerState = MakeUnique<FWorkerState>();
```

### Avoid

Do not use `TUniquePtr` for UObjects.

Do not promote unique ownership to shared ownership merely because multiple functions need temporary access. Pass a reference or raw observer instead.

---

## 5. Prefer `MakeUnique`

### Recommendation

Construct uniquely owned native objects with `MakeUnique` when possible.

### Classification

**Epic pattern**

### Why

Factory construction:

- reduces repeated type spelling;
- makes ownership immediate;
- avoids exposing raw allocation;
- improves exception safety in general C++;
- communicates intent.

### Avoid

Avoid:

```cpp
TUniquePtr<FData> Data(new FData());
```

Prefer:

```cpp
TUniquePtr<FData> Data = MakeUnique<FData>();
```

---

## 6. Use `TSharedRef` for Non-Nullable Shared Native Ownership

### Recommendation

Prefer `TSharedRef<T>` over `TSharedPtr<T>` when the shared native object is required to exist.

### Classification

**Official Epic recommendation**

### Why

Epic documents `TSharedRef` as the preferred shared reference form when null is not a valid state.

It:

- provides shared ownership;
- cannot be null;
- communicates a stronger contract;
- removes repeated validity checks.

### Apply When

Use it for:

- required Slate models;
- shared tool state;
- immutable shared native data;
- APIs that must always receive an object.

### Avoid

Do not use it when absence is a meaningful state.

Do not use it for UObjects.

---

## 7. Use `TSharedPtr` Only for Genuine Shared Native Ownership

### Recommendation

Use `TSharedPtr<T>` when multiple native owners independently extend an object's lifetime and null is valid.

### Classification

**Official Epic smart-pointer capability**

### Why

A shared pointer maintains a reference count and destroys the object when the final strong owner releases it.

This is useful, but shared ownership has costs:

- reference-count operations;
- extra control-block memory;
- less obvious destruction timing;
- risk of ownership cycles;
- more complex reasoning.

### Apply When

Use shared ownership when no single natural owner exists and multiple consumers genuinely retain the object.

### Avoid

Do not use `TSharedPtr` as the default replacement for a raw pointer.

Do not use it simply because ownership has not been designed.

Do not use it for UObjects.

---

## 8. Prefer `MakeShared` to `MakeShareable`

### Recommendation

Use `MakeShared` where construction and layout requirements permit it.

### Classification

**Official Epic performance recommendation**

### Why

Epic documents that `MakeShared` can avoid a second heap allocation by allocating the object and shared-reference controller together.

This may reduce:

- allocation count;
- allocator overhead;
- fragmentation;
- cache misses.

### Apply When

Use `MakeShared` for normal construction of shared native objects.

### Reconsider When

Separate allocation may be required when:

- adopting an existing pointer;
- using a custom deleter;
- object and control-block lifetime must be separated;
- API constraints require `MakeShareable`.

---

## 9. Use `TWeakPtr` to Break Native Shared-Ownership Cycles

### Recommendation

Use `TWeakPtr<T>` for native observations that must not extend the target's lifetime.

### Classification

**Official Epic recommendation**

### Why

Two shared objects holding strong references to each other create a reference-count cycle.

A weak pointer:

- does not own the object;
- allows expiration;
- makes observer semantics explicit;
- can be pinned to a strong pointer before use.

### Apply When

Use weak pointers for:

- listeners;
- parent back-references;
- caches;
- optional observers;
- asynchronous callbacks where the target may disappear.

### Validation

Pin once for the operation:

```cpp
if (TSharedPtr<FModel> Model = WeakModel.Pin())
{
    Model->Refresh();
}
```

### Avoid

Do not repeatedly call `Pin()` within one logical operation.

Do not dereference without obtaining a valid strong pointer.

---

## 10. Do Not Use Unreal Shared Pointers for UObjects

### Recommendation

Never use `TSharedPtr`, `TSharedRef`, `TWeakPtr`, or `TUniquePtr` to control UObject lifetime.

### Classification

**Official requirement**

### Why

Epic explicitly states that the Unreal Smart Pointer Library does not work with the UObject system.

UObjects use:

- Unreal object allocation;
- reachability-based garbage collection;
- object pointer types such as `TObjectPtr`;
- engine-specific destruction and reference tracking.

The two lifetime systems do not overlap.

### Correct Separation

For native non-UObject types, use:

- values;
- `TUniquePtr`;
- `TSharedRef`;
- `TSharedPtr`;
- `TWeakPtr`.

For UObjects, use:

- `TObjectPtr`;
- raw short-lived pointers;
- `TWeakObjectPtr`;
- `TSoftObjectPtr`;
- `TStrongObjectPtr` only when justified;
- reflected and engine-managed ownership.

---

## 11. Distinguish `TWeakPtr` From `TWeakObjectPtr`

### Recommendation

Use the weak-reference type belonging to the target's ownership system.

### Classification

**Official API distinction**

| Target | Weak Reference |
|---|---|
| Native shared object | `TWeakPtr<T>` |
| UObject | `TWeakObjectPtr<T>` |

### Why

They are not interchangeable.

`TWeakPtr` observes an object owned by Unreal's shared-reference system.

`TWeakObjectPtr` observes a UObject tracked by Unreal's object system.

---

## 12. Use `TObjectPtr` for Persistent Reflected UObject References

### Recommendation

Use `UPROPERTY()` with `TObjectPtr<T>` for persistent UObject references stored in reflected classes or structs.

### Classification

**Official Epic recommendation**

### Why

When reflected, `TObjectPtr` can:

- keep the target reachable;
- participate in serialization;
- participate in replication when supported;
- support cook-time dependency tracking;
- support GC barriers required by incremental collection features.

### Example

```cpp
UPROPERTY()
TObjectPtr<UMyAsset> Asset;
```

### Avoid

Do not assume an unreflected raw member pointer keeps a UObject alive.

---

## 13. Use Raw UObject Pointers for Short-Lived Access

### Recommendation

Use `T*` for parameters, return values, locals, and brief observation when lifetime is already guaranteed.

### Classification

**Official Epic guidance**

### Why

A raw pointer is lightweight and appropriate when it does not own or retain the object.

### Apply When

Use it when:

- the operation is synchronous;
- another strong reference guarantees lifetime;
- the pointer is not stored;
- the pointer is not captured into delayed work.

### Avoid

Do not convert every local raw pointer into a persistent field.

Do not assume non-null means valid in all lifecycle states.

---

## 14. Use `TWeakObjectPtr` for Non-Owning UObject Observation

### Recommendation

Use `TWeakObjectPtr<T>` when a UObject may be destroyed independently and the reference must not keep it alive.

### Classification

**Official Epic recommendation**

### Apply When

Use it for:

- Actor observers;
- caches;
- back-references;
- optional targets;
- delayed callbacks;
- systems that should tolerate target destruction.

### Validation

Resolve and validate the weak pointer close to use.

### Avoid

Do not use a strong reference simply to avoid handling target disappearance when non-ownership is the real contract.

---

## 15. Use Soft References to Control Asset Loading

### Recommendation

Use `TSoftObjectPtr` or `TSoftClassPtr` when the dependency should not force the target asset or class to load immediately.

### Classification

**Official Epic guidance**

### Why

Soft references represent an object path and support deferred loading.

They can reduce hard-reference chains and startup memory, but introduce explicit loading states.

### Apply When

Use them for:

- optional content;
- cosmetic variants;
- streamed levels or regions;
- large assets;
- data-selected classes;
- content loaded through the Asset Manager;
- UI or gameplay content not always needed.

### Avoid

Do not assume a soft reference consumes no memory after loading.

Do not access it as though the asset is always resident.

Do not replace required always-loaded dependencies with soft references without designing asynchronous behavior.

---

## 16. Use `TStrongObjectPtr` With Caution

### Recommendation

Use `TStrongObjectPtr<T>` only when a non-UObject owner must keep a UObject alive and normal reflected ownership is unavailable.

### Classification

**Official caution**

### Why

Epic warns that `TStrongObjectPtr`:

- always keeps the target alive;
- cannot be marked as `UPROPERTY`;
- is less visible to reference-debugging tools;
- adds a GC-tracked reference;
- is comparatively expensive to create and destroy;
- can create uncollectable cycles.

### Apply When

Use it for long-lived, infrequently changed references from native non-UObject owners.

### Avoid

Do not use it for high-churn per-frame data.

Do not use it inside a UObject as a substitute for a reflected `TObjectPtr`.

---

## 17. Never Allocate or Delete UObjects Manually

### Recommendation

Create and destroy UObjects only through Unreal-supported mechanisms.

### Classification

**Official requirement**

### Creation

Use:

- `NewObject<T>()`;
- `CreateDefaultSubobject<T>()`;
- `SpawnActor<T>()`;
- supported asset and duplication APIs.

### Destruction

Do not call `delete`.

For Actors, use `Destroy()` according to framework rules.

For ordinary UObjects, release strong references and allow garbage collection unless an API defines a more specific lifecycle action.

### Why

Manual allocation or deletion bypasses Unreal's:

- object registration;
- naming;
- flags;
- constructors and generated code;
- reachability tracking;
- destruction pipeline.

---

## 18. Reason About UObject Lifetime Through Reachability

### Recommendation

Identify the strong-reference path that keeps every live UObject reachable.

### Classification

**Official Unreal object model**

### Typical Strong Paths

- reflected `TObjectPtr`;
- reflected arrays, sets, and maps containing object references;
- engine-managed object graphs;
- root-set membership;
- custom reference collection;
- `TStrongObjectPtr`.

### Typical Non-Owning Paths

- raw local pointer;
- `TWeakObjectPtr`;
- `TSoftObjectPtr`;
- object path;
- untracked native pointer.

### Review Question

For every persistent UObject pointer, ask:

> Which tracked reference keeps this object alive for the complete period in which this pointer may be used?

If the answer is unclear, the lifetime design is incomplete.

---

## 19. Do Not Treat `Outer` as a Universal Strong Owner

### Recommendation

Use `Outer` for object containment and identity, but verify the actual GC reference path.

### Classification

**Context-dependent guidance**

### Why

`Outer` participates in:

- object paths;
- package ownership;
- serialization;
- duplication;
- containment semantics.

It should not be used as a vague substitute for explicit lifetime design.

### Avoid

Do not choose an arbitrary long-lived Outer merely to prevent collection.

Do not assume every inner object is kept alive solely because it has an Outer.

---

## 20. Avoid Rooting as a General Ownership Strategy

### Recommendation

Prefer normal tracked ownership over `AddToRoot`.

### Classification

**Epic pattern**

### Why

Rooting prevents collection independently of normal object graphs.

Misuse can cause:

- leaks;
- stale editor state;
- shutdown problems;
- hidden ownership;
- difficult reference debugging.

### Apply When

Use rooting only for controlled low-level cases with an explicit paired `RemoveFromRoot`.

### Avoid

Do not use `AddToRoot` as the first response to an object disappearing.

Fix the missing owner.

---

## 21. Use Custom GC Reference Collection Only When Required

### Recommendation

Use `FGCObject`, `AddReferencedObjects`, or equivalent supported hooks when references cannot be represented through reflected properties.

### Classification

**Official engine mechanism**

### Apply When

Use manual enumeration for:

- native non-UObject owners;
- unsupported native containers;
- low-level engine integrations;
- specialized object graphs.

### Avoid

Do not use manual reference collection when a normal `UPROPERTY TObjectPtr` works.

### Validation

Custom reference collectors require tests that prove:

- all strong references are enumerated;
- removed references stop retaining objects;
- shutdown paths unregister correctly;
- cycles do not remain permanently reachable.

---

## 22. Release External Resources Before UObject GC When Timing Matters

### Recommendation

Do not wait for unpredictable UObject reclamation to release scarce external resources.

### Classification

**Inference**

### Why

GC destruction is deferred.

Resources such as:

- sockets;
- file handles;
- third-party sessions;
- platform handles;
- subscriptions;
- GPU allocations;
- asynchronous requests;

may need deterministic shutdown.

### Implementation Guidance

Provide an idempotent explicit method such as:

```cpp
void Shutdown();
```

Call it from the framework lifecycle point where use must end.

The destructor or final UObject destruction hook may remain a defensive fallback, not the primary timing mechanism.

---

## 23. Make Cleanup Idempotent

### Recommendation

Design cleanup so repeated calls are safe.

### Classification

**Defensive programming pattern**

### Why

Unreal objects can encounter multiple overlapping teardown paths:

- explicit shutdown;
- EndPlay;
- world cleanup;
- subsystem deinitialization;
- PIE termination;
- module shutdown;
- failed initialization;
- UObject destruction.

### Implementation Guidance

After cleanup:

- clear handles;
- unbind delegates;
- reset pointers;
- mark state as stopped;
- cancel pending work;
- tolerate a second call.

### Avoid

Do not rely on one exact teardown sequence unless the framework guarantees it.

---

## 24. Use Containers as Owners of Their Elements

### Recommendation

Treat Unreal containers as value owners unless they explicitly store pointer-like elements.

### Classification

**Official container semantics**

### Common Containers

- `TArray`;
- `TSet`;
- `TMap`;
- `TQueue`;
- `TDeque`;
- `TSparseArray`;
- specialized engine containers.

### Why

For value elements, the container controls:

- element construction;
- relocation;
- destruction;
- capacity;
- allocation.

For pointer elements, the pointer type controls the referenced object's ownership semantics.

### Example

```cpp
TArray<FItemData> Items;                  // Owns values.
TArray<TUniquePtr<FNode>> Nodes;          // Owns native objects uniquely.
TArray<TObjectPtr<UObject>> Objects;      // Reflected strong UObject refs when property-tracked.
TArray<TWeakObjectPtr<AActor>> Targets;   // Non-owning UObject refs.
```

---

## 25. Understand Container Reallocation and Pointer Invalidation

### Recommendation

Assume that growth or mutation may invalidate addresses, references, and iterators unless the specific container contract guarantees otherwise.

### Classification

**Official container behavior and general C++ rule**

### Why

A `TArray` may relocate its elements when capacity changes.

This can invalidate:

- pointers to elements;
- references to elements;
- iterators;
- indices interpreted as permanent identity.

### Apply When

If an element needs stable identity:

- store an owning pointer;
- use a stable handle;
- use an index with generation validation;
- choose a suitable sparse or stable storage structure;
- separate identity from physical storage.

### Avoid

Do not capture `&Array[Index]` across operations that may resize or reorder the array.

---

## 26. Reserve Capacity When Growth Is Predictable

### Recommendation

Use `Reserve` when an approximate final element count is known and repeated growth would be significant.

### Classification

**Performance pattern**

### Why

Reserving can reduce:

- repeated allocations;
- element relocation;
- allocator contention;
- fragmentation;
- temporary peak memory.

### Apply When

Use it for:

- batch parsing;
- replicated or serialized collections with known counts;
- frame-local accumulation;
- known maximum participant counts;
- data conversion.

### Avoid

Do not reserve a large theoretical maximum without measurement.

Unused capacity is still memory.

---

## 27. Use `Reset`, `Empty`, and `Shrink` Deliberately

### Recommendation

Choose container clearing operations according to whether capacity should be retained or released.

### Classification

**Official API behavior and context-dependent guidance**

### Guidance

Conceptually:

- clearing while retaining capacity can reduce future allocation churn;
- emptying with reduced capacity can return memory;
- shrinking may compact excess allocation.

Exact API behavior should be verified for the container and engine version.

### Apply When

Retain capacity for collections reused frequently with similar sizes.

Release or shrink capacity when:

- a large temporary peak has ended;
- the container will remain small;
- memory pressure matters more than future reuse;
- a level or mode transition creates a natural cleanup point.

### Avoid

Do not call `Shrink` every frame.

Compaction itself has a cost and may cause churn.

---

## 28. Avoid Per-Frame Heap Allocation in Hot Paths

### Recommendation

Minimize repeated dynamic allocation and deallocation in high-frequency code.

### Classification

**Epic performance pattern**

### Why

Per-frame allocations can cause:

- allocator overhead;
- cache misses;
- fragmentation;
- contention;
- noisy traces;
- frame-time spikes;
- unnecessary constructor and destructor work.

### Apply When

Inspect:

- Tick functions;
- animation evaluation;
- AI updates;
- rendering-adjacent code;
- Mass processors;
- networking serialization;
- frequently executed Blueprint-callable functions.

### Alternatives

- reuse containers;
- reserve capacity;
- use stack values;
- use frame or arena allocators where supported;
- cache immutable data;
- batch operations;
- use pooling only when measurement justifies it.

---

## 29. Prefer Stack Allocation for Small Bounded Temporaries

### Recommendation

Use stack storage for small objects whose size and lifetime are bounded and appropriate for the platform.

### Classification

**General C++ guidance**

### Why

Stack allocation is fast and deterministic.

### Avoid

Do not place very large arrays or deeply recursive structures on the stack.

Stack size is limited and platform-dependent.

Use a container or explicit allocator when size is large or data must outlive the scope.

---

## 30. Use Inline Allocation for Small Common Container Sizes When Proven Useful

### Recommendation

Consider inline allocators when a container usually holds a small bounded number of elements and avoiding a heap allocation matters.

### Classification

**Context-dependent optimization**

### Why

An inline allocator stores some elements within the container object before allocating externally.

This can reduce heap allocation for common small cases.

### Costs

It may:

- increase the containing object's size;
- increase copy cost;
- waste memory when many containers are mostly empty;
- complicate API compatibility;
- provide no benefit for large typical sizes.

### Apply When

Use only after measuring allocation pressure and distribution of element counts.

---

## 31. Avoid Premature Custom Allocators

### Recommendation

Use engine and platform allocators unless profiling demonstrates a concrete unmet requirement.

### Classification

**Inference**

### Why

Custom allocators introduce:

- maintenance;
- alignment requirements;
- platform variation;
- debugging complexity;
- tooling integration challenges;
- lifetime assumptions;
- fragmentation risks.

### Apply When

A custom allocator may be justified for:

- fixed-size high-volume objects;
- frame arenas;
- specialized streaming systems;
- deterministic memory regions;
- third-party API integration;
- measured allocator contention.

### Validation

A custom allocator requires:

- alignment tests;
- overflow behavior;
- reset and shutdown correctness;
- thread-safety definition;
- instrumentation;
- memory-pressure behavior;
- platform validation.

---

## 32. Respect Alignment Requirements

### Recommendation

Use Unreal or standard allocation APIs that preserve the alignment required by the stored type.

### Classification

**Official low-level requirement**

### Why

Incorrect alignment can cause:

- crashes;
- corrupted data;
- platform-specific failures;
- SIMD faults;
- performance degradation.

### Avoid

Do not store arbitrary typed objects in raw byte buffers without correct alignment and construction.

Do not use `memcpy` as object construction for non-trivially copyable types.

---

## 33. Do Not Use `FMemory::Memcpy` for Arbitrary C++ Objects

### Recommendation

Use raw memory operations only for types and ranges whose representation permits them.

### Classification

**General C++ requirement**

### Apply When

Raw copy may be valid for:

- bytes;
- trivially copyable data;
- explicitly serialized binary layouts under controlled conditions.

### Avoid

Do not raw-copy types containing:

- `FString`;
- `TArray`;
- smart pointers;
- UObject references requiring barriers;
- virtual functions;
- nontrivial constructors or destructors;
- internal ownership.

Use assignment, move operations, or serialization instead.

---

## 34. Use Move Semantics for Ownership Transfer and Large Values

### Recommendation

Move native resources and large containers when ownership transfers and the source no longer needs its contents.

### Classification

**Modern C++ and Epic-compatible guidance**

### Why

Moving may avoid:

- deep copies;
- allocation duplication;
- temporary peak memory;
- reference-count churn.

### Apply When

Use `MoveTemp` intentionally when:

- transferring a container into an owner;
- returning or storing a unique pointer;
- consuming an input value;
- replacing state atomically.

### Avoid

Do not move from a value that will still be read as though unchanged.

Do not use `MoveTemp` mechanically on every return value; normal return-value optimization often applies.

---

## 35. Pass Large Native Objects by Reference

### Recommendation

Avoid unnecessary copies of large structs, containers, and shared-pointer handles.

### Classification

**Epic coding and smart-pointer performance guidance**

### Guidance

Prefer:

- `const T&` for read-only non-null input;
- `T&` for explicit mutation;
- pointer when null is meaningful;
- value when copying or ownership transfer is intended;
- rvalue reference or consumed value for transfer APIs.

Epic specifically cautions that passing shared-pointer wrappers by value can incur reference-count overhead when the function only needs the referenced object.

### Avoid

Do not accept `TSharedPtr<T>` by value merely to read `T`.

Prefer the referenced object or a suitable reference when ownership is not being transferred or retained.

---

## 36. Make Thread-Safety Mode Deliberate for Shared Pointers

### Recommendation

Use thread-safe shared pointer modes only when references genuinely cross threads.

### Classification

**Official Epic capability**

### Why

Unreal smart pointers support thread-safe reference counting, but thread safety has overhead.

Reference-count safety also does not make the pointed object's state thread-safe.

### Apply When

Define separately:

- whether pointer ownership crosses threads;
- whether object state is immutable;
- which synchronization protects mutable state;
- on which thread destruction may occur.

### Avoid

Do not assume `ESPMode::ThreadSafe` makes arbitrary member access safe.

---

## 37. Protect Lifetime Across Asynchronous Work

### Recommendation

Every asynchronous operation must explicitly define how its inputs and callback targets remain valid.

### Classification

**Defensive programming requirement**

### Native Shared Object Pattern

Capture a weak pointer and pin inside the callback when the task should not extend lifetime.

Capture a strong shared pointer when the operation intentionally owns the object until completion.

### UObject Pattern

Use a weak UObject pointer when the target may disappear.

Do not access mutable UObject state from worker threads unless the API and object lifetime explicitly support it.

Copy plain data for worker-thread processing and return to the game thread for UObject interaction.

### Cancellation

Long-lived operations should support:

- cancellation;
- owner shutdown;
- world teardown;
- module shutdown;
- stale-result rejection.

### Avoid

Do not capture raw `this` into delayed work without a proven lifetime guarantee.

---

## 38. Beware Lambda Capture Ownership

### Recommendation

Review lambda captures as ownership decisions.

### Classification

**Inference**

### Why

A lambda may silently:

- extend a shared object's lifetime;
- create a cycle;
- retain a large object graph;
- capture a dangling raw pointer;
- retain an asset or UObject indirectly;
- outlive a world or module.

### Review Questions

- Is `this` captured?
- Who owns the callback?
- Can the callback owner also be owned by the captured object?
- Is the delegate removed?
- Does the callback survive world teardown?
- Is a weak capture more appropriate?
- Is a copied payload unexpectedly large?

---

## 39. Unbind Delegates and Event Subscriptions

### Recommendation

Pair every long-lived registration with an explicit unregistration path.

### Classification

**Epic lifecycle pattern**

### Why

Delegates and callbacks can retain:

- native shared objects;
- UObjects through bindings;
- lambdas and their captures;
- module or subsystem state.

They can also invoke stale targets after shutdown.

### Apply When

Track delegate handles for:

- global delegates;
- subsystem callbacks;
- ticker registrations;
- asset-manager events;
- editor delegates;
- third-party callbacks.

### Avoid

Do not assume destruction alone removes every registration safely.

Use documented automatic UObject binding behavior where applicable, but still reason about ordering and payload captures.

---

## 40. Use Pooling Only After Measuring

### Recommendation

Introduce object or buffer pools only when profiling shows allocation churn or creation cost is significant.

### Classification

**Context-dependent guidance**

### Benefits

Pooling can reduce:

- allocation frequency;
- initialization cost;
- fragmentation;
- recurring resource creation.

### Costs

Pooling can increase:

- resident memory;
- stale state bugs;
- ownership complexity;
- reset complexity;
- peak retention;
- GC roots or references;
- debugging difficulty.

### Apply When

Pool objects that:

- are homogeneous;
- have expensive creation;
- are reused frequently;
- can be reset completely;
- have bounded or controlled pool sizes.

### Avoid

Do not pool every Actor or UObject by default.

For Actors, pooling must correctly address:

- visibility;
- collision;
- ticking;
- replication;
- ownership;
- attachments;
- components;
- delegates;
- timers;
- gameplay state;
- dormancy;
- destruction expectations.

---

## 41. Cap Pools and Caches

### Recommendation

Every pool and cache should have an explicit retention policy.

### Classification

**Inference**

### Define

- maximum entries;
- maximum bytes;
- age or recency eviction;
- level-transition behavior;
- memory-pressure behavior;
- platform-specific budgets;
- invalidation rules.

### Why

An unbounded cache is a memory leak with a lookup table.

A pool that only grows converts temporary peaks into permanent resident memory.

---

## 42. Separate Leak, Retention, and Fragmentation Problems

### Recommendation

Diagnose the type of memory problem before applying a fix.

### Classification

**Profiling methodology**

### Leak

Memory is no longer needed but cannot be released because ownership or cleanup is missing.

### Retention

Memory remains intentionally referenced longer than necessary.

Examples:

- cache never evicts;
- subsystem retains old world data;
- delegate capture keeps a model alive;
- hard asset reference keeps a package loaded.

### Fragmentation

Free memory exists but is split into regions unsuitable for requested allocations or incurs allocator inefficiency.

### Peak Memory

A temporary operation requires too much memory even if all allocations are later released.

### Churn

Many short-lived allocations create CPU cost and noisy memory behavior without a long-term leak.

### Why

Different problems require different solutions.

Pooling may reduce churn but worsen retention.

Soft references may reduce asset residency but not native heap fragmentation.

---

## 43. Track Asset Memory Separately From Native Heap Memory

### Recommendation

Distinguish gameplay/native allocations from memory retained by loaded assets and rendering resources.

### Classification

**Epic profiling pattern**

### Asset-Related Memory May Include

- textures;
- meshes;
- animation data;
- audio;
- shaders;
- packages;
- bulk data;
- derived runtime resources;
- duplicated editor objects;
- Blueprint-generated classes.

### Why

A small C++ object may hold one hard asset reference that retains a large dependency graph.

The pointer's size is not the dependency's memory cost.

### Apply When

For unexpected residency, inspect:

- hard-reference chains;
- primary asset rules;
- soft references;
- preload bundles;
- streaming state;
- package ownership;
- level references;
- CDO defaults;
- data assets;
- Blueprint variables.

---

## 44. Treat Hard Asset References as Memory Decisions

### Recommendation

Review every hard asset or class reference for loading and residency consequences.

### Classification

**Epic asset-management pattern**

### Why

Hard references can cause referenced content and dependencies to load with the referring object or package.

A hard reference in:

- a CDO;
- a commonly loaded Blueprint;
- a GameInstance object;
- a persistent subsystem;
- a frequently used Data Asset;

can retain significant content.

### Apply When

Use hard references for content that is:

- required;
- always used with the owner;
- acceptable to load together;
- small enough for the target budget.

### Reconsider

Use soft references and explicit loading when content is:

- optional;
- large;
- mutually exclusive;
- platform-dependent;
- selected late;
- rarely used.

---

## 45. Do Not Confuse Unloading With Garbage Collection

### Recommendation

Understand the complete reference and package lifecycle required for asset memory to be released.

### Classification

**Context-dependent Unreal behavior**

### Why

An asset cannot become unloadable while strong references remain.

Even after logical use ends, memory may remain because of:

- CDO references;
- global systems;
- caches;
- streamable handles;
- loaded packages;
- editor references;
- pending rendering resources;
- asynchronous operations.

### Avoid

Do not call garbage collection as a substitute for fixing hard references.

GC cannot collect reachable objects.

---

## 46. Release Streamable Handles According to the Loading Contract

### Recommendation

Define whether an asynchronous load handle intentionally keeps loaded assets resident.

### Classification

**Epic asset-loading pattern**

### Why

Streaming handles can participate in residency management.

Keeping a handle may be required while assets are in use.

Retaining it indefinitely may prevent unloading.

### Apply When

For each handle, define:

- owner;
- required lifetime;
- cancellation behavior;
- release point;
- callback lifetime;
- behavior during world or subsystem shutdown.

---

## 47. Establish Platform Memory Budgets

### Recommendation

Define memory budgets per target platform and major memory category.

### Classification

**Official performance methodology**

### Potential Categories

- total process memory;
- UObject memory;
- native allocations;
- textures;
- render targets;
- meshes;
- audio;
- animation;
- physics;
- navigation;
- UI;
- streaming pools;
- transient peaks;
- platform or middleware allocations.

### Why

A project cannot evaluate optimization without a target.

Desktop success does not imply console or mobile viability.

### Apply When

Budgets should account for:

- operating system reservation;
- platform services;
- multiplayer player count;
- worst-case maps;
- transitions;
- streaming overlap;
- save/load;
- frontend-to-game transitions;
- editor-only exclusions;
- safety margin.

---

## 48. Measure Representative Peaks, Not Only Steady State

### Recommendation

Profile complete workflows that produce worst-case overlap.

### Classification

**Profiling methodology**

### Test Scenarios

- cold startup;
- frontend load;
- map transition;
- seamless travel;
- streaming into dense regions;
- opening heavy UI;
- spawning maximum player count;
- respawn waves;
- loading save data;
- returning to menus;
- repeated match cycles;
- editor PIE start and stop where editor workflows matter.

### Why

Peak memory often occurs while old and new states overlap.

A stable scene may fit while transition peaks exceed the platform limit.

---

## 49. Use Memory Insights for Allocation Investigation

### Recommendation

Use Memory Insights to investigate allocation lifetime, callstacks, leaks, churn, and changes over time.

### Classification

**Official Epic recommendation**

### Capabilities

Epic documents that Memory Insights can:

- trace allocations, reallocations, and frees;
- associate callstacks;
- display Low Level Memory tags;
- query live allocations at timestamps;
- find short-lived allocations;
- find long-lived allocations;
- investigate suspected leaks;
- group by allocation callstack, tag, asset package, class, and size.

### Investigation Workflow

1. Reproduce a representative scenario.
2. Capture a memory trace with the required channels and symbols.
3. Mark relevant moments:
   - before operation;
   - after allocation growth;
   - after expected cleanup.
4. Run queries appropriate to the hypothesis.
5. Group by:
   - allocation callstack;
   - LLM tag;
   - asset package;
   - class;
   - size.
6. Verify whether allocations are:
   - still live;
   - intentionally retained;
   - leaked;
   - short-lived churn;
   - part of a temporary peak.
7. Fix ownership or loading behavior.
8. Capture again and compare.

### Avoid

Do not optimize based solely on one total-memory number.

Use allocation lifetime and ownership evidence.

---

## 50. Use Memory Leak Queries as Evidence, Not Automatic Proof

### Recommendation

Interpret Memory Insights leak-style queries in the context of expected lifetime.

### Classification

**Official tool capability with contextual interpretation**

### Why

An allocation surviving between selected timestamps may be:

- a true leak;
- a cache;
- a singleton;
- an intentionally persistent subsystem;
- an allocator reserve;
- a resource released later;
- a module-lifetime object.

### Apply When

Validate suspected leaks by reproducing cycles:

- enter and leave a level repeatedly;
- create and destroy the feature repeatedly;
- compare stable baselines;
- inspect ownership and callstacks;
- verify whether retained memory grows monotonically.

---

## 51. Use LLM Tags to Understand Memory Categories

### Recommendation

Use Low Level Memory tracking and tags to attribute memory to systems and categories.

### Classification

**Official Epic tooling**

### Why

LLM provides a categorized view that complements detailed allocation tracing.

It helps answer:

- which system owns growth;
- whether memory is CPU or another tracked address space;
- which category exceeds budget;
- whether a change shifts memory between systems.

### Avoid

Do not treat tags as perfect object-level ownership proof.

Use detailed allocation callstacks and object/reference tools when finer attribution is required.

---

## 52. Preserve Symbols for Actionable Callstacks

### Recommendation

Capture and retain matching symbols for builds used in memory investigation.

### Classification

**Official tooling requirement**

### Why

Unresolved callstacks reduce allocation traces to addresses or incomplete frames.

Actionable profiling requires:

- matching executable and modules;
- symbol files;
- known build identity;
- trace files;
- source revision.

### Apply When

For team investigations, archive:

- trace;
- build identifier;
- commit;
- platform;
- configuration;
- reproduction steps;
- symbols or symbol-server path.

---

## 53. Compare Memory Before and After a Controlled Operation

### Recommendation

Use controlled A/B timestamps around one feature or lifecycle transition.

### Classification

**Profiling methodology**

### Example

For a level transition:

- **A:** stable before transition;
- **B:** after new content loads;
- **C:** after old content should be released.

Then inspect:

- allocations created between A and B;
- allocations surviving through C;
- packages and classes retained;
- callstacks for native allocations;
- asset references preventing unload.

### Why

Controlled windows reduce noise and connect memory to behavior.

---

## 54. Repeat Lifecycle Tests to Detect Accumulation

### Recommendation

Run the same create/use/destroy cycle multiple times and compare post-cycle baselines.

### Classification

**Defensive validation pattern**

### Why

Many leaks and retention bugs are invisible after one cycle.

Look for monotonic growth in:

- live allocations;
- UObject count;
- loaded packages;
- delegate registrations;
- pooled elements;
- task objects;
- streamable handles;
- native resources.

### Avoid

Do not accept one successful cleanup as proof when the feature is repeatedly entered during normal use.

---

## 55. Instrument Ownership Boundaries

### Recommendation

Add tracing, counters, and diagnostics at resource acquisition and release boundaries when a system is difficult to analyze.

### Classification

**Context-dependent guidance**

### Apply When

Instrument:

- active request count;
- pooled object count;
- cache bytes;
- loaded entries;
- outstanding handles;
- registered listeners;
- native resource count;
- UObject instance count for critical classes.

### Why

A high-level counter can show which invariant failed before detailed allocation analysis.

---

## 56. Do Not Force Garbage Collection Casually

### Recommendation

Do not call full garbage collection as a routine fix for memory pressure or stale objects.

### Classification

**Context-dependent caution**

### Why

Garbage collection can:

- cause frame hitches;
- wait for other threads;
- process large object graphs;
- mask missing lifecycle cleanup;
- fail to release reachable assets;
- create unstable performance if called too often.

### Apply When

Explicit collection may be appropriate at controlled transitions or specialized workflows after profiling and platform validation.

### Avoid

Do not call it every frame or frequently during gameplay.

Do not use it to compensate for strong references that should have been released.

---

## 57. Control UObject Churn

### Recommendation

Avoid creating very large numbers of short-lived UObjects when value or native data would suffice.

### Classification

**Epic performance guidance**

### Why

UObject churn affects:

- allocation;
- object registration;
- reflection metadata interaction;
- GC graph traversal;
- collection workload;
- destruction;
- debugging.

### Alternatives

Use:

- USTRUCT values;
- native structs;
- batched records;
- Mass entities for suitable large-scale data-oriented simulation;
- reusable objects where correct;
- compact arrays and handles.

### Avoid

Do not convert every gameplay event, temporary result, or message into a UObject.

---

## 58. Establish UObject Count Budgets

### Recommendation

Track UObject population for scenarios with large object graphs.

### Classification

**Official Epic GC performance guidance**

### Apply When

Monitor counts across:

- startup;
- map load;
- peak gameplay;
- match restart;
- travel;
- return to frontend;
- repeated editor play sessions.

### Why

A stable memory footprint may still have excessive object count and GC traversal cost.

---

## 59. Treat Incremental Garbage Collection as Version- and Project-Dependent

### Recommendation

Evaluate incremental GC only against the current engine version, project requirements, and Epic's documented feature status.

### Classification

**Experimental or context-dependent guidance**

### Why

Incremental collection behavior, requirements, and maturity can change between engine versions.

Epic's object-pointer guidance also ties `TObjectPtr` to barriers that enable incremental marking.

### Apply When

Before enabling or depending on it:

- verify current official documentation;
- convert persistent reflected object references appropriately;
- profile frame-time distribution;
- test all target platforms;
- test object-heavy scenarios;
- verify plugins and custom reference collectors.

### Avoid

Do not present incremental GC as a universal drop-in solution.

---

## 60. Separate CPU Memory From GPU Memory

### Recommendation

Diagnose system memory and graphics memory as distinct but interacting budgets.

### Classification

**Official profiling model**

### Why

A resource can have:

- CPU-side source data;
- GPU-side resource memory;
- staging or upload buffers;
- streaming metadata;
- duplicated editor data.

Memory Insights can represent multiple address spaces and heaps, while rendering tools and platform profilers may be required for detailed GPU attribution.

### Avoid

Do not conclude that a texture or render target problem is solved because only process heap allocations decreased.

---

## 61. Avoid Large Temporary Duplication

### Recommendation

Design transforms, serialization, and loading pipelines to avoid holding multiple full copies of large data simultaneously.

### Classification

**Performance pattern**

### Common Causes

- pass-by-value;
- temporary conversion buffers;
- decompression plus source plus final buffer;
- duplicated arrays;
- repeated string conversion;
- asset transition overlap;
- staging uploads;
- copy-before-move APIs.

### Apply When

Use:

- moves;
- streaming or chunking;
- in-place transformation where safe;
- bounded scratch buffers;
- phased release;
- reservation;
- consumer APIs that accept ownership.

---

## 62. Treat Serialization Buffers as Peak-Memory Risks

### Recommendation

Measure save, load, replication, compression, and cooking paths for temporary memory peaks.

### Classification

**Context-dependent guidance**

### Why

Serialization may allocate:

- source data;
- destination archive;
- compression buffers;
- decoded representation;
- object graph fixups;
- temporary strings;
- version-conversion data.

### Avoid

Do not evaluate only the final serialized size.

Peak resident memory during transformation may be much larger.

---

## 63. Make Cache Value Size Visible

### Recommendation

Track cache cost in bytes, not only entry count.

### Classification

**Inference**

### Why

Ten cached textures and ten cached names have radically different costs.

A cache should report:

- entry count;
- current bytes;
- peak bytes;
- hit rate;
- miss cost;
- eviction count;
- average and maximum entry size.

### Avoid

Do not justify a cache without measuring whether its hit rate offsets its memory cost.

---

## 64. Handle Failure Without Leaking Partial State

### Recommendation

Make multi-step initialization transactional or clean up successfully acquired resources when a later step fails.

### Classification

**Defensive programming requirement**

### Example

If initialization performs:

1. allocate buffer;
2. register callback;
3. create platform handle;
4. start asynchronous request;

then failure at step 4 must release steps 1–3.

### Implementation Guidance

Use:

- RAII members;
- local owners moved into final state only after success;
- scoped guards;
- one idempotent cleanup function;
- explicit initialization state.

---

## 65. Avoid Partially Initialized Owners

### Recommendation

Construct native owners into valid states or use factory functions that can report failure.

### Classification

**General C++ best practice**

### Why

An object that exists but owns an unpredictable subset of resources complicates:

- cleanup;
- method preconditions;
- moves;
- retries;
- testing.

### Apply When

Use factories returning:

- `TUniquePtr`;
- result types;
- optional values;
- explicit error data;

when construction can meaningfully fail.

### UObject Note

Because UObject constructors cannot use arbitrary constructor parameters and participate in CDO creation, use an explicit initialization method when runtime configuration is required, with clear failure and cleanup behavior.

---

## 66. Use Early Returns Without Losing Cleanup

### Recommendation

Prefer early returns for invalid preconditions, but ensure acquired resources are protected by RAII or explicit cleanup.

### Classification

**Epic pattern plus defensive programming**

### Why

Early returns simplify control flow, but manual resource management can leak when exit paths multiply.

### Correct Pattern

Acquire through scoped owners, then return freely.

### Avoid

Do not require every future return path to remember a separate manual release sequence.

---

## 67. Avoid Hidden Allocation in Performance-Critical APIs

### Recommendation

Document or expose allocation behavior for APIs used in hot paths.

### Classification

**Performance engineering guidance**

### Hidden Allocation Sources

- returning large containers;
- string formatting;
- implicit conversions;
- delegate copies;
- shared-pointer creation;
- container growth;
- Blueprint marshaling;
- temporary arrays;
- sorting helpers;
- captured lambdas.

### Apply When

For hot APIs, state whether they:

- append to caller-provided storage;
- replace output;
- allocate;
- reuse capacity;
- return a view;
- retain references.

---

## 68. Prefer Views for Non-Owning Ranges Where Supported

### Recommendation

Use non-owning views or spans when a function only reads an existing contiguous range and the lifetime contract is clear.

### Classification

**Context-dependent modern C++ guidance**

### Why

Views can avoid:

- copying arrays;
- creating temporary containers;
- transferring ownership unnecessarily.

### Avoid

Do not return a view into temporary or unstable storage.

Do not retain a view beyond the owner's lifetime or across reallocations.

---

## 69. Be Explicit About String Ownership and Conversion

### Recommendation

Avoid repeated allocation-heavy conversion among string types in hot paths.

### Classification

**Performance pattern**

### Common Types

- `FString`;
- `FName`;
- `FText`;
- string views;
- platform or third-party string types.

### Guidance

Use:

- `FName` for identity-like repeated names;
- `FText` for localized user-facing text;
- `FString` for mutable general strings;
- views for temporary non-owning access where supported.

### Avoid

Do not use user-facing `FText` as a cheap internal identifier.

Do not format strings every frame for logging or UI when values have not changed.

---

## 70. Disable or Bound Debug Memory in Shipping Paths

### Recommendation

Ensure debug histories, traces, snapshots, and retained diagnostics have build-appropriate limits.

### Classification

**Defensive performance guidance**

### Why

Debug-only systems can become production memory leaks when they retain:

- every event;
- complete payloads;
- strings;
- screenshots;
- object references;
- unbounded logs.

### Apply When

Use:

- compile-time guards;
- bounded ring buffers;
- configurable sample rates;
- explicit clearing;
- shipping defaults;
- privacy and platform constraints.

---

## 71. Review Editor-Only Memory Separately

### Recommendation

Distinguish editor workflows from cooked runtime memory.

### Classification

**Official engine distinction**

### Why

The editor retains additional data for:

- transactions and undo;
- thumbnails;
- asset editors;
- Blueprint compilation;
- details panels;
- preview worlds;
- source assets;
- editor subsystems.

### Apply When

Profile cooked builds for runtime budgets.

Profile the editor separately when team productivity or editor stability is the problem.

### Avoid

Do not derive shipping-memory conclusions solely from PIE.

---

## 72. Validate Repeated PIE Sessions for Tooling Leaks

### Recommendation

For editor modules and tools, repeat PIE and asset-editor workflows to detect retained worlds, delegates, and preview objects.

### Classification

**Epic editor lifecycle pattern**

### Look For

- old worlds;
- duplicated objects;
- editor delegates;
- Slate widgets;
- shared tool models;
- preview scenes;
- async tasks;
- ticker handles;
- rooted objects;
- module singletons.

### Why

Editor process lifetime is long, so small lifecycle leaks accumulate significantly.

---

## 73. Do Not Keep Raw Pointers Across Module Shutdown

### Recommendation

Release cross-module native references and callbacks before the providing module unloads.

### Classification

**Official module-lifecycle implication**

### Why

A pointer may remain numerically non-null while its code or static storage no longer exists.

### Apply When

On shutdown:

- unregister modular features;
- remove delegates;
- stop tasks;
- destroy native implementation objects;
- release shared references;
- clear function callbacks;
- validate dependency shutdown order.

---

## 74. Avoid Static Initialization for Complex Owners

### Recommendation

Do not use complex global or static objects when their construction or destruction depends on engine systems or module order.

### Classification

**Epic architectural pattern**

### Why

Static initialization and destruction order across translation units and modules is difficult to control.

This can create:

- use-before-initialization;
- shutdown crashes;
- leaked resources;
- allocator use after shutdown;
- stale UObject references.

### Apply When

Use module startup/shutdown, Subsystems, or function-local statics only when their lifecycle contract is appropriate and independent.

---

## 75. Memory Management Review Checklist

### Resource Classification

- Is the resource a value, native object, UObject, external resource, or asset dependency?
- Which lifetime system owns it?
- Is heap identity genuinely required?
- Is object identity distinct from value data?

### Native Ownership

- Is exclusive ownership represented by `TUniquePtr`?
- Is shared ownership genuinely required?
- Can `TSharedRef` replace a nullable `TSharedPtr`?
- Are weak references breaking cycles?
- Is `MakeShared` used where suitable?
- Are shared pointer wrappers passed without unnecessary reference-count churn?
- Is thread-safe mode justified?

### UObject Ownership

- Are UObjects created through Unreal APIs?
- Is every persistent strong reference tracked?
- Are `TObjectPtr`, weak, soft, and strong object pointers selected by contract?
- Is `TStrongObjectPtr` justified and long-lived?
- Is `AddToRoot` avoided or paired with removal?
- Are custom reference collectors complete?
- Is `Outer` used semantically rather than as hidden ownership?

### Asynchronous Work

- Does every task define input and target lifetime?
- Are raw `this` captures avoided where unsafe?
- Are UObject accesses returned to the game thread when required?
- Can work be canceled during shutdown?
- Are stale results rejected?
- Are delegate and callback registrations removed?

### Containers and Allocation

- Could value semantics replace pointers?
- Is predictable capacity reserved?
- Are addresses or iterators retained across reallocation?
- Is capacity intentionally retained or released?
- Are per-frame allocations measured?
- Is inline allocation justified by element-count data?
- Are custom allocators truly necessary?
- Are alignment and construction rules respected?
- Are moves used for real ownership transfer?

### Cleanup

- Is release deterministic where timing matters?
- Is cleanup idempotent?
- Are partial-initialization failures safe?
- Are external resources released before deferred GC destruction?
- Are teardown paths compatible with world, module, and editor shutdown?

### Pools and Caches

- Is pooling based on profiling evidence?
- Can every pooled object be fully reset?
- Is the pool bounded?
- Is cache cost tracked in bytes?
- Are eviction and transition policies defined?
- Does retained capacity exceed realistic reuse needs?

### Assets

- Are hard references intentional?
- Could a CDO or persistent subsystem retain a large dependency graph?
- Are soft references paired with a loading strategy?
- Are streamable handles released correctly?
- Can packages actually unload after use?
- Are CPU and GPU resource costs distinguished?

### Profiling

- Are target-platform budgets defined?
- Are representative worst-case transitions captured?
- Are symbols available?
- Are Memory Insights queries based on controlled timestamps?
- Are leaks distinguished from retention, fragmentation, churn, and peaks?
- Are repeated lifecycle cycles tested?
- Are LLM tags and detailed allocation callstacks both used?
- Has the fix been verified with a second trace?

### UObject and GC Performance

- Is UObject count justified?
- Are short-lived records implemented as value data where possible?
- Is explicit garbage collection rare and controlled?
- Is incremental GC classified according to current documentation?
- Are GC costs profiled on target platforms?

### Editor and Modules

- Are cooked runtime and editor memory evaluated separately?
- Do repeated PIE sessions return to a stable baseline?
- Are editor delegates, Slate models, preview worlds, and tasks released?
- Are cross-module pointers cleared before unload?
- Are complex static owners avoided?

---

## Summary

Memory management in Unreal Engine requires choosing the correct ownership system before choosing a pointer.

The key rules are:

- prefer values and automatic storage;
- use RAII for native and external resources;
- use `TUniquePtr` for exclusive native ownership;
- use `TSharedRef` or `TSharedPtr` only for genuine shared ownership;
- use weak references to break cycles and represent observation;
- never use Unreal shared pointers to own UObjects;
- use `TObjectPtr` for persistent reflected UObject references;
- use raw UObject pointers for short-lived access;
- use weak UObject pointers for non-ownership;
- use soft pointers to control asset loading;
- use `TStrongObjectPtr` and root-set manipulation cautiously;
- reason about UObject lifetime through reachability;
- minimize high-frequency allocation and container churn;
- reserve, reuse, shrink, pool, and cache only according to measured behavior;
- treat hard asset references as loading and memory decisions;
- release external resources deterministically;
- design asynchronous work around explicit lifetime and cancellation;
- distinguish leaks, retention, fragmentation, peaks, and churn;
- establish target-platform budgets;
- investigate memory with repeatable scenarios, symbols, LLM tags, and Memory Insights;
- verify every optimization with another capture.

The goal is not to eliminate every allocation.

The goal is to make ownership explicit, lifetime correct, memory usage bounded, and performance measurable.

---

## Primary Sources

- [Smart Pointers in Unreal Engine](https://dev.epicgames.com/documentation/en-us/unreal-engine/smart-pointers-in-unreal-engine)
- [Shared Pointers](https://dev.epicgames.com/documentation/en-us/unreal-engine/shared-pointers-in-unreal-engine)
- [Shared References](https://dev.epicgames.com/documentation/en-us/unreal-engine/shared-references-in-unreal-engine)
- [Weak Pointers](https://dev.epicgames.com/documentation/en-us/unreal-engine/weak-pointers-in-unreal-engine)
- [Object Pointers](https://dev.epicgames.com/documentation/en-us/unreal-engine/object-pointers-in-unreal-engine)
- [Objects](https://dev.epicgames.com/documentation/en-us/unreal-engine/objects-in-unreal-engine)
- [Memory Insights](https://dev.epicgames.com/documentation/en-us/unreal-engine/memory-insights-in-unreal-engine)
- [Unreal Insights](https://dev.epicgames.com/documentation/en-us/unreal-engine/unreal-insights-in-unreal-engine)
- [Introduction to Performance Profiling and Configuration](https://dev.epicgames.com/documentation/en-us/unreal-engine/introduction-to-performance-profiling-and-configuration-in-unreal-engine)
- [Incremental Garbage Collection](https://dev.epicgames.com/documentation/en-us/unreal-engine/incremental-garbage-collection-in-unreal-engine)
- [TArray](https://dev.epicgames.com/documentation/en-us/unreal-engine/array-containers-in-unreal-engine)
- [TMap](https://dev.epicgames.com/documentation/en-us/unreal-engine/map-containers-in-unreal-engine)
- [TSet](https://dev.epicgames.com/documentation/en-us/unreal-engine/set-containers-in-unreal-engine)
