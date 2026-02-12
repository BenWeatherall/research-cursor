# AI Context: Cursor Config (.cursor)

- **Version:** 1
- **Last Updated:** 2026-02-13
- **Tags:** cursor-config, component
- **Cross-References:** [AI_CONTEXT_REPOSITORY.md](AI_CONTEXT_REPOSITORY.md) | [AI_CONTEXT_QUICK_REFERENCE.md](AI_CONTEXT_QUICK_REFERENCE.md) | [AI_CONTEXT_PATTERNS.md](AI_CONTEXT_PATTERNS.md)

---

## Module Overview

| Path | Responsibility |
|------|----------------|
| **agents/** | Personas: researcher, architect, task-decomposer, implementer, debug-specialist, archivist, ai-context-writer. Used by workflows and commands. |
| **commands/** | Orchestrator commands: `feature-pipeline`, `bug-pipeline`, `update_context`, `commit`, `content_length`. Invoked from Composer. |
| **rules/** | Always-applied or glob-scoped rules: environment, development_practices, documentation, content_length. |
| **skills/** | ai-context-quick-reference, ai-context-repository, ai-context-patterns, ai-context-component. Used by ai-context-writer when generating context docs. |
| **workflows/** | Phase workflows (e.g. 1-research-feature, 2-plan-feature, …); invoked by pipeline commands, not directly. |
| **CONVENTIONS.md** | Path placeholders (`{features_dir}`, `{context_docs_dir}`, etc.) and folder roles. |
| **scratchpad.md** | Handoff between pipeline phases. |

---

## Public Interfaces

- **Commands:** User-facing entry points. Each command file defines purpose, prerequisites, and phases; paths use CONVENTIONS placeholders.
- **Path placeholders:** Resolved from CONVENTIONS.md; overrides in README or `.cursor/rules`. Defaults: `_features`, `_bugs`, `_archive`, `reports`, `docs/AI_CONTEXT`, `.cursor/scratchpad.md`.
- **Rules:** `.mdc` files; `alwaysApply` or `glob` for scope. No programmatic API; agents and workflows follow them by convention.

---

## Lifecycle / Entry Points

- **Start:** User runs a command in Cursor Composer (e.g. `/feature-pipeline`, `/update_context`). No persistent process.
- **Flow:** Command orchestrates phases; each phase may invoke workflows and agents; scratchpad passes context between phases.
- **Shutdown:** N/A (stateless invocation per run).

---

## Extension Points

| Extension | Location | Notes |
|-----------|----------|--------|
| New agent | `agents/<name>.md` | Define persona and when to use. |
| New command | `commands/<name>.md` | Define phases and delegate to workflows. |
| New workflow | `workflows/<name>.md` | Use CONVENTIONS placeholders; referenced by commands. |
| New rule | `rules/<name>.mdc` | Set `alwaysApply` or `glob`; document in README if needed. |
| New skill | `skills/<skill-name>/SKILL.md` | Used by ai-context-writer when generating a document type. |
| Path overrides | README or `.cursor/rules` | Override CONVENTIONS defaults. |

---

## Examples

**Resolving a path:**
`{context_docs_dir}` → default `docs/AI_CONTEXT` (CONVENTIONS.md). If README says `context_docs_dir: docs/context`, use that.

**Pipeline phase order (feature):**
Research (1-research-feature) → Plan (2-plan-feature, 3-task_list, 4-feature-tasks) → Implement (5-plan-task-feature, implement-task-feature / implement-feature) → Archive (6-archive-feature-summary).

**Updating context:**
`/update_context` runs phases: Quick Reference → Repository → Patterns → Per-component (one doc per discovered component); each phase uses the corresponding ai-context-* skill and writes into `docs/AI_CONTEXT/`.
