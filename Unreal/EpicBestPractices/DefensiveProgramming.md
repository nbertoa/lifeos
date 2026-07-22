# Defensive Programming in Unreal Engine

## Research Metadata

- **Research date:** 2026-07-22
- **Documentation baseline:** Unreal Engine 5.8
- **Primary authority:** Epic Games documentation and Unreal Engine API patterns
- **Scope:** Assertions, preconditions, pointer validation, guard clauses, errors, diagnostics, and resilient API design
- **Status:** Living document

---

## Purpose

Defensive programming makes assumptions explicit, detects programmer errors near their origin, validates untrusted data,
and prevents invalid state from propagating through the system.

In Unreal Engine, defensive programming commonly uses:

- `check` assertions;
- `verify` assertions;
- `ensure` diagnostics;
- explicit precondition validation;
- early returns and guard clauses;
- pointer-type selection;
- `IsValid`;
- checked and non-checked API variants;
- explicit result types;
- logs;
- data validation;
- network authority validation;
- lifetime-safe callbacks;
- tests.

Defensive programming does not mean allowing the program to continue silently after every invalid condition.

The central principle is:

> Distinguish programmer errors from expected failures, detect each as early as possible, and choose a response that
> matches whether safe recovery is possible.

---

## 1. Classify the Failure Before Choosing the Mechanism

### Recommendation

Determine what kind of failure occurred before selecting `check`, `ensure`, a return value, a log, or another response.

### Classification

**Inference based on Epic assertion and logging systems**

### Failure Categories

| Failure Category                 | Meaning                                                              | Typical Response                                               |
|----------------------------------|----------------------------------------------------------------------|----------------------------------------------------------------|
| Violated internal invariant      | The program reached a state that correct code should make impossible | `check`, `checkf`, specialized check macro                     |
| Violated internal precondition   | A programmer called an API incorrectly                               | `check`, `checkf`, or a checked API                            |
| Unexpected but recoverable state | The state indicates a probable bug, but local recovery is safe       | `ensure`, `ensureMsgf`, fallback, early return                 |
| Expected domain failure          | Failure is a normal possible outcome                                 | Explicit return value, result object, optional value, callback |
| Invalid user or designer input   | Authored or user-provided data can legitimately be wrong             | Validation, clear error reporting, safe rejection              |
| Invalid network input            | Remote clients are not trusted                                       | Server-side validation, rejection, logging, security response  |
| External-system failure          | Disk, backend, network, platform, or service can fail                | Explicit error handling, retry or fallback policy              |
| Resource absence                 | An optional resource is unavailable                                  | Optional result, fallback, feature disablement                 |
| Fatal unrecoverable condition    | Continuing would corrupt state or produce unsafe behavior            | Fatal log, crash, controlled shutdown                          |

### Why

Using the wrong mechanism can make a system less safe.

Examples:

- Using `check` for a network timeout turns a normal external failure into a crash.
- Returning silently after an impossible internal state hides a programming defect.
- Using `ensure` and then continuing through code that requires the failed condition creates secondary failures.
- Logging an error without returning can leave the function operating on invalid data.

### Core Decision

Ask:

1. Could this happen in a correct program?
2. Is the input controlled by trusted internal code?
3. Can the failure be handled locally?
4. Is continuing safe?
5. Must the expression execute in Shipping?
6. Does the caller need to know that the operation failed?

---

## 2. Treat Assertions as Executable Contracts

### Recommendation

Use assertions to document and verify assumptions that correct internal code must satisfy.

### Classification

**Official Epic diagnostic mechanism**

### Why

An assertion communicates more than a null check.

It states:

> This condition is required by the program's internal contract. Its failure means the implementation or one of its
> callers is defective.

Assertions help:

- detect defects near their source;
- document preconditions and invariants;
- generate useful call stacks;
- stop invalid state from propagating;
- distinguish programming bugs from expected failures;
- make assumptions visible during review.

### Appropriate Assertion Subjects

- Required pointers.
- Valid indices.
- Non-zero divisors.
- Valid state transitions.
- Required initialization.
- Correct thread context.
- Mutually exclusive states.
- Impossible enum cases.
- Internal ownership assumptions.
- Required module availability.
- Required configuration established by code.

### Avoid

- Asserting on user input.
- Asserting on packet contents.
- Asserting on recoverable file absence.
- Asserting on backend availability.
- Asserting on optional content.
- Using assertions as the only validation for security-sensitive data.
- Adding assertions without deciding what Shipping behavior remains.

### Sources

