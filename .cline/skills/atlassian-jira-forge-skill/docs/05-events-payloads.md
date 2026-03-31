# Events & Payloads Reference

## Overview

Forge apps respond to various events fired by Atlassian products. Understanding event structures is crucial for building robust applications that react correctly to user actions.

## Event Categories

| Category | Product | Description |
|----------|---------|-------------|
| **Jira Events** | Jira | Issue creation, updates, workflow transitions, comments, etc. |
| **Bitbucket Events** | Bitbucket | Repository changes, PR events, merge checks |
| **Confluence Events** | Confluence | Page creation, updates, macro execution |

## Common Event Structure

All Forge events share a common structure:

```javascript
{
  // Product-specific payload data
  issue: { ... },
  repository: { ... },
  
  // Execution context
  context: {
    cloudId: "ari:cloud:identity::site/...",
    moduleKey: "my-app-module"
  },
  
  // Atlassian ID of triggering user
  atlassianId: "557058:1234-5678-abcd-efgh"
}
```

## Jira Events

### Issue Events

| Event Name | Description |
|------------|-------------|
| `avi:jira:created:issue` | When a new issue is created |
| `avi:jira:updated:issue` | When an issue's fields are updated |
| `avi:jira:deleted:issue` | When an issue is deleted |
| `avi:jira:transited:issue` | When an issue's status changes |

### Issue Payload Structure

```javascript
{
  "eventType": "avi:jira:created:issue",
  "atlassianId": "557058:1234-5678-abcd-efgh",
  "context": {
    "cloudId": "ari:cloud:identity::site/...",
    "moduleKey": "my-app-trigger"
  },
  "issue": {
    "id": "12345",
    "key": "PROJ-123",
    "fields": {
      "summary": "Bug in login module",
      "description": "User cannot log in with valid credentials",
      "issuetype": {
        "id": "10000",
        "name": "Bug"
      },
      "project": {
        "id": "10001",
        "key": "PROJ"
      },
      "status": {
        "id": "10000",
        "name": "To Do"
      },
      "assignee": {
        "accountId": "557058:abc123def456",
        "displayName": "John Doe"
      },
      "labels": ["critical", "ui"],
      "priority": {
        "id": "2",
        "name": "High"
      }
    },
    "changelog": [  // Only in updated events
      {
        "field": "status",
        "from": "10000",
        "to": "10001",
        "fromString": "To Do",
        "toString": "In Progress"
      }
    ]
  }
}
```

### Comment Events

| Event Name | Description |
|------------|-------------|
| `avi:jira:created:comment` | When a comment is added to an issue |
| `avi:jira:updated:comment` | When a comment is edited |
| `avi:jira:deleted:comment` | When a comment is deleted |

### Comment Payload

```javascript
{
  "eventType": "avi:jira:created:comment",
  "atlassianId": "557058:abc123",
  "issue": {
    "id": "12345",
    "key": "PROJ-123"
  },
  "comment": {
    "id": "10000",
    "body": "This bug is related to the API timeout issue.",
    "author": {
      "accountId": "557058:abc123",
      "displayName": "Jane Smith"
    }
  },
  "context": { ... }
}
```

### Workflow Events

| Event Name | Description |
|------------|-------------|
| `avi:jira:workflow:transition` | When any workflow transition occurs |

## Trigger Module Configuration

```yaml
modules:
  trigger:
    - key: issue-created-trigger
      events:
        - avi:jira:created:issue
        - avi:jira:updated:issue
      function: handleIssueEvents
      
    - key: pr-merge-check
      events:
        - avi:bitbucket:pr:opened
        - avi:bitbucket:pr:updated
      function: handlePR Events
```

## Function Handler Signature

```javascript
export const handleIssueEvents = async (event, context) => {
  console.log('Event type:', event.eventType);
  
  switch(event.eventType) {
    case 'avi:jira:created:issue':
      return handleCreatedIssue(event);
    case 'avi:jira:updated:issue':
      return handleUpdatedIssue(event);
  }
};

export const handlePR Events = async (event, context) => {
  console.log('PR trigger:', event.pullrequest.title);
  // Your PR handling logic
};
```

## Failed Expression Event

When a workflow condition/validator expression fails:

```javascript
{
  "eventType": "avi:jira:failed:expression",
  "extensionId": "ari:cloud:ecosystem::extension/appId/envId/static/forge-condition",
  "workflowId": "workflow-uuid",
  "workflowName": "Software Workflow",
  "conditionId": "my-condition",  // or "validatorId"
  "expression": "issue.assignee.unknown",
  "errorMessages": [
    "Evaluation failed: 'issue.assignee.unknown' - Unrecognized property"
  ],
  "context": {
    "issue": { "id": "10000", "key": "TEST-1" },
    "project": { "id": "10000", "key": "TEST" },
    "user": { "accountId": "abc-123" },
    "transition": { ... }
  }
}
```

## Accessing Event Data

```javascript
// Issue data from any Jira event
const issueKey = event.issue.key;
const issueId = event.issue.id;

// Project information
const projectKey = event.issue.fields.project.key;
const projectId = event.issue.fields.project.id;

// User who triggered the event
const triggerUserId = event.atlassianId;

// Custom field access (use field ID, not name)
const customFieldValue = event.issue.fields.customfield_10001;
```

## Best Practices

1. **Validate Event Type**: Always check `event.eventType` before processing
2. **Handle Missing Data**: Issue fields may be undefined in some events
3. **Error Recovery**: Don't let one failed event break your app
4. **Rate Limiting**: External API calls should handle rate limits gracefully

## Next Steps

- **Jira Modules**: Understand which modules respond to which events
- **API Endpoints**: Learn how to query additional data from Jira REST API
- **Storage**: Persist event-related data using Forge KVS