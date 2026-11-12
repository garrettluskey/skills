# Test quality

**Question:** Does the PR have the tests it needs, and only those?

Agents can review code statically far more cheaply than a team can maintain tests, so every test has to earn its upkeep. Keep high-value tests. Flag low- and medium-value tests for removal.

## High value (keep)

- Real branching logic, edge cases, and boundary conditions
- A regression test for a bug the PR fixes
- Invariants and contracts between components
- Integration across a real boundary: serialization round-trips, network or sync, persistence
- Concurrency and ordering behaviour

## Low or medium value (flag to drop)

- Tests of getters, setters, constructors, or trivial mapping
- Tests that only check mocks were called in the order the implementation calls them, which just restates the implementation
- Near-duplicates of another test with a different literal. Merge them into one parameterized test or drop them.
- Tests of framework, language, or library behaviour
- Snapshot or golden tests nobody will read
- Tests that would still pass if the code under test were deleted
- Tests where the mocks are the test

## Also check

- **Missing tests.** New non-trivial logic, or a bug fix, with no test of that behaviour. Score this down and name the test to add.
- **Assertions.** Specific to the outcome, not `NotNull` or "didn't throw" when a real value can be checked.
- **Names.** Test names say the behaviour and the expected result.
- **Flakiness.** Sleeps, real time, real network, or order dependence between tests.
- **Setup.** Arrange code so long the point of the test gets lost.

List each test to drop as its own item: `path:line: drop <TestName>. <why it's low value>.`

Use N/A only when the PR adds no tests and no testable logic.

## Not this category

- Whether the production code can be tested goes to Testability.
