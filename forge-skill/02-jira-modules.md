# Jira Forge Modules Reference

## Overview

Forge provides several module types for extending Jira functionality. Each module type serves a specific purpose in the workflow and issue management lifecycle.

## Module Types by Category

### Workflow Integration

| Module | Description | Docs |
|--------|-------------|------|
| `jira:workflowValidator` | Validate issue fields during transitions | [Workflow Validators](./jira-modules/workflow-validators.md) |
| `jira:workflowCondition` | Control visibility of transitions | [Workflow Conditions](./jira-modules/workflow-conditions.md) |
| `jira:workflowPostFunction` | Execute logic after successful transition | [Workflow Post Functions](./jira-modules/workflow-post-functions.md) |

### Admin & Configuration

| Module | Description | Docs |
|--------|-------------|------|
| `jira:adminPage` | Create admin configuration pages | [Admin Pages](./jira-modules/admin-pages.md) |
| `jira:issueTracker` | Add custom issue trackers to Jira | - |

### Event-Driven

| Module | Description | Docs |
|--------|-------------|------|
| `trigger` with `avi:jira:*` events | Execute on issue events (create, update, delete) | [Events & Payloads](./../06-events-payloads.md) |

### UI Extensions

| Module | Description | Docs |
|--------|-------------|------|
| `jira:issueViewPanel` | Add panels to issue views | - |
| `jira:projectSettingsPage` | Add project settings pages | - |

## Common Module Properties

All modules share these core properties:

```yaml
modules:
  jira:someModule:
    - key: unique-key-123          # Required: alphanumeric with hyphens/underscores
      name: Display Name           # Required: What users see in UI
      description: Description text # Required: Explains functionality
      function: resolver-name       # Optional: Function to execute
```

### Project Type Restrictions

```yaml
projectTypes:
  - company-managed    # Jira Cloud premium
  - team-managed       # Jira Core/Standard
```

## Resolver Configuration

For modules requiring UI configuration:

```yaml
modules:
  jira:workflowValidator:
    - key: my-validator
      name: My Validator
      description: Validates issue fields
      function: validate
      resolver:
        function: resolver  # Links to function in manifest
      
      create:             # UI for creating new instance
        resource: config-ui
      edit:               # UI for editing existing
        resource: config-ui
      view:               # Read-only summary
        resource: view-panel
```

## Permissions Required

Most Jira modules require these scopes:

```yaml
permissions:
  scopes:
    - read:jira-work       # View issues and workflow
    - read:workflow:jira   # Access workflow configuration
    - read:project:jira    # Access project data
    - storage:app          # Persist app data
```

For write operations (post functions updating fields):

```yaml
permissions:
  scopes:
    - write:jira-work      # Update issues
```

## Event Flow for Workflow Modules

1. **User initiates transition** in Jira workflow editor or issue view
2. **Create/Edit/View UI renders** using configured resources
3. **Configuration saved** via `workflowRules.onConfigure()` bridge API
4. **Function triggers** during actual workflow execution
5. **Result returned** - pass/fail for validators/conditions

## Configuration Persistence

Workflow modules persist configuration via the Forge Jira bridge:

```javascript
import { workflowRules } from '@forge/jira-bridge';

const onConfigureFn = async () => {
  const config = {
    fieldId: 'description',
    prompt: 'Validate content quality',
    enabled: true
  };
  return JSON.stringify(config);
};

await workflowRules.onConfigure(onConfigureFn);
```

The configuration is available in the function context as `config`.

## Next Steps

- **Detailed Module Docs**: See individual files in `jira-modules/` directory
- **Events**: Review [Events & Payloads](./../06-events-payloads.md) for trigger data
- **API Calls**: Learn about making REST API calls in [API Endpoints](./../07-api-endpoints.md)