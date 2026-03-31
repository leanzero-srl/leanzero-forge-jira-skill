# Jira Workflow Validators

## Overview

The `jira:workflowValidator` module allows Forge apps to validate issue fields during workflow transitions. Validators run before a transition completes and can reject the transition if validation fails.

## Module Configuration

```yaml
modules:
  jira:workflowValidator:
    - key: my-validator               # Required: Unique identifier
      name: Field Quality Validator   # Required: Display name
      description: Validates field content quality # Required: Description
      
      function: validate              # Optional: Function to execute
      expression: issue.summary != null  # OR: Jira expression
      
      errorMessage: "Field does not meet quality requirements"
      
      projectTypes:
        - company-managed
        - team-managed
        
      create:
        resource: validator-config
      edit:
        resource: validator-config
      view:
        resource: validator-view
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
| `expression` | string | Jira expression for simple validation |
| `errorMessage` | string \| i18n \| object | Error message on failure |
| `projectTypes` | array | Allowed project types: `company-managed`, `team-managed` |

## Two Configuration Approaches

### Approach 1: Function-Based Validation (Full Control)

```yaml
modules:
  jira:workflowValidator:
    - key: ai-content-validator
      name: AI Content Validator
      description: Validates issue content using AI
      function: validateContent
      
      create:
        resource: config-ui
```

Function implementation (`src/index.js`):

```javascript
export const validateContent = async (args) => {
  const { issue, configuration } = args;
  
  // Configuration contains what user set in UI
  const fieldId = configuration.fieldId || 'description';
  const prompt = configuration.prompt || 'Validate content quality';
  
  // Get field value and validate
  const fieldValue = await getFieldValue(issue.key, fieldId);
  
  // Call AI service or validation logic
  const isValid = await callValidationService(fieldValue, prompt);
  
  if (isValid) {
    return { result: true };
  } else {
    return { 
      result: false, 
      errorMessage: `Field validation failed: ${reason}` 
    };
  }
};
```

### Approach 2: Jira Expression (Simple Validation)

```yaml
modules:
  jira:workflowValidator:
    - key: mandatory-summary-validator
      name: Summary Required Validator
      description: Ensures summary field is not empty
      expression: issue.summary != null && issue.summary.length > 5
      errorMessage: "Summary must be at least 5 characters"
```

## Event Payload

When the validator triggers, it receives:

```javascript
{
  issue: {
    id: "12345",
    key: "PROJ-123",
    fields: {
      summary: "Bug report",
      description: "Detailed description...",
      customfield_10001: "some value"
    }
  },
  transition: {
    id: "11",
    name: "In Progress",
    from: { id: "1" },
    to: { id: "2" }
  },
  workflow: {
    id: "workflow-uuid",
    name: "Software Workflow"
  },
  configuration: {
    fieldId: "description",
    prompt: "Validate content quality",
    enabled: true
  }
}
```

## Response Format

### Success (Allow transition)

```javascript
return { result: true };
```

### Failure (Block transition)

```javascript
return { 
  result: false, 
  errorMessage: "Description must contain at least 20 characters" 
};
```

## UI Bridge for Configuration

For function-based validators, use the Jira bridge to save configuration:

```javascript
import { workflowRules } from '@forge/jira-bridge';

const onConfigureFn = async () => {
  // Get values from UI form inputs
  const fieldId = document.getElementById('field-select').value;
  const prompt = document.getElementById('prompt-input').value;
  
  return JSON.stringify({
    fieldId,
    prompt,
    enabled: true
  });
};

await workflowRules.onConfigure(onConfigureFn);
```

## Permissions Required

```yaml
permissions:
  scopes:
    - read:jira-work       # View issue data
    - read:workflow:jira   # Access workflow info
```

## Common Use Cases

1. **Content Quality Validation**: Ensure descriptions meet minimum quality standards
2. **Required Fields**: Verify required fields are populated
3. **Format Validation**: Validate email addresses, URLs, dates
4. **Business Rules**: Enforce company-specific validation rules
5. **AI-Assisted Validation**: Call external AI services for intelligent validation

## Event: Expression Evaluation Failure

When a Jira expression-based validator fails, Atlassian fires this event:

```json
{
  "eventType": "avi:jira:failed:expression",
  "extensionId": "ari:cloud:ecosystem::extension/appId/envId/static/forge-validator",
  "workflowId": "workflow-uuid",
  "workflowName": "Software Workflow",
  "validatorId": "validator-key",
  "expression": "issue.summary.length > 10",
  "errorMessages": ["Evaluation failed: expression syntax error"],
  "context": {
    "issue": { "id": "10000", "key": "TEST-1" },
    "project": { "id": "10000", "key": "PROJ" }
  }
}
```

## Next Steps

- **Workflow Conditions**: Similar to validators but control UI visibility
- **Workflow Post Functions**: Execute after successful transitions
- **Events Payloads**: Understand what data is available when validators trigger