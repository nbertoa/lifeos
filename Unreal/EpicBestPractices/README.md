# Epic Games Unreal Engine Best Practices

## Purpose

This directory is a living, source-backed handbook of Unreal Engine development practices recommended, demonstrated, or
strongly implied by Epic Games.

It consolidates guidance that is otherwise distributed across:

- Unreal Engine documentation.
- Epic's C++ coding standard.
- Official sample projects.
- Engine source code.
- Unreal Fest presentations.
- Epic Developer Community learning material.
- Documentation for individual engine systems.
- Profiling, testing, deployment, and platform guides.

The handbook is intended to answer three questions:

1. What does Epic recommend?
2. Why does Epic recommend it?
3. Under which conditions should the recommendation be applied?

This collection documents Epic's guidance separately from personal engineering preferences and project-specific rules.

---

## Scope

The target is broad coverage of production Unreal Engine development, including:

- Project architecture.
- C++.
- Blueprints.
- Gameplay Framework.
- Gameplay Ability System.
- Gameplay Tags.
- Components.
- Subsystems.
- Modules.
- Plugins.
- Game Feature Plugins.
- Asset management.
- Content organization.
- Data-driven design.
- Input.
- User interfaces.
- Animation.
- Artificial intelligence.
- Networking.
- Replication.
- Physics.
- Rendering.
- Materials.
- Lighting.
- Niagara.
- Audio.
- World building.
- World Partition.
- Save systems.
- Testing.
- Debugging.
- Profiling.
- Performance.
- Memory.
- Packaging.
- Build automation.
- Platform-specific development.
- Source control.
- Team workflows.
- Official sample-project architecture.

No single recommendation is universally correct for every Unreal project.

Practices must be interpreted according to:

- Engine version.
- Target platform.
- Team size.
- Project scale.
- Game genre.
- Performance target.
- Networking requirements.
- Content-production workflow.
- Prototype or production status.

---

## Source Policy

Every substantive recommendation should be traceable to an authoritative source.

### Primary Sources

Preferred sources, in order:

1. Current Unreal Engine documentation.
2. Epic's official coding standards and technical guidelines.
3. Unreal Engine source code and source comments.
4. Official Epic sample projects.
5. Official Unreal Fest and Epic technical presentations.
6. Epic Developer Community tutorials created by Epic staff.

### Secondary Evidence

The following may be used to clarify or investigate a topic, but should not be presented as an Epic recommendation
without primary-source support:

- Community tutorials.
- Forum answers.
- Blog posts by third-party developers.
- Marketplace documentation.
- Unofficial sample projects.
- Personal experience.

### Source Classification

Every documented practice should use one of these labels:

- **Official requirement**: Required by Unreal Build Tool, Unreal Header Tool, the engine, or a supported platform.
- **Official recommendation**: Explicitly recommended by Epic documentation or presentations.
- **Epic pattern**: Repeatedly demonstrated in engine code or official sample projects.
- **Context-dependent guidance**: Recommended only for particular platforms, scales, or use cases.
- **Experimental guidance**: Associated with an experimental or beta feature.
- **Inference**: A conclusion derived from several Epic sources but not stated directly by Epic.

Inferences must be clearly labeled.

---

## Version Policy

Unreal Engine changes continuously.

Every document should record:

- The engine version or documentation version researched.
- The research date.
- Whether the feature is Production-Ready, Beta, Experimental, or Deprecated.
- Known differences between major engine versions.
- Whether the recommendation is universal or version-dependent.

Documentation for deprecated systems must not be presented as current guidance.

A recommendation should be revalidated when:

- The project upgrades Unreal Engine.
- Epic changes the feature's maturity status.
- A replacement system becomes production-ready.
- The official sample architecture changes.
- Platform requirements change.

---

## Recommendation Format

Each best practice should be documented using the following structure.

### Recommendation

A concise statement of the practice.

### Classification

One of the source classifications defined in this document.

### Why

The engineering reason behind the recommendation.

### Apply When

The conditions under which the recommendation is useful.

### Avoid or Reconsider When

