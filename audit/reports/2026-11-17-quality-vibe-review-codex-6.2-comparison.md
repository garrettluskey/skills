# quality-vibe-review: Codex 6.2 reviewer comparison

Date: 2026-11-17. Skill: `quality-vibe-review`, PR mode, dry run. A rerun of the [2026-11-12 comparison](2026-11-12-quality-vibe-review-model-comparison.md) on the same code with the Codex 6.2 models.

## Setup

- **Subject:** the same change as the earlier run, [Bannerlord-Coop-Team/BannerlordCoop#4165](https://github.com/Bannerlord-Coop-Team/BannerlordCoop/pull/4165) at commit [`f74ddfd7e`](https://github.com/Bannerlord-Coop-Team/BannerlordCoop/commit/f74ddfd7e6fcba149f5363657d0df5d68cfe455b). It's diffed against merge base [`65f88333f4`](https://github.com/Bannerlord-Coop-Team/BannerlordCoop/commit/65f88333f47fb023b6afe31a6e9341c8ca3cbfee): 13 files, +1001/−59. The PR has since merged, so the diff has to use that merge base, not `origin/development`.
- **Reviewers:** one subagent per category, eight per config, each given the same brief, lens file and `references/scoring.md` as before, plus the PR description as it read when the baseline was posted.
- **Configs:** Codex `gpt-6.2-sol` and `gpt-6.2-astra`, each at reasoning effort `high` and `xhigh`, started as Codex child tasks: 32 reviewers in all. No 6.2 Terra model exists.
- **Ground truth:** the same 23 findings as the earlier run: the 21 comments in the [posted Opus review](https://github.com/Bannerlord-Coop-Team/BannerlordCoop/pull/4165#pullrequestreview-5884752533), plus the two Codex-only findings. Every new claim was checked by hand against the code at `f74ddfd7e`.

## Scores (raw, before verification)

| Config | Arch | Read | Maint | Coup | Coh | Testab | TestQ | Err | Mean |
|---|---|---|---|---|---|---|---|---|---|
| Opus 5.5 (baseline) | 5 | 6 | 6 | 6 | 6 | 7 | 7 | 6 | **6.1** |
| Sol 6.2 high | 7 | 7 | 7 | 6 | 7 | 7 | 7 | 6 | **6.75** |
| Sol 6.2 xhigh | 7 | 7 | 7 | 6 | 7 | 8 | 7 | 6 | **6.88** |
| Astra 6.2 high | 7 | 7 | 7 | 7 | 7 | 8 | 8 | 7 | **7.25** |
| Astra 6.2 xhigh | 7 | 7 | 7 | 7 | 7 | 8 | 7 | 6 | **7.0** |
| *6.1 Codex range* | *8–10* | *9* | *8–9* | *7–9* | *10* | *8–10* | *8–10* | *7–10* | *8.5–9.25* |

The 6.2 configs land within 0.5 of each other and about 0.6 to 1.1 points above Opus, where the 6.1 configs were 2.5 to 3 points above. Every 6.2 config scored Architecture, Readability, Maintainability and Cohesion 7. In 6.1, Cohesion was a 10 in all six Codex configs and Architecture a 10 in five.

## Ground-truth findings recovered

✓ found, ✗ missed, ~ found partly or under another item. The 6.1 column counts how many of its six configs found the item.

| Finding | Opus | Sol H | Sol X | Astra H | Astra X | 6.1 Codex |
|---|---|---|---|---|---|---|
| A doc comment moved onto the wrong method | ✓ | ✓ | ✓ | ✓ | ✓ | 6/6 |
| A dependency silently falls back to a container lookup, then a global singleton | ✓ | ✓ | ✓ | ✓ | ✓ | 6/6 |
| Cancel path no longer releases its lock when teardown throws (regression) | ✓ | ✓ | ✓ | ✓ | ✓ | 4/6 |
| Cancel handler re-implements an existing cancel helper | ✓ | ✓ | ✓ | ✓ | ✓ | 4/6 |
| Disconnect path copies the completion sequence and its send order | ✓ | ✓ | ✓ | ✓ | ✓ | 4/6 |
| A 10-clause decision buried in a side-effecting method, so it can't be tested alone | ✓ | ✓ | ✓ | ✓ | ✓ | 3/6 |
| Request-lifecycle state placed in an unrelated service | ✓ | ✓ | ✓ | ✓ | ✓ | ~1/6 |
| A "finished" flag set after a call that can throw, stranding clients | ✓ | ✓ | ✓ | ✓ | ✓ | 0/6 |
| A field read and cleared on the network thread but written on the game thread | ✓ | ✓ | ~ | ✗ | ✓ | 1/6 |
| An ordering test checks a stand-in, while the real handler skips the code under test | ✗ | ✓ | ✓ | ✗ | ✗ (praised it) | 3/6 |
| Completion bypasses the component that owns finalizing (contested) | ✓ | ✓ | rejected | ✗ | ✗ | 0/6 |
| Direct-call teardown tests are low value under the repo's test rules (contested) | ✓ | ✗ | ~ | ✗ | ✓ | 0/6 |
| A 240-line test theory whose second row only repeats the first | ✓ | ✗ | ✗ | ✓ | ✗ | 2/6 |
| One outcome now ends two different ways, client-driven or server-driven | ✓ | ✗ | ✗ | ✗ | ✗ | 0/6 |
| A failed id lookup can strand a restarted flow (low likelihood) | ✗ | ✗ | ✗ | ✗ | ✗ | 1/6 |
| Minor readability and maintainability items (8) | 8 | 1 | 0 | 0 | 0 | 0 |
| **Ground-truth findings found (of 23)** | **21** | **12** | **11** | **9** | **10** | **4–7** |

## New verified findings

These are real issues in the change that the posted review didn't raise. Each was checked against the code.

| Finding | Sol H | Sol X | Astra H | Astra X |
|---|---|---|---|---|
| The denial test seeds the pending request itself, so it passes if the real handler stops tracking it | ✓ | ✓ | ✓ | ✓ |
| The 240-line raid test buries its two battle sequences; split it into named phases | ✓ | ✓ | ✓ | ✓ |
| The reinforced raid repeats the first raid's drive-to-victory loop in the same test | ✓ | ✓ | ✓ | ✗ |
| The rejected-start test never asserts the start reply that unblocks the requester | ✓ | ✓ | ✗ | ✓ |
| The handler reads `PlayerEncounter.Current` before calling its injected tracker | ✓ | ✓ | ✗ | ✗ |
| A new raid fixture field is only ever copied into another fixture | ✓ | ✗ | ✗ | ✓ |
| The pacer-disconnect test counts loot after cleanup, so it can't catch a send moved below finalization | ✗ | ✗ | ✓ | ✓ |
| A blocking server dispatch that times out sends no reply, and the client then drops every later village request (low likelihood) | ✗ | ✓ | ✗ | ✗ |
| The Retreat handler nests the leader-or-joiner decision under three checks | ✗ | ✓ | ✗ | ✗ |
| The disconnect catch logs "cancelling" but falls through and finalizes a round that set a winner before throwing | ✗ | ✗ | ✗ | ✓ |
| The first raid calls `UpdateInternalAfterBattle` directly instead of driving the map tick | ✗ | ✗ | ✗ | ✓ |
| **New verified findings** | **6** | **7** | **4** | **7** |
| **Distinct verified findings (ground truth + new)** | **18** | **18** | **13** | **17** |

Not counted: a late advance reaching a destroyed event (Sol `xhigh`; the session lifetime predates the PR), clearing a newer battle's side links (Sol `high` and `xhigh`; no route for a new battle to attach during the loot screens), moving the destroyed-defeat menu out of the init barrier (Sol `xhigh`; that code predates the PR), and putting the native round call behind a new adapter only to test one catch (Sol `high`).

## Observations

**6.2 closes most of the gap to Opus.** Each 6.2 config found 9 to 12 of the 23 ground-truth findings, against 4 to 7 for 6.1. All four found the same eight core findings, including the request state in the wrong service and the "finished" flag set after a call that can throw. In 6.1 at most one config found the first, and none found the second. Scores followed the findings: the 6.2 means sit near Opus instead of near 9.

**The sibling-convention rule no longer hides the structural findings.** All four configs raised the request-state cohesion problem, and every Cohesion score was a 7. Sol `xhigh` still rejected the contested finalize-owner item: routing completion through the finalize handler would send encounter-close messages and break the result screens. Sol `high` raised it with a different cost Opus missed: simulation completion skips `BattleFinalizeHandler`'s troop-reserve cleanup, and abandoned missions keep reserve health (`BattleHostHandler.cs:1048`), so it can leak per event. No config raised the clearest Opus architecture finding, a won raid ending two different ways.

**Effort barely mattered.** `xhigh` and `high` averaged 10.5 ground-truth findings each, and 17.5 against 15.5 distinct verified findings. In 6.1, the buried-decision Testability item was the clearest effort gap: all three `xhigh` runs kept it and all three `high` runs dropped it. In 6.2 all four configs kept it.

**Test quality is still where Codex adds the most.** Seven of the eleven new findings are test gaps or test duplication. The denial-test gap came up in all four configs, and the missing start-reply assertion in three. Opus raised neither. Astra `xhigh` again praised the stand-in ordering test as a strength, the same mistake one 6.1 config made.

**Codex still skips the small items.** Opus's eight minor items found only one partial match in 6.2: Sol `high` asked for one staged-raid check in place of two. Two configs considered the duplicated rejection reply and dropped it as belonging to another category.

**Per model:**
- **Sol `high`** found the most ground-truth items (12), including the contested finalize-owner item with a concrete cost. It missed the theory-row duplication.
- **Sol `xhigh`** tied for the most distinct findings (18) and was the only config to find the lost-reply hole in the request flow.
- **Astra `xhigh`** found the most new test items (6), but praised the one test that checks a stand-in.
- **Astra `high`** was the most lenient (7.25) and found the fewest (13). It was the only 6.2 config to find the theory-row duplication, and it missed the cross-thread field.

## Recommendation

- **Reviewers:** Opus stays the poster. It's still the only model that found the won raid ending two different ways and the minor readability items. But a 6.2 Codex scorecard would no longer read as nearly done.
- **Second opinion:** replace the planned `gpt-6.1-sol` `xhigh` Test quality reviewer with `gpt-6.2-sol` at `high`, and widen it to Test quality and Error handling. Those two categories account for most of the verified Codex-only findings, and `high` matched `xhigh` here.
- **Codex alone** is now a usable fallback when Opus isn't available: Sol at `high` or `xhigh`, keeping the verify and audit steps.

## Limits

- One change, one run per config, so no variance estimate. Several one-config findings, such as the disconnect catch fall-through or the lost-reply hole, look like run-to-run variance.
- The ground truth still starts from the Opus review. 6.2 found 11 real issues outside it, so recall against it understates Codex.
- Time and token cost weren't recorded.
- Raw reviewer outputs and the claim checks were kept locally and aren't part of this report.
