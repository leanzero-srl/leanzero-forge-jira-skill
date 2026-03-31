---
name: atlassian-jira-forge-skill
description: Atlassian Jira Forge app development. Use when creating workflow validators, conditions, post-functions, custom UIs for workflow rules, or integrating with Jira REST APIs from a Forge app.
---

# Atlassian Jira Forge Development

This skill provides comprehensive documentation for building Forge apps that extend Jira.

## When to Use This Skill

- Creating workflow validators (validate fields before transition completes)
- Creating workflow conditions (control transition visibility)
- Creating workflow post-functions (execute logic after transition)
- Building custom UIs for workflow rule configuration
- Making Jira REST API calls from a Forge app
- Understanding Forge manifest structure and permissions

## Quick Reference

| Task | Module Type | Function/Expression |
|------|-------------|---------------------|
| Validate issue fields before transition | `jira:workflowValidator` | `function` or `expression` |
| Control transition visibility | `jira:workflowCondition` | `function` or `expression` |
| Execute logic after successful transition | `jira:workflowPostFunction` | `function` |
| Build configuration UI | Custom React UI with @forge/bridge | N/A |

---

## Core Concepts

### What is Forge?

Forge is Atlassian's serverless platform for building apps that extend Jira, Confluence, Bitbucket, and Jira Service Management. Apps run in a secure environment on Atlassian infrastructure.

### Key Components

| Component | Description |
|-----------|-------------|
| **Module** | A capability (e.g., workflow validator, macro) declared in manifest.yml |
| **Function** | The code that executes when a module is triggered |
| **Resource** | Static assets for Custom UI (HTML/CSS/JS/JSX) |
| **Resolver** | Bridge between frontend UI and backend functions |

### App Manifest (`manifest.yml`)

The central configuration file defining modules, resources, permissions, and runtime settings.

```yaml
modules:
  jira:workflowValidator:
    - key: my-validator
      name: My Validator
      description: Validates issue fields
      function: validate
      
function:
  - key: validate
    handler: index.validate

permissions:
  scopes:
    - read:jira-work
    - storage:app

app:
  runtime:
    name: nodejs22.x
  id: ari:cloud:ecosystem::app/YOUR-APP-ID
```

### Context Object

Every function receives two arguments:

```javascript
export const handler = async (payload, context) => {
  // payload: Module-specific data
  // context: Execution environment information
  
  console.log(context.installContext);
  console.log(context.accountId);
  console.log(context.workspaceId);
  
  return { result: true };
};
```

## Workflow Validators

### Module Configuration

```yaml
modules:
  jira:workflowValidator:
    - key: ai-content-validator
      name: AI Content Validator
      description: Validates content using AI
      function: validateContent
      errorMessage: "Field validation failed"
      
      create:
        resource: config-ui
```

### Function Implementation

```javascript
export const validateContent = async (args) => {
  const { issue, configuration } = args;
  
  // Configuration contains user's settings
  const fieldId = configuration.fieldId || 'description';
  
  // Validate and return result
  if (isValid) {
    return { result: true };
  } else {
    return { 
      result: false, 
      errorMessage: "Validation failed" 
    };
  }
};
```

### Response Format

- **Allow transition**: `return { result: true };`
- **Block transition**: `return { result: false, errorMessage: "..." };`

### Permissions Required

```yaml
permissions:
  scopes:
    - read:jira-work       # View issue data
    - read:workflow:jira   # Access workflow info
```

## Workflow Conditions

**Key Difference from Validators**: Conditions control UI visibility (hide/show transitions), while validators block transitions after validation.

### Module Configuration

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

### Function Implementation

```javascript
export const checkLicense = async (args) => {
  const { configuration, context } = args;
  
  // Check license status
  const isActive = context.license?.isActive === true;
  
  return { result: isActive };  // true=show, false=hide
};
```

## Workflow Post Functions

**Key Difference**: Post functions execute AFTER the transition completes (unlike validators/conditions).

### Module Configuration

```yaml
modules:
  jira:workflowPostFunction:
    - key: ai-summary-enhancer
      name: AI Summary Enhancer
      description: Enhances issue summary using AI
      function: enhanceSummary
      
      create:
        resource: config-ui
```

### Function Implementation

