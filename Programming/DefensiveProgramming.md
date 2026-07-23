# Defensive Programming

## Purpose

Defensive programming is not the practice of adding checks everywhere. It is the discipline of making assumptions explicit, detecting broken contracts close to their source, preventing invalid state from spreading, and choosing failure behavior deliberately. Its purpose is to make normal control flow readable while making unsafe conditions visible and contained.

Excessive validation can obscure the design problem it is meant to prevent. Repeatedly checking an unclear dependency does not establish ownership, and returning silently from every unexpected condition can create false confidence. A check is useful only when its failure has an understood meaning and a deliberate consequence.

## Core Principle

> Validate at trust boundaries, assert internal contracts, and design invalid states out where practical.

Defensive programming begins with API and data design, not with macros. Prefer strong types, constrained construction, clear ownership, narrow interfaces, explicit optionality, and immutable state where they clarify the model. These choices make whole categories of invalid input impossible or difficult to construct.

Checks should reinforce a sound design, not compensate for an unclear one. When a condition must be checked throughout the system, reconsider whether the responsibility, lifecycle, or representation is wrong.

## Expected and Unexpected Failure

Expected failure is a normal outcome that callers can reasonably encounter and handle: invalid user input, a failed network request, an absent optional asset, a lookup with no result, or an operation rejected by business or gameplay rules. It belongs in normal control flow and should be represented by an explicit result, optional value, error, or documented no-op.

Unexpected failure is evidence that an internal contract has been broken: a required dependency was not initialized, an invariant no longer holds, supposedly unreachable code executes, or an object violates its lifecycle. It should be surfaced loudly and close to its source because continuing may spread corruption or conceal the actual defect.

Not every failure is exceptional, and programming errors should not be hidden behind ordinary return values. The question is not whether a condition is inconvenient, but whether the system was designed to encounter and recover from it.

## Preconditions, Postconditions, and Invariants

A precondition is what a caller must provide before an operation can run. A postcondition is what the operation guarantees when it completes. An invariant is a condition that must remain true for a valid object or subsystem during its defined lifetime.

The owner of a public or trust-boundary API is responsible for validating or sanitizing untrusted input. The caller is responsible for satisfying documented preconditions. Tightly controlled internal code may assert those preconditions instead of repeatedly validating them. The implementation is responsible for preserving its postconditions and invariants, including when it returns early or reports failure.

Reject invalid input when it can legitimately arrive. Assert when a controlled caller violates a contract. Sanitize external input before it enters the trusted model. Return an explicit failure result when the caller needs to choose a different action. The mechanism should express who can act on the failure and where that action belongs.

## Invalid States

> Prefer making invalid states unrepresentable over repeatedly detecting them.

Factories or constructors can establish validity before an object exists. Enums can replace loosely related booleans; dedicated value types can restrict invalid values; and explicit optional types can distinguish absence from a default value. Separate initialized from uninitialized states, avoid partially valid objects, restrict mutation, and expose only operations that preserve the model.

Some frameworks require staged initialization. In Unreal Engine, lifecycle and reflection constraints can make a fully valid object unavailable at construction time. Such transitional states should be explicit, narrow, and guarded at the boundary where they become usable rather than treated as a permanent excuse for null checks everywhere.

## Early Returns and Guard Clauses

Guard clauses are useful for failed preconditions, unavailable optional dependencies, unsupported states, and authorization failures. They reduce nesting and keep the successful path visible. When an object reference may legitimately be absent, a direct guard is often clearer than nesting the entire operation:

```cpp
if (!IsValid(Target))
{
    return;
}
```

This is appropriate only when absence is expected and doing nothing is an understood outcome. If `Target` is required at this point, the same return would suppress an invariant violation; logging, asserting, or returning a meaningful failure may be required instead. An early return is not automatically defensive when it hides a bug.

There is no universal rule requiring one return statement or requiring early returns everywhere. Choose the structure that makes preconditions, failure behavior, and the normal path easiest to understand.

## Assertions and Runtime Validation

Assertions expose programmer errors and broken internal assumptions. Recoverable validation handles conditions that can legitimately occur in production. Error reporting communicates a failure; logging adds diagnostic context; a returned failure value lets the caller decide what to do next. These roles can be combined, but they should not be confused.

Do not continue after a broken invariant unless the code can demonstrate that state remains valid. Do not use assertions for external input that must be handled in shipping behavior. Do not rely on checks that may be disabled in a build when the operation requires a production safeguard. Logging without a control-flow decision often only records corruption after it has spread. Duplicating the same validation at every layer also obscures the responsible boundary.

