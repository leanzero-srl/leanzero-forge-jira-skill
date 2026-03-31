# Bitbucket Merge Check Module

## Overview

The `bitbucket:mergeCheck` module allows Forge apps to validate pull requests before merging. Merge checks run during the merge process and can block merges if validation fails.

## Module Configuration

```yaml
permissions:
  scopes:
    - read:pullrequest:bitbucket
    
modules:
  bitbucket:mergeCheck:
    - key: my-merge-check         # Required: Unique identifier
      name: Title Validator       # Required: Display name
      description: Validates PR title format # Required: Description
      
      function: checkTitle        # Function to execute
      
      triggers:
        - on-code-pushed          # OR: on-reviewer-status-changed, on-merge
        
  function:
    - key: checkTitle
      handler: index.checkTitle
```

## Trigger Types

| Trigger | When it runs |
|---------|-------------|
| `on-code-pushed` | Before merge, when new commits are pushed |
| `on-reviewer-status-changed` | Before merge, when reviewer status changes |
| `on-merge` | Just before the merge completes |

## Function Implementation

```javascript
export const checkTitle = async (event, context) => {
  const { pullrequest } = event;
  
  // Validate PR title starts with JIRA ticket
  const ticketPattern = /^[A-Z]{2,10}-\d+:/;
  const isValid = ticketPattern.test(pullrequest.title);
  
  return {
    success: isValid,
    message: isValid 
      ? "PR title follows format" 
      : "Title must start with JIRA-123: "
  };
};
```

## Response Format

### Success (Allow merge)

```javascript
return {
  success: true,
  message: "Validation passed"
};
```

### Failure (Block merge)

```javascript
return {
  success: false,
  message: "Title must include JIRA ticket number",
  details: [
    { title: "Format: JIRA-123: Description" }
  ]
};
```

## Event Payload

```javascript
{
  "workspace": {
    "uuid": "{workspace-uuid}",
    "slug": "myworkspace"
  },
  "repository": {
    "uuid": "{repo-uuid}",
    "slug": "my-repo"
  },
  "pullrequest": {
    "id": 42,
    "title": "Feature implementation",
    "state": "OPEN",
    
    "source": {
      "branch": { 
        "name": "feature/my-feature",
        "latestcommit": "abc123..."
      }
    },
    
    "destination": {
      "branch": { 
        "name": "main",
        "latestcommit": "def456..."
      }
    }
  },
  "repositoryChangeset": {
    "fromHash": "old-commit-hash",
    "toHash": "new-commit-hash"
  }
}
```

## Permissions Required

```yaml
permissions:
  scopes:
    - read:pullrequest:bitbucket