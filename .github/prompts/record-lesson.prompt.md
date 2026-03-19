---
agent: agent
description: "Save a lesson, pattern, or pitfall to repo memory for future sessions"
---

# Record Lesson

Capture something learned during this session so future sessions benefit from it.

## Steps

1. **Identify the lesson.** Ask the user what they want to record, or infer from the current conversation context.

2. **Categorize:**
   - **Coding pitfall** — a mistake or anti-pattern to avoid
   - **Architecture pattern** — an approach that works well in this codebase
   - **Workflow tip** — a useful command, shortcut, or process improvement
   - **Environment/config** — a configuration insight or env var behavior

3. **Write to repo memory.** Add to `/memories/repo/lessons-learned.md` under the appropriate section. Format as a concise bullet point:
   - What the lesson is (one line)
   - Why it matters or what goes wrong without it (one line)
   - Example if helpful (code snippet or command)

4. **Check for promotion.** If the same category now has 3+ similar lessons, suggest promoting the pattern to a `.instructions.md` file in `copilot/.github/standards/` so Copilot applies it automatically for matching files.

5. **Confirm.** Tell the user what was recorded and where.
