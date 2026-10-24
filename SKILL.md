---
name: clean-architecture
description: Use when making structural decisions - where to draw a module, package or layer boundary, which way a dependency should point, whether a framework or database belongs in the core, how to make hard-to-test code testable, or when reviewing an architecture that is expensive to change. Encodes the Dependency Rule, policy-versus-detail separation, SOLID as architecture rather than class hygiene, component cohesion and coupling metrics, and the humble object pattern.
---

# Clean Architecture

Structural decisions are expensive to reverse. This skill gives the rules for where boundaries go, which direction dependencies point, and how to keep the core of a system independent of the things around it.

Derived from Robert C. Martin's *Clean Architecture: A Craftsman's Guide to Software Structure and Design* (2017). Chapter references are given as (Ch N) so you can go back to the source.

## The one rule

**Source code dependencies point inward, toward higher-level policy. Nothing inner may name anything outer.** (Ch 22)

Concretely, in the innermost layers:

- No import of a database driver, ORM, HTTP framework, serialization format, or vendor SDK.
- No type from the web layer, the persistence layer, or the UI appears in a signature.
- No SQL string, no route path, no HTML, no JSON schema.

Data may cross a boundary in both directions. Types must not leak outward-in. When control must flow outward, invert it: the inner layer declares the interface it needs, and an outer layer implements it. (Ch 11, Ch 22)

Test: open the file containing your core business rules. If you can name the transport, the storage engine, or the framework from what is written there, the rule is broken.

## Separate policy from detail

Software has two kinds of value: the value of its behavior, and the value of its structure. Behavior pays today; structure is what lets behavior keep changing. Structure is the greater value, because it is the reason software can be changed at all. (Ch 2)

- **Policy** is the business rules and procedures. That is where the value lives.
- **Detail** is everything needed to let humans and machines talk to the policy: databases, web servers, frameworks, protocols, IO, dependency injection. Detail does not change what the policy decides. (Ch 15)

The job is to shape the system so detail becomes irrelevant to policy, which lets you defer detail decisions.

**A good architect maximizes the number of decisions not made.** (Ch 15)

Practical consequences:

- Do not choose a database, web server, or delivery mechanism in the early days. If the policy does not know it is being delivered over the web, you have not had to decide.
- If a decision was already forced on you by your organization, shape the code as if it had not been. Keep the ability to reverse it.
- The longer a detail decision is deferred, the more information you have when you finally make it. Deferring also buys you room to run experiments against several options.

### How to tell whether structure is good

Do not judge architecture by elegance. Judge it by **the effort required to satisfy the next request, tracked release over release**. (Ch 1)

- Cost per change flat across releases: the structure is working.
- Cost per change climbing every release, while headcount climbs to compensate: the structure is failing, and the headcount curve hides it.
- Haste now, clean up later: the cleanup is never funded, because the market pressure that caused the haste does not let up.

The cost that matters is cost proportional to the *shape* of a request — how awkwardly it fits what exists — not merely its scope. (Ch 2)

## Where to draw a boundary

**Draw a boundary where there is an axis of change** — where the two sides change at different rates, or at different times, or for different reasons. (Ch 17)

- The UI changes on a different schedule than business rules, so a boundary belongs between them.
- The database schema changes for reasons unrelated to the rules, so a boundary belongs between them.
- Two use cases that will be requested by different people at different times deserve separation even if they look alike today.

This is the Single Responsibility Principle applied at architectural scale, and it is what tells you where to put the line. (Ch 7, Ch 17)

**Level means distance from inputs and outputs.** (Ch 19) The further a policy sits from both, the higher its level. Input/output handling is the lowest level. Dependencies should point toward higher level — which means source code dependencies deliberately do *not* follow the data flow. Couple to level, not to flow.

Boundaries make the inner side immune to the outer side, in the way a plugin is immune to its host: the plugin knows the host, the host does not know the plugin. Arrange your system so the volatile, replaceable parts are plugins to the stable core. (Ch 17)

## SOLID, as architecture

These are usually taught as class-level hygiene. Here they are structural.

