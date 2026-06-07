# Technical Context

## Tech Stack

- **Language**: TypeScript
- **Frontend Framework**: React
- **Build Tool / Bundler**: Vite
- **Testing**: Vitest
- **Package Manager**: Yarn (version 1.22.x) with Yarn Workspaces
- **Core Dependencies**: RoughJS, Jotai

## Monorepo Structure

The project uses Yarn Workspaces to split the application into logical boundaries:

- `excalidraw-app/`: The standalone web application.
- `packages/excalidraw/`: The main React component and core logic.
- `packages/element/`: Element definitions and handlers.
- `packages/math/`: Geometry and mathematical utilities.
- `packages/common/`: Shared configuration and types.

## Development Setup

### Installation

Run the following in the root directory to install dependencies and validate workspaces:

```bash
yarn clean-install
# or simply
yarn install
```

### Running Locally

To start the Vite development server for the main app:

```bash
yarn start
```

_(This maps to `yarn --cwd ./excalidraw-app start`)_

### Testing

To run the Vitest suite:

```bash
yarn test
```

### Building

To build the application for production:

```bash
yarn build
```

_(Maps to `yarn --cwd ./excalidraw-app build`)_
