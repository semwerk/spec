# Unified Coordinate Specification

**Version:** 1.0.0
**Status:** Draft

<!--semcontext:segment start key="overview" type="overview" audience="developer,engineering"-->
## Overview

A unified, ID-based reference system for all resources: content, code, assets, projects, versions, journeys, and concepts.

The coordinate system enables tools to reference any resource with:
- **Type safety** - Explicit type prefix eliminates ambiguity
- **Stability** - ID-based references survive refactoring
- **Versioning** - Track specific versions of resources
- **Integrity** - Optional checksums verify content
- **Portability** - Works across environments without absolute paths

**Coordinates reference:** See also [content:@doc-linkage-mapping/overview](#) and [content:@doc-segment-markers/syntax](#)
<!--semcontext:segment end-->

<!--semcontext:segment start key="format-specification" type="reference" audience="developer"-->
## Format

### Basic Format
```
<type>:@<id>/<segment-id>
```

### Extended Formats

**With version:**
```
<type>:@<id>@<version>/<segment-id>
```

**With subheadings (content only):**
```
content:@<id>/<segment>.<subheading>.<subsubheading>
```

**With checksum (at end, # separator):**
```
<type>:@<id>/<segment-id>#<checksum>
<type>:@<id>@<version>/<segment-id>#<checksum>
```

**All features:**
```
content:@<id>@<version>/<segment>.<subheading>#<checksum>
```

### Components

| Component | Required | Separator | Description | Example |
|-----------|----------|-----------|-------------|---------|
| Type | Yes | `:` | Resource type | `content`, `code`, `asset` |
| ID | Yes | `@` | Unique identifier | `doc-abc123`, `cfn_xyz789` |
| Version | No | `@` | Version/revision | `v2.0.0`, `main`, `2024-01-15` |
| Segment | No | `/` | Element within resource | `overview`, `Authenticate` |
| Subheading | No | `.` | Hierarchical navigation (content only) | `.prerequisites.installation` |
| Checksum | No | `#` | Content hash (always last) | `sha256:abc123...` |

**Critical:** All coordinates MUST use IDs (not free-form paths)

**Separator Convention:**
- `:` - Type-to-ID boundary
- `@` - Prefixes IDs and versions
- `/` - Hierarchical path to segment
- `.` - Subheading hierarchy (content only)
- `#` - Checksum (like git commit hash, always last)

### Resource Types

| Type | ID Format | Description | Example |
|------|-----------|-------------|---------|
| `content` | `doc-*` or `doc_*` | User-facing content | `content:@doc-quickstart/getting-started` |
| `code` | `cfn_*` | Code function | `code:@cfn_abc123/Authenticate` |
| `asset` | `ast_*`, `gdoc-*`, `notion-*` | Business doc | `asset:@gdoc-xyz789/introduction` |
| `project` | `prj_*` | Project | `project:@prj_payment_platform` |
| `version` | `pv_*` or semver | Version | `version:@v2.0.0` |
| `journey` | `jrn_*` | User journey | `journey:@jrn_onboarding/node-setup` |
| `concept` | `cpt_*` | Semantic concept | `concept:@cpt_authentication` |
| `persona` | `per_*` | User persona | `persona:@per_backend_dev` |
| `task` | `stsk_*` | SpecBook task | `task:@stsk_onboard_dev` |
| `problem` | `sprb_*` | SpecBook problem | `problem:@sprb_auth_complexity` |
| `solution` | `ssol_*` | SpecBook solution | `solution:@ssol_quickstart` |
| `term` | `term_*` | Terminology | `term:@term_jwt` |

**Full list:** content, code, asset, project, version, journey, concept, persona, task, problem, solution, outcome, validation, next_step, impact, feature, term (17 types)
<!--semcontext:segment end-->

<!--semcontext:segment start key="examples" type="example" audience="developer"-->
## Examples

### Content (User-Facing Documentation, Marketing)

**Basic:**
```yaml
content:@doc-quickstart/getting-started
content:@doc-api-reference/authentication
content:@doc-marketing/features
```

**With version:**
```yaml
content:@doc-api@v2.0.0/authentication
content:@doc-quickstart@main/getting-started
content:@doc-marketing@2024-Q1/campaign
```

**With subheadings:**
```yaml
content:@doc-quickstart/getting-started.prerequisites
content:@doc-quickstart/getting-started.prerequisites.installation
content:@doc-api/authentication.oauth.setup.credentials
content:@doc-marketing/features.pricing.enterprise.sla
```

**With checksum:**
```yaml
content:@doc-api/authentication#sha256:abc123def456
content:@doc-guide@v2.0.0/setup#sha256:xyz789abc123
content:@doc-api@v2.0.0/auth.oauth.setup#sha256:abc123
```

### Code (Functions, Methods, Classes)

**Basic:**
```yaml
code:@cfn_abc123/Authenticate
code:@cfn_xyz789/AuthService.Login
code:@cfn_def456/ProcessPayment
```

**With version:**
```yaml
code:@cfn_abc123@v2.0.0/Authenticate
code:@cfn_xyz789@main/Login
code:@cfn_def456@abc123def/ProcessPayment
```

**With checksum:**
```yaml
code:@cfn_abc123/Authenticate#sha256:xyz789
code:@cfn_xyz789@v2.0.0/Login#sha256:abc123
```

### Assets (Google Docs, Notion, Confluence, etc.)

**Basic:**
```yaml
asset:@gdoc-abc123/introduction
asset:@notion-xyz789/requirements
asset:@confluence-123456/architecture
asset:@figma-abc123/frame-onboarding
```

**With version:**
```yaml
asset:@gdoc-abc123@2024-01-15/introduction
asset:@notion-xyz789@revision-42/requirements
asset:@confluence-123@v3/architecture
```

**With checksum:**
```yaml
asset:@gdoc-abc123/introduction#sha256:def456
asset:@ast_payment_001@v2.0.0/overview#sha256:abc123
```

### Complete Example

```yaml
# All features combined
content:@doc-api@v2.0.0/authentication.oauth.setup.credentials#sha256:abc123def456
  │      │   │       │  │              │     │    │            │    │
  │      │   │       │  │              │     │    │            │    └─ checksum
  │      │   │       │  │              │     │    │            └─ subsubsubheading
  │      │   │       │  │              │     │    └─ subsubheading
  │      │   │       │  │              │     └─ subheading
  │      │   │       │  │              └─ segment
  │      │   │       │  └─ version
  │      │   │       └─ content ID
  │      │   └─ @ prefix
  │      └─ type

Type: content
ID: doc-api
Version: v2.0.0
Segment: authentication
Subheadings: [oauth, setup, credentials]
Checksum: sha256:abc123def456
```

**Related specifications:** [content:@doc-journey-graph/nodes](#), [content:@doc-concept-definition/relationships](#)
<!--semcontext:segment end-->

<!--semcontext:segment start key="validation-rules" type="reference" audience="developer"-->
## Validation Rules

**Valid coordinates:**
```yaml
content:@doc-abc123/overview                            ✓
content:@doc-abc123@v2.0.0/overview                     ✓
content:@doc-abc123/overview.prerequisites              ✓
content:@doc-abc123@v2.0.0/overview#sha256:xyz789       ✓
code:@cfn_xyz789/Authenticate                           ✓
code:@cfn_xyz789@main/Authenticate#sha256:abc123        ✓
asset:@gdoc-abc123/intro#sha256:def456                  ✓
```

**Invalid coordinates:**
```yaml
@doc-abc123/overview                                    ✗ Missing type
content:doc-abc123/overview                             ✗ Missing @ before ID
content:@/path/to/file.md/section                       ✗ Path-based
content:@doc-abc123#overview                            ✗ Using # for segment
https://docs.google.com/document/d/abc123               ✗ URL
```

### Regex Pattern

```regex
(content|code|asset|project|version|journey|concept):@[a-z0-9_-]+(?:@[a-z0-9.-]+)?(?:\/[a-z0-9._-]+)?(?:#sha256:[a-f0-9]+)?
```

**Breakdown:**
- `(content|code|...)` - Type (required)
- `:@` - Separator + ID prefix (required)
- `[a-z0-9_-]+` - ID (required)
- `(?:@[a-z0-9.-]+)?` - Optional version
- `(?:\/[a-z0-9._-]+)?` - Optional segment with subheadings
- `(?:#sha256:[a-f0-9]+)?` - Optional checksum (always last)
<!--semcontext:segment end-->

<!--semcontext:segment start key="use-cases" type="howto" audience="developer,product"-->
## Use Cases

### 1. Version-Specific References

Reference specific versions to prevent documentation drift:

```yaml
# Reference v2.0.0 docs specifically
content:@doc-api@v2.0.0/authentication.oauth

# Ensure code hasn't changed
code:@cfn_auth@v2.0.0/Login#sha256:xyz789
```

### 2. Content Integrity

Verify referenced content for compliance and security:

```yaml
# Regulatory compliance - verify doc hasn't changed
content:@doc-compliance@v1.0/gdpr#sha256:abc123

# Security audit - verify code matches expected
code:@cfn_security@v2.0.0/ValidateToken#sha256:def456
```

### 3. Deep Navigation

Navigate to specific subsections in documentation:

```yaml
# Navigate to specific subsection
content:@doc-quickstart/setup.prerequisites.docker.installation

# Reference nested API docs
content:@doc-api/endpoints.payments.create-charge.request-body
```

### 4. Cross-Resource Linking

Link between code, docs, and business assets:

```yaml
# Journey node referencing multiple resources
journey:@jrn_onboarding@v2.0.0/setup-auth
  references:
    - code:@cfn_oauth@v2.0.0/AuthorizeUser
    - content:@doc-auth@v2.0.0/oauth.setup
    - asset:@gdoc-product-spec@2024-01-15/authentication-requirements
```

**Cross-references in this doc:** [content:@doc-journey-graph/coordinates](#), [content:@doc-concept-graph/references](#)
<!--semcontext:segment end-->

## Migration

### From Current Semcontext

```yaml
# Old
doc_123#intro
cfn_abc123

# New
content:@doc_123/intro
code:@cfn_abc123/FunctionName
```

### From Structured Coordinates

```yaml
# Old
doc_coordinate:
  target: developer-portal
  key: /docs/auth
  section: getting-started

# New
content:@doc-auth-guide/getting-started
```

## See Also

- [content:@doc-project-definition/overview](#) - Project definitions
- [content:@doc-journey-graph/format](#) - Journey graph format
- [content:@doc-concept-definition/relationships](#) - Concept relationships
- [content:@doc-linkage-mapping/coordinates](#) - Code-doc linking
