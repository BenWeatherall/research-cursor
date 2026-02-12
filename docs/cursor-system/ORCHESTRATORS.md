# Orchestrators (Commands)

Orchestrator commands live in `.cursor/commands/`. They coordinate agents and workflows; run them from Composer (e.g. `/feature-pipeline`, `/bug-pipeline`, `/update_context`).

---

## 1. Command Summary

| Command             | Role | When to use |
|---------------------|------|-------------|
| **feature-pipeline** | Multi-phase feature implementation | Feature file in `{features_dir}/{feature_name}.md`; user provides feature name/path. |
| **bug-pipeline**      | Multi-phase bug investigation and fix | User provides bug name; pipeline creates/uses `{bugs_dir}/{bug_name}/`. See note below. |
| **update_context**   | Regenerate AI context docs | Refresh `{context_docs_dir}` (default `docs/AI_CONTEXT/`). Idempotent. |
| **commit**            | Suggest Conventional Commits message | After changes; does not run `git commit`. |
| **content_length**    | Enforce doc/code length rule | Pass file or folder; applies `content_length.mdc`. |

---

## 2. Feature Pipeline (feature-pipeline)

- **Prerequisites**: Feature file at `{features_dir}/{feature_name}.md` with Background, This Task, Testing Needed; user provides feature name or path.
- **Data restriction**: No phase may use `{reports_dir}` as input unless the user explicitly provides content from there. Reports remain a valid **output** destination.

### 2.1 Resume detection (before Phase 1)

1. Read `{scratchpad}`; determine `{feature_name}` from the user request.
2. If scratchpad has a "Feature: {feature_name}" section with Pipeline State and Task Completion Status:
   - **Phases 1–4**: Skip any phase marked "Complete".
   - **Phase 5**: Remaining tasks = Execution Order minus Task Completion Status; run only those.
3. If no matching state or scratchpad empty: add "Feature: {feature_name}" with Pipeline State (all not started), Task Completion Status (empty), Execution Order (empty); then start Phase 1.

### 2.2 Phase summary

| Phase | Agent | Workflows | Commit after |
|-------|--------|-----------|---------------|
| 1 Research | /researcher | 1-research-feature | docs(feature): research {feature_name} |
| 2 Plan | /architect | 2-plan-feature | docs(feature): plan {feature_name} |
| — | — | **STOP: wait for user approval** | — |
| 3 Task list | /task-decomposer | 3-task_list, 4-feature-tasks | docs(feature): generate task list for {feature_name} |
| 4 Per-task plan | /architect (per task) | 5-plan-task-feature | docs(feature): plan tasks for {feature_name} |
| 5 Implement | /implementer (per task) | implement-task-feature | Implementer commits per task with inferred type/scope |
| 6 Clean up | /archivist | 6-archive-feature-summary | docs(feature): archive summary for {feature_name} |

After Phase 6: delete `{features_dir}/{feature_name}.md` and the folder `{features_dir}/{feature_name}/`; clear the feature from the scratchpad.

---

## 3. Bug Pipeline (bug-pipeline)

- **Prerequisites**: User provides bug name/identifier; initial error description or reproduction.
- **Note**: The bug pipeline is **valuable even when there is no application code**. It provides structured investigation (debug-specialist), research, planning, task decomposition, and archival. In repos without a test suite, the “run full test suite” step in the commit procedure is effectively a no-op until tests exist; the rest of the flow (investigation, plans, archive) still applies.

### 3.1 Resume detection

Same idea as feature pipeline: if scratchpad has "Bug: {bug_name}" with Pipeline State and Task Completion Status, skip completed phases and run only remaining tasks in Phase 6.

### 3.2 Phase summary

| Phase | Agent | Workflows |
|-------|--------|-----------|
| 1 Investigate | /debug-specialist | 1-investigate-bug |
| 2 Research | /researcher | 2-research-bug |
| 3 Plan | /architect | 3-plan-bug-fix |
| — | **STOP: wait for user approval** | — |
| 4 Task list | /task-decomposer | 4-bug-fix-tasks |
| 5 Per-task plan | /architect (per task) | 5-plan-task-bug |
| 6 Implement | /implementer (per task) | implement-task-fix |
| 7 Clean up | /archivist | 6-archive-bug-summary |

Archive path: `{archive_dir}/fix-{bug_name}/fix-{bug_name}.md`. Then delete bug request and `{bugs_dir}/{bug_name}/`.

---

## 4. Update Context (update_context)

Orchestrator for AI context documentation; **does not** run feature or bug pipelines.

- **Rule**: Do not use `{reports_dir}` as input for generating/updating AI context unless the user explicitly provides it.
- **Phases**: Quick Reference → Repository → Patterns → Per-component (for each discovered component). Each phase invokes `/ai-context-writer` once with a **fresh agent context** and the corresponding skill.
- **Output**: All files under `{context_docs_dir}` (default `docs/AI_CONTEXT/`). Document types and skills are listed in [SKILLS_AND_RULES.md](SKILLS_AND_RULES.md).

---

## 5. Commit Procedure (shared)

Used after each successful phase or task in feature-pipeline (and similarly in bug-pipeline). A phase/task is successful when required steps are done and **all project tests pass** (excluding explicitly expected failures).

1. **Run full test suite** (per project rules). Any other failure is a regression and must be fixed before committing.
2. **Stage**: `git add .`
3. **Safety check**: Ensure no `.env`, `*.key`, `*.pem`, or `secrets.*` are staged; if found, unstage and warn.
4. **Message**: Conventional Commits per `.cursor/commands/commit.md`: `<type>(<scope>): <description>`; optional body (goal-focused, &lt;100 chars per line). Type/scope are given per phase (see table above); for implementation tasks the implementer infers from changes.
5. **Commit**: `git commit -m "<message>"`

---

## 6. Commit Command (standalone)

The **commit** command only **suggests** a message; it does not run `git commit`.

- **Phases**: (1) Gather uncommitted state, (2) Safety check (blocklist: `.env`, `*.key`, `*.pem`, `secrets.*`), (3) Analyze changes for type/scope/goal, (4) Output Conventional Commits message in a code block.
- If any file is flagged in the safety check, the command pauses and asks the user how to handle it; it does not proceed to generate a message until the user confirms.

---

## 7. Content Length Command

- **Input**: Optional file or folder (default: discover markdown/code files and apply to all).
- **Behaviour**: Applies the rule in `.cursor/rules/content_length.mdc` (e.g. 500-line cap; split/index pattern for docs; module conversion for long code).
