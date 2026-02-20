## 2026-02-19 - Task 4 Variant Keep/Drop Policy

- Adopt balanced variant policy: keep `ultrabrain` at `xhigh`; keep `prometheus`, `metis`, and `momus` at `high`.
- Downshift `deep` and `artistry` by removing explicit `high` variant assignments (inherit default).
- Keep/downshift sets are non-overlapping and aligned with cost optimization guardrails.

## 2026-02-19 - Task 2 Balanced Routing Matrix

- Keep premium lanes on `openai/gpt-5.3-codex`: agents `sisyphus`, `hephaestus`, `oracle`, `prometheus`, `metis`, `momus`; categories `ultrabrain`, `deep`, `artistry`, `unspecified-high`.
- Route nano lanes to `opencode/gpt-5-nano`: agents `explore`, `librarian`; categories `quick`, `visual-engineering`.
- Route glm-free lanes to `opencode/glm-5-free`: agent `multimodal-looker`; categories `writing`, `unspecified-low`; keep `atlas` on `opencode/glm-5-free`.
- Evidence captured in `.sisyphus/evidence/task-2-matrix.txt`.
