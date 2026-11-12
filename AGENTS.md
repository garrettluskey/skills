# skills

Agent skills that work in any harness that loads `SKILL.md` skills, including Claude Code and Codex. There is one copy of each skill, in a top-level folder named after it.

## Skills

| Skill | What it does |
|---|---|
| `quality-vibe-review` | Scores a GitHub PR 1-10 on architecture, readability, maintainability, coupling, cohesion, testability, test quality and error handling. One subagent per category reports a score and what would make it a 10. The findings are audited, then posted as a single `COMMENT` review with inline comments. It never approves or blocks. |

## Installing a skill

Link the skill folder into each agent's skills folder, so one copy serves every agent and edits land in this repo. On Windows, use a directory junction, which needs no admin rights:

```bat
mklink /J "%USERPROFILE%\.claude\skills\<skill>" "<clone>\<skill>"
mklink /J "%USERPROFILE%\.codex\skills\<skill>" "<clone>\<skill>"
```

On macOS or Linux, use `ln -s <clone>/<skill> ~/.claude/skills/` and the same for `~/.codex/skills/`. If a real folder is already at the link path, back it up and compare it with the repo before replacing it, so no local edits are lost.

## Writing and editing skills

Every skill must run unchanged in any harness. When you add or edit one:

- **Harness-specific details go in one table.** Name subagent tools, models and slash commands only in a per-harness table in `SKILL.md`, with an "anything else" row and a fallback for harnesses without subagents. Elsewhere, say "your subagent tool" or "your file-writing tool".
- **Paths are relative to the skill folder.** Refer to bundled files as `<skill>/references/...`. Tell the agent to resolve `<skill>` to an absolute path before handing paths to subagents, because subagents don't know where the skill was loaded from.
- **No machine-specific paths.** Use the temp and worktree folders the user's instructions name, falling back to the system temp folder. Never hard-code a drive or home path.
- **Self-contained.** Don't point at skills or files outside this repo. Copy the parts a skill needs into its `references/` folder.
- **Repo instructions.** Have reviewer subagents read `AGENTS.md` and `CLAUDE.md` in the repo they review, if those exist.
- **Codex metadata.** `agents/openai.yaml` holds Codex's display name and default prompt. Other harnesses ignore it.
- **Line endings are LF.** `.gitattributes` enforces this.

Update the skills table above when you add, rename or remove a skill.

## Requirements

- `gh`, authenticated with access to the repos being reviewed
- `git` and `python` on the path
