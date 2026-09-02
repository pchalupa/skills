# C4 in Mermaid flowcharts

Mermaid ships an experimental native C4 mode (`C4Context`, `C4Container`, …), but its auto-layout is unreliable and **many renderers draw it incorrectly or not at all**. This skill therefore renders every C4 level as a **styled `flowchart`**. You give up the semantic C4 keywords, but you gain layout you can control and output that renders everywhere Mermaid runs.

The trade you accept: a flowchart only knows "nodes", "edges", and "subgraphs", so **you** carry the C4 method by hand — write the type into each node, label every edge with a preposition, use subgraphs for boundaries, and follow the diagram with a key table. Everything below is a convention layered on plain flowchart syntax; keep it consistent across a diagram set and the flowchart *reads* as C4.

## Table of contents
1. Elements (people, systems, containers, components)
2. Boundaries
3. Relationships
4. Layout, styling & the key
5. Worked example: System Context
6. Worked example: Container
7. Worked example: Component
8. Worked example: Dynamic
9. Worked example: Deployment
10. System Landscape

---

## 1. Elements

Each element is one flowchart node. Put **name / type / (technology) / description** inside the node text, separated by `<br/>`, with the type line in italics. This is the single biggest thing that makes the diagram good — never ship bare named boxes.

Node text pattern (fill in technology only for containers/components):

```
alias["Name<br/><i>[Type: Technology]</i><br/>Short responsibility."]
```

Pick the **node shape by abstraction**, so shape alone signals the kind:

| Abstraction | Shape | Syntax | Type line to write |
|---|---|---|---|
| Person | stadium | `alias(["…"])` | `[Person]` or `[Person, External]` |
| Software system | rectangle | `alias["…"]` | `[Software System]` / `[Software System, External]` |
| Container (app) | rectangle | `alias["…"]` | `[Container: Java, Spring Boot]` |
| Container / component data store | cylinder | `alias[("…")]` | `[Container: MySQL]` |
| Container / component queue/topic | subroutine | `alias[["…"]]` | `[Container: RabbitMQ]` |
| Component | rectangle | `alias["…"]` | `[Component: Spring MVC]` |

Colour — not shape — carries **internal vs. external** and any other differentiation; assign it with a `classDef` (see §4). Use the cylinder for data stores and the subroutine (double-bar) shape for queues/topics — that is how you honour the rule that **queues are data-store containers**, not a "message bus" system. Write the type line yourself for every node; nothing adds it for you.

Examples:

```
customer(["Personal Banking Customer<br/><i>[Person]</i><br/>A customer of the bank."])
ib["Internet Banking System<br/><i>[Software System]</i><br/>Online banking."]
backend["Backend<br/><i>[Container: Java, Spring Boot]</i><br/>JSON/HTTP API."]
db[("Database<br/><i>[Container: MySQL]</i><br/>Stores credentials.")]
queue[["Event Queue<br/><i>[Container: RabbitMQ]</i><br/>Customer update events."]]
```

## 2. Boundaries

A boundary is a `subgraph`. Give it an id and a quoted label; the label should name the thing you are zooming into.

```
subgraph ib["Internet Banking System"]
    spa["…"]
    backend["…"]
end
```

- On a **container diagram**, wrap your containers in a subgraph named after your system. Keep external people/systems *outside* it.
- On a **component diagram**, wrap your components in a subgraph named after the container you are zooming into.
- Use a nested subgraph to overlay non-C4 **groupings** — microservice boundaries, architectural layers, teams, cloud regions. Distinguish a grouping from a system boundary with a `style` fill or a `[type]` suffix in its label, and describe it in the key.

Subgraphs nest, which is what makes deployment diagrams (§9) work. Add a direction inside a subgraph with `direction LR` when its children should stack differently from the parent.

## 3. Relationships

An edge is a labelled arrow. Keep the label a **sentence ending in a preposition** so the arrow direction is unambiguous, and put the protocol on its own line for inter-process calls.

```
customer -->|"Views balances and makes payments using"| spa
spa -->|"Makes API calls to<br/>[JSON/HTTPS]"| backend
```

