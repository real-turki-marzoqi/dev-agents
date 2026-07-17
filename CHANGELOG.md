# Changelog

## v0.1.0 — 2026-07-17

First public release, restructured as a Claude Code plugin.

- Plugin layout: `.claude-plugin/` manifests, `agents/`, `skills/`,
  `commands/`, `standards/`.
- 11-agent pipeline (`/dev-team:run-task`): adversarial planning, one human
  approval, coordinator-run test gate, code + security review, docs phase,
  failure-routing table, per-role memory, `--resume`.
- **New:** mandatory Environment Discovery step — the coordinator maps the
  environment (existing codebase, framework conventions such as Odoo,
  monorepo, greenfield) into `environment.md` before planning; all agents
  work from that shared picture.
- **New:** persistent per-project `design-system.md` — created on the first
  UI task, binding for every later task; tokens are locked values.
- **New:** bundled Anthropic `frontend-design` skill (Apache 2.0, unmodified)
  as the design-quality bar, plus `standards/frontend-design-extras.md`
  (consistency, i18n/RTL, accessibility baseline).
- **New:** `standards/project-structure.md` — feature-first layout standard
  for greenfield projects; existing/framework conventions always win.
- **New:** product language rule — default English; the pipeline asks which
  additional languages to support, and RTL rules activate only when an RTL
  language is selected.
- **New:** `/dev-team:setup` — one-time project onboarding: opt-in
  project-scoped `bypassPermissions`, workspace folders, test-command
  detection.
