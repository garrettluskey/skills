# Architecture

**Question:** Does the change put responsibilities in the right layers and components, at a size proportional to the problem?

Read `~/.codex/skills/architecture-review/SKILL.md` for the full checklist, but score only the system-level items below.

## Look for

- **Placement.** Code in the wrong layer or component: UI reaching into persistence, domain logic in transport or handler code, game or engine logic in the networking layer.
- **Abstractions that don't earn their keep.** One-use interfaces, factories, wrappers, pass-through services, and registration or plugin systems built for one case.
- **Over-engineering.** Configuration, extension points, or generality for requirements nobody has today.
- **State ownership.** More than one source of truth, unclear lifecycle ownership, or implicit state machines.
- **Control flow.** Events, callbacks, reflection, or async machinery where a direct call would do.
- **Change cost.** How many files and layers an ordinary follow-up change to this feature would touch.
- **Size.** A handwritten file approaching 800 lines whose parts belong in different layers or components. Splitting a class by responsibility goes to Cohesion.

## Not this category

- Line-level clarity goes to Readability.
- Duplication, naming and convention fit go to Maintainability.
- Dependency direction and hidden dependencies go to Coupling.
- Single responsibility inside a class, and data flow, go to Cohesion.
