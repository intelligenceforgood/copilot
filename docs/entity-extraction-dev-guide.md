# Entity Extraction v2 — Developer Guide

This guide covers the three most common developer tasks in the extraction system.

---

## Adding a New Extraction Module

1. **Create the module** in `core/src/i4g/extraction/modules/<name>.py`:

   ```python
   from i4g.extraction.types import ModuleProtocol, ScoredEntity

   class MyModule:
       """Implements ModuleProtocol."""

       @property
       def name(self) -> str:
           return "my_module"

       @property
       def authority(self) -> dict[str, float]:
           return {
               "person": 0.6,
               "organization": 0.6,
           }

       def extract(self, text: str) -> list[ScoredEntity]:
           results = []
           # ... extraction logic ...
           results.append(ScoredEntity(
               entity_type="person",
               value=raw_name,
               canonical_value=raw_name.strip().title(),
               confidence=0.7,
               source_module=self.name,
               span=(start, end),
           ))
           return results
   ```

2. **Register the module** in `core/src/i4g/extraction/orchestrator.py`:

   ```python
   from i4g.extraction.modules.my_module import MyModule

   _MODULE_CLASSES: dict[str, type] = {
       # ... existing entries ...
       "my_module": MyModule,
   }
   ```

   If the module needs external dependencies (like `LLMModule` needs `llm_client`), add a
   condition in `_build_module()`.

3. **Enable it** in `core/config/settings.default.toml`:

   ```toml
   [extraction]
   enabled_modules = ["regex", "llm", "my_module"]
   ```

4. **Set authority values** thoughtfully. Authority controls how much the merge engine trusts this
   module relative to others. Guidelines:
   - **1.0** — the entity type is defined by a pattern this module owns (e.g., regex for wallets)
   - **0.7–0.9** — strong signal (LLM, trained model)
   - **0.4–0.6** — noisy signal that needs corroboration
   - **0.0** — module doesn't produce this type

5. **Write tests** in `core/tests/unit/extraction/modules/test_<name>.py`:
   - Verify `name` and `authority` properties
   - Test extraction on known inputs
   - Verify `ScoredEntity` output format

6. **Run the full suite** to confirm nothing breaks:

   ```bash
   conda run -n i4g pytest tests/unit/extraction/ -x
   ```

---

## Adding a New Entity Type

1. **Define the type** — Add the canonical type string to `CANONICAL_ENTITY_TYPES` in
   `core/src/i4g/utils/entity_types.py` and a human-readable label to `ENTITY_TYPE_LABELS`.

2. **Add a confidence gate** in `core/config/settings.default.toml`:

   ```toml
   [extraction]
   gate_my_type = 0.5
   ```

3. **Update module authority** — for each module that should extract this type, add an entry in
   its `authority` property. At minimum, one module must declare authority > 0.

4. **Add blocklist entries** if needed — edit `core/config/entity_blocklist.toml`:

   ```toml
   [my_type]
   values = ["Known False Positive"]
   ```

5. **Add golden labels** — update test bundles with expected entities of the new type:

   ```bash
   conda run -n i4g i4g entity-qa bundle add-case \
       --bundle regression-v1 \
       --text "Sample text containing the new type" \
       --label '{"my_type": ["expected_value"]}'
   ```

6. **Verify quality**:

   ```bash
   conda run -n i4g i4g entity-qa test orchestrator --bundle regression-v1
   conda run -n i4g i4g entity-qa score --bundle regression-v1
   ```

---

## Debugging a False Positive

When an analyst reports that the system extracted a wrong entity (e.g., "Revenue Service" classified
as a person):

1. **Reproduce** — run the orchestrator on the problematic text:

   ```bash
   conda run -n i4g python -c "
   from i4g.extraction import extract_entities
   result = extract_entities('...paste the text...', include_merge_log=True)
   for d in result.merge_log:
       print(f'{d.action.value}: {d.entity_type}/{d.value} conf={d.final_confidence} reason={d.reason} sources={d.sources}')
   "
   ```

