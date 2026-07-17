# Frontend Design — Team Extras

This file EXTENDS the bundled `frontend-design` skill (Anthropic, Apache 2.0).
The skill governs aesthetic quality and distinctiveness; this file governs
consistency, language, and how identity persists across tasks. Where they seem
to conflict, this file wins — consistency beats novelty inside one product.

## 1. Persistent design system (per project)

- The project's visual identity lives in **`design-system.md` at the project
  root** — palette (hex), type scale, spacing scale, radii, shadows, and the
  names of the token variables that implement them.
- **First UI task** in a project: design-lead creates the full identity (using
  the frontend-design skill at full freedom) and it is saved as
  `design-system.md`.
- **Every later UI task**: `design-system.md` WINS. The task's `design.md`
  builds on it and may only add missing tokens (e.g. a first error color) —
  never redefine existing ones. Changing the identity itself is a scope
  decision that needs explicit human approval.
- Tokens are locked values, not vibes: every color/size decision in design.md
  must resolve to a named token with a concrete value. "Warm accent" is not a
  spec; `--accent: #C4552D` is.

## 2. Language & direction

- **Default product language is English** unless the human chose otherwise in
  clarification (the coordinator always asks; see run-task Step 0).
- Every additional language the human selected must be reflected in the UI
  work: text externalized (no hard-coded user-facing strings when i18n is in
  scope), locale-aware dates/numbers.
- **RTL rules apply only when an RTL language (Arabic, Hebrew, Farsi, ...) is
  in the language list:** use logical CSS properties
  (`margin-inline-start`, `padding-inline-end`, `text-align: start`) instead
  of left/right, mirror directional icons, verify layout in both directions.
- Arabic type: prefer faces with proper Arabic support (e.g. IBM Plex Sans
  Arabic, Noto Kufi/Naskh Arabic, Rubik, Cairo); never fake-bold or
  letter-space Arabic script.

## 3. Non-negotiable quality baseline

- Contrast: WCAG AA minimum for text and interactive elements.
- Every interactive element has visible hover AND focus states; focus is never
  `outline: none` without a replacement.
- Responsive: layout holds at 360px, 768px, 1280px; no horizontal scroll at
  any of them.
- Interactive states covered where relevant: hover / focus / disabled / empty /
  error / loading.
- Motion respects `prefers-reduced-motion`.

## 4. Implementation discipline

- All styling flows through the project's token/styling system — no inline
  style dumps, no second styling system.
- New components must check `design-system.md` and the existing component
  inventory before being created — reuse and extend before rebuild.
