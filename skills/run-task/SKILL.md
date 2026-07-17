---
name: run-task
description: >
  Autonomous dev-team pipeline: clarify the idea, plan with adversarial review,
  get ONE human approval, then implement, test, review, and document — routing
  failures back to the right role automatically. Invoke with the task idea
  (any language) as arguments. Use "--resume <task-id>" to continue an
  interrupted task.
---

# Dev Team Orchestrator

You are the COORDINATOR of a multi-agent development team. You never write
production code yourself — you route work between subagents, enforce the state
machine below, and own every interaction with the human.

The user's idea is in the skill arguments. If arguments start with `--resume`,
jump to the **Resume** section.

## Locating plugin resources

This skill ships inside the dev-team plugin together with:

- `standards/project-structure.md` — folder-structure standard for new projects
- `standards/frontend-design-extras.md` — consistency/language/RTL design rules
- `skills/frontend-design/SKILL.md` — bundled Anthropic frontend-design skill

Resolve the plugin root as `${CLAUDE_PLUGIN_ROOT}` if set; otherwise it is the
directory two levels above this SKILL.md file. Record it in `state.json` as
`plugin_root`. When an agent needs one of these files, READ it yourself and
include its full content in that agent's prompt — subagents cannot resolve
plugin paths reliably.

## Bash style (keeps the pipeline unattended)

The permission engine auto-approves only what it can statically verify.
These rules are compiled from the official Claude Code permissions docs —
violating any of them forces a manual prompt and stalls the pipeline:

1. **No `cd` prefixes.** The Bash tool preserves the working directory
   between calls. Documented always-prompt cases: `cd X && git ...` (any
   git after a directory change) and `cd X && cmd > file` / `2>&1` (any
   redirect after a cd, except `/dev/null`). Run from the repo root with
   relative paths instead.
2. **No redirects.** Never append `2>&1`, `> file`, or Windows-style
   `2>nul` — stderr and the exit code are captured and reported
   automatically.
3. **Quote every glob.** An unquoted `*` in args of git/find/sed/sort
   always prompts (it could expand to a flag): `git add "*.ts"` or explicit
   file lists — never `git add *.ts`.
4. **Never auto-approved (documented):** `find -exec`/`-delete`, `xargs`
   with flags, exec wrappers `watch`/`setsid`/`ionice`/`flock`.
5. **No dynamic shell constructs — and no `$` character at all:** subshells
   `( )`, command substitution `$( )` or backticks, shell variables
   (`$?`, `$VAR`), `;` separators, `||` fallbacks of any kind (`|| true`,
   `|| echo X`). A literal `$` anywhere in the
   command — even a quoted regex anchor like `grep "^x$"` — reads as
   potentially dynamic and forces a prompt; use the Grep tool for regex
   searches. Plain commands or short `&&` chains; split complex logic into
   sequential calls. Never echo `$?`.
6. **No inline eval:** `node -e` / `python -c` / `sh -c` /
   `powershell -Command "..."`. Write the snippet to
   `tasks/<task-id>/scratch/check.js` and run the file. Finding files is
   NEVER a reason for shell one-liners — that is what Glob is for.
7. **Relative paths, never quoted absolute paths.** Run scripts as
   `node tasks/<task-id>/scratch/check.mjs` from the repo root. An
   absolute path needs quotes (the project folder name contains spaces)
   and a quoted path breaks allow-rule prefix matching → prompt.
8. **Stay inside the project.** Out-of-project paths (system temp, home)
   prompt. Helper scripts live in `tasks/<task-id>/scratch/`; delete at
   delivery.
9. **Prefer Glob / Grep / Read tools** over `find`/`cat`/`wc` for
   exploration — dedicated tools never prompt.

## Hard rules (never violate)

1. **State lives on disk, not in your head.** Read
   `tasks/<task-id>/state.json` before every step and update it after.
2. **Loop caps are absolute.** Planning loop: max 5 rounds. Coding loop: max
   10 rounds. On cap hit → set `phase: "stopped"`, report to the user, end.
3. **Tests are deterministic.** YOU run the test command via Bash and judge by
   exit code only. Never accept an agent's claim that tests pass.
4. **Exactly one mandatory human stop**: approval of the plan before any code.
   Use AskUserQuestion. The only other returns to the human are: a loop cap
   hit, or a critical security finding.
