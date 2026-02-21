---
name: deploy
description: Deploy mobile app to TestFlight (iOS) or Play Store (Android). Use when user wants to deploy, release, publish, or upload their app to app stores.
argument-hint: "[ios|android|internal|alpha|beta|production] [--changelog TEXT]"
user-invocable: true
disable-model-invocation: true
context: fork
allowed-tools: Bash, Read
---

# Deploy Mobile App

Deploy the current mobile app to app stores using the `store-deploy` CLI.

## Pre-flight: MANDATORY — Install or Update CLI

**You MUST run this exact command first before any other command.** This ensures the CLI is installed and up-to-date. Do NOT simplify or skip this step.

```bash
npm config set @egdw:registry https://artifactory.eg.dk/artifactory/api/npm/egdw-store-deploy-npm-local/ && npm install -g @egdw/store-deploy
```

This will install the CLI if missing, or update it to the latest version if outdated. npm handles both cases.

## Pre-flight: Check Credentials

```bash
test -f .deploy-config.json && echo "OK" || echo "Missing"
```

If credentials are missing, inform the user to run `store-deploy setup` first.

## Pre-flight: Check Current Version

```bash
store-deploy version get --json
```

## Deployment Commands

### iOS (TestFlight)

```bash
store-deploy ios --changelog "CHANGELOG_TEXT"
```

Flags:
- `--changelog TEXT` or `-c TEXT` - Release notes for TestFlight
- `--skip-sync` - Skip version sync from TestFlight before deploy
- `--skip-signing-setup` - Skip automatic signing setup
- `--clean` or `-C` - Clean build before deploying
- `--update-gems` or `-u` - Update Fastlane before deploying
- `--json` or `-j` - Output results as JSON

### Android (Play Store)

```bash
store-deploy android --changelog "CHANGELOG_TEXT"
```

Or deploy to specific tracks:

```bash
store-deploy internal --changelog "CHANGELOG_TEXT"   # Internal testing
store-deploy alpha --changelog "CHANGELOG_TEXT"      # Alpha track
store-deploy beta --changelog "CHANGELOG_TEXT"       # Beta track
store-deploy production --changelog "CHANGELOG_TEXT" # Production release
```

Flags:
- `--changelog TEXT` or `-c TEXT` - Release notes
- `--skip-sync` - Skip version sync from Play Store
- `--track [internal|alpha|beta|production]` or `-t` - Target track (default: internal)
- `--apk` - Build APK instead of AAB
- `--release-status [draft|completed]` or `-s` - Release status
- `--update-gems` or `-u` - Update Fastlane before deploying
- `--json` or `-j` - Output results as JSON

## Credential Resolution

The CLI resolves credentials in this order:

1. **HashiCorp Vault** (if AppRole configured) - Auto-downloads credentials
2. **Config file** - Reads from `.deploy-config.json`
3. **Interactive prompts** - Falls back to manual entry

## Workflow

1. Run pre-flight checks
2. If user didn't provide changelog, offer to generate one from git history
3. Optionally sync version from store (unless --skip-sync)
4. Execute deployment command
5. Report success/failure with version details

## Error Handling

| Error | Solution |
|-------|----------|
| CLI not found | `npm config set @egdw:registry https://artifactory.eg.dk/artifactory/api/npm/egdw-store-deploy-npm-local/ && npm install -g @egdw/store-deploy` |
| Credentials missing | Run `store-deploy setup` |
| Signing not configured | Run with auto-signing prompt or configure manually |
| Build failure | Check Xcode/Gradle configuration |
| Bundle install failed | Check Ruby installation: `ruby --version` |
| Version already exists | Bump version: `store-deploy version patch` |

## Post-Deploy

After successful deployment:

```bash
# Check iOS processing status
store-deploy status ios --json

# Query Play Store versions
store-deploy store android --json

# Bump version for next release
store-deploy version patch --json
```

## Output Formatting

You have two output modes: **clean** (default) and **verbose**.

### Detecting Mode

- **Default: clean mode**
- Switch to **verbose** if the user passes `--verbose`, `-v`, or says "verbose", "show logs", "debug", "show output"
- **Failure escalation**: If any step fails in clean mode, automatically show that step's full raw output

### Clean Mode (default)

**Suppress raw output.** NEVER show raw Bash output. Run commands, parse JSON results silently, present structured summaries only.

**Always use `--json` flag** on all store-deploy commands.

**Render a step tracker** after each phase completes. Use these markers:
- `✓` completed (with key result)
- `◉` currently running (with `...`)
- `○` pending

```
  ✓ CLI installed (v2.4.1)
  ✓ Credentials verified
  ✓ Version: 1.2.4 (10204)
  ✓ Changelog ready (4 items)
  ◉ Deploying to TestFlight...
  ○ Post-deploy verification
```

Re-render the full tracker after each step so the user sees complete state.

**Show a summary panel on completion:**
```
┌─────────────────────────────────────────┐
│  ✓ Deploy Complete                      │
├─────────────────────────────────────────┤
│  App:        MyApp                      │
│  Platform:   iOS → TestFlight           │
│  Version:    1.2.4 (10204)             │
│  Changelog:  4 items                    │
│  Status:     Processing                 │
└─────────────────────────────────────────┘
```

**On failure**, show error panel then full raw output:
```
┌─────────────────────────────────────────┐
│  ✗ Deploy Failed                        │
├─────────────────────────────────────────┤
│  Phase:      Signing                    │
│  Error:      Credentials expired        │
│  Suggestion: Run store-deploy setup     │
└─────────────────────────────────────────┘
```

### Verbose Mode

Show everything — each command with `$` prefix, full raw output in code blocks, step tracker with nested output:
```
  ✓ CLI installed (v2.4.1)
    $ npm config set @egdw:registry ... && npm install -g @egdw/store-deploy
    added 142 packages in 11.8s

  ✓ Credentials verified
    $ test -f .deploy-config.json && echo "OK"
    OK

  ◉ Deploying to TestFlight...
    $ store-deploy ios --changelog "- Feature 1"
```

Still show the summary panel at the end.

## Arguments

$ARGUMENTS

If no arguments provided, ask user which platform to deploy (iOS or Android).
