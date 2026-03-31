# Jira Workflow Condition Module

## Overview

The `jira:workflowCondition` module allows Forge apps to control whether transitions are visible in the workflow. Conditions are evaluated when the transition screen renders - if false, the transition is hidden from users.

**Key Difference**: Validators block transitions (after validation), conditions hide transitions (before rendering).

## Module Configuration

```yaml
modules:
  jira:workflowCondition:
    - key: my-condition               # Required: Unique identifier
      name: Release Condition         # Required: Display name
      description: Only show release transition for managers # Required: Description
      
      function: checkRelease          # Optional: Function to execute
      expression: user.inGroup('release-managers')  # OR: Jira expression
      
      projectTypes:
        - company-managed
        - team-managed
        
      create:
        resource: condition-config
      edit:
        resource: condition-config
      view:
        resource: condition-view
```

### Required Properties

| Property | Type | Description |
|----------|------|-------------|
| `key` | string | Unique alphanumeric identifier (regex: `^[a-zA-Z0-9_-]+$`) |
| `name` | string \| i18n | Display name shown in workflow editor |
| `description` | string \| i18n | Description shown in workflow editor |

### Optional Properties

| Property | Type | Description |
|----------|------|-------------|
| `function` | string | Reference to function handler (if not using expression) |
| `expression` | string | Jira expression for simple condition |
| `projectTypes` | array | Allowed project types: `company-managed`, `team-managed` |
| `configurationDescription` | object | Dynamic configuration summary |
| `create`/`edit`/`view` | object | UI resource references |

## Two Configuration Approaches

### Approach 1: Function-Based Condition (Full Control)

```yaml
modules:
  jira:workflowCondition:
    - key: licensing-condition
      name: Licensing Check Condition
      description: Only show if app license is active
      function: checkLicense
      
      create:
        resource: config-ui
```

Function implementation:

```javascript
export const checkLicense = async (args) => {
  const { configuration, context } = args;
  
  // Check license status from context
  const isActive = context.license?.isActive === true;
  
  if (isActive) {
    return { result: true };  // Show transition
  } else {
    return { result: false }; // Hide transition
  }
};
```

### Approach 2: Jira Expression (Simple Condition)

```yaml
modules:
  jira:workflowCondition:
    - key: has-assignee-condition
      name: Must Be Assigned Condition
      description: Only show if issue is assigned
      expression: issue.assignee != null
```

## Event Payload

When the condition triggers:

```javascript
{
  issue: {
    id: "12345",
    key: "PROJ-123",
    fields: { ... }
  },
  transition: {
    id: "11",
    name: "Done",
    from: { id: "1" },
    to: { id: "3" }
  },
  workflow: {
    id: "workflow-uuid",
    name: "Software Workflow"
  },
  configuration: {
    requiredGroup: "release-managers",
    enabled: true
  }
}
```

## Response Format

### Show Transition (Condition Met)

```javascript
return { result: true };
```

### Hide Transition (Condition Not Met)

```javascript
return { result: false };
```

## UI Bridge for Configuration

```javascript
import { workflowRules } from '@forge/jira-bridge';

const onConfigureFn = async () => {
  const requiredGroup = document.getElementById('group-select').value;
  
  return JSON.stringify({
    requiredGroup,
    invertCondition: false
  });
};

await workflowRules.onConfigure(onConfigureFn);
```

## Permissions Required

```yaml
permissions:
  scopes:
    - read:jira-work
    - read:workflow:jira
    - read:user:jira  # For user/group checks
```

## Common Use Cases

1. **Licensing Check**: Only show transition if app license is active
2. **Role-Based Visibility**: Show transitions only for specific roles/groups
3. **Business Logic**: Hide transitions until prerequisites are met
4. **Feature Flags**: Conditionally enable based on configuration

## Event: Expression Evaluation Failure

```json
{
  "eventType": "avi:jira:failed:expression",
  "extensionId": "ari:cloud:ecosystem::extension/appId/envId/static/forge-condition",
  "workflowId": "workflow-uuid",
  "workflowName": "Software Workflow",
  "conditionId": "condition-key",
  "expression": "user.inGroup('nonexistent-group')",
  "errorMessages": ["Group not found"],
  "context": { ... }
}
```

## Relationship to Validators

| Aspect | Condition | Validator |
|--------|-----------|-----------|
| **When it runs** | Before transition screen renders | After user submits transition |
| **Purpose** | Hide/show transitions in UI | Validate data before completion |
| **Failure behavior** | Transition hidden from user | Transition blocked, error shown |
| **User experience** | User doesn't see the option | User sees option but gets error |

## Next Steps

- **Workflow Validators**: Validate data when transition completes
- **Workflow Post Functions**: Execute after successful transitions
- **Admin Pages**: Create configuration interfaces for complex conditions