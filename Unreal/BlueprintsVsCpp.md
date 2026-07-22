# Blueprints and C++

## Purpose

Blueprints and C++ are complementary tools.

Neither should be used exclusively as a matter of principle.

The best choice depends on the stage of the project, the people working on it, the responsibility of the system, and the
cost of maintaining it over time.

The objective is balance.

---

## Prototype Quickly With Blueprints

When I am working alone and need to prototype an idea quickly, I often use Blueprints.

Blueprints make it easy to:

- Explore an idea.
- Test gameplay behavior.
- Adjust values rapidly.
- Visualize execution.
- Iterate without unnecessary setup.

For isolated prototypes, immediate feedback is often more valuable than long-term structure.

---

## Prefer C++ for Collaborative Development

When several programmers are working on the same project, I prefer C++.

Text-based source code is easier to:

- Review.
- Compare.
- Merge.
- Resolve after conflicts.
- Discuss during code review.
- Track through version control.

Blueprints can become difficult to merge and review when several people modify the same assets.

The choice between Blueprints and C++ is therefore not only technical. It also affects collaboration.

---

## AI Changes the Productivity Balance

Historically, Blueprints often provided a significant speed advantage over C++.

AI-assisted development has reduced that difference.

AI makes it faster to:

- Explore Unreal APIs.
- Generate initial C++ implementations.
- Understand engine systems.
- Identify errors.
- Discuss architectural alternatives.
- Write repetitive integration code.

This makes C++ more practical for rapid development than it was previously, especially when the implementation will be
maintained by a team.

AI does not remove the need to understand or review the code.

It reduces the mechanical cost of producing it.

---

## Do Not Rewrite Working Systems Without a Reason

A system should not be rewritten in C++ simply because it was originally created in Blueprint.

If a Blueprint system:

- Works correctly.
- Meets its performance requirements.
- Is understandable.
- Is maintainable.
- Does not create collaboration problems.

Then it can remain in Blueprint.

Rewriting introduces cost and risk.

Migration should be driven by a concrete problem or benefit, not by technical dogma.

---

## Use C++ for Foundational Systems

Some responsibilities naturally belong in C++.

I prefer C++ for:

- Plugins.
- Core project systems.
- Subsystems.
- Reusable frameworks.
- Low-level functionality.
- External service integrations.
- API integrations.
- Systems that require third-party libraries.
- Infrastructure shared by multiple features.

Examples include integrations with services such as Spotify or external artificial intelligence systems.

These systems often require stronger control over initialization, lifecycle, error handling, networking, authentication,
data conversion, and dependencies.

---

## C++ Defines the Foundation

When combining C++ and Blueprints, I use C++ to define the structural foundation of the system.

C++ is responsible for:

- Architecture.
- Core rules.
- Shared behavior.
- Reusable functionality.
- System boundaries.
- Stable interfaces.
- Lifecycle management.
- Protected implementation details.

This creates a reliable base that can be extended without duplicating foundational logic.

---

## Blueprints Configure and Specialize

Blueprints operate on top of the C++ foundation.

Blueprints are responsible for:

- Configuring values.
- Assigning assets.
- Selecting classes.
- Implementing visual behavior.
- Defining project-specific variations.
- Connecting gameplay events.
- Supporting rapid iteration by designers.

C++ should expose appropriate functions, properties, and events so that designers can work without modifying
foundational code.

This keeps systems flexible without sacrificing structure.

---

## Protect Important Invariants

Not every implementation detail should be exposed to Blueprints.

Core rules and invariants should remain protected in C++ when allowing arbitrary modification could make the system
unreliable.

Blueprint exposure should be intentional.

A system should expose what designers need to configure or extend while protecting the assumptions that keep it valid.

---

## Avoid Absolute Rules

Doing everything in Blueprint creates limitations in collaboration, version control, reuse, and access to lower-level
functionality.

Doing everything in C++ can reduce iteration speed and make simple content configuration unnecessarily difficult.

Neither extreme is ideal.

The correct solution is usually a deliberate combination of both tools.

---

## Decision Criteria

When deciding between Blueprints and C++, I consider:

- Whether the work is exploratory or foundational.
- Whether I am working alone or with a team.
- How frequently the behavior will change.
- Whether designers need direct control.
- Whether the system must integrate with external services.
- Whether the implementation needs to be reused.
- Whether merge conflicts are likely.
- Whether Blueprint provides sufficient performance and maintainability.
- Whether migration would solve a real problem.

The choice should follow the needs of the system rather than personal preference.

---

## What Success Looks Like

A successful Unreal architecture:

- Uses Blueprints for fast iteration where appropriate.
- Uses C++ for stable and reusable foundations.
- Supports collaboration through mergeable source code.
- Allows designers to configure and extend systems safely.
- Protects important rules inside C++.
- Avoids unnecessary rewrites.
- Uses each tool according to its strengths.
- Maintains a practical balance between speed and structure.