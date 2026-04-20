# Component Candidates

Use this file only when the task needs examples or boundary checks.

## Usually Good Fits

- `DisposableAction`
  Wrap one cleanup action into `IDisposable`.
- `CompositeDisposable`
  Aggregate multiple cleanup handles under one owner.
- `ScopedValue<T>`
  Set a temporary value and restore it on dispose.
- `OnceFlag`
  Gate one-time execution without introducing a state machine.
- `VersionStamp`
  Mark stale handles or cached reads with a monotonic version.
- `ManualLatch`
  Express a simple open or closed gate with explicit transitions.

## Usually Bad Fits

- `Property<T>` as a second reactive foundation beside an existing observable model.
- Global event queue or event bus wrappers that only rename dispatch methods.
- Generic command layer that competes with an existing per-target command channel.
- Universal scheduler, update runner, or coroutine host added for hypothetical reuse.
- Abstract base classes that mainly exist to save a few repeated lines.

## Upgrade Rules

Allow a small component to grow only when at least one is true:

- Two or more real call sites repeat the same awkward ceremony.
- A concrete bug came from unclear ownership or disposal semantics.
- A project-independent Unity boundary became obvious, such as `UnityEngine.Object` lifetime checks.

If none of these are true, keep the component smaller.
