# Customization Guide

How to extend this system with new routines, standards, and instructions.

## Architecture

```
copilot/.github/
├── copilot-instructions.md      # Always loaded — repo description
├── prompts/                     # Routines (invoked from chat picker)
│   └── *.prompt.md
├── standards/                   # Auto-loaded by file type
│   └── *.instructions.md
└── shared/                      # Reference docs (cross-referenced)
    └── *.instructions.md
```

### How VS Code Loads These

In a multi-root workspace:

- **`copilot-instructions.md`** — loaded into every conversation for each workspace root
- **`prompts/*.prompt.md`** — appear in the chat prompt picker from any workspace root
- **`*.instructions.md` with `applyTo`** — loaded when editing files matching the glob pattern, from any root
- **`*.instructions.md` without `applyTo`** — loaded on reference (when other instructions mention them)

## Adding a New Routine

1. Create `copilot/.github/prompts/<name>.prompt.md`
2. Add YAML frontmatter:
   ```yaml
   ---
   mode: agent
   description: "One-line description shown in the prompt picker"
   ---
   ```
3. Write numbered steps in markdown. Focus on:
   - **Decision points**, not single commands
   - **What to check** before acting
   - **What to report** after each step
4. Update `docs/routine-catalog.md` with the new routine
5. Commit to the `copilot` repo

### Good vs. Bad Routines

**Good routine (decision-making workflow):**

- Pre-merge review — requires judgment, multiple steps, multi-repo coordination
- Plan work — requires understanding scope, architecture, dependencies
- Fix bug — requires investigation, root-cause analysis, regression testing

**Bad routine (single command with no decisions):**

- Bootstrap sandbox — just a command to run
- Install dependencies — `pip install -e .`
- Start dev server — `uvicorn ...`

Commands belong in docs or READMEs, not in routines.

## Adding a New Standard

1. Create `copilot/.github/standards/<name>.instructions.md`
2. Add YAML frontmatter with the file glob:
   ```yaml
   ---
   applyTo: "**/*.sql"
   ---
   ```
3. Write concise bullet-point rules. Keep it under 20 lines — this is injected into every conversation involving matching files.
4. Commit

### Example: SQL Standard

```yaml
---
applyTo: "**/*.sql,**/alembic/versions/*.py"
---
# SQL / Migration Standards
- Table and column names: `snake_case`, singular
- Use SQLAlchemy ORM/Core expressions; raw SQL with parameterized queries only
- Alembic migrations: descriptive message, idempotent guards
- Never modify a migration already applied to a shared environment
```

## Adding Shared Reference Docs

These are longer documents that other instructions cross-reference. They don't auto-load — they're pulled in when a routine or standard mentions them.

1. Create `copilot/.github/shared/<name>.instructions.md`
2. No `applyTo` needed
3. Write comprehensive reference content
4. Cross-reference from routines or standards

## The Learning System

### How Repo Memory Works

- **Repo memory** (`/memories/repo/`) persists across all Copilot sessions in this workspace
- It's stored internally by VS Code, not as files on disk
- Any Copilot session can read and write to it
- Two key files:
  - `lessons-learned.md` — pitfalls, patterns, tips
  - `workflow-patterns.md` — preferences and conventions

### The Promotion Cycle

```
Work → Discover lesson → Record in repo memory
                              ↓
              3+ similar lessons accumulate
                              ↓
              Promote to .instructions.md
                              ↓
              Auto-loaded for all matching files
```

When the `record-lesson` routine detects 3+ similar lessons in a category, it suggests creating a `.instructions.md` file to make the pattern permanent and automatic.

## Keeping Product Repos Thin

Each product repo keeps only repo-specific context in `.github/copilot-instructions.md`:

- Conda environment name
- Build/test commands
- Architecture unique to that repo
- Repo-specific pre-commit procedure

Everything shared lives here in the `copilot` repo. When updating shared standards:

1. Edit here, not in product repos
2. Product repos should cross-reference, not duplicate

### Template for New Repos

See `copilot/.github/repo-templates/copilot-instructions.template.md` for the template to use when adding a new repo to the workspace.
