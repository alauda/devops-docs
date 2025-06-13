# GitHub Actions Documentation Sync Setup

This repository contains GitHub Actions workflows to automatically sync documentation from the `private-docs` repository to the `docs-sync` repository.

## Workflows

### 1. Basic Sync (`sync-docs.yml`)
- **Trigger**: Pushes to main branch that modify `docs/` or `README.md`
- **Function**: Simple sync of documentation files
- **Features**: Basic error handling and commit messaging

### 2. Advanced Sync (`sync-docs-advanced.yml`)
- **Trigger**:
  - Pushes to main branch that modify `docs/` or `README.md`
  - Manual workflow dispatch with force sync option
- **Function**: Comprehensive sync with advanced features
- **Features**:
  - Backup creation
  - Detailed sync information
  - Enhanced commit messages
  - Workflow summaries
  - File counting and metadata

## Setup Instructions

### 1. Create Personal Access Token (PAT)

1. Go to GitHub Settings → Developer settings → Personal access tokens → Tokens (classic)
2. Click "Generate new token (classic)"
3. Set expiration and select these scopes:
   - `repo` (Full control of private repositories)
   - `workflow` (Update GitHub Action workflows)

### 2. Add Repository Secret

1. Go to your `private-docs` repository
2. Navigate to Settings → Secrets and variables → Actions
3. Click "New repository secret"
4. Name: `SYNC_TOKEN`
5. Value: Your PAT from step 1

### 3. Repository Permissions

Ensure the PAT has access to both repositories:
- `danielfbm/private-docs` (source)
- `danielfbm/docs-sync` (target)

## How It Works

1. **Trigger**: When you push changes to the main branch of `private-docs` that affect documentation
2. **Checkout**: Both repositories are checked out to the runner
3. **Sync**: Documentation files are copied from `private-docs` to `docs-sync`
4. **Commit**: Changes are committed with detailed metadata
5. **Push**: Updates are pushed to the `docs-sync` repository

## Files Synced

- `docs/` directory (entire folder structure)
- `README.md` file
- `SYNC_INFO.md` (auto-generated sync metadata)

## Manual Sync

You can manually trigger the advanced workflow:

1. Go to Actions tab in the `private-docs` repository
2. Select "Advanced Sync Documentation to docs-sync"
3. Click "Run workflow"
4. Optionally enable "Force sync" to sync even if no changes are detected

## Troubleshooting

### Common Issues

1. **Permission Denied**: Check that your `SYNC_TOKEN` is valid and has the required scopes
2. **Repository Not Found**: Ensure the PAT has access to both repositories
3. **No Changes Detected**: The workflow skips commits when no changes are found (unless force sync is enabled)

### Monitoring

- Check the Actions tab for workflow runs
- Review the workflow summary for sync details
- Check the `SYNC_INFO.md` file in the `docs-sync` repository for the latest sync information

## Customization

You can modify the workflows to:
- Change which files are synced
- Modify commit message formats
- Add additional processing steps
- Change trigger conditions
- Add notifications (Slack, email, etc.)

## Security Notes

- The PAT should have minimal required permissions
- Consider using environment-specific tokens for production use
- Regularly rotate your access tokens
- Monitor workflow runs for any suspicious activity
