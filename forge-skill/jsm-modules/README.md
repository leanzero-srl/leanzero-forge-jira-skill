# Jira Service Management (JSM) Forge Modules

## Overview

Jira Service Management Forge apps extend customer service workflows through portal interfaces and request management.

## Available Module Types

| Module | Description |
|--------|-------------|
| `jiraServiceManagement:portalRequestDetail` | Add panels to portal request views |
| `jiraServiceManagement:portalRequestDetailPanel` | Add content at bottom of request panel |
| `jiraServiceManagement:portalHeader` | Custom header in customer portal |
| `jiraServiceManagement:portalFooter` | Custom footer in customer portal |

## Core Concepts

### Portal Request Details

Add custom panels below the Activity section on portal requests:

```yaml
modules:
  jiraServiceManagement:portalRequestDetail:
    - key: my-detail-panel
      resource: main
      title: My Panel
      description: Shows additional request information
```

### Unlicensed Access

JSM modules support unlicensed access for customer users:

```yaml
modules:
  jiraServiceManagement:portalRequestDetail:
    - key: my-detail-panel
      name: Public Info Panel
      unlicensedAccess:
        - customer
        - unlicensed
```

## Permissions Required

```yaml
permissions:
  scopes:
    - read:jira-work          # View issue data
    - read:issue:jira         # Issue details
    - read:user:jira          # User information
    - read:project:jira       # Project data
```

## Context Information

Portal modules receive context including:

```javascript
{
  "type": "jiraServiceManagement:portalRequestDetail",
  "portal": { "id": 123 },
  "request": {
    "key": "SVC-456",
    "typeId": 789
  }
}
```

## Common Use Cases

1. **Service Level Monitoring**: Show SLA timers in request views
2. **Knowledge Base Integration**: Display related articles
3. **Custom Field Display**: Show additional data not in default portal
4. **Action Triggers**: Add buttons to trigger external actions

## Next Steps

- Read [getting-started.md](../01-getting-started.md) for core concepts