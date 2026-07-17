---
description: One-time project setup for the dev-team pipeline — permissions, workspace folders, and test command.
---

Set up the current project for the dev-team pipeline. Do these steps in order.

## 1. Permissions (ask exactly one clear question)

The pipeline is designed to run unattended; that requires full permissions
inside THIS project only. Ask the user via AskUserQuestion:

> "The dev-team pipeline runs autonomously and works best with full permissions
> scoped to this project (`bypassPermissions` in `.claude/settings.json`).
> Claude will be able to run commands and edit files here without asking each
> time. Enable it?"
>
> Options: **Enable (recommended for this pipeline)** / **Skip — I'll approve
> actions manually**

- If **Enable**: merge into the project's `.claude/settings.json` (create it if
  missing, PRESERVE existing keys):

```json
{
  "permissions": {
    "defaultMode": "bypassPermissions"
  }
}
```

- If **Skip**: change nothing, and tell the user the pipeline will pause for
  permission prompts.

## 2. Workspace folders

Create if missing (never overwrite existing content):

- `tasks/` — one folder per task will be created by `/dev-team:run-task`.
- `memory/` — per-role lesson files accumulate here as tasks run.

If the project is a git repo, ensure `tasks/*/scratch/` is in `.gitignore`.

## 3. Test command

Detect the project's test command (from `package.json` scripts, `pyproject.toml`,
`Makefile`, framework conventions — for Odoo modules, the odoo-bin test
invocation). If ambiguous or absent, ask the user once. Record the answer for
the pipeline by writing it to `memory/project.md` under a `## Test command`
heading.

## 4. Report

Tell the user: what was configured, and that they can now run
`/dev-team:run-task <idea>` in any language — the pipeline asks its
clarifying questions, gets one plan approval, and runs to delivery on its own.
