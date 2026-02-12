# docs

This folder is **only** for documentation that belongs to this repository:

- **Cursor workflow** — Workflows, agents, commands, rules, and related Cursor configuration (the authoritative sources live under `.cursor/`; this folder may hold supplementary or generated workflow docs).
- **Cursor system documentation** — The `cursor-system/` subfolder documents the `.cursor` setup: orchestrators (commands), agents, skills, workflows, and rules. See [cursor-system/README.md](cursor-system/README.md) for the overview and index. Audience: developers experienced with Cursor and LLM/agent-assisted development.
- **AI context** — The `AI_CONTEXT/` subfolder holds AI-generated context (quick reference, repository overview, patterns, per-component docs) as defined in `.cursor/CONVENTIONS.md`.

Human-authored project documentation (e.g. architecture, runbooks) may also live at the **repository root** (e.g. `README.md`) or **here alongside** `AI_CONTEXT/`.

**Reports** (research reports, tool reports, long-form outputs) do **not** go in `docs/`. They live under **`reports/{report-name}/`** so they can exceed standard length limits and include multiple files. Example: `reports/typer-cli-report/typer-cli-report.md`.
