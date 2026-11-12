# Coupling

**Question:** How much do the changed units know about and depend on each other?

## Look for

- **Dependency direction.** High-level policy that depends on concrete low-level details instead of an abstraction it owns (Dependency Inversion). Expect abstractions at real boundaries: IO, network, persistence, time, randomness, the game engine, other modules. Inside a module, depending on a concrete class is fine. Don't ask for an interface that has one implementation and no boundary or test need.
- **Hidden dependencies.** Statics, singletons, service locators, or ambient context reached into from deep inside the code instead of being passed in.
- **Knowledge leaks.** Reaching through objects (`a.B.C.D`), depending on another component's internals, shared mutable state, and implementation types leaking across module boundaries.
- **Fan-out.** A class that depends on many others, or a change that forces edits in many unrelated places.
- **Cycles.** Circular dependencies between classes, namespaces, or projects.
- **Temporal coupling.** "Call X before Y" with nothing enforcing it.
- **Message contracts.** Senders and receivers coupled through shared mutable payloads or through knowledge of each other's internals.

## Not this category

- Whether a unit is focused, and how data flows through it, go to Cohesion.
- Over-abstraction and layer placement go to Architecture.
- Whether dependencies can be swapped in tests goes to Testability.
- A class with many dependencies because it does two jobs goes to Cohesion, as a split.
