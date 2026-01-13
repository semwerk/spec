# External Annotation Files

**Version:** 1.0.0
**Status:** Draft
**Format:** JSON

<!--semcontext:segment start key="overview" type="overview" audience="developer"-->
## Overview

External annotation files (`.semwerk.anno.json`) provide an alternative to inline segment markers and frontmatter for annotating content. Placed alongside the original file, they define segments without modifying the source.

**Use cases:**
- Files you don't control (third-party docs, generated files)
- Files where comments aren't allowed (JSON, CSV, binary formats)
- Separation of content from metadata
- Multi-source annotations (different teams annotating same file)

**File naming:** `<original-file>.semwerk.anno.json`

**Examples:**
- `README.md` → `README.md.semwerk.anno.json`
- `api.json` → `api.json.semwerk.anno.json`
- `diagram.png` → `diagram.png.semwerk.anno.json`
<!--semcontext:segment end-->

<!--semcontext:segment start key="format" type="reference" audience="developer"-->
## Format Schema

### Complete Schema (All Fields)

```json
{
  "version": "1",
  "source_file": "<string>",
  "source_checksum": "<string>",

  "semantics": {
    "tasks": ["<task-ref>"],
    "problems": ["<problem-ref>"],
    "solutions": ["<solution-ref>"],
    "outcomes": ["<outcome-ref>"],
    "features": ["<feature-ref>"],
    "personas": ["<persona-ref>"]
  },

  "segments": [
    {
      "id": "<string>",
      "type": "<string>",
      "audience_role": ["<string>"] | "<string>",
      "concepts": ["<string>"] | "<string>",
      "boost": <number>,

      "semantics": {
        "tasks": ["<task-ref>"],
        "problems": ["<problem-ref>"],
        "solutions": ["<solution-ref>"],
        "outcomes": ["<outcome-ref>"],
        "validations": ["<validation-ref>"],
        "next_steps": ["<next-step-ref>"],
        "impacts": ["<impact-ref>"],
        "features": ["<feature-ref>"]
      },

      "byte_range": {
        "start": <number>,
        "end": <number>
      },
      "line_range": {
        "start": <number>,
        "end": <number>
      },
      "heading": "<string>",
      "segment_checksum": "<string>",
      "segment_ref": "<string>",
      "can_edit": <boolean>,

      "return": {
        "max_tokens": <number>,
        "min_tokens": <number>
      },
      "generate": {
        "max_tokens": <number>,
        "min_tokens": <number>,
        "model": "<string>",
        "temperature": <number>,
        "context": "<string>"
      }
    }
  ],
  "metadata": {
    "annotated_by": "<string>",
    "description": "<string>",
    "date_created": "<timestamp>",
    "date_updated": "<timestamp>"
  }
}
```

### Field Requirements and Rationale

#### Top-Level Fields

| Field | Requirement | Type | Description | Why You Need It |
|-------|-------------|------|-------------|-----------------|
| `version` | **MUST** | string | Schema version (currently "1") | Version evolution, backward compatibility |
| `source_file` | **MUST** | string | Relative path to annotated file | Which file this annotates |
| `source_checksum` | **SHOULD** | string | Checksum of entire source file | Detect if file changed at all, fail fast before processing segments |
| `semantics` | **MAY** | object | Content-level semantic entity references | Document-level SpecBook entity linking (tasks, problems, solutions, outcomes, features, personas) |
| `segments` | **MUST** | array | Array of segment definitions | The actual annotations |
| `metadata` | **MAY** | object | Annotation metadata | Tracking, audit trail |

#### Content-Level Semantics

| Field | Requirement | Type | Description | Why You Need It |
|-------|-------------|------|-------------|-----------------|
| `semantics.tasks` | **MAY** | array | Task references (task:@stsk_*) | Link content to user tasks |
| `semantics.problems` | **MAY** | array | Problem references (problem:@sprb_*) | Link content to pain points |
| `semantics.solutions` | **MAY** | array | Solution references (solution:@ssol_*) | Link content to solutions |
| `semantics.outcomes` | **MAY** | array | Outcome references (outcome:@sout_*) | Link content to desired outcomes |
| `semantics.features` | **MAY** | array | Feature references (feature:@sftr_*) | Link content to product features |
| `semantics.personas` | **MAY** | array | Persona references (persona:@per_*) | Link content to target personas |

#### Segment Fields (Core Identity)