**SRP — one reason to change, expressed as one actor.** A module should answer to a single group of stakeholders. The trap is splitting by "one function does one thing" and missing that two unrelated departments both drive the same class. Two actors, two modules — even when the code looks identical, because it will diverge. Symptoms of violation: accidental duplication (two copies of a rule that must change together) and merge conflicts (two teams editing the same file for unrelated reasons). (Ch 7)

**OCP — extend by adding, not editing.** You get this through *directional control*: the component that must be stable owns the interface, the volatile component implements it. Combine with information hiding so the stable side does not depend on internal detail of the volatile side. New behavior should arrive as new code plugged into an existing seam. (Ch 8)

**LSP — substitutability is a contract, and architecture can violate it.** A subtype must honor everything a caller is entitled to assume. Watch for: implementers that throw on methods they cannot support; subclasses that quietly weaken a postcondition; and violations introduced at the *architecture* level by choosing a plugin whose capabilities do not satisfy the contract the core expects. A classic failure is modeling a special case as a subtype of the general case when the special case cannot honor all behavior. (Ch 9)

**ISP — do not force a client to depend on what it does not use.** Split interfaces per client, so that changes to methods a client never calls cannot force it to recompile or redeploy. This matters more across boundaries than within a class hierarchy. (Ch 10)

**DIP — depend on abstractions, and own the abstraction.** The stable, high-level component declares the interface it needs; the low-level, volatile component implements it. Never let the core import a concrete detail in order to call out. (Ch 11)

Achieving DIP in practice:

- Put the interface in the component that *uses* it, not the one that implements it.
- An interface-only component (an abstract component) is a legitimate, and in statically typed languages often necessary, tactic. It is maximally stable, so it is a safe thing for volatile components to depend on.
- In dynamically typed languages you often get inversion for free, without declaring the interface.
- A factory belongs in the stable side: it shields the core from knowing which concrete implementation was chosen.

## Components: what belongs together

Three cohesion principles, permanently in tension. You cannot maximize all three; you choose a position on the triangle. (Ch 13)

- **Reuse/Release Equivalence** — things released together, and versioned together, belong in one component. Classes in a component should form a coherent, releasable whole. If there is no release process, you have no reuse.
- **Common Closure** — things that change together, for the same reasons, belong in one component. Optimize for the cost of redeploying and revalidating rather than for the cost of rebuilding. This is SRP applied to components.
- **Common Reuse** — things used together belong in one component. Do not force a consumer to pull in a component for one class it needs; unrelated classes in a shared component create needless releases and needless breakage.

The position shifts over the project's life. Early, you want reuse pressure low and the team able to move quickly. As the project matures and gains consumers, closure and reuse pressure rise. Expect to re-partition.

**Do not eliminate duplication reflexively.** Distinguish true duplication — every change must be repeated everywhere — from accidental duplication, where two similar pieces will evolve on different schedules. Two screens that look alike today will diverge; sharing them prematurely fuses two use cases that should have been independent. Likewise, passing a database row straight up to the view because it happens to match saves little and re-couples the layers. (Ch 16)

## Components: which way dependencies point

**Acyclic Dependencies — no cycles.** A cycle means every component in it must be built, tested, and released together; the whole group becomes one unit and the "morning after" problem arrives. Break a cycle by applying DIP, by extracting the shared piece into a new component, or by restructuring to a top-down layering. (Ch 14)

**Stable Dependencies — depend in the direction of stability.** Instability is measured per component:

```
I = Ce / (Ca + Ce)
```

where `Ca` is classes outside that depend on this component (fan-in, afferent) and `Ce` is classes here that depend on others (fan-out, efferent). `I = 0` means maximally stable and hard to change; `I = 1` means maximally unstable. Dependencies must flow toward decreasing `I`: nothing stable should depend on something volatile. (Ch 14)

Not everything should be stable. A fully stable system cannot change, so keep the volatile, frequently edited parts unstable and at the edge.

**Stable Abstractions — a component should be as abstract as it is stable.** Stability means difficulty of change; abstraction means freedom to extend without change. Keep them in balance:

```
A = Na / Nc        # Na = abstract classes + interfaces, Nc = total classes
D = |A + I - 1|    # distance from the Main Sequence, range [0,1]
```

The Main Sequence runs from `(I=1, A=0)` (maximally unstable and concrete) to `(I=0, A=1)` (maximally stable and abstract). Target `D ≈ 0`. (Ch 14)

