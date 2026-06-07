## Why

Excalidraw has first-class primitives for rectangle, diamond, and ellipse, but triangles still require line/polygon workarounds that do not participate in the same editing, binding, text-container, conversion, and export flows. Adding a native triangle fills a common diagramming gap and makes the generic shape model more complete.

## What Changes

- Add `triangle` as a first-class generic element and tool type throughout the editor model.
- Expose triangle in desktop and mobile shape pickers, help/shortcut surfaces, and generic shape conversion flows.
- Render triangles through the shared `ShapeCache` canvas/SVG/export pipeline.
- Support selection, resize, restore, collision/hit-testing, snapping, and generic type conversion for triangles.
- Support bound text inside triangles and arrow binding to triangle edges.
- Add tests for creation, restore, conversion, text layout, binding, and export behavior.

## Capabilities

### New Capabilities

- `triangle-shape`: Add a native triangle shape that behaves like other generic diagram shapes across creation, editing, text binding, arrow binding, restore, and export.

### Modified Capabilities

_None._

## Impact

- `packages/common/src/constants.ts`
- `packages/element/src/{types,typeChecks,newElement,shape,bounds,collision,distance,binding,textElement,utils}`
- `packages/excalidraw/{types,data/restore.ts,renderer/staticSvgScene.ts,snapping.ts}`
- `packages/excalidraw/components/{icons.tsx,shapes.tsx,HelpDialog.tsx,MobileToolBar.tsx,ConvertElementTypePopup.tsx}`
- Tests in `packages/element/tests/` and `packages/excalidraw/tests/`
- Durable docs describing the supported shape system, if implementation changes behavior that is documented today
