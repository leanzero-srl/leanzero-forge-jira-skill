# Jira Workflow Post Function Module

## Overview

The `jira:workflowPostFunction` module allows Forge apps to execute logic after a workflow transition completes successfully. Post functions run in sequence and can modify issues, create related issues, send notifications, or call external APIs.

**Key Difference**: Post functions execute AFTER the transition is complete (unlike validators/conditions).

## Module Configuration

```yaml
modules:
  jira:workflowPostFunction:
    - key: my-post-function          # Required: Unique identifier
      name: Create Follow-up Issue   # Required: Display name
      description: Creates a follow-up issue after transition # Required: Description
      
      function: createFollowUp        # Function to execute
      
      projectTypes:
        - company-managed
        - team-managed
        
      create:
        resource: postfunction-config
      edit:
        resource: postfunction-config
      view:
        resource: postfunction-view
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
| `function` | string | Reference to function handler (required for function-based) |
| `projectTypes` | array | Allowed project types: `company-managed`, `team-managed` |
| `create`/`edit`/`view` | object | UI resource references |

## Function-Based Post Function

```yaml
modules:
  jira:workflowPostFunction:
    - key: ai-summary-enhancer
      name: AI Summary Enhancer
      description: Enhances issue summary using AI after transition
      function: enhanceSummary
      
      create:
        resource: config-ui
```

Function implementation:

```javascript
import api, { route } from '@forge/api';

export const enhanceSummary = async (args) => {
  const { issue, configuration } = args;
  
  // Configuration contains what user set in UI
  const promptTemplate = configuration.promptTemplate || 
    "Improve this summary for clarity: {summary}";
  
  // Get original summary
  const originalSummary = issue.fields.summary;
  
  try {
    // Call external AI service
    const response = await api.asApp().requestJira(route`/rest/api/3/myself`);
    
    // Call your external API (configure in manifest.external.fetch)
    // const aiResponse = await fetch('https://api.yourservice.com/enhance', {...});
    
    const enhancedSummary = `ENHANCED: ${originalSummary}`;
    
    // Update the issue with enhanced summary
    await api.asApp().requestJira(route`/rest/api/3/issue/${issue.id}`, {
      method: 'PUT',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        fields: {
          summary: enhancedSummary
        }
      })
    });
    
    return { result: true };
  } catch (error) {
    console.error('Post function error:', error);
    return { 
      result: false,
      errorMessage: `Enhancement failed: ${error.message}` 
    };
  }
};
```

## Event Payload

When the post function triggers:

```javascript
{
  issue: {
    id: "12345",
    key: "PROJ-123",
    fields: {
      summary: "Original title",
      description: "Original description...",
      customfield_10001: "some value"
    }
  },
  transition: {
    id: "11",
    name: "In Progress",
    from: { id: "1" },
    to: { id: "2" },
    executionId: "6540951a-7c88-4620-835b-61aab8bbb13e"
  },
  workflow: {
    id: "workflow-uuid",
    name: "Software Workflow"
  },
  changelog: [
    {
      field: "status",
      from: "To Do (1)",
      to: "In Progress (2)"
    }
  ],
  configuration: {
    relatedProjectKey: "OPS",
    issueType: "Task",
    enabled: true
  }
}
```

## Response Format

### Success (Continue workflow)

```javascript
return { result: true };
```

### Failure (Log error but don't block)

```javascript
return { 
  result: false,
  errorMessage: "External API call failed - continuing anyway" 
};
```

**Note**: Post function failures typically log errors but don't roll back transitions (unlike validators).

## Configuration Persistence

```javascript
import { workflowRules } from '@forge/jira-bridge';

const onConfigureFn = async () => {
  return JSON.stringify({
    relatedProjectKey: document.getElementById('project').value,
    issueType: document.getElementById('issuetype').value,
    assignee: document.getElementById('assignee').value,
    summaryTemplate: "Follow-up from {parent}: {summary}"
  });
};

await workflowRules.onConfigure(onConfigureFn);
```

## Permissions Required

```yaml
permissions:
  scopes:
    - read:jira-work        # View issue data
    - write:jira-work       # Update issues (for modifying fields)
    - read:workflow:jira    # Access workflow info
    
external:
  fetch:
    backend:
      - "api.yourservice.com"   # External API domains
```

## Common Use Cases

1. **Create Related Issues**: Automatically create subtasks or linked issues
2. **Update Fields**: Modify issue fields based on transition context
3. **External Integration**: Call external APIs to synchronize data
4. **Notification**: Send notifications via email/webhook/slack
5. **Audit Logging**: Log transition events to external systems

## Post Function Execution Order

Post functions execute in the order they're configured in Jira's workflow editor. Your post function should be placed after transitions complete.

Common execution sequence:
1. Workflow engine processes transition
2. Update issue fields (if any)
3. **Forge post function executes**
4. Send notifications
5. Log events

## Error Handling Best Practices

```javascript
export const myPostFunction = async (args) => {
  try {
    // Do work
    return { result: true };
  } catch (error) {
    console.error('[MyPostFunction] Error:', error);
    
    // Don't block transition for non-critical errors
    return { 
      result: true,
      warnings: [
        "External API returned warning - continuing"
      ]
    };
  }
};
```

## Testing Post Functions

1. **Test in development**: Use `forge tunnel` for local testing
2. **Log extensively**: Check `forge logs` for output
3. **Handle errors gracefully**: Ensure failures don't break workflows
4. **Limit external calls**: Be mindful of API rate limits

## Next Steps

- **Workflow Validators**: Validate data before transition completes
- **Events & Payloads**: Understand what data is available
- **Jira REST APIs**: Learn how to modify issues and call other endpoints