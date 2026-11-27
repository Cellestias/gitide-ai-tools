# gtide-ai

Teach AI coding agents to use [Git Tide](https://github.com/Cellestias/git-tide) (`gtide`) instead of raw Git.

| File | What |
|---|---|
| [skills/gtide/SKILL.md](skills/gtide/SKILL.md) | Skill: rules, task→command map, how to react to refusals. |
| [agents/gtide.md](agents/gtide.md) | Subagent: a Git worker that follows the skill and reports briefly. |

## Install with `npx skills`

```bash
npx skills add Cellestias/gitide-ai-tools                 # pick the agents interactively
npx skills add Cellestias/gitide-ai-tools --skill gtide   # just this skill
npx skills add Cellestias/gitide-ai-tools -g              # global, all projects
npx skills add Cellestias/gitide-ai-tools --list          # see what it contains, install nothing
```

This installs the **skill** only, for whichever agent you choose (Claude Code, Cursor, Codex, Copilot, ...). The CLI finds it at `skills/gtide/SKILL.md`. It needs the repo to be public on GitHub as `Cellestias/gitide-ai-tools`. A full URL or a local path (`npx skills add ./gtide-ai`) also works.

The subagent in `agents/` is Claude Code specific, so it is not part of `npx skills`; copy it by hand (below).

## Install by hand (Claude Code)

```bash
# skill, for every project
mkdir -p ~/.claude/skills && cp -R skills/gtide ~/.claude/skills/

# subagent, for every project
mkdir -p ~/.claude/agents && cp agents/gtide.md ~/.claude/agents/

# or per project: use <project>/.claude/skills and <project>/.claude/agents
```

Restart the session. The skill triggers on git tasks in repos with `.tide.yml`; the agent is available as `gtide`.

## Other tools

`skills/gtide/SKILL.md` is plain Markdown. For Cursor, Copilot, Codex, Gemini CLI and similar, paste its body (without the frontmatter) into the tool's rules file (`AGENTS.md`, `.cursor/rules`, `.github/copilot-instructions.md`).

## Keeping it right

The command map is written against `gtide` v1.5.0, plus the pull request review and compare rows (v1.11.0, Git 2.40 for compare), the `pick`, `restore`, `revert` and `fetch` rows (v1.12.0), and the `--json` section, which needs v1.10.0 for every command (v1.6.0 for the first read commands). When commands or flags change, re-check with `gtide <command> --help` and update the skill.
