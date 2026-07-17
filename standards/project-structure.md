# Project Structure Standard

Scope rule — read this first:

- **Existing project or framework with its own conventions (Odoo, Django, Rails,
  Laravel, Next.js app router, ...): the existing/framework structure WINS.**
  This standard applies only where the project or framework leaves the choice open.
- **New project from scratch:** this standard is mandatory.

## Layout for new projects

Organize by **feature/domain, not by technical layer**. A feature's code, styles,
and tests live together, so the project scales by adding folders — not by growing
god-directories.

```
<root>/
├── src/
│   ├── features/            # one folder per feature/domain
│   │   └── <feature>/
│   │       ├── components/  # UI for this feature (if any)
│   │       ├── api|services/# feature logic / data access
│   │       └── tests/       # tests for THIS feature (or __tests__/)
│   ├── shared/              # cross-feature code ONLY (used by 2+ features)
│   │   ├── components/      # shared UI primitives
│   │   ├── utils/
│   │   └── styles/          # tokens, theme, global styles
│   └── app|main entry       # composition root: wiring, routing, config
├── tests/                   # cross-feature/e2e tests only
├── docs/                    # architecture notes
├── design-system.md         # persistent visual identity (UI projects; see below)
└── README.md
```

Adapt names to the stack's idiom (`src/` vs package dir in Python, `lib/` in some
ecosystems) — keep the principles:

## Principles (apply to any stack)

1. **Feature-first.** New feature → new folder under `features/`. Never grow a
   flat `utils/`, `helpers/`, or `components/` dumping ground at the root.
2. **`shared/` is earned, not default.** Code goes to `shared/` only when a
   second feature actually needs it. Don't design "for reuse" speculatively.
3. **One composition root.** Wiring (routes, DI, config, app startup) lives in
   one place; features never import each other directly — they meet at the root
   or through `shared/`.
4. **Tests live next to what they test.** Feature tests inside the feature;
   only cross-feature/e2e suites at the top level.
5. **File size guard.** A file approaching ~300 lines is a smell; split by
   responsibility before it grows past it.
6. **Naming.** Folders: kebab-case. Files: follow the stack's dominant idiom
   (PascalCase components in React, snake_case in Python, ...). Consistency
   inside the project beats any global rule.
7. **No parallel structures.** One styling system, one test runner, one state
   approach. Adding a second of any of these is an architectural decision that
   needs planning-phase approval, never an implementation-time convenience.
