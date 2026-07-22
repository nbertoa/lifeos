# Debugging

## Purpose

This document captures my approach to debugging software systems.

Debugging is not about randomly trying fixes until something works. It is a structured process of reducing uncertainty,
validating assumptions, and identifying the root cause of a problem.

The objective is not only to solve the current issue, but to improve my understanding of the system.

---

# Philosophy

## Understand Before Fixing

I avoid changing code before I understand why the problem exists.

Quick fixes may hide the symptom while leaving the underlying issue unresolved.

Understanding the cause produces better long-term solutions.

---

## Every Bug Has a Cause

Bugs are not random.

If software behaves unexpectedly, there is always an explanation, even if it is currently unknown.

The objective of debugging is to discover that explanation.

---

## Reduce Uncertainty

I try to eliminate assumptions as quickly as possible.

Every test, log, breakpoint, or experiment should answer a specific question.

Debugging should increase confidence, not create more guesses.

---

# Process

## 1. Reproduce the Issue

The first objective is obtaining a reliable way to reproduce the problem.

A bug that cannot be reproduced cannot be investigated efficiently.

---

## 2. Define the Expected Behavior

Before looking for the problem, I clarify what should happen.

Unexpected behavior is only meaningful when the expected behavior is clearly understood.

---

## 3. Isolate the Problem

I reduce the scope until only the smallest failing scenario remains.

Smaller problems are easier to reason about.

---

## 4. Form a Hypothesis

Instead of changing code randomly, I create a hypothesis explaining why the issue occurs.

Every debugging action should validate or invalidate that hypothesis.

---

## 5. Verify the Root Cause

Finding where the error appears is not enough.

I try to understand why it happens.

The first visible symptom is often not the real source of the bug.

---

## 6. Implement the Fix

Once the cause is understood, I implement the smallest solution that correctly resolves it.

Whenever possible, I avoid introducing unnecessary complexity.

---

## 7. Verify the Solution

A fix is complete only after confirming:

- the original issue is solved;
- no obvious regressions were introduced;
- the system still behaves as expected.

---

# Common Mistakes

## Guessing

Changing code without a hypothesis usually creates more confusion.

---

## Multiple Changes at Once

Changing several things simultaneously makes it impossible to know which modification actually solved the issue.

---

## Fixing Symptoms

A disappearing symptom does not necessarily mean the bug was solved.

---

## Ignoring Evidence

Logs, debugger output, assertions, and reproduction steps are evidence.

Assumptions are not.

---

# Unreal Engine

## Prefer Engine Tools

Before writing additional debugging code, I prefer using Unreal's existing debugging tools whenever appropriate.

Examples include:

- Visual Studio debugger
- Blueprint Debugger
- Gameplay Debugger
- Output Log
- Message Log
- Unreal Insights
- DrawDebug helpers

---

## Logging With Purpose

Logs should answer specific questions.

Excessive logging without intent creates noise instead of information.

---

## Blueprint and C++

The debugging strategy should be independent of the implementation language.

Whether the issue exists in Blueprint or C++, the objective remains the same:

- understand the problem;
- identify the cause;
- validate the solution.

---

# Continuous Improvement

Every bug teaches something about the software.

Whenever a debugging session reveals a recurring mistake or a misunderstanding of the engine, I try to extract a
reusable lesson instead of treating it as an isolated incident.