| Field | Requirement | Type | Description | Why You Need It |
|-------|-------------|------|-------------|-----------------|
| `id` | **MUST** | string | Unique segment identifier | References, lookups, coordinates (`content:@doc-id/segment-id`) |
| `type` | **SHOULD** | string | Segment type (overview, howto, reference, etc.) | Content classification, filtering, retrieval strategy |
| `heading` | **SHOULD** | string | Associated heading/title | Human readability, TOC generation, display |
| `segment_ref` | **MAY** | string | Full coordinate reference | Pre-computed coordinate for quick lookup (e.g., `content:@doc-abc123/overview`) |

#### Segment Fields (Content Extraction)

| Field | Requirement | Type | Description | Why You Need It |
|-------|-------------|------|-------------|-----------------|
| `byte_range` | **SHOULD** | object | Byte offset range {start, end} | Precise binary extraction, works with any file type |
| `line_range` | **SHOULD** | object | Line number range {start, end} | Human-readable extraction for text files, easier to maintain |
| `segment_checksum` | **SHOULD** | string | Checksum of segment content | Detect segment-level drift without re-hashing entire file, selective re-processing |
| `can_edit` | **MAY** | boolean | Whether segment can be edited | Generated vs. static segments, edit permissions |

**Note:** At least ONE of `byte_range` or `line_range` MUST be present. Use `byte_range` for binary files, `line_range` for text files (preferred).

#### Segment Fields (Semantic Metadata)

| Field | Requirement | Type | Description | Why You Need It |
|-------|-------------|------|-------------|-----------------|
| `audience_role` | **SHOULD** | array or string | Target audience(s) | Content filtering, personalization, role-based access |
| `concepts` | **SHOULD** | array or string | Associated concept keys | Semantic search, concept-based retrieval, knowledge graph linking |
| `boost` | **MAY** | number (0-10) | Relevance multiplier (default: 1.0) | Ranking, prioritization, relevance scoring in RAG systems |
| `semantics` | **MAY** | object | Segment-level SpecBook entity references | Link specific segment to tasks, problems, solutions, etc. |

#### Segment-Level Semantics

| Field | Requirement | Type | Description | Why You Need It |
|-------|-------------|------|-------------|-----------------|
| `semantics.tasks` | **MAY** | array | Task references (task:@stsk_*) | Link segment to user tasks it addresses |
| `semantics.problems` | **MAY** | array | Problem references (problem:@sprb_*) | Link segment to problems it solves |
| `semantics.solutions` | **MAY** | array | Solution references (solution:@ssol_*) | Link segment to solution approaches |
| `semantics.outcomes` | **MAY** | array | Outcome references (outcome:@sout_*) | Link segment to outcomes it enables |
| `semantics.validations` | **MAY** | array | Validation references (validation:@sval_*) | Link segment to validation criteria |
| `semantics.next_steps` | **MAY** | array | Next step references (next_step:@snxt_*) | Link segment to recommended actions |
| `semantics.impacts` | **MAY** | array | Impact references (impact:@simp_*) | Link segment to impacts/benefits |
| `semantics.features` | **MAY** | array | Feature references (feature:@sftr_*) | Link segment to product features |

**Note:** Content-level semantics (tasks, problems, solutions, outcomes, features, personas) apply to entire document. Segment-level semantics apply to specific section only.

#### Segment Fields (Token Budgets)

| Field | Requirement | Type | Description | Why You Need It |
|-------|-------------|------|-------------|-----------------|
| `return.max_tokens` | **SHOULD** | number | Maximum tokens when retrieving | LLM context management, cost control, prevent context overflow |
| `return.min_tokens` | **MAY** | number | Minimum tokens when retrieving | Ensure enough context, quality threshold |
| `generate.max_tokens` | **MAY** | number | Maximum tokens when generating | Generation budget control, consistency, prevent runaway generation |
| `generate.min_tokens` | **MAY** | number | Minimum tokens when generating | Ensure sufficient detail, quality threshold (reject too-short outputs) |
| `generate.model` | **MAY** | string | LLM model for generation | Model-specific generation (gpt-4, claude-3-5-sonnet) |
| `generate.temperature` | **MAY** | number (0.0-2.0) | Generation temperature | Creativity control (0=deterministic, 2=creative) |
| `generate.context` | **MAY** | string | Context type (code_diff, git_log, none) | What context to include when generating |

### RFC 2119 Keywords

- **MUST** - Absolute requirement
- **SHOULD** - Recommended, may have valid reasons to ignore
- **MAY** - Truly optional, use when needed

