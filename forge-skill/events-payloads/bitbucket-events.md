# Bitbucket Forge Events Reference

## Overview

Forge apps respond to events fired by Bitbucket related to repositories, pull requests, and merge checks.

---

## Event Categories

### Repository Events

| Event Name | Description |
|------------|-------------|
| `avi:bitbucket:repo:created` | A new repository was created |
| `avi:bitbucket:repo:updated` | Repository settings were changed |
| `avi:bitbucket:repo:deleted` | A repository was deleted |

### Pull Request Events

| Event Name | Description |
|------------|-------------|
| `avi:bitbucket:pr:opened` | A pull request was created |
| `avi:bitbucket:pr:updated` | PR title/description changed |
| `avi:bitbucket:pr:merged` | PR was merged successfully |
| `avi:bitbucket:pr:declined` | PR was declined |

### Merge Check Events

Merge check events are fired when your configured merge check is triggered.

---

## Common Event Structure

```javascript
{
  "eventType": "avi:bitbucket:pr:opened",
  
  // Atlassian ID of the user who triggered the event
  "atlassianId": "557058:abc-123-def",
  
  // Execution context
  "context": {
    "cloudId": "ari:cloud:identity::site/...",
    "moduleKey": "my-app-trigger"
  },
  
  // Bitbucket-specific data
  "workspace": { ... },
  "repository": { ... },
  "pullrequest": { ... }
}
```

---

## Pull Request Opened Event Payload

```javascript
{
  "eventType": "avi:bitbucket:pr:opened",
  "atlassianId": "557058:abc-123-def",
  "context": {
    "cloudId": "ari:cloud:identity::site/123"
  },
  "workspace": {
    "uuid": "{workspace-uuid}",
    "slug": "myworkspace"
  },
  "repository": {
    "uuid": "{repo-uuid}",
    "slug": "my-repo",
    "name": "My Repository",
    "project": { "key": "PROJ" }
  },
  "pullrequest": {
    "id": 42,
    "title": "Feature implementation",
    "description": "Adds new feature for X",
    "state": "OPEN",
    "createdOn": "2023-10-01T10:00:00Z",
    "updatedOn": "2023-10-01T10:00:00Z",
    
    "source": {
      "branch": { 
        "name": "feature/my-feature",
        "latestcommit": "abc123..."
      },
      "repository": {
        "uuid": "{repo-uuid}",
        "slug": "my-repo"
      }
    },
    
    "destination": {
      "branch": { 
        "name": "main",
        "latestcommit": "def456..."
      }
    },
    
    "reviewers": [
      {
        "user": {
          "accountId": "557058:abc-123-def",
          "displayName": "John Doe"
        }
      }
    ],
    
    "participants": [
      {
        "user": { ... },
        "role": "AUTHOR",
        "status": "unapproved"
      }
    ]
  }
}
```

---

## Merge Check Trigger Event

When a merge check is triggered:

```javascript
{
  "eventType": "avi:bitbucket:merge-check:triggered",
  "atlassianId": "557058:abc-123-def",
  "context": {
    "cloudId": "ari:cloud:identity::site/123"
  },
  
  // Workspace info
  "workspace": {
    "uuid": "{workspace-uuid}",
    "slug": "myworkspace"
  },
  
  // Repository info
  "repository": {
    "uuid": "{repo-uuid}",
    "slug": "my-repo"
  },
  
  // Pull request data
  "pullrequest": {
    "id": 42,
    "title": "Feature implementation",
    "state": "OPEN",
    
    "source": {
      "branch": { "name": "feature/my-feature" },
      "commit": { "hash": "abc123..." }
    },
    
    "destination": {
      "branch": { "name": "main" },
      "commit": { "hash": "def456..." }
    }
  },
  
  // Changeset info (for pre-merge triggers)
  "repositoryChangeset": {
    "fromHash": "old-commit-hash",
    "toHash": "new-commit-hash"
  }
}
```

---

## Trigger Configuration

```yaml
modules:
  trigger:
    - key: pr-creation-trigger
      events:
        - avi:bitbucket:pr:opened
      function: handlePRCreated
      
    - key: merge-check-trigger
      events:
        - avi:bitbucket:merge-check:triggered
      function: runMergeCheck
      
  bitbucket:mergeCheck:
    - key: title-validator
      function: checkTitle
      name: Check PR Title
      description: Validates PR title format
      triggers:
        - on-merge
        
  function:
    - key: handlePRCreated
      handler: index.handlePRCreated
    - key: runMergeCheck
      handler: index.runMergeCheck
    - key: checkTitle
      handler: index.checkTitle
```

---

## Merge Check Response

```javascript
export const checkTitle = async (event, context) => {
  const { pullrequest } = event;
  
  // Validate PR title format
  const isValid = pullrequest.title.startsWith('JIRA-');
  const message = isValid 
    ? 'PR title follows convention'
    : 'PR title must start with JIRA ticket number';
  
  return {
    success: isValid,
    message: message
  };
};
```

---

## Best Practices

1. **Validate event type**: Always check `event.eventType`
2. **Handle merge check failures gracefully**: Return structured errors
3. **Use git hashes for caching**: Avoid re-processing unchanged commits
4. **Log trigger events**: Helps debug why checks are failing

---

## Next Steps

- Read [07-api-endpoints.md](../07-api-endpoints.md) to learn Bitbucket REST API calls