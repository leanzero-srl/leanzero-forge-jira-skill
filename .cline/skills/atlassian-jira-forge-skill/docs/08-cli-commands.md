# Forge CLI Commands Reference

## Overview

The Forge CLI (`forge`) is used to develop, deploy, and manage Forge apps. All commands must be run from your app's root directory.

---

## Setup & Initialization

| Command | Description |
|---------|-------------|
| `forge init` | Create a new Forge app |
| `forge --version` | Show CLI version |
| `forge login` | Log in to Atlassian account |
| `forge logout` | Log out of current session |

### Initialize New App

```bash
# Start interactive setup
forge init

# Or specify template
forge init --template react-nodejs
```

---

## Development Commands

| Command | Description |
|---------|-------------|
| `forge deploy` | Deploy app to development environment |
| `forge install` | Install app on current site |
| `forge install --upgrade` | Update installation with new version |
| `forge uninstall` | Remove app from current site |

### Deployment Flow

```bash
# Make code changes
vim src/index.js

# Deploy the app
forge deploy

# Upgrade the installation (new permissions may need approval)
forge install --upgrade

# Monitor logs
forge logs -n 50
```

---

## Local Testing

| Command | Description |
|---------|-------------|
| `forge tunnel` | Create local tunnel for testing |
| `forge status` | Show app deployment status |

### Using Forge Tunnel

```bash
# Start tunnel (keeps terminal open)
forge tunnel

# In another terminal, run other commands
# App is accessible via tunnel URL
```

---

## Log Monitoring

| Command | Description |
|---------|-------------|
| `forge logs` | View app logs |
| `forge logs -n 50` | Show last 50 log entries |
| `forge logs --follow` | Stream logs in real-time |

### Example Log Output

```bash
$ forge logs -n 10
2023-10-01T10:00:00Z [INFO] Function started: validateContent
2023-10-01T10:00:01Z [INFO] Issue key: PROJ-123
2023-10-01T10:00:02Z [DEBUG] External API call made
2023-10-01T10:00:03Z [INFO] Validation completed successfully
```

---

## Manifest & Code Quality

| Command | Description |
|---------|-------------|
| `forge lint` | Check manifest for issues |
| `forge lint --fix` | Auto-fixable issues |

### Linting Examples

```bash
# Check for errors
forge lint

# Fix auto-fixable issues
forge lint --fix
```

---

## Environment Management

| Command | Description |
|---------|-------------|
| `forge env list` | List available environments |
| `forge deploy -e production` | Deploy to specific environment |

---

## Resource & Module Inspection

| Command | Description |
|---------|-------------|
| `forge display` | Show app configuration summary |
| `forge module list` | List configured modules |

---

## Complete CLI Reference

### All Available Commands

```
forge [command] [options]

Commands:
  init                  Create a new Forge app
  deploy                Deploy app to environment
  install               Install app on site
  uninstall             Remove app from site
  tunnel                Start local development server
  logs                  Show app logs
  lint                  Check manifest for issues
  status                Show deployment status
  env                   Environment management
  display               Show app configuration
  module                Module inspection
  help                  Show help

Options:
  -e, --env <name>      Target environment (default: development)
  -n, --number <count>  Number of log entries to show
  --follow              Stream logs in real-time
```

---

## Common Development Workflow

### Initial Setup

```bash
# 1. Create new app
forge init my-forge-app
cd my-forge-app

# 2. Add your custom logic
vim src/index.js

# 3. Update manifest.yml with modules
vim manifest.yml

# 4. Deploy to development
forge deploy

# 5. Install on site
forge install --upgrade
```

### Ongoing Development

```bash
# 1. Make code changes
# 2. Redeploy
forge deploy

# 3. Test in browser (with tunnel running)
# 4. Check logs for errors
forge logs -n 50

# 5. Fix any issues and repeat
```

---

## Environment Variables

For sensitive data like API keys:

```bash
# Set environment variable
export MY_API_KEY="secret-value"

# In your function:
import { env } from '@forge/api';
const apiKey = process.env.MY_API_KEY;
```

Or use Forge's built-in secret storage (if available in your environment).

---

## Next Steps

- **Permissions**: Understand what scopes are needed for your modules
- **API Endpoints**: Learn how to make REST API calls