- **Solid arrow `-->` = synchronous; dashed arrow `-.->` = asynchronous.** This is the flowchart stand-in for C4's line-style convention — if you use it, say so in the key.
- **Arrows are unidirectional**, from initiator to receiver. Collapse a request/response pair into one arrow. Draw two only when the interactions genuinely differ (sync request + async event).
- **Force direction** with the diagram-level `flowchart TB`/`LR` and by ordering nodes; there is no per-edge `Rel_U/Rel_D`. Nudge stubborn nodes with invisible edges (`a ~~~ b`) or by reordering declarations.
- **No protocol on context-level arrows** — keep those high-level; technology appears one level down on the container diagram.

## 4. Layout, styling & the key

**Direction.** `flowchart TB` (top-to-bottom) or `flowchart LR` (left-to-right) is your main layout lever. People usually read best at the top (`TB`).

**Colour by class.** Define classes once and assign them — this is how you encode internal vs. external, existing vs. new, owned vs. third-party. The palette below matches the usual C4 look (dark blue = your system, grey = external, lighter blue = person).

```
classDef person fill:#08427b,color:#fff,stroke:#052e56;
classDef system fill:#1168bd,color:#fff,stroke:#0b4884;
classDef ext    fill:#999,color:#fff,stroke:#6b6b6b;

class customer person;
class spa,backend,db system;
class core,ses ext;
```

**Key.** A flowchart has no built-in C4 key, so every diagram needs one. Put it in a **Markdown table directly beneath the diagram**, not in a `subgraph` inside it. A legend subgraph competes with the real elements for layout, and the renderer will float it wherever it likes; a table sits where you put it, wraps on narrow screens, and stays readable in a diff.

One row per notation you used to *differentiate* things — every colour, shape, and line style:

```markdown
| Notation | Meaning |
|---|---|
| Dark blue | The system in focus |
| Grey | External system, outside your control |
| Stadium shape | Person or role |
| Cylinder | Data store |
| Subroutine shape | Message queue or topic |
| Dashed edge | Asynchronous communication |
| Dashed border | Out of scope for this release |
```

List only the notation this diagram actually uses. If you colour-code for a release ("new in this version"), add the class and give it a row — every colour gets an explanation.

## 5. Worked example — System Context

Relationships carry **no protocol/technology** here — that is deliberate. On a system context diagram, keep relationships high-level; protocols belong on the container diagram (§6).

```mermaid
flowchart TB
    customer(["Personal Banking Customer<br/><i>[Person]</i><br/>A customer of the bank with personal bank accounts."])
    ib["Internet Banking System<br/><i>[Software System]</i><br/>Lets customers view balances and make payments."]
    core["Core Banking System<br/><i>[Software System, External]</i><br/>Stores accounts and transactions; handles payments."]
    ses["Amazon SES<br/><i>[Software System, External]</i><br/>Sends e-mails to customers (MFA, fraud alerts)."]

    customer -->|"Views balances and makes payments using"| ib
    ib -->|"Gets account data from and makes payments using"| core
    ib -->|"Sends e-mail using"| ses

    classDef person fill:#08427b,color:#fff,stroke:#052e56;
    classDef system fill:#1168bd,color:#fff,stroke:#0b4884;
    classDef ext    fill:#999,color:#fff,stroke:#6b6b6b;
    class customer person;
    class ib system;
    class core,ses ext;
```

| Notation | Meaning |
|---|---|
| Dark blue | The system in focus |
| Grey | External system, outside your control |
| Stadium shape | Person |

*Each diagram below carries its own key in real use. They're omitted from the remaining worked examples to keep the notation in focus.*

## 6. Worked example — Container

Wrap your containers in a subgraph named after the system; keep external people/systems outside it. Protocols now appear on the arrows.

