---
applyTo: "**/settings*.toml,**/settings*.py,**/.env*,**/pyproject.toml"
---

# Settings & Configuration Standards

- **Settings access:** Always via `get_settings()` in code. Never hard-code paths, URLs, or credentials.
- **Environment overrides:** Use env vars with project prefix (e.g., `I4G_*`). Double underscores for nesting (`I4G_LLM__PROVIDER`).
- **TOML config:** `snake_case` keys. Section headers in `[brackets]`.
- **Env profiles:** `local` → SQLite/mock. `dev`/`prod` → PostgreSQL/cloud services.
- **Path resolution:** Use framework's path resolver — pass project-relative references, not manual `Path` math.
- **Env var contract:** When adding/changing env vars:
  1. Add test coverage under `tests/unit/settings/`
  2. Update config documentation/manifests
  3. Run local smoke test before cloud deployment
- **.env files:** `UPPER_SNAKE_CASE` keys. Non-public secrets go in `.env.local` (gitignored).
