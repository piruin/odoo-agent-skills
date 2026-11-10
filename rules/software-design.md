# Software Design Principles

Module-level design: how dependencies point and where boundaries sit. `clean-code.md` covers the function and naming level.

## 1. Manage Dependencies Deliberately

- Minimize coupling, not maximize abstraction.
- Prefer dependencies toward stable business rules rather than volatile implementation details.
- Keep dependencies explicit, intentional, and easy to understand.
- Avoid circular dependencies and unnecessary cross-module coupling.
- Evaluate dependency direction when introducing new components or modifying existing boundaries.
- Declare every dependency a module relies on at runtime, not only the ones it imports. A dependency reached through a registry, service locator, plugin lookup, or string-keyed reference is still a dependency; left undeclared, it works where the other component happens to be present and breaks where it is not.

## 2. Protect Meaningful Boundaries

- Separate responsibilities when they have different reasons to change.
- Keep implementation details from leaking across meaningful boundaries.
- Introduce interfaces or dependency inversion when they provide actual value in isolating dependencies.
- Direct dependencies are acceptable when coupling is low-risk and separation adds no meaningful benefit.
- Respect framework conventions and use the framework's supported extension points rather than patching its internals, unless breaking a convention provides a clear advantage.

## 3. Avoid Unnecessary Abstraction

- Every abstraction must justify its existence by protecting a meaningful boundary or reducing a real dependency risk.
- Do not introduce interfaces, repositories, services, adapters, or layers merely to follow an architectural pattern.
- Avoid speculative extensibility and unnecessary indirection.

## 4. Design for Change

- Favor high cohesion within modules and low coupling between modules.
- Keep changes localized wherever practical.
- Preserve existing contracts unless a change is justified.
- Prefer incremental improvements over large structural rewrites.

## 5. Before Adding an Abstraction, Layer, or Dependency

1. What concrete problem does this solve?
2. What coupling or dependency risk does it reduce?
3. Could a simpler design achieve the same result?
4. Does the added complexity justify its maintenance cost?

If the benefits are unclear, prefer the simpler design.

## 6. Agent Behavior

- Follow these principles without assuming any specific architectural pattern.
- Do not restructure existing code solely to conform to a textbook architecture.
- Before significant architectural changes, explain the proposed dependency relationships and trade-offs.
- When reviewing code, identify concrete dependency problems rather than reporting deviations from architectural patterns.