### Complete Field Example (All Features)

```json
{
  "version": "1",
  "source_file": "docs/authentication.md",
  "source_checksum": "sha256:file1234567890abcdef",

  "semantics": {
    "tasks": ["task:@stsk_setup_auth", "task:@stsk_configure_oauth"],
    "problems": ["problem:@sprb_auth_complexity"],
    "solutions": ["solution:@ssol_simplified_auth"],
    "outcomes": ["outcome:@sout_secure_auth"],
    "features": ["feature:@sftr_oauth_integration"],
    "personas": ["persona:@per_backend_dev", "persona:@per_devops"]
  },

  "segments": [
    {
      "id": "oauth-setup",
      "type": "howto",
      "audience_role": ["developer", "operator"],
      "concepts": ["oauth", "authentication", "setup"],
      "boost": 1.5,

      "semantics": {
        "tasks": ["task:@stsk_configure_oauth"],
        "problems": ["problem:@sprb_oauth_config_difficulty"],
        "solutions": ["solution:@ssol_oauth_wizard"],
        "outcomes": ["outcome:@sout_oauth_working"],
        "validations": ["validation:@sval_oauth_flow"],
        "next_steps": ["next_step:@snxt_test_oauth"],
        "impacts": ["impact:@simp_faster_integration"],
        "features": ["feature:@sftr_oauth_pkce"]
      },

      "byte_range": {
        "start": 1250,
        "end": 3480
      },
      "line_range": {
        "start": 45,
        "end": 89
      },
      "heading": "## OAuth Setup",
      "segment_checksum": "sha256:segment9876543210fedcba",
      "segment_ref": "content:@doc-authentication@v2.0.0/oauth-setup",
      "can_edit": true,

      "return": {
        "max_tokens": 800,
        "min_tokens": 200
      },
      "generate": {
        "max_tokens": 1000,
        "min_tokens": 300,
        "model": "gpt-4",
        "temperature": 0.3,
        "context": "code_diff"
      }
    }
  ],
  "metadata": {
    "annotated_by": "auth-team",
    "description": "Authentication documentation segments with full semantic linking",
    "date_created": "2026-01-10T00:00:00Z",
    "date_updated": "2026-01-10T12:00:00Z"
  }
}
```
<!--semcontext:segment end-->

<!--semcontext:segment start key="examples" type="example" audience="developer"-->
## Examples

### Markdown File Annotation

**File:** `README.md`
**Annotation:** `README.md.semwerk.anno.json`

```json
{
  "version": "1",
  "source_file": "README.md",
  "source_checksum": "sha256:abc123def456",

  "semantics": {
    "tasks": ["task:@stsk_understand_product"],
    "personas": ["persona:@per_developer", "persona:@per_user"],
    "outcomes": ["outcome:@sout_product_clarity"]
  },

  "segments": [
    {
      "id": "overview",
      "type": "overview",
      "audience_role": ["user", "developer"],
      "concepts": ["getting-started"],
      "boost": 1.5,

      "semantics": {
        "tasks": ["task:@stsk_read_overview"],
        "outcomes": ["outcome:@sout_understand_purpose"],
        "next_steps": ["next_step:@snxt_read_installation"]
      },

      "line_range": {
        "start": 1,
        "end": 25
      },
      "heading": "# Overview",
      "segment_checksum": "sha256:1a2b3c4d5e6f",

      "return": {
        "max_tokens": 800,
        "min_tokens": 200
      }
    },
    {
      "id": "installation",
      "type": "howto",
      "audience_role": ["developer"],
      "concepts": ["installation", "setup"],
      "boost": 1.2,

      "semantics": {
        "tasks": ["task:@stsk_install_package"],
        "problems": ["problem:@sprb_install_errors"],
        "solutions": ["solution:@ssol_package_manager"],
        "validations": ["validation:@sval_install_success"]
      },

      "line_range": {
        "start": 27,
        "end": 65
      },
      "heading": "## Installation",
      "segment_checksum": "sha256:7g8h9i0j1k2l",

      "return": {
        "max_tokens": 600,
        "min_tokens": 150
      },
      "generate": {
        "max_tokens": 800,
        "min_tokens": 200,
        "model": "gpt-4",
        "temperature": 0.2,
        "context": "none"
      }
    },
    {
      "id": "api-reference",
      "type": "reference",
      "audience_role": ["developer"],
      "concepts": ["api"],
      "boost": 1.0,

      "semantics": {
        "features": ["feature:@sftr_api_v2"],
        "tasks": ["task:@stsk_use_api"]
      },

      "line_range": {
        "start": 67,
        "end": 150
      },
      "heading": "## API Reference",
      "segment_checksum": "sha256:3m4n5o6p7q8r",

      "return": {
        "max_tokens": 1200,
        "min_tokens": 400
      }
    }
  ],
  "metadata": {
    "annotated_by": "product-team",
    "date_created": "2026-01-10T00:00:00Z",
    "date_updated": "2026-01-10T12:00:00Z"
  }
}
```