2. **Read the merge log** — it tells you:
   - Which module(s) produced the entity (`sources`)
   - What confidence it got after merge (`final_confidence`)
   - Whether it was boosted by multi-source agreement

3. **Fix** — choose the right approach:

   | Situation                          | Fix                                           |
   | ---------------------------------- | --------------------------------------------- |
   | Known false positive that recurs   | Add to `config/entity_blocklist.toml`         |
   | Module assigns wrong entity type   | Fix the module's extraction logic             |
   | Confidence too high for noisy type | Lower the gate in `settings.default.toml`     |
   | Module shouldn't extract this type | Remove the type from the module's `authority` |

4. **Add a regression test** — capture the case in a test bundle:

   ```bash
   conda run -n i4g i4g entity-qa bundle add-case \
       --bundle bad-examples-v1 \
       --text "The problematic text" \
       --label '{"person": ["Correct Person Name"]}'
   ```

5. **Verify the fix**:

   ```bash
   conda run -n i4g i4g entity-qa test orchestrator --bundle bad-examples-v1
   ```

---

## Local Environment Setup

The `entity-qa test module` command bypasses confidence gates so you see raw module output.
The `test orchestrator` command applies the full merge pipeline including gates.

All modules must be explicitly wired with their dependencies to work in the CLI. The
`_build_extraction_deps()` helper in `core/src/i4g/cli/entity_qa/__init__.py` handles this
automatically based on which modules are requested and what's configured in settings.

### Enabling modules locally

The default local config enables three modules (`core/config/settings.local.toml`):

```toml
[extraction]
enabled_modules = ["regex", "llm", "heuristic"]
```

### Module-specific setup

| Module      | Dependency      | Local requirement                              |
| ----------- | --------------- | ---------------------------------------------- |
| `regex`     | None            | Works out of the box                           |
| `heuristic` | None            | Works out of the box                           |
| `llm`       | `llm_client`    | Ollama running locally (`provider = "ollama"`) |
| `ml_ner`    | `ml_predict_fn` | Deployed ML endpoint (dev/prod only)           |

#### LLM module

Requires Ollama with a model pulled (e.g., `ollama pull llama3.1`). Settings:

```toml
[llm]
chat_model = "llama3.1"
provider = "ollama"
```

The `ml_ner` module uses a fine-tuned BERT model served via the ML platform. It runs
automatically in dev/prod where the ML serving endpoint is deployed. It is not included in
the default local config — if `platform_base_url` is empty or unreachable, the module is
skipped gracefully.

### Testing

```bash
# Single module (no confidence gating — raw output)
conda run -n i4g i4g entity-qa test module heuristic
conda run -n i4g i4g entity-qa test module llm

# Full orchestrator (applies merge + confidence gates)
conda run -n i4g i4g entity-qa test orchestrator
```

---

## Quick Reference

| Task                        | Command                                                                     |
| --------------------------- | --------------------------------------------------------------------------- |
| Run unit tests              | `conda run -n i4g pytest tests/unit/extraction/ -x`                         |
| Test a single module        | `conda run -n i4g i4g entity-qa test module regex --bundle regression-v1`   |
| Test full pipeline          | `conda run -n i4g i4g entity-qa test orchestrator --bundle regression-v1`   |
| Score against golden labels | `conda run -n i4g i4g entity-qa score --bundle regression-v1`               |
| Compare modules             | `conda run -n i4g i4g entity-qa compare --bundle regression-v1`             |
| List blocklist entries      | `conda run -n i4g i4g entity-qa blocklist list`                             |
| Add to blocklist            | `conda run -n i4g i4g entity-qa blocklist add person "False Positive Name"` |
| Find false positives        | `conda run -n i4g i4g entity-qa analyze-fps`                                |