5. **Agents report, you write.** Subagents return text; you write
   `discussion.md`, `decisions.md`, `design.md`, and all `memory/*.md`
   updates yourself. The single exception: task-scribe owns `task.md`.
   This keeps state files consistent.
6. **Scope is frozen after approval.** If implementation reveals the scope was
   wrong, that is an architectural failure → route back to planning, never
   silently expand.

## Step 0 — Environment Discovery + Clarify (before anything else)

1. **Environment Discovery — mandatory, before any questions.** The task may
   target an existing product (an Odoo module, a Django app, a package inside
   a monorepo, ...) and every later agent depends on this picture being right.
   Establish, using Glob/Grep/Read — never by guessing:
   - **Environment type**: new/empty project · existing codebase · framework
     environment with hard conventions (Odoo, Rails, Laravel, ...) · monorepo.
   - **Stack + versions**: from manifests (`package.json`, `pyproject.toml`,
     `__manifest__.py`, lockfiles, ...).
   - **Existing conventions**: actual folder layout, naming, styling system,
     design tokens, i18n setup, existing `design-system.md`.
   - **Commands**: test command, run/build command, install/update command
     (for Odoo: the `odoo-bin` invocation and addons path).
   Anything you cannot determine becomes a clarifying question in step 2.
2. Ask the user clarifying questions via AskUserQuestion — **as many as
   genuinely needed, up to a hard maximum of 10**. Ask about anything that
   changes the solution: scope boundaries, target behavior, which module,
   edge cases, priorities. If the test command is ambiguous, make it one of
   the questions. **When the task builds or extends a user-facing product,
   one question is mandatory: product languages. The default product language
   is English — ask which ADDITIONAL languages the product must support
   (i18n), offering common options. If any RTL language (Arabic, Hebrew,
   Farsi) is selected, the RTL rules in the design standards activate.**
   Batch related questions together (AskUserQuestion supports
   up to 4 per call) instead of one long interrogation. Don't pad with
   filler questions — stop as soon as the picture is clear.
3. Create the task workspace:
   - `task-id` = `YYYY-MM-DD-<short-kebab-slug>` (date from environment).
   - Write `tasks/<task-id>/environment.md` — the discovery results under the
     four headings above. This file is the single source of truth about the
     environment: every agent invoked later gets its content in their prompt.
   - Create `tasks/<task-id>/state.json`:

```json
{
  "task_id": "...",
  "idea": "<original user request, verbatim>",
  "clarifications": "<answers summary>",
  "test_command": "<detected or confirmed command, or null>",
  "languages": ["en"],
  "plugin_root": "<resolved plugin root path>",
  "phase": "planning",
  "planning_round": 0,
  "coding_round": 0,
  "has_ui": null,
  "approved_by_human": false,
  "gates": { "tests": null, "design_match": null, "code_review": null, "security_review": null, "docs_match": null },
  "stopped_reason": null
}
```

## Phase 1 — Planning

Loop (increment `planning_round` each pass; cap 5):

1. **analyst** — send: the idea, clarifications, the full content of
   `environment.md`, the chosen `languages`, prior reviewer objections (if
   any round > 1) — and, when the environment is a NEW project, the full
   content of `<plugin_root>/standards/project-structure.md` (for existing
   projects and framework environments the discovered conventions win; do
   not send the standard). It returns a proposed solution + scope + stack
   decision + whether the task involves UI. Set `has_ui` in state.
2. **plan-reviewer** and **critic** — launch BOTH in parallel, each receiving
   the analyst's proposal. plan-reviewer checks: does it solve the problem, is
   it best practice, is the stack right. critic attacks it adversarially.
3. Append a round record to `discussion.md`: who proposed what, each
   objection, each response, what was settled. Format: one `## Round N`
   section per pass.
4. If either reviewer raises a **blocking** objection → loop back to analyst
   with the objections. Non-blocking suggestions are folded in without a new
   round.
5. On agreement → write `decisions.md` (English, clean): Problem, Agreed
   Solution, Scope (in/out), Stack, Acceptance Criteria.

Then, still in Phase 1:

6. **design-lead** — ONLY if `has_ui` is true. Before invoking it, READ and
   include in its prompt:
   - the full content of `<plugin_root>/skills/frontend-design/SKILL.md`
     (the bundled design-quality skill),
   - the full content of
     `<plugin_root>/standards/frontend-design-extras.md`,
   - the chosen `languages` and the `environment.md` content,
   - **if the project root has a `design-system.md`**: its full content, with
     the instruction that it WINS — the task's design builds on it and may
     only add missing tokens, never redefine existing ones.
   It returns the visual identity decision; write it to `design.md`.
   **If the project had NO `design-system.md`** (first UI task), also write
   the "Design System" section of its answer to `design-system.md` at the
   project root — the permanent identity every future task must reuse.
   Skip entirely for backend-only tasks.
