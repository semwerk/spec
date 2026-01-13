# Segment Versioning (Extension)

**Status:** Draft  
**Version:** 0.1.0  
**Applies To:** `<!--semcontext:segment ...-->` markers and `segment_ref` resolution  
**Goal:** Make segment version semantics explicit (latest + semver), with inline metadata as the canonical source.

## Scope
- Adds optional version metadata to segments for systems that support multi-version docs.
- Inline attributes are canonical; frontmatter MAY exist but MUST NOT be required to resolve versions.
- Aligns with coordinate spec (`type:@id[@version][/segment]`) so `segment_ref` can include version.

## Inline Attributes (additions)

Segments MAY declare version metadata directly in the start marker:

```md
<!--semcontext:segment start
  key="overview"
  type="overview"
  audience="developer"
  concepts="architecture"
  boost="1.5"
  version="1.4.0"
-->
```

Supported attributes:

| Attribute  | Description | Examples |
|------------|-------------|----------|
| `version`  | Exact version for this segment. Overrides document default. | `version="1.4.0"`, `version="latest"` |
| `version_range` | Semver range this segment is valid for (npm/semver syntax). Mutually exclusive with `version`. | `version_range="^1.2.0"`, `version_range=">=1.0.0 <2.0.0"` |

Rules:
- Use **either** `version` **or** `version_range`, not both.
- If neither is present, the segment inherits the document version (e.g., folder/version metadata).
- `latest` is allowed for `version` to signal the “current” line.

## Resolution & segment_ref

- `segment_ref` format remains `<document_id>#<segment_id>`.
- When a document version is known (e.g., `doc_id@1.4.0`), resolvers SHOULD emit `segment_ref` as `<document_id>@<version>#<segment_id>`.
- If a segment declares `version`/`version_range`, resolvers MUST respect it when building indices and routing; a segment with an exact `version` MUST NOT be used outside that version.

## Validation

Validators SHOULD enforce:
- `version` and `version_range` are not both present.
- `version` conforms to semver or `latest`.
- `version_range` conforms to semver range syntax.
- If document version is unknown and segment declares neither, warn (but do not fail) that version inheritance is ambiguous.

## Processing Guidance

- **Inline-first**: consumers must be able to resolve segments with no frontmatter present.
- **Inheritance**: when a folder/version manifest declares `latest` or `vX.Y.Z`, that is the default for all contained segments unless overridden inline.
- **Reuse**: shared segments reused across versions may declare a `version_range` to avoid duplication while remaining precise.
- **Indexing**: indices SHOULD store both `segment_ref` and `segment_ref_with_version` to keep backward compatibility.

## Examples

Exact version:
```md
<!--semcontext:segment start key="install" version="1.3.0" audience="developer"-->
Install instructions for v1.3.0
<!--semcontext:segment end-->
```

Version range:
```md
<!--semcontext:segment start key="api-auth" version_range="^2.0.0" audience="developer"-->
Auth details for all 2.x versions
<!--semcontext:segment end-->
```

Inherit document version:
```md
<!--semcontext:segment start key="overview" type="overview"-->
Overview inherits the doc/version folder (e.g., latest).
<!--semcontext:segment end-->
```
