# Workflows

Workflow files live in `.cursor/workflows/`. They are **phase-specific step definitions** executed by **agents** when invoked by orchestrator commands. Do **not** run workflow files directly; use **feature-pipeline**, **bug-pipeline**, or **update_context** so that scratchpad, commits, and phase order stay correct.

Path placeholders (`{features_dir}`, `{bugs_dir}`, etc.) are in [CONVENTIONS.md](../../.cursor/CONVENTIONS.md).

---

## 1. Workflow Summary by Pipeline

### 1.1 Feature pipeline

| Phase | Workflow(s) | Agent | Purpose |
|-------|-------------|--------|---------|
| 1 Research | 1-research-feature | /researcher | Planning docs: impacted-systems, new-systems, possible-solutions, selected-solution in `planning/`. |
| 2 Plan | 2-plan-feature | /architect | Master implementation plan in `plans/master/` (overview, architecture, interfaces, etc.). |
| 3 Task list | 3-task_list, 4-feature-tasks | /task-decomposer | Task list + per-task docs in `tasks/`. |
| 4 Per-task plan | 5-plan-task-feature | /architect | One plan per task in `plans/tasks/{task_file}.md`. |
| 5 Implement | implement-task-feature | /implementer | Implement one task; TDD; run tests; commit. |
| 6 Clean up | 6-archive-feature-summary | /archivist | One summary at `{archive_dir}/{feature_name}/{feature_name}.md`. |

Standalone (not per-phase): **implement-feature** (full feature in one go, with user approval); **implement-task-feature** is the per-task workflow used by the pipeline.

### 1.2 Bug pipeline

| Phase | Workflow(s) | Agent | Purpose |
|-------|-------------|--------|---------|
| 1 Investigate | 1-investigate-bug | /debug-specialist | Investigation docs in `investigation/` (observed-behavior, reproduction-steps, logs-and-traces, affected-components, root-cause-hypothesis). |
| 2 Research | 2-research-bug | /researcher | Research docs in `research/`. |
| 3 Plan | 3-plan-bug-fix | /architect | Fix plan in `plans/master/`. |
| 4 Task list | 4-bug-fix-tasks | /task-decomposer | Task docs in `tasks/`. |
| 5 Per-task plan | 5-plan-task-bug | /architect | One plan per task in `plans/tasks/{task_file}.md`. |
| 6 Implement | implement-task-fix | /implementer | Implement one fix task; tests; commit. |
| 7 Clean up | 6-archive-bug-summary | /archivist | One summary at `{archive_dir}/fix-{bug_name}/fix-{bug_name}.md`. |

Standalone: **implement-fix** (full fix in one go); **implement-task-fix** is the per-task workflow used by the pipeline.

### 1.3 AI context (update_context)

No numbered workflows in `workflows/`. The **update_context** command invokes **/ai-context-writer** per document type (and per component); the writer follows the **skills** in `.cursor/skills/ai-context-*/` — see [SKILLS_AND_RULES.md](SKILLS_AND_RULES.md).

---

## 2. Workflow File Reference

```
.cursor/workflows/
├── 1-investigate-bug.md      # Bug Phase 1
├── 1-research-feature.md     # Feature Phase 1
├── 2-plan-feature.md         # Feature Phase 2
├── 2-research-bug.md         # Bug Phase 2
├── 3-plan-bug-fix.md         # Bug Phase 3
├── 3-task_list.md            # Feature Phase 3 (task list review)
├── 4-bug-fix-tasks.md        # Bug Phase 4
├── 4-feature-tasks.md        # Feature Phase 3 (task decomposition)
├── 5-plan-task-bug.md        # Bug Phase 5
├── 5-plan-task-feature.md    # Feature Phase 4
├── 6-archive-bug-summary.md  # Bug Phase 7
├── 6-archive-feature-summary.md  # Feature Phase 6
├── implement-feature.md      # Full feature (standalone)
├── implement-fix.md          # Full fix (standalone)
├── implement-task-feature.md # Feature Phase 5 (per task)
├── implement-task-fix.md     # Bug Phase 6 (per task)
└── README.md                 # Usage note: use pipelines, not direct invoke
```

---

## 3. Feature Workflow Highlights

- **1-research-feature**: Produce planning docs under `planning/`; no use of `reports/` as input. Adhere to development_practices and content_length.
- **2-plan-feature**: Prerequisites: planning docs + feature file. Read planning docs and feature file; create `plans/master/` (e.g. overview.md, architecture.md, interfaces.md, implementation.md, testing.md, todos). No use of `reports/` as input.
- **3-task_list**: Review plan; ensure clear objectives and research; produce high-level task list with dependencies; TDD and interfaces first.
- **4-feature-tasks**: Read plan + template `_template.md`; decompose into detailed task docs in `tasks/` with dependencies and TDD ordering.
- **5-plan-task-feature**: One task file in; one plan file out at `plans/tasks/{task_file}.md`.
- **implement-task-feature**: Context gathering (AI context, changelog, task, plan, scratchpad); implementation (TDD, verify fail then pass, quality tools, full test suite); completion (changelog, docs, scratchpad, commit). Regression gate: all tests must pass except expected failures.
- **6-archive-feature-summary**: Read request + all artifacts; write one doc with The request, Planned approach, What was implemented.

---

## 4. Bug Workflow Highlights

- **1-investigate-bug**: Understand bug, gather facts, write investigation docs in `investigation/`.
- **2-research-bug**: Read investigation; produce research docs in `research/`.
- **3-plan-bug-fix**: Read investigation + research; produce fix plan in `plans/master/`.
- **4-bug-fix-tasks**: Read plan; produce task docs in `tasks/`.
- **5-plan-task-bug**: One task → one plan in `plans/tasks/`.
- **implement-task-fix**: Same structure as implement-task-feature but for a single fix task.
- **6-archive-bug-summary**: Same three sections; output path `{archive_dir}/fix-{bug_name}/fix-{bug_name}.md`.

---

## 5. Data and Length Rules

- **Reports**: No workflow (feature, bug, or AI context) may use `{reports_dir}` as **input** unless the user explicitly provides content from there. Reports are a valid **output** destination.
- **Length**: Workflows and agents follow `content_length.mdc` (e.g. 500-line cap; split/index for long docs).
