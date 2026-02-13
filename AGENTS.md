# Ralph Agent Instructions

## Overview

Ralph is an autonomous AI agent loop that runs AI coding tools (Copilot CLI or Claude Code) repeatedly until all PRD items are complete. Each iteration is a fresh instance with clean context.

## Commands

```powershell
# Run Ralph-Windows with Copilot CLI (default)
.\ralph.ps1 [-MaxIterations 10]

# Run Ralph-Windows with Claude Code
.\ralph.ps1 -Tool claude [-MaxIterations 10]
```

## Key Files

- `ralph.ps1` - The PowerShell loop that spawns fresh AI instances (supports `-Tool copilot` or `-Tool claude`)
- `CLAUDE.md` - Shared prompt template used by both tools
- `prd.json.example` - Example PRD format

## Patterns

- Each iteration spawns a fresh AI instance (Copilot CLI or Claude Code) with clean context
- Memory persists via git history, `progress.txt`, and `prd.json`
- Stories should be small enough to complete in one context window
- Always update AGENTS.md with discovered patterns for future iterations
