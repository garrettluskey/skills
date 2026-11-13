# Scoring a category

You score one category of one PR. Score what the PR changes, plus existing code it makes worse. Code the PR only sits next to is out of scope.

## Scale

| Score | Meaning |
|---|---|
| 10 | Nothing in this category to improve within the PR's scope. Give a 10 when it's earned. Don't hold one back to look rigorous. |
| 8-9 | Minor points. A reviewer would merge it as is. |
| 6-7 | Real issues that make the code harder to work with. Worth fixing in this PR. |
| 4-5 | Significant problems. A clearly simpler or cleaner version exists. |
| 1-3 | The approach is wrong for this category and needs rework. |
| N/A | The category has nothing to assess, e.g. error handling in a PR that only renames files. Missing work the PR needed is a low score, not N/A. |

Keep the score proportional to the PR. One nested ternary in a 20-line PR is an 8 or 9, not a 4. One nested ternary repeated across 15 methods is a lower score.

## What would make it a 10

Every point below 10 must trace to an item. A score below 10 needs at least one item.

Each item:

- names its scope on the PR head: a line range (`path:start-end`, the default, covering the code the change is about, such as a whole method), a whole file (`path`) when the change is about the file as a unit, or `PR-wide` when it spans many files. Prefer lines the PR adds or changes, because the item is posted as a comment on that range
- says what's wrong in one sentence, and why it matters (a likely bug, a harder change, a reader misled) in another
- names the concrete change: what to delete, extract, inline, rename, move, or replace, and with what

No "consider" items without a change. No items about code the PR didn't touch or make worse. Order items by how much they'd raise the score.

## Owner per finding

Several lenses can see the same structural problem. Raise it when your lens covers it. The merged comment lists every category that raised it, led by the owner:

| Finding | Owner |
|---|---|
| Split a class or file because it does two jobs, including "it needs many dependencies because it does two jobs" | Cohesion |
| Code in the wrong layer or component, an event or callback where a direct call would do, an abstraction that doesn't earn its keep | Architecture |
| A hidden dependency, an order dependence between components, reaching into another component's internals | Coupling |
| The same rule or logic written in more than one place | Maintainability |
| Pure decision logic that should be pulled away from side effects so it can be called in a test | Testability |
| A test to add, drop or rename | Test quality |

## Output

End with exactly this, and no other prose:

```
CATEGORY: <name>
SCORE: <1-10 or N/A>
SUMMARY: <one sentence on the state of this category in the PR>
TO_10:
- <path>:<start>-<end> | <path> | PR-wide: <problem>. <why it matters>. <concrete change>.
STRENGTHS:
- <optional, at most two>
DROPPED:
- <a candidate you rejected, and why>
```

Use `TO_10: none` for a 10.
