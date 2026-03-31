# Jira REST API Reference

## Overview

Forge apps access Jira data through the Jira REST API. All requests are authenticated using the Forge platform's secure context.

## Base URL

All API calls use: `https://your-site.atlassian.net`

## Authentication

Forge automatically handles authentication. Use `api.asApp()` to make requests:

```javascript
import api, { route } from '@forge/api';

// Get current user (authenticated by Forge)
const response = await api.asApp().requestJira(route`/rest/api/3/myself`);
```

## Common Endpoints

### Issue Operations

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/rest/api/3/issue/{idOrKey}` | Get issue details |
| PUT | `/rest/api/3/issue/{idOrKey}` | Update issue fields |
| POST | `/rest/api/3/issue` | Create new issue |
| DELETE | `/rest/api/3/issue/{idOrKey}` | Delete issue |

### Project Operations

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/rest/api/3/project/{idOrKey}` | Get project details |
| GET | `/rest/api/3/project/{id}/versions` | Get project versions |

### User & Group Operations

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/rest/api/3/user` | Get current user info |
| POST | `/rest/api/3/user/bulk` | Get multiple users by accountId |

### Workflow Operations

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/rest/api/3/issue/{id}/transitions` | Get available transitions |
| POST | `/rest/api/3/issue/{id}/transitions` | Execute transition |

## Example: Create Issue

```javascript
import api, { route } from '@forge/api';

const response = await api.asApp().requestJira(route`/rest/api/3/issue`, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    fields: {
      project: { key: "PROJ" },
      summary: "Issue created via API",
      description: "Description text",
      issuetype: { id: "10000" }
    }
  })
});

const createdIssue = await response.json();
console.log(`Created: ${createdIssue.key}`);
```

## Example: Update Issue

```javascript
await api.asApp().requestJira(route`/rest/api/3/issue/${issueKey}`, {
  method: 'PUT',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    fields: {
      summary: "Updated title",
      description: "New description"
    }
  })
});
```

## Example: Get Issue Transitions

```javascript
const response = await api.asApp().requestJira(
  route`/rest/api/3/issue/${issueKey}/transitions`
);
const transitions = await response.json();
console.log(transitions.transitions); // Array of available transitions