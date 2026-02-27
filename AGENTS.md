# Ralph Agent Instructions

## Overview

Ralph is an autonomous AI agent loop (PowerShell) that spawns fresh Copilot CLI, Claude Code, or OpenAI Codex CLI instances in a loop, each completing one user story from a `prd.json` file until all stories pass. Each iteration has **no memory** of previous iterations — state persists only through git history, `prd.json`, and `progress.txt`.

## Architecture

```
ralph.ps1              # Main loop — spawns AI instances, checks for <promise>COMPLETE</promise>
CLAUDE.md              # Shared prompt injected into every AI iteration (all tools)
prd.json.example       # Reference format for the task list
init-ralph.ps1         # One-liner installer — copies ralph into scripts/ralph/ in a target project
skills/prd/SKILL.md    # Skill: generates PRD markdown from feature descriptions
skills/ralph/SKILL.md  # Skill: converts PRD markdown → prd.json for autonomous execution
.claude-plugin/        # Plugin manifest for Claude Code marketplace discovery
```

### How ralph.ps1 Works

1. Archives previous run if `prd.json` has a different `branchName` than `.last-branch`
2. Reads `CLAUDE.md` as the prompt and passes it to `copilot`, `claude`, or `codex` CLI
3. For Copilot CLI: streams plain text stdout line-by-line
4. For Claude Code: streams `--output-format stream-json` and parses `assistant` messages
5. For Codex CLI: streams `--json` output and parses `item.completed`/`item.started` events
6. Checks output for `<promise>COMPLETE</promise>` to know all stories are done
7. Loops up to `-MaxIterations` (default 10) with 2-second pauses between iterations

### Skill System

Skills live in `skills/<name>/SKILL.md` and are installed to `.agents/skills/` (as well as symlinks to `.claude/skills/` and `.github/skills/`). The `prd` skill generates markdown PRDs; the `ralph` skill converts them to `prd.json`.

## Commands

```powershell
# Run with Copilot CLI (default)
.\ralph.ps1 [-MaxIterations 10]

# Run with Claude Code
.\ralph.ps1 -Tool claude [-MaxIterations 10]

# Run with OpenAI Codex CLI
.\ralph.ps1 -Tool codex [-MaxIterations 10]

# Check story status
(Get-Content prd.json | ConvertFrom-Json).userStories | Select-Object id, title, passes
```

## Key Conventions

- **Commit format**: `feat: [Story ID] - [Story Title]` (e.g., `feat: US-001 - Add priority field`)
- **Branch naming**: `ralph/<feature-name-kebab-case>` — defined in `prd.json.branchName`
- **Progress logging**: Always append to `progress.txt`, never overwrite. Include a "Learnings for future iterations" section.
- **AGENTS.md updates**: After discovering reusable patterns, add them to the nearest `AGENTS.md` so future iterations (and humans) benefit
- **Story sizing**: Each story must be completable in one context window. If it takes more than 2-3 sentences to describe, split it.
- **Story ordering**: Dependencies first — schema → backend → UI → aggregation views
- **Stop signal**: When all stories pass, the AI outputs `<promise>COMPLETE</promise>` and the loop exits
- **One story per iteration**: Never implement multiple stories in a single AI invocation
