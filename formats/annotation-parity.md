# Annotation Method Parity

**Version:** 1.0.0
**Status:** Draft

<!--werkcontext:segment start key="overview" type="overview" audience="developer"-->
## Overview

All three annotation methods (inline markers, frontmatter, external annotations) support the same fields. This document shows field parity across methods.

**Annotation methods:**
1. **Inline markers** - HTML comments in file (preferred)
2. **Frontmatter** - YAML at top of file
3. **External annotations** - `.semwerk.anno.json` file

**Rule:** NEVER use inline markers + frontmatter together in the same file.
<!--werkcontext:segment end-->

<!--werkcontext:segment start key="field-comparison" type="reference" audience="developer"-->
## Field Comparison

### Inline Markers

```markdown
<!--werkcontext:segment start
  key="oauth-setup"
  type="howto"
  audience="developer,operator"
-->
## OAuth Setup

Content here...
<!--werkcontext:segment end-->
```

**Supported fields:**
- `key` (id)
- `type`
- `audience` (audience_role)

**Limitations:**
- Cannot specify: boost, max_tokens, concepts, checksums
- Must be defined in frontmatter if needed

### Frontmatter

```yaml
---
werkcontext:
  segments:
    - id: oauth-setup
      type: howto
      audience_role: [developer, operator]
      concepts: [oauth, authentication]
      boost: 1.5
      return:
        max_tokens: 800
      generate:
        max_tokens: 1000
        model: "gpt-4"
        temperature: 0.3
        context: code_diff
---

## OAuth Setup
Content here...
```

**Supported fields:**
- `id`
- `type`
- `audience_role` (array or string)
- `concepts` (array or string)
- `boost`
- `return.max_tokens`
- `generate.max_tokens`
- `generate` (all sub-fields)

**Limitations:**
- Cannot specify: byte_range, line_range, heading (inferred), segment_checksum, segment_ref, can_edit

### External Annotations

```json
{
  "version": "1",
  "source_file": "authentication.md",
  "source_checksum": "sha256:file123",
  "segments": [
    {
      "id": "oauth-setup",
      "type": "howto",
      "audience_role": ["developer", "operator"],
      "concepts": ["oauth", "authentication"],
      "boost": 1.5,
      "byte_range": {"start": 1250, "end": 3480},
      "line_range": {"start": 45, "end": 89},
      "heading": "## OAuth Setup",
      "segment_checksum": "sha256:seg456",
      "segment_ref": "content:@doc-auth@v2.0.0/oauth-setup",
      "can_edit": true,
      "return": {"max_tokens": 800},
      "generate": {
        "max_tokens": 1000,
        "model": "gpt-4",
        "temperature": 0.3,
        "context": "code_diff"
      }
    }
  ]
}
```

**Supported fields:**
- ALL fields from inline markers
- ALL fields from frontmatter
- PLUS: byte_range, line_range, segment_checksum, segment_ref, can_edit, source_checksum

**Benefit:** Most expressive, supports all features
<!--werkcontext:segment end-->

<!--werkcontext:segment start key="field-parity-table" type="reference" audience="developer"-->
## Field Parity Matrix

| Field | Inline Marker | Frontmatter | External Anno | Why Supported |
|-------|---------------|-------------|---------------|---------------|
| **Identity** |
| `id`/`key` | ✓ (key) | ✓ (id) | ✓ (id) | **MUST** - Unique identifier |
| `type` | ✓ | ✓ | ✓ | **SHOULD** - Classification |
| `heading` | ✗ | ✗ (inferred) | ✓ | External only - explicit title |
| `segment_ref` | ✗ | ✗ | ✓ | External only - pre-computed coordinate |
| **Audience** |
| `audience_role` | ✓ (audience) | ✓ | ✓ | **SHOULD** - Content targeting |
| **Semantics** |
| `concepts` | ✗ | ✓ | ✓ | **SHOULD** - Semantic tagging |
| `boost` | ✗ | ✓ | ✓ | **MAY** - Relevance ranking |
| **Location** |
| `byte_range` | ✗ | ✗ | ✓ | External only - binary extraction |
| `line_range` | ✗ | ✗ | ✓ | External only - text extraction |
| **Integrity** |
| `segment_checksum` | ✗ | ✗ | ✓ | External only - drift detection |
| `can_edit` | ✗ | ✗ | ✓ | External only - edit control |
| **Token Budgets** |
| `return.max_tokens` | ✗ | ✓ | ✓ | **SHOULD** - Context management |
| `generate.max_tokens` | ✗ | ✓ | ✓ | **MAY** - Generation control |
| `generate.model` | ✗ | ✓ | ✓ | **MAY** - Model selection |
| `generate.temperature` | ✗ | ✓ | ✓ | **MAY** - Creativity control |
| `generate.context` | ✗ | ✓ | ✓ | **MAY** - Context type |

### Key Insights

**Inline markers:**
- Minimal fields (key, type, audience only)
- Best for quick segmentation
- Combine with frontmatter for full metadata

**Frontmatter:**
- Full semantic metadata
- Token budgets and generation config
- Missing: location ranges, checksums (not needed, content is in same file)

**External annotations:**
- EVERYTHING from inline + frontmatter
- PLUS: location ranges (byte/line)
- PLUS: checksums (file + segment)
- PLUS: explicit heading and segment_ref
- Most powerful, for files you can't modify
<!--werkcontext:segment end-->

<!--werkcontext:segment start key="examples" type="example" audience="developer"-->