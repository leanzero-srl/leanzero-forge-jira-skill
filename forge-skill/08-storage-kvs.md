# Forge Storage & KVS Reference

## Overview

Forge provides Key-Value Store (KVS) for persisting data across function invocations. KVS supports simple key-value operations and entity-based queries with indexing.

## Two Storage Approaches

| Approach | Use Case |
|----------|----------|
| **Simple KVS** | Single values, global configuration |
| **Entity-based KVS** | Complex data structures, indexed queries |

---

## Simple KVS Operations

### Setting Values

```javascript
import kvs from '@forge/kvs';

// String key, any value type
await kvs.set('myKey', 'stringValue');
await kvs.set('config', { setting1: true, setting2: 42 });
```

### Getting Values

```javascript
const value = await kvs.get('myKey');        // Returns the stored value
const defaultValue = await kvs.get('key') || 'default';
```

### Deleting Values

```javascript
await kvs.delete('myKey');
```

### Checking Existence

```javascript
const value = await kvs.get('myKey');
if (value !== undefined) {
  // Key exists and has a value
}
```

---

## Entity-Based KVS

Entity-based storage is for structured data that needs querying.

### Creating Entities

```javascript
import kvs from '@forge/kvs';

// Store user data with ID as key
await kvs.entity('users').set(userId, {
  name: 'John Doe',
  email: 'john@example.com',
  preferences: { theme: 'dark' }
});

// Store multiple items in same entity
await kvs.entity('settings').set('app-config', { featureFlags: {} });
```

### Reading Entities

```javascript
const user = await kvs.entity('users').get(userId);
const settings = await kvs.entity('settings').get('app-config');
```

### Querying Entities

```javascript
// Get all entities in a type
const result = await kvs.entity('users').query().getMany();
const users = result.results;  // Array of { key, value } objects

// With index (partition only)
const result = await kvs.entity('users')
  .query()
  .index('by-country', { partition: ['US'] })
  .getMany();

// With prefix matching
import kvs, { WhereConditions } from '@forge/kvs';

const result = await kvs.entity('users')
  .query()
  .index('by-name', { partition: ['US'] })
  .where(WhereConditions.beginsWith('john'))
  .getMany();
```

### Deleting Entities

```javascript
await kvs.entity('users').delete(userId);
await kvs.entity('settings').delete('app-config');
```

---

## Indexes

Indexes enable efficient queries on entity data.

### Defining Indexes in Code

```javascript
// Define index when storing
await kvs.entity('users')
  .set(userId, userData, { indexes: ['by-country', 'by-name'] });
```

### Common Index Patterns

```javascript
// Single-field partition (country)
.index('by-country', { partition: [userData.country] })

// Multi-field with sort key
.index('by-date-name', {
  partition: [userData.date.split('-')[0]],  // year
  sort: userData.name
})

// Nested objects (JSON path)
.index('by-preference-theme', {
  partition: [userData.preferences.theme]
})
```

---

## Data Persistence in Workflow Modules

```javascript
import { workflowRules } from '@forge/jira-bridge';
import kvs from '@forge/kvs';

const onConfigureFn = async () => {
  const config = {
    fieldId: document.getElementById('field-select').value,
    prompt: document.getElementById('prompt-input').value
  };
  
  // Persist configuration for this issue
  await kvs.set(`config-${issueKey}`, JSON.stringify(config));
  
  return JSON.stringify(config);
};

await workflowRules.onConfigure(onConfigureFn);

// Later, in your function:
const savedConfig = await kvs.get(`config-${issueKey}`);
```

---

## Storage Limits

| Limit | Value |
|-------|-------|
| Key length | Max 1024 characters |
| Value size | Max 1 MB per item |
| Indexes per entity | Max 5 indexes |
| Total storage | Dependent on Atlassian plan |

---

## Best Practices

### Use Simple KVS For:
- Application-level configuration
- Cache data that can be regenerated
- Small, simple data structures

### Use Entity-Based KVS For:
- User-specific data
- Data that needs querying
- Large collections of similar items

### Error Handling

```javascript
import kvs from '@forge/kvs';

try {
  await kvs.set('myKey', { large: 'data' });
} catch (error) {
  console.error('KVS error:', error);
  
  if (error.message.includes('Size limit')) {
    // Handle size exceeded
  }
}
```

### Data Serialization

```javascript
// Objects must be JSON-stringifiable
await kvs.set('myKey', { nested: { object: true } });

// Retrieval returns parsed JSON
const data = await kvs.get('myKey');
console.log(data.nested.object);  // true
```

---

## Permissions Required

```yaml
permissions:
  scopes:
    - storage:app
```

---

## Example: User Preferences Storage

```javascript
import kvs from '@forge/kvs';

export const saveUserPreferences = async (accountId, preferences) => {
  await kvs.entity('user-preferences')
    .set(accountId, preferences);
};

export const getUserPreferences = async (accountId) => {
  return await kvs.entity('user-preferences').get(accountId) || {};
};
```

---

## Example: Issue Configuration

```javascript
import kvs from '@forge/kvs';

// Save issue-specific config
await kvs.set(`issue-config:${issueKey}`, {
  enabledFields: ['summary', 'description'],
  promptTemplate: 'Review content quality'
});

// Retrieve in function
const config = await kvs.get(`issue-config:${issueKey}`) || {};
```

---

## Next Steps

- **API Endpoints**: Learn how to persist issue-related data
- **Permissions**: Configure storage access for your app