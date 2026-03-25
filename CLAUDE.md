# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Plugin marketplace for Claude Code. Hosts reusable agents, commands, and skills as `.md` files with YAML frontmatter. No build system — content is pure markdown.

## Architecture

```
.claude-plugin/marketplace.json    # root manifest listing all plugins
plugins/{plugin-name}/
  .claude-plugin/plugin.json       # plugin metadata (name, description, version)
  agents/{name}.md                 # subagent definitions (frontmatter: name, description, model, tools)
  commands/{name}.md               # orchestration commands (frontmatter: description, argument-hint)
  skills/{name}/SKILL.md           # skill workflows (frontmatter: name, description)
  docs/{name}.md                   # shared reference docs included by agents/skills
```

### Current Plugin: `tsc-react-dev-plugin`

TypeScript/React development toolkit with:

- **agents/coder.md** — code implementation agent (model: sonnet, no tools)
- **agents/code-reviewer.md** — review agent (model: haiku, tools: Glob/Grep/Read/WebFetch)
- **commands/plan-and-execute.md** — multi-phase orchestrator: investigate → plan → execute (coder→reviewer loops) → final review (3 parallel reviewers) → verify
- **skills/unit-test/SKILL.md** — test orchestrator: investigate → plan → execute (coder→reviewer loops) → verify coverage
- **skills/unit-test/references/testing-guidance.md** — Kent C. Dodds testing philosophy, patterns, and checklists (referenced by skill and agents)

### Frontmatter Format

Agents use:

```yaml
---
name: agent-name
description: "When to use this agent..."
model: sonnet | haiku
tools: Glob, Grep, Read, WebFetch # optional
---
```

Commands use:

```yaml
---
description: What the command does
argument-hint: <what the user passes>
---
```

Skills use:

```yaml
---
name: skill-name
description: "Trigger phrases and use cases..."
---
```

## Key Patterns

- Agents reference each other by name via Task tool `subagent_type` parameter
- Commands orchestrate by dispatching coder then code-reviewer in loops
- Skills define multi-phase workflows with user checkpoints between phases
- Skills can bundle reference files (e.g., `skills/unit-test/references/testing-guidance.md`) for subagents to read
- The `plan-and-execute` command never writes code — all work dispatched to subagents
- Review loops cap at 3 iterations before escalating to user

## Adding a New Plugin

1. Create `plugins/{name}/` with the directory structure above
2. Add `plugin.json` with name, description, version
3. Register in root `marketplace.json` plugins array