```javascript
import api, { route } from '@forge/api';

export const enhanceSummary = async (args) => {
  const { issue, configuration } = args;
  
  try {
    // Make Jira API calls with proper auth
    await api.asApp().requestJira(route`/rest/api/3/issue/${issue.id}`, {
      method: 'PUT',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ fields: { summary: "Enhanced" } })
    });
    
    return { result: true };
  } catch (error) {
    console.error('Error:', error);
    return { result: false, errorMessage: error.message };
  }
};
```

### Response Format

- **Continue workflow**: `return { result: true };`
- **Log but don't block**: `return { result: false, warnings: [...] };`

## Events & Payloads

### Validator/Condition Trigger Payload

```json
{
  "issue": {
    "id": "12345",
    "key": "PROJ-123",
    "fields": { ... }
  },
  "transition": {
    "id": "11",
    "name": "In Progress"
  },
  "workflow": { ... },
  "configuration": { ... }
}
```

### Post Function Trigger Payload

Includes `changelog` array showing what changed:

```json
{
  "changelog": [{
    "field": "status",
    "from": "To Do (1)",
    "to": "In Progress (2)"
  }]
}
```

## API Endpoints Reference

### @forge/api - Making REST Calls

```javascript
import api, { route } from '@forge/api';

// GET request
const response = await api.asApp().requestJira(route`/rest/api/3/issue/${issueKey}`);
const data = await response.json();

// POST request with body
await api.asApp().requestJira(route`/rest/api/3/issue`, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ fields: { summary: "New" } })
});

// Query parameters using URLSearchParams
const params = new URLSearchParams({
  fields: 'summary,description,status',
  expand: 'changelog'
});
await api.asApp().requestJira(route`/rest/api/3/issue/${issueKey}?${params}`);
```

### Common Jira REST API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/rest/api/3/issue/{id}` | Get issue details |
| PUT | `/rest/api/3/issue/{id}` | Update issue fields |
| POST | `/rest/api/3/issue` | Create new issue |
| POST | `/rest/api/3/issue/bulk` | Bulk create issues |
| POST | `/rest/api/3/issue/{id}/transitions` | Execute transition |
| GET | `/rest/api/3/project/{key}` | Get project details |
| GET | `/rest/api/3/search` | Search issues with JQL |
| GET | `/rest/api/3/user/bulk` | Get multiple users |

### Available Scopes

| Scope | Description |
|-------|-------------|
| `read:jira-work` | View issues, projects, workflows |
| `write:jira-work` | Create/update/delete issues |
| `read:workflow:jira` | Read workflow configurations |
| `storage:app` | Access Forge KVS storage |

### Additional Available Scopes

| Scope | Description |
|-------|-------------|
| `read:project:jira` | Read project data |
| `read:user:jira` | Read user information |
| `write:project:jira` | Create/update projects |
| `read:workflow-scheme:jira` | Read workflow schemes |
| `read:issuetype:jira` | Read issue types |

## Permissions & Scopes

### Required Permission Structure

```yaml
permissions:
  scopes:
    - read:jira-work
    - write:jira-work      # For post functions that modify issues
    - read:workflow:jira
    - read:user:jira       # For user/group checks
    
  external:
    fetch:
      backend:
        - "api.openai.com"   # External API domains
```

### Configuration Permissions

```yaml
app:
  licensing:
    enabled: true          # For license checking
```

## CLI Commands Reference

| Command | Purpose |
|---------|---------|
| `forge init` | Create new app |
| `forge deploy` | Deploy to development site |
| `forge install --upgrade` | Install/update on site |
| `forge tunnel` | Local testing with live environment |
| `forge logs -n 50` | View last 50 log entries |
| `forge lint` | Check manifest/code for issues |

### Development Workflow

```bash
# Initialize new app
forge init my-app
cd my-app

# Deploy to development
forge deploy

# Install on site
forge install --upgrade -e development

# Test locally with tunnel
forge tunnel
```

## UI Configuration Bridge

For workflow rules, use the Jira bridge for custom configuration:

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

### UI Modifications API

Modify Jira UI elements programmatically:

```javascript
import { uiModificationsApi } from '@forge/jira-bridge';

uiModificationsApi.onInit(
  ({ api }) => {
    // Get field by ID
    const priority = api.getFieldById('priority');
    
    // Hide field
    priority?.setVisible(false);
    
    // Change label
    const summary = api.getFieldById('summary');
    summary?.setName('Modified Label');
    
    // Get field value
    const labels = api.getFieldById('labels');
    const labelsData = labels?.getValue() || [];
  },
  () => ['priority', 'summary', 'labels']
);
```

