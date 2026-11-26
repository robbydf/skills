---
name: design-alternatives
description: Explore several radically different designs for the same problem in parallel, then compare them and recommend one. Use before committing to an interface, API, module boundary, data schema, architecture, or workflow, or when the user says "design it twice", asks for alternatives or options, or wants to compare approaches.
---

# Design Alternatives

Based on "Design It Twice" (John Ousterhout, _A Philosophy of Software Design_): your first idea is unlikely to be the best. Produce several **radically different** designs for the same problem, each from a fresh context, then compare them and give a strong recommendation.

Works for any design decision: a module or API interface, a data schema, a service architecture, a workflow or process. When the thing being designed is a code interface, also apply [Code-interface mode](#code-interface-mode).

## Process

### 1. Frame the problem space

Before generating any designs, write a short user-facing explanation of the problem:

- The **constraints** any design must satisfy (functional requirements, performance, compatibility, team or operational limits).
- The **dependencies** it relies on and how they behave (in code mode, classify them; see below).
- A rough illustrative sketch (pseudo-code, a schema fragment, a box diagram) that makes the constraints concrete. This is not a proposal, just grounding.

If a fact you need lives in the codebase or docs, look it up; don't ask the user for it. If the project has a glossary or domain docs (`GLOSSARY.md`, `CONTEXT.md`, `CLAUDE.md`, ADRs), use its terms. None of this is required: work from whatever exists.

Show the framing to the user, then immediately proceed to step 2. The user reads and thinks while the designs are being produced.

### 2. Generate the designs in parallel

Spawn **3 or more sub-agents in parallel** (e.g. the Agent tool), each with a fresh context window, each producing one design. Give every agent:

- A **technical brief**, separate from the user-facing framing: relevant file paths, current structure and coupling, constraints, dependencies, and any domain vocabulary to use.
- A **different design constraint**, so the designs genuinely diverge. Pick the ones that fit the problem, for example:
  - "Minimise the surface: as few entry points / tables / steps as possible. Maximise what each one does."
  - "Maximise flexibility: support many use cases and future extension."
  - "Optimise for the most common case: make the default path trivial, even if rare cases get harder."
  - A domain-specific lens: ports & adapters across a dependency (code); normalised vs. denormalised / event-sourced (schemas); synchronous vs. asynchronous / push vs. pull (workflows and services); centralised vs. distributed ownership (architecture).
- An instruction **not to edit any files**.

Each agent returns:

1. **The design**: the concrete shape (types, methods and params with invariants, ordering and error modes; or tables and relationships; or components and their messages; or the steps and who owns them).
2. **Usage example**: how a caller, user, or operator actually uses it in the common case.
3. **What it hides**: the complexity it absorbs so its users don't have to know it.
4. **Dependency strategy**: how it handles the things it relies on.
5. **Trade-offs**: where it is strong, where it is thin, what it makes hard later.

If two designs come back too similar, re-run one with an explicit "do not use approach X" constraint.

If you can't spawn sub-agents, produce the designs yourself one at a time, deliberately starting each from its constraint rather than from the previous design.

### 3. Present, compare, recommend

Present the designs one after another so the user can absorb each, then compare them in prose (not just a table). Compare on:

- **Simplicity for the user of the design**: how much they must learn per unit of capability they get.
- **Locality of change**: when requirements shift, where does the change land, in one place or scattered?
- **Fit with constraints and existing code**: migration cost, risk, how naturally it fits what's already there.

Then give your own recommendation: which design is strongest and why. If elements from different designs combine well, propose a hybrid. Be opinionated: the user wants a strong read, not a menu.

## Code-interface mode

When designing a code interface, use this vocabulary in the briefs and the comparison, and use the terms exactly.

- **Module**: anything with an interface and an implementation, at any scale (function, class, package, service slice).
- **Interface**: everything a caller must know to use the module correctly: the type signature plus invariants, ordering constraints, error modes, required configuration, and performance characteristics.
- **Depth**: leverage at the interface, the behaviour a caller (or test) gets per unit of interface they must learn. A **deep** module hides a lot behind a small interface; a **shallow** one has an interface nearly as complex as its implementation.
- **Seam**: the place where you can alter behaviour without editing in that place; where the module's interface lives. Where to put the seam is its own decision.
- **Adapter**: a concrete thing that satisfies an interface at a seam.
- **Leverage** (for callers) and **locality** (for maintainers): what depth buys.

Principles to hold each design to:

- **The deletion test.** Imagine deleting the module. If complexity vanishes, it was a pass-through. If it reappears across many callers, it earns its keep.
- **The interface is the test surface.** If tests must reach past the interface, the module is the wrong shape.
- **One adapter means a hypothetical seam; two adapters mean a real one.** Don't add a seam unless something actually varies across it (typically production and test).

Classify each dependency in the brief, because the category decides the seam:

1. **In-process** (pure computation, in-memory state): merge freely and test through the interface; no adapter needed.
2. **Local-substitutable** (has a local stand-in, e.g. an in-memory DB or filesystem): test with the stand-in; the seam stays internal.
3. **Remote but owned** (your own services over a network): define a port at the seam, with a network adapter in production and an in-memory adapter in tests.
4. **True external** (third-party services): inject it as a port and use a mock adapter in tests.

In code mode, compare designs by **depth**, **locality**, and **seam placement**.

---

_Adapted from "Design It Twice" in [mattpocock/skills](https://github.com/mattpocock/skills) (MIT)._