```mermaid
flowchart TB
    customer(["Personal Banking Customer<br/><i>[Person]</i><br/>A customer of the bank."])

    subgraph ib["Internet Banking System"]
        spa["Single-Page App<br/><i>[Container: JavaScript, Angular]</i><br/>Provides Internet banking features in the browser."]
        static["Static Content<br/><i>[Container: Directory]</i><br/>Delivers the SPA's HTML/CSS/JS."]
        backend["Backend<br/><i>[Container: Java, Spring Boot]</i><br/>Provides a JSON/HTTP API to the SPA."]
        db[("Database<br/><i>[Container: MySQL]</i><br/>Stores usernames and hashed credentials.")]
        store[("Statement Store<br/><i>[Container: AWS S3]</i><br/>Caches generated PDF statements.")]
    end

    core["Core Banking System<br/><i>[Software System, External]</i><br/>Stores accounts and transactions."]
    ses["Amazon SES<br/><i>[Software System, External]</i><br/>Sends e-mails to customers."]

    customer -->|"Loads the SPA from<br/>[HTTPS]"| static
    customer -->|"Views balances and makes payments using"| spa
    spa -->|"Makes API calls to<br/>[JSON/HTTPS]"| backend
    backend -->|"Reads from and writes to<br/>[SQL/TCP]"| db
    backend -->|"Caches and reads statements in<br/>[S3 API]"| store
    backend -->|"Makes API calls to<br/>[XML/HTTPS]"| core
    backend -->|"Sends e-mail using<br/>[HTTPS]"| ses

    classDef person fill:#08427b,color:#fff,stroke:#052e56;
    classDef system fill:#1168bd,color:#fff,stroke:#0b4884;
    classDef ext    fill:#999,color:#fff,stroke:#6b6b6b;
    class customer person;
    class spa,static,backend,db,store system;
    class core,ses ext;
```

## 7. Worked example — Component

Scope is a **single container** (here, the backend). Wrap components in a subgraph named after the container; show only the neighbours the container talks to.

```mermaid
flowchart TB
    spa["Single-Page App<br/><i>[Container: JavaScript, Angular]</i><br/>Provides banking features in the browser."]
    db[("Database<br/><i>[Container: MySQL]</i><br/>Stores credentials.")]
    core["Core Banking System<br/><i>[Software System, External]</i><br/>Stores accounts and transactions."]
    ses["Amazon SES<br/><i>[Software System, External]</i><br/>Sends e-mails."]

    subgraph backend["Backend"]
        signin["Sign In API<br/><i>[Component: Spring MVC]</i><br/>Handles sign-in requests."]
        accounts["Accounts Summary API<br/><i>[Component: Spring MVC]</i><br/>Provides a customer's bank accounts."]
        statements["Statement API<br/><i>[Component: Spring MVC]</i><br/>Provides bank statements."]
        security["Security Component<br/><i>[Component: Spring Bean]</i><br/>Validates credentials and issues/validates tokens."]
        email["E-mail Component<br/><i>[Component: Spring Bean]</i><br/>Sends e-mail via SES."]
        coreadapter["Core Banking Adapter<br/><i>[Component: Spring Bean]</i><br/>Calls the Core Banking System."]
    end

    spa -->|"Makes API calls to<br/>[JSON/HTTPS]"| signin
    spa -->|"Makes API calls to<br/>[JSON/HTTPS]"| accounts
    spa -->|"Makes API calls to<br/>[JSON/HTTPS]"| statements
    signin -->|"Uses"| security
    accounts -->|"Uses"| security
    statements -->|"Uses"| security
    security -->|"Reads from and writes to<br/>[SQL/TCP]"| db
    accounts -->|"Uses"| coreadapter
    statements -->|"Uses"| coreadapter
    email -->|"Sends e-mail using<br/>[HTTPS]"| ses
    coreadapter -->|"Makes API calls to<br/>[XML/HTTPS]"| core

    classDef system fill:#1168bd,color:#fff,stroke:#0b4884;
    classDef comp   fill:#85bbf0,color:#000,stroke:#5d82a8;
    classDef ext    fill:#999,color:#fff,stroke:#6b6b6b;
    class spa,db system;
    class signin,accounts,statements,security,email,coreadapter comp;
    class core,ses ext;
```

## 8. Worked example — Dynamic

A flowchart won't auto-number relationships, so **number them yourself** in the edge labels — the sequence tells the story. Show only the subset of elements the feature touches. Use `-.->` for any async step.

