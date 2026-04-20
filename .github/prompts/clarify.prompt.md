---
agent: agent
description: "[Executor] Bounce a blocking ambiguity back to the Planner and halt"
---

# Clarify

You (the Executor) are blocked on an ambiguity or contradiction in the Task Manifest, or you've discovered state that the manifest did not anticipate. Do not guess. Produce a short structured question and stop work.

## Steps

1. **Summarize what you were doing** — which manifest, which step, what you had completed so far.

2. **State the block precisely.** Use one of these categories and keep the body to a few lines:
   - **Missing info** — the manifest does not specify X.
   - **Contradiction** — the manifest says X in one place and Y in another.
   - **Out-of-scope need** — the manifest's `<files>` list does not include a file that must change to complete the task.
   - **Unexpected state** — the code or data does not match what the manifest assumes.
   - **Failing verification** — an acceptance command fails in a way that implies the plan is wrong, not the implementation.

3. **Produce the handoff block.** Format:

   ```markdown
   ## Clarify Request

   <clarify>
   **Manifest:** <path, version>
   **Step:** <number + title>
   **Category:** <one of the five above>
   **Block:** <1–3 sentences>
   **Options considered:** <A, B, C — what would each imply?>
   **Recommendation:** <your best guess, so the Planner can approve or reject>
   **Files changed so far:** <list, or "none committed">
   </clarify>
   ```

4. **Do not keep working.** Do not commit speculative changes. Leave the workspace in a clean state (either fully revert the in-progress step, or leave it on a scratch branch and say so in the report).

5. **Tell the user to switch to the Planner model** and feed this block back in; the Planner will update the manifest (bumping its version) or send a targeted instruction.