Cases where blindly following the recommendation could create unnecessary complexity or conflict with project
requirements.

### Implementation Guidance

Practical Unreal-specific guidance.

### Validation

How to verify that the practice is working correctly.

### Sources

The authoritative Epic material supporting the recommendation.

---

## Core Principles Already Supported by Epic Guidance

The initial research reveals several recurring principles across Epic documentation.

### Combine C++ and Blueprints Deliberately

Epic supports projects built entirely with either tool, but states that most projects benefit from combining C++ and
Blueprints.

A common division is:

- C++ defines stable foundational systems and APIs.
- Blueprints extend, configure, and assemble those systems.
- Designers receive intentional extension points.
- Core invariants remain protected.

The choice should consider maintainability, iteration speed, debugging, performance, diffing, merging, and team
responsibilities.

---

### Follow Epic's C++ Coding Standard

Epic's coding standard is not only cosmetic.

It supports:

- Consistency across a large codebase.
- Easier code review.
- Clear interaction with Unreal Header Tool.
- Predictable naming.
- Reduced ambiguity.
- Portability across supported compilers and platforms.

Project code should normally follow Epic's standard unless the project has an explicitly documented alternative.

---

### Use Gameplay Framework Classes According to Their Responsibilities

Gameplay Framework provides specialized classes rather than one universal gameplay object.

Typical responsibilities include:

- `GameMode`: server-authoritative game rules.
- `GameState`: replicated state relevant to the game.
- `PlayerController`: player control and player-specific coordination.
- `PlayerState`: replicated player information that can outlive a Pawn.
- `Pawn`: a controllable or possessable game-world representation.
- `Character`: a Pawn specialized for humanoid-style movement.
- `HUD` and UI systems: player-facing presentation.
- `GameInstance`: state and services whose lifetime spans map changes.

Systems should be placed according to lifetime, authority, ownership, replication, and responsibility.

---

### Treat the Server as Authoritative in Networked Games

Epic's networking model considers the server's game state authoritative.

Clients should generally:

- Request actions rather than establish authoritative results.
- Receive replicated state from the server.
- predict only where supported and appropriate;
- avoid trusting client-provided gameplay outcomes.
- minimize unnecessary replicated data and remote calls.

Replication must be designed as part of gameplay architecture, not added mechanically after implementation.

---

### Replicate Only What Is Required

Replication has CPU, memory, and bandwidth costs.

A networked project should deliberately decide:

- Which Actors replicate.
- Which properties replicate.
- Which events require RPCs.
- Who owns an Actor.
- Which connections need each Actor.
- How frequently state must update.
- Whether state or an event is the correct representation.

Large-scale projects may require relevance systems such as Replication Graph or newer replication technology, but those
systems should be selected according to project requirements and feature maturity.

---

### Profile Before Optimizing

Epic's performance guidance consistently emphasizes measurement.

Optimization should begin by identifying:

- The target frame budget.
- The constrained hardware.
- Whether the bottleneck is on the game thread, render thread, GPU, network, memory, loading, or another system.
- A reproducible test scenario.
- The cost before and after the change.

Tools such as Unreal Insights and specialized profilers should guide optimization.

Visual complexity alone is not reliable evidence of runtime cost.

---

### Design Around Platform Budgets

A feature that performs well on one platform may be unacceptable on another.

Teams should define budgets for:

- Frame time.
- Draw calls.
- Triangles.
- Material complexity.
- Texture memory.
- Streaming.
- Skeletal meshes.
- Animation.
- Niagara.
- Audio.
- Network bandwidth.
- Loading time.
- Package size.

Budgets should be tested on representative target hardware throughout development, not only near release.

---

### Use Automated Tests at Appropriate Levels

Unreal provides different testing systems for different purposes.

Testing may include:

- Low-level tests.
- Automation tests.
- Automation Specs.
- Functional Tests.
- Content validation.
- User-flow simulation.
- Multiplayer or multi-process sessions.
- Platform test sessions.
- Build and deployment validation.

The testing strategy should match the behavior under test instead of forcing every test into one framework.

