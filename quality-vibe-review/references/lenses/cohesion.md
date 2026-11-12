# Cohesion

**Question:** Does each changed unit do one thing, and does data flow through the change as simply as possible?

## Look for

- **Mixed responsibilities.** A class or method whose description needs "and". Unrelated logic bundled together, or related logic scattered across unrelated places.
- **Feature envy.** A method that mostly works with another class's data, and belongs there.
- **Tramp data.** Values passed through layers that don't use them, just to reach the one that does.
- **Extra conversions.** Data converted between representations more often than needed, or intermediate DTOs and mappers with no purpose.
- **Parallel state.** Data copied into a second place that must be kept in sync, or more than one source of truth for the same value.
- **Round-trips.** Fetch, transform, write back, then fetch again, when the value was already in hand.
- **Action at a distance.** Out-params, mutated arguments, or shared fields used to return results instead of a return value.

## Not this category

- Dependency direction and hidden dependencies go to Coupling.
- Placement across layers goes to Architecture.
- Duplicated logic goes to Maintainability.