Two exclusion zones:

- **Zone of Pain — near `(0,0)`**: stable and concrete. Cannot be extended, and cannot easily be changed. A database schema is the archetype; a concrete utility library that everyone depends on does the same.
- **Zone of Uselessness — near `(1,1)`**: abstract with no dependents. Unimplemented abstractions and speculative interfaces nobody calls.

Combined, SDP plus SAP amount to the DIP at component scale: dependencies run toward stability, stability implies abstraction, therefore dependencies run toward abstraction.

## The shape that follows

Four concentric concerns, innermost first: **entities** (enterprise-wide rules) → **use cases** (application-specific rules) → **interface adapters** (controllers, presenters, gateways, serializers) → **frameworks and drivers** (web, database, devices). (Ch 20, Ch 22)

- **Entities** hold rules that would survive a change of application. They are the least likely to change, so they are the most reused.
- **Use cases** orchestrate entities to accomplish one thing the system is hired to do. They define when and how entities are used.
- **Use cases take a request model in and return a response model out.** Do not pass entities across the boundary, and do not let the response model embed web or database shapes. This is what keeps the use case ignorant of who called it.
- **Interface adapters** translate between the use case's models and whatever the outside world speaks: HTTP, SQL, XML, JSON, a view model.

When crossing a boundary, data structures may cross freely; dependencies always point inward. Use a plain structure, not a live entity, and let the outer side convert.

**Derive the structure from actors, then use cases, then components.** (Ch 33) Identify who can demand a change (in a video-sales system: viewers, purchasers, authors, administrators), give each its own view, presenter, interactor, and controller components, and keep each one separately deployable. When two use cases are near-duplicates for different actors, unify them under an abstract use case rather than copying them — but only when they truly change together.

**Main is the dirtiest component, and that is correct.** (Ch 26) The entry point is the lowest-level policy in the system; only the operating system depends on it. Do all dependency injection, factory wiring, configuration loading, and framework setup there, then hand control to the abstract core. Keep the DI container out of the inner layers. Where convenient, produce one Main per configuration (dev, test, production) so configuration is a build/deploy choice rather than branches in code.

**Architecture should expose the intent of the system.** If it is a shopping cart application, the top level should say so, with use-case names as first-class visible elements — not a directory tree that says "controllers, models, views". A developer should not have to hunt for behavior. (Ch 21)

**Frameworks are tools, not ways of life.** (Ch 21, Ch 32) You marry a framework and it courts other suitors: its authors do not have your interests in mind, and the commitment is asymmetric. Keep the framework at the outermost ring. Do not derive your entities from framework base classes, and do not let framework annotations become the only place your design is expressed. If the framework must be adopted at a particular moment, take it in small pieces and keep a layer between it and your core.

## Details that are always details

**The database is a detail.** It is a mechanism for storage and retrieval, not part of the architecture. The real damage is not the choice of engine, it is letting the data model — tables, columns, normalization, keys — shape the object model and the rules. Do not design the domain around the schema. Keep a boundary so the schema can change without the rules changing. (Ch 30)

**The web is a detail.** Whether the system is delivered over the web is a delivery decision. The core should not know it is being served over HTTP. Controllers and presenters are adapters. (Ch 31)

**The service boundary is not an architecture.** Splitting into services buys operational independence at best, and only partially: it does not partition the *rules*. A system can be split into many services and still be a tangled mess of coupled business logic. Structure comes from where the rules live and which way dependencies point, not from process boundaries. (Ch 27)

## Testability is an architectural property

**Humble Object pattern.** When a piece of code is hard to test — GUI, IO, a device, a framework callback — split it into two: a humble side that does almost nothing (wires calls through), and a tested side that holds all the logic. The humble part is trivial enough not to need tests. This is not a testing trick, it is a structural decision that makes the system testable. (Ch 23)

Apply it to: presenters and views (view has no logic; presenter formats), database gateways (SQL in the adapter; rules in the core), data mappers, service listeners.

**Tests are the outermost component, and the most coupled.** They are the one part that always depends on everything. Protect against that: design a testing API — a narrow, stable surface the tests use — rather than having tests reach into volatile internals. Keep test data out of production structures, and avoid schemas in tests that fight the domain model. (Ch 28)

