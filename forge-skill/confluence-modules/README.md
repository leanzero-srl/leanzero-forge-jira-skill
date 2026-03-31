# Confluence Forge Modules

## Overview

Confluence Forge apps extend Confluence's functionality through macros, pages, and UI modifications.

## Available Module Types

| Module | Description |
|--------|-------------|
| **[macro.md](./macro.md)** | Create embedded content macros |
| **[full-page.md](./full-page.md)** | Add full page views to Confluence |

## Core Concepts

### Macros

Macros embed custom content into Confluence pages. They can be:
- **Native UI Kit**: Use Forge UI components
- **Custom UI**: Render HTML/React apps

### Full Page Modules

Full page modules create standalone pages accessible from the Confluence sidebar.

## Configuration Example

```yaml
modules:
  macro:
    - key: my-macro
      resource: main
      render: native
      resolver:
        function: resolver
      title: My Macro
      description: Inserts custom content
      
  confluence:fullPage:
    - key: my-full-page
      resource: fullpage
      routePrefix: myapp
      title: My App
  
  function:
    - key: resolver
      handler: index.handler

resources:
  - key: main
    path: src/frontend/index.jsx
```

## Permissions Required

```yaml
permissions:
  scopes:
    - read:confluence-content
    - write:confluence-content  # For updates
```

## Next Steps

- Read [getting-started.md](../01-getting-started.md) for core concepts