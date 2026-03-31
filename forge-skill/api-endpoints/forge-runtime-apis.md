# Forge Runtime APIs

## Overview

Forge provides several runtime APIs through npm packages. These enable communication between frontend and backend, make REST API calls to Atlassian products, and provide access to storage.

---

## @forge/api

Core module for Forge functions. Provides:
- Jira REST API access
- Confluence REST API access
- Secure context information

### Usage

```javascript
import api, { route } from '@forge/api';

// GET request
const response = await api.asApp().requestJira(route`/rest/api/3/issue/${issueKey}`);
const data = await response.json();

// POST request with body
await api.asApp().requestJira(route`/rest/api/3/issue`, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    fields: {
      summary: "New issue",
      issuetype: { id: "10000" }
    }
  })
});
```

### API Methods

| Method | Description |
|--------|-------------|
| `api.asApp().requestJira(route)` | Make Jira REST API call as app |
| `api.asUser().requestJira(route)` | Make Jira REST API call as user |

---

## @forge/resolver

Enables frontend-to-backend communication.

### Backend Setup

```javascript
import Resolver from '@forge/resolver';

const resolver = new Resolver();

resolver.define('fetchData', async ({ payload, context }) => {
  // Can access KVS and make Jira API calls
  const data = await kvs.get(`config:${context.accountId}`);
  return { config: data };
});

export const handler = resolver.getDefinitions();
```

### Frontend Invocation

```javascript
import { invoke } from '@forge/bridge';

const result = await invoke('fetchData', {
  payload: { some: 'data' }
});
```

---

## @forge/jira-bridge

For Custom UI configuration and UI modifications.

### Workflow Rules Configuration

```javascript
import { workflowRules } from '@forge/jira-bridge';

const onConfigureFn = async () => {
  const config = {
    fieldId: 'description',
    prompt: 'Validate content quality'
  };
  
  return JSON.stringify(config);
};

await workflowRules.onConfigure(onConfigureFn);
```

### UI Modifications

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
  },
  () => ['priority', 'summary']
);
```

---

## @forge/kvs

Key-value storage for data persistence.

### Simple Operations

```javascript
import kvs from '@forge/kvs';

// Set value
await kvs.set('myKey', { setting: true });

// Get value
const data = await kvs.get('myKey');

// Delete value
await kvs.delete('myKey');
```

### Entity-Based Storage

```javascript
import kvs, { WhereConditions } from '@forge/kvs';

// Store with entity type and key
await kvs.entity('users').set(userId, userData);

// Retrieve
const user = await kvs.entity('users').get(userId);

// Query
const result = await kvs.entity('users')
  .query()
  .index('by-country', { partition: ['US'] })
  .where(WhereConditions.beginsWith('john'))
  .getMany();
```

---

## @forge/bridge (Frontend)

For Custom UI frontend communication.

### Context Access

```javascript
import { view, modal } from '@forge/bridge';

// Get secure context
const context = await view.getContext();

console.log(context.installContext);  // App installation ID
console.log(context.accountId);       // User's account ID
```

### Invoke Backend Resolver

```javascript
import { invoke } from '@forge/bridge';

try {
  const result = await invoke('myFunction', {
    payload: { some: 'data' }
  });
  
  console.log(result);
} catch (error) {
  console.error('Resolver error:', error);
}
```

### Navigation

```javascript
import { navigate } from '@forge/bridge';

// Navigate to a route
await navigate.to('/dashboard');

// Get current URL parameters
const params = await navigate.getParams();