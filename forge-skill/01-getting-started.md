# Getting Started with Atlassian Forge

## Core Concepts

### What is Forge?

Forge is Atlassian's serverless platform for building apps that extend Jira, Confluence, Bitbucket, and Jira Service Management. Apps run in a secure, isolated environment on Atlassian infrastructure.

### Key Components

| Component | Description |
|-----------|-------------|
| **Module** | A capability your app provides (e.g., workflow validator, macro) |
| **Function** | The code that executes when a module is triggered |
| **Resource** | Static assets for Custom UI (HTML/CSS/JS/JSX) |
| **Resolver** | Bridge between frontend UI and backend functions |

### App Manifest (`manifest.yml`)

The central configuration file defining your app's modules, resources, permissions, and runtime settings.

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

resources:
  - key: config-ui
    path: static/config/build

permissions:
  scopes:
    - read:jira-work
    - storage:app

app:
  id: ari:cloud:ecosystem::app/YOUR-APP-ID
  runtime:
    name: nodejs22.x
```

## App Structure

### Typical Project Layout

```
your-forge-app/
├── manifest.yml           # App configuration
├── package.json           # Dependencies
├── src/                   # Source code
│   └── index.js          # Main functions file
├── static/               # Static resources (Custom UI)
│   └── config-ui/
│       ├── build/        # Compiled output
│       ├── public/
│       └── src/          # React source files
└── forge.yml            # Forge environment configuration
```

### Module Resolution Flow

1. **Trigger**: User action in Jira (workflow transition, issue creation)
2. **Function Execution**: Forge runtime executes your function with payload + context
3. **API Calls**: Your function calls Jira/Atlassian APIs as needed
4. **Response**: Result returned to Jira

## Manifest.yml Reference

### Required Sections

| Section | Purpose |
|---------|---------|
| `modules` | Define all app capabilities |
| `resources` | Declare static asset locations |
| `permissions` | Request required scopes |
| `app.id` | Unique app identifier (ARN format) |
| `app.runtime` | Node.js version and memory |

### Optional Sections

- `function` - Define executable functions
- `resolver` - Define resolver functions for UI communication
- `external.fetch` - Allow calls to external APIs (e.g., OpenAI)
- `queue` - For async event processing

## Context Object

Every function receives two arguments:

```javascript
export const handler = async (payload, context) => {
  // payload: Module-specific data (e.g., issue details for validators)
  // context: Execution environment information
  
  console.log(context.installContext);    // App installation ID
  console.log(context.accountId);         // User's Atlassian account ID
  console.log(context.workspaceId);       // Workspace identifier
  console.log(context.license);           // License info (if applicable)
  
  return { result: 'success' };
};
```

### Context Properties

| Property | Description |
|----------|-------------|
| `accountId` | User's Atlassian account ID |
| `accountType` | 'licensed', 'unlicensed', 'customer', or 'anonymous' |
| `cloudId` | Cloud instance identifier |
| `installContext` | App installation ARI |
| `workspaceId` | Workspace identifier (newer architecture) |
| `principal` | User identity information |
| `license` | License status for paid apps |
| `installation` | Installation details including ARI |

## Function Types

### Trigger Functions
Executed when specific events occur.

```javascript
// Event: Issue created in Jira
export const issueCreated = async (event, context) => {
  // event contains issue data
  return { status: 'processed' };
};
```

### Resolver Functions
Called from Custom UI to backend logic.

```javascript
import Resolver from '@forge/resolver';

const resolver = new Resolver();

resolver.define('fetchData', async ({ payload }, context) => {
  // Can make Jira API calls with proper auth
  const response = await api.asApp().requestJira(route`/rest/api/3/myself`);
  return { data: await response.json() };
});

export const handler = resolver.getDefinitions();
```

### Workflow Module Functions

| Module Type | Function Signature |
|-------------|-------------------|
| `jira:workflowValidator` | Returns `{ result: true }` or `{ result: false, errorMessage: '...' }` |
| `jira:workflowCondition` | Same as validator |
| `jira:workflowPostFunction` | Returns `{ result: true }` |

## Testing Best Practices

1. **Use Forge Tunnel** for local development:
   ```bash
   forge tunnel
   ```

2. **Deploy to staging** before production:
   ```bash
   forge deploy -e staging
   forge install --upgrade -e staging
   ```

3. **Check logs** during debugging:
   ```bash
   forge logs -n 50
   ```

4. **Lint before deploying**:
   ```bash
   forge lint
   forge lint --fix  # Auto-fix some issues
   ```

## Common Commands Reference

| Command | Purpose |
|---------|---------|
| `forge init` | Create new app |
| `forge deploy` | Deploy to development site |
| `forge install --upgrade` | Install/update on site |
| `forge tunnel` | Local testing with live environment |
| `forge logs -n 50` | View last 50 log entries |
| `forge lint` | Check manifest/code for issues |

## Next Steps

- **Jira Modules**: Learn about workflow validators, conditions, and post functions
- **Events & Payloads**: Understand what data is available when modules trigger
- **API Endpoints**: Know how to call Jira REST APIs from your app
- **Permissions**: Configure required scopes for your app's functionality