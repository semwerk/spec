# Provenance Model

**Version:** 1.0.0
**Status:** Draft

## Overview

The Provenance Model defines how to track the origin and lineage of documentation content and metadata. Critical for:

- Attribution of AI-generated content
- Distinguishing inferred vs. human-asserted metadata
- Audit trails for compliance
- Confidence scoring for automated extraction
- Reproducibility

## Origin Types

### inferred

Content or metadata **automatically determined** by a system:
- LLM classification
- Heuristic analysis
- Pattern matching
- Semantic analysis

**Characteristics:**
- May be incorrect
- Has confidence score
- Can be overridden by assertions
- Should be reviewed

**Examples:**
- LLM classifies doc as "tutorial" (confidence: 0.8)
- Heuristic detects audience as "developer" from keywords
- Semantic search links code to doc segment

### asserted

Content or metadata **explicitly stated** by a human:
- Author-written frontmatter
- Manual linkage creation
- Explicit segment markers
- Code comments

**Characteristics:**
- Assumed correct
- No confidence score needed
- Takes precedence over inferred
- Authoritative

**Examples:**
- Author writes `type: tutorial` in frontmatter
- Developer adds segment marker with `audience="developer"`
- Manual code-to-doc link creation

## Provenance Structure

```yaml
provenance:
  origin: inferred | asserted
  source: <string>              # System or person that created this
  source_path: <string>         # Optional: file/function that generated
  source_checksum: <string>     # Optional: checksum of source content
  extracted_at: <timestamp>     # ISO 8601
  confidence: <float>           # 0.0-1.0 (only for inferred)
```

### Fields

| Field | Required | Description |
|-------|----------|-------------|
| `origin` | Yes | `inferred` or `asserted` |
| `source` | Yes | Who/what created this (e.g., "llm-classifier-v1", "user@example.com") |
| `source_path` | No | Reference to source material |
| `source_checksum` | No | Hash of source content (for reproducibility) |
| `extracted_at` | Yes | When this was created (ISO 8601) |
| `confidence` | Conditional | Required for `inferred`, not allowed for `asserted` |

## Examples

### Inferred by LLM

```yaml
segment:
  id: overview
  type: overview
  audience_role: [developer]
  concepts: [authentication]
  provenance:
    origin: inferred
    source: "llm-classifier-v1.2.0"
    source_path: "docs/auth.md"
    source_checksum: "sha256:abc123..."
    extracted_at: "2026-01-10T15:30:00Z"
    confidence: 0.85
```

### Inferred by Heuristic

```yaml
linkage:
  code: "auth.go:Login"
  doc: "docs/api/auth.md"
  segment_id: "login-endpoint"
  provenance:
    origin: inferred
    source: "keyword-matcher-v2.1"
    extracted_at: "2026-01-10T12:00:00Z"
    confidence: 0.65
```

### Asserted by Author

```yaml
segment:
  id: getting-started
  type: tutorial
  audience_role: [user]
  provenance:
    origin: asserted
    source: "author@semwerk.com"
    extracted_at: "2026-01-09T10:00:00Z"
```

### Asserted via Frontmatter

```markdown
---
semcontext:
  segments:
    - id: overview
      type: overview              # Asserted by author
      audience_role: developer    # Asserted by author
---

# Documentation

<!--semcontext:segment start key="overview" type="overview"-->
Content here (marker is assertion)
<!--semcontext:segment end-->
```

Metadata in frontmatter or markers is implicitly `asserted` with:
```yaml
provenance:
  origin: asserted
  source: "frontmatter"
  extracted_at: <file_modified_time>
```

## Confidence Scores

For `inferred` origin only:

| Range | Interpretation | Action |
|-------|---------------|---------|
| 0.9 - 1.0 | Very high confidence | Auto-accept |
| 0.7 - 0.89 | High confidence | Review recommended |
| 0.5 - 0.69 | Medium confidence | Manual review required |
| 0.0 - 0.49 | Low confidence | Likely incorrect, discard |

## Conflict Resolution

When both inferred and asserted metadata exist:

**Rule:** Asserted always wins

```yaml
# Author writes in frontmatter
type: tutorial              # asserted

# LLM analyzes and suggests
suggested_type: howto       # inferred, confidence: 0.8

# Result: Use "tutorial" (asserted takes precedence)
```

## Use Cases

### 1. AI Transparency

```markdown
> This content was automatically classified as "tutorial" by AI (confidence: 82%).
> [Edit classification](#)
```

### 2. Audit Trail

```json
{
  "segment_id": "auth-overview",
  "classification_history": [
    {
      "type": "overview",
      "provenance": {
        "origin": "inferred",
        "source": "llm-v1.0",
        "confidence": 0.75,
        "extracted_at": "2026-01-09T10:00:00Z"
      }
    },
    {
      "type": "reference",
      "provenance": {
        "origin": "asserted",
        "source": "author@semwerk.com",
        "extracted_at": "2026-01-10T14:00:00Z"
      }
    }
  ]
}
```

### 3. Quality Scoring

```python
# Calculate quality score based on provenance
def quality_score(metadata):
    if metadata.provenance.origin == 'asserted':
        return 1.0  # Human-verified
    else:
        return metadata.provenance.confidence * 0.8
```

### 4. Regeneration

```bash
# Regenerate only inferred metadata (keep assertions)
semcode analyze --regenerate-inferred
```

## Validation

- `origin` MUST be `inferred` or `asserted`
- `confidence` MUST be 0.0-1.0
- `confidence` REQUIRED for `inferred`, MUST NOT be present for `asserted`
- `extracted_at` MUST be ISO 8601 timestamp
- `source` MUST identify the creator

## Best Practices

1. **Always track provenance** for AI-generated content
2. **Include confidence scores** for inferred metadata
3. **Store checksums** for reproducibility
4. **Preserve history** when overriding inferred with asserted
5. **Document the source** (which LLM model, which version)

## See Also

- [Frontmatter](../formats/frontmatter.md) - Asserted metadata in frontmatter
- [Segment Markers](../formats/segment-markers.md) - Asserted markers
- [Classification Config](./classification.md) - Inference rules