---

### Prefer Data-Driven Configuration Where Appropriate

Unreal supports data-driven workflows through:

- Data Assets.
- Primary Data Assets.
- Data Tables.
- Curves.
- Configuration files.
- Gameplay Tags.
- Gameplay Effects.
- Asset Manager rules.
- Blueprint defaults.
- Developer Settings.

Frequently changed content and tuning values should generally be configurable without recompiling foundational C++.

Data-driven design should not remove validation, ownership, or clear schemas.

---

### Manage Asset Loading Deliberately

Hard references can cause assets and their dependency chains to load together.

Projects should understand:

- Hard references.
- Soft references.
- Primary Assets.
- Secondary Assets.
- Asset Bundles.
- Asset Manager rules.
- Cooking rules.
- Asynchronous loading.
- Dependency chains.

Asset-loading architecture affects memory, startup time, level transitions, patching, cooking, and package size.

---

### Separate Presentation From Gameplay State

Gameplay state and visual presentation should not become inseparable.

Examples include:

- Gameplay Effects for gameplay modifications.
- Gameplay Cues for presentation associated with GAS state.
- Animation systems consuming gameplay state rather than authoritatively defining it.
- UI observing state instead of owning core gameplay rules.
- Cosmetic network events remaining separate from authoritative outcomes.

This separation improves testing, networking, reuse, and iteration.

---

### Use Tags as Structured Identifiers

Gameplay Tags provide hierarchical identifiers that can categorize, query, match, and filter gameplay concepts.

They are useful for:

- Abilities.
- States.
- Inputs.
- Events.
- Damage categories.
- Equipment categories.
- Cooldowns.
- Blocking relationships.
- Team or faction concepts.
- Asset classification.

Tags should follow a governed taxonomy.

Using tags for everything without naming rules can replace code coupling with uncontrolled semantic coupling.

---

### Keep Blueprints Maintainable

Blueprints should be treated as production code when they contain production logic.

Practices include:

- Use meaningful names.
- Keep graphs readable.
- Prefer functions for reusable behavior.
- Use macros only when their execution semantics are appropriate.
- Avoid unnecessary Tick logic.
- Avoid repeated expensive searches and casts.
- Use local variables and clear data flow.
- Break large responsibilities into suitable classes or components.
- expose intentional APIs from C++;
- compile and validate Blueprint assets regularly.

Visual scripting does not eliminate the need for architecture.

---

### Make Ownership and Lifetime Explicit

Many Unreal architecture problems originate from storing a system in an object with the wrong lifetime or ownership.

Before selecting a host, determine:

- Who owns the data.
- Who may modify it.
- Whether it exists on clients, the server, or both.
- Whether it survives Pawn destruction.
- Whether it survives level transitions.
- Whether it belongs to one world, one player, one local machine, or the complete application.
- Whether it needs replication.
- Whether it needs editor support.

Actors, Actor Components, Subsystems, Game Framework classes, UObject services, modules, and plugins are not
interchangeable.

---

### Validate Content Before Shipping

Projects should detect invalid content before it reaches runtime or a release build.

Validation can cover:

- Required references.
- Naming.
- Asset metadata.
- Gameplay Tag usage.
- Unsupported platform content.
- Missing collision.
- Invalid data combinations.
- Packaging rules.
- Primary Asset configuration.
- Blueprint compilation.
- Map errors.

Validation should be automated where practical and included in continuous integration.

---

## Planned Documents

### Foundations

- `Architecture.md`
- `ProjectSetup.md`
- `GameplayFramework.md`
- `ObjectsActorsAndComponents.md`
- `Subsystems.md`
- `DataDrivenDesign.md`
- `LifecycleAndOwnership.md`

### Programming

- `CppCodingStandard.md`
- `CppArchitecture.md`
- `ReflectionAndUObjects.md`
- `MemoryManagement.md`
- `DelegatesEventsAndInterfaces.md`
- `Modules.md`
- `Plugins.md`
- `GameFeaturePlugins.md`
- `BuildSystem.md`

### Blueprints

