# Partner Directory

Manage SAP Cloud Integration Partner Directory parameters with version control and automation.

## Table of Contents

- [Quick Start](#quick-start)
- [Commands](#commands)
- [Configuration](#configuration)
- [File Structure](#file-structure)
- [Examples](#examples)
- [CI/CD Integration](#cicd-integration)
- [Troubleshooting](#troubleshooting)

---

## Quick Start

**1. Snapshot (download) parameters from SAP CPI:**

```bash
flashpipe pd-snapshot \
  --tmn-host "tenant.hana.ondemand.com" \
  --oauth-host "tenant.authentication.eu10.hana.ondemand.com" \
  --oauth-clientid "your-client-id" \
  --oauth-clientsecret "your-client-secret"
```

**2. Deploy (upload) parameters to SAP CPI:**

```bash
flashpipe pd-deploy \
  --tmn-host "tenant.hana.ondemand.com" \
  --oauth-host "tenant.authentication.eu10.hana.ondemand.com" \
  --oauth-clientid "your-client-id" \
  --oauth-clientsecret "your-client-secret"
```

Parameters are stored in `./partner-directory` by default.

---

## Commands

### pd-snapshot

Download Partner Directory parameters from SAP CPI to local files.

**Syntax:**
```bash
flashpipe pd-snapshot [flags]
```

**Flags:**

| Flag | Type | Default | Description |
|------|------|---------|-------------|
| `--resources-path` | string | `./partner-directory` | Local directory path |
| `--replace` | bool | `true` | Overwrite existing local files |
| `--pids` | string | `""` | Filter specific Partner IDs (comma-separated) |

**Examples:**

```bash
# Download all parameters
flashpipe pd-snapshot

# Download to custom path
flashpipe pd-snapshot --resources-path "./my-pd-params"

# Download specific Partner IDs
flashpipe pd-snapshot --pids "PID_001,PID_002"

# Add-only mode (preserve existing local values)
flashpipe pd-snapshot --replace=false
```

### pd-deploy

Upload Partner Directory parameters from local files to SAP CPI.

**Syntax:**
```bash
flashpipe pd-deploy [flags]
```

**Flags:**

| Flag | Type | Default | Description |
|------|------|---------|-------------|
| `--resources-path` | string | `./partner-directory` | Local directory path |
| `--replace` | bool | `true` | Update existing remote parameters |
| `--full-sync` | bool | `false` | Delete remote parameters not in local |
| `--dry-run` | bool | `false` | Preview changes without applying |
| `--pids` | string | `""` | Filter specific Partner IDs (comma-separated) |

**Examples:**

```bash
# Deploy all parameters
flashpipe pd-deploy

# Dry run to preview changes
flashpipe pd-deploy --dry-run

# Deploy specific Partner IDs
flashpipe pd-deploy --pids "PID_001,PID_002"

# Add-only mode (don't update existing)
flashpipe pd-deploy --replace=false

# Full sync (delete remote not in local)
flashpipe pd-deploy --full-sync
```

---

## Configuration

### Global Config File

Create `flashpipe.yaml`:

```yaml
# OAuth Authentication (recommended)
tmn-host: tenant.hana.ondemand.com
oauth-host: tenant.authentication.sap.hana.ondemand.com
oauth-clientid: ${OAUTH_CLIENT_ID}
oauth-clientsecret: ${OAUTH_CLIENT_SECRET}

# Partner Directory settings
pd-snapshot:
  resources-path: ./partner-directory
  replace: true
  pids:
    - PID_001
    - PID_002

pd-deploy:
  resources-path: ./partner-directory
  replace: true
  full-sync: false
  dry-run: false
  pids:
    - PID_001
    - PID_002
```

Then run without flags:
```bash
flashpipe pd-snapshot
flashpipe pd-deploy
```

### Deployment Modes

| Mode | Description | Use Case |
|------|-------------|----------|
| **Replace (default)** | Update existing, add new | Standard updates |
| **Add-Only** | Add new only, skip existing | Preserve manual changes |
| **Full Sync** | Mirror local to remote, delete extras | Complete sync |

**Examples:**

```bash
# Replace mode (default)
flashpipe pd-deploy --replace=true

# Add-only mode
flashpipe pd-deploy --replace=false

# Full sync
flashpipe pd-deploy --full-sync
```

---

## File Structure

Parameters are organized by Partner ID:

```
partner-directory/
├── PID_001/
│   ├── parameters/
│   │   ├── DatabaseURL.txt
│   │   ├── APIKey.txt
│   │   └── Timeout.txt
│   └── binaries/
│       ├── certificate.crt
│       └── config.xml
└── PID_002/
    └── parameters/
        └── Endpoint.txt
```

### String Parameters

Stored as `.txt` files in `parameters/` folder:

**DatabaseURL.txt:**
```
jdbc:mysql://localhost:3306/mydb
```

### Binary Parameters

Stored as-is in `binaries/` folder:
- Certificates (`.crt`, `.pem`)
- XML configs (`.xml`)
- XSLT transforms (`.xsl`)

---

## Examples

### Example 1: Initial Setup

```bash
# 1. Download from production
flashpipe pd-snapshot \
  --resources-path ./cpars-prod \
  --tmn-host prod.hana.ondemand.com

# 2. Commit to version control
git add cpars-prod/
git commit -m "Initial Partner Directory snapshot"

# 3. Deploy to dev environment
flashpipe pd-deploy \
  --resources-path ./cpars-prod \
  --tmn-host dev.hana.ondemand.com \
  --dry-run

# 4. Apply if looks good
flashpipe pd-deploy \
  --resources-path ./cpars-prod \
  --tmn-host dev.hana.ondemand.com
```

### Example 2: Environment Promotion

```bash
# DEV → QA → PROD with same parameters
export RESOURCES_PATH=./partner-directory

# Deploy to DEV
flashpipe pd-deploy --tmn-host dev-tenant.hana.ondemand.com

# Deploy to QA
flashpipe pd-deploy --tmn-host qa-tenant.hana.ondemand.com

# Deploy to PROD (with dry-run first)
flashpipe pd-deploy --tmn-host prod-tenant.hana.ondemand.com --dry-run
flashpipe pd-deploy --tmn-host prod-tenant.hana.ondemand.com
```

### Example 3: Selective Updates

```bash
# Update only specific Partner IDs
flashpipe pd-deploy \
  --pids "CRITICAL_PARTNER_01,CRITICAL_PARTNER_02" \
  --dry-run
```

---

## CI/CD Integration

### GitHub Actions

```yaml
name: Deploy Partner Directory

on:
  push:
    branches: [main]
    paths:
      - 'partner-directory/**'

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Deploy Partner Directory
        env:
          FLASHPIPE_TMN_HOST: ${{ secrets.CPI_HOST }}
          FLASHPIPE_OAUTH_HOST: ${{ secrets.CPI_OAUTH_HOST }}
          FLASHPIPE_OAUTH_CLIENTID: ${{ secrets.CPI_CLIENT_ID }}
          FLASHPIPE_OAUTH_CLIENTSECRET: ${{ secrets.CPI_CLIENT_SECRET }}
        run: |
          docker run --rm \
            -e FLASHPIPE_TMN_HOST \
            -e FLASHPIPE_OAUTH_HOST \
            -e FLASHPIPE_OAUTH_CLIENTID \
            -e FLASHPIPE_OAUTH_CLIENTSECRET \
            -v $(pwd):/workspace \
            engswee/flashpipe:latest \
            pd-deploy --dry-run
```

### Azure Pipelines

```yaml
steps:
  - task: Bash@3
    displayName: Deploy Partner Directory
    env:
      FLASHPIPE_TMN_HOST: $(CPI_HOST)
      FLASHPIPE_OAUTH_HOST: $(CPI_OAUTH_HOST)
      FLASHPIPE_OAUTH_CLIENTID: $(CPI_CLIENT_ID)
      FLASHPIPE_OAUTH_CLIENTSECRET: $(CPI_CLIENT_SECRET)
    inputs:
      targetType: inline
      script: |
        flashpipe pd-deploy --dry-run
        flashpipe pd-deploy
```

---

## Troubleshooting

### Enable Debug Logging

```bash
export FLASHPIPE_DEBUG=true
flashpipe pd-deploy
```

### Common Issues

| Issue | Solution |
|-------|----------|
| Authentication failed | Verify OAuth credentials, check token permissions |
| Permission denied | Ensure OAuth client has Partner Directory write access |
| Parameter not found | Check Partner ID spelling, verify parameter exists |
| File encoding issues | Ensure UTF-8 encoding for text files |
| Batch operation failed | Try smaller batches or check network connection |

### Always Use Dry Run First

```bash
# Preview changes before applying
flashpipe pd-deploy --dry-run
```

### Check Exit Codes

```bash
#!/bin/bash
if flashpipe pd-deploy; then
  echo "✅ Deployment successful"
else
  echo "❌ Deployment failed"
  exit 1
fi
```

---

## Best Practices

✅ **DO:**
- Version control Partner Directory parameters
- Use dry-run before deploying to production
- Test in non-production environments first
- Use OAuth authentication (not basic auth)
- Enable debug logging when troubleshooting
- Use descriptive Partner ID naming conventions

❌ **DON'T:**
- Commit sensitive credentials to Git
- Skip dry-run in production
- Deploy without testing
- Use basic authentication in production
- Manually edit parameters in SAP CPI

---

## Reference

### Global Flags

All FlashPipe commands support:
- `--config` - Path to config file
- `--debug` - Enable debug logging
- `--tmn-host` - CPI tenant host
- `--oauth-host` - OAuth token host
- `--oauth-clientid` - OAuth client ID
- `--oauth-clientsecret` - OAuth client secret

### Exit Codes

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | Error occurred |

---

## See Also

- [OAuth Client Setup](oauth_client.md) - Configure authentication
- [Orchestrator](orchestrator.md) - Full artifact deployment
- [Configure](configure.md) - Artifact parameter configuration
- [Example Config](../examples/flashpipe-cpars-example.yml) - Complete example