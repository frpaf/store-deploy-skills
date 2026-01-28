---
name: status
description: Check app store versions and deployment status. Use when user wants to see what version is live on TestFlight or Play Store.
argument-hint: "[ios|android|both]"
user-invocable: true
disable-model-invocation: false
allowed-tools: Bash, Read
---

# Deployment Status

Query current versions from app stores and compare with local version.

## Commands

### Local Version

```bash
store-deploy version get --json
```

Returns:
```json
{
  "version": "1.2.3",
  "code": 10203,
  "projectType": "expo"
}
```

### iOS (TestFlight)

```bash
store-deploy store ios --json
```

Returns the latest TestFlight build version and code.

### iOS Detailed Status

```bash
store-deploy status ios --json
```

Returns detailed status including:
- TestFlight version and processing status
- App Store version (if published)
- Review state

### Android (Play Store)

```bash
store-deploy store android --json
```

Returns versions across all tracks:
- Internal testing
- Alpha
- Beta
- Production

### All Store Versions

```bash
store-deploy store --json
```

Queries both iOS and Android versions.

## Workflow

1. Query local version
2. Query store version(s) based on platform argument
3. Compare versions
4. Recommend actions:
   - If local > store: "Ready to deploy"
   - If local = store: "Already deployed, bump version first"
   - If local < store: "Local version behind, sync recommended"

## Version Comparison

```bash
# Get all info
store-deploy version get --json
store-deploy store ios --json
store-deploy store android --json
```

Compare the `code` values:
- Local code > Store code = Ready to deploy
- Local code = Store code = Need to bump version
- Local code < Store code = Need to sync from store

## Sync Commands

To sync local version from store (sets local to store version + 1 patch):

```bash
store-deploy sync ios --json      # Sync from TestFlight
store-deploy sync android --json  # Sync from highest Play Store track
store-deploy sync --json          # Sync from highest of both stores
```

## Version Code Formula

```
code = major * 10000 + minor * 100 + patch
```

Examples:
- 1.2.3 -> 10203
- 2.0.0 -> 20000
- 1.15.7 -> 11507

## Quick Status Check Script

```bash
echo "=== Local Version ==="
store-deploy version get --json

echo "=== iOS (TestFlight) ==="
store-deploy store ios --json 2>/dev/null || echo "iOS not configured"

echo "=== Android (Play Store) ==="
store-deploy store android --json 2>/dev/null || echo "Android not configured"
```

## Post-Deploy Status

After deploying to iOS, check processing status:

```bash
# Wait a few minutes for processing
store-deploy status ios --json
```

TestFlight processing typically takes 5-30 minutes.

## Arguments

$ARGUMENTS

If no arguments provided, query both platforms and compare with local version.
