# skills

Agent skills for Claude Code and Codex. Each skill has a Claude version under `claude/` and a Codex version under `codex/`. The two differ only in how they start subagents and where they look for their own files.

## Skills

| Skill | What it does |
|---|---|
| `quality-vibe-review` | Scores a GitHub PR 1-10 on architecture, readability, maintainability, coupling, cohesion, testability, test quality and error handling. One subagent per category reports a score and what would make it a 10. The findings are audited, then posted as a single `COMMENT` review with inline comments. It never approves or blocks. |

## Install

```bash
# Claude Code
cp -r claude/quality-vibe-review ~/.claude/skills/

# Codex
cp -r codex/quality-vibe-review ~/.codex/skills/
```

Then run `/quality-vibe-review <PR number or URL>` (Claude) or ask Codex to use `$quality-vibe-review`. Add `--dry-run` to build and audit the review without posting it.

## Requirements

- `gh`, authenticated with access to the repo you're reviewing
- `git` and `python` on the path
- The skill keeps clones under `G:/.tmp/pr-reviews` and worktrees under `G:/.worktrees`. Change those paths in `SKILL.md` to suit your machine.
