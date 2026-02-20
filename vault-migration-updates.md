# Update Skills for HashiCorp Vault Migration

The `store-deploy` CLI migrated from EG Vault (Secret Server Cloud, static Bearer tokens) to HashiCorp Vault (AppRole auth, KV v2 engine at `smd-mobile/`). Update the skill files to reflect this.

## Changes Required

### 1. `skills/setup/SKILL.md`

**Line 36** — Change "Check for EG Vault token" → "Check for HashiCorp Vault credentials (AppRole)"

**Line 47** — Update credential resolution:
```
- Before: 1. **EG Vault** - If vault token is configured, auto-downloads credentials
- After:  1. **HashiCorp Vault** - If AppRole credentials configured (VAULT_ROLE_ID + VAULT_SECRET_ID), auto-downloads credentials
```

**Lines 119-122** — Replace vault config in `.deploy-config.json` example:
```json
// OLD:
"vault": {
  "token": "...",
  "baseUrl": "..."
}

// NEW:
"vault": {
  "roleId": "...",
  "secretId": "...",
  "address": "https://vault-egdw.cto.aksdev.egdev.eu",
  "enginePath": "smd-mobile"
}
```

### 2. `skills/mobile-deployment/SKILL.md`

**Lines 66-71** — Replace Vault Integration commands table:
```
// OLD:
| `store-deploy vault verify` | Verify EG Vault token |
| `store-deploy vault secrets` | Search/display vault secrets |
| `store-deploy vault folders` | List vault folders |

// NEW:
| `store-deploy vault verify` | Verify HashiCorp Vault AppRole credentials |
| `store-deploy vault secrets --path PATH` | Read a secret at a KV v2 path |
| `store-deploy vault list` | List keys in KV v2 engine |
```

**Line 135** — Update credential resolution:
```
- Before: 1. **EG Vault** (if token configured) - Auto-downloads .p8, JSON keys, keystores
- After:  1. **HashiCorp Vault** (if AppRole configured) - Auto-downloads .p8, JSON keys, keystores
```

### 3. `skills/deploy/SKILL.md`

**Line 69** — Update credential resolution:
```
- Before: 1. **EG Vault** (if configured) - Auto-downloads credentials
- After:  1. **HashiCorp Vault** (if AppRole configured) - Auto-downloads credentials
```

## Verification

After making changes, grep for stale references:
```bash
grep -r "EG Vault\|EG_VAULT_TOKEN\|vault folders\|\"token\":" skills/
```

Should return zero matches.