## Pointer and Reference Safety

A required dependency and an optional dependency should be distinguishable in the API. A pointer may communicate optionality or observation; a reference or stronger type may better represent a requirement. Ownership, lifetime, and the right to access an object must be understood separately from whether its address is non-null.

`nullptr` can be an expected state, such as an optional target, or evidence of a programming error, such as a missing required service after initialization. External references and Unreal object references may become stale or pending destruction, so they should be validated near the boundary before use. Repeated null checks inside trusted code are a signal to reconsider ownership or lifecycle design.

For a missing required dependency, assert or fail explicitly. For a missing optional dependency, branch normally. For an external or stale object reference, validate before use. Blindly returning on every null pointer hides the difference between these cases and makes initialization failures harder to diagnose.

## Error Communication

Choose an error mechanism according to what the caller needs to know. A boolean can express a simple expected outcome; a nullable or optional result can express absence; a result/error type or error enum can preserve a reason; an exception may be appropriate where the environment supports it; and an assertion can expose an internal contract failure.

The caller should be able to determine whether failure was expected, whether retry is meaningful, whether state changed partially, and whether it is safe to continue. Avoid generic success/failure signals when different reasons require different behavior. A log plus a failure result is useful when both diagnosis and caller control are needed. Fatal termination is reserved for unrecoverable corruption where continuing cannot preserve a valid system.

## Recovery and Containment

Detection does not require local recovery. The correct response may be to fail fast, roll back, use a defined fallback, propagate the failure, isolate a subsystem, or stop the process. Recovery is valid only when the code can restore or preserve a known valid state. Ignoring a failure is not recovery.

Use transactional thinking where practical: validate before mutation, perform related changes atomically when possible, and leave the object in a defined state on failure. A degraded feature can be acceptable when its boundaries are clear; continuing with corrupted shared state is not. The closer the response is to the source of the failed assumption, the more useful its diagnostic context will be.

## Unreal Engine Application

Unreal mechanisms should be selected by failure category, not treated as interchangeable null-checking tools. Exact behavior varies by build configuration and engine version, so code-level choices should be verified against current official Unreal Engine source or documentation.

- `check` and `checkf` express internal assumptions that must hold; use them when continuing would be invalid, not for expected player, network, content, or external-input failures.
- `ensure` and `ensureMsgf` report an unexpected condition while allowing the current operation to stop or degrade only when valid state can be preserved. They are not permission to continue blindly.
- `ensureAlways` is appropriate rarely, when repeated reporting is genuinely valuable and will not create noise.
- `verify` suits an expression that must execute in all relevant builds while its result is still validated in assertion-enabled configurations. Do not hide unclear side effects in assertion-oriented expressions.
- `IsValid` is useful for object references that may legitimately be stale or pending destruction. It is not a universal replacement for clear ownership and lifecycle contracts.
- Logging and explicit return values communicate expected runtime outcomes and should tell the caller or operator what happened and what control flow follows.

## Common Failure Modes

- Returning silently on every invalid pointer, which hides missing initialization and ownership defects.
- Validating the same invariant at every call layer instead of fixing the responsible contract.
- Using `ensure` for routine control flow or `check` on external input.
- Logging an error and then continuing with invalid state.
- Swallowing errors with a default value that changes caller behavior without explanation.
- Creating partially initialized objects and treating their state as normal.
- Defensively copying data without an ownership or lifetime justification.
- Catching every exception or using overly broad fallbacks that erase the failure cause.
- Adding checks that no caller can act on.
- Treating symptoms instead of correcting the contract that allowed them.

## Review Checklist

- Is this failure expected or unexpected?
- Is the assumption explicit, and is validation at the correct boundary?
- Can design prevent the invalid state instead?
- Does the chosen mechanism match the failure category?
- Can execution safely continue, and is an early return hiding a bug?
- Are required and optional dependencies distinguishable?
- Is the object left in a valid state after failure?
- Will the caller understand why the operation failed and whether retry is possible?
- Is the same check duplicated across layers?
- Would this behavior remain appropriate in a shipping build?

## Long-Term Standard

Defensive programming should make contracts clearer, not control flow noisier. Expected failures belong in normal program logic, while broken invariants should be exposed near their source. Recovery must preserve a known valid state, and repeated checks often indicate a design or ownership problem. Prefer preventing invalid states over detecting them repeatedly. Fail deliberately, not accidentally and not silently.
