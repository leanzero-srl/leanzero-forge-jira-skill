# Forge Resolver & Context Reference

## Overview

The **Resolver** pattern enables frontend Custom UI to communicate with backend functions. The **Context** object provides secure, unalterable information about the execution environment.

---

## Resolver Pattern

### Purpose

Resolvers allow your frontend (React app) to call server-side logic that has:
- Access to Jira/Atlassian REST APIs
- Secure storage access (KVS)
- Proper authentication context

### Backend Setup

```javascript
import Resolver from '@forge/resolver';

const resolver = new Resolver();

// Define resolvable functions
resolver.define('fetchIssueData', async (request) => {
  const { payload, context } = request;
  
  // Make Jira API calls here
  const response = await api.asApp().requestJira(
    route`/rest/api/3/issue/${payload.issueKey}`
  );
  
  return { issue: await response.json() };
});

resolver.define('saveConfig', async (request) => {
  const { payload, context } = request;
  
  // Store in KVS
  await kvs.set(`config-${context.installContext}`, payload.config);
  
  return { success: true };
});

// Export handler
export const handler = resolver.getDefinitions();
```

### Frontend Invocation (Custom UI)

```javascript
import { invoke } from '@forge/bridge';

const fetchIssueData = async () => {
  try {
    const result = await invoke('fetchIssueData', {
      payload: { issueKey: 'PROJ-123' }
    });
    
    console.log(result.issue);
  } catch (error) {
    console.error('Resolver error:', error);
  }
};
```

---

## Context Object

### Structure

```javascript
{
  // Core identification
  accountId: "557058:abc-123-def",     // User's Atlassian ID
  cloudId: "ari:cloud:identity::site/...",   // Cloud instance ID
  
  // App context
  installContext: "ari:cloud:forge::site/123",
  installation: {
    ari: { installationId: "..." },
    contexts: [...]
  },
  
  // Environment
  workspaceId: "workspace-uuid",        // Workspace identifier
  environmentId: "env-id",
  environmentType: "development|production"
}
```

### Available in Resolvers

```javascript
resolver.define('myFunction', async (request) => {
  const { payload, context } = request;
  
  console.log(context.accountId);       // Current user's ID
  console.log(context.installContext);  // Installation identifier
  console.log(context.cloudId);         // Cloud instance
});
```

### Available in Function Handlers

```javascript
export const myFunction = async (payload, context) => {
  console.log(context.installContext);
  console.log(context.accountId);
  console.log(context.workspaceId);     // If available
  
  return { result: 'success' };
};
```

---

## Security Features

### Unalterable Context

Context values are:
- Guaranteed to be from the Forge runtime
- Cannot be modified by frontend
- Signed and verified by Atlassian

### Access Control

```javascript
resolver.define('adminOnly', async (request) => {
  const { context } = request;
  
  // Check if user has admin role
  const isAdmin = context.installContext.includes('admin');
  
  return { admin: isAdmin };
});
```

---

## Common Resolver Patterns

### Pattern 1: Fetch Issue Data

```javascript
// Backend
resolver.define('getIssueDetails', async ({ payload, context }) => {
  const response = await api.asApp().requestJira(
    route`/rest/api/3/issue/${payload.issueKey}`
  );
  
  return { issue: await response.json() };
});

// Frontend
const [issue, setIssue] = useState(null);
useEffect(() => {
  invoke('getIssueDetails', { payload: { issueKey } })
    .then(data => setIssue(data.issue));
}, []);
```

### Pattern 2: Save User Configuration

```javascript
// Backend
resolver.define('saveUserConfig', async ({ payload, context }) => {
  await kvs.set(`config:${context.accountId}`, payload.config);
  return { success: true };
});

// Frontend
await invoke('saveUserConfig', {
  payload: { config: { theme: 'dark', notifications: true } }
});
```

### Pattern 3: Trigger External Action

```javascript
resolver.define('triggerSync', async ({ payload, context }) => {
  // Call external API
  const response = await fetch('https://api.example.com/sync', {
    method: 'POST',
    headers: { 'Authorization': `Bearer ${process.env.API_KEY}` },
    body: JSON.stringify(payload)
  });
  
  return { result: await response.json() };
});
```

---

## Context Security in Jira Bridge

The bridge provides secure context for Custom UI:

```javascript
import { view, modal } from '@forge/bridge';

// Get context securely
const context = await view.getContext();

console.log(context.installContext);  // App installation ID
console.log(context.accountId);       // User's account ID
```

---

## Error Handling

### Resolver Errors

```javascript
resolver.define('safeFunction', async ({ payload, context }) => {
  try {
    const result = await doSomething();
    return { success: true, data: result };
  } catch (error) {
    console.error('[Resolver] Error:', error);
    
    // Return structured error
    return { 
      success: false,
      errorMessage: error.message,
      errorCode: 'INTERNAL_ERROR'
    };
  }
});
```

### Frontend Error Handling

```javascript
try {
  const result = await invoke('safeFunction', { payload });
  
  if (!result.success) {
    console.error('Resolver returned failure:', result.errorMessage);
  }
} catch (error) {
  console.error('Network/JSON error:', error);
}
```

---

## Function Handler Context vs Resolver Context

| Aspect | Function Handler | Resolver |
|--------|------------------|----------|
| **Trigger** | Module execution (workflow, event) | Frontend invocation |
| **Context Access** | Full context available | Secure context from bridge |
| **Use Case** | Automated logic | User-initiated actions |

---

## Best Practices

1. **Validate Input**: Always validate payload data in resolvers
2. **Handle Errors**: Return structured error responses
3. **Minimize Data**: Only send necessary data over the bridge
4. **Cache Wisely**: Use KVS to reduce API calls
5. **Secure Context**: Never trust context from frontend (use bridge)

---

## Next Steps

- **API Endpoints**: Combine resolvers with REST API calls
- **Storage**: Persist resolver state using KVS