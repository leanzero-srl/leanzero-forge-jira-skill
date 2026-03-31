# Bitbucket Forge Modules

## Overview

Bitbucket Forge apps extend Bitbucket's functionality through merge checks and repository integrations.

## Available Module Types

| Module | Description |
|--------|-------------|
| **[merge-checks.md](./merge-checks.md)** | Validate pull requests before merging |

## Core Concepts

### Merge Checks

Merge checks validate pull request contents before allowing merges. They can be:
- **Pre-merge**: Run on push or reviewer status change
- **On-merge**: Run just before merge completes

## Configuration Example

```yaml
permissions:
  scopes:
    - read:pullrequest:bitbucket
    
modules:
  bitbucket:mergeCheck:
    - key: my-check
      function: checkPR
      name: Check PR Title
      description: Validates PR title format
      triggers:
        - on-merge
        
  function:
    - key: checkPR
      handler: index.run

app:
  runtime:
    name: nodejs24.x
```

## Trigger Events

| Trigger | Description |
|---------|-------------|
| `on-code-pushed` | When new commits are pushed (pre-merge) |
| `on-reviewer-status-changed` | When reviewer status changes (pre-merge) |
| `on-merge` | Just before merge completes |

## Response Format

```javascript
export const run = async (event, context) => {
  // Your validation logic
  const isValid = validatePr(event);
  
  return {
    success: isValid,
    message: isValid ? "OK" : "Title must start with JIRA-"
  };
};
```

## Permissions Required

```yaml
permissions:
  scopes:
    - read:pullrequest:bitbucket
```

For write operations:

```yaml
permissions:
  scopes:
    - write:repository:bitbucket
```

## Next Steps

- Read [getting-started.md](../01-getting-started.md) for core concepts