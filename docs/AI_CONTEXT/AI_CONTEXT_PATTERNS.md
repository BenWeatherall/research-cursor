# AI Context Patterns

- **Version:** 1
- **Last Updated:** 2026-02-13
- **Tags:** patterns, conventions
- **Cross-References:** [AI_CONTEXT_REPOSITORY.md](AI_CONTEXT_REPOSITORY.md) | [AI_CONTEXT_QUICK_REFERENCE.md](AI_CONTEXT_QUICK_REFERENCE.md) | [AI_CONTEXT_CURSOR_CONFIG.md](AI_CONTEXT_CURSOR_CONFIG.md)

---

This repository contains **no application code**; it holds Cursor config (agents, commands, rules, skills, workflows). The patterns below reflect **project rules** that apply when this config is used in a repository that has source and tests. They are the conventions agents should follow for code organization, types, testing, and validation.

---

## Code Organization

- **Discover structure** from README and `.cursor/CONVENTIONS.md`; do not assume fixed directory names (e.g. `src/`, `python_service/`). Use placeholders (`{features_dir}`, `{context_docs_dir}`, etc.) in commands and workflows.
- **Docs:** Project docs in `docs/` or repo root; reports in `reports/{report-name}/`; AI context in `docs/AI_CONTEXT/`. See content length rules for splitting (e.g. index + subfiles when exceeding limit).

---

## Type & Model Patterns

- **Data classes / models:** Use models for complex or nested data transfer (per `.cursor/rules/development_practices.mdc`).
- **Validation over testing:** Ingest and parse via a model; if validation fails, do not use the data. Once valid, do not re-check validity.

---

## Error Handling & Logging

- Not specified in this repo’s rules; follow project-specific conventions where application code exists.

---

## Testing & TDD

- **TDD:** Write a test first.
- **Black box:** Tests assert behavior and outcomes, not internals (no "called_once", no log-line checks). For external data changes, mocks must provide the changed object; for internal data, assert returned or referenced state; for exceptions, use expect-exception style.
- **Working and valid tests:** Tests must run and pass when behavior is correct, and **must be able to fail** when behavior is wrong.
- **Regression gate:** Any failing test not marked as expected failure (`@pytest.mark.xfail`, `@unittest.expectedFailure`) is a regression and must be fixed before task completion.

---

## Dependency Injection & Validation

- **Non-deterministic systems** (e.g. `time`, `google.cloud.firestore.Client`) must be **injected as parameters**, not hard-wired.
- **Validation:** Prefer loading into a model; use validation success/failure as the gate for using data.

---

## Q&A Behavior Examples

**Q: How do I add a new pipeline command?**  
A: Add a command file under `.cursor/commands/` and document it in README. Use path placeholders from CONVENTIONS.md.

**Q: Where do reports go?**  
A: Under `reports/{report-name}/` only. Not in `docs/`. Reports may exceed standard doc length.

**Q: How do I add or change path conventions?**  
A: Edit `.cursor/CONVENTIONS.md` for defaults. Override in project README or `.cursor/rules`; agents resolve placeholders from those sources.

**Q: How should I run the feature pipeline?**  
A: In Cursor Composer run `/feature-pipeline` with a feature file in `_features/`. Do not invoke workflow files directly.

**Q: How is AI context updated?**  
A: Run `/update_context`. It (re)generates quick reference, repository, patterns, and per-component docs in `docs/AI_CONTEXT/` using skills under `.cursor/skills/ai-context-*/`. Do not use `reports/` as input unless the user explicitly provides it.

**Q: What if a doc or code file would exceed the length limit?**  
A: Per `content_length.mdc`: split into logical subfiles in a folder named after the original, with an `index.md` listing subfiles and brief descriptions. Exception: content under `reports/{report-name}/` may exceed limits.