### Non-Markdown File (JSON API spec)

**File:** `openapi.json`
**Annotation:** `openapi.json.semwerk.anno.json`

```json
{
  "version": "1",
  "source_file": "openapi.json",
  "source_checksum": "sha256:xyz789abc123",
  "segments": [
    {
      "id": "auth-endpoints",
      "type": "api",
      "audience_role": ["developer"],
      "concepts": ["authentication", "api"],
      "boost": 1.3,
      "byte_range": {
        "start": 1250,
        "end": 3480
      },
      "heading": "Authentication Endpoints",
      "segment_checksum": "sha256:9s0t1u2v3w4x",
      "return": {
        "max_tokens": 1000
      }
    },
    {
      "id": "payment-endpoints",
      "type": "api",
      "audience_role": ["developer"],
      "concepts": ["payments", "api"],
      "boost": 1.5,
      "byte_range": {
        "start": 3481,
        "end": 8920
      },
      "heading": "Payment Endpoints",
      "segment_checksum": "sha256:5y6z7a8b9c0d"
    }
  ],
  "metadata": {
    "annotated_by": "api-team",
    "date_created": "2026-01-10T00:00:00Z"
  }
}
```

### Binary File (Image/Diagram)

**File:** `architecture-diagram.png`
**Annotation:** `architecture-diagram.png.semwerk.anno.json`

```json
{
  "version": "1",
  "source_file": "architecture-diagram.png",
  "source_checksum": "sha256:def456abc789",
  "segments": [
    {
      "id": "system-architecture",
      "type": "conceptual",
      "audience_role": ["engineering", "product"],
      "concepts": ["architecture", "system-design"],
      "boost": 1.8,
      "heading": "System Architecture Diagram",
      "segment_checksum": "sha256:def456abc789",
      "return": {
        "max_tokens": 1200
      }
    }
  ],
  "metadata": {
    "annotated_by": "architecture-team",
    "description": "System architecture overview diagram",
    "note": "For binary files, segment_checksum typically equals source_checksum",
    "date_created": "2026-01-10T00:00:00Z"
  }
}
```

### Third-Party Documentation

**File:** `node_modules/@stripe/stripe-js/README.md`
**Annotation:** `node_modules/@stripe/stripe-js/README.md.semwerk.anno.json`

```json
{
  "version": "1",
  "source_file": "node_modules/@stripe/stripe-js/README.md",
  "source_checksum": "sha256:stripe123abc",
  "segments": [
    {
      "id": "stripe-quickstart",
      "type": "tutorial",
      "audience_role": ["developer"],
      "concepts": ["stripe", "payments", "integration"],
      "boost": 1.0,
      "line_range": {
        "start": 15,
        "end": 89
      },
      "heading": "Quick Start",
      "segment_checksum": "sha256:quickstart456def"
    }
  ],
  "metadata": {
    "annotated_by": "integration-team",
    "note": "Third-party docs we don't control",
    "date_created": "2026-01-10T00:00:00Z"
  }
}
```
<!--semcontext:segment end-->

<!--semcontext:segment start key="precedence" type="reference" audience="developer"-->
## Annotation Method Precedence

When multiple annotation methods exist for the same file:

**Priority order:**
1. **Inline segment markers** (highest) - HTML comments in file
2. **Frontmatter** - YAML at top of file
3. **External annotation file** (lowest) - `.semwerk.anno.json`

**Rules:**
- **NEVER use inline markers + frontmatter together** (choose one)
- External annotations are overridden by inline markers or frontmatter (if present)
- External annotations are additive (multiple `.anno.json` files can exist for same source)
- If segments conflict, highest priority wins
- External annotations have ALL the same fields as inline/frontmatter (parity)

**Example conflict resolution:**

File has inline marker:
```markdown
<!--semcontext:segment start key="overview" type="overview"-->
## Overview
<!--semcontext:segment end-->
```

