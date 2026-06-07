# AGENTS.md

Project-specific operating rules for AI coding agents working in this repository. These rules are derived from the actual config and codebase — follow them exactly. For deeper context read [`docs/technical/architecture.md`](docs/technical/architecture.md) and the memory bank in [`docs/memory/`](docs/memory/).

## What this project is

Excalidraw — a TypeScript **Yarn-1 workspaces monorepo**. A deployable app (`excalidraw-app/`) consumes an embeddable, environment-agnostic editor library (`@excalidraw/excalidraw`), which is built on the layered packages under `packages/*`. See architecture doc for the full picture.

## Setup & commands

- **Package manager: Yarn 1** (`yarn@1.22.22`). Do **not** use `npm` or `pnpm`, and do not add/commit a `package-lock.json` (it's git-ignored). Node `>=18` (the workshop targets Node 22+).
- Install: `yarn install` (or `yarn clean-install` to reset `node_modules`).
- Run the app: `yarn start` (Vite dev server for `excalidraw-app`).
- Tests: `yarn test` (Vitest, watch) / `yarn test:app --watch=false` (single run) / `yarn test:update` (update snapshots).
- Lint: `yarn test:code` (ESLint, `--max-warnings=0`) · fix with `yarn fix:code`.
- Format: `yarn test:other` (Prettier check) · fix with `yarn fix:other`.
- Typecheck: `yarn test:typecheck` (`tsc`, `noEmit`).
- **Full gate (run before declaring work done):** `yarn test:all` (= typecheck + eslint + prettier + tests). This is also what CI expects.
- Build packages in dependency order: `yarn build:packages`. App build: `yarn build`.

## Import boundaries (enforced by ESLint — treat as hard rules)

These come from [`.eslintrc.json`](.eslintrc.json) and [`packages/eslintrc.base.json`](packages/eslintrc.base.json). Violations are **errors**, not warnings.

1. **Never import `jotai` directly.** Use `editor-jotai` (inside `packages/excalidraw`) or `app-jotai` (inside `excalidraw-app`). Jotai is isolated per-editor on purpose (see architecture §3.5).
2. **No barrel / index imports inside `packages/excalidraw/`.** Import the specific module with a direct relative path, not from `.`, `..`, `../index`, or the `@excalidraw/excalidraw` barrel. (Type-only imports from the barrel are allowed.)
3. **Cross-package imports must respect the dependency layering.** Use the package aliases (`@excalidraw/common`, `@excalidraw/math`, `@excalidraw/element`, …; see `tsconfig.json` `paths`). Do **not** introduce upward/cyclic edges. The allowed direction is: `common ← math ← element ← excalidraw`; `fractional-indexing` and `utils` are standalone. A lower-layer package importing a higher one (e.g. `math` importing from `element`, or any package importing runtime code from `@excalidraw/excalidraw`) is forbidden — only **type** imports are tolerated where the base config allows.
4. **Use `import type` for type-only imports** (`consistent-type-imports` is an error; fix style is _separate_ type imports). Keep `import/order` groups tidy (`@excalidraw/**` sorts after other externals).

## Code style

- Enforced by `.editorconfig` + Prettier: UTF-8, **LF** line endings, **2-space** indent, final newline, no trailing whitespace. Don't fight the formatter — run `yarn fix`.
- TypeScript is `strict`. Don't weaken types with `any` or `// @ts-ignore` to make errors disappear; fix the underlying type. Match the surrounding code's idioms.
- A husky `pre-commit` hook runs `lint-staged` (ESLint `--fix` + Prettier) on staged files. Do not bypass hooks (`--no-verify`) unless the user explicitly asks.

## Architectural conventions (follow existing patterns)

- **State lives in two places, not one:** `AppState` (UI/viewport) vs the `Scene` (document elements). Mutate elements via `scene.mutateElement` / `replaceAllElements`, never by hand-editing arrays. Don't store element data in `AppState` — reference by id.
- **Durable changes go through the `Store`/delta engine.** When adding an action, return the correct `CaptureUpdateAction` (`IMMEDIATELY` for undoable edits, `NEVER` for remote/init, `EVENTUALLY` for async multi-step). Undo/redo and collaboration depend on this — don't bypass it with ad-hoc history handling.
- **New user operations are `Action` objects** (`packages/excalidraw/actions/`) with `name` / `perform` / `captureUpdate`, registered via the `ActionManager` — this is how toolbar buttons and shortcuts are generated. Prefer this over bespoke handlers.
- **Rendering is event-driven** (RAF-throttled, three layered canvases). Don't add continuous render loops. New element rendering should reuse `ShapeCache` and the per-element canvas cache rather than drawing ad hoc.
- New geometry helpers belong in `@excalidraw/math`; shared constants/utils in `@excalidraw/common`; element logic in `@excalidraw/element`.

## Testing conventions

- Vitest, with `vitest/globals` and `@testing-library/jest-dom` (see `tsconfig.json`, `setupTests.ts`).
- Co-locate unit tests as `*.test.ts` / `*.test.tsx` next to the code, or under a package's `tests/` directory (both patterns exist in `packages/excalidraw/`).
- Any new capability (e.g. a new shape, shortcut, or export format) **must ship with at least one test**, and `yarn test` must pass.

## Documentation upkeep

After every feature or behavior change, update the docs **in the same change** — but only with information that is durable and relevant to the file you touch. Treat docs as a reflection of current truth, not a log of activity.

- **Update only what changed in behavior or design.** Pure refactors, renames, bug fixes, and test-only changes usually need **no** doc edit. If a change has no doc-relevant impact, skip it (don't pad files to "prove" work). Conversely, if you change behavior, the relevant doc must not be left stale.
- **Route each fact to the one file that owns it** — don't duplicate across files:
  - [`docs/technical/architecture.md`](docs/technical/architecture.md) — new/changed modules, data flow, package boundaries, or layering. Keep it **code-verified**: cite real paths/symbols inline and match the existing section style.
  - [`docs/memory/systemPatterns.md`](docs/memory/systemPatterns.md) — a new or changed _convention/pattern_ (e.g. how actions, state, or rendering are structured).
  - [`docs/memory/techContext.md`](docs/memory/techContext.md) — stack, tooling, scripts, or dependency changes.
  - [`docs/memory/projectbrief.md`](docs/memory/projectbrief.md) — scope/goals shifts only.
- **Do not** turn docs into a changelog or PR diary, restate code line-by-line, or add speculative/“future work” notes. No dates, task IDs, or "recently added" phrasing — write in the present tense as if the feature has always existed.
- **Do not** edit [`docs/walkthrough.md`](docs/walkthrough.md) (workshop instructions) for feature work. The per-change spec under `openspec/changes/<name>/` is the record of _why/what_; the docs above capture the resulting _current state_.
- Keep edits minimal and grounded — verify each claim against the code before writing it.

## Do / Don't

**Do**

- Cross-check every claim and change against the actual code before acting.
- After a feature/behavior change, update the owning doc per **Documentation upkeep** — relevant facts only, no changelog noise.
- Keep changes scoped to the right workspace package per the layering above.
- Run `yarn test:all` before considering a task complete.

**Don't**

- Don't edit generated/vendored output: `dist/`, `build/`, `coverage/`, `node_modules/`, `*.min.*`, snapshots — change the source instead.
- Don't add new runtime dependencies without a clear need; prefer existing utilities in `@excalidraw/common` / `@excalidraw/math`.
- Don't commit, push, or open PRs unless explicitly asked.
- Don't disable lint/type rules to silence errors.
