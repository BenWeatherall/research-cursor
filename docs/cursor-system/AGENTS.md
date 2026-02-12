# Agents

Agents are personas defined in `.cursor/agents/`. They are invoked by **orchestrator commands** (e.g. feature-pipeline, bug-pipeline, update_context) via Cursor’s `/name` syntax (e.g. `/researcher`, `/architect`). Each agent runs in an **isolated context**; the **scratchpad** (`.cursor/scratchpad.md`) is used for handoff between phases.

---

## 1. Agent Overview

| Agent | Description | Primary use |
|-------|-------------|-------------|
| **researcher** | Technical research; solution comparison; external/library research | Feature research (1-research-feature); bug research (2-research-bug). |
| **architect** | System design; structural planning; dependency mapping; implementation plans | Feature/bug master plan; per-task plans (5-plan-task-feature, 5-plan-task-bug). |
| **task-decomposer** | Task breakdown; dependency analysis | Feature tasks (3-task_list + 4-feature-tasks); bug tasks (4-bug-fix-tasks). |
| **implementer** | TDD implementation; feature and fix implementation | Single-task or full feature/fix (implement-task-feature, implement-task-fix, implement-feature, implement-fix). |
| **debug-specialist** | Root cause analysis; reproduction; evidence gathering | Bug investigation (1-investigate-bug). |
| **archivist** | Synthesise request + plans + tasks into one summary | Feature archive (6-archive-feature-summary); bug archive (6-archive-bug-summary). |
| **ai-context-writer** | LLM-oriented context docs in `{context_docs_dir}` | Invoked by update_context; uses skills per document type. |

---

## 2. Agent Details

### 2.1 Researcher (`/researcher`)

- **Role**: Senior Software Architect; technical research from internal docs and external sources. No business/time/cost focus.
- **Resources**: `@docs/AI_CONTEXT/`, web search for libraries/errors, `development_practices.mdc`, `content_length.mdc`.
- **Modes**:
  - **Feature**: Run `1-research-feature.md`; read feature file; produce planning docs in `{features_dir}/{feature_name}/planning/` (impacted-systems, new-systems, possible-solutions, selected-solution).
  - **Bug**: Run `2-research-bug.md`; read investigation docs from `{bugs_dir}/{bug_name}/investigation/`; produce research docs in `{bugs_dir}/{bug_name}/research/`.
- **Output**: Summary to scratchpad (feature/bug name, output dir, key findings, selected solution or fixes).

### 2.2 Architect (`/architect`)

- **Role**: Senior Software Architect; scalability, structure, interfaces, best practices. No business/time/cost focus.
- **Resources**: `@docs/AI_CONTEXT/`, `development_practices.mdc`, `content_length.mdc`.
- **Modes**:
  - **Feature plan**: `2-plan-feature.md`; read planning docs; write implementation plan in `{features_dir}/{feature_name}/plans/master/`.
  - **Bug fix plan**: `3-plan-bug-fix.md`; read investigation + research; write fix plan in `{bugs_dir}/{bug_name}/plans/master/`.
  - **Task plan (feature)**: `5-plan-task-feature.md`; feature name + task file path → plan at `{features_dir}/{feature_name}/plans/tasks/{task_file}.md`.
  - **Task plan (bug)**: `5-plan-task-bug.md`; bug name + task file path → plan at `{bugs_dir}/{bug_name}/plans/tasks/{task_file}.md`.
- **Output**: Summary to scratchpad (plan location, key decisions, prerequisites for next step).

### 2.3 Task Decomposer (`/task-decomposer`)

- **Role**: Senior Software Architect; task decomposition and dependency analysis. No business/time/cost focus.
- **Resources**: `@docs/AI_CONTEXT/`, `{features_dir}/_template.md` (task doc structure), `development_practices.mdc`, `content_length.mdc`.
- **Modes**:
  - **Feature tasks**: `3-task_list.md` + `4-feature-tasks.md`; read plan from `plans/master/`; create task docs in `{features_dir}/{feature_name}/tasks/`.
  - **Bug fix tasks**: `4-bug-fix-tasks.md`; read plan from `plans/master/`; create task docs in `{bugs_dir}/{bug_name}/tasks/`.
