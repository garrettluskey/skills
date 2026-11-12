# Maintainability

**Question:** How cheap will this change be to live with and modify later?

## Look for

- **Duplication.** Copy-pasted logic, duplicated business rules, validation or constants, and new code that redoes what an existing helper already does. Search the repo for an existing helper before you call something new.
- **Complexity.** New or changed methods with many branches, many parameters, boolean flag parameters that switch behaviour, or long switches on type.
- **New paradigms.** A library, pattern, framework, async model, DI style, or messaging mechanism the repo doesn't already use. It needs an exceptional reason stated in the PR or obvious from the problem. Otherwise flag it and name the existing approach to use instead.
- **Consistency.** Folder layout, file naming, and patterns that differ from how the repo already solves the same kind of problem. Compare against two or three sibling implementations before you flag anything.
- **Naming.** Names should say what the thing is or does and use the repo's existing domain vocabulary. Flag misleading names, unclear abbreviations, `Manager`/`Helper`/`Util` grab-bags, and two names for one concept.
- **Leftovers.** Dead code, commented-out code, debug logging, and TODOs with no issue or owner.

## Not this category

- Line-level clarity goes to Readability.
- Whether the tests are good goes to Test quality.
- Try/catch structure goes to Error handling.
