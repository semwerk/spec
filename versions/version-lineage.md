# Version Lineage

**Version:** 1.0.0
**Status:** Draft

## Overview

Version lineage tracks ancestry and relationships between versions, enabling version history graphs and migration paths.

## Lineage Types

### ancestor

**Description:** Direct predecessor relationship

**Semantics:** `A ancestor B` means "A evolved from B"

**Direction:** Child → Parent (newer → older)

**Example:**
```yaml
# v2.1.0 evolved from v2.0.0
project_version:
  id: pv_v2_1_0
  semver: {major: 2, minor: 1, patch: 0}
  parent_version: version:@pv_v2_0_0
  lineage_type: ancestor
```

**Chain:**
```
v1.0.0 ← v2.0.0 ← v2.1.0 ← v2.2.0
```

### descendant

**Description:** Direct successor relationship (inverse of ancestor)

**Semantics:** `A descendant B` means "B evolved into A"

**Note:** Usually implicit from ancestor relationships

### fork

**Description:** Divergent version branch

**Semantics:** `A fork B` means "A diverged from B"

**Use Case:** Long-lived branches, LTS versions, experimental variants

**Example:**
```yaml
# v2-lts forked from v2.0.0
project_version:
  id: pv_v2_lts
  freeform: {version: "v2-lts"}
  parent_version: version:@pv_v2_0_0
  lineage_type: fork
  description: "Long-term support branch"
```

**Graph:**
```
v2.0.0 ← v2.1.0 ← v2.2.0
  ↓
v2-lts ← v2-lts-patch1
```

### merge

**Description:** Convergent version merge

**Semantics:** `A merge B` means "A incorporated B"

**Use Case:** Merging experimental features back to main line

**Example:**
```yaml
# v3.0.0 merged experimental branch
project_version:
  id: pv_v3_0_0
  semver: {major: 3, minor: 0, patch: 0}
  parent_version: version:@pv_experimental_v3
  lineage_type: merge
```

### branch

**Description:** Short-lived variant

**Semantics:** `A branch B` means "A is a short-lived variant of B"

**Use Case:** Feature branches, hotfix branches

**Example:**
```yaml
# Hotfix branch from v2.0.0
project_version:
  id: pv_v2_0_1_hotfix
  semver: {major: 2, minor: 0, patch: 1}
  parent_version: version:@pv_v2_0_0
  lineage_type: branch
  type: hotfix
```

### port

**Description:** Cross-platform or cross-project port

**Semantics:** `A port B` means "A is a port of B to different context"

**Use Case:** Python version of Go library, mobile port of web app

**Example:**
```yaml
# Python SDK ported from Go SDK
project_version:
  id: pv_python_v1_0_0
  project: project:@prj_payment_sdk_python
  semver: {major: 1, minor: 0, patch: 0}
  parent_version: version:@pv_go_v2_0_0  # From different project
  lineage_type: port
  description: "Python port of Go SDK v2.0.0"
```

## Version Graph Examples

### Linear Progression

```
v1.0.0 (eol) ← v2.0.0 (deprecated) ← v2.1.0 (supported) ← v3.0.0-beta (beta)
  ancestor      ancestor               ancestor
```

### Fork with LTS

```
v2.0.0 ← v2.1.0 ← v2.2.0 ← v3.0.0
  ↓
  fork
  ↓
v2-lts ← v2-lts-patch1 ← v2-lts-patch2
```

### Merge from Experimental

```
v2.0.0 ← v2.1.0
           ↓
         merge ← experimental-v3 (branch)
           ↓
         v3.0.0
```

### Cross-Project Port

```
Project A (Go):
  v1.0.0 ← v2.0.0 ← v2.1.0

Project B (Python):
  v1.0.0 (port from Project A v2.0.0)
```

## YAML Representation

```yaml
# Version with lineage
version: "1"
kind: project-version

project_version:
  id: pv_v2_1_0
  project: project:@prj_payment_api
  key: v2-1-0
  name: "Version 2.1.0"

  mode: semver
  semver:
    major: 2
    minor: 1
    patch: 0

  status: supported
  release_date: "2024-11-01"
  is_latest: true

  # Lineage
  parent_version: version:@pv_v2_0_0
  lineage_type: ancestor

  # Git tracking
  source_commit: abc123def456
  source_tag: v2.1.0
```

## Graph Visualization

```yaml
# Mermaid diagram
graph LR
  v1[v1.0.0<br/>EOL] -->|ancestor| v2[v2.0.0<br/>Deprecated]
  v2 -->|fork| v2lts[v2-LTS<br/>Supported]
  v2 -->|ancestor| v21[v2.1.0<br/>Supported]
  v21 -->|ancestor| v3[v3.0.0-beta<br/>Beta]
```

## Use Cases

### 1. Migration Paths

```typescript
// Find upgrade path from v1 to v3
const path = findUpgradePath(version:@v1.0.0, version:@v3.0.0);
// Returns: [v1.0.0, v2.0.0, v2.1.0, v3.0.0]
```

### 2. Breaking Change Detection

```python
# Check if upgrade has breaking changes
def has_breaking_changes(from_version, to_version):
    path = get_lineage_path(from_version, to_version)
    for version in path:
        if version.type == 'major_release':
            return True
    return False
```

### 3. LTS Branch Management

```yaml
# Track LTS branches
lts_versions = versions.filter(v => v.lineage_type === 'fork' && v.key.includes('lts'));
```

## Validation

- Lineage type MUST be from taxonomy
- Parent version MUST exist
- Circular lineage not allowed (v2 → v1 → v2)
- Fork/branch SHOULD have different project or key

## See Also

- [Version Specification](./version-specification.md) - Version format
- [Lifecycle States](./lifecycle-states.md) - Version states
- [Coordinate System](../coordinates/COORDINATE_SPEC.md) - Reference format