- **Output**: Summary to scratchpad (tasks dir, task count, ordered list of task file names, execution order, blocking dependencies).

### 2.4 Implementer (`/implementer`)

- **Role**: TDD and code implementation; follows approved plans and component patterns.
- **Regression rules**: Failing tests (other than explicitly expected failures) are regressions and must be fixed. Full test suite must pass before task/phase complete.
- **Resources**: `@docs/AI_CONTEXT/`, `development_practices.mdc`, `environment.mdc`, `content_length.mdc`, scratchpad.
- **Modes**:
  - **Full feature**: `implement-feature.md` (context, plan, implement, completion; user approval before implementation).
  - **Full fix**: `implement-fix.md` (prioritise regression tests; user approval before implementation).
  - **Single task (feature)**: `implement-task-feature.md`; one task only; plan pre-created by architect.
  - **Single task (bug)**: `implement-task-fix.md`; one task only; plan pre-created by architect.
- **Output**: Checklist complete; update CHANGELOG, `{context_docs_dir}`; archive per workflow; report completion.

### 2.5 Debug Specialist (`/debug-specialist`)

- **Role**: Senior Software Architect; debugging and root cause analysis. No business/time/cost focus.
- **Resources**: `@docs/AI_CONTEXT/`, `development_practices.mdc`, `content_length.mdc`.
- **Workflow**: `1-investigate-bug.md` — understand bug, gather facts, create under `{bugs_dir}/{bug_name}/investigation/`: observed-behavior, reproduction-steps, logs-and-traces, affected-components, root-cause-hypothesis.
- **Output**: Summary to scratchpad (bug name, investigation dir, root cause hypothesis, affected components).

### 2.6 Archivist (`/archivist`)

- **Role**: Synthesise pipeline artifacts into one summary; no code or behaviour changes.
- **Resources**: Path placeholders (CONVENTIONS.md), `documentation.mdc`, `content_length.mdc`.
- **Invocation**:
  - **Feature (Phase 6)**: Feature name, `{features_dir}`, `{archive_dir}`. Run `6-archive-feature-summary.md`; read request + all docs under `{features_dir}/{feature_name}/`; write `{archive_dir}/{feature_name}/{feature_name}.md` (sections: The request, Planned approach, What was implemented).
  - **Bug (Phase 7)**: Bug name, `{bugs_dir}`, `{archive_dir}`. Run `6-archive-bug-summary.md`; read request + all docs under `{bugs_dir}/{bug_name}/`; write `{archive_dir}/fix-{bug_name}/fix-{bug_name}.md` (same three sections).

### 2.7 AI Context Writer (`/ai-context-writer`)

- **Role**: LLM-oriented, developer-facing context files in `{context_docs_dir}`. No management content.
- **Resources**: README, existing `{context_docs_dir}/` files, source/tests per skill, `.cursor/rules/*.mdc`. Trust code over outdated docs.
- **Modes** (each driven by a skill; see [SKILLS_AND_RULES.md](SKILLS_AND_RULES.md)):
  - **Quick Reference**: skill `ai-context-quick-reference` → `AI_CONTEXT_QUICK_REFERENCE.md`.
  - **Repository**: skill `ai-context-repository` → `AI_CONTEXT_REPOSITORY.md`.
  - **Patterns**: skill `ai-context-patterns` → `AI_CONTEXT_PATTERNS.md`.
  - **Component**: skill `ai-context-component`; orchestrator passes component name + path → `AI_CONTEXT_{COMPONENT_NAME}.md`.
- **Output**: All files under `{context_docs_dir}`; metadata (version, last updated, tags, cross-refs); under content length limit.

---

## 3. Invocation Pattern

```
Orchestrator command (e.g. feature-pipeline)
    → reads scratchpad
    → invokes /agent with phase-specific instructions + workflow reference
    → agent runs workflow (reads/writes feature or bug dir, updates scratchpad)
    → orchestrator runs commit procedure (if phase successful)
    → next phase or stop
```

Each subagent run uses a **fresh agent context**; the scratchpad is the only shared state between phases.
