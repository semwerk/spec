# Linkage Mapping Format

**Version:** 1.0.0
**Status:** Draft
**Format:** YAML

## Overview

The Linkage Mapping format defines a bidirectional mapping between code symbols (functions, methods, classes) and documentation assets (markdown files, specific segments within docs). This enables tools to:

- Navigate from code to relevant documentation
- Navigate from documentation to implementing code
- Detect stale documentation when code changes
- Generate documentation coverage reports
- Power IDE extensions with contextual doc links

## File Location

`.werkcontext/linkage.yaml`

This file should be committed to version control and maintained alongside your codebase.

## Schema

### Top-Level Structure

```yaml
version: "1"                    # Schema version (required)
created_at: <timestamp>         # ISO 8601 timestamp
updated_at: <timestamp>         # ISO 8601 timestamp
meta:                           # Optional metadata
  source: <string>              # How this mapping was generated
code_to_assets: <object>        # Map from code symbols to docs
asset_to_code: <object>         # Reverse mapping from docs to code
```

### Code-to-Assets Mapping

Maps code symbols (identified by `file:function`) to documentation assets.

```yaml
code_to_assets:
  "<file_path>:<symbol_name>":
    created_at: <timestamp>
    updated_at: <timestamp>
    assets:
      - path: <string>          # Relative path to doc file
        segments:               # Specific sections within the doc
          - id: <string>        # Segment identifier
            heading: <string>   # Markdown heading
            lines: [start, end] # Line range (0,0 if unknown)
        relevance: <enum>       # primary | supporting | related
        doc_type: <enum>        # overview | howto | reference | etc.
```

**Symbol Naming:**
- Format: `<file_path>:<symbol_name>`
- `file_path`: Relative to repository root
- `symbol_name`: Function name, method name, or class name
- Examples:
  - `src/api/users.go:CreateUser`
  - `lib/parser/tokenizer.py:Tokenizer.parse`
  - `cmd/server/main.go:main`

**Relevance Levels:**
- `primary`: Primary documentation for this symbol
- `supporting`: Additional context or examples
- `related`: Tangentially related documentation

**Document Types:**
- `overview`: High-level overview
- `howto`: Step-by-step guide
- `reference`: API reference
- `tutorial`: Learning-oriented tutorial
- `troubleshooting`: Problem-solving guide
- `faq`: Frequently asked questions
- `example`: Code example
- `conceptual`: Concept explanation

### Asset-to-Code Mapping

Reverse mapping from documentation files to code symbols.

```yaml
asset_to_code:
  "<doc_path>":
    code_refs:
      - path: <string>          # Relative path to code file
        functions: [<string>]   # List of function names
        lines: [start, end]     # Line range (0,0 if unknown)
```

## Examples

### Simple Example

```yaml
version: "1"
created_at: 2026-01-10T00:00:00Z
updated_at: 2026-01-10T00:00:00Z

code_to_assets:
  "src/auth/login.go:Authenticate":
    created_at: 2026-01-10T00:00:00Z
    updated_at: 2026-01-10T00:00:00Z
    assets:
      - path: docs/api/authentication.md
        segments:
          - id: authenticate-endpoint
            heading: "### Authenticate Endpoint"
            lines: [45, 89]
        relevance: primary
        doc_type: reference

asset_to_code:
  "docs/api/authentication.md":
    code_refs:
      - path: src/auth/login.go
        functions: [Authenticate]
        lines: [0, 0]
```

### Complex Example

