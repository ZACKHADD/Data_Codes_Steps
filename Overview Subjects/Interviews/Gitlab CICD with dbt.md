# Complete dbt Commands Reference & Slim CI Configuration

## Essential dbt Commands

### Core Development Commands

#### `dbt run`
Executes SQL models in your dbt project.
```bash
# Run all models
dbt run

# Run specific models
dbt run --models my_model
dbt run --models +my_model  # Include upstream dependencies
dbt run --models my_model+  # Include downstream dependencies
dbt run --models +my_model+ # Include both upstream and downstream

# Run models by tag
dbt run --models tag:daily
dbt run --models tag:finance,tag:marketing

# Run models in specific directory
dbt run --models models/staging
dbt run --models staging.salesforce

# Exclude specific models
dbt run --exclude my_model
dbt run --exclude tag:deprecated

# Run with threads (parallel execution)
dbt run --threads 4
```

#### `dbt test`
Runs tests defined in your project.
```bash
# Run all tests
dbt test

# Run tests for specific models
dbt test --models my_model
dbt test --models staging.users

# Run specific test types
dbt test --select test_type:generic
dbt test --select test_type:singular

# Run tests with store_failures (save failed test results)
dbt test --store-failures
```

#### `dbt build`
Runs models, tests, snapshots, and seeds in dependency order.
```bash
# Build everything
dbt build

# Build specific selection
dbt build --models +my_model+
dbt build --select tag:daily
```

### Model Selection and Graph Operations

#### Advanced Selection Syntax
```bash
# Multiple model selection
dbt run --models model_a model_b model_c

# Graph operators
dbt run --models @my_model      # @ operator (shorthand for +model+)
dbt run --models 1+my_model     # 1 degree upstream
dbt run --models my_model+2     # 2 degrees downstream
dbt run --models 2+my_model+1   # 2 upstream, 1 downstream

# Resource type selection
dbt build --select resource_type:model
dbt build --select resource_type:test
dbt build --select resource_type:snapshot
dbt build --select resource_type:seed

# Package selection
dbt run --models package:my_package
```

### Development & Debugging Commands

#### `dbt compile`
Compiles dbt models to raw SQL without executing.
```bash
dbt compile
dbt compile --models my_model
```

#### `dbt parse`
Parses dbt project and updates manifest.json.
```bash
dbt parse
```

#### `dbt ls` (list)
Lists resources in your dbt project.
```bash
# List all models
dbt ls --resource-type model

# List models with selection
dbt ls --models staging.*
dbt ls --select tag:daily

# Output formats
dbt ls --output json
dbt ls --output name
dbt ls --output path
```

#### `dbt show`
Preview SQL results without materializing.
```bash
# Show compiled SQL and sample results
dbt show --models my_model

# Limit number of rows
dbt show --models my_model --limit 10
```

### Data Management Commands

#### `dbt seed`
Loads CSV files from data/ directory into your warehouse.
```bash
# Load all seeds
dbt seed

# Load specific seed
dbt seed --select my_seed_file

# Full refresh (drop and recreate)
dbt seed --full-refresh
```

#### `dbt snapshot`
Executes snapshot models for slowly changing dimensions.
```bash
# Run all snapshots
dbt snapshot

# Run specific snapshot
dbt snapshot --select my_snapshot
```

#### `dbt run-operation`
Executes macros directly.
```bash
# Run a macro
dbt run-operation my_macro

# Run macro with arguments
dbt run-operation grant_select --args '{table: "my_table", role: "reporting"}'
```

### Documentation & Lineage

#### `dbt docs`
```bash
# Generate documentation
dbt docs generate

# Serve documentation locally
dbt docs serve --port 8080

# Generate and serve
dbt docs generate && dbt docs serve
```

### Environment & Configuration

#### `dbt debug`
Validates your dbt installation and project configuration.
```bash
dbt debug
```

#### `dbt deps`
Downloads dependencies specified in packages.yml.
```bash
dbt deps
```

#### `dbt clean`
Removes dbt artifacts (target/, logs/, dbt_packages/).
```bash
dbt clean
```

### State-based Commands (Slim CI)

#### `dbt run --state`
```bash
# Run modified models only
dbt run --models state:modified --state path/to/manifest

# Run new models
dbt run --models state:new --state ./prod-manifest

# Run modified and downstream
dbt run --models state:modified+ --state ./target-prod
```

## Slim CI Configuration

### What is Slim CI?

Slim CI is a dbt feature that enables running only the modified models and their dependencies, dramatically reducing CI/CD execution time and cost. It compares the current state of your project against a previous state (usually production) to determine what has changed.

### Prerequisites for Slim CI

1. **State artifacts**: You need manifest.json from your production environment
2. **dbt version**: 0.18.0 or higher
3. **Git workflow**: Changes detected through file modifications

