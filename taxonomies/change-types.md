# Change Type Taxonomy

**Version:** 1.0.0
**Status:** Draft

## Overview

Standard vocabulary for classifying code and API surface changes. Enables consistent change documentation, impact analysis, and release notes generation.

## Code Change Types

### Additions

**function_added**
- New function or method created
- Net new functionality
- Example: Added `ValidateEmail()` function

**class_added**
- New class or struct created
- New type introduced
- Example: Added `EmailValidator` class

### Modifications

**function_modified**
- Function body changed
- Logic updated
- Example: Modified `ParseUser()` to handle new field

**signature_changed**
- Function signature changed (parameters, return type)
- Breaking or non-breaking
- Example: `Login(user)` → `Login(user, options)`

**function_renamed**
- Function renamed
- May affect callers
- Example: `GetUser()` → `FetchUser()`

### Removals

**function_removed**
- Function deleted
- Breaking change
- Example: Removed deprecated `OldAuth()` function

**class_removed**
- Class/struct deleted
- Breaking change
- Example: Removed `LegacyParser` class

## Surface Change Types

### API Changes

**api_endpoint_added**
- New REST/GraphQL endpoint
- Example: `POST /v1/users`

**api_endpoint_modified**
- Endpoint behavior changed
- Request/response schema changed
- Example: `/v1/users` now requires `email` field

**api_endpoint_removed**
- Endpoint deleted or deprecated
- Example: `DELETE /v1/legacy/users` removed

**api_breaking_change**
- Non-backwards-compatible API change
- Requires client updates
- Example: Changed response format from XML to JSON

### CLI Changes

**cli_flag_added**
- New command-line flag
- Example: Added `--verbose` flag

**cli_flag_modified**
- Flag behavior changed
- Default value changed
- Example: `--timeout` default changed from 30s to 60s

**cli_flag_removed**
- Flag deleted or deprecated
- Example: Removed `--legacy-mode`

**cli_command_added**
- New CLI command
- Example: Added `app deploy` command

**cli_command_removed**
- Command deleted
- Example: Removed `app migrate` command

### Configuration Changes

**config_field_added**
- New configuration field
- Example: Added `database.pool_size`

**config_field_modified**
- Config field behavior changed
- Type or default changed
- Example: `timeout` now accepts duration string instead of integer

**config_field_removed**
- Config field deprecated/removed
- Example: Removed `legacy_mode`

**config_breaking_change**
- Non-backwards-compatible config change
- Requires config file updates
- Example: Changed YAML structure

### Behavioral Changes

**behavior_modified**
- Observable behavior changed
- Side effects changed
- Example: Errors now logged to stderr instead of stdout

**security_change**
- Security-related change
- Authentication, authorization, encryption
- Example: Added API key requirement

## Change Severity

### Breaking

Changes that break existing integrations:
- `api_breaking_change`
- `signature_changed` (incompatible)
- `function_removed`
- `cli_command_removed`
- `config_breaking_change`

### Non-Breaking

Changes that maintain compatibility:
- `function_added`
- `function_modified` (internal only)
- `cli_flag_added`
- `config_field_added`

### Deprecation

Changes that signal future removal:
- Any change with deprecation notice
- Typically gives 2-3 version grace period

## Usage

### In Linkage Mapping

```yaml
code_to_assets:
  "auth.go:Authenticate":
    assets:
      - path: docs/changelog.md
        segments:
          - id: v2-auth-changes
            heading: "### Authentication Changes"
        relevance: primary
        doc_type: reference
    change_type: signature_changed
    breaking: true
```

### In Release Notes

```markdown
## v2.0.0 - Breaking Changes

### API Changes

- **function_removed**: Removed `OldAuth()` (use `Authenticate()` instead)
- **signature_changed**: `Login()` now requires `LoginOptions` parameter

### CLI Changes

- **cli_flag_removed**: Removed `--legacy-mode`
- **cli_command_added**: Added `app deploy` command
```

### In Change Detection

```typescript
// Classify git diff
const changes = analyzeCommit(diff);

changes.forEach(change => {
  console.log(`${change.type}: ${change.symbol}`);
  if (isBreaking(change.type)) {
    console.warn('BREAKING CHANGE');
  }
});
```

## Validation

- Change type MUST be from this taxonomy
- Breaking changes SHOULD be documented prominently
- Deprecations SHOULD include migration path

## See Also

- [Symbol Taxonomy](./symbols.md) - Symbol kinds
- [Linkage Mapping](../formats/linkage-mapping.md) - Link changes to docs
