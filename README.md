# skills

Agent skills that work in any harness that loads `SKILL.md` skills, including Claude Code and Codex. Each skill describes what it needs (subagents, a model, file paths) in terms any agent can map to its own tools, so there is one copy per skill.

## Skills

| Skill | What it does |
|---|---|
| `quality-vibe-review` | Scores a GitHub PR 1-10 on architecture, readability, maintainability, coupling, cohesion, testability, test quality and error handling. One subagent per category reports a score and what would make it a 10. The findings are audited, then posted as a single `COMMENT` review with inline comments. It never approves or blocks. |

## Install

Copy or link the skill folder into each agent's skills folder:

```bash
# Claude Code
cp -r quality-vibe-review ~/.claude/skills/

# Codex
cp -r quality-vibe-review ~/.codex/skills/
```

To edit one copy for every agent, link instead of copying. On Windows, a directory junction needs no admin rights:

```bat
mklink /J "%USERPROFILE%\.claude\skills\quality-vibe-review" "<clone>\quality-vibe-review"
mklink /J "%USERPROFILE%\.codex\skills\quality-vibe-review" "<clone>\quality-vibe-review"
```

On macOS or Linux, use `ln -s <clone>/quality-vibe-review ~/.claude/skills/` and the same for `~/.codex/skills/`.

Then run `/quality-vibe-review <PR number or URL>` in Claude Code, or ask Codex to use `$quality-vibe-review`. Add `--dry-run` to build and audit the review without posting it.

## Requirements

- `gh`, authenticated with access to the repo you're reviewing
- `git` and `python` on the path
- Clones, worktrees and review files go in the temp and worktree folders your agent instructions name, or the system temp folder if they don't name one.