```yaml
version: "1"
created_at: 2026-01-10T00:00:00Z
updated_at: 2026-01-10T00:00:00Z
meta:
  source: "werkcode-analyzer"

code_to_assets:
  "pkg/parser/ast.go:Parse":
    created_at: 2026-01-09T10:00:00Z
    updated_at: 2026-01-10T15:30:00Z
    assets:
      - path: docs/architecture/parser.md
        segments:
          - id: ast-parsing
            heading: "## AST Parsing"
            lines: [120, 180]
        relevance: primary
        doc_type: overview
      - path: docs/guides/extending-parser.md
        segments:
          - id: parser-internals
            heading: "### Parser Internals"
            lines: [45, 67]
        relevance: supporting
        doc_type: howto
      - path: examples/parser-usage.md
        segments:
          - id: example-basic
            heading: "# Basic Usage"
            lines: [10, 25]
        relevance: related
        doc_type: example

  "pkg/parser/ast.go:Walk":
    created_at: 2026-01-09T10:00:00Z
    updated_at: 2026-01-09T10:00:00Z
    assets:
      - path: docs/architecture/parser.md
        segments:
          - id: ast-traversal
            heading: "## AST Traversal"
            lines: [200, 250]
        relevance: primary
        doc_type: overview

asset_to_code:
  "docs/architecture/parser.md":
    code_refs:
      - path: pkg/parser/ast.go
        functions: [Parse, Walk]
        lines: [0, 0]

  "docs/guides/extending-parser.md":
    code_refs:
      - path: pkg/parser/ast.go
        functions: [Parse]
        lines: [0, 0]

  "examples/parser-usage.md":
    code_refs:
      - path: pkg/parser/ast.go
        functions: [Parse]
        lines: [0, 0]
```

## Use Cases

### 1. IDE Extension - Go to Documentation

```typescript
// Parse linkage.yaml
const linkage = parseLinkage('.werkcontext/linkage.yaml');

// User clicks on function name in editor
const symbol = 'src/auth/login.go:Authenticate';
const docs = linkage.code_to_assets[symbol];

// Show documentation links in sidebar
docs.assets.forEach(asset => {
  showDocLink(asset.path, asset.segments[0].heading);
});
```

### 2. Documentation Coverage Report

```python
# Find all exported functions
all_functions = find_exported_functions()

# Check linkage
linkage = parse_linkage('.werkcontext/linkage.yaml')
documented = set(linkage['code_to_assets'].keys())

# Report coverage
coverage = len(documented) / len(all_functions) * 100
print(f"Documentation coverage: {coverage:.1f}%")
```

### 3. Stale Documentation Detection

```bash
# Get modified functions from git
git diff main --name-only --diff-filter=M | grep '.go$'

# Check linkage.yaml
# For each modified function, report linked docs that may need updates
```

### 4. Documentation Graph Visualization

```javascript
// Build graph of docs ↔ code relationships
const graph = {
  nodes: [],  // docs + code files
  edges: []   // linkages
};

// Visualize in force-directed graph
// Color nodes by doc coverage, staleness, etc.
```

## Validation

JSON Schema: [linkage-mapping-schema.json](./linkage-mapping-schema.json)

### Required Fields

- `version` (string)
- `code_to_assets` (object)
- `asset_to_code` (object)

### Constraints

- Symbol names MUST be unique within `code_to_assets`
- Asset paths MUST be unique within `asset_to_code`
- All paths MUST be relative to repository root
- Timestamps MUST be ISO 8601 format
- `code_to_assets` and `asset_to_code` MUST be consistent (bidirectional)

## Compatibility

| Version | Status | Notes |
|---------|--------|-------|
| 1.0.0 | Current | Initial release |

## Migration

### From unstructured comments

If you currently use code comments like:

```go
// See docs/api/authentication.md for details
func Authenticate() {}
```

You can extract these into linkage.yaml:

```yaml
code_to_assets:
  "auth.go:Authenticate":
    assets:
      - path: docs/api/authentication.md
        relevance: primary
        doc_type: reference
```

## Tools

- **werkspec-ts**: TypeScript parser/validator
- **werkspec-python**: Python parser/validator
- **werkspec-go**: Go parser/validator

## See Also

- [Segment Markers](./segment-markers.md) - How to mark segments within docs
- [Frontmatter](./frontmatter.md) - Document metadata schema
