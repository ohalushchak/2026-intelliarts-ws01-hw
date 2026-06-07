## ADDED Requirements

### Requirement: Triangle is available as a first-class generic shape tool

The system SHALL expose triangle as a built-in generic shape alongside rectangle, diamond, and ellipse in the editor tool model and user-facing shape pickers.

#### Scenario: Select triangle from the editor UI

- **WHEN** the user opens the desktop toolbar, mobile generic-shape picker, or generic shape conversion UI
- **THEN** triangle is offered as an available generic shape option

#### Scenario: Activate triangle with the keyboard

- **WHEN** the user presses the triangle shortcut while focus is in the editor
- **THEN** the active tool switches to triangle without changing the existing numeric shortcuts of other tools

### Requirement: Triangle renders and restores like other generic shapes

The system SHALL persist, restore, and render triangle elements through the same canvas and export pipelines used by other built-in generic shapes.

#### Scenario: Restore a serialized scene with triangle elements

- **WHEN** a scene payload contains an element with type `triangle`
- **THEN** restore returns a valid triangle element instead of dropping or coercing it

#### Scenario: Export a scene containing triangles

- **WHEN** the user exports a scene containing triangle elements
- **THEN** the canvas/SVG export output includes triangles with the element's current geometry and styling

### Requirement: Triangle supports generic shape editing flows

Triangle elements SHALL support the same generic editing flows as the other built-in shape elements, including selection, resize, snapping, and type conversion.

#### Scenario: Resize a triangle

- **WHEN** the user resizes a triangle element
- **THEN** the editor updates the triangle bounds and keeps the rendered triangle aligned to the resized box

#### Scenario: Convert another generic shape into a triangle

- **WHEN** the user converts a rectangle, diamond, or ellipse into a triangle
- **THEN** the element becomes a triangle while preserving shared generic element properties that remain valid for the target type

### Requirement: Triangle supports bound text and arrow binding

Triangle elements SHALL behave as valid text containers and arrow-binding targets.

#### Scenario: Bind text to a triangle

- **WHEN** the user adds bound text to a triangle
- **THEN** the text is positioned within a triangle-specific safe text region and remains inside the shape during resize

#### Scenario: Bind an arrow to a triangle

- **WHEN** the user binds an arrow endpoint to a triangle edge
- **THEN** the endpoint remains attached to the triangle outline and updates correctly as the triangle moves or resizes