External annotation also defines "overview":
```json
{
  "segments": [
    {"id": "overview", "type": "reference"}
  ]
}
```

**Result:** Inline marker wins (type: `overview`, not `reference`)
<!--semcontext:segment end-->

<!--semcontext:segment start key="parser-integration" type="howto" audience="developer"-->
## Parser Integration

### Discovery Algorithm

```typescript
function findSegmentAnnotations(filePath: string): SegmentSource {
  const content = readFile(filePath);

  // 1. Check for inline markers (highest priority)
  if (hasInlineMarkers(content)) {
    return {
      method: 'inline',
      segments: parseInlineMarkers(content),
    };
  }

  // 2. Check for frontmatter
  if (hasFrontmatter(content)) {
    return {
      method: 'frontmatter',
      segments: parseFrontmatter(content).segments,
    };
  }

  // 3. Check for external annotation file
  const annoFile = `${filePath}.semwerk.anno.json`;
  if (fileExists(annoFile)) {
    return {
      method: 'external',
      segments: parseExternalAnnotations(annoFile).segments,
    };
  }

  return { method: 'none', segments: [] };
}
```

### Loading External Annotations

```typescript
import { parseExternalAnnotations } from '@semwerk/semspec';

const annotations = parseExternalAnnotations('README.md.semwerk.anno.json');

// Validate checksum matches source
const source = readFile(annotations.source_file);
const checksum = sha256(source);

if (checksum !== annotations.source_checksum) {
  console.warn('Annotation file checksum mismatch - source may have changed');
}

// Use segments
for (const segment of annotations.segments) {
  console.log(`Segment: ${segment.id} (${segment.type})`);

  // Extract content by line range
  if (segment.line_range) {
    const lines = source.split('\n');
    const content = lines
      .slice(segment.line_range.start - 1, segment.line_range.end)
      .join('\n');
  }

  // Or by byte range
  if (segment.byte_range) {
    const content = source.substring(
      segment.byte_range.start,
      segment.byte_range.end
    );
  }
}
```

### Python Example

```python
from semspec import parse_external_annotations

annotations = parse_external_annotations('README.md.semwerk.anno.json')

# Validate checksum
with open(annotations.source_file, 'rb') as f:
    content = f.read()
    checksum = hashlib.sha256(content).hexdigest()

if f'sha256:{checksum}' != annotations.source_checksum:
    print('Warning: Source file has changed since annotation')

# Extract segments by line range
for segment in annotations.segments:
    if segment.line_range:
        lines = content.decode().split('\n')
        segment_content = '\n'.join(
            lines[segment.line_range.start-1:segment.line_range.end]
        )
```

### Go Example

```go
import "github.com/semwerk/semspec-go/annotations"

anno, err := annotations.Parse("README.md.semwerk.anno.json")
if err != nil {
    log.Fatal(err)
}

// Validate checksum
source, _ := os.ReadFile(anno.SourceFile)
checksum := sha256.Sum256(source)
expectedChecksum := fmt.Sprintf("sha256:%x", checksum)

if anno.SourceChecksum != "" && anno.SourceChecksum != expectedChecksum {
    log.Printf("Warning: Source file checksum mismatch")
}

// Extract segments
for _, segment := range anno.Segments {
    if segment.LineRange != nil {
        lines := bytes.Split(source, []byte("\n"))
        content := bytes.Join(
            lines[segment.LineRange.Start-1:segment.LineRange.End],
            []byte("\n"),
        )
    }
}
```
<!--semcontext:segment end-->

<!--semcontext:segment start key="benefits-tradeoffs" type="reference" audience="developer,engineering"-->
## Benefits and Tradeoffs

### Inline Markers (Preferred)

**Benefits:**
- Segments visible in source
- No separate file to manage
- Segments move with content
- Works with git diff/merge

**Tradeoffs:**
- Requires file modification
- Only works with comment-friendly formats
- Visible in rendered output (HTML comments)

### Frontmatter

**Benefits:**
- Structured metadata at top
- Easy to parse
- Standard in static site generators

**Tradeoffs:**
- Requires file modification
- Only works with formats that support frontmatter
- Pushes content below metadata

### External Annotations

**Benefits:**
- **No file modification** - Original untouched
- Works with **any file type** (binary, JSON, generated)
- Multiple annotation files possible
- Annotations can be versioned separately

