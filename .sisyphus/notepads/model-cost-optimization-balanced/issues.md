## 2026-02-19 - Task 10 smoke checks failed agent resolution

- `opencode run "Reply with EXACTLY: OK" --agent explore` exited 0 and printed `OK`, but warned: `agent "explore" is a subagent, not a primary agent. Falling back to default agent`.
- `opencode run "Reply with EXACTLY: OK" --agent sisyphus` exited 0 and printed `OK`, but warned: `agent "sisyphus" not found. Falling back to default agent`.
- No `ProviderModelNotFoundError` occurred, but both checks failed lane targeting because both commands fell back to default agent.