- `BlueprintBestPractices.md`
- `BlueprintCppBoundaries.md`
- `BlueprintPerformance.md`
- `BlueprintAPIDesign.md`

### Gameplay

- `GameplayAbilitySystem.md`
- `GameplayTags.md`
- `EnhancedInput.md`
- `Movement.md`
- `SavingAndPersistence.md`
- `ArtificialIntelligence.md`

### Networking

- `NetworkingArchitecture.md`
- `Replication.md`
- `RPCs.md`
- `NetworkOwnershipAndRelevancy.md`
- `ReplicationGraph.md`
- `Iris.md`
- `NetworkProfiling.md`

### Content

- `ContentOrganization.md`
- `NamingConventions.md`
- `AssetManagement.md`
- `AssetLoading.md`
- `DataAssetsAndDataTables.md`
- `SourceControl.md`

### User Interface

- `UMGArchitecture.md`
- `UMGPerformance.md`
- `CommonUI.md`
- `Slate.md`
- `Accessibility.md`

### Animation

- `AnimationArchitecture.md`
- `AnimationBlueprints.md`
- `AnimationPerformance.md`
- `MotionMatching.md`
- `ControlRig.md`

### Visuals and Audio

- `Rendering.md`
- `Materials.md`
- `Lighting.md`
- `Nanite.md`
- `Lumen.md`
- `Niagara.md`
- `Audio.md`

### Worlds and Physics

- `WorldPartition.md`
- `LevelStreaming.md`
- `PhysicsAndChaos.md`
- `Collision.md`
- `Navigation.md`

### Quality

- `Debugging.md`
- `Testing.md`
- `ContentValidation.md`
- `Profiling.md`
- `Performance.md`
- `MemoryOptimization.md`

### Delivery

- `Cooking.md`
- `Packaging.md`
- `BuildAutomation.md`
- `Configuration.md`
- `PlatformGuidelines.md`
- `Mobile.md`
- `XR.md`

### Official Architecture Studies

- `Lyra.md`
- `GameFeaturesInLyra.md`
- `GASInLyra.md`
- `ActionRPG.md`
- `StackOBot.md`
- `CitySample.md`
- `EngineSourcePatterns.md`

---

## Research Order

The collection should be developed in the following order:

1. Architecture and object responsibilities.
2. C++ and Blueprint development.
3. Assets, loading, and content organization.
4. Gameplay systems.
5. Networking.
6. UI, animation, AI, audio, and visual systems.
7. Testing, debugging, profiling, and performance.
8. Packaging, build automation, and platforms.
9. Official sample-project studies.
10. Cross-topic checklists and production audits.

This order establishes foundational concepts before specialized systems.

---

## Planned Cross-Topic Audits

After the topic documents are complete, the collection should provide practical audits.

### New Project Audit

Questions to validate project setup and architecture before production begins.

### Feature Architecture Audit

Questions to review a new gameplay or engine feature.

### Blueprint Audit

Questions to detect maintainability, coupling, and performance risks.

### C++ Audit

Questions covering Unreal conventions, ownership, reflection, APIs, and modules.

### Asset Audit

Questions covering references, loading, naming, cooking, and memory.

### Multiplayer Audit

Questions covering authority, ownership, replication, prediction, bandwidth, and security.

### Performance Audit

Questions covering measurement, budgets, profiling, scalability, and representative hardware.

### Release Audit

Questions covering tests, validation, packaging, configuration, platform requirements, and observability.

---

## Definition of Complete

This collection is complete enough to use when:

- Every major Unreal development area has a dedicated document.
- Each recommendation has authoritative evidence.
- Requirements are distinguished from optional practices.
- Version-dependent advice is labeled.
- Experimental and deprecated systems are identified.
- Contradictions between sources are documented.
- Sample-project patterns are not misrepresented as universal rules.
- Each document explains context and tradeoffs.
- Each topic includes implementation and validation guidance.
- Cross-topic production checklists are available.
- The collection has a documented review date.

It should never be treated as permanently finished.

Unreal Engine evolves, and this handbook must evolve with it.