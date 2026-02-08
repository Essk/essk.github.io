---
title: Ask claude about a currently running session
date: 2026-02-08
layout: default
---

Claude Code writes session transcripts to `~/.claude/projects/` as `.jsonl` files in real-time. You can open a second `claude` session in another terminal and have it read the live transcript from your working session.

## Finding active sessions

```bash
ls -lt ~/.claude/projects/-home-sarah-repos-project-name/*.jsonl | head -5
```

The most recently modified file is your active session. Each line is a JSON object with a `type` field: `user`, `assistant`, `tool_result`, `file-history-snapshot`, etc.

## Reading from another session

From a meta-session, ask Claude to read the `.jsonl` file:

```
Read the last 50 lines of ~/.claude/projects/.../[session-id].jsonl and summarize what's happening
```

The two sessions are fully isolated — the meta session can't send instructions to the working session or modify its behavior. It's read-only observation.

## Use cases

- Ask questions about approach or workflow without polluting the working session
- Review what's been done so far
- Debug stuck sessions
- Develop better prompting patterns by observing what works

## Session memory summaries

Claude also writes `session-memory/summary.md` files for some sessions (at session end). These are easier to read than raw JSONL but only exist for completed sessions.