**Tradeoffs:**
- **Separate file to manage** - Can get out of sync
- Byte/line offsets brittle (content changes break ranges)
- Requires checksum validation
- Less discoverable

**Recommendation:**
1. Use **inline markers** when possible (preferred)
2. Use **frontmatter** for static sites with frontmatter support
3. Use **external annotations** only when you can't modify source file
<!--semcontext:segment end-->

<!--semcontext:segment start key="use-cases" type="howto" audience="developer"-->
## Use Cases

### 1. Third-Party Documentation

Annotate dependencies without forking:

```bash
# Original file (don't modify)
node_modules/@stripe/stripe-js/README.md

# Your annotation
node_modules/@stripe/stripe-js/README.md.semwerk.anno.json
```

### 2. Generated Files

Annotate auto-generated files:

```bash
# Generated OpenAPI spec
api/openapi.json

# Your semantic annotations
api/openapi.json.semwerk.anno.json
```

### 3. Binary Assets

Annotate images, PDFs, videos:

```bash
# Architecture diagram
docs/architecture.png

# Semantic metadata
docs/architecture.png.semwerk.anno.json
```

### 4. Multi-Team Annotations

Different teams annotate same file:

```bash
README.md
README.md.semwerk.anno.marketing.json    # Marketing team's view
README.md.semwerk.anno.engineering.json  # Engineering team's view
```

### 5. Version-Specific Annotations

Different annotations for different versions:

```bash
api-v1.json
api-v1.json.semwerk.anno.json            # v1 annotations

api-v2.json
api-v2.json.semwerk.anno.json            # v2 annotations (different segments)
```
<!--semcontext:segment end-->

<!--semcontext:segment start key="validation" type="reference" audience="developer"-->
## Validation

### Schema Validation

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "required": ["version", "source_file", "segments"],
  "properties": {
    "version": {"type": "string", "const": "1"},
    "source_file": {"type": "string"},
    "source_checksum": {"type": "string"},
    "segments": {
      "type": "array",
      "items": {
        "type": "object",
        "required": ["id"],
        "properties": {
          "id": {"type": "string"},
          "type": {"type": "string"},
          "boost": {"type": "number", "minimum": 0, "maximum": 10}
        }
      }
    }
  }
}
```

### Checksum Validation

**File-level checksum** (`source_checksum`):

```typescript
const anno = parseExternalAnnotations('file.semwerk.anno.json');
const source = readFile(anno.source_file);
const actualChecksum = `sha256:${sha256(source)}`;

if (anno.source_checksum && anno.source_checksum !== actualChecksum) {
  throw new Error(
    `Source file has changed! Expected: ${anno.source_checksum}, Got: ${actualChecksum}`
  );
}
```

**Segment-level checksums** (`segment_checksum` - recommended):

```typescript
for (const segment of anno.segments) {
  // Extract segment content by range
  let segmentContent: string;

  if (segment.byte_range) {
    segmentContent = source.substring(
      segment.byte_range.start,
      segment.byte_range.end
    );
  } else if (segment.line_range) {
    const lines = source.split('\n');
    segmentContent = lines
      .slice(segment.line_range.start - 1, segment.line_range.end)
      .join('\n');
  }

  // Validate segment checksum
  const actualSegmentChecksum = `sha256:${sha256(segmentContent)}`;

  if (segment.segment_checksum && segment.segment_checksum !== actualSegmentChecksum) {
    console.warn(
      `Segment ${segment.id} has changed! Expected: ${segment.segment_checksum}, Got: ${actualSegmentChecksum}`
    );
    // Mark segment as stale
    segment._stale = true;
  }
}
```

**Benefits of segment checksums:**
- Detect which specific segments changed (not just whole file)
- Selectively re-process only changed segments
- Track segment-level drift over time
- More granular than file-level checksums

### Range Validation

Byte and line ranges must be valid:

```typescript
// Line range validation
if (segment.line_range) {
  const lines = source.split('\n');
  if (segment.line_range.end > lines.length) {
    throw new Error(`Line range exceeds file length`);
  }
}

// Byte range validation
if (segment.byte_range) {
  if (segment.byte_range.end > source.length) {
    throw new Error(`Byte range exceeds file size`);
  }
}
```
<!--semcontext:segment end-->

## See Also

- [content:@doc-segment-markers/overview](#) - Inline marker format
- [content:@doc-frontmatter/overview](#) - Frontmatter format
- [content:@doc-coordinate-spec/examples](#) - Coordinate system
