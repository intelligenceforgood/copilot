---
applyTo: "**/*.py"
---

# Python Standards

- **Type hints:** Required on all functions — parameters + return type.
- **Docstrings:** Google-style. Required on public functions, classes, modules. Private helpers (`_name`) get at least a one-liner.
- **Formatter:** Black at 120 chars. isort (profile = black).
- **Naming:** `snake_case` functions/variables/methods/files. `PascalCase` classes. `UPPER_SNAKE_CASE` constants. Singular nouns for constants and config keys.
- **Pydantic:** `snake_case` fields in Python. Use `CamelModel` with `alias_generator = to_camel` for JSON output — never write manual casing functions.
- **Imports:** Absolute preferred (`from i4g.store.review_store import ReviewStore`). Relative only within same sub-package. No wildcards.
- **Error handling:** Catch specific exceptions. Never bare `except:`. Log with `logger.exception()`.
- **Datetime:** Use `datetime.now(datetime.UTC)`, never `datetime.utcnow()`.
- **Logging:** `logger = logging.getLogger(__name__)` per module. Structured key-value pairs.
- **Settings:** Always via `get_settings()`, not hard-coded values. Override via env vars, not code.
- **Stores:** Use factories (e.g., `factories.py`) — do not instantiate stores directly.
- **TYPE_CHECKING guard:** For expensive runtime-only imports used solely as types (e.g., Playwright `Page`), guard with `if TYPE_CHECKING:`.
