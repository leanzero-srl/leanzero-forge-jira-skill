# Confluence Cloud REST API Reference for Forge Apps

This guide provides comprehensive documentation for Confluence Cloud REST API v2 endpoints.

## Table of Contents
1. [Content Operations](#content-operations)
2. [Space Operations](#space-operations)
3. [Comment Operations](#comment-operations)
4. [Attachment Operations](#attachment-operations)
5. [Label Operations](#label-operations)
6. [User & Permission Operations](#user--permission-operations)

---

## Base URL

All API calls use: `https://your-site.atlassian.net/wiki`

## Authentication

```javascript
import api, { route } from '@forge/api';

// Confluence API calls
const response = await api.asApp().requestJira(
  route`/wiki/rest/api/content/${pageId}`
);

const data = await response.json();
```

---

## Content Operations

### Get Page/Content Details

```javascript
await api.asApp().requestJira(
  route`/wiki/rest/api/content/${contentId}?expand=body.storage,version`
);
```

**Query Parameters:**
- `expand` - Comma-separated list of fields to expand
  - `body.storage` - Content in storage format (HTML)
  - `body.view` - Content in view format
  - `version` - Version information
  - `ancestors` - Parent page information
  - `children` - Child content
- `status` - Filter by status (current, archived, trashed)
- `version` - Specific version number

**Response includes:**
- `id` - Content ID
- `title` - Page title
- `type` - Content type (page, blogpost, comment)
- `space.key` - Space key
- `body.storage.value` - HTML content
- `status` - Content status
- `version.number` - Version number

### List Pages/Content

```javascript
const response = await api.asApp().requestJira(
  route`/wiki/rest/api/content?start=0&limit=25&spaceKey=DOCS&expand=body.storage`
);

const data = await response.json();
data.results.forEach(page => {
  console.log(`${page.title} (${page.id})`);
});
```

**Query Parameters:**
- `spaceKey` - Filter by space key
- `start` - Pagination start index (default: 0)
- `limit` - Results per page (default: 25, max: 100)
- `expand` - Fields to expand
- `status` - Filter by status
- `type` - Filter by content type
- `title` - Filter by title

### Create Page/Content

```javascript
await api.asApp().requestJira('/wiki/rest/api/content', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    type: 'page',
    title: 'New Page Title',
    space: { key: 'DOCS' },
    ancestors: [{ id: '123456789' }],
    body: {
      storage: {
        value: '<p>This is the page content.</p>',
        representation: 'storage'
      }
    }
  })
});
```

**Request Body Fields:**
- `type` - Content type ('page', 'blogpost', 'comment')
- `title` - Page title (required)
- `space.key` - Space key (required)
- `ancestors.id` - Parent page ID for hierarchy
- `body.storage.value` - HTML content (storage format)
- `status` - Content status ('current', 'draft')

### Update Page/Content

```javascript
await api.asApp().requestJira(
  route`/wiki/rest/api/content/${contentId}`,
  {
    method: 'PUT',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      id: contentId,
      type: 'page',
      title: 'Updated Title',
      space: { key: 'DOCS' },
      version: { number: 2, message: 'Update description' },
      body: {
        storage: {
          value: '<p>Updated content</p>',
          representation: 'storage'
        }
      }
    })
  }
);
```

**Request Body Fields:**
- `id` - Content ID (required)
- `type` - Content type (required)
- `title` - Page title
- `space.key` - Space key
- `version.number` - Current version number (required for update)
- `version.message` - Version comment
- `body.storage.value` - Updated content

### Delete Page/Content

```javascript
await api.asApp().requestJira(
  route`/wiki/rest/api/content/${contentId}?status=current`,
  { method: 'DELETE' }
);
```

**Query Parameters:**
- `status` - Content status (default: current)
- `version` - Version number to delete

### Archive Page

```javascript
await api.asApp().requestJira(
  route`/wiki/rest/api/content/${contentId}`,
  {
    method: 'PUT',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      status: 'archived'
    })
  }
);
```

---

## Space Operations

### Get Space Details

```javascript
await api.asApp().requestJira(
  route`/wiki/rest/api/space/${spaceKey}?expand=description.plain,homepage`
);
```

**Query Parameters:**
- `expand` - Fields to expand (description, homepage, icon, permissions)
- `start` - Pagination start index
- `limit` - Results per page

### List Spaces

```javascript
const response = await api.asApp().requestJira(
  route`/wiki/rest/api/space?start=0&limit=25`
);

const data = await response.json();
data.results.forEach(space => {
  console.log(`${space.name} (${space.key})`);
});
```

**Query Parameters:**
- `start` - Pagination start index
- `limit` - Results per page (default: 25)
- `expand` - Fields to expand
- `type` - Filter by space type ('global', 'personal')

### Create Space

```javascript
await api.asApp().requestJira('/wiki/rest/api/space', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    key: 'NEWSPACE',
    name: 'New Space Name',
    description: { plain: { value: 'Space description' } },
    type: 'global'
  })
});
```

**Request Body Fields:**
- `key` - Space key (required, uppercase letters and numbers)
- `name` - Space name (required)
- `description.plain.value` - Space description
- `type` - Space type ('global' or 'personal')

### Update Space

```javascript
await api.asApp().requestJira(
  route`/wiki/rest/api/space/${spaceKey}`,
  {
    method: 'PUT',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      name: 'Updated Space Name',
      description: { plain: { value: 'Updated description' } }
    })
  }
);
```

### Delete Space

```javascript
await api.asApp().requestJira(
  route`/wiki/rest/api/space/${spaceKey}?purge=true`,
  { method: 'DELETE' }
);
```

**Query Parameters:**
- `purge` - Whether to permanently delete (default: false)

---

## Comment Operations

### Get Comments

```javascript
await api.asApp().requestJira(
  route`/wiki/rest/api/content/${contentId}/child/comment?expand=body.storage`
);
```

**Query Parameters:**
- `start` - Pagination start index
- `limit` - Results per page
- `expand` - Fields to expand

### Add Comment

```javascript
await api.asApp().requestJira(
  route`/wiki/rest/api/content/${parentId}/child/comment`,
  {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      type: 'comment',
      body: {
        storage: {
          value: '<p>This is a comment.</p>',
          representation: 'storage'
        }
      },
      container: { id: parentId, type: 'page', status: 'current' }
    })
  }
);
```

**Request Body Fields:**
- `type` - Content type ('comment')
- `body.storage.value` - Comment HTML content
- `container.id` - Parent content ID
- `status` - Status ('current')

### Update Comment

```javascript
await api.asApp().requestJira(
  route`/wiki/rest/api/content/${commentId}`,
  {
    method: 'PUT',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      id: commentId,
      type: 'comment',
      version: { number: 2, message: 'Update description' },
      body: {
        storage: {
          value: '<p>Updated comment.</p>',
          representation: 'storage'
        }
      }
    })
  }
);
```

### Delete Comment

```javascript
await api.asApp().requestJira(
  route`/wiki/rest/api/content/${commentId}`,
  { method: 'DELETE' }
);
```

---

## Attachment Operations

### Get Attachments

```javascript
const response = await api.asApp().requestJira(
  route`/wiki/rest/api/content/${contentId}/child/attachment?expand=body.storage`
);

const data = await response.json();
data.results.forEach(attachment => {
  console.log(`${attachment.title} (${attachment.id})`);
});
```

**Query Parameters:**
- `start` - Pagination start index
- `limit` - Results per page
- `expand` - Fields to expand

### Get Attachment Details

```javascript
await api.asApp().requestJira(
  route`/wiki/rest/api/content/${contentId}/child/attachment/${attachmentId}`
);
```

### Upload Attachment

```javascript
const formData = new FormData();
formData.append('file', fileBlob);
formData.append('comment', 'File comment');
formData.append('minorEdit', 'true');

await api.asApp().requestJira(
  route`/wiki/rest/api/content/${contentId}/child/attachment`,
  {
    method: 'POST',
    headers: { 'X-Atlassian-Token': 'no-check' },
    body: formData
  }
);
```

### Download Attachment

```javascript
const response = await api.asApp().requestJira(
  route`/wiki/rest/api/content/${contentId}/child/attachment/${attachmentId}/body`
);

const content = await response.text();
```

### Delete Attachment

```javascript
await api.asApp().requestJira(
  route`/wiki/rest/api/content/${contentId}/child/attachment/${attachmentId}?version=1`,
  { method: 'DELETE' }
);
```

---

## Label Operations

### Add Labels to Content

```javascript
await api.asApp().requestJira(
  route`/wiki/rest/api/content/${contentId}/label`,
  {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify([
      { id: 'my-label' }
    ])
  }
);
```

**Request Body:**
```json
[
  { "id": "label-name" },
  { "prefix": "global", "name": "priority-high" }
]
```

### Get Content Labels

```javascript
await api.asApp().requestJira(
  route`/wiki/rest/api/content/${contentId}/label`
);
```

### Delete Label

```javascript
await api.asApp().requestJira(
  route`/wiki/rest/api/content/${contentId}/label?name=tagName`,
  { method: 'DELETE' }
);
```

---

## User & Permission Operations

### GetAuthenticated User

```javascript
await api.asApp().requestJira('/wiki/rest/api/user');
```

**Response includes:**
- `accountId` - User's account ID
- `displayName` - Display name
- `emailAddress` - Email address
- `active` - Whether user is active

### Get Space Permissions

```javascript
await api.asApp().requestJira(
  route`/wiki/rest/api/space/${spaceKey}/permission`
);
```

**Response includes:**
- `permissions` - List of permission objects
- Each permission has: operation, subject,restrictions

### Add User to Space

```javascript
await api.asApp().requestJira(
  route`/wiki/rest/api/space/${spaceKey}/permission`,
  {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      permissions: [
        {
          operation: 'VIEW',
          subject: {
            type: 'User',
            identifier: 'user-account-id'
          }
        }
      ]
    })
  }
);
```

### Remove User from Space

```javascript
await api.asApp().requestJira(
  route`/wiki/rest/api/space/${spaceKey}/permission?permissionId=123`,
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
  "status": 400,
  "statusCode": 400,
  "message": "Content with id '123' does not exist"
}
```

### Example Error Handling

```javascript
try {
  const response = await api.asApp().requestJira(
    route`/wiki/rest/api/content/${pageId}`
  );

  if (!response.ok) {
    const error = await response.json();
    console.error('Confluence API Error:', error);
    
    if (error.status === 403) {
      // Insufficient permissions
    }
    return;
  }

  const data = await response.json();
} catch (error) {
  console.error('Network Error:', error);
}
```

---

## Forge Module Integration

### Confluence Macro Example

```javascript
import api from '@forge/api';

export default async function (req) {
  const { contentId, spaceKey } = req.body;
  
  // Get page details via Confluence API
  const response = await api.asApp().requestJira(
    route`/wiki/rest/api/content/${contentId}?expand=body.storage`
  );
  
  const page = await response.json();
  
  return {
    title: 'Page Details',
    content: page.body.storage.value,
    status: 'success'
  };
}
```

### Confluence Event Handling

```javascript
export default async function (req) {
  const { event } = req.body;
  
  if (event.key === 'content-created') {
    // Handle content creation
    await handleContentCreated(event);
  } else if (event.key === 'space-updated') {
    // Handle space update
    await handleSpaceUpdated(event);
  }
}