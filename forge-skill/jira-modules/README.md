# Jira Forge Modules

## Overview

Jira Forge apps extend Jira's functionality through workflow integration, UI extensions, and event handling.

## Available Module Types

### Workflow Integration Modules

| Module | Description |
|--------|-------------|
| **[workflow-validators.md](./workflow-validators.md)** | Validate issue fields during transitions |
| **[workflow-conditions.md](./workflow-conditions.md)** | Control visibility of transitions |
| **[workflow-post-functions.md](./workflow-post-functions.md)** | Execute logic after successful transition |

### Module Properties

All modules share these required properties:

```yaml
modules:
  jira:someModule:
    - key: unique-key             # Required: alphanumeric identifier
      name: Display Name          # Required: UI display name
      description: Description    # Required: Module description
```

### Common Optional Properties

| Property | Type | Description |
|----------|------|-------------|
| `function` | string | Reference to function handler |
| `expression` | string | Jira expression (alternative to function) |
| `errorMessage` | string | Custom error message on failure |
| `projectTypes` | array | Allowed project types |

### UI Configuration

Workflow modules support custom configuration interfaces:

```yaml
modules:
  jira:workflowValidator:
    - key: my-validator
      name: My Validator
      description: Validates fields
      
      create:
        resource: config-ui
      edit:
        resource: config-ui
      view:
        resource: view-panel
```

## Permissions Required

Most Jira modules require these scopes:

```yaml
permissions:
  scopes:
    - read:jira-work       # View issues and workflows
    - read:workflow:jira   # Access workflow configuration
```

For write operations (post functions):

```yaml
permissions:
  scopes:
    - write:jira-work      # Update issues
```

## Event Flow

1. **User action** (transition, issue creation)
2. **Module triggers** (condition/validator/function)
3. **Context provided** (issue data, transition info)
4. **Response returned** (pass/fail for validators)

## Next Steps

- Read [getting-started.md](../01-getting-started.md) for core concepts
- Review [events-payloads.md](../06-events-payloads.md) for trigger data
- Check [api-endpoints.md](../07-api-endpoints.md) for Jira REST API calls