# Content Types

**Version:** 1.0.0
**Status:** Draft

## Overview

ContentType extends the Diataxis framework with additional documentation categories. Used to classify content by its pedagogical purpose and structure at the document level.

## Content Types

### Diataxis-Aligned Types

These map directly to the Diataxis documentation framework:

| Type | Diataxis Equivalent | Description |
|------|---------------------|-------------|
| `tutorial` | TUTORIAL | Step-by-step learning lessons for beginners |
| `how-to` | HOW_TO | Task-oriented practical guides (assumes knowledge) |
| `reference` | REFERENCE | Technical descriptions for lookup |
| `explanation` | EXPLANATION | Conceptual understanding and theory |

### Extended Types

Additional types that extend beyond Diataxis:

| Type | Description | Use Case |
|------|-------------|----------|
| `guide` | General guidance, not strictly step-by-step | Product guides, getting started flows |
| `mixed` | Multiple content types in one document | Long-form docs with tutorials + reference |
| `info` | Informational content | Announcements, release notes, updates |

## Usage

### In DocType

DocType can specify a content type for classification:

```proto
message DocType {
  // ...
  optional ContentType content_type = 16;
}
```

```go
type DocType struct {
    ContentType *ContentType `json:"content_type,omitempty"`
}
```

### In GraphQL

```graphql
enum ContentType {
  TUTORIAL
  HOW_TO
  GUIDE
  REFERENCE
  EXPLANATION
  MIXED
  INFO
}

type DocType {
  contentType: ContentType
}
```

## Relationship to DiataxisType

ContentType is a superset of DiataxisType:

```go
// ToDiataxis converts a ContentType to its DiataxisType equivalent
func (c ContentType) ToDiataxis() *DiataxisType {
    switch c {
    case ContentTypeTutorial:
        return &DiataxisTutorial
    case ContentTypeHowTo:
        return &DiataxisHowTo
    case ContentTypeReference:
        return &DiataxisReference
    case ContentTypeExplanation:
        return &DiataxisExplanation
    default:
        return nil  // guide, mixed, info have no direct equivalent
    }
}
```

## Classification Guidelines

| Type | Answers | Voice | Structure |
|------|---------|-------|-----------|
| tutorial | "Can you teach me..." | Imperative | Sequential steps, hands-on |
| how-to | "How do I..." | Imperative | Goal-oriented, procedural |
| guide | "Help me understand..." | Explanatory | Flexible, conceptual |
| reference | "What is..." | Descriptive | Structured lookup |
| explanation | "Why..." | Explanatory | Narrative, conceptual |
| mixed | Multiple of above | Varies | Document-dependent |
| info | "What's new..." | Informative | Announcements, updates |

## Validation

- ContentType is optional on DocType
- When provided, must be one of the 7 defined values
- `guide`, `mixed`, and `info` have no Diataxis equivalent

## See Also

- [Segments](./segments.md) - Segment-level classification
- [Diataxis Framework](https://diataxis.fr/) - Original framework