### GitHub Actions Slim CI Configuration

```yaml
# .github/workflows/dbt_ci.yml
name: dbt CI/CD Pipeline

on:
  pull_request:
    branches: [ main, master ]
  push:
    branches: [ main, master ]

env:
  DBT_PROFILES_DIR: ./
  DBT_PROJECT_DIR: ./

jobs:
  dbt-ci:
    runs-on: ubuntu-latest
    
    services:
      postgres:
        image: postgres:13
        env:
          POSTGRES_PASSWORD: postgres
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    steps:
    - name: Checkout code
      uses: actions/checkout@v4
      with:
        fetch-depth: 0  # Important: fetch full history for state comparison

    - name: Setup Python
      uses: actions/setup-python@v4
      with:
        python-version: '3.9'

    - name: Install dependencies
      run: |
        pip install dbt-postgres>=1.0.0
        dbt deps

    - name: Download production artifacts
      run: |
        # Download manifest.json from production
        # This could be from S3, GCS, or artifact storage
        aws s3 cp s3://my-bucket/prod-artifacts/manifest.json ./prod-manifest.json
      env:
        AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
        AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}

    - name: dbt Debug
      run: dbt debug

    - name: Run dbt (Slim CI)
      if: github.event_name == 'pull_request'
      run: |
        # Only run modified models and their dependencies
        dbt build \
          --select state:modified+ \
          --defer \
          --state ./prod-manifest.json \
          --target ci

    - name: Run dbt (Full Build)
      if: github.event_name == 'push' && github.ref == 'refs/heads/main'
      run: |
        dbt build --target prod

    - name: Upload artifacts
      if: github.event_name == 'push' && github.ref == 'refs/heads/main'
      run: |
        # Upload new artifacts to production
        aws s3 cp ./target/manifest.json s3://my-bucket/prod-artifacts/manifest.json
```

### GitLab CI Slim CI Configuration

```yaml
# .gitlab-ci.yml
stages:
  - test
  - deploy

variables:
  DBT_PROFILES_DIR: "./"
  PIP_CACHE_DIR: "$CI_PROJECT_DIR/.cache/pip"

cache:
  paths:
    - .cache/pip
    - dbt_packages/

before_script:
  - python --version
  - pip install dbt-postgres>=1.0.0
  - dbt deps

dbt_test:
  stage: test
  image: python:3.9
  script:
    - dbt debug
    # Download production state
    - aws s3 cp s3://my-dbt-artifacts/prod/manifest.json ./prod-manifest.json || echo "No prod manifest found"
    # Run slim CI
    - |
      if [ -f "prod-manifest.json" ]; then
        dbt build --select state:modified+ --defer --state ./prod-manifest.json --target ci
      else
        echo "No production manifest found, running full build"
        dbt build --target ci
      fi
  only:
    - merge_requests
  environment:
    name: ci

dbt_deploy:
  stage: deploy
  image: python:3.9
  script:
    - dbt debug
    - dbt build --target prod
    # Upload new artifacts
    - aws s3 cp ./target/manifest.json s3://my-dbt-artifacts/prod/manifest.json
  only:
    - main
  environment:
    name: production
```

### Azure DevOps Slim CI Configuration

```yaml
# azure-pipelines.yml
trigger:
- main

pr:
- main

pool:
  vmImage: ubuntu-latest

variables:
  - group: dbt-variables
  - name: python.version
    value: '3.9'

stages:
- stage: Test
  condition: eq(variables['Build.Reason'], 'PullRequest')
  jobs:
  - job: dbt_ci_test
    steps:
    - task: UsePythonVersion@0
      inputs:
        versionSpec: '$(python.version)'

    - script: |
        pip install dbt-postgres>=1.0.0
        dbt deps
      displayName: 'Install dbt and dependencies'

    - task: AzureCLI@2
      inputs:
        azureSubscription: 'my-service-connection'
        scriptType: 'bash'
        scriptLocation: 'inlineScript'
        inlineScript: |
          # Download production manifest
          az storage blob download \
            --account-name mystorageaccount \
            --container-name dbt-artifacts \
            --name prod/manifest.json \
            --file ./prod-manifest.json
      displayName: 'Download production artifacts'

    - script: |
        dbt debug
        if [ -f "prod-manifest.json" ]; then
          dbt build --select state:modified+ --defer --state ./prod-manifest.json --target ci
        else
          dbt build --target ci
        fi
      displayName: 'Run dbt Slim CI'

- stage: Deploy
  condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))
  jobs:
  - job: dbt_deploy
    steps:
    - task: UsePythonVersion@0
      inputs:
        versionSpec: '$(python.version)'

    - script: |
        pip install dbt-postgres>=1.0.0
        dbt deps
      displayName: 'Install dbt and dependencies'

    - script: |
        dbt debug
        dbt build --target prod
      displayName: 'Run dbt production build'

    - task: AzureCLI@2
      inputs:
        azureSubscription: 'my-service-connection'
        scriptType: 'bash'
        scriptLocation: 'inlineScript'
        inlineScript: |
          # Upload new artifacts
          az storage blob upload \
            --account-name mystorageaccount \
            --container-name dbt-artifacts \
            --name prod/manifest.json \
            --file ./target/manifest.json \
            --overwrite
      displayName: 'Upload production artifacts'
```

