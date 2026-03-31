# Confluence Full Page Module

## Overview

The `confluence:fullPage` module allows Forge apps to add standalone pages accessible from the Confluence sidebar.

## Module Configuration

```yaml
modules:
  confluence:fullPage:
    - key: my-full-page           # Required: Unique identifier
      name: Analytics Dashboard   # Required: Display name
      description: View analytics data # Required: Description
      
      icon:
        url: /static/icon.png
        width: 24
        height: 24
        
      routePrefix: dashboard    # URL path prefix
      
      create:
        resource: fullpage-ui
      edit:
        resource: fullpage-ui
```

## Page Configuration

```yaml
modules:
  confluence:fullPage:
    - key: analytics-page
      name: Analytics Dashboard
      description: Analytics and metrics visualization
      icon:
        url: /static/icon.png
      routePrefix: analytics
      
      create:
        resource: main
      edit:
        resource: main
        
  function:
    - key: resolveDashboard
      handler: index.resolveDashboard

resources:
  - key: main
    path: src/fullpage/build
```

## Route Structure

Pages are accessible at: `{site-url}/plugins/servlet/ac/{app-id}/{routePrefix}`

Example:
- App ID: `ari:cloud:ecosystem::app/123`
- Route Prefix: `analytics`
- URL: `https://your-site.atlassian.net/plugins/servlet/ac/ari:cloud:ecosystem::app/123/analytics`

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