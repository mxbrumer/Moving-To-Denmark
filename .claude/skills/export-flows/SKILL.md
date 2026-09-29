---
name: export-flows
description: Export LangFlow flows from the running container into langflow/flows/*.json and commit any changes. Use after editing flows in the LangFlow UI or when the user says "export flows".
---

# Export LangFlow flows

1. Make sure the stack is up (`scripts/dev.ps1 up`). If `scripts/export-flows.ps1` does not exist yet (it is created in plan 08), tell the user and stop.
2. Run `powershell -File scripts/export-flows.ps1`.
3. Run `git diff --stat langflow/flows` and check the changed JSON contains no secrets (API keys, passwords) or personal data.
4. Commit changed flow files with `feat(flows): update <flow names>` (or `fix(flows):` for corrections).
5. Report which flows changed.
