# Version Specification

**Version:** 1.0.0
**Status:** Draft
**Format:** YAML

## Overview

Versions represent specific releases or snapshots of a project. Supports semantic versioning and freeform versioning modes with rich lifecycle management.

## Format

```yaml
version: "1"
kind: project-version

project_version:
  id: <string>                    # Unique version ID (pv_* or semver)
  project: <project-ref>          # Parent project reference
  key: <string>                   # Kebab-case key
  name: <string>                  # Display name

  # Versioning mode
  mode: <enum>                    # semver | freeform

  # Semantic versioning (mode: semver)
  semver:
    major: <int>                  # Required for semver mode
    minor: <int>                  # Required for semver mode
    patch: <int>                  # Required for semver mode
    prerelease: <string>          # Optional: alpha.1, beta.2, rc.1
    build: <string>               # Optional: build metadata

  # Freeform versioning (mode: freeform)
  freeform:
    version: <string>             # Any string (e.g., "Winter 2024", "Sprint 5")

  # Lifecycle
  status: <enum>                  # draft | alpha | beta | rc | supported | deprecated | eol
  type: <enum>                    # major_release | minor_release | patch_release | hotfix

  # Dates
  start_date: <date>              # When version development started
  release_date: <date>            # When version was released
  target_end_date: <date>         # Planned end-of-support
  eol_date: <date>                # End-of-life date

  # Flags
  is_default: <boolean>           # Default version for project
  is_latest: <boolean>            # Latest released version
  is_prerelease: <boolean>        # Is this a prerelease

  # Lineage
  parent_version: <version-ref>   # Previous version
  lineage_type: <enum>            # ancestor | descendant | fork | merge | branch | port

  # Source tracking
  source_commit: <string>         # Git commit SHA
  source_tag: <string>            # Git tag

  # Release metadata
  description: <string>
  release_notes: <string>
  breaking_changes: [<string>]

metadata:
  release_manager: <string>
  ...
```

## Versioning Modes

### Semantic Versioning (semver)

Standard semantic versioning: `MAJOR.MINOR.PATCH[-PRERELEASE][+BUILD]`

```yaml
project_version:
  id: pv_v2_1_0
  mode: semver
  semver:
    major: 2
    minor: 1
    patch: 0
  status: supported
```

**With prerelease:**
```yaml
project_version:
  id: pv_v3_0_0_beta1
  mode: semver
  semver:
    major: 3
    minor: 0
    patch: 0
    prerelease: "beta.1"
  status: beta
  is_prerelease: true
```

**With build metadata:**
```yaml
project_version:
  id: pv_v2_1_5
  mode: semver
  semver:
    major: 2
    minor: 1
    patch: 5
    build: "20240115.1"
  status: supported
```

### Freeform Versioning

For projects without semantic versioning:

```yaml
project_version:
  id: pv_winter_2024
  mode: freeform
  freeform:
    version: "Winter 2024 Release"
  status: supported
  release_date: "2024-01-15"
```

```yaml
project_version:
  id: pv_sprint_42
  mode: freeform
  freeform:
    version: "Sprint 42"
  status: draft
```

## Lifecycle States

See [Lifecycle States](./lifecycle-states.md) for details.

**Development:**
- `draft` - In development
- `alpha` - Early testing
- `beta` - Broader testing
- `rc` - Release candidate

**Released:**
- `supported` - Actively supported
- `deprecated` - End-of-life announced
- `eol` - No longer supported

## Version Lineage

Versions can have ancestry relationships:

```yaml
# v2.0.0 is ancestor of v2.1.0
project_version:
  id: pv_v2_1_0
  parent_version: version:@pv_v2_0_0
  lineage_type: ancestor
```

See [Version Lineage](./version-lineage.md) for relationship types.

## Examples

### Product with Multiple Versions

```yaml
# v1.0.0 (deprecated)
---
version: "1"
kind: project-version

project_version:
  id: pv_v1_0_0
  project: project:@prj_payment_api
  key: v1-0-0
  name: "Version 1.0.0"
  mode: semver
  semver:
    major: 1
    minor: 0
    patch: 0
  status: deprecated
  release_date: "2023-01-15"
  eol_date: "2024-01-15"
  source_tag: v1.0.0

---
# v2.0.0 (supported)
version: "1"
kind: project-version

project_version:
  id: pv_v2_0_0
  project: project:@prj_payment_api
  key: v2-0-0
  name: "Version 2.0.0"
  mode: semver
  semver:
    major: 2
    minor: 0
    patch: 0
  status: supported
  type: major_release
  release_date: "2024-01-15"
  is_default: true
  is_latest: true
  source_tag: v2.0.0
  source_commit: abc123def456

  parent_version: version:@pv_v1_0_0
  lineage_type: ancestor

  release_notes: |
    Major API redesign with improved performance
    and new authentication system.

  breaking_changes:
    - "Removed v1 endpoints"
    - "New authentication required"
    - "Database schema migration needed"

---
# v3.0.0-alpha.1 (prerelease)
version: "1"
kind: project-version

project_version:
  id: pv_v3_0_0_alpha1
  project: project:@prj_payment_api
  key: v3-0-0-alpha-1
  name: "Version 3.0.0 Alpha 1"
  mode: semver
  semver:
    major: 3
    minor: 0
    patch: 0
    prerelease: "alpha.1"
  status: alpha
  type: major_release
  is_prerelease: true

  parent_version: version:@pv_v2_0_0
  lineage_type: ancestor

  source_commit: xyz789abc123
```

## Coordinates

Versions can be referenced:

```yaml
version:@pv_abc123                              # By version ID
version:@v2.0.0                                 # By semver string
version:@pv_v2_0_0#sha256:xyz789                # With checksum
```

## See Also

- [Lifecycle States](./lifecycle-states.md) - Status taxonomy
- [Version Lineage](./version-lineage.md) - Version relationships
- [Project Definition](../projects/project-definition.md) - Parent projects
- [Coordinate System](../coordinates/COORDINATE_SPEC.md) - Reference format
