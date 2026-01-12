# Frontmatter Schema

**Version:** 1.0.0
**Status:** Draft
**Format:** YAML Frontmatter

## Overview

Frontmatter provides document-level metadata in YAML format at the beginning of markdown files. It defines:

- Segment specifications for the document
- Template associations
- Audience targeting
- Concept mappings
- Token budgets for LLM generation/retrieval

## Syntax

Frontmatter appears at the start of markdown files between `---` delimiters:

```markdown
---
template_id: @mytemplate

werkcontext:
  segments:
    - id: overview
      type: overview
      audience_role: [user, developer]
      concepts: [authentication, security]
      boost: 1.2
      return:
        max_tokens: 800
---

# Document Content

Your markdown content here...
```

## Schema

### Top-Level Structure

```yaml
template_id: <string>           # Optional template identifier

werkcontext:
  semantics:                    # Content-level semantic entities (optional)
    tasks: [<task-ref>]
    problems: [<problem-ref>]
    solutions: [<solution-ref>]
    outcomes: [<outcome-ref>]
    features: [<feature-ref>]
    personas: [<persona-ref>]

  segments:                     # Segment definitions
    - id: <string>              # Required
      type: <enum>              # Optional
      audience_role: <array>    # Optional
      concepts: <array>         # Optional
      boost: <number>           # Optional

      semantics:                # Segment-level semantic entities (optional)
        tasks: [<task-ref>]
        problems: [<problem-ref>]
        solutions: [<solution-ref>]
        outcomes: [<outcome-ref>]
        validations: [<validation-ref>]
        next_steps: [<next-step-ref>]
        impacts: [<impact-ref>]
        features: [<feature-ref>]

      return:                   # Optional
        max_tokens: <number>
        min_tokens: <number>
      generate:                 # Optional
        max_tokens: <number>
        min_tokens: <number>
        model: <string>
        temperature: <number>
        context: <string>
```

### Segment Definition

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string | Yes | Unique identifier for this segment |
| `type` | enum | No | Segment type (see [Segment Taxonomy](../taxonomies/segments.md)) |
| `audience_role` | string or array | No | Target audience(s) |
| `concepts` | string or array | No | Associated concepts (dot notation: `"auth.jwt"`) |
| `boost` | number | No | Relevance multiplier for retrieval (default: 1.0) |
| `semantics` | object | No | Segment-level SpecBook entity references (tasks, problems, solutions, etc.) |
| `return.max_tokens` | number | No | Maximum tokens when retrieving this segment |
| `return.min_tokens` | number | No | Minimum tokens when retrieving (quality threshold) |
| `generate.max_tokens` | number | No | Maximum tokens when generating this segment |
| `generate.min_tokens` | number | No | Minimum tokens when generating (quality threshold) |
| `generate.model` | string | No | LLM model to use (e.g., "gpt-4", "claude-3-5-sonnet") |
| `generate.temperature` | number | No | Generation temperature (0.0-2.0, default: 0.7) |
| `generate.context` | string | No | Context type (code_diff, git_log, none) |

### Audience Roles

- `user` - End users
- `developer` - Software developers
- `operator` - DevOps/SRE
- `admin` - System administrators
- `security` - Security engineers
- `product` - Product managers
- `engineering` - Engineering leads
- `executive` - Executive stakeholders

### Segment Types

- `overview` - High-level introduction
- `howto` - Step-by-step guide
- `reference` - API/technical reference
- `tutorial` - Learning-oriented walkthrough
- `troubleshooting` - Problem-solving guide
- `faq` - Frequently asked questions
- `example` - Code example
- `conceptual` - Concept explanation
- `rfc` - RFC/design document
- `policy` - Policy documentation

See [Segment Taxonomy](../taxonomies/segments.md) for complete list.

## Examples

### Simple Example

```yaml
---
werkcontext:
  semantics:
    tasks: [task:@stsk_quickstart]
    personas: [persona:@per_developer]

  segments:
    - id: quickstart
      type: tutorial
      audience_role: developer
      semantics:
        tasks: [task:@stsk_first_steps]
        next_steps: [next_step:@snxt_advanced_guide]
---

# Quick Start

Install and run in 5 minutes...
```

### Multiple Segments

