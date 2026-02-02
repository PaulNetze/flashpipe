# Azure Pipelines Integration

FlashPipe integrates with Azure Pipelines for CI/CD workflows with SAP Cloud Integration.

## Table of Contents

- [Quick Start](#quick-start)
- [Setup](#setup)
- [Pipeline Examples](#pipeline-examples)
- [Variable Groups](#variable-groups)
- [Troubleshooting](#troubleshooting)

---

## Quick Start

**1. Create pipeline file:**

Add `azure-pipelines.yml` to repository root:

```yaml
trigger:
  - main

pool:
  vmImage: 'ubuntu-latest'

variables:
  - group: cpi-dev

resources:
  containers:
    - container: flashpipe
      image: engswee/flashpipe:latest

jobs:
  - job: deploy
    container: flashpipe
    steps:
      - bash: |
          flashpipe orchestrator --update --deploy-config ./deploy-config.yml
        env:
          FLASHPIPE_TMN_HOST: $(CPI_HOST)
          FLASHPIPE_OAUTH_HOST: $(CPI_OAUTH_HOST)
          FLASHPIPE_OAUTH_CLIENTID: $(CPI_CLIENT_ID)
          FLASHPIPE_OAUTH_CLIENTSECRET: $(CPI_CLIENT_SECRET)
```

**2. Create variable group:**

In Azure DevOps:
- Go to **Pipelines** → **Library** → **+ Variable group**
- Name: `cpi-dev`
- Add variables (mark secrets as secret):
  - `CPI_HOST`
  - `CPI_OAUTH_HOST`
  - `CPI_CLIENT_ID`
  - `CPI_CLIENT_SECRET` (lock/secret)

**3. Commit and run!**

---

## Setup

### Prerequisites

- Azure DevOps project
- Git repository with SAP CPI artifacts
- SAP CPI tenant with OAuth client configured
- FlashPipe Docker image access

### Authentication

**OAuth (Recommended):**

Create OAuth client in SAP BTP (see [OAuth Client Setup](oauth_client.md)).

**Variable group variables:**
- `CPI_HOST` - e.g., `tenant.hana.ondemand.com`
- `CPI_OAUTH_HOST` - e.g., `tenant.authentication.eu10.hana.ondemand.com`
- `CPI_CLIENT_ID` - OAuth client ID
- `CPI_CLIENT_SECRET` - OAuth client secret (mark as secret)

---

## Pipeline Examples

### 1. Deploy with Orchestrator

```yaml
trigger:
  - main

pool:
  vmImage: 'ubuntu-latest'

variables:
  - group: cpi-credentials

resources:
  containers:
    - container: flashpipe
      image: engswee/flashpipe:latest

jobs:
  - job: deploy
    container: flashpipe
    steps:
      - bash: |
          flashpipe orchestrator --update \
            --deploy-config ./deploy-config.yml
        displayName: 'Deploy Integration Flows'
        env:
          FLASHPIPE_TMN_HOST: $(CPI_HOST)
          FLASHPIPE_OAUTH_HOST: $(CPI_OAUTH_HOST)
          FLASHPIPE_OAUTH_CLIENTID: $(CPI_CLIENT_ID)
          FLASHPIPE_OAUTH_CLIENTSECRET: $(CPI_CLIENT_SECRET)
```

### 2. Partner Directory Deployment

```yaml
trigger:
  - main
  paths:
    include:
      - partner-directory/**

pool:
  vmImage: 'ubuntu-latest'

variables:
  - group: cpi-credentials

resources:
  containers:
    - container: flashpipe
      image: engswee/flashpipe:latest

jobs:
  - job: deploy_pd
    container: flashpipe
    steps:
      - bash: |
          flashpipe pd-deploy --dry-run
          flashpipe pd-deploy
        displayName: 'Deploy Partner Directory'
        env:
          FLASHPIPE_TMN_HOST: $(CPI_HOST)
          FLASHPIPE_OAUTH_HOST: $(CPI_OAUTH_HOST)
          FLASHPIPE_OAUTH_CLIENTID: $(CPI_CLIENT_ID)
          FLASHPIPE_OAUTH_CLIENTSECRET: $(CPI_CLIENT_SECRET)
```

### 3. Multi-Environment Deployment

```yaml
trigger:
  - main

pool:
  vmImage: 'ubuntu-latest'

stages:
  - stage: DeployDev
    displayName: 'Deploy to DEV'
    variables:
      - group: cpi-dev
    jobs:
      - deployment: DeployDev
        container: flashpipe
        environment: development
        strategy:
          runOnce:
            deploy:
              steps:
                - bash: |
                    flashpipe orchestrator --update \
                      --deployment-prefix "DEV_" \
                      --deploy-config ./deploy-config.yml
                  env:
                    FLASHPIPE_TMN_HOST: $(CPI_HOST)
                    FLASHPIPE_OAUTH_HOST: $(CPI_OAUTH_HOST)
                    FLASHPIPE_OAUTH_CLIENTID: $(CPI_CLIENT_ID)
                    FLASHPIPE_OAUTH_CLIENTSECRET: $(CPI_CLIENT_SECRET)

  - stage: DeployQA
    displayName: 'Deploy to QA'
    dependsOn: DeployDev
    variables:
      - group: cpi-qa
    jobs:
      - deployment: DeployQA
        container: flashpipe
        environment: qa
        strategy:
          runOnce:
            deploy:
              steps:
                - bash: |
                    flashpipe orchestrator --update \
                      --deployment-prefix "QA_" \
                      --deploy-config ./deploy-config.yml
                  env:
                    FLASHPIPE_TMN_HOST: $(CPI_HOST)
                    FLASHPIPE_OAUTH_HOST: $(CPI_OAUTH_HOST)
                    FLASHPIPE_OAUTH_CLIENTID: $(CPI_CLIENT_ID)
                    FLASHPIPE_OAUTH_CLIENTSECRET: $(CPI_CLIENT_SECRET)

  - stage: DeployProd
    displayName: 'Deploy to PROD'
    dependsOn: DeployQA
    variables:
      - group: cpi-prod
    jobs:
      - deployment: DeployProd
        container: flashpipe
        environment: production
        strategy:
          runOnce:
            deploy:
              steps:
                - bash: |
                    flashpipe orchestrator --update \
                      --deployment-prefix "PROD_" \
                      --deploy-config ./deploy-config.yml
                  env:
                    FLASHPIPE_TMN_HOST: $(CPI_HOST)
                    FLASHPIPE_OAUTH_HOST: $(CPI_OAUTH_HOST)
                    FLASHPIPE_OAUTH_CLIENTID: $(CPI_CLIENT_ID)
                    FLASHPIPE_OAUTH_CLIENTSECRET: $(CPI_CLIENT_SECRET)
```

### 4. Conditional Deployment

```yaml
trigger:
  - main
  paths:
    include:
      - packages/**
      - deploy-config.yml

pool:
  vmImage: 'ubuntu-latest'

variables:
  - group: cpi-credentials

resources:
  containers:
    - container: flashpipe
      image: engswee/flashpipe:latest

jobs:
  - job: deploy
    container: flashpipe
    steps:
      - bash: |
          CHANGED=$(git diff --name-only HEAD~1 HEAD | grep "packages/" || true)
          if [ -n "$CHANGED" ]; then
            echo "##vso[task.setvariable variable=shouldDeploy]true"
          fi
        displayName: 'Check for changes'
      
      - bash: |
          flashpipe orchestrator --update --deploy-config ./deploy-config.yml
        displayName: 'Deploy if changed'
        condition: eq(variables['shouldDeploy'], 'true')
        env:
          FLASHPIPE_TMN_HOST: $(CPI_HOST)
          FLASHPIPE_OAUTH_HOST: $(CPI_OAUTH_HOST)
          FLASHPIPE_OAUTH_CLIENTID: $(CPI_CLIENT_ID)
          FLASHPIPE_OAUTH_CLIENTSECRET: $(CPI_CLIENT_SECRET)
```

---

## Variable Groups

### Creating Variable Groups

1. Navigate to **Pipelines** → **Library**
2. Click **+ Variable group**
3. Name the group (e.g., `cpi-dev`)
4. Add variables:

| Variable Name | Example Value | Secret |
|---------------|---------------|--------|
| `CPI_HOST` | `tenant.hana.ondemand.com` | No |
| `CPI_OAUTH_HOST` | `tenant.authentication.eu10.hana.ondemand.com` | No |
| `CPI_CLIENT_ID` | `sb-client-xxx` | No |
| `CPI_CLIENT_SECRET` | `secret-value-xxx` | **Yes** |

5. Click the lock icon on `CPI_CLIENT_SECRET` to mark as secret
6. Save the variable group

### Using Variable Groups

Reference in pipeline:

```yaml
variables:
  - group: cpi-dev

jobs:
  - job: deploy
    steps:
      - bash: |
          flashpipe orchestrator --update
        env:
          FLASHPIPE_TMN_HOST: $(CPI_HOST)
          FLASHPIPE_OAUTH_CLIENTID: $(CPI_CLIENT_ID)
```

### Multiple Environments

Create separate variable groups:
- `cpi-dev`
- `cpi-qa`
- `cpi-prod`

Use in different stages:

```yaml
stages:
  - stage: DeployDev
    variables:
      - group: cpi-dev
  
  - stage: DeployProd
    variables:
      - group: cpi-prod
```

---

## Troubleshooting

### Pipeline Not Triggering

**Check:**
- Trigger branch name matches (`main` vs `master`)
- Path filters are correct
- Pipeline is not disabled
- Service connection is configured

### Container Image Pull Errors

```
Error: Failed to pull image engswee/flashpipe:latest
```

**Solution:**
- Use specific version tag instead of `latest`
- Check Docker Hub availability
- Verify network connectivity

### Authentication Errors

```
Error: failed to get OAuth token
```

**Solution:**
- Verify variable group name matches pipeline
- Check variable names are correct (case-sensitive)
- Ensure `CPI_CLIENT_SECRET` is marked as secret
- Validate OAuth client permissions

### Permission Issues

```
Error: ##[error]Bash exited with code '1'.
```

**Solution:**
- Check pipeline has permissions to environment
- Verify service connections
- Review Azure DevOps project permissions

### Debug Logging

Enable debug mode:

```yaml
- bash: |
    flashpipe orchestrator --update --deploy-config ./deploy-config.yml
  env:
    FLASHPIPE_DEBUG: true
    FLASHPIPE_TMN_HOST: $(CPI_HOST)
```

View detailed logs in pipeline run.

---

## Best Practices

✅ **DO:**
- Use variable groups for credentials
- Mark secrets as secret (lock icon)
- Use specific FlashPipe image versions
- Implement multi-stage deployments
- Use Azure DevOps environments for approvals
- Add path filters to triggers

❌ **DON'T:**
- Hardcode credentials in pipeline
- Skip `--dry-run` for production
- Use `latest` tag in production
- Deploy on every commit unnecessarily
- Share variable groups across unrelated projects

---

## See Also

- [OAuth Client Setup](oauth_client.md) - Configure SAP BTP OAuth
- [Orchestrator](orchestrator.md) - High-level deployment command
- [GitHub Actions](github-actions.md) - GitHub Actions integration
- [FlashPipe CLI](flashpipe-cli.md) - Complete command reference