**Structure the system so most decisions are testable without the slow parts.** If testing a rule requires a database, a browser, or a network, the boundary is in the wrong place.

## Packaging

How the source tree is organized is part of the architecture, and layer-based packaging is usually the weakest option:

- **Package by layer** — `controllers/`, `services/`, `repositories/`. Fails because every use case is spread across every layer, so nothing is encapsulated and all layers must be touched for one feature.
- **Package by feature** — better, groups by subject area, but still usually lets everything see everything.
- **Ports and adapters** — a boundary per port, adapters outside. Good, but if the core is not enforced as encapsulated it degrades into layers again.
- **Package by component** — one package per coarse-grained component with a single public surface, everything else hidden. Preferred when the language can actually enforce the hiding. (Ch 34)

The deciding question is whether the language can prevent outsiders from reaching in. If encapsulation is only nominal — everything public by default — then the "boundary" is a convention and will erode. Use module systems, internal packages, or build-time enforcement to make it real.

## Costs and when to skip

Boundaries are not free: they cost indirection, extra types, and work at every crossing. Decide deliberately.

- A full boundary needs reciprocal interfaces, boundary-crossing data structures, and dependency inversion on both sides. That is a lot for a small system.
- **Skip the last step**: create the reciprocal interface and the separation, but do not maintain the full isolation on both sides, if the cost is not yet justified.
- **One-dimensional boundaries** (a facade, a narrow interface between two components) buy a lot of the benefit for little cost when you only need to protect one seam.
**Decoupling has three modes, and you choose per seam.** (Ch 16) Source level (separate modules, one process, function calls — cheap and chatty), binary/deployment level (separate deployable units, possibly separate processes), and service level (network only — slow, tens of milliseconds to seconds, so not chatty). It is hard to know which is right early, and the right one changes as the system matures. Service-level-by-default is expensive and pushes you toward coarse boundaries that are never fine enough.

- Prefer to push decoupling to the point where a service *could* be extracted, then leave the code in one address space. Start as a monolith that could become services, not as services that would prefer to be a monolith.
- A system should be able to grow from a monolith to separately deployable units and back. The decoupling mode is itself a deferred decision. (Ch 16)

Do not spend the budget on boundaries nobody is asking for. A boundary inside a single team's fast-moving code, where both sides change together, is usually ceremony.

## Review checklist

Answer these against the code, not the diagram:

1. Can the core business rules compile without the framework, the database driver, and the web library present?
2. Does any inner-layer file name a type, annotation, or string belonging to an outer layer?
3. For each boundary: what actually changes on each side, and at what rate? If the answer is "the same", remove it.
4. Is there a cycle in the component graph?
5. Do the arrows point toward stability, and is the stable end also abstract?
6. Can a use case be tested end to end with fakes, no IO, in milliseconds?
7. Does the top-level structure announce what the system does, or only what frameworks it uses?
8. Does the domain model follow the schema, or does the schema follow the domain?
9. Is everything public, so that "packages" hide nothing?
10. Which decisions have you been forced into that you could still defer?

## Smells

- Business rules reference SQL, HTTP, or framework types.
- Entities or domain objects used directly as request/response payloads or view models.
- Controllers or views containing branching business logic.
- A single shared "utils" or "common" component everyone depends on and anyone can edit.
- Tests that require a live database, browser, or network.
- A layer whose only job is to pass calls through.
- Component graph with a cycle; or a stable component depending on a volatile one.
- Interfaces that exist for every class (symmetrical) rather than interfaces owned by the user side.
- The dependency structure mirrors the data flow rather than the level.
- Directory layout expresses layers only; no use case is visible anywhere in the tree.
- Layers skipping their neighbours — a controller reaching straight for the repository, bypassing the domain.
- One broad "infrastructure" tree that lets any adapter call any other without passing through the core.
- Every type is public, so all four packaging styles compile to the same dependency graph and differ only on paper.

## When this skill does not apply

- Throwaway scripts, spikes, and one-off analysis. Structure a script like a script.
- Code where the boundary's two sides are owned by one person and change together, always.
- Very small systems where the cost of every boundary exceeds the expected change.

Keep the reasoning even where you skip the structure: knowing which decisions you are deferring, and which you have made by default, is the point.