- [Asserts in Unreal Engine](https://dev.epicgames.com/documentation/unreal-engine/asserts-in-unreal-engine)

---

## 3. Use `check` for Conditions That Must Be True

### Recommendation

Use `check` when the condition represents a required internal invariant and continuing after failure is unsafe or
meaningless.

### Classification

**Official Epic recommendation**

### Example

```cpp
void UInventoryComponent::AddItemInstance(
    UItemInstance* ItemInstance)
{
    check(ItemInstance);

    Items.Add(ItemInstance);
}
```

The function's internal contract requires a valid item instance.

Passing `nullptr` is a programmer error.

### Formatted Variant

Use `checkf` when diagnostic context improves investigation:

```cpp
checkf(
    Quantity > 0,
    TEXT("AddItem requires a positive quantity. Received %d for item %s."),
    Quantity,
    *GetNameSafe(ItemDefinition));
```

### Apply When

- The condition is controlled by internal code.
- Correct callers must satisfy it.
- There is no meaningful local recovery.
- Continuing could corrupt state.
- A call stack should point directly to the violation.

### Avoid or Reconsider When

- The failure can occur during normal operation.
- The value came from an untrusted source.
- The API is intentionally fallible.
- A caller needs a recoverable result.
- Shipping behavior would become unsafe when checks are compiled out.

### Important Build Consideration

`check` expressions may not execute in configurations where checks are disabled.

Never place required side effects inside a `check`.

Incorrect:

```cpp
check(InitializeRequiredState());
```

If checks are compiled out, initialization may never execute.

Correct:

```cpp
const bool bInitialized = InitializeRequiredState();
check(bInitialized);
```

### Sources

- [Asserts in Unreal Engine](https://dev.epicgames.com/documentation/unreal-engine/asserts-in-unreal-engine)

---

## 4. Use Specialized Check Macros to Communicate Intent

### Recommendation

Use Unreal's specialized check macros when they describe the violated invariant more precisely than a generic condition.

### Classification

**Official Epic diagnostic mechanism**

### Common Macros

| Macro                                 | Intended Meaning                                                                |
|---------------------------------------|---------------------------------------------------------------------------------|
| `check(Expression)`                   | Expression must be true                                                         |
| `checkf(Expression, Format, ...)`     | Expression must be true, with formatted diagnostic context                      |
| `checkSlow(Expression)`               | Expensive invariant check enabled only in configurations supporting slow checks |
| `checkfSlow(Expression, Format, ...)` | Formatted slow check                                                            |
| `checkNoEntry()`                      | This execution path must never be reached                                       |
| `checkNoReentry()`                    | This code must not be re-entered before the first invocation exits              |
| `checkNoRecursion()`                  | This code must not recurse                                                      |
| `unimplemented()`                     | This code path has not been implemented                                         |

### `checkNoEntry`

Use for logically impossible branches:

```cpp
switch (MovementMode)
{
case EMovementMode::Walking:
    HandleWalking();
    break;

case EMovementMode::Falling:
    HandleFalling();
    break;

default:
    checkNoEntry();
    break;
}
```

Prefer an explicit impossible-path diagnostic over silently ignoring an unrecognized internal state.

### `checkNoReentry`

Use when the function must not be entered while a previous call is still active:

```cpp
void UExampleSystem::RebuildCache()
{
    checkNoReentry();

    // Rebuild work.
}
```

### `checkNoRecursion`

Use when any recursive call indicates a defect:

```cpp
void UExampleGraph::ResolveDependencies()
{
    checkNoRecursion();

    // Resolution work.
}
```

### `checkSlow`

Use for valuable but potentially expensive invariant verification:

```cpp
checkSlow(DoesInternalIndexMatchEntries());
```

Slow checks are appropriate when validating the invariant each time would impose unacceptable cost in ordinary
development builds.

### Avoid

- Using `checkNoEntry` instead of safely handling externally supplied enum values.
- Using recursion macros as a replacement for understanding legitimate nested calls.
- Depending on slow checks for correctness.
- Leaving `unimplemented()` in a reachable production path.
- Using specialized macros without a clear invariant.

---

## 5. Use `verify` When the Expression Must Always Execute

### Recommendation

Use `verify` when an expression must execute in every build configuration but its failure should behave as an assertion
when assertion diagnostics are enabled.

### Classification

**Official Epic recommendation**

### Example

```cpp
verify(InitializeSubsystem());
```

Unlike a `check` expression, the expression passed to `verify` is still evaluated when ordinary assertion diagnostics
are disabled.

### Formatted Variant

```cpp
verifyf(
    RegisterHandler(),
    TEXT("Failed to register the gameplay handler for %s."),
    *GetNameSafe(this));
```

### Apply When

- The expression has required side effects.
- The function is expected to succeed as an internal invariant.
- Failure represents a defect rather than a normal outcome.
- The operation must still happen in Shipping.

### Avoid or Reconsider When

- Failure is expected and should be handled explicitly.
- The result needs to reach the caller.
- The side effect should not be hidden inside an assertion-like macro.
- A clear two-line implementation would communicate intent better.

### Prefer Clarity for Complex Work

Instead of:

```cpp
verify(InitializeA() && InitializeB() && RegisterC());
```

prefer:

```cpp
const bool bInitializedA = InitializeA();
const bool bInitializedB = InitializeB();
const bool bRegisteredC = RegisterC();

verify(bInitializedA);
verify(bInitializedB);
verify(bRegisteredC);
```

This produces clearer diagnostics and avoids short-circuiting required operations unintentionally.

### Sources

- [Asserts in Unreal Engine](https://dev.epicgames.com/documentation/unreal-engine/asserts-in-unreal-engine)

---

## 6. Use `ensure` for Unexpected but Recoverable Conditions

### Recommendation

Use `ensure` when a condition should be true, its failure deserves diagnostic reporting, but the current operation has a
known safe recovery path.

### Classification

**Official Epic recommendation**

### Example

```cpp
void UInteractionComponent::RefreshTarget()
{
    if (!ensureMsgf(
            OwnerController,
            TEXT("%s expected an OwnerController before refreshing its target."),
            *GetNameSafe(this)))
    {
        ClearCurrentTarget();
        return;
    }

    // Safe to use OwnerController.
}
```

### Why

`ensure` occupies the space between:

- silently handling a normal failure;
- and terminating execution through a failed `check`.

It reports a probable defect while permitting execution to continue.

### Critical Rule

> Continuing after an `ensure` failure is acceptable only if the recovery path is actually safe.

This is unsafe:

```cpp
ensure(Target);

Target->ExecuteAction();
```

The code reports the problem and then dereferences the invalid pointer.

Correct:

```cpp
if (!ensure(Target))
{
    return;
}

Target->ExecuteAction();
```

### Common Variants

| Macro                                       | Behavior                                                         |
|---------------------------------------------|------------------------------------------------------------------|
| `ensure(Expression)`                        | Reports an unexpected false condition, normally once per session |
| `ensureMsgf(Expression, Format, ...)`       | Adds formatted context                                           |
| `ensureAlways(Expression)`                  | Reports every time the condition fails                           |
| `ensureAlwaysMsgf(Expression, Format, ...)` | Reports every failure with formatted context                     |

### Reporting Frequency

Ordinary `ensure` macros are designed to avoid repeatedly reporting the same failure throughout an Engine or Editor
session.

Use `ensureAlways` only when every occurrence provides useful diagnostic information and the volume will remain
controlled.

### Apply When

- The condition indicates a likely programming defect.
- Safe local recovery exists.
- Crashing would be disproportionate.
- The diagnostic should reach crash-reporting or development telemetry systems.
- The function can abort or substitute a safe fallback.

### Avoid

- Using `ensure` for ordinary optional values.
- Using `ensureAlways` in high-frequency execution.
- Continuing as though the condition succeeded.
- Filling logs with repeated non-actionable diagnostics.
- Treating `ensure` as input validation.
- Using `ensure` instead of fixing an invariant that should be a `check`.

### Sources

- [Asserts in Unreal Engine](https://dev.epicgames.com/documentation/unreal-engine/asserts-in-unreal-engine)
- [Crash Reporting in Unreal Engine](https://dev.epicgames.com/documentation/unreal-engine/crash-reporting-in-unreal-engine)

---

## 7. Do Not Use `ensure` for Expected Optional State

### Recommendation

Do not report an `ensure` merely because an optional value is absent.

### Classification

**Inference based on Epic's definition of ensure**

### Incorrect

```cpp
if (!ensure(OptionalTarget))
{
    return;
}
```

This is incorrect when having no target is a normal gameplay condition.

### Correct

```cpp
if (!OptionalTarget)
{
    return;
}
```

### Why

An `ensure` communicates that the state is unexpected and likely defective.

Overusing it for normal conditions:

- generates noise;
- reduces trust in reports;
- hides meaningful failures among expected states;
- encourages developers to ignore diagnostics.

### Decision Question

Ask:

> Would I create a bug ticket if this condition occurred once?

If not, an `ensure` is probably inappropriate.

---

## 8. Distinguish Preconditions, Postconditions, and Invariants

### Recommendation

Express function contracts according to whether they constrain inputs, outputs, or persistent object state.

### Classification

**General defensive-programming principle implemented with Unreal diagnostics**

### Preconditions

Conditions callers must satisfy before invoking a function.

```cpp
void UInventoryComponent::RemoveQuantity(
    UItemInstance* Item,
    const int32 Quantity)
{
    check(Item);
    checkf(Quantity > 0, TEXT("Quantity must be positive."));

    // Implementation.
}
```

### Postconditions

Conditions the function guarantees when it completes successfully.

```cpp
void UInventoryComponent::RebuildLookup()
{
    RebuildInternalLookup();

    checkf(
        Lookup.Num() == Items.Num(),
        TEXT("Inventory lookup must contain one entry per item."));
}
```

### Invariants

Conditions that must remain true throughout the valid lifetime of an object or system.

```cpp
void UHealthComponent::ValidateState() const
{
    check(MaxHealth > 0.0f);
    check(CurrentHealth >= 0.0f);
    check(CurrentHealth <= MaxHealth);
}
```

### Why

Thinking in contracts clarifies:

- who is responsible for validation;
- whether failure is a caller bug or implementation bug;
- what downstream code may safely assume;
- what tests must verify.

### Avoid

- Rechecking the same trusted internal precondition throughout every implementation layer.
- Using assertions instead of validating untrusted boundaries.
- Declaring a precondition that the API signature could make impossible.
- Writing postcondition checks with side effects.
- Creating expensive invariant checks in hot paths without considering `checkSlow`.

---

## 9. Prefer Types That Make Invalid Calls Harder

### Recommendation

Prevent invalid state structurally before adding runtime checks.

### Classification

**Inference aligned with Epic's strong typing and API patterns**

### Example: Avoid Ambiguous Booleans

Weak:

```cpp
ApplyDamage(Target, 25.0f, true, false);
```

Better:

```cpp
FDamageApplicationOptions Options;
Options.bBypassArmor = true;
Options.bCanKill = false;

ApplyDamage(Target, 25.0f, Options);
```

### Example: Avoid Invalid Parallel Parameters

Weak:

```cpp
StartCooldown(
    FName CooldownName,
    float Duration,
    bool bInfinite);
```

Better:

```cpp
StartCooldown(const FCooldownDefinition& Definition);
```

### Example: Required Reference

Where null is not a valid concept, a reference can express that more clearly for non-UObject APIs:

```cpp
void ProcessDefinition(const FItemDefinition& Definition);
```

### Techniques

- Enums instead of integer modes.
- Structs instead of long parameter lists.
- Dedicated identifier types.
- Required constructor parameters for native types.
- Result types for fallible operations.
- Separate APIs for semantically distinct operations.
- Private state mutation.
- Validating factories.
- Immutable definitions.

### Principle

> The strongest defensive API is one that cannot represent the invalid request.

---

## 10. Use Early Returns as Guard Clauses

### Recommendation

Use early returns to reject invalid prerequisites and exceptional paths before the main operation.

### Classification

**Defensive-programming practice consistent with Epic code patterns; not a universal explicit Epic rule**

### Why

Guard clauses keep the successful path visually prominent.

They reduce:

- nesting;
- mental state tracking;
- distance between validation and response;
- accidental execution after a failure;
- complex conditional combinations.

### Nested Version

```cpp
void UInteractionComponent::TryInteract()
{
    if (OwnerController)
    {
        if (CurrentTarget)
        {
            if (CurrentTarget->CanInteract(OwnerController))
            {
                CurrentTarget->Interact(OwnerController);
            }
        }
    }
}
```

### Guard-Clause Version

```cpp
void UInteractionComponent::TryInteract()
{
    if (!OwnerController)
    {
        return;
    }

    if (!CurrentTarget)
    {
        return;
    }

    if (!CurrentTarget->CanInteract(OwnerController))
    {
        return;
    }

    CurrentTarget->Interact(OwnerController);
}
```

The main action is no longer hidden inside three levels of nesting.

### Apply When

- A prerequisite is not satisfied.
- An optional condition means there is nothing to do.
- A recoverable validation failure should abort the current operation.
- Authority or ownership is incorrect.
- A feature is disabled.
- Required runtime context is unavailable.
- The current state makes the operation irrelevant.
- An async response is stale.
- A cast or lookup fails normally.

### Example: Authority Guard

```cpp
void AExampleActor::ApplyAuthoritativeState()
{
    if (!HasAuthority())
    {
        return;
    }

    // Server-authoritative implementation.
}
```

### Example: Stale Async Result

```cpp
void UProfileSystem::HandleResponse(
    const int32 RequestGeneration,
    const FProfileResponse& Response)
{
    if (RequestGeneration != ActiveRequestGeneration)
    {
        return;
    }

    if (!Response.IsSuccessful())
    {
        HandleRequestFailure(Response);
        return;
    }

    ApplyProfile(Response.Profile);
}
```

### Example: Ensure Plus Early Return

```cpp
void UAbilityComponent::ActivateRequiredAbility()
{
    if (!ensureMsgf(
            AbilitySystemComponent,
            TEXT("%s requires an AbilitySystemComponent."),
            *GetNameSafe(this)))
    {
        return;
    }

    AbilitySystemComponent->TryActivateAbilityByClass(
        RequiredAbilityClass);
}
```

### Principle

> Validate at the top, return on failure, and let the remainder describe the valid operation.

---

## 11. Do Not Turn Early Return Into Silent Failure

### Recommendation

Choose whether an early return should be silent, logged, ensured, or reported to the caller.

### Classification

**Defensive-programming inference**

### Silent Return

Appropriate when the condition is normal and the caller does not need a result:

```cpp
if (!OptionalTarget)
{
    return;
}
```

### Ensure and Return

Appropriate when the condition indicates a probable internal defect but local recovery is safe:

```cpp
if (!ensure(RequiredComponent))
{
    return;
}
```

### Log and Return

Appropriate when the failure needs operational context:

```cpp
if (!SaveSlot.IsValid())
{
    UE_LOG(
        LogSaveSystem,
        Error,
        TEXT("Cannot load invalid save slot '%s'."),
        *SaveSlot.ToString());

    return;
}
```

### Return a Result

Appropriate when the caller must react:

```cpp
bool UInventoryComponent::TryRemoveItem(
    UItemInstance* Item)
{
    if (!IsValid(Item))
    {
        return false;
    }

    return Items.RemoveSingle(Item) > 0;
}
```

### Explicit Error

Appropriate when failure details matter:

```cpp
FInventoryOperationResult
UInventoryComponent::TryAddItem(
    const UItemDefinition* Definition,
    const int32 Quantity)
{
    if (!IsValid(Definition))
    {
        return FInventoryOperationResult::Failure(
            EInventoryError::InvalidDefinition);
    }

    if (Quantity <= 0)
    {
        return FInventoryOperationResult::Failure(
            EInventoryError::InvalidQuantity);
    }

    // Add item.

    return FInventoryOperationResult::Success();
}
```

### Avoid

```cpp
if (!RequiredComponent)
{
    return;
}
```

when `RequiredComponent` being null proves the object was initialized incorrectly.

The return prevents the crash but conceals the defect.

---

## 12. Keep Guard Clauses Semantically Ordered

### Recommendation

Order early-return checks from cheapest and most fundamental prerequisites toward more expensive or specific validation.

### Classification

**Defensive-programming practice**

### Suggested Order

A common sequence is:

1. Execution-context checks.
2. Lifetime and pointer checks.
3. Authority and ownership checks.
4. State checks.
5. Cheap argument validation.
6. Expensive queries.
7. Side effects.
8. Main operation.

### Example

```cpp
bool UInteractionComponent::TryInteract(
    AActor* Candidate)
{
    if (!IsActive())
    {
        return false;
    }

    if (!IsValid(Candidate))
    {
        return false;
    }

    if (!OwnerController)
    {
        return false;
    }

    if (!Candidate->Implements<UInteractable>())
    {
        return false;
    }

    if (!IsWithinInteractionRange(Candidate))
    {
        return false;
    }

    ExecuteInteraction(Candidate);
    return true;
}
```

### Why

This minimizes unnecessary work and makes dependencies clear.

### Avoid

- Performing expensive traces before checking whether the feature is active.
- Mutating state before completing validation.
- Logging multiple errors for one rejected request.
- Ordering guards in a way that hides the most fundamental violation.
- Introducing guard clauses whose order changes behavior unintentionally.

---

## 13. Avoid Excessive Fragmentation From Early Returns

### Recommendation

Use early returns to clarify control flow, not automatically for every conditional.

### Classification

**Defensive-programming tradeoff**

### Reconsider Early Return When

- Cleanup must occur through one shared exit path.
- Several branches produce one result most clearly expressed once.
- The function is already very short.
- Multiple returns obscure a state transition.
- Resource management is manual and not protected by RAII.
- A single structured conditional communicates the alternatives better.
- Output parameters must be finalized consistently.

### Example Where One Result May Be Clearer

```cpp
EInteractionAvailability
UInteractionComponent::GetAvailability(
    const AActor* Candidate) const
{
    EInteractionAvailability Result =
        EInteractionAvailability::Available;

    if (!IsValid(Candidate))
    {
        Result = EInteractionAvailability::InvalidTarget;
    }
    else if (!IsWithinRange(Candidate))
    {
        Result = EInteractionAvailability::OutOfRange;
    }
    else if (IsBlocked())
    {
        Result = EInteractionAvailability::Blocked;
    }

    return Result;
}
```

An early-return implementation would also be valid. The clearer form depends on whether the function is best understood
as sequential guards or classification among alternatives.

### Principle

> Prefer the control flow that makes the contract and main path easiest to verify.

---

## 14. Do Not Hide Required Cleanup Behind Multiple Returns

### Recommendation

Use RAII, scope guards, or explicit cleanup design when functions have several exit paths.

### Classification

**General C++ defensive-programming practice compatible with Unreal**

### Risk

```cpp
BeginTemporaryOperation();

if (!StepA())
{
    return false;
}

if (!StepB())
{
    return false;
}

EndTemporaryOperation();
return true;
```

Both failure returns skip cleanup.

### Better Direction

Use an RAII object or scope guard whose destructor performs cleanup:

```cpp
BeginTemporaryOperation();

ON_SCOPE_EXIT
{
    EndTemporaryOperation();
};

if (!StepA())
{
    return false;
}

if (!StepB())
{
    return false;
}

return true;
```

### Apply to

- locks;
- temporary registrations;
- transaction-like state;
- native allocations;
- file handles;
- temporary flags;
- profiling scopes;
- external SDK resources.

### Avoid

- Rejecting early returns entirely because one function manages resources manually.
- Depending on callers to repair partially completed local work.
- Repeating cleanup before every return when scope-based cleanup would be safer.

---

## 15. Validate Pointers According to Their Semantics

### Recommendation

Do not apply one universal pointer check to every pointer type and context.

### Classification

**Official object-handling behavior with defensive-programming inference**

### Raw Native Pointer

For a non-UObject native pointer:

```cpp
if (Pointer == nullptr)
{
    return;
}
```

A null check answers whether the pointer contains an address.

It does not prove the address is valid if lifetime has already been violated.

### UObject Pointer

For a UObject that may be pending destruction or garbage:

```cpp
if (!IsValid(Object))
{
    return;
}
```

`IsValid` tests whether the UObject is non-null and usable according to the object's destruction and garbage state.

### Strong `TObjectPtr`

When the surrounding lifetime contract guarantees validity, direct use may be appropriate after a precondition
assertion:

```cpp
check(RequiredComponent);
RequiredComponent->Execute();
```

### Weak UObject Pointer

```cpp
if (UExampleComponent* Component = WeakComponent.Get())
{
    Component->Execute();
}
```

### Soft Object Pointer

A soft pointer may represent an unloaded asset.

```cpp
if (Asset.IsNull())
{
    return;
}

if (!Asset.IsValid())
{
    RequestAssetLoad(Asset);
    return;
}

UseAsset(Asset.Get());
```

### Shared Native Pointer

```cpp
if (!SharedService.IsValid())
{
    return;
}
```

### Principle

> Pointer validation must match whether the pointer is native, garbage-collected, weak, soft, or reference-counted.

### Sources

- [IsValid API](https://dev.epicgames.com/documentation/unreal-engine/API/Runtime/CoreUObject/UObject/IsValid)
- [Unreal Object Handling](https://dev.epicgames.com/documentation/unreal-engine/unreal-object-handling-in-unreal-engine)

---

## 16. Use `IsValid` When Object Destruction Is a Real Possibility

### Recommendation

Use `IsValid` for UObject references whose targets may have entered destruction or garbage-collection state.

### Classification

**Official API behavior**

### Apply When

- Observing Actors that another system may destroy.
- Handling deferred callbacks.
- Reading weak or optional world-object references.
- Reacting after level streaming or travel.
- Processing queued work.
- Working with user-selected editor objects.
- Receiving object references from broad framework callbacks.

### Reconsider When

- The pointer is a mandatory internal precondition.
- A null value proves initialization is broken.
- The lifetime is guaranteed by the immediate call context.
- Repeated validity checks are hiding a missing ownership contract.

### Weak Pattern

```cpp
void URequiredSystem::Execute()
{
    if (!IsValid(RequiredDependency))
    {
        return;
    }

    RequiredDependency->Execute();
}
```

If `RequiredDependency` must always exist while this system is operational, this silently converts a broken invariant
into missing behavior.

### Stronger Pattern

```cpp
void URequiredSystem::Execute()
{
    checkf(
        IsValid(RequiredDependency),
        TEXT("%s requires a valid RequiredDependency while operational."),
        *GetNameSafe(this));

    RequiredDependency->Execute();
}
```

Or, when safe recovery is required:

```cpp
void URequiredSystem::Execute()
{
    if (!ensureMsgf(
            IsValid(RequiredDependency),
            TEXT("%s lost its RequiredDependency."),
            *GetNameSafe(this)))
    {
        ShutdownSafely();
        return;
    }

    RequiredDependency->Execute();
}
```

### Sources

- [IsValid API](https://dev.epicgames.com/documentation/unreal-engine/API/Runtime/CoreUObject/UObject/IsValid)

---

## 17. Do Not Use Low-Level Validity Checks as Routine Gameplay Validation

### Recommendation

Avoid low-level object-integrity functions as ordinary pointer checks.

### Classification

**Official API intent and engine pattern**

### Why

Low-level validity checks are intended for diagnosing corrupted object state, invalid memory, or severe engine-level
defects.

They are not a substitute for:

- correct ownership;
- `UPROPERTY`;
- `TObjectPtr`;
- `TWeakObjectPtr`;
- `IsValid`;
- lifecycle-aware design.

### Avoid

Patterns such as:

```cpp
if (Object && Object->IsValidLowLevel())
{
    Object->Execute();
}
```

in ordinary gameplay code.

### Why This Is Dangerous

- It may create false confidence about pointer safety.
- Calling a method through a genuinely dangling pointer is already unsafe.
- It can hide the root ownership bug.
- It may be significantly more expensive than intended.
- It communicates the wrong abstraction level.

### Prefer

- A correct GC-visible strong reference.
- A weak pointer that can be validated safely.
- `IsValid` for a legitimate UObject reference.
- An assertion when the pointer must exist.
- Fixing the stale-reference source.

---

## 18. Prefer Checked APIs When Absence Is a Programmer Error

### Recommendation

Use `Checked` API variants when the requested value is required and absence indicates incorrect internal code.

### Classification

**Epic API pattern**

### Common Naming Pattern

Unreal APIs frequently distinguish:

- `Get` or `Find`: retrieval with API-specific behavior;
- `GetChecked` or `FindChecked`: value must exist;
- `TryGet` or `Try...`: failure is expected and explicitly reported;
- `GetSafe` or safe helper: absence is tolerated;
- `FindRef`: return a value or a default value, depending on container semantics.

### Example

```cpp
FModuleManager::GetModuleChecked<FExampleModule>(
    TEXT("ExampleModule"));
```

The checked form communicates that the module must already be available.

### Apply Checked Variants When

- Configuration guarantees the value.
- Missing data is a programmer or startup defect.
- There is no meaningful local fallback.
- Failing near the lookup is preferable to a later null dereference.

### Use Fallible Variants When

- Absence is normal.
- Content may be optional.
- A plugin may be disabled.
- The caller should decide the fallback.
- User-authored data is being queried.
- Runtime loading may fail.

### Avoid

- Calling checked variants on untrusted names or network data.
- Using safe variants everywhere and ignoring required configuration.
- Assuming naming suffixes have identical semantics across every API without checking documentation.
- Creating project APIs with inconsistent `Checked`, `Safe`, and `Try` naming.

### Sources

- [FModuleManager::GetModuleChecked](https://dev.epicgames.com/documentation/unreal-engine/API/Runtime/Core/FModuleManager/GetModuleChecked)

---

## 19. Use `CastChecked` Only for Required Type Relationships

### Recommendation

Use `CastChecked` when the object's type is guaranteed by an internal contract and failure indicates a programming
defect.

### Classification

**Epic API pattern**

### Required Relationship

```cpp
AExamplePlayerState* PlayerState =
    CastChecked<AExamplePlayerState>(
        Controller->PlayerState);
```

This is appropriate only when project configuration guarantees that the Controller uses `AExamplePlayerState`.

### Optional Relationship

```cpp
if (AInteractableActor* Interactable =
        Cast<AInteractableActor>(Candidate))
{
    Interactable->Interact();
}
```

This is appropriate when the candidate may legitimately be another type.

### Prefer Interfaces for Capabilities

Repeatedly casting unrelated objects to concrete classes may indicate that an interface is more appropriate:

```cpp
if (Candidate->Implements<UInteractable>())
{
    IInteractable::Execute_Interact(
        Candidate,
        OwnerController);
}
```

### Avoid

- Using `CastChecked` on arbitrary overlap results.
- Using it on network-provided object references without validation.
- Using ordinary `Cast` and silently returning when the type is architecturally required.
- Using casts as a replacement for stable interfaces and ownership.

---

## 20. Design `Try`, `Checked`, and Required APIs Deliberately

### Recommendation

Create separate APIs when callers need different failure contracts.

### Classification

**Epic API naming pattern and defensive-programming inference**

### Fallible API

```cpp
bool TryGetEquippedWeapon(
    UWeaponInstance*& OutWeapon) const;
```

Use when having no weapon is normal.

### Checked API

```cpp
UWeaponInstance& GetEquippedWeaponChecked() const;
```

Use when the caller operates only in a state where an equipped weapon is required.

### Optional Pointer API

```cpp
UWeaponInstance* GetEquippedWeapon() const;
```

Use only when the nullable return contract is obvious and consistently handled.

### Result API

```cpp
FEquipResult TryEquipItem(
    UItemInstance* Item);
```

Use when callers need failure reasons.

### Benefits

- Makes assumptions visible at call sites.
- Avoids repetitive defensive branches in code that already guarantees the invariant.
- Keeps normal failure separate from programming defects.
- Improves tests.
- Makes Blueprint exposure more intentional.

### Avoid

- One ambiguous function whose behavior changes depending on hidden state.
- A function named `Get` that may crash unexpectedly without a documented checked contract.
- Returning `bool` when callers need meaningful failure context.
- Adding `Safe` suffixes without defining what safety means.
- Creating a checked API for conditions controlled by external data.

---

## 21. Validate Function Inputs at the Correct Boundary

### Recommendation

Validate data once at the boundary where its trust level changes, then preserve valid invariants internally.

### Classification

**Defensive-programming principle**

### Boundaries Include

- Blueprint to C++.
- Network client to server.
- Serialized data to runtime objects.
- Data Asset to gameplay instance.
- User input to command.
- Backend response to local model.
- Plugin API to project code.
- Public module API to private implementation.
- Async worker to game-thread application.

### Example

A public request validates external conditions:

```cpp
FInventoryOperationResult
UInventoryComponent::RequestAddItem(
    const UItemDefinition* Definition,
    const int32 Quantity)
{
    if (!IsValid(Definition))
    {
        return FInventoryOperationResult::Failure(
            EInventoryError::InvalidDefinition);
    }

    if (Quantity <= 0)
    {
        return FInventoryOperationResult::Failure(
            EInventoryError::InvalidQuantity);
    }

    AddItemInternal(*Definition, Quantity);
    return FInventoryOperationResult::Success();
}
```

The internal function may rely on the validated contract:

```cpp
void UInventoryComponent::AddItemInternal(
    const UItemDefinition& Definition,
    const int32 Quantity)
{
    check(Quantity > 0);

    // Internal implementation.
}
```

### Why

Repeatedly treating all internal data as untrusted creates noise and weakens contracts.

Never validating boundary data allows invalid state to penetrate too deeply.

### Principle

> Validate at trust boundaries; assert invariants within trusted implementation layers.

---

## 22. Validate Blueprint-Callable Functions Defensively

### Recommendation

Treat Blueprint calls as an API boundary whose inputs may be incomplete, null, or incorrectly configured.

### Classification

**Inference aligned with Epic Blueprint and C++ boundary guidance**

### Why

Blueprint callers may:

- leave optional pins unconnected;
- pass null object references;
- invoke functions in unexpected lifecycle phases;
- call from derived content not known to C++;
- pass editor-authored values outside intended ranges.

### Guidance

For Blueprint-facing APIs:

- Use metadata to communicate requirements.
- Prefer enums and structs over ambiguous primitive combinations.
- Return success or failure explicitly for fallible operations.
- Log or ensure when internal content configuration violates an invariant.
- Avoid crashing on normal content-authoring mistakes when a clear validation error is more useful.
- Use content validation and editor-time diagnostics.
- Keep low-level checked implementation private when appropriate.

### Example

```cpp
UFUNCTION(BlueprintCallable, Category = "Inventory")
bool TryAddItem(
    UItemDefinition* ItemDefinition,
    int32 Quantity);
```

```cpp
bool UInventoryComponent::TryAddItem(
    UItemDefinition* ItemDefinition,
    const int32 Quantity)
{
    if (!IsValid(ItemDefinition))
    {
        UE_LOG(
            LogInventory,
            Warning,
            TEXT("%s received an invalid ItemDefinition."),
            *GetNameSafe(this));

        return false;
    }

    if (Quantity <= 0)
    {
        UE_LOG(
            LogInventory,
            Warning,
            TEXT("%s received invalid quantity %d."),
            *GetNameSafe(this),
            Quantity);

        return false;
    }

    AddItemInternal(*ItemDefinition, Quantity);
    return true;
}
```

### Avoid

- Exposing a function that performs `check(Input)` for a commonly misconfigured Blueprint pin without considering
  authoring workflow.
- Silently correcting every invalid value.
- Returning success after applying only part of the request.
- Logging every expected failed attempt as an error.
- Exposing internal checked functions directly to Blueprint.

---

## 23. Validate Data Assets and Configuration Before Runtime

### Recommendation

Move deterministic content validation into editor-time or build-time validation where practical.

### Classification

**Epic content-validation pattern**

### Validate

- Required asset references.
- Value ranges.
- Mutually exclusive settings.
- Gameplay Tag presence.
- Class compatibility.
- Asset Manager configuration.
- Unsupported combinations.
- Circular references.
- Platform restrictions.
- Duplicate identifiers.

### Why

A runtime `check` can identify invalid content, but finding the problem before packaging or play begins is better.

### Runtime Still Needs Protection

Editor validation does not eliminate runtime defensive code when:

- content can be loaded dynamically;
- builds may contain stale or externally supplied data;
- network or backend data affects selection;
- code can construct definitions at runtime;
- validation rules are version-dependent.

### Strategy

1. Validate authored content before runtime.
2. Reject invalid content at loading boundaries.
3. Assert internal invariants after successful validation.
4. Provide useful asset names and property context in diagnostics.

---

## 24. Never Trust Client-Provided Gameplay Outcomes

### Recommendation

Validate client requests on the authoritative server and calculate authoritative results from server-owned state.

### Classification

**Official Unreal networking model with security-oriented inference**

### Validate

- RPC caller ownership.
- Requested target.
- Distance and line of sight.
- Ability or action availability.
- Costs and cooldowns.
- Inventory ownership.
- Team and permission rules.
- Rate and frequency.
- State transition legality.
- Index and identifier ranges.

### Incorrect

```cpp
UFUNCTION(Server, Reliable)
void ServerApplyDamage(
    AActor* Target,
    float ClientCalculatedDamage);
```

The client declares the result.

### Better Direction

```cpp
UFUNCTION(Server, Reliable)
void ServerRequestAttack(
    AActor* Target,
    FGameplayTag AttackType);
```

The server validates the request and calculates the result.

### Response to Invalid Requests

Depending on severity:

- Reject silently for common stale requests.
- Return or replicate a rejection reason.
- Log suspicious behavior with rate limiting.
- Increment anti-abuse metrics.
- Disconnect only under a deliberate policy.
- Never rely on `check` as network security.

### Avoid

- Crashing the server because a client sent invalid data.
- Trusting client-provided positions, damage, inventory, or cooldown completion.
- Using an `ensureAlways` for spam-controlled invalid packets.
- Logging unbounded attacker-controlled strings.
- Assuming reliable RPC means trustworthy RPC.

---

## 25. Guard Against Lifetime Changes in Deferred Work

### Recommendation

Revalidate object lifetime and request relevance when deferred or asynchronous work completes.

### Classification

**Defensive lifecycle practice**

### Example

```cpp
void UThumbnailLoader::LoadThumbnail(
    TSoftObjectPtr<UTexture2D> ThumbnailAsset)
{
    TWeakObjectPtr<UThumbnailLoader> WeakThis(this);

    RequestAsyncLoad(
        ThumbnailAsset,
        [WeakThis, ThumbnailAsset]()
        {
            UThumbnailLoader* StrongThis = WeakThis.Get();
            if (!StrongThis)
            {
                return;
            }

            UTexture2D* Thumbnail = ThumbnailAsset.Get();
            if (!ensure(Thumbnail))
            {
                return;
            }

            StrongThis->ApplyThumbnail(Thumbnail);
        });
}
```

### Revalidate

- Request owner still exists.
- World is still current.
- Pawn has not been replaced.
- Request generation is current.
- Target still exists.
- Feature remains active.
- Result still matches the requested identifier.
- Callback is on the required thread.
- Server authority remains applicable.

### Avoid

- Checking validity only before starting asynchronous work.
- Capturing raw `this`.
- Applying old responses to a new session.
- Assuming cancellation prevents every callback race.
- Using one boolean to represent several independent request generations.

---

## 26. Use Logging to Explain Recoverable Failures

### Recommendation

Log failures that require investigation or operational visibility, using an appropriate category and verbosity.

### Classification

**Official Epic logging mechanism**

### Useful Diagnostic Context

Include information such as:

- Object name.
- Actor role or network context.
- Player or connection identifier where safe.
- Asset path.
- Gameplay Tag.
- Requested operation.
- State.
- Failure reason.
- Relevant values.
- Whether fallback occurred.

### Example

```cpp
UE_LOG(
    LogInventory,
    Warning,
    TEXT("%s rejected item %s because quantity %d is invalid."),
    *GetNameSafe(this),
    *GetNameSafe(ItemDefinition),
    Quantity);
```

### Select Verbosity Deliberately

| Verbosity     | General Use                                   |
|---------------|-----------------------------------------------|
| `Fatal`       | Unrecoverable condition; terminates execution |
| `Error`       | Operation failed and needs investigation      |
| `Warning`     | Unexpected or degraded behavior               |
| `Display`     | User-visible development-console information  |
| `Log`         | Ordinary diagnostic record                    |
| `Verbose`     | Detailed opt-in diagnostics                   |
| `VeryVerbose` | Extremely detailed opt-in diagnostics         |

### Avoid

- Logging every expected branch.
- Logging the same failure at every stack layer.
- Logging every frame.
- Using `Error` for harmless optional absence.
- Using vague messages such as `Something failed`.
- Logging secrets, credentials, or excessive personal data.
- Logging and continuing through unsafe state.
- Logging instead of returning an error to the caller.

### Sources

- [Logging in Unreal Engine](https://dev.epicgames.com/documentation/unreal-engine/logging-in-unreal-engine)
- [ELogVerbosity::Type](https://dev.epicgames.com/documentation/unreal-engine/API/Runtime/Core/ELogVerbosity__Type)

---

## 27. Avoid Duplicate Diagnostics

### Recommendation

Report a failure at the layer with enough context to explain it, rather than logging the same failure at every
propagation layer.

### Classification

**Defensive observability practice**

### Noisy Pattern

```cpp
bool LoadDefinition()
{
    if (!ReadFile())
    {
        UE_LOG(LogExample, Error, TEXT("ReadFile failed."));
        return false;
    }

    return true;
}

bool InitializeFeature()
{
    if (!LoadDefinition())
    {
        UE_LOG(LogExample, Error, TEXT("LoadDefinition failed."));
        return false;
    }

    return true;
}

bool StartGame()
{
    if (!InitializeFeature())
    {
        UE_LOG(LogExample, Error, TEXT("InitializeFeature failed."));
        return false;
    }

    return true;
}
```

One failure creates three errors without necessarily adding useful context.

### Better Strategies

- Log at the source with complete technical details and propagate a structured result.
- Log at the boundary with user or feature context.
- Add context while preserving one primary diagnostic.
- Use verbose tracing for propagation when needed.

### Principle

> One failure should normally produce one actionable primary report.

---

## 28. Do Not Continue After Fatal Validation Failure

### Recommendation

After detecting a condition that makes the remainder unsafe, terminate the current operation immediately.

### Classification

**Defensive-programming principle**

### Incorrect

```cpp
if (!IsValid(Target))
{
    UE_LOG(LogCombat, Error, TEXT("Invalid target."));
}

Target->ApplyDamage(Damage);
```

### Correct

```cpp
if (!IsValid(Target))
{
    UE_LOG(LogCombat, Error, TEXT("Invalid target."));
    return;
}

Target->ApplyDamage(Damage);
```

### With Ensure

```cpp
if (!ensureMsgf(
        IsValid(Target),
        TEXT("Combat action requires a valid target.")))
{
    return;
}
```

### Why

A diagnostic without control-flow enforcement can make the eventual crash occur farther from the defect and with less
useful context.

---

## 29. Avoid Defensive Code That Masks Defects

### Recommendation

Do not add fallback behavior that makes an invalid internal state appear successful.

### Classification

**Defensive-programming principle**

### Masking Example

```cpp
float UHealthComponent::GetMaxHealth() const
{
    if (MaxHealth <= 0.0f)
    {
        return 100.0f;
    }

    return MaxHealth;
}
```

This conceals invalid initialization.

### Stronger Direction

```cpp
float UHealthComponent::GetMaxHealth() const
{
    checkf(
        MaxHealth > 0.0f,
        TEXT("%s has invalid MaxHealth %f."),
        *GetNameSafe(this),
        MaxHealth);

    return MaxHealth;
}
```

Or, when data-driven content may be invalid and recovery is required:

```cpp
float UHealthComponent::GetMaxHealth() const
{
    if (!ensureMsgf(
            MaxHealth > 0.0f,
            TEXT("%s has invalid MaxHealth %f."),
            *GetNameSafe(this),
            MaxHealth))
    {
        return MinimumSafeHealth;
    }

    return MaxHealth;
}
```

### Requirements for Fallbacks

A fallback should be:

- Explicit.
- Safe.
- Observable when unexpected.
- Semantically acceptable.
- Tested.
- Clearly separate from successful primary behavior.

### Avoid

- Default-constructing silent substitutes everywhere.
- Swallowing failed operations.
- Returning success after fallback without telling the caller.
- Repeatedly normalizing corrupt state without identifying its source.

---

## 30. Use Clamping Only When Clamping Is the Contract

### Recommendation

Clamp values only when out-of-range input is expected and saturation is the intended behavior.

### Classification

**Defensive-programming principle**

### Valid Clamping

```cpp
CurrentHealth = FMath::Clamp(
    CurrentHealth,
    0.0f,
    MaxHealth);
```

Health is intentionally bounded.

### Potentially Dangerous Clamping

```cpp
const int32 SafeIndex =
    FMath::Clamp(Index, 0, Items.Num() - 1);

return Items[SafeIndex];
```

An invalid index now accesses a different item, hiding the caller bug.

### Prefer

```cpp
if (!Items.IsValidIndex(Index))
{
    return nullptr;
}

return Items[Index];
```

Or, for an internal required index:

```cpp
check(Items.IsValidIndex(Index));
return Items[Index];
```

### Principle

> Do not transform invalid input into a different valid request unless that transformation is part of the documented
> API.

---

## 31. Validate Collection Access Explicitly

### Recommendation

Use container APIs and assertions that communicate whether missing elements are expected.

### Classification

**Epic container API pattern**

### Array Index

Expected possibility:

```cpp
if (!Items.IsValidIndex(Index))
{
    return nullptr;
}

return Items[Index];
```

Required invariant:

```cpp
check(Items.IsValidIndex(Index));
return Items[Index];
```

### Map Lookup

Optional:

```cpp
if (const FItemData* ItemData = ItemMap.Find(ItemId))
{
    return ItemData;
}

return nullptr;
```

Required:

```cpp
return &ItemMap.FindChecked(ItemId);
```

Default-value semantics:

```cpp
return ItemMap.FindRef(ItemId);
```

Use `FindRef` only when returning a default-constructed value for absence is semantically valid and distinguishable
where necessary.

### Avoid

- Index clamping to conceal invalid callers.
- Dereferencing `Find` without checking.
- Using `FindRef` when default value and stored default value are ambiguous.
- Using `FindChecked` for user-authored arbitrary identifiers without prior validation.

---

## 32. Handle Enum Exhaustiveness Deliberately

### Recommendation

Choose whether an unknown enum value is impossible, forward-compatible, or externally invalid.

### Classification

**Defensive-programming practice**

### Internal Closed Enum

```cpp
switch (State)
{
case ESystemState::Inactive:
    return HandleInactive();

case ESystemState::Active:
    return HandleActive();

case ESystemState::Suspended:
    return HandleSuspended();
}

checkNoEntry();
return false;
```

### External or Serialized Enum

```cpp
switch (Message.Type)
{
case EMessageType::Start:
    HandleStart(Message);
    return true;

case EMessageType::Stop:
    HandleStop(Message);
    return true;

default:
    UE_LOG(
        LogProtocol,
        Warning,
        TEXT("Rejected unknown message type %d."),
        static_cast<int32>(Message.Type));

    return false;
}
```

### Why

Unknown values may appear due to:

- corrupted data;
- version mismatch;
- untrusted packets;
- newly added enum values;
- invalid casts;
- uninitialized memory.

### Avoid

- `checkNoEntry` on data received from untrusted clients.
- Silently treating every unknown value as the first enum entry.
- Omitting a defensive default when processing versioned external protocols.
- Using a permissive default that grants capabilities.

---

## 33. Validate Arithmetic Preconditions

### Recommendation

Check arithmetic assumptions before operations that can overflow, divide by zero, underflow, or produce invalid ranges.

### Classification

**Defensive-programming practice supported by assertion examples**

### Examples

```cpp
check(Divisor != 0);
const int32 Result = Dividend / Divisor;
```

```cpp
if (Duration <= 0.0f)
{
    return EStartResult::InvalidDuration;
}
```

```cpp
checkf(
    RequestedCount <= MaxAllowedCount,
    TEXT("RequestedCount %d exceeds MaxAllowedCount %d."),
    RequestedCount,
    MaxAllowedCount);
```

### External Data

For external data, reject safely:

```cpp
if (Packet.ElementCount < 0 ||
    Packet.ElementCount > MaxPacketElements)
{
    return EPacketResult::Rejected;
}
```

### Consider

- Signed and unsigned conversions.
- Integer multiplication before allocation.
- Floating-point non-finite values.
- Negative durations.
- Zero-size ranges.
- Invalid normalization.
- Precision loss.
- Unit mismatches.

### Avoid

- Trusting dimensions used for allocation.
- Checking after overflow has already occurred.
- Using assertions as the only protection against malicious arithmetic values.
- Normalizing a zero-length vector without an explicit policy.

---

## 34. Validate Thread Context When Required

### Recommendation

Assert or reject execution on the wrong thread when an API has thread-affinity requirements.

### Classification

**Epic engine pattern**

### Example

```cpp
check(IsInGameThread());
```

Use when the implementation accesses game-thread-only UObject or world state.

### Async Boundary

```cpp
AsyncTask(
    ENamedThreads::GameThread,
    [WeakThis, Result = MoveTemp(Result)]()
    {
        UExampleSystem* StrongThis = WeakThis.Get();
        if (!StrongThis)
        {
            return;
        }

        StrongThis->ApplyResult(Result);
    });
```

### Avoid

- Assuming callbacks always return on the game thread.
- Accessing Actors or Components from arbitrary worker threads.
- Using a thread assertion on external callbacks without providing proper dispatch.
- Treating pointer validity as thread safety.
- Holding raw UObject pointers across unsynchronized background work.

---

## 35. Distinguish Development Diagnostics From Shipping Correctness

### Recommendation

Ensure the program remains correct when development-only assertion diagnostics are disabled.

### Classification

**Official assertion build behavior with defensive-programming inference**

### Requirements

- Do not put required side effects inside `check`.
- Do not depend on `check` to reject untrusted input.
- Do not rely on `ensure` as the only recovery logic.
- Keep bounds and security validation active in Shipping.
- Decide whether checked invariants require Shipping checks for a specific project.
- Test Shipping-like configurations before release.

### Example

Incorrect:

```cpp
check(ServerRequest.IsAuthorized());
ApplyServerRequest(ServerRequest);
```

If the check is disabled, unauthorized execution may continue.

Correct:

```cpp
if (!ServerRequest.IsAuthorized())
{
    RejectServerRequest(ServerRequest);
    return;
}

ApplyServerRequest(ServerRequest);
```

A development assertion may additionally document an internal expectation, but it must not replace security enforcement.

### `USE_CHECKS_IN_SHIPPING`

Unreal provides configuration support for enabling checks in Shipping under selected project or platform policies.

This should be a deliberate build decision, not a substitute for production error handling.

### Sources

- [Asserts in Unreal Engine](https://dev.epicgames.com/documentation/unreal-engine/asserts-in-unreal-engine)

---

## 36. Use Fatal Errors Only When Continuing Is Impossible

### Recommendation

Use fatal termination only for conditions where continuing cannot preserve meaningful or safe program behavior.

### Classification

**Official logging capability with context-dependent application**

### Example

```cpp
UE_LOG(
    LogCoreSystem,
    Fatal,
    TEXT("Required runtime registry could not be initialized."));
```

### Apply When

- Foundational process state is irreparably invalid.
- Continuing risks corruption.
- No fallback or controlled feature disablement exists.
- Startup cannot produce a meaningful application.
- The failure represents an environment the product cannot support.

### Avoid

- Fatal errors for optional assets.
- Fatal errors for temporary backend unavailability.
- Fatal errors for invalid client requests.
- Fatal errors for recoverable feature initialization.
- Using `Fatal` to make a bug more visible when an assertion is semantically correct.

---

## 37. Use Tests to Confirm Defensive Contracts

### Recommendation

Test both valid behavior and rejected invalid behavior.

### Classification

**Inference aligned with Epic automation support**

### Test

- Required preconditions.
- Optional null values.
- Invalid indices.
- Invalid state transitions.
- Duplicate operations.
- Repeated initialization.
- Repeated teardown.
- Stale async callbacks.
- Destroyed weak targets.
- Invalid assets.
- Invalid network requests.
- Boundary values.
- Enum unknowns.
- Shipping-equivalent behavior.
- Fallback correctness.
- Diagnostic rate limiting.

### Example Test Questions

- Does a `Try` function leave state unchanged on failure?
- Does an invalid request return the correct reason?
- Does an early return occur before any side effect?
- Does cleanup still run after every exit?
- Does an `ensure` recovery path remain safe?
- Does the Shipping path retain required validation?
- Can a stale request overwrite newer state?
- Does a checked API fail immediately when its invariant is violated?

### Principle

> A defensive branch that has never been tested is only an assumption about recovery.

---

## Assertion Selection Guide

| Situation                                                        | Recommended Starting Point            |
|------------------------------------------------------------------|---------------------------------------|
| Internal condition must be true; continuing is unsafe            | `check`                               |
| Same, with useful values or object names                         | `checkf`                              |
| Expensive internal invariant                                     | `checkSlow` / `checkfSlow`            |
| Code path must be impossible                                     | `checkNoEntry`                        |
| Function must not be re-entered                                  | `checkNoReentry`                      |
| Function must not recurse                                        | `checkNoRecursion`                    |
| Expression must execute in all builds and is expected to succeed | `verify` / `verifyf`                  |
| Unexpected condition, safe recovery exists                       | `ensure` / `ensureMsgf` plus recovery |
| Every unexpected occurrence must be reported                     | `ensureAlways` / `ensureAlwaysMsgf`   |
| Failure is expected                                              | Explicit result or error handling     |
| Input is untrusted                                               | Runtime validation and rejection      |
| Failure makes startup or execution impossible                    | Fatal error or controlled termination |

---

## Pointer Defense Guide

| Situation                                  | Recommended Starting Point                               |
|--------------------------------------------|----------------------------------------------------------|
| Required UObject dependency                | Strong reference plus `check` at contract boundary       |
| Optional UObject dependency                | Null or `IsValid` check                                  |
| UObject may be destroyed independently     | `TWeakObjectPtr` and `Get()`                             |
| Asset may be unloaded                      | `TSoftObjectPtr` / `TSoftClassPtr` and loading policy    |
| Required concrete type                     | `CastChecked`                                            |
| Optional concrete type                     | `Cast`                                                   |
| Optional capability across unrelated types | Unreal interface                                         |
| Native shared object                       | `TSharedPtr` and `IsValid()`                             |
| Untrusted object reference                 | Authority, ownership, lifetime, and semantic validation  |
| Suspected memory corruption                | Specialized debugging tools, not routine gameplay checks |

---

## Early Return Guide

| Condition                                         | Typical Response                                     |
|---------------------------------------------------|------------------------------------------------------|
| Feature inactive                                  | Silent early return                                  |
| Optional target absent                            | Silent early return or `false`                       |
| Required dependency unexpectedly absent           | `ensure` plus early return, or `check`               |
| Caller violated a recoverable public API contract | Error result                                         |
| Caller violated a required internal contract      | `check`                                              |
| Wrong authority                                   | Early return, possibly diagnostic during development |
| Stale async response                              | Silent early return or verbose trace                 |
| Invalid user input                                | Validation result                                    |
| Invalid client request                            | Server rejection                                     |
| External operation failed                         | Explicit failure result and contextual log           |
| Main operation cannot continue safely             | Diagnostic followed by immediate return              |

---

## Defensive API Naming Guide

| Name Pattern     | Intended Contract                                                            |
|------------------|------------------------------------------------------------------------------|
| `Try...`         | Failure is expected and explicitly communicated                              |
| `...Checked`     | Failure is an internal programming error                                     |
| `...Safe`        | Absence or failure is tolerated according to documented semantics            |
| `Find...`        | Lookup may fail unless documentation states otherwise                        |
| `FindChecked...` | Entry must exist                                                             |
| `Can...`         | Query whether an operation is currently permitted                            |
| `Request...`     | Ask an authoritative or asynchronous system to attempt an operation          |
| `Validate...`    | Produce validity information without performing the primary operation        |
| `Ensure...`      | Avoid this name when it could be confused with Unreal's `ensure` diagnostics |
| `Get...`         | Return contract must clearly state whether null or failure is possible       |

Project APIs should apply these suffixes consistently.

---

## Defensive Programming Review Checklist

### Failure Classification

- [ ] The condition is classified as an invariant violation, expected failure, external failure, or untrusted input.
- [ ] The selected mechanism matches that classification.
- [ ] Expected failures do not crash the process.
- [ ] Internal defects are not silently ignored.
- [ ] Continuing after a failure is demonstrably safe.

### Assertions

- [ ] `check` is used only for required internal conditions.
- [ ] `checkf` includes actionable context.
- [ ] `verify` is used only when expression evaluation must remain.
- [ ] `ensure` has an explicit safe recovery path.
- [ ] `ensureAlways` will not generate uncontrolled volume.
- [ ] No required side effect exists only inside `check`.
- [ ] Slow checks are not required for correctness.
- [ ] Shipping behavior remains correct when diagnostics are disabled.

### Preconditions and Invariants

- [ ] Function preconditions are explicit.
- [ ] Inputs controlled by callers are distinguished from external data.
- [ ] Internal invariants are checked near their source.
- [ ] Postconditions are validated where the implementation is complex.
- [ ] Types make invalid calls difficult where practical.
- [ ] Assertions have no side effects.

### Early Returns

- [ ] Guard clauses remove invalid paths before the main operation.
- [ ] The valid path is easy to identify.
- [ ] Silent returns represent genuinely normal conditions.
- [ ] Unexpected returns include an ensure, log, or error result where appropriate.
- [ ] No side effect occurs before all required validation completes.
- [ ] Cleanup executes for every return path.
- [ ] Multiple returns do not obscure the function's result.
- [ ] Guard checks are ordered from fundamental and cheap to specific and expensive.

### Pointers

- [ ] Pointer validation matches the pointer type.
- [ ] Required pointers are not treated as silently optional.
- [ ] Optional UObjects use an appropriate validity check.
- [ ] Weak pointers are resolved at use time.
- [ ] Soft references have a loading policy.
- [ ] Low-level validity checks are not used routinely.
- [ ] Deferred callbacks do not capture unsafe raw UObject pointers.
- [ ] Longer-lived systems do not retain stale world objects.

### API Design

- [ ] Fallible APIs communicate failure explicitly.
- [ ] Checked APIs are reserved for internal required contracts.
- [ ] `Try`, `Checked`, and `Safe` naming is consistent.
- [ ] Callers receive failure details when they need them.
- [ ] Invalid state is prevented structurally where possible.
- [ ] Blueprint APIs validate content-facing inputs.
- [ ] Internal checked implementations are separated from public fallible APIs where useful.

### Collections and Values

- [ ] Indices are validated or asserted.
- [ ] Missing map entries use the appropriate lookup variant.
- [ ] Clamping is used only when saturation is the contract.
- [ ] Arithmetic ranges are validated.
- [ ] Enum unknowns are handled according to trust boundary.
- [ ] Default values do not conceal missing required data.

### Networking

- [ ] Client-provided results are not trusted.
- [ ] Server RPC inputs are validated.
- [ ] Authority and ownership are checked.
- [ ] Invalid requests are rejected without crashing the server.
- [ ] Security validation remains active in Shipping.
- [ ] Diagnostic output is rate-limited where abuse is possible.

### Asynchronous Work

- [ ] Requester lifetime is revalidated at completion.
- [ ] Stale generations are rejected.
- [ ] World and session changes are handled.
- [ ] Callbacks execute on the required thread.
- [ ] Cancellation and late completion are both safe.
- [ ] Results cannot be applied to replacement objects accidentally.

### Logging

- [ ] Log category is specific.
- [ ] Verbosity matches severity.
- [ ] Messages include actionable context.
- [ ] The same failure is not reported redundantly.
- [ ] High-frequency failures are rate-limited.
- [ ] Sensitive information is excluded.
- [ ] Logging does not replace caller-visible error handling.
- [ ] Fatal logging is reserved for unrecoverable conditions.

### Testing

- [ ] Invalid inputs are tested.
- [ ] Rejected operations leave state consistent.
- [ ] Fallback behavior is tested.
- [ ] Ensure recovery paths are tested.
- [ ] Async destruction races are tested.
- [ ] Shipping-equivalent validation is tested.
- [ ] Every important early-return path is covered.
- [ ] Cleanup is tested after partial execution.

---

## Common Defensive Programming Failure Modes

### Silent Null Return

```cpp
if (!RequiredComponent)
{
    return;
}
```

The component is architecturally required, but the defect disappears as missing behavior.

**Correction:** Use `check`, or `ensure` plus a deliberate safe fallback.

### Check on External Failure

```cpp
check(HttpResponse.IsSuccessful());
```

HTTP failure is a normal possibility.

**Correction:** Return or propagate an explicit external-operation error.

### Ensure Without Recovery

```cpp
ensure(Target);
Target->Execute();
```

The code reports the invalid condition and then continues unsafely.

**Correction:** Branch on the ensure result and stop or fall back.

### Side Effect Inside Check

```cpp
check(RegisterRequiredService());
```

The registration may not execute in builds where checks are disabled.

**Correction:** Execute first, store the result, then check or verify.

### Defensive Check Everywhere

Every internal helper repeatedly validates the same trusted inputs.

**Correction:** Validate at the boundary and assert preserved invariants internally.

### Early Return Hides Initialization Bug

A required dependency is null, so the function quietly does nothing.

**Correction:** Differentiate optional state from broken setup.

### Log and Continue

The function logs invalid state but continues to mutate data.

**Correction:** Pair the diagnostic with immediate control-flow termination.

### Clamp the Invalid Request

An invalid array index is clamped to a valid item.

**Correction:** Reject or assert the invalid index.

### `IsValidLowLevel` as Pointer Safety

Gameplay code calls low-level validity methods on potentially stale pointers.

**Correction:** Fix ownership and use strong or weak UObject references correctly.

### `Cast` Hides Required Configuration

A project guarantees one PlayerState type, but every use performs `Cast` and silently returns on failure.

**Correction:** Assert the configured type relationship with `CastChecked` at the appropriate internal boundary.

### `CastChecked` on Arbitrary World Data

An overlap Actor is assumed to be one concrete class.

**Correction:** Use optional `Cast` or an interface.

### Ensure Spam

A repeated Tick condition uses `ensureAlwaysMsgf`.

**Correction:** Fix the state, disable the invalid execution path, and report once or rate-limit diagnostics.

### Duplicate Error Logs

Every layer logs the same propagated failure.

**Correction:** Choose one primary reporting layer and pass structured context.

### Check as Security

A server uses `check` to validate a client-supplied value.

**Correction:** Use active Shipping validation and reject the request.

### Async Raw Capture

A callback captures `this` after the UObject may have been destroyed.

**Correction:** Capture a weak reference and revalidate at callback time.

### Too Many Early Returns

A long function contains many exits, partial mutations, and inconsistent cleanup.

**Correction:** Split the function into validation and execution stages, use RAII, or return a structured result.

### No Early Return

The main operation is buried under deeply nested prerequisites.

**Correction:** Convert exceptional conditions into ordered guard clauses.

---

## Recommended Function Structure

A defensive Unreal function commonly follows this shape:

```cpp
FOperationResult UExampleComponent::TryExecuteOperation(
    AActor* Target,
    const FOperationRequest& Request)
{
    // 1. Object and lifecycle prerequisites.
    if (!IsActive())
    {
        return FOperationResult::Failure(
            EOperationError::Inactive);
    }

    if (!IsValid(Target))
    {
        return FOperationResult::Failure(
            EOperationError::InvalidTarget);
    }

    // 2. Authority and ownership.
    if (!GetOwner()->HasAuthority())
    {
        return FOperationResult::Failure(
            EOperationError::NotAuthoritative);
    }

    // 3. Request validation.
    if (!Request.IsValid())
    {
        return FOperationResult::Failure(
            EOperationError::InvalidRequest);
    }

    // 4. Domain rules.
    if (!CanExecuteOperation(Target, Request))
    {
        return FOperationResult::Failure(
            EOperationError::NotPermitted);
    }

    // 5. Internal invariants.
    check(RequiredRuntimeState);

    // 6. Side effects begin only after validation.
    ApplyOperation(Target, Request);

    // 7. Postcondition where valuable.
    checkSlow(IsInternalStateConsistent());

    return FOperationResult::Success();
}
```

This structure separates:

- normal rejection;
- authority;
- untrusted input;
- domain rules;
- internal invariants;
- mutation;
- postconditions.

---

## Core Principles

### Fail Near the Defect

Detect invalid internal state as close as possible to where it first becomes invalid.

### Reject Untrusted Data Safely

Never depend on development assertions to protect runtime boundaries.

### Do Not Continue Unsafely

An error report is not a recovery strategy.

### Keep the Valid Path Clear

Use guard clauses so the main operation remains readable.

### Make Failure Part of the API

Expected failures should be visible in function names, return types, and call sites.

### Preserve Invariants Internally

After data crosses a validated boundary, internal code should be able to rely on established contracts.

### Prefer Prevention Over Detection

Strong types, constrained APIs, explicit ownership, and correct lifetimes prevent entire classes of invalid state.

### Keep Shipping Correct

Development diagnostics may disappear; essential validation and control flow must not.

---

## Core Conclusion

Defensive programming in Unreal Engine is not the indiscriminate addition of null checks.

It is the deliberate separation of:

- impossible internal states;
- violated programmer contracts;
- unexpected recoverable defects;
- normal gameplay rejection;
- external-system failure;
- untrusted input;
- object-lifetime changes.

The strongest recurring rule is:

> Assert what correct internal code must guarantee, validate what the program cannot trust, return early when the
> current operation cannot continue, and never hide a broken invariant behind silent failure.

---

## Primary Sources

- [Asserts in Unreal Engine](https://dev.epicgames.com/documentation/unreal-engine/asserts-in-unreal-engine)
- [Epic C++ Coding Standard](https://dev.epicgames.com/documentation/unreal-engine/epic-cplusplus-coding-standard-for-unreal-engine)
- [Logging in Unreal Engine](https://dev.epicgames.com/documentation/unreal-engine/logging-in-unreal-engine)
- [Crash Reporting in Unreal Engine](https://dev.epicgames.com/documentation/unreal-engine/crash-reporting-in-unreal-engine)
- [IsValid API](https://dev.epicgames.com/documentation/unreal-engine/API/Runtime/CoreUObject/UObject/IsValid)
- [Unreal Object Handling](https://dev.epicgames.com/documentation/unreal-engine/unreal-object-handling-in-unreal-engine)
- [FModuleManager::GetModuleChecked](https://dev.epicgames.com/documentation/unreal-engine/API/Runtime/Core/FModuleManager/GetModuleChecked)
- [TArray: Arrays in Unreal Engine](https://dev.epicgames.com/documentation/unreal-engine/array-containers-in-unreal-engine)
- [Networking and Multiplayer](https://dev.epicgames.com/documentation/unreal-engine/networking-and-multiplayer-in-unreal-engine)