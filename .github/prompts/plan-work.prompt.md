---
mode: agent
description: "Break a feature or task into actionable steps and start working"
---

# Plan Work

Take a feature request, task description, or user story and break it into implementable steps.

## Steps

1. **Clarify scope.** Read the task description (from planning/, a PR, or user input). If it's ambiguous, list assumptions and confirm with the user.

2. **Identify affected repos.** Determine which repos need changes (core/, ui/, ssi/, infra/, etc.) and what kind of changes (API, UI, database, infrastructure).

3. **Check architecture.** Read `core/.github/architecture-cheatsheet.instructions.md` for relevant patterns, especially:
   - Request routing (UI → API proxy → FastAPI)
   - Store/factory patterns
   - Worker/job patterns for background tasks

4. **Break into steps.** Create a numbered task list:
   - Order by dependency (database first, then API, then UI)
   - Each step should be independently testable
   - Flag steps that require manual actions (migrations, deploys)

5. **Identify risks.** Note:
   - Breaking changes to existing APIs or schemas
   - New environment variables (need docs + tests)
   - Database migrations (need careful sequencing)

6. **Track with todos.** Create a todo list to track progress through the steps.

7. **Start working.** Begin the first step, or ask the user which step to start with.
