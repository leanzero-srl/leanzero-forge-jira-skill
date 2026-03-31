# Bitbucket Cloud REST API Reference for Forge Apps

This guide provides comprehensive documentation for Bitbucket Cloud REST API v2.0 endpoints.

## Table of Contents
1. [Repository Operations](#repository-operations)
2. [Branch Operations](#branch-operations)
3. [Pull Request Operations](#pull-request-operations)
4. [Commit Operations](#commit-operations)
5. [Merge Check Operations](#merge-check-operations)
6. [Pipeline Operations](#pipeline-operations)
7. [Webhook Operations](#webhook-operations)
8. [User & Permission Operations](#user--permission-operations)

---

## Base URL

All API calls use: `https://api.bitbucket.org/2.0`

## Authentication

```javascript
import api, { route } from '@forge/api';

// Bitbucket API calls
const response = await api.asApp().requestJira(
  route`https://api.bitbucket.org/2.0/repositories/workspace/repo`
);

const data = await response.json();
```

---

## Repository Operations

### Get Repository Details

```javascript
await api.asApp().requestJira(
  route`https://api.bitbucket.org/2.0/repositories/{workspace}/{repo}`
);
```

**Response includes:**
- `uuid` - Repository UUID
- `full_name` - Workspace/repo format
- `description` - Repository description
- `is_private` - Whether repo is private
- `mainbranch.name` - Default branch name
- `project.name` - Associated project

### List Repositories

```javascript
const response = await api.asApp().requestJira(
  route`https://api.bitbucket.org/2.0/repositories/{workspace}`
);
```

**Query Parameters:**
- `page` - Page number for pagination
- `pagelen` - Results per page (default: 10, max: 100)
- `q` - Search query

### Create Repository

```javascript
await api.asApp().requestJira('/rest/api/2.0/repositories/{workspace}', {
  method: 'POST',
  body: JSON.stringify({
    name: 'new-repo',
    description: 'Repository description',
    is_private: true,
    project: { key: 'PROJ' },
    mainbranch: { name: 'main' }
  })
});
```

**Request Body Fields:**
- `name` - Repository name (required)
- `description` - Repository description
- `is_private` - Whether repo is private
- `project.key` - Associated project key
- `scm` - Source control type ('git' or 'hg')
- `has_wiki` - Enable wiki
- `has_issues` - Enable issues

### Update Repository

```javascript
await api.asApp().requestJira(
  route`https://api.bitbucket.org/2.0/repositories/{workspace}/{repo}`,
  {
    method: 'PUT',
    body: JSON.stringify({
      description: 'Updated description',
      is_private: false
    })
  }
);
```

### Delete Repository

```javascript
await api.asApp().requestJira(
  route`https://api.bitbucket.org/2.0/repositories/{workspace}/{repo}`,
  { method: 'DELETE' }
);
```

---

## Branch Operations

### List Branches

```javascript
const response = await api.asApp().requestJira(
  route`https://api.bitbucket.org/2.0/repositories/{workspace}/{repo}/branches`
);

const data = await response.json();
data.values.forEach(branch => {
  console.log(`${branch.name} - ${branch.target.hash}`);
});
```

**Query Parameters:**
- `page` - Page number
- `pagelen` - Results per page
- `q` - Filter query (e.g., `name="main"`)

### Get Branch Details

```javascript
await api.asApp().requestJira(
  route`https://api.bitbucket.org/2.0/repositories/{workspace}/{repo}/branches/{branch}`
);
```

### Protect Branch

```javascript
await api.asApp().requestJira(
  route`https://api.bitbucket.org/2.0/repositories/{workspace}/{repo}/branching-model/settings/branches`,
  {
    method: 'POST',
    body: JSON.stringify({
      pattern: 'feature/*',
      restrict_merge_approvals: true,
      require_approvals_by_lines: false
    })
  }
);
```

### Delete Branch

```javascript
await api.asApp().requestJira(
  route`https://api.bitbucket.org/2.0/repositories/{workspace}/{repo}/refs/branches/{branch}`,
  { method: 'DELETE' }
);
```

---

## Pull Request Operations

### List Pull Requests

```javascript
const response = await api.asApp().requestJira(
  route`https://api.bitbucket.org/2.0/repositories/{workspace}/{repo}/pullrequests`
);

const data = await response.json();
data.values.forEach(pr => {
  console.log(`PR #${pr.id}: ${pr.title} - ${pr.state}`);
});
```

**Query Parameters:**
- `page` - Page number
- `pagelen` - Results per page (max: 100)
- `state` - Filter by state ('open', 'merged', 'declined')
- `order` - Sort order ('oldest', 'newest', '-updated')
- `target.branch.name` - Filter by target branch

### Get Pull Request Details

```javascript
await api.asApp().requestJira(
  route`https://api.bitbucket.org/2.0/repositories/{workspace}/{repo}/pullrequests/{prId}`
);
```

**Response includes:**
- `id` - PR ID
- `title` - PR title
- `description` - PR description
- `state` - Open, Merged, Declined
- `author` - User who created the PR
- `from_ref` - Source branch info
- `to_ref` - Target branch info
- `reviewers` - List of reviewers

### Create Pull Request

```javascript
await api.asApp().requestJira(
  route`https://api.bitbucket.org/2.0/repositories/{workspace}/{repo}/pullrequests`,
  {
    method: 'POST',
    body: JSON.stringify({
      title: 'Feature implementation',
      description: 'Detailed description of changes',
      source: { branch: { name: 'feature/new-feature' } },
      destination: { branch: { name: 'main' } }
    })
  }
);
```

**Request Body Fields:**
- `title` - PR title (required)
- `description` - PR description
- `source.branch.name` - Source branch name (required)
- `destination.branch.name` - Target branch name (required)
- `reviewers.uuid` - Array of reviewer UUIDs

### Update Pull Request

```javascript
await api.asApp().requestJira(
  route`https://api.bitbucket.org/2.0/repositories/{workspace}/{repo}/pullrequests/{prId}`,
  {
    method: 'PUT',
    body: JSON.stringify({
      title: 'Updated PR title',
      description: 'Updated description'
    })
  }
);
```

### Merge Pull Request

```javascript
await api.asApp().requestJira(
  route`https://api.bitbucket.org/2.0/repositories/{workspace}/{repo}/pullrequests/{prId}/merge`,
  { method: 'POST' }
);
```

**Query Parameters:**
- `version` - PR version number (for optimistic locking)

### Decline Pull Request

```javascript
await api.asApp().requestJira(
  route`https://api.bitbucket.org/2.0/repositories/{workspace}/{repo}/pullrequests/{prId}/decline`,
  {
    method: 'POST',
    body: JSON.stringify({ version: 1 })
  }
);
```

### Add Comment to PR

```javascript
await api.asApp().requestJira(
  route`https://api.bitbucket.org/2.0/repositories/{workspace}/{repo}/pullrequests/{prId}/comments`,
  {
    method: 'POST',
    body: JSON.stringify({
      content: { raw: 'Please review this change' }
    })
  }
);
```

### List PR Comments

```javascript
await api.asApp().requestJira(
  route`https://api.bitbucket.org/2.0/repositories/{workspace}/{repo}/pullrequests/{prId}/comments`
);
```

---

## Commit Operations

### Get Commit Details

```javascript
await api.asApp().requestJira(
  route`https://api.bitbucket.org/2.0/repositories/{workspace}/{repo}/commit/{hash}`
);
```

**Response includes:**
- `hash` - Commit hash
- `message` - Commit message
- `author.uuid` - Author UUID
- `date` - Commit date
- `parents` - Parent commits

### List Commits

```javascript
const response = await api.asApp().requestJira(
  route`https://api.bitbucket.org/2.0/repositories/{workspace}/{repo}/commits`
);

const data = await response.json();
data.values.forEach(commit => {
  console.log(`${commit.hash.substring(0, 7)}: ${commit.message}`);
});
```

**Query Parameters:**
- `page` - Page number
- `pagelen` - Results per page
- `author` - Filter by author UUID
- `branch` - Filter by branch

### Get Commit Comments

```javascript
await api.asApp().requestJira(
  route`https://api.bitbucket.org/2.0/repositories/{workspace}/{repo}/commit/{hash}/comments`
);
```

### Add Commit Comment

```javascript
await api.asApp().requestJira(
  route`https://api.bitbucket.org/2.0/repositories/{workspace}/{repo}/commit/{hash}/comments`,
  {
    method: 'POST',
    body: JSON.stringify({ content: { raw: 'Good change!' } })
  }
);
```

---

## Merge Check Operations

### List Merge Checks

```javascript
await api.asApp().requestJira(
  route`https://api.bitbucket.org/2.0/repositories/{workspace}/{repo}/merge-checks`
);
```

**Response includes:**
- `passing` - Whether the merge is passing all checks
- `checks` - List of individual check results

### Get Merge Check Result

```javascript
await api.asApp().requestJira(
  route`https://api.bitbucket.org/2.0/repositories/{workspace}/{repo}/merge-checks/{checkId}`
);
```

---

## Pipeline Operations

### List Pipelines

```javascript
const response = await api.asApp().requestJira(
  route`https://api.bitbucket.org/2.0/repositories/{workspace}/{repo}/pipelines`
);

const data = await response.json();
data.values.forEach(pipeline => {
  console.log(`Pipeline #${pipeline.number}: ${pipeline.state.name}`);
});
```

### Trigger Pipeline

```javascript
await api.asApp().requestJira(
  route`https://api.bitbucket.org/2.0/repositories/{workspace}/{repo}/pipelines`,
  {
    method: 'POST',
    body: JSON.stringify({
      target: { type: 'commit', commit: { hash: '{commit_hash}' } },
      branch: { name: 'main' }
    })
  }
);
```

**Request Body Fields:**
- `target.type` - Must be 'commit'
- `target.commit.hash` - Commit hash to build
- `branch.name` - Branch name (optional)
- `selector.type` - Pipeline selector type

### Get Pipeline Details

```javascript
await api.asApp().requestJira(
  route`https://api.bitbucket.org/2.0/repositories/{workspace}/{repo}/pipelines/{pipelineUuid}`
);
```

### Stop Pipeline

```javascript
await api.asApp().requestJira(
  route`https://api.bitbucket.org/2.0/repositories/{workspace}/{repo}/pipelines/{pipelineUuid}`,
  { method: 'DELETE' }
);
```

---

## Webhook Operations

### List Repository Webhooks

```javascript
const response = await api.asApp().requestJira(
  route`https://api.bitbucket.org/2.0/repositories/{workspace}/{repo}/webhooks`
);

const data = await response.json();
data.values.forEach(webhook => {
  console.log(`Webhook ${webhook.uuid}: ${webhook.description}`);
});
```

### Create Webhook

```javascript
await api.asApp().requestJira(
  route`https://api.bitbucket.org/2.0/repositories/{workspace}/{repo}/webhooks`,
  {
    method: 'POST',
    body: JSON.stringify({
      description: 'My webhook',
      url: 'https://example.com/hook',
      events: [
        'repo:push',
        'pullrequest:created',
        'pullrequest:updated'
      ],
      active: true
    })
  }
);
```

**Request Body Fields:**
- `description` - Webhook description (required)
- `url` - Callback URL (required)
- `events` - Array of event types to trigger webhook
- `active` - Whether webhook is active

### Delete Webhook

```javascript
await api.asApp().requestJira(
  route`https://api.bitbucket.org/2.0/repositories/{workspace}/{repo}/webhooks/{webhookUuid}`,
  { method: 'DELETE' }
);
```

---

## User & Permission Operations

### GetAuthenticated User

```javascript
await api.asApp().requestJira('https://api.bitbucket.org/2.0/user');
```

**Response includes:**
- `uuid` - User UUID
- `display_name` - User's display name
- `account_id` - Account ID
- `nickname` - Username

### Get Workspace Members

```javascript
const response = await api.asApp().requestJira(
  route`https://api.bitbucket.org/2.0/workspaces/{workspace}/members`
);

const data = await response.json();
data.values.forEach(member => {
  console.log(`${member.account.display_name} - ${member.role}`);
});
```

**Query Parameters:**
- `page` - Page number
- `pagelen` - Results per page

### Get Workspace Permissions

```javascript
await api.asApp().requestJira(
  route`https://api.bitbucket.org/2.0/workspaces/{workspace}/permissions`
);
```

### Add User to Team

```javascript
await api.asApp().requestJira(
  route`https://api.bitbucket.org/2.0/workspaces/{workspace}/members`,
  {
    method: 'POST',
    body: JSON.stringify({
      account_id: 'user-account-id'
    })
  }
);
```

### Remove User from Team

```javascript
await api.asApp().requestJira(
  route`https://api.bitbucket.org/2.0/workspaces/{workspace}/members/{accountUUID}`,
  { method: 'DELETE' }
);
```

---

## Error Handling

### Common HTTP Status Codes

| Code | Description |
|------|-------------|
| 200 | Success |
| 201 | Created |
| 400 | Bad Request - Invalid parameters |
| 401 | Unauthorized |
| 403 | Forbidden - Insufficient permissions |
| 404 | Not Found |

### Error Response Format

```json
{
  "error": {
    "title": "Bad Request",
    "detail": "Repository already exists"
  }
}
```

### Example Error Handling

```javascript
try {
  const response = await api.asApp().requestJira(
    route`https://api.bitbucket.org/2.0/repositories/{workspace}/{repo}`
  );

  if (!response.ok) {
    const error = await response.json();
    console.error('Bitbucket API Error:', error);
    return;
  }

  const data = await response.json();
} catch (error) {
  console.error('Network Error:', error);
}
```

---

## Forge Module Integration

### Bitbucket Merge Check

```javascript
import api from '@forge/api';

export default async function (req) {
  const { repository, pull_request } = req.body;
  
  // Call Bitbucket API to check merge status
  const response = await api.asApp().requestJira(
    route`https://api.bitbucket.org/2.0/repositories/${repository.full_name}/merge-checks`
  );
  
  const checks = await response.json();
  
  return {
    title: 'Merge Status',
    content: `Passing: ${checks.passing}`,
    status: checks.passing ? 'success' : 'error'
  };
}
```

### Bitbucket Event Handling

```javascript
export default async function (req) {
  const { event } = req.body;
  
  if (event.key === 'repo:push') {
    // Handle push event
    await handlePushEvent(event);
  } else if (event.key === 'pullrequest:created') {
    // Handle PR creation
    await handlePRCreated(event);
  }
}