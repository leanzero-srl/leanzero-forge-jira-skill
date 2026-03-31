# Jira Cloud REST API Reference for Forge Apps

This guide provides comprehensive documentation for Jira Cloud REST API endpoints, organized by operation type. All endpoints use the `/rest/api/3` base path.

## Table of Contents
1. [Issue Operations](#issue-operations)
2. [Bulk Issue Operations](#bulk-issue-operations)
3. [Issue Search (JQL)](#issue-search-jql)
4. [Worklog Operations](#worklog-operations)
5. [Issue Linking](#issue-linking)
6. [Comment Operations](#comment-operations)
7. [Project Operations](#project-operations)
8. [Workflow Operations](#workflow-operations)
9. [Field Operations](#field-operations)
10. [Permission & Security Operations](#permission--security-operations)

---

## Issue Operations

### Create Issue

Create a new issue in Jira.

```javascript
import api from '@forge/api';

const response = await api.asUser().requestJira('/rest/api/3/issue', {
  method: 'POST',
  headers: {
    'Accept': 'application/json',
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    fields: {
      project: {
        key: 'PROJ'
      },
      summary: 'Issue summary',
      description: 'Issue description text',
      issuetype: {
        name: 'Task'
      }
    }
  })
});

const issue = await response.json();
console.log(`Created issue ${issue.key}`);
```

**Request Body Fields:**
- `fields.project` - Project key or ID object
- `fields.summary` - Issue summary (required)
- `fields.description` - Issue description
- `fields.issuetype.name` - Issue type name (e.g., 'Bug', 'Task', 'Story')
- `fields.priority.id` - Priority ID
- `fields.assigneeaccountId` - Assignee account ID
- `components` - Array of component objects with `name`
- `labels` - Array of label strings

**Response:**
```json
{
  "id": "12345",
  "key": "PROJ-123",
  "self": "https://your-domain.atlassian.net/rest/api/3/issue/12345"
}
```

### Get Issue

Retrieve issue details by key or ID.

```javascript
const response = await api.asUser().requestJira('/rest/api/3/issue/{issueKeyOrId}', {
  method: 'GET',
  headers: {
    'Accept': 'application/json'
  },
  query: {
    expand: 'changelog,renderedFields,names,schema,operations,editmeta,changelog'
  }
});

const issue = await response.json();
```

**Query Parameters:**
- `expand` - Comma-separated list of fields to expand (changelog, renderedFields, names, schema, operations, editmeta)
- `fields` - Comma-separated list of fields to return
- `properties` - Comma-separated list of properties to return
- `renderedFields` - Whether to return rendered field values
- `updateHistory` - Whether to update the issue's last view timestamp

### Update Issue

Update an existing issue (edit operations require proper permissions).

```javascript
await api.asUser().requestJira('/rest/api/3/issue/{issueKeyOrId}', {
  method: 'PUT',
  headers: {
    'Accept': 'application/json',
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    fields: {
      summary: 'Updated summary',
      description: 'Updated description'
    }
  })
});
```

**Request Body:**
```json
{
  "fields": {
    "summary": "New summary",
    "description": "New description",
    "priority": {"id": "2"},
    "assignee": {"accountId": "user-account-id"}
  }
}
```

### Delete Issue

Delete an issue from Jira.

```javascript
await api.asUser().requestJira('/rest/api/3/issue/{issueKeyOrId}', {
  method: 'DELETE',
  headers: {
    'Accept': 'application/json'
  }
});
```

**Query Parameters:**
- `deleteSubtasks` - Whether to delete subtasks (default: true)
- `notifyUsers` - Whether to notify users

---

## Bulk Issue Operations

### Create Issues (Bulk)

Create multiple issues in a single request.

```javascript
await api.asUser().requestJira('/rest/api/3/issue/bulk', {
  method: 'POST',
  headers: {
    'Accept': 'application/json',
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    update: {},
    issues: [
      {
        "fields": {
          "project": {"key": "PROJ"},
          "summary": "First issue",
          "issuetype": {"name": "Task"}
        }
      },
      {
        "fields": {
          "project": {"key": "PROJ"},
          "summary": "Second issue",
          "issuetype": {"name": "Task"}
        }
      }
    ]
  })
});
```

### Bulk Get Issues

Retrieve multiple issues by ID.

```javascript
const response = await api.asUser().requestJira('/rest/api/3/issue/bulk', {
  method: 'GET',
  headers: {
    'Accept': 'application/json'
  },
  query: {
    id: ['12345', '67890', '11111']
  }
});
```

### Bulk Update Issues

Update multiple issues using bulk operation.

```javascript
await api.asUser().requestJira('/rest/api/3/bulk/issue', {
  method: 'PUT',
  headers: {
    'Accept': 'application/json',
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    update: {
      "fields": {
        "summary": "Updated via bulk",
        "description": "Bulk updated issue"
      }
    },
    issues: ['PROJ-1', 'PROJ-2', 'PROJ-3']
  })
});
```

### Bulk Delete Issues

Delete multiple issues.

```javascript
await api.asUser().requestJira('/rest/api/3/bulk/issue', {
  method: 'DELETE',
  headers: {
    'Accept': 'application/json'
  },
  query: {
    id: ['12345', '67890', '11111']
  }
});
```

---

## Issue Search (JQL)

### Search Issues

Search for issues using JQL (Jira Query Language).

```javascript
const response = await api.asUser().requestJira('/rest/api/3/search', {
  method: 'POST',
  headers: {
    'Accept': 'application/json',
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    jql: 'project = PROJ AND status != Done ORDER BY created DESC',
    start: 0,
    maxResults: 50,
    fields: ['summary', 'status', 'assignee', 'created'],
    expand: 'schema,names'
  })
});

const result = await response.json();
console.log(`Found ${result.total} issues`);

// Process issues
result.issues.forEach(issue => {
  console.log(`${issue.key}: ${issue.fields.summary}`);
});
```

**Request Body Fields:**
- `jql` - JQL search string (required)
- `start` - Index of first issue to return (default: 0)
- `maxResults` - Maximum number of issues to return (default: 50, max: 100)
- `fields` - Array of field keys or IDs to include
- `expand` - Fields to expand (schema, names, renderings, operations, changelog)
- `properties` - Issue properties to include
- `validateQuery` - Whether to validate the JQL (default: true)

**Common JQL Queries:**
```javascript
// Issues assigned to me
'assignee = currentUser()'

// Open issues in project
'project = PROJ AND status in (Open, "In Progress", Reopened)'

// Created this week
'created >= startOfWeek()'

// Updated recently
'updated >= -7d'

// High priority bugs
'issuetype = Bug AND priority = High'
```

**Search Response:**
```json
{
  "startAt": 0,
  "maxResults": 50,
  "total": 123,
  "issues": [
    {
      "id": "12345",
      "key": "PROJ-123",
      "fields": {
        "summary": "Issue summary",
        "status": {"name": "Open"},
        "assignee": {"accountId": "..."}
      }
    }
  ],
  "names": {
    "summary": "Summary",
    "description": "Description"
  },
  "schema": {
    "type": "object",
    "properties": {}
  }
}
```

---

## Worklog Operations

### Create Worklog

Record work on an issue.

```javascript
await api.asUser().requestJira('/rest/api/3/issue/{issueKeyOrId}/worklog', {
  method: 'POST',
  headers: {
    'Accept': 'application/json',
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    comment: 'Completed initial implementation',
    timeSpent: '2h',
    startDate: '2024-01-15T09:00:00.000+0000',
    visibility: {
      type: 'group',
      name: 'jira-developers'
    }
  })
});
```

**Request Body Fields:**
- `timeSpent` - Time spent (e.g., '2h', '30m', '1d')
- `comment` - Worklog comment
- `startDate` - Start date in ISO format
- `visibility` - Visibility restriction (group or role)
- `adjustEstimate` - How to adjust issue estimates (new, reduceBy, increaseBy, provideNew)
- `notifyUsers` - Whether to notify users (default: true)
- `overrideDuration` - Duration override (when using reduceBy or increaseBy)

### Get Worklogs

Retrieve worklogs for an issue.

```javascript
const response = await api.asUser().requestJira(
  '/rest/api/3/issue/{issueKeyOrId}/worklog',
  { method: 'GET' }
);
```

### Get Specific Worklog

```javascript
await api.asUser().requestJira(
  '/rest/api/3/issue/{issueKeyOrId}/worklog/{worklogId}',
  { method: 'GET' }
);
```

### Update Worklog

Modify an existing worklog.

```javascript
await api.asUser().requestJira(
  '/rest/api/3/issue/{issueKeyOrId}/worklog/{worklogId}',
  {
    method: 'PUT',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      comment: 'Updated worklog description',
      timeSpent: '3h'
    })
  }
);
```

### Delete Worklog

Remove a worklog entry.

```javascript
await api.asUser().requestJira(
  '/rest/api/3/issue/{issueKeyOrId}/worklog/{worklogId}',
  { method: 'DELETE' }
);
```

---

## Issue Linking

### Create Issue Link

Create a link between two issues.

```javascript
await api.asUser().requestJira('/rest/api/3/issueLink', {
  method: 'POST',
  headers: {
    'Accept': 'application/json',
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    type: { name: 'Blocks' },
    inwardIssue: { key: 'PROJ-100' },
    outwardIssue: { key: 'PROJ-200' },
    comment: {
      body: 'Blocking this issue until dependencies are resolved',
      visibility: { type: 'group', name: 'jira-developers' }
    }
  })
});
```

**Link Types:**
- `Blocks` - Inward: Blocked by, Outward: Blocks
- `Cloners` - Inward: Is cloned by, Outward: Clones
- `Copied to` - Inward: Is copied to, Outward: Copies
- `Dependencies` - Inward: Is blocked by, Outward: Blocks
- `Duplicate` - Inward: Is duplicated by, Outward: Duplicates
- `Relates` - Inward: Related to, Outward: Related to

### Get Issue Links

Retrieve all links for an issue.

```javascript
const response = await api.asUser().requestJira(
  '/rest/api/3/issue/{issueKeyOrId}/remotelink',
  { method: 'GET' }
);
```

### Delete Issue Link

Remove a link between issues.

```javascript
await api.asUser().requestJira('/rest/api/3/issueLink/{issueLinkId}', {
  method: 'DELETE'
});
```

---

## Comment Operations

### Add Comment

Add a comment to an issue.

```javascript
await api.asUser().requestJira('/rest/api/3/issue/{issueKeyOrId}/comment', {
  method: 'POST',
  headers: {
    'Accept': 'application/json',
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    body: 'This is a comment on the issue',
    visibility: {
      type: 'group',
      name: 'jira-developers'
    }
  })
});
```

**Request Body Fields:**
- `body` - Comment text (required)
- `visibility` - Visibility restriction (optional)
- `renderedBody` - Pre-rendered HTML comment
- `startedDate` - Start date for time tracking

### Get Comments

Retrieve all comments for an issue.

```javascript
const response = await api.asUser().requestJira(
  '/rest/api/3/issue/{issueKeyOrId}/comment',
  { method: 'GET' }
);
```

### Update Comment

Modify a comment.

```javascript
await api.asUser().requestJira(
  '/rest/api/3/issue/{issueKeyOrId}/comment/{commentId}',
  {
    method: 'PUT',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      body: 'Updated comment text'
    })
  }
);
```

### Delete Comment

Remove a comment.

```javascript
await api.asUser().requestJira(
  '/rest/api/3/issue/{issueKeyOrId}/comment/{commentId}',
  { method: 'DELETE' }
);
```

---

## Project Operations

### Get Projects

List all projects accessible to the user.

```javascript
const response = await api.asUser().requestJira('/rest/api/3/project', {
  method: 'GET',
  query: {
    startAt: 0,
    maxResults: 50,
    expand: 'description,lead,assigneeType,components'
  }
});
```

**Query Parameters:**
- `startAt` - Index of first project to return
- `maxResults` - Maximum number of projects (default: 50)
- `expand` - Fields to expand (description, lead, assigneeType, components)

### Get Project

Retrieve project details.

```javascript
await api.asUser().requestJira('/rest/api/3/project/{projectIdOrKey}', {
  method: 'GET',
  query: {
    expand: 'description,lead,assigneeType,components'
  }
});
```

### Create Project

Create a new project.

```javascript
await api.asUser().requestJira('/rest/api/3/project', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    key: 'NEWPROJ',
    name: 'New Project Name',
    projectTypeKey: 'software',
    projectTemplateKey: 'com.pyxis.greenhopper.jira:gh-simplified-agile-classic',
    description: 'Project description',
    leadAccountId: 'user-account-id',
    assigneeType: 'PROJECT_LEAD'
  })
});
```

**Project Template Keys (Common):**
- `com.pyxis.greenhopper.jira:gh-simplified-agile-classic` - Simplified Scrum
- `com.pyxis.greenhopper.jira:gh-simplified-agile-basic` - Simplified Kanban
- `com.atlassian.jira-core-project-templates:jira-core-generic-template` - Jira Core

### Update Project

Modify project settings.

```javascript
await api.asUser().requestJira('/rest/api/3/project/{projectIdOrKey}', {
  method: 'PUT',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    name: 'Updated Project Name',
    description: 'Updated description'
  })
});
```

### Delete Project

Delete a project.

```javascript
await api.asUser().requestJira('/rest/api/3/project/{projectIdOrKey}', {
  method: 'DELETE',
  query: { deleteSubTasks: true }
});
```

---

## Workflow Operations

### Transition Issue

Perform a workflow transition on an issue.

```javascript
await api.asUser().requestJira('/rest/api/3/issue/{issueKeyOrId}/transitions', {
  method: 'POST',
  headers: {
    'Accept': 'application/json',
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    transition: { id: '21' }
  })
});
```

**With Additional Fields:**
```javascript
await api.asUser().requestJira('/rest/api/3/issue/{issueKeyOrId}/transitions', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    transition: { id: '21' },
    fields: {
      resolution: { name: 'Done' },
      fixVersions: [{ id: '12345' }]
    }
  })
});
```

### Get Workflow Transitions

List available transitions for an issue.

```javascript
const response = await api.asUser().requestJira(
  '/rest/api/3/issue/{issueKeyOrId}/transitions',
  { method: 'GET' }
);
```

**Response:**
```json
{
  "expand": "transitions",
  "transitions": [
    {
      "id": "21",
      "name": "Start Progress",
      "to": {"statusCategory": {"key": "indeterminate"}},
      "fields": {
        "assignee": {
          "required": true,
          "schema": {"type": "user"}
        }
      }
    }
  ]
}
```

---

## Field Operations

### Get Fields

List all available fields in Jira.

```javascript
const response = await api.asUser().requestJira('/rest/api/3/field', {
  method: 'GET',
  query: {
    startAt: 0,
    maxResults: 50
  }
});
```

### Get Field

Retrieve details about a specific field.

```javascript
await api.asUser().requestJira('/rest/api/3/field/{fieldId}', { method: 'GET' });
```

**Common Field IDs:**
- `summary` - Issue summary
- `description` - Issue description
- `project` - Project field
- `issuetype` - Issue type
- `priority` - Priority
- `assignee` - Assignee
- `reporter` - Reporter
- `labels` - Labels
- `components` - Components
- `fixVersions` - Fix versions
- `versions` - Affects versions

### Get Field Options

Retrieve available options for select fields (dropdowns, checkboxes).

```javascript
await api.asUser().requestJira('/rest/api/3/field/{fieldId}/option', {
  method: 'GET',
  query: { startAt: 0, maxResults: 100 }
});
```

---

## Permission & Security Operations

### Get Permissions

Check what permissions the user has.

```javascript
const response = await api.asUser().requestJira('/rest/api/3/mypermissions', {
  method: 'GET',
  query: {
    projectKey: 'PROJ',
    issueKey: 'PROJ-123'
  }
});
```

### Get Permission Schemes

List permission schemes.

```javascript
await api.asUser().requestJira('/rest/api/3/permissionscheme', {
  method: 'GET',
  query: { startAt: 0, maxResults: 50 }
});
```

### Get Security Levels

Retrieve issue security levels for a project.

```javascript
const response = await api.asUser().requestJira(
  '/rest/api/3/project/{projectIdOrKey}/securitylevel',
  { method: 'GET' }
);
```

### Get User Permissions

Check specific permissions for a user.

```javascript
await api.asUser().requestJira('/rest/api/3/user/permission/search', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    permissions: ['ADMINISTER', 'PROJECT_ADMIN'],
    projectKey: 'PROJ'
  })
});
```

---

## Error Handling

### Common HTTP Status Codes

| Code | Description |
|------|-------------|
| 200 | Success |
| 201 | Created |
| 400 | Bad Request - Invalid parameters |
| 401 | Unauthorized - Invalid or missing API token |
| 403 | Forbidden - Insufficient permissions |
| 404 | Not Found - Resource doesn't exist |
| 409 | Conflict - Issue in expected state |

### Error Response Format

```json
{
  "errorMessages": ["Issue with key PROJ-123 does not exist"],
  "errors": {
    "summary": "Summary is required",
    "assignee": "Assignee must be a valid user"
  }
}
```

### Example Error Handling

```javascript
try {
  const response = await api.asUser().requestJira('/rest/api/3/issue', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ fields: { summary: 'Test' } })
  });

  if (!response.ok) {
    const error = await response.json();
    console.error('API Error:', error);
    
    if (response.status === 401) {
      // Token expired or invalid
    } else if (response.status === 403) {
      // Insufficient permissions
    }
  }

  const issue = await response.json();
} catch (error) {
  console.error('Network Error:', error);
}
```

---

## Forge vs REST API

### When to Use Each

| Scenario | Recommended Approach |
|----------|---------------------|
| Basic CRUD operations | Jira REST API |
| Workflow validators/post-functions | Forge workflow modules |
| Frontend UI rendering | Forge UI modules |
| Complex business logic | Combine both approaches |

### Best Practices

1. **Use Forge Modules** for workflow rules, conditions, and post-functions
2. **Use REST API** for bulk operations and external system integration
3. **Batch Requests** when possible to reduce API calls
4. **Handle Rate Limits** - Jira Cloud has rate limiting (typically 5 req/sec)
5. **Cache Results** when appropriate to reduce API usage

### Forge Runtime APIs for Common Operations

```javascript
import jira from '@forge/api';

// Get issue fields via resolver
const fields = await jira.asUser().requestJira('/rest/api/3/issue/{id}', {
  query: { expand: 'changelog' }
});

// Update issue via resolver
await jira.asUser().requestJira('/rest/api/3/issue/{id}', {
  method: 'PUT',
  body: JSON.stringify({ fields: { summary: 'Updated' } })
});

// Search issues
await jira.asUser().requestJira('/rest/api/3/search', {
  method: 'POST',
  body: JSON.stringify({ jql: 'project = PROJ' })
});