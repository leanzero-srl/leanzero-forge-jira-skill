# Jira Forge Events Reference

## Overview

Forge apps respond to various events fired by Jira. Each event provides structured data about what happened and context about the execution environment.

---

## Event Categories

### Issue Events

| Event Name | Description |
|------------|-------------|
| `avi:jira:created:issue` | A new issue has been created |
| `avi:jira:updated:issue` | An existing issue's fields have changed |
| `avi:jira:deleted:issue` | An issue has been deleted |
| `avi:jira:transited:issue` | An issue's status has changed |

### Comment Events

| Event Name | Description |
|------------|-------------|
| `avi:jira:created:comment` | A comment was added to an issue |
| `avi:jira:updated:comment` | A comment was edited |
| `avi:jira:deleted:comment` | A comment was deleted |

### Workflow Events

| Event Name | Description |
|------------|-------------|
| `avi:jira:workflow:transition` | Any workflow transition occurred |

### Notification Events

| Event Name | Description |
|------------|-------------|
| `avi:jira:created:notification` | A notification was created |
| `avi:jira:deleted:notification` | A notification was deleted |

---

## Common Event Structure

All Jira events follow this structure:

```javascript
{
  "eventType": "avi:jira:created:issue",
  
  // Atlassian ID of the user who triggered the event
  "atlassianId": "557058:1234-5678-abcd-efgh",
  
  // Execution context
  "context": {
    "cloudId": "ari:cloud:identity::site/...",
    "moduleKey": "my-app-trigger"
  },
  
  // Product-specific payload data
  "issue": { ... }
}
```

---

## Issue Created Event Payload

```javascript
{
  "eventType": "avi:jira:created:issue",
  "atlassianId": "557058:abc-123-def",
  "context": {
    "cloudId": "ari:cloud:identity::site/123",
    "moduleKey": "my-trigger"
  },
  "issue": {
    "id": "123456",
    "key": "PROJ-100",
    "fields": {
      "summary": "Bug in login module",
      "description": "User cannot log in with valid credentials",
      "issuetype": {
        "id": "10000",
        "name": "Bug"
      },
      "project": {
        "id": "10001",
        "key": "PROJ",
        "name": "Project Name"
      },
      "status": {
        "id": "10000",
        "name": "To Do",
        "statusCategory": { "id": 2, "key": "new" }
      },
      "assignee": {
        "accountId": "557058:abc-123-def",
        "displayName": "John Doe"
      },
      "reporter": {
        "accountId": "557058:def-456-ghi",
        "displayName": "Jane Smith"
      },
      "labels": ["critical", "ui"],
      "priority": {
        "id": "2",
        "name": "High"
      },
      "created": "2023-10-01T10:00:00.000+0000",
      "updated": "2023-10-01T10:00:00.000+0000"
    }
  }
}
```

---

## Issue Updated Event Payload

```javascript
{
  "eventType": "avi:jira:updated:issue",
  "atlassianId": "557058:abc-123-def",
  "context": {
    "cloudId": "ari:cloud:identity::site/123"
  },
  "issue": {
    "id": "123456",
    "key": "PROJ-100",
    "fields": { ... }
  },
  "changelog": [
    {
      "field": "status",
      "from": "10000",           // Status ID
      "to": "10001",
      "fromString": "To Do",
      "toString": "In Progress"
    },
    {
      "field": "assignee",
      "from": null,
      "to": "557058:abc-123-def",
      "fromString": "Unassigned",
      "toString": "John Doe"
    }
  ]
}
```

---

## Comment Created Event Payload

```javascript
{
  "eventType": "avi:jira:created:comment",
  "atlassianId": "557058:abc-123-def",
  "context": {
    "cloudId": "ari:cloud:identity::site/123"
  },
  "issue": {
    "id": "123456",
    "key": "PROJ-100"
  },
  "comment": {
    "id": "100000",
    "body": "This bug is related to the API timeout issue.",
    "author": {
      "accountId": "557058:abc-123-def",
      "displayName": "John Doe"
    },
    "created": "2023-10-01T10:30:00.000+0000",
    "updated": "2023-10-01T10:30:00.000+0000"
  }
}
```

---

## Trigger Configuration

```yaml
modules:
  trigger:
    - key: issue-created-trigger
      events:
        - avi:jira:created:issue
        - avi:jira:updated:issue
      function: handleIssueEvents
      
    - key: comment-trigger
      events:
        - avi:jira:created:comment
      function: handleCommentEvent
```

---

## Function Handler Implementation

```javascript
export const handleIssueEvents = async (event, context) => {
  console.log('Received event:', event.eventType);
  
  switch(event.eventType) {
    case 'avi:jira:created:issue':
      return handleCreatedIssue(event);
      
    case 'avi:jira:updated:issue':
      return handleUpdatedIssue(event);
      
    default:
      console.warn('Unknown event type:', event.eventType);
  }
};

const handleCreatedIssue = async (event) => {
  const { issue, atlassianId } = event;
  
  // Log the new issue
  console.log(`New issue created: ${issue.key}`);
  console.log(`Reporter: ${issue.fields.reporter.displayName}`);
  console.log(`Triggered by: ${atlassianId}`);
  
  return { status: 'processed' };
};
```

---

## Failed Expression Event

When a workflow condition or validator expression fails:

```javascript
{
  "eventType": "avi:jira:failed:expression",
  "extensionId": "ari:cloud:ecosystem::extension/appId/envId/static/forge-condition",
  "workflowId": "workflow-uuid",
  "workflowName": "Software Workflow",
  "conditionId": "my-condition",   // or "validatorId"
  "expression": "issue.assignee.unknown",
  "errorMessages": [
    "Evaluation failed: 'issue.assignee.unknown' - Unrecognized property"
  ],
  "context": {
    "issue": { "id": "10000", "key": "TEST-1" },
    "project": { "id": "10000", "key": "PROJ" },
    "user": { "accountId": "abc-123" },
    "transition": {
      "id": "11",
      "name": "In Progress"
    }
  }
}
```

---

## Best Practices

1. **Validate event type**: Always check `event.eventType` before processing
2. **Handle missing data**: Issue fields may be undefined in some events
3. **Error recovery**: Don't let one failed event break your app
4. **Logging**: Log important information for debugging

---

## Next Steps

- Read [07-api-endpoints.md](../07-api-endpoints.md) to learn how to query additional data