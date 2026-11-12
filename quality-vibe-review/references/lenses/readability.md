# Readability

**Question:** Can a teammate read the changed code top to bottom and understand it on the first pass?

## Look for

- **Conditional expressions.** Ternaries (`a ? b : c`) are hard to read, so prefer pattern matching (switch expressions, `is` patterns, `match`) or a plain `if`. Flag nested or chained ternaries, ternaries with long operands or side effects, and ternaries used as statements. One short ternary picking between two simple values is fine.
- **Nesting.** More than two levels deep. Prefer guard clauses and early returns over wrapping the main path in `if` blocks.
- **Unnamed conditions.** Long boolean expressions that should be a well-named variable or method.
- **Long methods.** Several distinct steps in one method with no structure.
- **Clever code.** LINQ or stream chains that hide what a loop would show plainly, operator tricks, implicit conversions, and magic numbers or strings.
- **Comments.** Comments that restate the code, or non-obvious code with no comment saying why.
- **Reading order.** Code that makes the reader jump around (helpers far from their single caller, flags set in one place and read far away) when it could read in order.

## Not this category

- Naming and consistency with the repo go to Maintainability.
- Duplication and complexity metrics go to Maintainability.
- Try/catch structure goes to Error handling.
