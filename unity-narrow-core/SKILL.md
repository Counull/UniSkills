---
name: unity-narrow-core
description: Design and review narrow Unity-native core abstractions that are larger than one-file helpers but still intentionally smaller than frameworks or global systems. Use for stable reusable cores such as state machine cores, virtual list cores, or local reversible command cores with explicit ownership, tight boundaries, and minimal host coupling.
---

# Unity Narrow Core

Use this skill when a design no longer fits the "tiny helper" bar, but still should not become a framework.
Prefer a small, explicit, layered core over a flexible system.

This skill sits between:

- tiny helpers such as leases, guards, scopes, and disposables
- large systems such as buses, schedulers, undo frameworks, UI frameworks, or gameplay architectures

If the problem can stay in one file with a tiny API, use `unity-minimal-components` instead.

## When To Use

Use this skill for candidates like:

- independent state machine cores
- Unity UI virtual list cores
- local reversible command cores
- other Unity-native reusable cores with explicit ownership and stable data flow

Do not use this skill for:

- one-file helpers
- project-specific feature modules
- global registries or managers
- editor tooling as the primary deliverable
- systems that already require buses, schedulers, history stacks, or orchestration layers

## Core Definition

A narrow core is a reusable lower-level abstraction that:

- solves one stable problem domain
- may span multiple files
- may have a core layer and a Unity adapter layer
- has explicit ownership and lifecycle
- avoids ambient global state
- is intentionally incomplete compared with a framework

It must still be possible to explain its responsibility in one sentence.

## Workflow

Follow this order:

1. Inspect current code and identify the exact repeated pain point.
2. State the core responsibility in one sentence.
3. State the non-goals before discussing the API.
4. Identify existing abstractions that overlap or compete.
5. Decide the minimum layering needed: pure core only, or core plus Unity adapter.
6. Define the owner, lifetime, and invalid state rules.
7. Cut the first version to the smallest boundary that covers today's repeated cases.
8. Validate with one or two realistic integration examples.

## Admit Or Reject

Admit a narrow core only if all of the following are true:

- the problem is larger than a helper but smaller than a system
- the responsibility remains singular and stable
- there are multiple realistic callers or a clearly recurring architectural pain point
- ownership is explicit and local, not ambient or global
- the first version can name concrete non-goals
- the design can separate core rules from host-framework glue
- the abstraction does not depend on current project vocabulary

Reject or delay when any of the following appears:

- the design needs a registry, manager singleton, or bus to make sense
- the design only looks reusable because current project coupling is being hidden
- most of the API exists for hypothetical future flexibility
- the first version already wants history, retry, concurrency, priorities, or plugins
- the design cannot define what stays outside the core
- the abstraction is really an adapter around one current subsystem rather than a reusable core

## Boundary Enforcement

Treat the reject or delay conditions above as blocking guards, not soft suggestions.

If a request crosses the narrow-core boundary, do not continue into implementation by default.
Instead, explicitly refuse the core in its current shape and redirect to one of these actions:

- narrow the core until its responsibility is singular and stable
- split the problem into a core plus adapter boundary
- keep the result at planning or documentation level until enough real callers exist
- explicitly classify it as a larger framework or system and stop pretending it is a narrow core

When refusing, state which boundary was crossed.
Typical refusal triggers include:

- the first version already needs history, redo, retries, priorities, concurrency, plugins, or editor tooling
- the proposal cannot state non-goals
- the proposal needs global ownership or registration to make sense
- the design is turning into a framework while still being described as a core

## Design Rules

Apply these constraints by default:

- prefer explicit ownership over ambient access
- prefer deterministic behavior over implicit automation
- prefer data flow that can be reasoned about without scene context
- prefer synchronous APIs unless the work is truly asynchronous
- prefer a pure C# core layer when Unity types are not the true boundary
- prefer a thin Unity adapter layer when ScrollRect, RectTransform, MonoBehaviour, or other Unity APIs are the true boundary
- prefer small sets of concrete types over plugin surfaces or option matrices

Keep the public API tight:

- expose only the operations that current repeated use cases require
- hide internal queues, transition tables, pooling maps, or mutation steps unless callers must coordinate them
- do not mix events, callbacks, virtual methods, and strategy interfaces without a hard reason
- do not add extension hooks before the second real extension need exists

