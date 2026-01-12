# Version Lifecycle States

**Version:** 1.0.0
**Status:** Draft

## Overview

Version lifecycle states track the maturity and support status of project versions.

## States

### Development States

**draft**
- Version in active development
- Not released
- May change significantly
- Internal use only

**alpha**
- Early testing phase
- Feature incomplete
- For testing/feedback only
- May have breaking changes

**beta**
- Feature complete
- Broader testing
- API stabilizing
- May have minor breaking changes

**rc** (Release Candidate)
- Final testing phase
- Feature freeze
- No planned breaking changes
- Production-ready pending validation

### Released States

**supported**
- Generally available (GA)
- Production-ready
- Actively maintained
- Bug fixes and security patches

**deprecated**
- End-of-life announced
- Still functional
- Migration recommended
- Security patches only

**eol** (End of Life)
- No longer supported
- No patches or updates
- Use at own risk
- Migration required

## State Transitions

```
draft → alpha → beta → rc → supported → deprecated → eol
  │                              │
  └──────────────────────────────┘
            (back to draft for major versions)
```

**Allowed transitions:**
```
draft → alpha
alpha → beta
beta → rc
beta → alpha (regression)
rc → supported
rc → beta (regression)
supported → deprecated
deprecated → eol
```

**Forbidden transitions:**
```
eol → supported (can't un-EOL)
supported → alpha (can't go backwards)
```

## State Durations

**Typical timelines** (guidelines only):

| State | Duration | Purpose |
|-------|----------|---------|
| draft | Weeks-months | Initial development |
| alpha | 1-4 weeks | Early testing |
| beta | 2-8 weeks | Broader testing |
| rc | 1-2 weeks | Final validation |
| supported | 6-24 months | Active support |
| deprecated | 3-12 months | Migration period |
| eol | Forever | Historical reference |

## Examples

### Semantic Versioning Lifecycle

```yaml
# v1.0.0 - EOL
version: "1"
project_version:
  id: pv_v1_0_0
  semver: {major: 1, minor: 0, patch: 0}
  status: eol
  release_date: "2023-01-15"
  eol_date: "2024-07-15"

# v2.0.0 - Deprecated
version: "1"
project_version:
  id: pv_v2_0_0
  semver: {major: 2, minor: 0, patch: 0}
  status: deprecated
  release_date: "2024-01-15"
  target_end_date: "2025-06-30"

# v2.1.5 - Supported (current)
version: "1"
project_version:
  id: pv_v2_1_5
  semver: {major: 2, minor: 1, patch: 5}
  status: supported
  release_date: "2024-11-01"
  is_default: true
  is_latest: true

# v3.0.0-beta.1 - Beta (next)
version: "1"
project_version:
  id: pv_v3_0_0_beta1
  semver: {major: 3, minor: 0, patch: 0, prerelease: "beta.1"}
  status: beta
  is_prerelease: true
```

### State-Specific Attributes

**draft:**
```yaml
status: draft
# No release_date required
# target_release_date optional
```

**alpha/beta/rc:**
```yaml
status: beta
is_prerelease: true
target_release_date: "2024-02-01"
```

**supported:**
```yaml
status: supported
release_date: "2024-01-15"
is_default: true              # Optional: default version
is_latest: true               # Optional: latest release
```

**deprecated:**
```yaml
status: deprecated
release_date: "2023-01-15"
target_end_date: "2024-12-31"  # When support ends
eol_date: "2025-01-15"         # Scheduled EOL
```

**eol:**
```yaml
status: eol
release_date: "2022-01-15"
eol_date: "2024-01-15"         # When EOL occurred
```

## Usage in Journeys

Version-specific journeys reference version states:

```yaml
journey:
  version_specific: true
  versions:
    - version:@v2.1.5              # supported
    - version:@v3.0.0-beta.1       # beta
  # Journey applies to both supported and beta versions
```

## State Queries

```typescript
// Find all supported versions
const supported = versions.filter(v => v.status === 'supported');

// Find all prereleases
const prereleases = versions.filter(v => v.is_prerelease);

// Find versions needing migration
const deprecated = versions.filter(v =>
  v.status === 'deprecated' || v.status === 'eol'
);
```

## Version Support Matrix

| Version | Status | Support Level | Updates |
|---------|--------|---------------|---------|
| v1.0.0 | eol | None | No updates |
| v2.0.0 | deprecated | Security only | Critical patches |
| v2.1.5 | supported | Full | Bug fixes + features |
| v3.0.0-beta.1 | beta | Testing | Active development |

## See Also

- [Version Specification](./version-specification.md) - Version format
- [Version Lineage](./version-lineage.md) - Version relationships
