---
applyTo: "**/*.py"
---

# Ruff Pitfalls

Common ruff violations encountered in this codebase and how to handle them.

- **B008 — Typer `Option()`/`Argument()` in defaults.** Typer's documented pattern `def cmd(name: str = typer.Option(...))` triggers B008. Don't refactor — add `"src/*/cli/*.py" = ["B008"]` to `[tool.ruff.lint.per-file-ignores]` in pyproject.toml.
- **UP038 — `isinstance` with tuple of types.** `isinstance(x, (int, float))` → `isinstance(x, int | float)`. Use union syntax on Python 3.10+.
- **UP042 — `str, Enum` base classes.** `class Foo(str, Enum)` → `class Foo(StrEnum)` using `from enum import StrEnum` (Python 3.11+).
- **SIM105 — try/except/pass.** Replace with `contextlib.suppress(ExceptionType)`. Remember to `import contextlib`.
- **SIM102 — nested `if` statements.** Merge `if a: if b:` into `if a and b:` when the outer `if` has no `else`.
- **F841 — unused variable assignment.** Remove the assignment or prefix with `_` if the call has side effects (e.g., `_client = SomeClass()`).
- **UP037 — quoted type annotations.** Remove quotes from annotations when `from __future__ import annotations` is not needed (Python 3.11+ with union syntax).
- **E501 — line too long.** Break long `if` conditions into parenthesized multi-line format. Break long strings with implicit concatenation.
- **Redundant except clauses.** `except (SpecificError, Exception)` is redundant — `Exception` subsumes `SpecificError`. Catch the specific exception only.
