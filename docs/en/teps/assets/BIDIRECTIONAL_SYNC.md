# Bidirectional Documentation Sync System

This setup implements a bidirectional sync system between `private-docs` (private repository) and `docs-sync` (public repository) to handle both internal changes and external contributions.

## How It Works

### Forward Sync (private-docs → docs-sync)
1. **Trigger**: When changes are pushed to `main` branch in `private-docs` affecting documentation
2. **Process**: Copies `docs/` and `README.md` from private to public repository
3. **Loop Prevention**: Skips sync if commit message contains `[reverse-sync]` marker

### Reverse Sync (docs-sync → private-docs)
1. **Trigger**: When external contributor PRs are merged in `docs-sync`
2. **Process**: Creates a PR in `private-docs` with the external changes
3. **Loop Prevention**: Skips if PR is from sync bot or contains sync keywords

## Sync Flow Diagram

```
┌─────────────────┐    Forward Sync     ┌─────────────────┐
│   private-docs  │ ──────────────────► │    docs-sync    │
│   (internal)    │                     │   (public)      │
└─────────────────┘                     └─────────────────┘
         ▲                                       │
         │              Reverse Sync             │
         │          (via Pull Request)           │
         └───────────────────────────────────────┘
```

## Repository Setup

### private-docs Repository Files:
- `.github/workflows/sync-docs-advanced.yml` - Forward sync with loop prevention
- `.github/SYNC_SETUP.md` - Setup instructions
- `scripts/local-sync.sh` - Local testing script

### docs-sync Repository Files:
- `.github/workflows/reverse-sync.yml` - Reverse sync workflow

## Security & Permissions

Both repositories need a shared secret:
- **Secret Name**: `SYNC_TOKEN`
- **Required Scopes**: `repo`, `workflow`
- **Access**: Both `danielfbm/private-docs` and `danielfbm/docs-sync`

## Workflow Details

### Forward Sync Features:
- ✅ Automatic loop prevention via commit message detection
- ✅ Comprehensive logging and summaries
- ✅ Backup creation before sync
- ✅ Manual trigger capability
- ✅ File change detection

### Reverse Sync Features:
- ✅ External contributor detection
- ✅ PR-based integration (requires manual review)
- ✅ Detailed commit messages with source attribution
- ✅ Patch-based or file-based sync methods
- ✅ Automatic branch creation and PR generation

## Usage Scenarios

### Scenario 1: Internal Documentation Update
1. Developer updates `docs/` in `private-docs`
2. Commits and pushes to `main`
3. Forward sync automatically updates `docs-sync`
4. No reverse sync triggered (internal change)

### Scenario 2: External Contribution
1. External contributor creates PR in `docs-sync`
2. Maintainer reviews and merges PR
3. Reverse sync creates PR in `private-docs`
4. Internal team reviews and merges PR
5. Forward sync does NOT trigger (loop prevention active)

### Scenario 3: Simultaneous Changes
1. If both repositories have changes, manual resolution may be needed
2. The system prevents infinite loops but conflicts require human intervention

## Loop Prevention Mechanisms

### Forward Sync Prevention:
```yaml
# Checks commit message for reverse sync marker
if echo "$commit_msg" | grep -q "\[reverse-sync\]"; then
  skip_sync=true
fi
```

### Reverse Sync Prevention:
```yaml
# Checks PR author and title for sync bot indicators
if [[ "$pr_author" == "github-actions[bot]" ]] ||
   [[ "$pr_title" == *"Sync documentation"* ]]; then
  skip_sync=true
fi
```

## Monitoring & Troubleshooting

### Check Sync Status:
1. **Forward Sync**: Check Actions tab in `private-docs`
2. **Reverse Sync**: Check Actions tab in `docs-sync`
3. **Sync Info**: Check `SYNC_INFO.md` in `docs-sync`

### Common Issues:

#### Infinite Loop Prevention Failed:
- Check commit messages contain proper markers
- Verify bot detection logic is working
- Review workflow conditions

#### Sync Not Triggering:
- Verify `SYNC_TOKEN` has proper permissions
- Check file path filters in workflow triggers
- Ensure repositories are accessible

#### Merge Conflicts:
- Manual resolution required in PRs
- Consider rebasing or manual sync
- Review conflicting changes

#### External PR Not Creating Reverse Sync:
- Check workflow logs in `docs-sync`
- Verify contributor is not detected as bot
- Ensure documentation files were changed

## Customization Options

### Change Synced Paths:
```yaml
# In workflow triggers
paths:
  - 'docs/**'
  - 'README.md'
  - 'additional-path/**'  # Add new paths
```

### Modify Loop Prevention:
```yaml
# Add custom markers or conditions
if echo "$commit_msg" | grep -q "\[custom-marker\]"; then
  skip_sync=true
fi
```

### Add Notifications:
```yaml
# Add Slack/email notifications after sync
- name: Notify team
  run: |
    curl -X POST webhook-url -d "Sync completed"
```

## Testing

### Local Testing:
```bash
# Test forward sync locally
cd private-docs
./scripts/local-sync.sh
```

### End-to-End Testing:
1. Create test changes in `private-docs`
2. Verify forward sync works
3. Create external PR in `docs-sync`
4. Verify reverse sync creates PR
5. Merge reverse sync PR
6. Verify no infinite loop occurs

## Best Practices

1. **Review All Reverse Sync PRs**: External contributions should be reviewed before merging
2. **Monitor Sync Status**: Regularly check workflow runs for failures
3. **Update Sync Markers**: Keep loop prevention logic up to date
4. **Document Changes**: Use clear commit messages to track sync origins
5. **Test Before Production**: Use the local sync script to test changes

## Security Considerations

- Limit `SYNC_TOKEN` to minimum required permissions
- Regularly rotate access tokens
- Monitor for unauthorized sync attempts
- Review external contributions carefully
- Consider approval workflows for sensitive changes

---

*This bidirectional sync system ensures that both internal documentation updates and external contributions are properly synchronized while preventing infinite loops and maintaining security.*
