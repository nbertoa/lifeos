# Unreal Engine C++ Coding Standard

## Research Metadata

- **Research date:** 2026-07-22
- **Documentation baseline:** Unreal Engine 5.8
- **Primary authority:** Epic Games C++ Coding Standard
- **Status:** Living document

---

# Purpose

Epic's coding standard is not primarily about style.

Its purpose is to make a codebase consisting of millions of lines remain understandable, maintainable, portable and
compatible with Unreal's tooling.

Consistency reduces cognitive load.

Every engineer should be able to open an unfamiliar file and immediately recognize:

- what something is,
- who owns it,
- how it should be used,
- and whether it follows Unreal conventions.

---

# Core Principles

Across the entire coding standard, Epic repeatedly optimizes for:

- readability;
- consistency;
- maintainability;
- explicit intent;
- portability;
- tooling compatibility;
- safe API design.

Whenever two implementations are equally correct, Epic generally prefers the one that is easier for another engineer to
understand.

---

# Prefer Self-Documenting Code

## Recommendation

Write code whose purpose is obvious without requiring comments.

### Why

Names remain synchronized with code.

Comments frequently do not.

Good names reduce documentation burden.

### Prefer

```cpp
ApplyDamage(Target);
```

instead of

```cpp
DoIt(Target);
```

---

# Use Unreal Naming Conventions

Epic's prefixes communicate semantic meaning.

Examples:

| Prefix | Meaning      |
|--------|--------------|
| U      | UObject      |
| A      | Actor        |
| F      | Struct       |
| S      | Slate widget |
| T      | Template     |
| I      | Interface    |
| E      | Enum         |
| b      | Boolean      |

A reader immediately knows the type before opening its declaration.

This consistency is relied upon throughout the engine.

---

# Prefer const Correctness

## Recommendation

Mark values immutable whenever mutation is not intended.

Benefits:

- documents intent;
- enables compiler diagnostics;
- reduces accidental mutation;
- improves reasoning.

Const should be applied consistently to:

- parameters;
- local variables;
- member functions;
- references;
- pointers where appropriate.

---

# Prefer References Over Copies

Pass large objects by const reference.

Avoid unnecessary copies.

Typical examples:

- FString
- FText
- FTransform
- FGameplayTagContainer

Small trivial types can usually be copied.

---

# Pass Ownership Explicitly

Function signatures should reveal ownership.

Bad APIs hide who owns the result.

Good APIs make ownership obvious.

---

# Minimize Includes

Include only what is required.

Prefer:

- forward declarations;
- private includes;
- module boundaries.

Reducing includes improves compile times across large projects.

---

# Separate Public and Private APIs

Headers define contracts.

Implementation belongs in cpp files.

Do not expose implementation details unnecessarily.

Every public symbol becomes part of your project's API surface.

---

# Keep Classes Small

Large classes usually indicate too many responsibilities.

Warning signs:

- thousands of lines;
- unrelated methods;
- unrelated member variables;
- multiple reasons to change.

Large classes should be decomposed.

---

# Prefer Composition

Favor Components and composition over inheritance when responsibilities are independent.

Inheritance should represent an "is-a" relationship.

---

# Avoid Deep Inheritance

Large inheritance trees increase coupling.

Inheritance should model specialization.

Not code reuse.

---

# Use Enums Instead of Magic Numbers

Bad:

```cpp
State = 3;
```

Good:

```cpp
State = EMovementState::WallRunning;
```

Meaning should be explicit.

---

# Prefer Strong Types

Use dedicated types instead of primitive values whenever meaning matters.

Example:

Bad

```cpp
float Health;
float Damage;
```

Better

```cpp
FHealth
FDamage
```

when semantics become sufficiently rich.

---

# Use Assertions

Epic provides:

- check
- checkf
- ensure
- ensureMsgf

These communicate assumptions.

Use them to detect programmer errors.

Do not use assertions for expected runtime failures.

---

# Log Intentionally

Logs are diagnostics.

Not user-facing messages.

Avoid:

- spam;
- every-frame logs;
- vague messages.

Prefer:

- relevant context;
- useful values;
- actionable information.

---

# Avoid Tick By Default

Per-frame work should be justified.

Prefer:

- events;
- delegates;
- timers;
- Gameplay Tags;
- Gameplay Effects;
- callbacks.

---

# Expose Minimal Blueprint APIs

Every BlueprintCallable function becomes part of the project's scripting API.

Expose:

- stable operations;
- intentional extension points.

Avoid exposing internal implementation.

---

# Prefer Data Over Hardcoded Values

Gameplay tuning belongs in:

- Data Assets;
- Gameplay Effects;
- Config;
- Curves;
- Data Tables;

not scattered constants.

---

# Keep Functions Focused

Functions should have one responsibility.

Warning signs:

- dozens of local variables;
- deeply nested branches;
- unrelated side effects.

---

# Prefer Early Returns

Reduce nesting.

Instead of

```cpp
if (...)
{
    if (...)
    {
        ...
    }
}
```

prefer

```cpp
if (...)
{
    return;
}

...
```

---

# Avoid Hidden Side Effects

Functions should do what their names imply.

Unexpected mutations increase bugs.

---

# Design Stable APIs

Changing public APIs has cascading cost.

Prefer thoughtful interfaces over exposing everything.

---

# Think About Lifetime

Never assume a UObject will remain valid.

Consider:

- ownership;
- GC;
- travel;
- destruction;
- async callbacks.

---

# Follow Unreal Patterns

When the engine consistently solves a problem one way, prefer understanding that pattern before inventing a different
architecture.

Consistency with engine patterns reduces surprise.

---

# Checklist

- [ ] Names communicate intent.
- [ ] Types follow Unreal conventions.
- [ ] Const correctness applied.
- [ ] Copies minimized.
- [ ] Includes minimized.
- [ ] APIs intentional.
- [ ] Classes focused.
- [ ] Functions focused.
- [ ] Tick justified.
- [ ] Assertions used appropriately.
- [ ] Logging useful.
- [ ] Blueprint API minimal.
- [ ] Lifetime considered.
- [ ] Engine conventions respected.

---

# Core Conclusion

Epic's C++ standard is fundamentally about communication.

It optimizes for engineers reading the code years later.

The strongest recurring principle is:

> Make the correct design the easiest design to understand.