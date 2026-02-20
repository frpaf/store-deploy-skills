---
name: setup
description: Guide through mobile app deployment credential setup. Use when user needs to configure iOS or Android deployment credentials.
argument-hint: "[ios|android]"
user-invocable: true
disable-model-invocation: true
allowed-tools: Bash, Read
---

# Deployment Setup Guide

The `store-deploy setup` command is interactive and must be run manually by the user.

## Check Current Setup

```bash
test -f .deploy-config.json && echo "Config exists" || echo "Not configured"
```

If configured, show current settings:

```bash
cat .deploy-config.json | head -30
```

## Run Setup Wizard

Direct the user to run:

```bash
store-deploy setup
```

This interactive wizard will:
1. Detect project type (Flutter, Expo, Native iOS/Android)
2. Check for HashiCorp Vault credentials (auto-fetch credentials if available)
3. Collect iOS credentials (if applicable)
4. Collect Android credentials (if applicable)
5. Create Fastlane configuration files (Fastfile, Appfile, .env)
6. Create Gemfile with pinned Fastlane version
7. Run `bundle install` for dependencies

## Credential Resolution

The setup wizard checks credentials in this order:

1. **HashiCorp Vault** - If AppRole credentials are configured (VAULT_ROLE_ID + VAULT_SECRET_ID), auto-downloads credentials
2. **CLI flags** - Explicit credentials passed via command line
3. **Interactive prompts** - Manual entry as fallback

## iOS Credentials Required

| Credential | Description | Where to Find |
|------------|-------------|---------------|
| Key ID | App Store Connect API Key ID | App Store Connect -> Users and Access -> Integrations -> Keys |
| Issuer ID | App Store Connect Issuer ID | Same page as Key ID (shown at top) |
| Team ID | Apple Developer Team ID | Developer Portal -> Membership |
| Bundle ID | App identifier (com.example.app) | Xcode project or app.json/pubspec.yaml |
| .p8 File | API Key file (AuthKey_XXXXX.p8) | Downloaded when creating API key (one-time download) |

### Creating App Store Connect API Key

1. Go to App Store Connect -> Users and Access -> Integrations -> Keys
2. Click "+" to add a new key
3. Name it (e.g., "Fastlane Deploy")
4. Select "Admin" or "App Manager" role
5. Download the .p8 file immediately (cannot re-download)
6. Note the Key ID and Issuer ID

## Android Credentials Required

| Credential | Description | Where to Find |
|------------|-------------|---------------|
| Package Name | App identifier (com.example.app) | build.gradle or app.json/pubspec.yaml |
| JSON Key | Google Play Service Account key | Google Cloud Console |
| Keystore Path | Release signing keystore (.jks) | Your project or generate new |
| Keystore Password | Keystore password | Set during keystore creation |
| Key Alias | Key alias in keystore | Set during keystore creation |
| Key Password | Key password | Set during keystore creation |

### Creating Google Play Service Account

1. Go to Google Cloud Console -> IAM & Admin -> Service Accounts
2. Create a new service account
3. Grant "Service Account User" role
4. Create JSON key and download
5. In Google Play Console -> Users and permissions
6. Invite the service account email with "Release Manager" role

### Keystore Options

During setup, you can:
- **Generate new keystore** - CLI will use `keytool` to create one
- **Use existing keystore** - Provide path to your .jks file

## Configuration File Structure

After setup, `.deploy-config.json` contains:

```json
{
  "projectType": "expo|flutter|native-android|native-ios",
  "ios": {
    "keyId": "XXXXXXXXXX",
    "issuerId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "teamId": "XXXXXXXXXX",
    "bundleId": "com.example.app",
    "p8FilePath": "path/to/AuthKey.p8"
  },
  "android": {
    "packageName": "com.example.app",
    "jsonKeyPath": "path/to/google-play-key.json",
    "buildType": "aab",
    "keystorePath": "path/to/release.jks",
    "keystorePassword": "...",
    "keyAlias": "...",
    "keyPassword": "..."
  },
  "vault": {
    "roleId": "...",
    "secretId": "...",
    "address": "https://vault-egdw.cto.aksdev.egdev.eu",
    "enginePath": "smd-mobile"
  }
}
```

## Fastlane Files Created

| File | Location | Purpose |
|------|----------|---------|
| Fastfile | ios/fastlane/ or fastlane/ | Lane definitions |
| Appfile | ios/fastlane/ or fastlane/ | App identifiers |
| .env | ios/fastlane/ or fastlane/ | Environment variables |
| Gemfile | ios/ or root | Ruby dependencies |

## Verify Setup

After setup completes:

```bash
# Check config
cat .deploy-config.json

# Check Fastlane files (Expo/Flutter)
ls -la ios/fastlane/ android/fastlane/

# Check Fastlane files (Native)
ls -la fastlane/

# Verify bundle
cd ios && bundle check || bundle install
```

## Teardown

To remove Fastlane setup and start fresh:

```bash
store-deploy teardown
```

Options:
- `--keep-credentials` - Keep .deploy-config.json
- `--keep-gemfile` - Keep Gemfile

## Arguments

$ARGUMENTS

If user mentions a specific platform, provide platform-specific guidance.
