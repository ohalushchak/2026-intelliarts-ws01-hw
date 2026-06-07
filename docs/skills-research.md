# Agent Skills Research

WS1 Task 2 — identifying the **recurring, multi-step workflows** in this Excalidraw monorepo that are good candidates to encode as Agent Skills (reusable, parameterized procedures an AI agent follows). Each candidate below is grounded in the actual code: the files it touches and the verification step. Companion docs: [`AGENTS.md`](../AGENTS.md), [`docs/technical/architecture.md`](technical/architecture.md).

## How candidates were chosen

A workflow is a good skill if it is (a) **repeated** across the codebase, (b) **pattern-driven** (there's a "do it like the existing ones" template), and (c) has a **clear verification** (a test or command that confirms success). The `actions/`, `packages/element/src/`, and `data/` directories each contain many near-identical files, which is exactly the signal that a skill would pay off.

---

## Skill 1 — "Add an Excalidraw action" (★ highest value)

**Why:** Actions are the single most repeated pattern in the editor. There are ~40 `action*.ts(x)` files in [`packages/excalidraw/actions/`](../packages/excalidraw/actions/), all built the same way. Every new command, toolbar toggle, or property control is one. Doing it by hand means remembering ~5 separate touch-points and the `CaptureUpdateAction` rule — easy to get subtly wrong.

**What it automates (the repeatable steps):**

1. Create `actionFoo.ts(x)` exporting `register({ name, label, perform, ... })` ([`actions/register.ts`](../packages/excalidraw/actions/register.ts) appends it to the global action list).
2. `perform()` returns `{ elements?, appState?, captureUpdate }` — pick the correct `CaptureUpdateAction` (`IMMEDIATELY` for undoable, `NEVER` for ephemeral/remote; see architecture §3.3).
3. Add the action `name` to the `ActionName` union in [`actions/types.ts`](../packages/excalidraw/actions/types.ts).
4. Re-export it from [`actions/index.ts`](../packages/excalidraw/actions/index.ts).
5. Optional: `keyTest` for a shortcut (+ entry in [`actions/shortcuts.ts`](../packages/excalidraw/actions/shortcuts.ts)), `predicate` for enable/disable, `PanelComponent` for UI.

**Inputs:** action name, trigger (menu/shortcut/panel), whether it's undoable.

**Verification:** add a test next to existing ones (e.g. `actionFlip.test.tsx`, `actionElementLock.test.tsx`) and run `yarn test`.

**Covers the WS1 Task 3 capability:** _new keyboard shortcut_ and _toolbar option_.

---

## Skill 2 — "Add a primitive shape/element type"

**Why:** Adding a shape is the most complex recurring change because the element model is spread across the `@excalidraw/element` package and the editor. It's error-prone to do from memory, and it maps directly to a WS1 Task 3 capability ("new primitive shape").

**What it automates (touch-points, verified):**

1. Extend the element type union and add a type guard in [`packages/element/src/types.ts`](../packages/element/src/types.ts) and [`typeChecks.ts`](../packages/element/src/typeChecks.ts).
2. Constructor in [`newElement.ts`](../packages/element/src/newElement.ts).
3. Geometry: bounds in [`bounds.ts`](../packages/element/src/bounds.ts), hit-testing in [`collision.ts`](../packages/element/src/collision.ts).
4. Rendering: the RoughJS shape in `_generateElementShape` ([`shape.ts`](../packages/element/src/shape.ts)) and the canvas/SVG paths in [`renderElement.ts`](../packages/element/src/renderElement.ts).
5. Tooling: register the tool in [`components/shapes.tsx`](../packages/excalidraw/components/shapes.tsx) (`SHAPES`) and wire pointer creation in `App.tsx`.
6. Respect the **layering rule** (geometry in `@excalidraw/math`/`element`, never import `@excalidraw/excalidraw` from lower packages — see `AGENTS.md`).

**Verification:** model new tests on [`tests/dragCreate.test.tsx`](../packages/excalidraw/tests/dragCreate.test.tsx) / `multiPointCreate.test.tsx`, using the helpers in [`tests/helpers/ui.ts`](../packages/excalidraw/tests/helpers/ui.ts) and `api.ts`; run `yarn test`.

---

## Skill 3 — "Run & write tests the Excalidraw way" (everyday workflow)

**Why:** Every change should ship with a passing test, and the repo has strong conventions and ready-made helpers that an agent should use instead of reinventing setup. This is the most _frequently_ invoked workflow.

**What it automates:**

- Choosing the run command: `yarn test` (watch) vs `yarn test:app --watch=false` (single run) vs `yarn test:update` (snapshots) vs the full gate `yarn test:all` (typecheck + eslint + prettier + tests).
- Scaffolding a `*.test.ts(x)` file (colocated or under a package `tests/` dir) using the shared helpers in [`tests/helpers/`](../packages/excalidraw/tests/helpers/) (`ui.ts`, `api.ts`, `mocks.ts`) and Vitest globals configured in [`setupTests.ts`](../setupTests.ts).
- Reminding the agent to run the **full gate before declaring done** (CI parity).

**Verification:** the command exit codes themselves; `yarn test:all` green.

---

## Skill 4 — "Add an export/import format" (optional, scoped)

**Why:** Export/import is a self-contained subsystem in [`packages/excalidraw/data/`](../packages/excalidraw/data/) + [`scene/export.ts`](../packages/excalidraw/scene/export.ts), and it's a WS1 Task 3 capability. A skill keeps a new format consistent with the existing serialize/blob/image paths and reuses the shared renderers rather than drawing ad hoc.

**What it automates:** add the serializer/deserializer alongside [`data/json.ts`](../packages/excalidraw/data/json.ts), [`data/blob.ts`](../packages/excalidraw/data/blob.ts), [`data/image.ts`](../packages/excalidraw/data/image.ts); wire the export action in [`actions/actionExport.tsx`](../packages/excalidraw/actions/actionExport.tsx); reuse `exportToCanvas`/`renderSceneToSvg` with the `isExporting` flag (architecture §5.4).

**Verification:** extend [`tests/export.test.tsx`](../packages/excalidraw/tests/export.test.tsx) / `tests/scene/export.test.ts`; run `yarn test`.

---

## Recommendation

| Skill | Recurrence | Complexity removed | Priority |
| --- | --- | --- | --- |
| 1. Add an action | Very high (~40 examples) | Medium | **Build first** |
| 2. Add a shape | Medium | High (cross-package) | High |
| 3. Run/write tests | Every change | Low–Medium | High (foundational) |
| 4. Export format | Low | Medium | Optional |

**Build first:** Skill 1 (highest recurrence, directly enables Task 3's shortcut/toolbar capabilities), paired with Skill 3 (tests are the verification gate for every other skill).

A concrete skill could be added under `.claude/skills/<name>/SKILL.md` (Claude Code) or `.cursor/skills/` (Cursor); this document is the research/justification step.
