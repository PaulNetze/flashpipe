# GitHub Actions Integration

FlashPipe integrates seamlessly with GitHub Actions for CI/CD workflows with SAP Cloud Integration.

## Table of Contents

- [Quick Start](#quick-start)
- [Setup](#setup)
- [Workflows](#workflows)
- [Examples](#examples)
- [Secrets Management](#secrets-management)
- [Troubleshooting](#troubleshooting)

---

## Quick Start

**1. Add workflow file:**

Create `.github/workflows/deploy.yml`:

```yaml
name: Deploy to SAP CPI

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Deploy Integration Flow
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
            orchestrator --update --deploy-config ./deploy-config.yml
```

**2. Add secrets:**

Go to **Settings** → **Secrets and variables** → **Actions** → **New repository secret**:
- `CPI_HOST`
- `CPI_OAUTH_HOST`
- `CPI_CLIENT_ID`
- `CPI_CLIENT_SECRET`

**3. Commit and push** - workflow runs automatically!

---

## Setup

### Prerequisites

- GitHub repository with SAP CPI artifacts
- SAP CPI tenant with OAuth client configured
- FlashPipe Docker image access

### Authentication

**OAuth (Recommended):**

Set up OAuth client in SAP BTP (see [OAuth Client Setup](oauth_client.md)).

**Secrets to create:**
- `CPI_HOST` - e.g., `tenant.hana.ondemand.com`
- `CPI_OAUTH_HOST` - e.g., `tenant.authentication.eu10.hana.ondemand.com`
- `CPI_CLIENT_ID` - OAuth client ID
- `CPI_CLIENT_SECRET` - OAuth client secret

**Basic Auth (Legacy):**

Not recommended for production. Use OAuth instead.

---

## Workflows

### 1. Upload/Deploy Artifact

Deploy integration flows when code changes.

```yaml
name: Deploy Integration Flow

on:
  push:
    branches: [main]
    paths:
      - 'packages/**'

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Deploy with Orchestrator
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
            orchestrator --update --deploy-config ./deploy-config.yml
```

### 2. Snapshot from Tenant

Download artifacts from SAP CPI to repository.

```yaml
name: Snapshot from SAP CPI

on:
  schedule:
    - cron: '0 2 * * *'  # Daily at 2 AM
  workflow_dispatch:

jobs:
  snapshot:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Download Artifacts
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
            snapshot --package-ids "MyPackage" --artifact-ids "MyFlow"
      
      - name: Commit Changes
        run: |
          git config user.name "GitHub Actions"
          git config user.email "actions@github.com"
          git add packages/
          git diff --staged --quiet || git commit -m "Snapshot from SAP CPI [skip ci]"
          git push
```

### 3. Sync to Tenant

Keep SAP CPI in sync with Git repository.

```yaml
name: Sync to SAP CPI

on:
  push:
    branches: [main]

jobs:
  sync:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Sync Artifacts
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
            sync --packages-dir ./packages
```

### 4. Partner Directory Deployment

Deploy Partner Directory parameters.

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
          
          docker run --rm \
            -e FLASHPIPE_TMN_HOST \
            -e FLASHPIPE_OAUTH_HOST \
            -e FLASHPIPE_OAUTH_CLIENTID \
            -e FLASHPIPE_OAUTH_CLIENTSECRET \
            -v $(pwd):/workspace \
            engswee/flashpipe:latest \
            pd-deploy
```

### 5. Multi-Environment Deployment

Deploy to DEV, QA, and PROD environments.

```yaml
name: Multi-Environment Deployment

on:
  push:
    branches: [main]

jobs:
  deploy-dev:
    runs-on: ubuntu-latest
    environment: development
    steps:
      - uses: actions/checkout@v4
      
      - name: Deploy to DEV
        env:
          FLASHPIPE_TMN_HOST: ${{ secrets.DEV_CPI_HOST }}
          FLASHPIPE_OAUTH_HOST: ${{ secrets.DEV_CPI_OAUTH_HOST }}
          FLASHPIPE_OAUTH_CLIENTID: ${{ secrets.DEV_CPI_CLIENT_ID }}
          FLASHPIPE_OAUTH_CLIENTSECRET: ${{ secrets.DEV_CPI_CLIENT_SECRET }}
        run: |
          docker run --rm \
            -e FLASHPIPE_TMN_HOST \
            -e FLASHPIPE_OAUTH_HOST \
            -e FLASHPIPE_OAUTH_CLIENTID \
            -e FLASHPIPE_OAUTH_CLIENTSECRET \
            -v $(pwd):/workspace \
            engswee/flashpipe:latest \
            orchestrator --update --deployment-prefix "DEV_" \
            --deploy-config ./deploy-config.yml

  deploy-qa:
    needs: deploy-dev
    runs-on: ubuntu-latest
    environment: qa
    steps:
      - uses: actions/checkout@v4
      
      - name: Deploy to QA
        env:
          FLASHPIPE_TMN_HOST: ${{ secrets.QA_CPI_HOST }}
          FLASHPIPE_OAUTH_HOST: ${{ secrets.QA_CPI_OAUTH_HOST }}
          FLASHPIPE_OAUTH_CLIENTID: ${{ secrets.QA_CPI_CLIENT_ID }}
          FLASHPIPE_OAUTH_CLIENTSECRET: ${{ secrets.QA_CPI_CLIENT_SECRET }}
        run: |
          docker run --rm \
            -e FLASHPIPE_TMN_HOST \
            -e FLASHPIPE_OAUTH_HOST \
            -e FLASHPIPE_OAUTH_CLIENTID \
            -e FLASHPIPE_OAUTH_CLIENTSECRET \
            -v $(pwd):/workspace \
            engswee/flashpipe:latest \
            orchestrator --update --deployment-prefix "QA_" \
            --deploy-config ./deploy-config.yml

  deploy-prod:
    needs: deploy-qa
    runs-on: ubuntu-latest
    environment: production
    steps:
      - uses: actions/checkout@v4
      
      - name: Deploy to PROD
        env:
          FLASHPIPE_TMN_HOST: ${{ secrets.PROD_CPI_HOST }}
          FLASHPIPE_OAUTH_HOST: ${{ secrets.PROD_CPI_OAUTH_HOST }}
          FLASHPIPE_OAUTH_CLIENTID: ${{ secrets.PROD_CPI_CLIENT_ID }}
          FLASHPIPE_OAUTH_CLIENTSECRET: ${{ secrets.PROD_CPI_CLIENT_SECRET }}
        run: |
          docker run --rm \
            -e FLASHPIPE_TMN_HOST \
            -e FLASHPIPE_OAUTH_HOST \
            -e FLASHPIPE_OAUTH_CLIENTID \
            -e FLASHPIPE_OAUTH_CLIENTSECRET \
            -v $(pwd):/workspace \
            engswee/flashpipe:latest \
            orchestrator --update --deployment-prefix "PROD_" \
            --deploy-config ./deploy-config.yml
```

---

## Examples

### Build Artifacts on Main Branch

Create build artifacts and upload them to GitHub.

```yaml
name: Build Artifacts

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Set up Go
        uses: actions/setup-go@v5
        with:
          go-version-file: 'go.mod'
      
      - name: Build All Platforms
        run: make build-all
      
      - name: Upload Windows Binary
        uses: actions/upload-artifact@v4
        with:
          name: flashpipe-windows-amd64
          path: bin/flashpipe-windows-amd64.exe
      
      - name: Upload Linux Binary
        uses: actions/upload-artifact@v4
        with:
          name: flashpipe-linux-amd64
          path: bin/flashpipe-linux-amd64
      
      - name: Upload macOS Binary
        uses: actions/upload-artifact@v4
        with:
          name: flashpipe-darwin-amd64
          path: bin/flashpipe-darwin-amd64
```

### Conditional Deployment

Deploy only when specific files change.

```yaml
name: Conditional Deploy

on:
  push:
    branches: [main]
    paths:
      - 'packages/MyPackage/**'
      - 'deploy-config.yml'

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 2
      
      - name: Check Changed Files
        id: changed
        run: |
          CHANGED=$(git diff --name-only HEAD~1 HEAD | grep "packages/MyPackage" || true)
          if [ -n "$CHANGED" ]; then
            echo "deploy=true" >> $GITHUB_OUTPUT
          fi
      
      - name: Deploy if Changed
        if: steps.changed.outputs.deploy == 'true'
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
            orchestrator --update --package-filter "MyPackage"
```

---

## Secrets Management

### Creating Secrets

1. Navigate to repository **Settings**
2. Click **Secrets and variables** → **Actions**
3. Click **New repository secret**
4. Add each secret:

| Secret Name | Example Value | Description |
|-------------|---------------|-------------|
| `CPI_HOST` | `tenant.hana.ondemand.com` | SAP CPI tenant host |
| `CPI_OAUTH_HOST` | `tenant.authentication.eu10.hana.ondemand.com` | OAuth token host |
| `CPI_CLIENT_ID` | `sb-client-id-xxx` | OAuth client ID |
| `CPI_CLIENT_SECRET` | `secret-value-xxx` | OAuth client secret |

### Using Environment Variables

Access secrets via `${{ secrets.SECRET_NAME }}`:

```yaml
env:
  FLASHPIPE_TMN_HOST: ${{ secrets.CPI_HOST }}
  FLASHPIPE_OAUTH_HOST: ${{ secrets.CPI_OAUTH_HOST }}
  FLASHPIPE_OAUTH_CLIENTID: ${{ secrets.CPI_CLIENT_ID }}
  FLASHPIPE_OAUTH_CLIENTSECRET: ${{ secrets.CPI_CLIENT_SECRET }}
```

### Environment-Specific Secrets

Use GitHub Environments for different deployment targets:

**Setup:**
1. Go to **Settings** → **Environments**
2. Create environments: `development`, `qa`, `production`
3. Add secrets to each environment

**Usage:**
```yaml
jobs:
  deploy-dev:
    environment: development
    steps:
      - name: Deploy
        env:
          FLASHPIPE_TMN_HOST: ${{ secrets.CPI_HOST }}
```

---

## Troubleshooting

### Workflow Not Triggering

**Check:**
- Branch name matches trigger (`main` vs `master`)
- File paths are correct
- Workflow file is in `.github/workflows/`
- Workflow is enabled (Actions tab)

### Authentication Errors

```
Error: failed to get OAuth token
```

**Solution:**
- Verify secret names match exactly
- Check OAuth client permissions
- Ensure OAuth host doesn't include `https://`
- Validate token endpoint path

### Docker Permission Issues

```
Error: permission denied while trying to connect to Docker daemon
```

**Solution:**
Use GitHub's runner which includes Docker by default. No additional setup needed.

### Timeout Issues

```
Error: The job running on runner has exceeded the maximum execution time
```

**Solution:**
```yaml
jobs:
  deploy:
    timeout-minutes: 30  # Default is 360
```

### Debug Logging

Enable debug mode:

```yaml
- name: Deploy with Debug
  env:
    FLASHPIPE_DEBUG: true
  run: |
    docker run --rm \
      -e FLASHPIPE_DEBUG \
      engswee/flashpipe:latest \
      orchestrator --update
```

---

## Best Practices

✅ **DO:**
- Use GitHub Environments for multi-stage deployments
- Store credentials in GitHub Secrets
- Use workflow triggers appropriately (push, schedule, manual)
- Add descriptive commit messages
- Test in non-production first
- Use specific FlashPipe image tags (not `latest`)

❌ **DON'T:**
- Hardcode credentials in workflows
- Skip `--dry-run` for production deployments
- Deploy on every commit (use path filters)
- Share secrets across repositories unnecessarily
- Use basic authentication in production

---

## See Also

- [OAuth Client Setup](oauth_client.md) - Configure SAP BTP OAuth
- [Orchestrator](orchestrator.md) - High-level deployment command
- [Azure Pipelines](azure-pipelines.md) - Azure DevOps integration
- [FlashPipe CLI](flashpipe-cli.md) - Complete command reference