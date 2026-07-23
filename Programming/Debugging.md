# Debugging

## Purpose

Debugging is not random code modification until a symptom disappears. It is the disciplined reduction of uncertainty: a process for moving from incomplete observations to justified conclusions. It should answer what happened, what should have happened, under which conditions failure occurs, where actual behavior first diverges from expected behavior, what evidence supports the proposed cause, and how the fix is known to be correct.

Fixing a visible symptom is not necessarily fixing a defect. A durable investigation improves both behavior and understanding of the system.

## Debugging Mindset

Observe before modifying. Separate facts from interpretations, change one meaningful variable at a time, seek disconfirming evidence, and prefer simple hypotheses first without assuming they are correct. Preserve a known reproduction while investigating and document discoveries that affect later decisions.

> Confidence is not evidence.

> A debugger should try to disprove a hypothesis, not only confirm it.

Do not become attached to the first plausible explanation. Stop guessing when evidence can be collected. Investigate the system and its history, not merely the line where it crashed.

## Define the Failure

Describe expected behavior, actual behavior, affected environment, first known occurrence, frequency, severity, scope, relevant inputs, lifecycle phase, and reproducibility. This distinguishes a crash, assertion, incorrect output, missing behavior, stale state, timing issue, performance symptom, editor-only issue, shipping-only issue, network divergence, and data corruption.

Reports such as “it does not work,” “it crashes sometimes,” “the state is wrong,” or “Unreal is broken” are starting observations, not useful definitions. A precise failure statement turns uncertainty into questions that can be answered.

## Reproduce the Problem

A reliable reproduction is the most valuable debugging asset. Capture the smallest failing case, data and configuration, build type, engine version where relevant, editor versus packaged behavior, local versus networked execution, and whether clean or persisted state matters.

For intermittent failures, record timing, collect repeated traces, identify correlations, automate repetition when useful, and avoid changing several variables at once. Inability to reproduce should change the investigation strategy; it does not justify speculative fixes. Preserve the best reproduction while reducing it, because a disappearing reproduction removes evidence rather than solving the problem.

## Gather Evidence

Evidence can include call stacks, logs, assertions, breakpoints, watch expressions, state snapshots, traces, network captures, memory diagnostics, source-control history, binary or configuration differences, input data, screenshots, video, and profiling when performance is the symptom.

Add instrumentation at meaningful boundaries. Logs should answer a question and identify relevant object, state, time, and context. Avoid high-volume logs without structure, logs that only prove a line executed, missing identity or state, instrumentation that changes timing accidentally, and treating an absent log as proof without understanding buffering or control flow.

## Build and Test Hypotheses

A hypothesis is a falsifiable explanation. It should predict evidence that should exist if it is correct, evidence that should not exist, and an experiment that distinguishes it from alternatives. For difficult issues, keep a short investigation log that labels confirmed facts, likely explanations, assumptions, and unresolved questions.

```text
Observation
    ↓
Hypothesis
    ↓
Prediction
    ↓
Experiment
    ↓
Evidence
    ↓
Refine or reject
```

This is not bureaucracy. It prevents a plausible story from being mistaken for a demonstrated cause and makes a failed experiment useful rather than wasted work.

## Reduce the Search Space

Use evidence to prioritize. Reduce a case through commit bisecting, selective subsystem disabling, known data, isolated callers, temporary removal of concurrency, comparison of working and failing environments, tracing data ownership, tracing state transitions, and finding the earliest divergence.

> The closer the investigation gets to the first divergence, the less code must be explained.

The final symptom may be far from the defect. Find the first incorrect value, transition, ordering event, or lifetime violation instead of investigating every system equally.

## Understand State and Time

Many difficult bugs are temporal rather than purely logical. Investigate lifecycle, ordering, stale references, asynchronous callbacks, race conditions, delayed initialization, teardown, replication, event order, cached values, and persisted state.

Ask when a value became incorrect, who last wrote it, whether an object was valid then, whether an operation occurred before initialization, whether a callback ran after teardown, and whether the state is local, replicated, serialized, or derived. Frame order and network order can turn a locally correct operation into a system failure.

## Use Tools Deliberately

Tool selection should follow the question. Use a debugger and conditional breakpoints to inspect a path; use a data breakpoint to find who mutates a value; use a call stack to understand how failure was reached; use structured logs and traces to understand state and timing; use static analysis, sanitizers, or memory tools for relevant memory and correctness questions; use history to identify a regression; and use assertions to capture invariant violations near their origin.

For Unreal work, Unreal Insights, Visual Logger, Gameplay Debugger, console variables, engine source, and the Blueprint debugger can provide evidence when they fit the question. Tools are not an exhaustive checklist and should not be chosen because they are available rather than informative.

## Identify the Root Cause

The root cause is the actionable defect whose correction removes the failure without relying on accidental conditions. Distinguish the symptom, triggering condition, contributing factor, root cause, missing defense, and missing observability.

