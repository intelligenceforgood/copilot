---
applyTo: "*"
---

# Merge & Commit Discipline

Hard-won rules for the commit-and-push phase of the merge routine. These apply
to **every** file edit made during or after a pre-merge review — not just the
main feature changes.

## 1. Format before staging (UI repo)

When editing JSON, TS, or TSX files via the editor or edit tool, the output
often diverges from Prettier's canonical formatting (array wrapping, trailing
newlines, indentation). Always run the formatter **before** `git add`:

```bash
cd ui/ && pnpm format        # Prettier writes canonical output
git add -A                   # now safe to stage
git commit -m "..."
```

Skipping this causes the committed content to differ from Prettier — the next
`pnpm format` run re-formats the file and leaves a dirty working tree that
looks like an uncommitted change.

## 2. Build before committing config changes

Any edit to `tsconfig.json`, `package.json`, build config, or settings files
must pass the full build **before** `git commit`:

```bash
# UI
cd ui/ && make build

# Python
cd core/ && conda run -n i4g pre-commit run --all-files
```

This catches breakage (e.g., removing `baseUrl` breaking path aliases) before
it reaches `origin/main`.

## 3. Post-push cleanliness sweep

After pushing, run `git status -sb` in **every** workspace repo — not just the
repos you changed. Formatters, editors, and hooks can silently modify files
after a commit. The merge is not done until all repos show a clean working tree.

```bash
for repo in copilot core ssi ui infra ml docs planning mobile; do
  dirty=$(cd /path/to/i4g/$repo && git status --porcelain 2>/dev/null)
  [[ -n "$dirty" ]] && echo "DIRTY: $repo" && echo "$dirty"
done
```

If any repo is dirty, diagnose whether it's a formatter artifact (commit it) or
an unintended change (revert it) before declaring the merge complete.
