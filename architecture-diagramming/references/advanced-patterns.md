# C4 advanced patterns

Guidance for the situations where the four levels aren't enough on their own: microservices, message-driven architectures, groupings/layers, and scaling to large systems. The through-line: **use grouping boxes over the fixed C4 abstractions rather than inventing new abstraction levels.**

## Groupings vs. new abstractions

People often ask to add levels (subsystems, DDD bounded contexts, architectural layers, libraries). Resist that. The power of C4 is its *fixed* small set of abstractions. Instead, overlay these concepts as **groupings** (a nested `subgraph`, styled distinctly and named in the key) around existing elements:

- **Architectural layers** (UI / business logic / data) → a `subgraph` grouping the components within a container. The layers don't replace components; they organise them.
- **Modules/libraries** (.jar, .dll, Maven/Gradle modules) → a `subgraph` grouping the components sourced from each module.
- **DDD bounded context** → a grouping of systems, containers, or components depending on how it maps to the code/org.
- **Organisational department** → a grouping of software systems (as in a system landscape diagram).

This keeps the semantics intact while telling the richer story. If you *genuinely* need a new abstraction level, treat it as an advanced maneuver: you must define it precisely, or you slide back into the ambiguous ad-hoc diagrams C4 exists to fix.

## Microservices — three stages

The right modeling depends on **ownership**, not on the word "microservice".

**Stage 1 — Monolith.** One web app + one database schema = two containers inside one system boundary. Straightforward.

**Stage 2 — Microservices, one team, one repo, one system.** Each service is typically an **API container + its own database-schema container** — i.e. a **group of containers**, not a single container. All of them live inside the one system subgraph. Wrap each service's container pair in a nested `subgraph` (styled as a grouping, not a system boundary) so the service is visually obvious:

```
flowchart TB
    subgraph xyz["XYZ [Software System]"]
        ui["Web App<br/><i>[Container: …]</i><br/>…"]
        subgraph svcA["Service A [microservice]"]
            apiA["Service A API<br/><i>[Container: Java]</i><br/>…"]
            dbA[("Service A DB<br/><i>[Container: MySQL schema]</i><br/>…")]
        end
        subgraph svcB["Service B [microservice]"]
            apiB["Service B API<br/><i>[Container: Java]</i><br/>…"]
            dbB[("Service B DB<br/><i>[Container: MySQL schema]</i><br/>…")]
        end
    end
```

Why not model a microservice as one container labeled "Java and MySQL"? Because Java and a MySQL schema can't run in the same OS process — the "same process?" heuristic fails, so it's two containers.

**Stage 3 — Split into teams.** When separate teams own the services in separate repos, **promote each service from a group-of-containers to a software system in its own right.** Now:

- Each team maintains **its own** system context + container diagrams.
- From team XYZ's viewpoint, services A/B/C are **external software systems** — opaque boxes. XYZ's context diagram shows them outside its boundary.
- Team A's own container diagram shows A's internals (API + schema).

**Don't draw a single container diagram spanning multiple teams' systems** (all their internals on one canvas). It looks fine but encodes coupling: you're asserting knowledge of *how* other teams' systems work internally, and it breaks the moment they refactor. Treat other teams' systems as opaque.

## Message-driven architectures

The common mistake is a hub-and-spoke diagram with a central **"Message Bus"** that everything "sends messages to". A C4 container is an application or a data store — a message bus is neither, and the hub obscures the real coupling.

**Model each queue/topic as a data-store container** — a subroutine-shaped node (`[[ … ]]`). This exposes the true producer→consumer relationships:

```
flowchart TB
    subgraph xyz["XYZ [Software System]"]
        a["Service A<br/><i>[Container: …]</i><br/>Produces events."]
        c["Service C<br/><i>[Container: …]</i><br/>Consumes events."]
        q1[["Queue 1<br/><i>[Container: RabbitMQ]</i><br/>Customer update events."]]
    end
    a -->|"Sends customer update events to<br/>[AMQP]"| q1
    q1 -->|"Subscribes to customer update events from<br/>[AMQP]"| c
```

Notes:
- Make the label specific: "Sends **customer update events** to", not the generic "Sends messages to".
- Modeling queues/topics as containers also decouples them from deployment: at dev time one broker may host all queues; in prod they may be split across clusters. That's a deployment-diagram concern, not a container-diagram one.
- **Simplified variant:** for simple point-to-point, you may omit the queue container and move its name onto the arrow ("Sends X to [via Queue 1]"). Cleaner, but the queue is less evident — a legitimate trade-off, your choice.
- **Pub/sub:** flip arrow directions to show one producer → topic → many subscribers when that's the story.
- If services are separate **software systems**, decide **who owns each queue** (message format + operation). Ownership affects which team's diagrams the queue appears in.

