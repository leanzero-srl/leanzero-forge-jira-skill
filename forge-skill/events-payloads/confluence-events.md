# Confluence Forge Events Reference

## Overview

Forge apps respond to events fired by Confluence related to pages, spaces, and macros.

---

## Event Categories

### Page Events

| Event Name | Description |
|------------|-------------|
| `avi:confluence:page:created` | A new page was created |
| `avi:confluence:page:updated` | An existing page was edited |
| `avi:confluence:page:deleted` | A page was moved to trash |

### Space Events

| Event Name | Description |
|------------|-------------|
| `avi:confluence:space:created` | A new space was created |
| `avi:confluence:space:updated` | Space settings were changed |
| `avi:confluence:space:deleted` | A space was moved to trash |

### Comment Events

| Event Name | Description |
|------------|-------------|
| `avi:confluence:comment:created` | A comment was added to a page |
| `avi:confluence:comment:updated` | A comment was edited |
| `avi:confluence:comment:deleted` | A comment was deleted |

### Macro Events

| Event Name | Description |
|------------|-------------|
| `avi:confluence:macro:rendered` | A macro was rendered on a page |

---

## Common Event Structure

```javascript
{
  "eventType": "avi:confluence:page:created",
  
  // Atlassian ID of the user who triggered the event
  "atlassianId": "557058:abc-123-def",
  
  // Execution context
  "context": {
    "cloudId": "ari:cloud:identity::site/...",
    "moduleKey": "my-app-trigger"
  },
  
  // Confluence-specific data
  "page": { ... },
  "space": { ... }
}
```

---

## Page Created Event Payload

```javascript
{
  "eventType": "avi:confluence:page:created",
  "atlassianId": "557058:abc-123-def",
  "context": {
    "cloudId": "ari:cloud:identity::site/123"
  },
  "page": {
    "id": "123456789",
    "type": "page",
    "title": "New Documentation Page",
    "space": {
      "id": "987654",
      "key": "DOCS",
      "name": "Documentation"
    },
    "status": "current",
    "version": {
      "number": 1,
      "minorEdit": false
    },
    "body": {
      "storage": {
        "value": "<p>This is the page content.</p>",
        "representation": "storage"
      }
    },
    "ancestors": [
      { "id": "987654321", "type": "page" }
    ],
    "children": {
      "comment": {
        "size": 0
      }
    },
    "operations": []
  },
  "space": {
    "id": "987654",
    "key": "DOCS",
    "name": "Documentation"
  }
}
```

---

## Page Updated Event Payload

```javascript
{
  "eventType": "avi:confluence:page:updated",
  "atlassianId": "557058:abc-123-def",
  "context": {
    "cloudId": "ari:cloud:identity::site/123"
  },
  "page": { ... },
  "version": {
    "number": 2,
    "minorEdit": false
  },
  "previousVersion": {
    "number": 1
  }
}
```

---

## Macro Rendered Event Payload

```javascript
{
  "eventType": "avi:confluence:macro:rendered",
  "atlassianId": "557058:abc-123-def",
  "context": {
    "cloudId": "ari:cloud:identity::site/123"
  },
  "macro": {
    "id": "macro-id-123",
    "name": "my-forge-macro",
    "body": "",
    "params": {}
  },
  "page": {
    "id": "123456789",
    "title": "Page Title"
  }
}
```

---

## Trigger Configuration

```yaml
modules:
  trigger:
    - key: page-created-trigger
      events:
        - avi:confluence:page:created
        - avi:confluence:page:updated
      function: handlePageEvents
      
    - key: macro-rendered-trigger
      events:
        - avi:confluence:macro:rendered
      function: handleMacroRendered
      
  function:
    - key: handlePageEvents
      handler: index.handlePageEvents
    - key: handleMacroRendered
      handler: index.handleMacroRendered
```

---

## Function Handler Example

```javascript
export const handlePageEvents = async (event, context) => {
  console.log('Received event:', event.eventType);
  
  switch(event.eventType) {
    case 'avi:confluence:page:created':
      return handlePageCreated(event);
      
    case 'avi:confluence:page:updated':
      return handlePageUpdated(event);
  }
};

const handlePageCreated = async (event) => {
  const { page, atlassianId } = event;
  
  console.log(`New page created: ${page.title}`);
  console.log(`Space: ${page.space.name} (${page.space.key})`);
  console.log(`Created by: ${atlassianId}`);
  
  return { status: 'processed' };
};
```

---

## Best Practices

1. **Validate event type**: Always check `event.eventType`
2. **Handle body formats**: Content can be in different representations
3. **Avoid infinite loops**: Be careful with update triggers that modify pages
4. **Log access patterns**: Helps understand macro usage

---

## Next Steps

- Read [07-api-endpoints.md](../07-api-endpoints.md) to learn Confluence REST API calls