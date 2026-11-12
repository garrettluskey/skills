# Testability

**Question:** Can the changed behaviour be tested in isolation without heroics?

Score the production code's design. Whether tests exist, and whether they're good, belongs to Test quality.

## Look for

- **Hard-wired dependencies.** IO, network, time, randomness, the game engine, statics, or singletons created or reached directly, with no way to substitute them.
- **Logic tangled with side effects.** Decision logic that can't be called without also doing IO or mutating global state. The pure part should be extractable.
- **Constructors that do work.** Constructors that connect, load, subscribe, or start threads.
- **Hidden state and order dependence.** Behaviour that depends on what ran before, through fields or globals the test can't see or set.
- **Unreachable behaviour.** Important logic that's private and only reachable through a large public surface.
- **Non-determinism.** Threads, timers, sleeps, or unordered iteration in logic that needs a predictable result.

## Not this category

- Tests themselves go to Test quality.
- Dependency direction for its own sake goes to Coupling. Here it only matters when it blocks testing.
- Splitting a class goes to Cohesion. Here, raise only the pure logic to pull out so it can be tested.
