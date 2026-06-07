## Context

The current generic-shape pipeline assumes three first-class shapes: `rectangle`, `diamond`, and `ellipse`. That assumption appears in the element/tool unions, tool registries, conversion popup, text-container helpers, binding logic, snapping, collision/hit-testing, and the shared rendering/export path (`ShapeCache` plus canvas/SVG renderers). A triangle feature therefore is not a single rendering change; it needs a coordinated update across the generic element model and the geometry helpers that existing editing flows depend on.

The good news is that the architecture already centralizes most shape behavior. `ShapeCache` drives both canvas and SVG export, restore treats generic elements uniformly, and there is already triangle geometry support in `packages/math/src/triangle.ts`. The design should extend those shared seams instead of introducing one-off UI or export logic.

## Goals / Non-Goals

**Goals:**

- Add `triangle` as a first-class generic element/tool without creating a separate element data model.
- Make triangles available anywhere users currently expect built-in generic shapes: toolbar, mobile picker, help dialog, restore, and convert-type UI.
- Reuse shared rendering/export infrastructure so triangle visuals stay consistent between live canvas and exported SVG/PNG.
- Support the expected editing affordances: selection, resize, snapping, collision/hit-testing, bound text, and arrow binding.
- Keep existing shortcuts stable for current tools.

**Non-Goals:**

- Arbitrary polygon or n-gon support.
- Converting existing line polygons into triangles automatically.
- A brand-new text layout engine for arbitrary polygons.
- Reworking the generic-shape toolbar layout beyond what triangle itself requires.

## Decisions

### 1. Model triangle as another generic element type

Add `triangle` to the existing generic element/tool unions instead of creating a bespoke element schema.

- **Why:** Triangles can use the same persisted fields as other generic shapes (`x`, `y`, `width`, `height`, `angle`, styling, bindings).
- **Alternative considered:** Represent triangle as a special polygon/line preset. Rejected because it would miss the generic-shape behaviors the issue explicitly asks for (bound text, convert-type, restore parity, arrow binding).

### 2. Use one canonical triangle geometry everywhere

Define triangle vertices from the element bounding box as an upright isosceles triangle (`top-center`, `bottom-right`, `bottom-left`), then rotate through the existing element angle.

- **Why:** A single canonical geometry can feed rendering, hit-testing, snapping outlines, and binding math.
- **Alternative considered:** Separate per-subsystem implementations (one for render, another for binding, another for export). Rejected because shape drift would be likely.
- **Implementation note:** Keep reusable point/triangle math in `@excalidraw/math` where practical, and keep element-specific adapters in `packages/element`.

### 3. Reuse the shared renderer/export path

Teach `ShapeCache.generateElementShape()` and SVG scene rendering about `triangle`, rather than adding export-only drawing logic.

- **Why:** The architecture already guarantees parity between live rendering and export by reusing the same shape generation path.
- **Alternative considered:** Draw triangles directly in SVG/canvas renderers. Rejected because it duplicates style handling and cache invalidation rules.

### 4. Add a dedicated alpha shortcut without renumbering existing numeric shortcuts

Use `Y` as the direct keyboard shortcut for triangle and do not shift the existing numeric shortcuts for downstream tools.

- **Why:** The issue asks for a shortcut, but renumbering arrow/line/freedraw/text/image/eraser would create avoidable churn for existing users.
- **Alternative considered:** Insert triangle into the numeric sequence and shift later tools. Rejected because it changes established muscle memory for unrelated tools.

### 5. Bound text uses a conservative triangle-specific safe box

Implement triangle text binding via a deterministic inscribed text box derived from the triangle bounds, and position bound text within that safe region.

- **Why:** This matches the existing rectangle/diamond/ellipse approach, keeps the implementation testable, and avoids building polygon-aware text layout.
- **Alternative considered:** Full polygon text layout/clipping. Rejected as too large for the requested feature.
- **Trade-off:** The usable text area may be slightly smaller than the mathematically maximal region if we bias toward visual centering and safety.

### 6. Extend binding/snapping with triangle-aware edge classification

Update the fixed-point and binding-side helpers so triangles expose stable edge/vertex targets for arrows, snapping, and elbow routing.

- **Why:** Arrow binding depends on more than simple intersection tests; the existing binding system also needs side classification and side midpoint lookup.
- **Alternative considered:** Treat triangle as a generic polygon with intersections only. Rejected because it would not be enough for side-based routing behaviors.

## Risks / Trade-offs

- **Triangle text box feels visually low or too small** → Start with conservative formulas plus focused tests; iterate on placement constants if manual QA shows poor centering.
- **Binding logic misses an edge/vertex case for elbow arrows** → Reuse existing adaptive-side plumbing, add targeted tests for each major attachment region, and verify with rotated triangles.
- **Shape-specific branches continue to spread** → Centralize new vertex helpers and reuse them across shape, bounds, collision, and binding code instead of duplicating point formulas.
- **Toolbar discoverability vs. shortcut stability tension** → Keep the direct alpha shortcut, but avoid renumbering existing tools.

## Migration Plan

No data migration is expected. Persisted scenes will start accepting and restoring the new `triangle` type once restore/type guards are updated. Older scenes remain unchanged.

## Open Questions

- Should the first iteration support triangle corner roundness, or should triangle launch as a sharp-corner generic shape only?
- Do we want triangle included anywhere flowchart-node helpers currently assume rectangle/diamond/ellipse, or should the first cut stay scoped to the issue requirements only?
