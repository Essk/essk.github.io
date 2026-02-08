---
title: Project-specific agents for Claude Code
date: 2026-02-08
layout: default
---

Claude Code can spawn sub-agents via the Task tool to handle work in parallel or in the background. By default these are generic — they know nothing about your project's conventions. You can fix this by defining project-specific agents in `.claude/agents/`.

## Defining an agent

Create a markdown file at `.claude/agents/<agent-name>.md`. The file should describe the agent's role and encode the project conventions it needs to follow.

```markdown
# api-impl

Server-side implementation agent.

## Scope

Server actions, persistence, API routes.

## Conventions

- All data access goes through the persistence adapter
- Server actions return `{ success: boolean, message?, errors? }`
- Validate with Zod, sanitize inputs, generate UUIDs
- Call `revalidatePath()` after mutations

## Reference files

- Pattern reference: `src/app/settings/measurements/actions.ts`
- Persistence: `src/app/api/persistence-adapter.ts`

## Code style

- TypeScript strict, 4 spaces, 80 char lines
- Named exports, interfaces for shapes
```

The key sections:
- **Scope** — what this agent is responsible for
- **Conventions** — project-specific patterns it must follow
- **Reference files** — where to look for examples
- **Code style** — formatting and language conventions

## Why bother?

Without pre-defined agents, every time you spawn a sub-agent you need to repeat your project's conventions in the prompt. This gets verbose and inconsistent. With agent definitions:

- **Conventions are baked in** — the agent knows your patterns without being told each time
- **Consistency** — every spawn of `api-impl` behaves the same way
- **Less prompt bloat** — you pass the task, not the task plus a wall of context
- **Independent tuning** — adjust one agent's behavior without affecting others

## Typical agent roles

The right roles depend on your project, but common splits:

| Agent | Responsibility |
|-------|---------------|
| `api-impl` | Server-side logic, data access, validation |
| `ui-impl` | Components, styling, client state |
| `tester` | Writing and running tests |

For a monorepo you might split by package. For a backend-only project you might split by domain (auth, billing, etc).

## Using agents as sub-agents

Spawn a single agent for focused work:

```
Task tool:
  subagent_type: "api-impl"
  prompt: "Implement the new endpoint per this plan: ..."
  run_in_background: true
```

The agent inherits your permission settings and picks up its conventions from its definition file.

## Using agents as a team

For larger features, create a team and assign agents to different workstreams:

1. **Create a team** — `TeamCreate` with a descriptive name
2. **Create tasks** — `TaskCreate` for each workstream with acceptance criteria
3. **Spawn teammates** — one Task per agent role, each joining the team
4. **Set dependencies** — API and UI can run in parallel; testing depends on both

```
api-impl tasks ──┐
                  ├──→ tester tasks
ui-impl tasks  ──┘
```

The team lead (your session) coordinates: monitoring progress, unblocking issues, and running a final check when everything's done.

## Automating the workflow with a skill

You can wrap the whole pattern in a custom skill (`.claude/skills/<name>/SKILL.md`) that:

1. Reads an approved plan
2. Creates a feature branch
3. Assesses complexity — single agent or team?
4. Spawns the appropriate agents with the plan
5. Reports back

This turns the workflow into: describe task → approve plan → `/implement` → agents do the work.

### Complexity heuristic

A simple decision rule:

- **Single agent** if: ≤ 3 files, single concern, no parallel workstreams
- **Agent team** if: multiple concerns, clear parallel tracks, >3 files

Tune this based on experience. Start with the single-agent path and graduate to teams when you find yourself waiting on sequential work that could parallelize.

## Tips from practice

- **Encode the gotchas.** The most valuable thing in an agent definition isn't the obvious conventions — it's the stuff that's easy to get wrong. Quirky mocking patterns, disconnect requirements, naming inconsistencies.
- **Keep definitions focused.** An agent that tries to do everything is just a generic agent with extra words. Narrow scope produces better results.
- **Reference files over rules.** "Follow the pattern in `src/settings/measurements/`" is often more effective than paragraphs of instructions.
- **Iterate.** Run the workflow a few times, note where agents go wrong, and update their definitions. These are living documents.