```text
Symptom: Null dereference.
Trigger: A callback executes during teardown.
Root cause: Callback lifetime is not tied to the owning object.
Missing defense: No validity assertion or weak-reference check.
```

A root-cause explanation should account for meaningful evidence. Do not search for philosophical ultimate causes once the actionable engineering defect is clear, but do not call the first crash location a root cause when it merely exposes a prior invalid state.

Correlation is useful for selecting an investigation path, but it is not proof of causality. A recent change, a particular map, a machine, or a timing pattern may be the trigger, an amplifier, or only the first place the defect becomes visible. Test whether removing the suspected condition removes the failure for the reason predicted, and look for cases where the condition exists without the failure. Contradictory evidence is especially valuable because it prevents a convenient explanation from becoming an incorrect fix.

When several causes interact, identify the smallest actionable contract or state transition that permits the failure. The goal is not to assign blame to a subsystem; it is to find the boundary at which the system can be made correct and observable again.

## Design the Fix

Fix the cause at the correct layer. Decide whether an invariant should be restored, invalid state prevented, ownership changed, ordering made explicit, an API contract changed, validation placed at an external boundary, or observability improved. Look for nearby duplicated defects only when the same cause makes them relevant.

Avoid a null check without understanding why null is possible, delays that hide timing bugs, indefinite retries, swallowed errors, silent resets of corrupted state, broad refactoring during urgent diagnosis, and fixing every theoretical issue discovered in the investigation. Prefer the smallest complete fix, not merely the smallest diff. [Defensive Programming](DefensiveProgramming.md) guides the choice between validation, assertions, and recovery.

## Verify the Fix

Verify that the original reproduction no longer fails, the proposed cause is no longer possible or is handled correctly, nearby behavior still works, failure paths are considered, and the result holds across relevant build and runtime contexts. Add regression protection where proportionate.

Evidence may include automated tests, targeted manual verification, repeated execution, networked scenarios, packaged builds, editor workflows, or lifecycle transitions. Recompilation alone does not verify a behavioral fix. For intermittent bugs, confidence requires repeated evidence rather than one successful run.

## Unreal Engine Considerations

Unreal investigation should account for Actor and Component lifecycle, construction versus runtime, class defaults, world context, PIE versus standalone versus packaged behavior, editor-only code, garbage collection, stale UObject references, delegates, latent actions, timers, async tasks, replication authority, client/server state, dormancy, initialization order, Blueprint execution, reflection, and shipping-build macro behavior.

Prefer current official documentation, engine source, reproducible experiments, Unreal Insights, Visual Logger, Gameplay Debugger, meaningful logs, and network emulation where relevant. Do not assume engine behavior from memory when current source or documentation can verify it. These engine-specific dimensions supplement, rather than replace, general investigation discipline.

## Common Failure Modes

- Changing code before reproducing destroys evidence and creates new variables.
- Debugging only from the final symptom misses the first divergence.
- Collecting logs without a question creates noise rather than evidence.
- Treating guesses as facts makes the next experiment unreliable.
- Modifying many variables simultaneously prevents causal conclusions.
- Fixing a crash while preserving invalid state postpones the defect.
- Hiding an invariant failure with an early return conceals ownership or lifecycle problems.
- Ignoring time and lifetime misses ordering and teardown defects.
- Blaming an external system without evidence delays investigation of local causes.
- Stopping when a symptom disappears once is weak verification for an intermittent bug.
- Broad refactoring before understanding the defect makes rollback and review harder.
- Ignoring recent changes or relevant build contexts misses common regressions.
- Leaving temporary diagnostics behind reduces clarity after their question is answered.

## Debugging Checklist

### Failure Definition

- What was expected, what happened, where, and under which conditions?
- How severe and frequent is the failure?

### Reproduction and Evidence

- Can it be reproduced, and what is the smallest case?
- Which environment details matter?
- What facts are confirmed, what remains assumed, and where does behavior first diverge?

### Hypothesis and Cause

- Is the hypothesis falsifiable, and what prediction does it make?
- What experiment distinguishes it from alternatives?
- Does the explanation identify the defect rather than only the trigger?
- Are ownership, lifecycle, or ordering involved?

### Fix and Verification

- Does the fix address the correct layer and restore or protect the invariant?
- Is it the smallest complete fix?
- Does the reproduction pass repeatedly, and are nearby behaviors safe?
- Is regression protection proportionate to the risk?

## Long-Term Standard

Define the failure before modifying the system. Preserve and reduce the reproduction. Separate facts, assumptions, and hypotheses. Find the earliest divergence, and treat time and lifetime as first-class dimensions. Gather evidence with a question in mind, fix the cause at the correct boundary, and do not hide broken invariants. Verify the explanation, not only the visible result. A good fix improves understanding as well as behavior.
