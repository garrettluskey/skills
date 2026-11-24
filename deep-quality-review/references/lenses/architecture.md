# Architecture

**Question:** Does the change put responsibilities in the right layers and components, at a size proportional to the problem?

Find the simplest shape that meets the current requirements and keeps the invariants the code needs. Judge the structure the change actually has, not hypothetical future requirements or a preferred pattern. Structure means the types and interfaces, their fields, the layer each sits in and the dependencies between components. Read a method body only to confirm a structural claim, such as that a service only passes calls through. Code inside a method goes to the other lenses.

## Signals worth a closer look

These are signals, not findings. A finding still needs a concrete maintenance, readability or correctness cost.

- A simple operation that crosses several pass-through layers
- An interface, factory or builder with one meaningful implementation
- An ordinary change that needs edits across many layers
- The same state held in more than one representation
- Event-driven code for what is really a direct request and response
- Permanent fallback or compatibility paths
- Several mechanisms solving the same problem
- Vague `Manager`, `Service` or `Coordinator` classes with broad reach

## Look for

- **Placement.** Code in the wrong layer or component: UI reaching into persistence, domain logic in transport or handler code, game or engine logic in the networking layer.
- **Abstractions that don't earn their keep.** One-use interfaces, factories, wrappers, pass-through services, and registration or plugin systems built for one case.
- **Over-engineering.** Configuration, extension points, or generality for requirements nobody has today.
- **State ownership.** More than one source of truth, such as two types holding the same value or a copy kept in sync with its original, unclear lifecycle ownership, or implicit state machines.
- **Control flow.** Events or callbacks declared between components where a direct call would do.
- **Size.** A handwritten file approaching 800 lines whose parts belong in different layers or components. Splitting a class by responsibility goes to Cohesion.

## Not this category

- Line-level clarity goes to Readability.
- Duplication, naming and convention fit go to Maintainability.
- The edges between components go to Coupling: dependency direction, cycles, hidden dependencies and reaching into internals. Architecture judges which units exist and where they sit.
- Single responsibility inside a class, and data flow, go to Cohesion.
- Reflection or async machinery inside a method goes to Maintainability as a new paradigm, or to Readability as clever code.
- A change that would touch many files is a signal, not a finding. Raise its cause in the lens that owns it.
