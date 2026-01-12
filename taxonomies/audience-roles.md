# Audience Role Taxonomy

**Version:** 1.0.0
**Status:** Draft

## Overview

Standard roles for targeting documentation content. Enables role-based content filtering and personalization.

## Roles

### Technical Roles

**developer**
- Software developers
- Engineers writing code
- Needs: API references, code examples, integration guides
- Skill level: Technical, programming knowledge

**operator**
- DevOps engineers
- SREs, infrastructure teams
- Needs: Deployment guides, monitoring, troubleshooting
- Skill level: Technical, infrastructure knowledge

**security**
- Security engineers
- AppSec, InfoSec teams
- Needs: Security policies, threat models, compliance
- Skill level: Technical, security expertise

### Administrative Roles

**admin**
- System administrators
- IT administrators
- Needs: Installation, configuration, user management
- Skill level: Technical, systems knowledge

**user**
- End users
- Product users
- Needs: User guides, tutorials, getting started
- Skill level: Non-technical to intermediate

### Business Roles

**product**
- Product managers
- Product owners
- Needs: Product requirements, roadmaps, user journeys
- Skill level: Business, non-technical

**engineering**
- Engineering managers
- Tech leads
- Needs: Architecture docs, technical strategy, RFCs
- Skill level: Technical leadership

**executive**
- C-level executives
- Senior leadership
- Needs: Strategy docs, high-level overviews, metrics
- Skill level: Business, strategic

**legal**
- Legal counsel
- Compliance officers
- Needs: Policies, terms, compliance documentation
- Skill level: Legal, regulatory

### Internal

**internal**
- Internal company use only
- Not for external distribution
- Needs: Internal processes, proprietary information
- Skill level: Varies

## Multi-Role Content

Documents can target multiple roles:

```yaml
audience_role: [developer, operator]  # For both devs and ops
```

```markdown
<!--werkcontext:segment start key="deployment" audience="developer,operator"-->
## Deployment Guide
...
<!--werkcontext:segment end-->
```

## Role Hierarchies

Some roles have hierarchical relationships:

```
technical
├── developer
├── operator
└── security

business
├── product
├── engineering
└── executive

administrative
├── admin
└── user
```

## Usage

### In Frontmatter

```yaml
---
werkcontext:
  segments:
    - id: getting-started
      type: tutorial
      audience_role: user           # Single role

    - id: api-reference
      type: reference
      audience_role: [developer]    # Array with single role

    - id: deployment
      type: howto
      audience_role: [developer, operator]  # Multiple roles
---
```

### In Segment Markers

```markdown
<!--werkcontext:segment start key="user-guide" audience="user"-->
For end users
<!--werkcontext:segment end-->

<!--werkcontext:segment start key="dev-guide" audience="developer,operator"-->
For technical teams
<!--werkcontext:segment end-->
```

### In Classification Config

```yaml
audience_roles:
  developer:
    keywords: [api, sdk, library, package, integration, code]
    linked_types: [reference, howto, example]

  user:
    keywords: [getting started, tutorial, guide, how to]
    linked_types: [tutorial, howto, faq]
```

## Use Cases

### 1. Content Filtering

```python
# Show only content for developers
dev_segments = [
    s for s in segments
    if 'developer' in s.audience_role
]
```

### 2. Personalized Portals

```typescript
// Build role-specific documentation portal
const userType = getCurrentUser().role;
const relevantDocs = docs.filter(doc =>
  doc.frontmatter.werkcontext.segments.some(seg =>
    seg.audience_role.includes(userType)
  )
);
```

### 3. Access Control

```go
// Check if user can access segment
func CanAccess(userRole string, segment Segment) bool {
    for _, role := range segment.AudienceRole {
        if role == userRole || role == "user" {
            return true
        }
    }
    return false
}
```

### 4. Analytics

```sql
-- Document usage by role
SELECT
  audience_role,
  COUNT(DISTINCT document_id) as doc_count
FROM segment_views
GROUP BY audience_role
```

## Validation

- Role MUST be from this taxonomy (or custom with prefix)
- Multiple roles allowed (array or comma-separated)
- Empty audience_role means accessible to all

## Custom Roles

Organizations can add custom roles with prefix:

```yaml
audience_role: "acme:field-engineer"
```

Standard roles (listed above) have no prefix.

## See Also

- [Frontmatter](../formats/frontmatter.md) - How to specify audience in frontmatter
- [Segment Markers](../formats/segment-markers.md) - How to specify audience in markers
- [Segment Taxonomy](./segments.md) - Segment types