```mermaid
flowchart TB
    spa["Single-Page App<br/><i>[Container: JavaScript, Angular]</i><br/>Browser UI."]
    db[("Database<br/><i>[Container: MySQL]</i><br/>Stores credentials.")]

    subgraph backend["Backend"]
        signin["Sign In API<br/><i>[Component: Spring MVC]</i><br/>Handles sign-in requests."]
        security["Security Component<br/><i>[Component: Spring Bean]</i><br/>Validates credentials."]
    end

    spa -->|"1. Submits credentials to<br/>[JSON/HTTPS]"| signin
    signin -->|"2. Validates credentials using"| security
    security -->|"3. Gets user data from<br/>[SQL/TCP]"| db
    db -->|"4. Returns user record to"| security
    security -->|"5. Returns session token to"| signin
    signin -->|"6. Returns session token to"| spa

    classDef system fill:#1168bd,color:#fff,stroke:#0b4884;
    classDef comp   fill:#85bbf0,color:#000,stroke:#5d82a8;
    class spa,db system;
    class signin,security comp;
```

## 9. Worked example — Deployment

Model infrastructure as **nested subgraphs** (one per deployment node), with the **container instances placed inside them**. Draw **one diagram per environment**. Label each subgraph with the node name and its type/technology.

```mermaid
flowchart TB
    subgraph bank["Bank WAN [Corporate network]"]
        subgraph laptop["Developer Laptop [Windows / macOS]"]
            subgraph browser["Web Browser [Chrome/Firefox]"]
                spa["Single-Page App<br/><i>[Container: JavaScript, Angular]</i><br/>Browser UI."]
            end
            subgraph docker["Docker [Docker Engine]"]
                subgraph nginx["nginx [Web server]"]
                    static["Static Content<br/><i>[Container: Directory]</i><br/>Delivers the SPA."]
                end
                subgraph mysqlnode["MySQL [Container]"]
                    db[("Database<br/><i>[Container: MySQL]</i><br/>Stores credentials.")]
                end
            end
            subgraph jvm["JVM [OpenJDK]"]
                backend["Backend<br/><i>[Container: Java, Spring Boot]</i><br/>JSON/HTTP API."]
            end
        end
    end

    spa -->|"Makes API calls to<br/>[JSON/HTTP]"| backend
    spa -->|"Loads from<br/>[HTTP]"| static
    backend -->|"Reads/writes<br/>[SQL/TCP]"| db

    classDef system fill:#1168bd,color:#fff,stroke:#0b4884;
    class spa,static,backend,db system;
```

For production, add subgraphs for the cloud region/services (e.g. AWS Fargate, RDS, S3) and switch protocols to their secure variants (HTTP → HTTPS).

## 10. System Landscape

A landscape is a context diagram without a single system in focus. Show many systems + people, and use a subgraph (styled as a grouping) to mark org/department boundaries.

```mermaid
flowchart TB
    customer(["Personal Banking Customer<br/><i>[Person]</i><br/>A bank customer."])
    staff(["Customer Service Staff<br/><i>[Person]</i><br/>Bank staff."])

    subgraph bank["Big Bank [Enterprise]"]
        ib["Internet Banking System<br/><i>[Software System]</i><br/>Online banking."]
        atm["ATM<br/><i>[Software System]</i><br/>Cash withdrawals."]
        core["Core Banking System<br/><i>[Software System]</i><br/>System of record."]
    end

    ses["Amazon SES<br/><i>[Software System, External]</i><br/>E-mail delivery."]

    customer -->|"Uses"| ib
    customer -->|"Withdraws cash using"| atm
    staff -->|"Uses"| core
    ib -->|"Gets account data from and makes payments using"| core
    atm -->|"Uses"| core
    ib -->|"Sends e-mail using"| ses

    classDef person fill:#08427b,color:#fff,stroke:#052e56;
    classDef system fill:#1168bd,color:#fff,stroke:#0b4884;
    classDef ext    fill:#999,color:#fff,stroke:#6b6b6b;
    class customer,staff person;
    class ib,atm,core system;
    class ses ext;
```
