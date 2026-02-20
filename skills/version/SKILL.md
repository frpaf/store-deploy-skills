---
name: version
description: View and manage mobile app version. Use when user wants to check, set, or bump the app version number.
argument-hint: "[get|set|patch|minor|major] [VERSION]"
user-invocable: true
disable-model-invocation: false
allowed-tools: Bash, Read
---

# Version Management

Manage app version numbers using the `store-deploy` CLI.

## Pre-flight: Ensure CLI Installed

```bash
command -v store-deploy >/dev/null 2>&1 || npm install -g @egdw/store-deploy --registry=https://artifactory.eg.dk/artifactory/api/npm/egdw-store-deploy-npm-local/
```

## Commands

### Get Current Version

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

### Set Specific Version

```bash
store-deploy version set 2.0.0 --json
```

Sets both the semantic version (2.0.0) and calculates the version code (20000).

### Bump Version

```bash
store-deploy version patch --json   # 1.2.3 -> 1.2.4 (code: 10203 -> 10204)
store-deploy version minor --json   # 1.2.3 -> 1.3.0 (code: 10203 -> 10300)
store-deploy version major --json   # 1.2.3 -> 2.0.0 (code: 10203 -> 20000)
```

## Version Code Formula

The version code is calculated as:
```
code = major * 10000 + minor * 100 + patch
```

Examples:
- 1.0.0 -> 10000
- 1.2.3 -> 10203
- 2.5.10 -> 20510
- 1.15.7 -> 11507

## Project Type Support

| Project Type | Version Location | Format | Notes |
|--------------|------------------|--------|-------|
| Expo | app.json | version + ios.buildNumber + android.versionCode | All three updated together |
| Flutter | pubspec.yaml | X.Y.Z+CODE | Format: `version: 1.2.3+10203` |
| Native Android | build.gradle | versionCode + versionName | Supports both Groovy and Kotlin DSL |
| Native iOS | Not supported | N/A | Version managed in Xcode |

## Expo Version Structure

```json
{
  "expo": {
    "version": "1.2.3",
    "ios": {
      "buildNumber": "10203"
    },
    "android": {
      "versionCode": 10203
    }
  }
}
```

## Flutter Version Structure

```yaml
version: 1.2.3+10203
```

## Native Android Version Structure

```groovy
// build.gradle
android {
    defaultConfig {
        versionCode 10203
        versionName "1.2.3"
    }
}
```

## Sync from Store

If store version is higher than local:

```bash
# Sync from TestFlight (iOS)
store-deploy sync ios --json

# Sync from Play Store (Android)
store-deploy sync android --json

# Sync from highest of both stores
store-deploy sync --json
```

Sync sets local version to store version + 1 patch.

## Common Workflows

### Before Deploy
```bash
# Check current version
store-deploy version get --json

# Check store versions
store-deploy store ios --json
store-deploy store android --json

# Bump if needed
store-deploy version patch --json
```

### After Deploy
```bash
# Bump for next release
store-deploy version patch --json
```

### Release Cycle
```bash
# Feature release
store-deploy version minor --json

# Major release
store-deploy version major --json

# Bug fix
store-deploy version patch --json
```

## Arguments

$ARGUMENTS

If no arguments provided, default to showing current version (`get`).
