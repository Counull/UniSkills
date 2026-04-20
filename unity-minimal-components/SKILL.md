---
name: unity-minimal-components
description: Design and review tiny reusable Unity framework components with single responsibility, low coupling, minimal API surface, and no speculative abstractions. Use when adding or evaluating small helpers such as disposables, leases, scoped values, flags, binders, queues, or guards intended for reuse across Unity projects while avoiding unnecessary dependencies on gameplay, UI, scene, editor, or project-specific systems.
---

# Unity Minimal Components

Use this skill to constrain framework-level helper code.
Prefer components that solve one narrow problem well over flexible mini-frameworks.

## Workflow

Follow this order:

1. Inspect existing code before creating anything new.
2. State the single problem the component solves in one sentence.
3. Identify the nearest existing abstraction that may already cover it.
4. Reject the new component if it overlaps existing property, event, command, update-loop, or async infrastructure.
5. Keep the public API as small as possible.
6. Implement the minimal version that solves today's real call sites.
7. Validate with one or two realistic usage examples.

## Admit Or Reject

Admit a new component only if all of the following are true:

- Solve one independent problem.
- Remain understandable from a single file.
- Avoid owning unrelated lifecycle, scheduling, or global state.
- Reuse plain C# semantics first.
- Depend on Unity only when Unity types are the real boundary.
- Fit multiple Unity projects without carrying current-project vocabulary.
- Use project-agnostic names and concepts rather than current-game terms, system names, or feature names.

Reject or delay a new component when any of the following appears:

- Wrap syntax without adding semantic value.
- Compete with an existing bus, observable base class, command channel, pool, or scheduler.
- Add extension points before the second concrete use case exists.
- Invent interfaces only to make the design look generic.
- Pull in `MonoBehaviour`, scene objects, editor APIs, or ScriptableObject without a hard need.
- Add thread-safety, async orchestration, or caching without an actual caller demanding it.
- Encode current project semantics into the component name, API, or folder shape when the underlying problem is generic.

## Boundary Enforcement

Treat the reject or delay conditions above as blocking guards, not soft suggestions.

If a request crosses the helper boundary, do not continue into implementation by default.
Instead, explicitly refuse the component in its current shape and redirect to one of these actions:

- narrow the responsibility until it fits a tiny helper
- move the discussion to `unity-narrow-core` if the problem is legitimately larger than a helper
- keep the outcome at design or documentation level instead of writing code

When refusing, state which boundary was crossed.
Typical refusal triggers include:

- the request is becoming a bus, scheduler, queue framework, or manager
- the request needs more than one independent responsibility
- the API needs option matrices, strategy hooks, or speculative extension points
- the implementation would encode current project semantics into a supposedly generic helper

## Design Rules

Apply these constraints by default:

- Prefer `sealed class`, `readonly struct`, or a tiny interface.
- Prefer explicit ownership over ambient globals.
- Prefer constructor injection over service lookup.
- Prefer `IDisposable`-style scope cleanup for temporary state.
- Prefer deterministic behavior over automatic magic.
- Prefer synchronous APIs unless the work is truly asynchronous.
- Prefer nullable-correct APIs and obvious invalid-state handling.

Keep the API surface tight:

- Expose only the operations needed by current call sites.
- Hide internal counters, queues, or mutation steps unless callers must coordinate them.
- Avoid callbacks plus events plus virtual methods in the same component.
- Avoid boolean option matrices; split components instead.
- Prefer generic nouns like `Lease`, `Handle`, `Scope`, `Gate`, or `Token`; avoid names derived from one project subsystem such as date, mail, rank, battle, or map unless that subsystem is the actual reusable boundary.

Keep dependencies tight:

- Default to `System` and `System.Collections.Generic`.
- Add UnityEngine dependency only when the component must accept or return Unity types.
- Do not couple framework helpers to input, UI, animation, scene loading, or networking subsystems unless that subsystem is the point of the component.

## Preferred Families

Good candidates usually look like this:

- Resource and lifetime helpers: lease, handle, disposable action, composite disposable.
- Scope helpers: temporary value restore, reentrancy guard, once flag.
- Small coordination helpers: local queue, local latch, version stamp, state token.
- Small adapters: convert one narrow callback pattern into another without introducing a bus.

See `references/component-candidates.md` for examples of good fits and common overdesign traps.

## Output Standard

When implementing a component:

1. Place it under a framework-level folder grouped by topic rather than by feature.
2. Keep it in one file unless a second file is forced by ownership or type clarity.
3. Add brief comments only where invariants are easy to misuse.
4. Include one minimal usage example in the explanation or adjacent docs when helpful.
5. Call out overlap risks with any existing abstractions you inspected.

## Documentation Sync

When a component proposal, priority change, design boundary, usage example, or implementation milestone is being recorded during collaboration, sync the corresponding result to the user's Notion documentation when that documentation set already exists for the topic.

Default expectations:

- keep the Notion plan, example, and design pages aligned with the latest accepted decision
- avoid leaving chat conclusions unsynchronized when the same topic is already tracked in Notion
- prefer updating the existing canonical page instead of creating scattered follow-up pages
- if a new Notion page is necessary, include a suitable Emoji page icon and a matching cover background

When reviewing a proposed component, answer in this order:

1. Restate the problem it claims to solve.
2. List overlapping existing abstractions.
3. Explain the data flow or ownership model.
4. Point out overdesign risk, lifecycle risk, and dependency risk.
5. Recommend the smallest acceptable shape.

## Default Template

Use this mental template before writing code:

- Responsibility: what exact problem does this file own
- Owner: who creates it and who disposes it
- Lifetime: when it becomes invalid
- Failure mode: what happens on misuse
- Dependency boundary: which namespaces it may depend on
- Exit rule: what future requirement would justify a second abstraction

## Anti-Patterns

Do not do these by default:

- Build a general event queue when a local callback list is enough.
- Build a general command framework when existing sequencing infrastructure already exists.
- Build a reactive property type beside an established observable data model without a clear boundary.
- Build a manager singleton for a helper that can stay local to one owner.
- Build extension-heavy APIs that hide order-of-operations or disposal semantics.

If a user asks for one of these in helper form, refuse that shape and explain the narrower acceptable alternative.
