# System Patterns

## Architecture Patterns

### Component Hierarchy

- The UI is built entirely in React.
- The `App` component acts as the main controller and state container.
- It consumes the `<Excalidraw>` component, which in turn manages its own internal canvas and toolsets.

### State Management Separation

- **`AppState`**: Represents the UI configuration, such as the currently selected tool, zoom level, background color, and theming. It's highly dynamic and often relies on Jotai atoms or React Context.
- **Scene Elements**: Represents the actual drawing data. Elements are treated immutably; any change to a shape creates a new instance of that shape. This makes implementing undo/redo (`history.ts`) straightforward.

### Action System

Actions (like copy, paste, group, align) are encapsulated in the `actions/` directory. Each action defines:

- A `name` and `perform` function.
- `trackEvent` for analytics.
- Key bindings (`keyTest`) and panel placement definitions. This pattern allows the UI to automatically render toolbar buttons and keyboard shortcuts based on available action objects.

### Rendering Strategy

- **HTML5 Canvas**: Used for high-performance rendering of many shapes.
- **RoughJS Integration**: Used to process standard geometric data into "sketchy" paths.
- **Render Caching**: Excalidraw avoids clearing and redrawing the entire canvas on every frame. It selectively re-renders based on visibility, zoom, and whether elements have been modified.

### Event Handling

Pointer events (mouse, touch, pen) are highly normalized to manage cross-device compatibility. Gestures (like pinch-to-zoom) are handled via custom logic to manipulate the viewport metrics inside `AppState`.
