# Development Workflow

## Purpose

This document defines the practical coordination and delivery habits that keep engineering work visible, incremental, and dependable. It applies after the problem direction is sufficiently clear. [Engineering Playbook](EngineeringPlaybook.md) defines the integrated engineering cycle, and [Problem Solving](ProblemSolving.md) covers uncertainty before implementation.

The workflow is not a complete decision framework or a substitute for engineering judgment. Its role is to make dependencies, progress, feedback, and delivery expectations clear while work proceeds.

## Make Dependencies Visible

Identify information, assets, decisions, access, and external responses that can affect progress. Communicate blockers and uncertainty early, including what is known, what is needed, and what work can continue safely. Unexpected blockers should not become surprises at delivery time.

## Work in Coherent Increments

Prefer increments that can be understood, verified, reviewed, and corrected without requiring the entire project to be complete. Keep related changes together and separate unrelated concerns. An increment may span several files when one decision requires them; artificial fragmentation does not improve safety.

Use feedback to update direction when evidence changes. This does not mean expanding the task continuously: changes in direction should be made explicit, evaluated against scope, and incorporated deliberately.

## Provide Useful Visibility

Communicate progress in terms that help others make decisions: what has been completed, what is in progress, what remains, which risks or dependencies exist, and which trade-offs matter. Use demonstrations, screenshots, videos, or written explanations when they provide better evidence than a status statement.

Visibility is not activity reporting. Its purpose is to expose information early enough for feedback, reprioritization, or correction to remain inexpensive.

## Prepare Delivery Deliberately

Before presenting work, ensure that the implementation has completed [Self Review](SelfReview.md) and that the relevant verification evidence is available. Describe what changed, how it was verified, what remains uncertain, and any next decision that requires feedback.

Delivery should make the result understandable and testable by the intended audience. It does not imply automatic commit or push; repository stages remain governed by explicit approval.

## Long-Term Standard

A strong workflow keeps work small enough to understand, dependencies visible enough to manage, and feedback early enough to improve the result. Coordination should support engineering judgment, not replace it.
