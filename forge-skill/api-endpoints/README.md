# Forge API Endpoints Documentation

## Overview

This directory contains detailed documentation for Forge and Atlassian REST API endpoints.

## Files

| File | Description |
|------|-------------|
| [forge-runtime-apis.md](./forge-runtime-apis.md) | Forge platform runtime APIs |
| [jira-rest-api.md](./jira-rest-api.md) | Jira REST API reference |
| [bitbucket-rest-api.md](./bitbucket-rest-api.md) | Bitbucket REST API reference |
| [confluence-rest-api.md](./confluence-rest-api.md) | Confluence REST API reference |

## Quick Reference

### Forge APIs
- `@forge/api` - Core runtime, Jira/Confluence REST calls
- `@forge/resolver` - Frontend-to-backend communication
- `@forge/jira-bridge` - UI configuration and modifications
- `@forge/kvs` - Key-value storage

### Atlassian APIs
- **Jira REST API** - Issue, project, workflow operations
- **Bitbucket REST API** - Repository, PR, merge check operations
- **Confluence REST API** - Page, space, content operations

## Learning Path

1. Start with `forge-runtime-apis.md` for core functionality
2. Review Jira API for issue management
3. Check Bitbucket/Confluence APIs as needed