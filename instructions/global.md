## Editing approach
- Never use scripts (sed, find/replace scripts, etc.) to make code changes. Edit each location explicitly.
- Reason about every change in isolation before making it.
- When a task requires modifying many places and looks like substantial work, spawn subagents to parallelize rather than doing it all in one pass.