## Layering Rules

Use the smallest layering that still protects the core boundary.

### Pure Core

Use a pure core when the reusable problem is independent from Unity object lifecycles.

Examples:

- transition rules for a state machine
- visible range calculation for a virtual list
- local execution plus compensation rules for a reversible command batch

### Core Plus Adapter

Add a Unity adapter only when Unity APIs are the real integration boundary.

Examples:

- `ScrollRect` and `RectTransform` for a virtual list
- scene hooks or update forwarding around an otherwise pure state machine
- binding a command result to a view or input pathway

The adapter may use `MonoBehaviour`.
The core should not require `MonoBehaviour` unless Unity object identity is itself the reusable problem.

## First Version Rules

The first version of a narrow core must be obviously smaller than the final system people may imagine.

The first version should:

- solve only the repeated base case
- reject advanced scenarios explicitly in docs
- avoid configuration surfaces that only prepare for future growth
- use one integration model at a time

Examples:

- virtual list: vertical, fixed-height, `ScrollRect`, local data only
- reversible command core: serial execution plus reverse compensation only
- state machine core: explicit transitions plus enter/exit hooks only

The first version should not include:

- global history
- redo stacks
- editor visualization
- nested graphs
- dynamic layout modes
- concurrency orchestration
- plugin ecosystems

## Host Coupling Rules

Do not bind the core directly to the current host framework unless the host abstraction is itself the intended reusable boundary.

Prefer:

- core owns rules and data flow
- host layer owns construction, lifecycle bridging, and wiring

Avoid:

- baking current component models into the core API
- forcing the core to know about scene hierarchy or dependency lookup
- making current game architecture the reason the core exists

## Overlap Check

Before approving a narrow core, explicitly compare it against:

- existing bus or message abstractions
- existing observable or property models
- existing pool or cache primitives
- existing sequencing, flow, or async infrastructure
- existing UI container patterns

If overlap is high, either:

- reject the core
- narrow it further
- or make it an adapter on top of the existing abstraction instead

## Output Standard

When implementing or reviewing a narrow core, answer in this order:

1. What exact stable problem does this core own?
2. Why is this too large for a tiny helper?
3. Why is this still smaller than a framework?
4. What are the non-goals?
5. Who owns it and who disposes or invalidates it?
6. Where is the Unity boundary, if any?
7. What is the smallest first version?
8. What future change would justify a second abstraction?

## Documentation Sync

When a narrow core candidate is discussed, prioritized, accepted, deferred, or reshaped, sync the corresponding design and planning result to the user's Notion documentation when that documentation set already exists for the topic.

Default expectations:

- keep core boundary pages, plan pages, and historical judgment pages aligned with the latest accepted conclusion
- avoid leaving chat-only conclusions for candidate cores that are already tracked in Notion
- prefer consolidating into the canonical page for that core instead of creating redundant follow-up pages
- if a new Notion page is necessary, include a suitable Emoji page icon and a matching cover background

## Default Template

Use this template before writing code:

- Responsibility: what exact stable problem this core owns
- Non-goals: what this core intentionally does not own
- Owner: who constructs it and who tears it down
- Lifetime: when it becomes invalid
- Data flow: what enters, what leaves, what stays internal
- Unity boundary: which types require an adapter layer
- Overlap risk: which existing abstractions may compete with it
- Expansion guard: which feature would mean it is becoming a system

## Anti-Patterns

Do not do these by default:

- build a general command framework when only local reversible batches are justified
- build a global state machine system when explicit transition rules are enough
- build a complete list framework when a virtualized range core plus adapter is enough
- build a host-specific wrapper and call it a reusable core
- build extension-heavy APIs to compensate for unclear boundaries
- build a manager singleton for something that should be locally owned

If a user asks for one of these in core form, refuse that shape and explain the narrower acceptable alternative.

## Relationship To Unity Minimal Components

Use `unity-minimal-components` when the abstraction should stay helper-sized.

Use `unity-narrow-core` when:

- multiple files are justified
- a core and adapter split is justified
- the abstraction is a reusable foundation for larger systems
- but the design still must stay far smaller than a framework