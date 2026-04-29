---
name: mock-isolation-guard
description: Ensure temporary mock code is isolated, removable, and controlled by compile-time or build-time symbols. Use when adding or reviewing network mock, API mock, UI mock, fake data, placeholder resources, greybox implementation, protocol stubs, no-UI prototypes, or resource-missing feature scaffolding.
---

# Mock Isolation Guard

Use this skill whenever a feature needs mock data, temporary network responses, UI placeholders, fake resources, protocol stubs, greybox screens, or no-UI gameplay/product prototypes. The goal is to let development continue before dependencies are ready, while making every mock easy to find, disable, and delete.

## Core Rules

- Network/API mock and UI mock must be separate classes or modules. Do not put fake transport/protocol data and placeholder UI behavior in the same implementation.
- Mock code must be controlled by explicit compile-time, build-time, or environment-gated symbols. Do not rely only on runtime booleans, config rows, local storage, player prefs, or comments.
- Production code should depend on a narrow provider, interface, service, adapter, or dependency-injection boundary, not directly on a mock implementation.
- Mock files must be easy to remove later. Keep them named with `Mock`, `Stub`, or `Fake`, and avoid scattering mock branches through feature logic.
- No mock should silently run in production-like builds. The default path should be real data or a clear unavailable state.
- Mock behavior must be deterministic unless the test explicitly needs randomness. If randomness is needed, inject or record the seed.

## Symbol Policy

Prefer feature-specific symbols instead of one global catch-all.

Recommended patterns:

- `FEATURE_NAME_MOCK_NETWORK`
- `FEATURE_NAME_MOCK_API`
- `FEATURE_NAME_MOCK_UI`
- `FEATURE_NAME_MOCK_DATA`
- `FEATURE_NAME_PLACEHOLDER_ASSETS`

Project teams may add a short organization or product prefix if that is already their convention, but the symbol still needs the feature name and mock category.

Avoid vague symbols such as:

- `USE_MOCK`
- `DEBUG_MOCK`
- `TEST`
- `TEMP`
- `FAKE`

When a mock is editor-only, local-only, or dev-only, combine gates intentionally:

```csharp
#if UNITY_EDITOR && FEATURE_NAME_MOCK_NETWORK
// editor-only mock implementation
#endif
```

```typescript
if (process.env.FEATURE_NAME_MOCK_API === '1') {
  // development mock provider selection
}
```

Use platform/editor/dev-only gates only when the mock is impossible or unsafe outside that context. If QA or staging builds need the mock, use a feature-specific build symbol and ensure it is not enabled for release builds.

## Separation Pattern

### Network Or API Mock

Network/API mock classes may provide:

- Fake protocol payloads.
- Fake request completion.
- Error, timeout, retry, and malformed-response cases for deterministic testing.
- Scenario selection helpers.
- Mock-local data reset.

Network/API mock classes must not:

- Open screens, layouts, modals, or views.
- Change UI state directly.
- Depend on prefab names, component hierarchies, DOM selectors, or view internals.
- Contain visual placeholder logic.

Suggested naming:

- `{Feature}NetworkMock`
- `{Feature}ApiMock`
- `{Feature}MockApiClient`
- `{Feature}MockResponseFactory`
- `{Feature}FakeRepository`

### UI Mock

UI mock classes may provide:

- Placeholder view models.
- Greybox display data.
- Temporary icons, labels, images, or layout metadata for missing resources.
- Development-only commands to open screens.
- Static sample states for visual review.

UI mock classes must not:

- Build fake server responses.
- Mutate authoritative business state outside the feature API.
- Pretend that a request succeeded without going through the feature data layer.
- Own protocol status codes or transport details.

Suggested naming:

- `{Feature}UIMock`
- `{Feature}MockViewModelProvider`
- `{Feature}PlaceholderAssetProvider`
- `{Feature}MockScreenStateFactory`

### Data Or Gameplay Mock

Data/gameplay mock classes may provide:

- Deterministic seed data.
- Local scenario snapshots.
- State-machine or rule-engine examples.
- Pure logic fixtures that do not depend on UI or transport.

Data/gameplay mock classes must not:

- Bypass production validation paths.
- Write incompatible data into real persistent storage.
- Mix UI placeholder state with domain state.

Suggested naming:

- `{Feature}MockScenarioFactory`
- `{Feature}FakeDataSource`
- `{Feature}MockStateBuilder`

## Recommended Architecture

Use a small interface or provider boundary when a mock substitutes real data.

Example shape:

```csharp
public interface IFeatureDataSource
{
    IEnumerator InitAsync(int entityId, Action<FeatureSnapshot> onSuccess, Action onFailure);
}

public class FeatureNetworkDataSource : IFeatureDataSource
{
    public IEnumerator InitAsync(int entityId, Action<FeatureSnapshot> onSuccess, Action onFailure)
    {
        // real request path
        yield break;
    }
}

#if FEATURE_NAME_MOCK_NETWORK
public class FeatureMockDataSource : IFeatureDataSource
{
    public IEnumerator InitAsync(int entityId, Action<FeatureSnapshot> onSuccess, Action onFailure)
    {
        onSuccess?.Invoke(FeatureMockResponseFactory.CreateSnapshot(entityId));
        yield break;
    }
}
#endif
```