## Slim CI State Selection Methods

### State Selection Operators

```bash
# Modified models only
dbt run --select state:modified --state ./prod-artifacts

# New models only  
dbt run --select state:new --state ./prod-artifacts

# Modified + downstream dependencies
dbt run --select state:modified+ --state ./prod-artifacts

# Modified + upstream dependencies
dbt run --select +state:modified --state ./prod-artifacts

# Modified + upstream and downstream
dbt run --select +state:modified+ --state ./prod-artifacts

# Combine with other selectors
dbt run --select state:modified,tag:daily --state ./prod-artifacts
```

### Defer Configuration

The `--defer` flag tells dbt to use production relations for unchanged models:

```bash
# Use production relations for unselected models
dbt run --select state:modified+ --defer --state ./prod-artifacts
```

### profiles.yml Configuration for Slim CI

```yaml
# profiles.yml
my_project:
  target: dev
  outputs:
    dev:
      type: postgres
      host: localhost
      user: dev_user
      password: dev_password
      port: 5432
      dbname: dev_db
      schema: dev_schema
      threads: 4

    ci:
      type: postgres  
      host: "{{ env_var('CI_DB_HOST') }}"
      user: "{{ env_var('CI_DB_USER') }}"
      password: "{{ env_var('CI_DB_PASSWORD') }}"
      port: 5432
      dbname: ci_db
      schema: "ci_{{ env_var('CI_PIPELINE_ID', 'default') }}"
      threads: 4

    prod:
      type: postgres
      host: "{{ env_var('PROD_DB_HOST') }}"
      user: "{{ env_var('PROD_DB_USER') }}" 
      password: "{{ env_var('PROD_DB_PASSWORD') }}"
      port: 5432
      dbname: prod_db
      schema: prod_schema
      threads: 8
```

## Advanced Slim CI Patterns

### State File Management Script

```bash
#!/bin/bash
# manage_state.sh

STATE_DIR="./state-files"
PROD_MANIFEST_URL="s3://my-bucket/dbt-artifacts/manifest.json"

download_state() {
    echo "Downloading production state..."
    mkdir -p $STATE_DIR
    aws s3 cp $PROD_MANIFEST_URL $STATE_DIR/manifest.json
    
    if [ $? -eq 0 ]; then
        echo "✅ State downloaded successfully"
        return 0
    else
        echo "❌ Failed to download state"
        return 1
    fi
}

upload_state() {
    echo "Uploading new state..."
    aws s3 cp ./target/manifest.json $PROD_MANIFEST_URL
    
    if [ $? -eq 0 ]; then
        echo "✅ State uploaded successfully" 
        return 0
    else
        echo "❌ Failed to upload state"
        return 1
    fi
}

run_slim_ci() {
    if [ -f "$STATE_DIR/manifest.json" ]; then
        echo "🚀 Running Slim CI..."
        dbt build \
            --select state:modified+ \
            --defer \
            --state $STATE_DIR \
            --target ci
    else
        echo "⚠️  No state file found, running full build"
        dbt build --target ci
    fi
}

# Usage
case "$1" in
    download)
        download_state
        ;;
    upload)
        upload_state
        ;;
    run)
        run_slim_ci
        ;;
    full)
        download_state && run_slim_ci
        ;;
    *)
        echo "Usage: $0 {download|upload|run|full}"
        exit 1
        ;;
esac
```

### Best Practices for Slim CI

1. **Always fetch full git history** in CI (`fetch-depth: 0`)
2. **Store state artifacts securely** (S3, GCS, Azure Blob)
3. **Handle missing state gracefully** (fallback to full build)
4. **Use separate schemas for CI** to avoid conflicts
5. **Clean up CI schemas regularly** to manage costs
6. **Test state comparison locally** before deploying
7. **Monitor CI performance** and adjust selection as needed
8. **Version your state artifacts** for rollback capability

### Troubleshooting Slim CI

```bash
# Debug state comparison
dbt ls --select state:modified --state ./prod-artifacts --output name

# Check what would be selected
dbt ls --select state:modified+ --state ./prod-artifacts

# Validate state file
dbt parse --state ./prod-artifacts

# Compare local vs production state  
dbt run --select state:modified --dry-run --state ./prod-artifacts
```