## Advanced Documentation

See the `docs/` folder for detailed documentation on each topic:

- **01-core-concepts.md** - Core Forge concepts and manifest structure
- **02-workflow-validators.md** - Detailed validator documentation
- **03-workflow-conditions.md** - Detailed condition documentation
- **04-workflow-post-functions.md** - Detailed post function documentation
- **05-events-payloads.md** - Event structures and payloads
- **06-api-endpoints.md** - Complete API reference
- **06-api-endpoints-enhanced.md** - Enhanced Jira REST API reference with comprehensive endpoints
- **07-permissions-scopes.md** - All available scopes
- **08-cli-commands.md** - CLI command reference

## Additional API Resources

The `forge-skill/api-endpoints/` directory contains additional detailed documentation:

- **jira-rest-api.md** - Comprehensive Jira REST API endpoint reference
- **bitbucket-rest-api.md** - Bitbucket Cloud REST API endpoints
- **confluence-rest-api.md** - Confluence REST API endpoints
- **forge-runtime-apis.md** - Forge runtime and resolver APIs

## Best Practices for API Calls

1. **Use `api.asApp()`** when making calls from within a workflow context (no user session)
2. **Use `api.asUser()`** when you need to preserve the current user's permissions
3. **Batch operations** whenever possible - bulk endpoints reduce API call count
4. **Handle rate limits** - Jira Cloud typically allows 5 requests per second
5. **Paginate results** using `startAt` and `maxResults` parameters for large datasets
6. **Use expand parameters** to minimize the number of API calls needed

### Example: Bulk Issue Operations

```javascript
// Search issues
const searchResponse = await api.asApp().requestJira('/rest/api/3/search', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    jql: 'project = PROJ AND status != Done',
    maxResults: 100
  })
});

const issues = await searchResponse.json();

// Bulk update
await api.asApp().requestJira('/rest/api/3/bulk/issues/fields', {
  method: 'PUT',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    update: { fields: { summary: 'Updated' } },
    issues: issues.issues.map(i => i.key)
  })
});
```

### Example: Error Handling

```javascript
try {
  const response = await api.asApp().requestJira('/rest/api/3/issue', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ fields: { summary: 'Test' } })
  });

  if (!response.ok) {
    const error = await response.json();
    console.error('API Error:', error);
    
    // Handle specific status codes
    if (response.status === 401) {
      throw new Error('Authentication failed');
    } else if (response.status === 403) {
      throw new Error('Insufficient permissions');
    }
  }

  return await response.json();
} catch (error) {
  console.error('Request failed:', error);
  throw error;
}
```

### Example: Complex JQL Queries

```javascript
// Find issues assigned to me that are overdue
const jql = `assignee = currentUser() AND duedate < startOfDay() AND status != Done`;

// Find issues created this week with high priority
const jql2 = `created >= startOfWeek() AND priority = High`;

// Find unassigned issues in a project
const jql3 = `project = PROJ AND assignee is EMPTY`;
```

## Common Use Cases

| Use Case | Solution |
|----------|----------|
| AI Content Validation | Validate issue descriptions using AI |
| Licensing Checks | Only show transitions if license is active |
| Summary Enhancement | Automatically improve issue summaries |
| Field Validation | Ensure required fields are populated |
| Cross-System Sync | Update external systems on transition |

## Troubleshooting

### Common Errors

| Error | Cause | Solution |
|-------|-------|----------|
| "Function not found" | Verify function key matches in manifest and code | Check `function` references in manifest.yml |
| "Permission denied" | Check scopes in manifest.yml | Add required scope to permissions.scopes |
| "Expression evaluation failed" | Validate Jira expression syntax | Test expressions in workflow editor |

### Debugging

1. Use `console.log()` for debugging
2. View logs with `forge logs -n 50`
3. Test locally with `forge tunnel`

## Quick Comparison: Validators vs Conditions vs Post Functions

| Aspect | Validator | Condition | Post Function |
|--------|-----------|-----------|---------------|
| **When it runs** | Before transition completes | Before UI renders | After transition completes |
| **Purpose** | Validate data before completion | Hide/show transitions in UI | Execute logic after success |
| **Failure behavior** | Transition blocked, error shown | Transition hidden from user | Error logged, workflow continues |
| **Use case** | Ensure required fields are set | Control access based on roles | Send notifications, update related issues |