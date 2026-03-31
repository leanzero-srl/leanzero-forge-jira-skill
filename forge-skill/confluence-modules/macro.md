# Confluence Macro Module

## Overview

The `macro` module allows Forge apps to embed custom content into Confluence pages using a rich UI editor.

## Module Configuration

```yaml
modules:
  macro:
    - key: my-macro               # Required: Unique identifier
      name: Data Visualization    # Required: Display name
      description: Shows real-time data from external API # Required: Description
      
      icon:
        url: /static/icon.png
        width: 24
        height: 24
        
      render: native              # OR: custom (for React/HTML)
      resolver:
        function: resolveMacro
    
      edit:
        resource: macro-config
```

## Two Render Types

### Native UI Kit Rendering

```yaml
modules:
  macro:
    - key: ui-kit-macro
      name: Status Panel
      description: Displays issue status
      render: native
      resolver:
        function: resolveStatus
        
      create:
        resource: config-ui
```

```javascript
// src/index.js
export const resolveStatus = async (args) => {
  const { contentId, macro } = args;
  
  // Get issue data from Jira or database
  const statusData = await getStatus(contentId);
  
  return {
    type: "jiraIssue",
    contentId,
    fields: {
      summary: true,
      status: true
    }
  };
};
```

### Custom UI Rendering (React/HTML)

```yaml
modules:
  macro:
    - key: custom-macro
      name: Analytics Dashboard
      description: Custom analytics visualization
      render: custom
      resolver:
        function: resolveDashboard
        
      create:
        resource: config-ui
```

```javascript
// src/index.js
export const resolveDashboard = async (args) => {
  const { contentId, macro } = args;
  
  // Your custom data processing
  return {
    html: '<div class="dashboard">Your HTML content</div>'
  };
};
```

## Macro Configuration UI

```javascript
import React from 'react';
import { Form, Select } from '@forge/bridge';

const onConfigureFn = async () => {
  const config = {
    theme: document.getElementById('theme').value,
    refreshInterval: parseInt(document.getElementById('interval').value)
  };
  
  return JSON.stringify(config);
};

// In your React component
export default function MacroConfig() {
  return (
    <Form>
      <Select name="theme" label="Theme">
        <option value="light">Light</option>
        <option value="dark">Dark</option>
      </Select>
      
      <Input name="interval" label="Refresh interval (seconds)" />
    </Form>
  );
}
```

## Permissions Required

```yaml
permissions:
  scopes:
    - read:confluence-content
    - write:confluence-content
```

For Jira data integration:

```yaml
permissions:
  scopes:
    - read:jira-work