## Feature-based / stream-aligned teams

When teams are organised by feature/capability (Spotify squads, Team Topologies stream-aligned teams) and a capability spans several services, **don't model the business capability as a software system** — that duplicates the underlying services' internals across diagrams. Keep one set of C4 diagrams **per service**; the feature teams modify those shared diagrams as their work touches each service.

## Hardware

C4 doesn't describe hardware internals, but you can include hardware as an integration point. Either model it as a software system, or introduce a lightweight **hardware system** notion (a distinct shape/icon, described in the key). Treat its internals as an opaque abstraction.

## Scaling to large / cluttered diagrams

When there are 70 or 700 elements, not 7:

- **"Not shown for brevity."** Cross-cutting elements (logging, auditing) with edges from everything create clutter. Either omit them with a diagram note ("all components log via a logging component, not shown for brevity"), or draw the element but annotate its users with a symbol described in the key instead of drawing every arrow.
- **Split the diagram.** Prefer several small, focused diagrams over one giant one: one context diagram per business capability; one container/component diagram per feature/slice/use-case. You lose the single "big picture" but each diagram is comprehensible. (Trivial with a modeling tool; tedious by hand.)
- **Perspectives.** To add ownership/security/tech-debt info without clutter, think of it as a *layer* over an existing diagram (colour-coding + a key, or tooltips) rather than a new diagram. Colour-code and describe it in the key.
- **Simplify the design.** If the diagram is an unmanageable tangle, the *architecture* may be the problem. The diagram is a feedback loop.
- **Alternative visualisations.** Boxes-and-arrows isn't the only option; a model can also be rendered as a force-directed graph or a tree for exploration. (Beyond Mermaid's scope, but worth suggesting for genuinely large models.)

## The model-code gap (why this matters)

We reason about software as components and layers, but languages have no "component" or "layer" keyword — those abstractions live in naming/packaging conventions that teams apply inconsistently. That gap is exactly why getting the **abstractions** right (Step 1 in SKILL.md) matters more than the notation. When you diagram from a written description or a codebase, name the components after the real groupings of code (packages/modules), so the map matches the territory.

## Diagramming vs. modeling (and why it scales)

Most teams *diagram*: they draw independent pictures in a general-purpose tool (Visio, draw.io, or hand-authored Mermaid like this skill produces). The tool only knows "shapes", so it can't validate anything, can't answer "show all dependencies of X", can't auto-build a key, and — worst at scale — doesn't know that a box on the context diagram and a box on the container diagram are the *same element*. Rename or recolour one and you must manually fix every copy.

*Modeling* fixes this: you define one non-visual **model** — a directed graph of elements (nodes) and relationships (edges), each defined once — and render many **views** (diagrams) as subsets of it. Renaming is trivial, consistency is free, and the same model can be rendered as boxes-and-arrows, a force-directed graph, a tree, or queried like a graph database. Structurizr (and its text DSL) is the canonical C4 modeling tool.

**What this means for this skill:** you produce Mermaid diagrams (the diagramming approach), so *you* are the one keeping elements consistent across a set — do it deliberately (same alias, name, tech, description everywhere). For a large or long-lived system where the user will maintain many synced diagrams, it's worth *recommending* a modeling tool (Structurizr, IcePanel, Likec4) rather than hand-maintained Mermaid — say so.

A rough maturity ladder the book uses, useful for meeting a team where they are:
1. **Initial** — no diagrams. 2. **Ad hoc** — ambiguous boxes-and-arrows.
3. **Defined** — C4 abstractions + notation guidelines (add text, use a key); tooling irrelevant. 4. **Modeled** — a hand-crafted model with generated views.
5. **Optimizing** — models partly generated from code/infra, org-wide landscapes auto-assembled, models queried as data. Adopting C4 with the guidance in this skill gets a team to level 3; recommend modeling tools to go further.

## C4 + AI

A structured model in a **textual** format (a DSL) gives an AI agent far more to work with than a static image or Visio file, which lack the semantics and often disagree across diagrams. With a textual model you can ask an agent to: summarize the architecture; detect **architecture drift** (compare the modeled/desired architecture against the actual code); **generate a model** from source code or infrastructure; or **generate code within a defined container/component boundary**. When a user wants AI-assisted architecture work at scale, steer them toward a textual model rather than image-based diagrams.
