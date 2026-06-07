## 1. Surface triangle in the generic shape model

- [x] 1.1 Add `triangle` to the shared tool/element/type registries, restore handling, and shape metadata used by the editor
- [x] 1.2 Add the triangle icon, localized label, help entry, toolbar/mobile picker entry, and `Y` keyboard shortcut without renumbering existing numeric shortcuts
- [x] 1.3 Include triangle in generic shape conversion flows and any generic-shape selection state that currently assumes rectangle/diamond/ellipse only

## 2. Implement triangle geometry and rendering

- [x] 2.1 Add canonical triangle vertex/geometry helpers and reuse them from the shared shape-generation pipeline
- [x] 2.2 Render triangles through `ShapeCache`, canvas rendering, and SVG/export rendering so live output and exports stay in sync
- [x] 2.3 Extend bounds, collision, hit-testing, and snapping helpers so triangles can be selected, resized, and snapped like other generic shapes

## 3. Add text and binding support

- [x] 3.1 Treat triangle as a valid text container and implement triangle-specific safe text-box sizing/positioning rules
- [x] 3.2 Extend arrow binding, binding-side midpoint lookup, and related outline/focus helpers so arrows can attach to triangle edges reliably
- [x] 3.3 Verify generic shape conversion preserves or recomputes bound text and bindings correctly when converting to or from triangle

## 4. Validate and document the feature

- [x] 4.1 Add targeted tests for triangle creation, restore, conversion, text layout, arrow binding, and export rendering
- [x] 4.2 Update any durable docs that describe supported shape behavior or the shared rendering pipeline if they become stale
- [x] 4.3 Run `yarn test:app --watch=false`, address failures, then run the full `yarn test:all` gate
