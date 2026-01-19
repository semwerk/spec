# Segment Markers

**Version:** 1.0.0
**Status:** Draft
**Format:** HTML Comments in Markdown

## Overview

Segment markers are HTML comments embedded in markdown files that identify distinct content sections. They enable:

- Precise targeting of doc sections from code
- Extraction of specific segments for RAG/LLM systems
- Granular documentation updates without touching surrounding content
- Reusable content segments across multiple docs

## Syntax

### Basic Marker

```markdown
<!--semcontext:segment start key="overview"-->
Content here...
<!--semcontext:segment end-->
```

### Marker with Type

```markdown
<!--semcontext:segment start key="api-auth" type="reference"-->
## Authentication API

The authentication API uses JWT tokens...
<!--semcontext:segment end-->
```

### Full Attributes

```markdown
<!--semcontext:segment start key="getting-started" type="tutorial" audience="developer,user"-->
# Getting Started

Follow these steps to...
<!--semcontext:segment end-->
```

### With Token Budgets (Inline)

Token budgets can be specified inline when frontmatter is not used or for quick annotations:

```markdown
<!--semcontext:segment start key="api-overview" type="reference" gen-max="1000" gen-min="200"-->
## API Overview

...
<!--semcontext:segment end-->
```

### Outline Placeholder

For outline-first workflows, use the placeholder marker to indicate sections awaiting generation:

```markdown
<!--semcontext:segment start key="authentication" type="reference" gen-max="800" gen-min="150"-->
## Authentication

<!--semcontext:outline-placeholder key="authentication"-->

<!--semcontext:segment end-->
```

## Attributes

| Attribute | Required | Description | Example Values |
|-----------|----------|-------------|----------------|
| `key` | Yes | Unique identifier within this document | `"overview"`, `"api-auth"`, `"troubleshooting-errors"` |
| `type` | No | Segment type | `"overview"`, `"howto"`, `"reference"`, `"tutorial"`, `"faq"`, `"example"` |
| `audience` | No | Target audience (comma-separated) | `"developer"`, `"user,operator"`, `"admin"` |
| `gen-max` | No | Maximum tokens for generation (inline shorthand for `generate.max_tokens`) | `"1000"`, `"500"` |
| `gen-min` | No | Minimum tokens for generation (inline shorthand for `generate.min_tokens`) | `"200"`, `"100"` |

**Note:** When both frontmatter and inline attributes are present, frontmatter takes precedence. Inline attributes are useful for quick annotations or when frontmatter is not practical.

## Examples

### API Documentation

```markdown
# User API

<!--semcontext:segment start key="create-user" type="reference" audience="developer"-->
## Create User

**Endpoint:** `POST /api/users`

**Request Body:**
```json
{
  "email": "user@example.com",
  "name": "John Doe"
}
```

**Response:**
```json
{
  "id": "usr_123",
  "email": "user@example.com"
}
```
<!--semcontext:segment end-->

<!--semcontext:segment start key="list-users" type="reference" audience="developer"-->
## List Users

**Endpoint:** `GET /api/users`

Returns a paginated list of users.
<!--semcontext:segment end-->
```

### Tutorial with Multiple Segments

```markdown
# Getting Started with Payments

<!--semcontext:segment start key="prerequisites" type="tutorial" audience="developer"-->
## Prerequisites

Before you begin, ensure you have:
- API key from the dashboard
- Node.js 18+ installed
- Basic understanding of REST APIs
<!--semcontext:segment end-->

<!--semcontext:segment start key="installation" type="tutorial" audience="developer"-->
## Installation

Install the SDK:

```bash
npm install @acme/payments
```
<!--semcontext:segment end-->

<!--semcontext:segment start key="first-charge" type="tutorial" audience="developer"-->
## Making Your First Charge

Create a charge:

```javascript
const charge = await payments.charges.create({
  amount: 1000,
  currency: 'usd'
});
```
<!--semcontext:segment end-->
```

## Rules

### 1. Nesting

Segments **cannot** be nested:

```markdown
<!-- ❌ INVALID -->
<!--semcontext:segment start key="outer"-->
  <!--semcontext:segment start key="inner"-->
  Content
  <!--semcontext:segment end-->
<!--semcontext:segment end-->
```

### 2. Key Uniqueness

Keys must be unique within a single file:

```markdown
<!-- ❌ INVALID -->
<!--semcontext:segment start key="example"-->
First example
<!--semcontext:segment end-->

<!--semcontext:segment start key="example"-->
Second example (duplicate key!)
<!--semcontext:segment end-->
```

### 3. Balanced Markers

Every `start` must have a corresponding `end`:

```markdown
<!-- ❌ INVALID -->
<!--semcontext:segment start key="incomplete"-->
Content without end marker
```

## Rendering

Segment markers are HTML comments, so they:
- Are invisible in rendered markdown (GitHub, static sites)
- Don't affect visual layout
- Are preserved in markdown processors
- Can be parsed with simple regex or HTML comment parsers

## Use Cases

### 1. RAG/LLM Context

Extract specific segments for AI context:

```python
# Extract only tutorial segments for beginners
tutorials = [s for s in segments if s.type == 'tutorial']
context = "\n\n".join([s.content for s in tutorials])
```

### 2. Documentation Coverage

```typescript
// Find code without linked doc segments
const linkedSegments = new Set();
linkage.code_to_assets.forEach(asset => {
  asset.segments?.forEach(seg => linkedSegments.add(seg.id));
});
```

### 3. Multi-Audience Docs

```python
# Extract segments for specific audience
developer_content = [
    s.content for s in segments
    if 'developer' in (s.audience or [])
]
```

## Tools

- **semspec-ts**: Parse/validate segment markers
- **semspec-python**: Python parser
- **semspec-go**: Go parser

## See Also

- [Linkage Mapping](./linkage-mapping.md) - Link segments to code
- [Frontmatter](./frontmatter.md) - Document-level metadata
- [Segment Taxonomy](../taxonomies/segments.md) - Segment types
