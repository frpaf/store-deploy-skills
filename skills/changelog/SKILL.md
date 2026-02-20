---
name: changelog
description: Generate changelog from git history. Use when user needs release notes for deployment or wants to see recent changes.
argument-hint: "[--since TAG] [--format bullet|markdown|conventional]"
user-invocable: true
disable-model-invocation: false
allowed-tools: Bash, Read
---

# Changelog Generation

Generate release notes from git commit history for app store deployments.

## Pre-flight: Ensure CLI Installed

```bash
REGISTRY="https://artifactory.eg.dk/artifactory/api/npm/egdw-store-deploy-npm-local/"
if ! command -v store-deploy >/dev/null 2>&1; then
  npm install -g @egdw/store-deploy --registry="$REGISTRY"
else
  CURRENT=$(npm list -g @egdw/store-deploy --json 2>/dev/null | node -e "try{const d=JSON.parse(require('fs').readFileSync('/dev/stdin','utf8'));console.log(d.dependencies['@egdw/store-deploy'].version)}catch{console.log('unknown')}")
  LATEST=$(npm view @egdw/store-deploy version --registry="$REGISTRY" 2>/dev/null || echo "unknown")
  if [ "$CURRENT" != "$LATEST" ] && [ "$LATEST" != "unknown" ]; then
    echo "Updating @egdw/store-deploy from $CURRENT to $LATEST"
    npm install -g @egdw/store-deploy --registry="$REGISTRY"
  fi
fi
```

## Find Last Tag

```bash
git describe --tags --abbrev=0 2>/dev/null || echo "No tags found"
```

## Get Commits Since Tag

```bash
# Since specific tag
git log v1.2.0..HEAD --oneline --no-merges

# Since last tag
git log $(git describe --tags --abbrev=0)..HEAD --oneline --no-merges

# If no tags, last 10 commits
git log -10 --oneline --no-merges
```

## Commit Details

For more context:

```bash
git log $(git describe --tags --abbrev=0)..HEAD --pretty=format:"%h %s" --no-merges
```

## Output Formats

### Bullet Format (default)
```
- Fix login bug
- Add dark mode support
- Update dependencies
```

### Markdown Format
```markdown
## What's New

### Features
- Add dark mode support

### Bug Fixes
- Fix login bug

### Maintenance
- Update dependencies
```

### Conventional Format
```
feat: Add dark mode support
fix: Fix login bug
chore: Update dependencies
```

## Parsing Conventional Commits

If commits follow conventional format (feat:, fix:, etc.):

```bash
git log $(git describe --tags --abbrev=0)..HEAD --pretty=format:"%s" --no-merges | grep -E "^(feat|fix|docs|style|refactor|test|chore):"
```

## Arguments

$ARGUMENTS

If no arguments, generate bullet-format changelog since last tag.

## Usage with Deploy

After generating changelog, user can deploy with:

```bash
# iOS (TestFlight)
store-deploy ios --changelog "GENERATED_CHANGELOG"

# Android (Play Store tracks)
store-deploy android --changelog "GENERATED_CHANGELOG"
store-deploy internal --changelog "GENERATED_CHANGELOG"
store-deploy alpha --changelog "GENERATED_CHANGELOG"
store-deploy beta --changelog "GENERATED_CHANGELOG"
store-deploy production --changelog "GENERATED_CHANGELOG"
```

## Changelog Best Practices

1. **User-facing language** - Write for app store users, not developers
2. **Highlight benefits** - Focus on what users gain
3. **Keep it concise** - App stores have character limits
4. **Group by category** - Features, fixes, improvements
5. **No internal jargon** - Avoid technical implementation details
