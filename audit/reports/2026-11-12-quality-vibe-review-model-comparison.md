# quality-vibe-review: reviewer model comparison

Date: 2026-11-12. Skill: `quality-vibe-review`, PR mode, dry run for every config except the baseline.

## Setup

- **Subject:** one real pull request in a private C# codebase, about 1,000 added lines across 13 files. It changes a server-authoritative simulation lifecycle (completion, cancellation, disconnect cleanup), adds a client-side continuation flow, and adds about 800 lines of end-to-end tests.
- **Reviewers:** one subagent per category, eight per config, all given the same brief, lens file and `references/scoring.md`.
- **Configs:**
  - Claude Opus 5.5 through Claude Code's subagent tool, at its default effort. This is the baseline, and its audited review was the one posted.
  - Codex `gpt-6.1-sol`, `gpt-6-astra` and `gpt-6.1-terra`, each at reasoning effort `high` and `xhigh`, started as Codex child tasks.
- **Ground truth:** the baseline's 21 findings after the skill's verify and audit steps, plus two findings only Codex produced. Every finding was checked by hand against the code.

A first `gpt-6.1-sol` pass at the default `low` effort was cut short and isn't counted. Its two finished reviewers scored Architecture 10 and Coupling 9, the same as the `high` run.

## Scores (raw, before verification)

| Config | Arch | Read | Maint | Coup | Coh | Testab | TestQ | Err | Mean |
|---|---|---|---|---|---|---|---|---|---|
| Opus 5.5 | 5 | 6 | 6 | 6 | 6 | 7 | 7 | 6 | **6.1** |
| Sol high | 10 | 9 | 9 | 8 | 10 | 10 | 9 | 7 | **9.0** |
| Sol xhigh | 10 | 9 | 8 | 9 | 10 | 9 | 9 | 7 | **8.9** |
| Astra high | 10 | 9 | 8 | 9 | 10 | 10 | 9 | 7 | **9.0** |
| Astra xhigh | 10 | 9 | 9 | 9 | 10 | 9 | 8 | 7 | **8.9** |
| Terra high | 10 | 9 | 8 | 9 | 10 | 10 | 10 | 8 | **9.25** |
| Terra xhigh | 8 | 9 | 8 | 7 | 10 | 8 | 8 | 10 | **8.5** |

The Codex configs land within 0.75 of each other and about 2.5 to 3 points above Opus. Cohesion was a 10 in all six Codex configs, and Architecture a 10 in five of the six.

## Findings recovered

✓ found, ✗ missed, ~ found under another category.

| Finding | Opus | Sol H | Sol X | Astra H | Astra X | Terra H | Terra X |
|---|---|---|---|---|---|---|---|
| A doc comment moved onto the wrong method | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| A dependency silently falls back to a container lookup, then a global singleton | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| Cancel path no longer releases its lock when teardown throws (regression) | ✓ | ✓ | ✓ | ✓ | ✓ | ✗ | ✗ |
| Cancel handler re-implements an existing cancel helper | ✓ | ✓ | ✓ | ✓ | ✓ | ✗ | ✗ |
| Disconnect path copies the completion sequence and its send order | ✓ | ✗ | ✓ | ✓ | ✗ | ✓ | ✓ |
| A 10-clause decision buried in a side-effecting method, so it can't be tested alone | ✓ | ✗ | ✓ | ✗ | ✓ | ✗ | ✓ |
| An ordering test checks a stand-in, while the real handler skips the code under test | ✗ | ✓ | ✗ | ✗ | ✓ | ✗ | ✓ |
| A 240-line test theory whose second row only repeats the first | ✓ | ✗ | ✓ | ✓ | ✗ | ✗ | ✗ |
| A field read and cleared on the network thread but written on the game thread | ✓ | ✗ | ✗ | ✗ | ✗ | ✗ | ✓ |
| Request-lifecycle state placed in an unrelated service | ✓ | ✗ | ✗ | ✗ | ✗ | ✗ | ~ |
| A failed id lookup can strand a restarted flow (low likelihood) | ✗ | ✗ | ✗ | ✗ | ✗ | ✓ | ✗ |
| One outcome now ends two different ways, client-driven or server-driven | ✓ | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ |
| Completion bypasses the component that owns finalizing (contested) | ✓ | ✗ | rejected | ✗ | ✗ | ✗ | ✗ |
| A "finished" flag set after a call that can throw, stranding clients | ✓ | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ |
| Direct-call teardown tests are low value under the repo's test rules (contested) | ✓ | ✗ | ✗ | rejected | rejected | ✗ | ✗ |
| Minor readability and maintainability items (8) | 8 | 0 | 0 | 0 | 0 | 0 | 0 |
| **Distinct verified findings** | **21** | **5** | **7** | **6** | **6** | **4** | **7** |

## Observations

**Codex is precise, Opus is broad.** Every Codex finding that was checked held up, with correct line ranges and fixes that fit the code. Opus found 3 to 5 times as many issues, but the audit step had to correct 4 wrong claims: a citation past the end of a file, "nothing calls these constructors" when four tests did, a claim about why some test patches existed, and off-by-one line numbers. The merge step also folded 11 duplicates across categories.

**Codex leans on the sibling-convention rule.** Most Codex 10s came with a DROPPED line saying sibling classes do the same thing, which is the skill's own rule in step 4.1. Sometimes that was right: one Codex run correctly showed that several other services also finalize directly, which makes one Opus architecture item debatable. But the rule also hid the clearest architecture problem in the change, an outcome that now ends through two different mechanisms, which no Codex config raised.

**Effort mattered more than model within Codex.** `xhigh` averaged 6.7 verified findings against 5.0 at `high`, while the mean score barely moved (8.75 against 9.08). The clearest case: all three `xhigh` runs asked to pull the buried decision out into a testable method, and all three `high` runs considered it and dropped it.

**Codex found a real test gap Opus missed.** The ordering test checked a stand-in condition in its own subscriber, while the production handler returned early in the test setup, so the behaviour the test was named for never ran. Three of six Codex configs caught it. One Codex config instead praised the same test as proof of the ordering.

**Per model:**
- **Sol `xhigh`** was the most balanced: it found the four findings most Codex runs agreed on, plus three more, and backed its one rejection of an Opus item with evidence.
- **Astra** behaved almost like Sol at the same effort.
- **Terra** was the most uneven. `xhigh` found the most unusual items (the cross-thread field, the misplaced state, the test gap) and scored harshest, but missed the error-handling regression that every Sol and Astra run caught. `high` was the most lenient and found the fewest findings.

## Recommendation

- **Reviewers:** use Opus for reviews that get posted, and keep the verify and audit steps. Opus was the only model to surface the structural and error-path problems, and the audit catches its wrong claims cheaply. A Codex scorecard on this change would have read as nearly done.
- **Optional second opinion:** add one Codex `gpt-6.1-sol` `xhigh` Test quality reviewer and merge its items in step 4. That's the one category where Codex found something Opus missed.
- **Avoid:** Codex alone (too lenient, with one repeated blind spot), Codex at `high` (drops findings `xhigh` keeps), and Terra for a single-reviewer pass (least consistent).

## Limits

- One change and one run per config, so there's no variance estimate. Some differences between configs, such as Terra `xhigh` catching one item and missing another, look like run-to-run variance.
- Time and token cost weren't recorded.
- Ground truth starts from the Opus review, which favours the kinds of findings Opus produces. The two Codex-only findings show the gap runs both ways.
- Codex efforts `max` and `ultra` weren't tested. Most Codex misses were deliberate rejections rather than oversights, so more effort may not close them.
