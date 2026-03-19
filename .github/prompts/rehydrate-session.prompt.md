---
mode: agent
description: "Start your day — check status, load context, identify what to work on"
---

# Session Rehydration

Kick off a new working session by gathering context.

## Steps

1. **Git status across repos.** Run `git status -sb` in core/, ui/, ssi/, and any other repos with recent activity. Report the current branch and any uncommitted changes.

2. **Recent commits.** Run `git log --oneline -5` in repos with changes to see what happened last session.

3. **Change log.** Read the last 30 lines of `planning/change_log.md` for recent decisions.

4. **Active tasks.** Search `planning/tasks/` for files with unchecked items (`- [ ]`). Identify the current sprint or work stream.

5. **Repo memory.** Read `/memories/repo/` files to recall lessons learned and workflow patterns from prior sessions.

6. **Architecture context.** Skim `core/.github/architecture-cheatsheet.instructions.md` for the system overview.

7. **Status report.** Present:
   - Current branch per repo
   - Uncommitted changes
   - Active task/sprint
   - Key context from change log
   - Relevant items from repo memory

8. **Suggest next action.** Based on the status, recommend what to work on and which routine to use (plan-work, fix-bug, pre-merge-review, etc.).