The real feature owner chooses the implementation in one narrow place:

```csharp
private IFeatureDataSource CreateDataSource()
{
#if FEATURE_NAME_MOCK_NETWORK
    return new FeatureMockDataSource();
#else
    return new FeatureNetworkDataSource();
#endif
}
```

Do not spread `#if FEATURE_NAME_MOCK_*` throughout unrelated UI, domain, reward, persistence, or config code.

## Workflow

### 1. Classify The Mock

Before coding, classify each temporary path as one of:

- `network mock`: fake request/response, API client, transport, or protocol data.
- `UI mock`: greybox display, placeholder binding, missing asset behavior, or static screen state.
- `config mock`: temporary config rows or in-memory config provider.
- `data mock`: fake repository, local snapshots, deterministic sample state.
- `domain mock`: deterministic rule-engine or state-machine scenario for no-UI tests.

If more than one category is needed, create separate classes or modules.

### 2. Define The Removal Boundary

Write down what file, symbol, provider registration, or dependency-injection binding will be removed when the real dependency arrives.

Each mock should answer:

- Which symbol or build gate enables it?
- Which real class/module replaces it?
- Which files should be deleted later?
- Is any persisted local data compatible with real data?
- What validation proves the no-mock path still works?

### 3. Keep Feature Logic Real

The feature logic should call the same public API regardless of mock or real implementation.

Good:

- A controller/data manager calls `IFeatureDataSource.InitAsync`.
- A view binds to a real domain model or view model.
- Domain logic receives a snapshot and executes real rules.
- Mock selection happens in a factory, registration method, or composition root.

Bad:

- A view creates fake protocol data.
- Domain logic checks `if (useMock)` in many methods.
- A mock response modifies UI directly.
- Mock-only item IDs or status values leak into production reward/business logic.

### 4. Make Mock State Resettable

For prototypes and scenario testing, provide deterministic reset helpers gated by the same symbol.

Example:

```csharp
#if FEATURE_NAME_MOCK_DATA
public void ResetMockState(int entityId)
{
    // reset only mock-local state
}
#endif
```

Do not clear real user/player/customer data from a mock reset method unless the user explicitly asks for a destructive debug action.

### 5. Document The Symbol

When adding a symbol-gated mock, update the relevant analysis, implementation, or development document with:

- Symbol name.
- Mock class/module names.
- How to enable it.
- How to remove it.
- What real dependency it replaces.
- Known limitations.

## Placement Guidance

Prefer locations that make mock ownership obvious:

- Next to the feature behind a clearly named `Mock`, `Mocks`, `Stub`, `Stubs`, `Fake`, `Fakes`, `Debug`, or `Development` folder.
- In a test fixture folder if the mock is only for automated tests.
- In an editor/devtools folder if the mock is only for editor or development tooling.
- In a composition-root or provider-registration area only for the small selection point, not for full mock implementations.

Avoid:

- Putting mock classes in generic production utility folders.
- Editing generated files to add mock behavior.
- Mixing mock fixtures into production config/data files that can ship accidentally.
- Adding mock-only fields to stable public protocols unless the protocol itself is explicitly a test contract.

## Review Checklist

Before finishing mock-related work, verify:

- Network/API mock and UI mock are separate classes or modules.
- Every mock implementation is behind a feature-specific symbol or build gate.
- The symbol is not generic.
- The no-symbol path still compiles and uses the real provider or an explicit unavailable state.
- Mock selection is centralized in one or two factory/provider/composition methods.
- Feature logic uses the same API in mock and real modes.
- Mock code does not write incompatible real persistent data.
- Mock reward, payment, entitlement, or inventory paths do not bypass production safety rules unless the feature is explicitly testing a no-side-effect prototype.
- Mock classes/modules are named so they can be found with searches like `rg "Mock|Stub|Fake|MOCK_.*|.*_MOCK"`.
- Documentation lists how to enable and remove the mock.

## Common Mistakes

- Hiding mock behavior behind editor/dev mode without a feature-specific symbol.
- Letting a temporary UI button directly mutate server-owned or authoritative state.
- Putting fake protocol payload creation inside a view.
- Adding a mock flag to data/config that might ship accidentally.
- Writing mock-only IDs into real local saves or persistent storage.
- Forgetting to verify the no-symbol build path after adding mock files.
- Reusing one giant mock class for UI, network, domain, and persistence.

## Output Expectations

When using this skill, include:

- The mock categories involved.
- The compile/build symbols or environment gates used.
- The files/classes/modules added for each mock category.
- The real integration point each mock replaces.
- A short removal plan.
- A validation note confirming the no-symbol path is clean or explaining why it was not checked.