# Self Review

## Purpose

Finishing an implementation is not the same as finishing the work.

Before sharing any feature with the client, I perform a structured self-review to ensure it is correct, understandable,
maintainable, and ready for feedback.

This review is part of the development process, not an optional final step.

---

## Verify Existing Functionality

The first priority is ensuring that the new implementation has not broken existing functionality.

I verify that previously working behavior continues to work as expected.

Introducing regressions is often more expensive than implementing the new feature itself.

---

## Test Different Use Cases

I test both the expected workflow and alternative scenarios.

The goal is to confirm that the feature behaves correctly under realistic usage rather than only under ideal conditions.

The implementation should satisfy the complete requirement, not just a happy path.

---

## Refactor Before Delivery

Once the feature works correctly, I review the implementation itself.

I simplify the code where possible, improve readability, remove unnecessary complexity, and leave the code in a
maintainable state.

Working code is only the first milestone.

Readable code is the final objective.

---

## Optimize for Readability

Before considering the work complete, I ask a simple question:

> Would another developer understand this without additional explanation?

Clear naming, straightforward logic, and understandable structure are more valuable than clever implementations.

Future maintenance always matters.

---

## Validate Against the Original Requirement

Before showing the feature, I verify that it actually solves the problem the client requested.

It is easy to accidentally implement what seemed correct instead of what was actually requested.

The original requirement is always the reference.

---

## Commit Small Changes

Once I am satisfied with the implementation, I push the changes as a small, meaningful commit.

Frequent small commits make progress easier to understand and reduce the risk associated with large batches of changes.

---

## Deliver a Build

I prepare a build that the client can test directly.

The objective is to let the client validate real behavior instead of relying solely on descriptions.

Software should be demonstrated, not merely explained.

---

## Demonstrate the Result

Along with the build, I prepare a demonstration.

This usually includes a video, screenshots, or a presentation showing:

- What was implemented.
- How it works.
- Any important technical decisions.
- Remaining work, if any.

Showing the result visually reduces misunderstandings and accelerates feedback.

---

## Success Is Client Understanding

A task is not complete simply because the code works.

It is complete when the client understands what was built, validates that it solves the intended problem, and can
confidently provide feedback for the next iteration.

Communication is part of delivery.

---

## What Success Looks Like

Before considering a feature complete, I can answer "yes" to all of the following:

- Existing functionality still works.
- Different use cases have been tested.
- The implementation has been refactored where appropriate.
- The code is easy to understand.
- The feature matches the original requirement.
- The changes have been committed.
- A build is available.
- The client has a clear demonstration of the work.