---
title: Setting up GitHub Pages with Claude Code
date: 2026-02-07
layout: default
---

# Setting up GitHub Pages with Claude Code

Created a minimal GitHub Pages site at `essk.github.io` for publishing notes about working with AI agents.

## Setup

- **Repository:** `essk.github.io` (user site, publishes from `main` branch)
- **Theme:** `pages-themes/minimal@v0.2.0` (clean, readable)
- **Workflow:** Write markdown in `_posts/`, push, auto-publishes via Jekyll

## Files created

- `_config.yml` — site title, description, theme, permalink structure
- `index.md` — homepage with `{% for post in site.posts %}` loop to auto-list entries
- `_posts/2026-02-07-first-post.md` — sample post

## Publishing workflow

To publish a new note:
1. Create `_posts/YYYY-MM-DD-title.md` with front matter (`title`, `date`, `layout: default`)
2. Write the content
3. `git add`, `git commit`, `git push`
4. Live at `https://essk.github.io/YYYY/MM/DD/title` within ~1 minute

## Custom skill

Created `/publish` skill at `~/.claude/skills/publish/SKILL.md`:
- Summarizes conversation findings into a post
- Stops for review/editing
- Commits and pushes on confirmation
- Uses Sonnet model for cost efficiency

The skill handles the whole workflow — just run `/publish <title>` at the end of a session.

````markdown
---
name: publish
description: Publish a post to essk.github.io
disable-model-invocation: true
argument-hint: [title]
model: claude-sonnet-4-5-20250929
allowed-tools: Write, Read, Edit, Bash(git *), Bash(cd *)
---

Publish a new post to essk.github.io. The title is: $ARGUMENTS

## Steps

1. **Write the post.** Create a file at `/home/sarah/repos/essk.github.io/_posts/YYYY-MM-DD-title.md` where the date is today and the title slug is derived from $ARGUMENTS. Use this front matter:

   ```
   ---
title: <title>
date: YYYY-MM-DD
layout: default
   ---
   ```

   For the body: summarise the key findings, decisions, or notes from this conversation. Write in a concise, direct style. Use markdown headings and lists where they help readability. Keep it short — these are working notes, not essays.

2. **Ask the user to review.** Show them the full file path and tell them to review and edit the post if needed. **Stop here and wait** — do not proceed until they confirm.

3. **Publish.** Once the user confirms:
   ```
cd /home/sarah/repos/essk.github.io
git add _posts/
git commit -m "Add: <title>"
git push
   ```

4. **Confirm.** Tell the user the post will be live at `https://essk.github.io/YYYY/MM/DD/title-slug` within about a minute.
````