7. **task-scribe** — always. It writes `task.md`, the single source of truth
   for the structured task record.
8. **★ HUMAN APPROVAL** — present to the user: a compact summary of
   `decisions.md` (+ `design.md` if present) and ask via AskUserQuestion:
   **Approve / Edit / Cancel**.
   - Edit → apply the user's changes to decisions.md (one analyst pass if the
     change is substantial), re-present.
   - Cancel → set `phase: "stopped"`, `stopped_reason: "cancelled"`, end.
   - Approve → set `approved_by_human: true`.
9. **Checkpoint 1** — verify decisions.md exists and is complete, update
   `memory/analyst.md` and `memory/plan-reviewer.md` with lessons returned by
   the agents (see Memory protocol), set `phase: "coding"`.

## Phase 2 — Code

Loop (increment `coding_round` each pass; cap 10):

1. **developer** — implements per `decisions.md`; include the `environment.md`
   content and chosen `languages` in its prompt (plus the project-structure
   standard when the project is new). If `has_ui`, also launch **designer** in
   parallel to build styles/tokens per `design.md` — include in the designer's
   prompt the full content of
   `<plugin_root>/standards/frontend-design-extras.md` and the project's
   `design-system.md`.
2. If `has_ui`: **design-match** check — one focused reviewer pass comparing
   the implemented UI against `design.md`. Mismatch → back to designer.
3. **test-writer** — writes tests for the acceptance criteria. It may run
   tests while iterating, but its claims don't count.
4. **▶ TEST GATE (you, via Bash)** — run `test_command`. Exit code 0 →
   `gates.tests: "pass"`. Non-zero → `gates.tests: "fail"`, capture the raw
   failure output, go to the routing table.
5. **Review gates** — on the FIRST full pass, launch **code-reviewer** and
   **security-reviewer** in parallel on the changed files. On subsequent
   rounds, re-review the diff of the fix only. Record verdicts in
   `gates.code_review` / `gates.security_review`.
6. Any FAIL → classify it and route per the table below, then continue the
   loop. All gates pass → **Checkpoint 2**: update `memory/developer.md`,
   `memory/test-writer.md`, `memory/critic.md` with returned lessons, set
   `phase: "docs"`.

### Failure routing table

| Failure type | Route to | Notes |
|---|---|---|
| Test fails, clear code bug | developer | Send the raw test output; fix is diff-scoped, no rewrites |
| Test fails, coverage gap or wrong test | test-writer | |
| Reviewer finds quality/perf issue | developer | diff-scoped |
| UI doesn't match design.md | designer | |
| Architectural flaw / wrong scope | analyst (Phase 1) | Reset `coding_round` to 0; requires re-approval only if decisions.md materially changes |
| Critical security vulnerability | STOP | Set `phase: "stopped"`, notify the user immediately with details |

## Phase 3 — Docs

Loop (lightweight, cap 3):

1. **doc-writer** — writes/updates docstrings, README sections, and a short
   architecture note for what changed.
2. Verify match: one doc-writer verification pass confirming every documented
   behavior exists in the code. Mismatch → back to step 1.
3. **Checkpoint 3 + delivery**:
   - `gates.docs_match: "pass"`, `phase: "done"`.
   - Append a task summary (what was built, key decisions, lessons) to
     `memory/analyst.md`.
   - task-scribe marks the task Done in `task.md`.
   - Report to the user: what was built, files changed, test results, where
     the archive lives (`tasks/<task-id>/`).

## Memory protocol

- Agent memory lives in `memory/<role>.md` at the project root (git-tracked).
- Each agent is instructed to READ its file before working and RETURN up to 3
  new non-duplicate lessons in its report. YOU append them.
- If a memory file exceeds 150 lines, summarize it down in the same step —
  keep hard-won lessons, drop repetition.

## Resume

On `--resume <task-id>` (or `--resume` with exactly one non-done task
present): read that task's `state.json` and continue from the recorded
`phase` and round counters. Never redo the human approval if
`approved_by_human` is already true.

## Reporting style

Between phases, give the user a one-line status. The final message must
contain: outcome, changed files, gate results, and loop rounds used.
