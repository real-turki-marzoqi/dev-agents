# Dev Team — Autonomous Multi-Agent Pipeline for Claude Code

A Claude Code plugin that turns one idea into delivered, tested, reviewed,
documented code — driven by a team of 11 specialized agents with exactly
**one human approval** in the middle.

```
idea ──▶ clarify ──▶ plan ⇄ adversarial review ──▶ ★ human approval
     ──▶ implement + design ──▶ test gate ──▶ code & security review
     ──▶ docs ──▶ delivered (full task archive on disk)
```

## Why

- **Adversarial planning** — an analyst proposes; a plan-reviewer and a
  critic attack the proposal in parallel until it survives (max 5 rounds).
- **Deterministic gates** — the coordinator runs the tests itself and trusts
  exit codes, never an agent's claim. Failures are routed to the right role
  (developer / test-writer / designer / back to planning) automatically.
- **Environment-aware** — before anything, the pipeline maps the environment
  it's in (existing codebase, Odoo module, monorepo, greenfield) and every
  agent works from that shared picture.
- **Persistent design system** — the first UI task creates
  `design-system.md` at your project root; every later task must reuse those
  tokens. Your app stays visually consistent across months of tasks.
- **Design quality built in** — bundles Anthropic's official
  [frontend-design](https://github.com/anthropics/skills/tree/main/skills/frontend-design)
  skill (Apache 2.0) so UI work avoids generic AI-styled output.
- **Memory** — each role accumulates lessons per project in `memory/` and
  reads them before working.

## Install

```
/plugin marketplace add real-turki-marzoqi/dev-agents
/plugin install dev-team
```

Run these inside the Claude Code CLI (`claude` in a terminal). Once
installed, the plugin is available in every environment — CLI, VS Code
extension, and desktop app.

Then, once per project:

```
/dev-team:setup
```

Setup asks a single question about enabling full permissions
(`bypassPermissions`) **scoped to that project only** — the pipeline is built
to run unattended, but that mode lets Claude run commands and edit files
without per-action prompts, so it is always your explicit choice. It also
creates the `tasks/` and `memory/` workspace and detects your test command.

## Use

```
/dev-team:run-task <your idea, in any language>
```

The pipeline asks its clarifying questions (product language defaults to
English — it will ask which additional languages you want), presents one plan
for approval, then runs to delivery. Resume an interrupted task with
`/dev-team:run-task --resume <task-id>`.

## The team

| Phase | Agents |
|---|---|
| Planning | analyst · plan-reviewer · critic · design-lead (UI only) · task-scribe |
| Coding | developer · designer (UI only) · test-writer |
| Gates | coordinator-run tests · code-reviewer · security-reviewer |
| Docs | doc-writer |

## What lives where

| Layer | Location | Published? |
|---|---|---|
| The system (agents, coordinator, standards, bundled skill) | this plugin | yes |
| Project state (`tasks/`, `memory/`, `design-system.md`) | each user project | never |

A project may override the bundled defaults by keeping its own
`standards/` files — discovered conventions always beat the shipped standard.

## License

MIT for this project. The bundled `skills/frontend-design/` skill is by
Anthropic, licensed Apache 2.0 — see `skills/frontend-design/LICENSE.txt`
(included unmodified).

---

## بالعربية

نظام فريق تطوير آلي لـ Claude Code: تكتب فكرتك بأي لغة، الفريق يخطط ويراجع
الخطة عدائيًا، تعتمدها **بموافقة واحدة**، ثم ينفّذ ويختبر ويراجع الكود
والأمان ويوثّق — ويسلّمك أرشيف كامل للتاسك. أول تاسك واجهات ينشئ هوية بصرية
دائمة (`design-system.md`) تلتزم بها كل التاسكات اللاحقة، فيبقى تطبيقك موحّد
الشكل. لغة المنتج الافتراضية إنجليزي، والنظام يسألك عن اللغات الإضافية
(ودعم RTL يتفعّل تلقائيًا مع العربية).