```yaml
---
template_id: @api-reference

werkcontext:
  semantics:
    features: [feature:@sftr_api_v2, feature:@sftr_auth_system]
    personas: [persona:@per_backend_dev]
    tasks: [task:@stsk_integrate_api]

  segments:
    - id: authentication
      type: reference
      audience_role: developer
      concepts: [auth, security]
      boost: 1.5

      semantics:
        tasks: [task:@stsk_setup_auth]
        features: [feature:@sftr_oauth, feature:@sftr_jwt]
        problems: [problem:@sprb_auth_complexity]
        solutions: [solution:@ssol_auth_sdk]

      return:
        max_tokens: 1000
        min_tokens: 300

    - id: authorization
      type: reference
      audience_role: developer
      concepts: [auth, permissions]
      boost: 1.2

      semantics:
        tasks: [task:@stsk_configure_permissions]
        features: [feature:@sftr_rbac]

      return:
        max_tokens: 800
        min_tokens: 200

    - id: examples
      type: example
      audience_role: developer
      concepts: [auth]

      semantics:
        tasks: [task:@stsk_understand_auth_flow]
        next_steps: [next_step:@snxt_test_auth]

      return:
        max_tokens: 500
        min_tokens: 150
---

# API Documentation

<!--werkcontext:segment start key="authentication"-->
## Authentication
...
<!--werkcontext:segment end-->

<!--werkcontext:segment start key="authorization"-->
## Authorization
...
<!--werkcontext:segment end-->

<!--werkcontext:segment start key="examples"-->
## Examples
...
<!--werkcontext:segment end-->
```

### Multi-Audience Documentation

```yaml
---
werkcontext:
  segments:
    - id: user-guide
      type: howto
      audience_role: [user, operator]
      concepts: [deployment]
      boost: 1.0

    - id: developer-guide
      type: howto
      audience_role: developer
      concepts: [deployment, configuration]
      boost: 1.0

    - id: architecture
      type: conceptual
      audience_role: [developer, engineering]
      concepts: [architecture, design]
      boost: 0.8
---
```

### Token Budget Control

```yaml
---
werkcontext:
  segments:
    - id: overview
      type: overview
      return:
        max_tokens: 500       # Limit retrieval to 500 tokens
        min_tokens: 150       # Skip if < 150 tokens (too short)
      generate:
        max_tokens: 1000      # Stop generation at 1000 tokens
        min_tokens: 300       # Regenerate if output < 300 tokens
        model: "gpt-4"
        temperature: 0.5
        context: "code_diff"
```

## Use Cases

### 1. Content Retrieval (RAG)

```python
# Parse frontmatter
fm = parse_frontmatter('docs/api.md')

# Find segments for developers
dev_segments = [
    s for s in fm.werkcontext.segments
    if 'developer' in s.get('audience_role', [])
]

# Respect token budgets
for seg in dev_segments:
    max_tokens = seg.get('return', {}).get('max_tokens', 2000)
    # Truncate content to max_tokens
```

### 2. Template-Based Generation

```typescript
// Check template association
const fm = parseFrontmatter(markdown);
if (fm.template_id === '@api-reference') {
  // Use API reference template
  generateWithTemplate(fm.template_id, fm.werkcontext.segments);
}
```

### 3. Concept-Based Filtering

```python
# Find all docs about authentication
auth_docs = []
for doc in all_docs:
    fm = parse_frontmatter(doc)
    for seg in fm.werkcontext.segments:
        if 'auth' in seg.get('concepts', []):
            auth_docs.append((doc, seg))
```

### 4. Boost-Based Ranking

```javascript
// Calculate relevance score
segments.forEach(seg => {
  const baseScore = semanticSimilarity(query, seg.content);
  const boost = seg.boost || 1.0;
  seg.finalScore = baseScore * boost;
});

// Sort by final score
segments.sort((a, b) => b.finalScore - a.finalScore);
```

## Validation

The frontmatter YAML must:
- Be valid YAML
- Appear at the very start of the file
- Be enclosed in `---` delimiters
- Have unique segment IDs within the document
- Reference valid segment types and audience roles

## Tools

- **werkspec-ts**: Parse/validate frontmatter
- **werkspec-python**: Python parser
- **werkspec-go**: Go parser

## See Also

- [Segment Markers](./segment-markers.md) - Mark segments in markdown
- [Segment Taxonomy](../taxonomies/segments.md) - Valid segment types
- [Audience Roles](../taxonomies/audience-roles.md) - Valid audience roles
