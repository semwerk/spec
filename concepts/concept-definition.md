# Concept Definition

**Version:** 1.0.0
**Status:** Draft
**Format:** YAML

## Overview

Concepts are semantic entities that represent ideas, technologies, domains, or topics. They form a knowledge graph with typed relationships.

## Format

```yaml
version: "1"
kind: concept-graph

concepts:
  - id: <string>                  # Unique concept ID (cpt_*)
    key: <string>                 # URL-safe identifier
    name: <string>                # Human-readable name
    description: <string>         # Concept description
    aliases: [<string>]           # Alternative names
    status: <enum>                # active | deprecated | proposed
    source: <enum>                # manual | discovered | imported
    confidence: <float>           # 0.0-1.0 (for discovered concepts)
    tags: [<string>]              # Classification tags

relationships:
  - from: <concept-ref>           # Source concept
    to: <concept-ref>             # Target concept
    kind: <enum>                  # parent | related | implements | documents | depends_on
    weight: <float>               # Relationship strength (0.0-1.0)
    description: <string>         # Optional: relationship description

metadata:
  created_by: <string>
  date_created: <date>
  date_updated: <date>
```

## Concept Fields

| Field | Required | Description |
|-------|----------|-------------|
| `id` | Yes | Unique identifier (cpt_*) |
| `key` | Yes | URL-safe key (unique within tenant) |
| `name` | Yes | Human-readable name |
| `description` | No | Detailed description |
| `aliases` | No | Alternative names, abbreviations |
| `status` | Yes | Lifecycle state |
| `source` | Yes | How concept was created |
| `confidence` | Conditional | Required for `discovered`, not for `manual` |
| `tags` | No | Classification tags |

## Concept Status

**active**
- Concept is current and in use
- Can be referenced in content/journeys

**deprecated**
- Concept still exists but discouraged
- Being phased out
- References should migrate to replacement

**proposed**
- Concept suggested but not yet adopted
- Under review
- May become active or rejected

## Concept Source

**manual**
- Explicitly created by person
- High confidence (1.0)
- Authoritative

**discovered**
- Automatically detected from code/docs
- Has confidence score (0.0-1.0)
- Should be reviewed

**imported**
- Imported from external system
- May have external ID mapping
- Confidence depends on source quality

## Relationship Types

See [Relationship Types](./relationship-types.md) for full taxonomy.

### parent

Hierarchical parent-child relationship

```yaml
- from: concept:@cpt_oauth
  to: concept:@cpt_authentication
  kind: parent
  weight: 1.0
```

### related

Thematic association (non-hierarchical)

```yaml
- from: concept:@cpt_authentication
  to: concept:@cpt_security
  kind: related
  weight: 0.8
```

### implements

Implementation relationship

```yaml
- from: concept:@cpt_jwt
  to: concept:@cpt_oauth
  kind: implements
  weight: 0.9
```

### documents

Documentation relationship

```yaml
- from: concept:@cpt_api_reference
  to: concept:@cpt_rest_api
  kind: documents
  weight: 1.0
```

### depends_on

Dependency relationship

```yaml
- from: concept:@cpt_payment_processing
  to: concept:@cpt_authentication
  kind: depends_on
  weight: 1.0
```

## Examples

### Simple Hierarchy

```yaml
version: "1"
kind: concept-graph

concepts:
  - id: cpt_authentication
    key: authentication
    name: "Authentication"
    description: "User identity verification"
    aliases: [auth, login, signin]
    status: active
    source: manual

  - id: cpt_oauth
    key: oauth
    name: "OAuth 2.0"
    description: "Delegated authorization protocol"
    status: active
    source: manual

  - id: cpt_jwt
    key: jwt
    name: "JSON Web Tokens"
    description: "Stateless authentication tokens"
    aliases: [jwt-token, bearer-token]
    status: active
    source: manual

relationships:
  - from: concept:@cpt_oauth
    to: concept:@cpt_authentication
    kind: parent
    weight: 1.0
    description: "OAuth is a type of authentication"

  - from: concept:@cpt_jwt
    to: concept:@cpt_authentication
    kind: parent
    weight: 1.0
    description: "JWT is a type of authentication"

  - from: concept:@cpt_jwt
    to: concept:@cpt_oauth
    kind: implements
    weight: 0.8
    description: "JWT commonly used with OAuth"
```

### Discovered Concepts

```yaml
version: "1"
kind: concept-graph

concepts:
  - id: cpt_rate_limiting
    key: rate-limiting
    name: "Rate Limiting"
    description: "API request throttling"
    status: active
    source: discovered
    confidence: 0.85
    tags:
      - api
      - performance
      - security

  - id: cpt_circuit_breaker
    key: circuit-breaker
    name: "Circuit Breaker Pattern"
    status: proposed
    source: discovered
    confidence: 0.65
    tags:
      - pattern
      - resilience

relationships:
  - from: concept:@cpt_rate_limiting
    to: concept:@cpt_api_design
    kind: related
    weight: 0.7
```

### Concept Network with Multiple Relationships

```yaml
version: "1"
kind: concept-graph

concepts:
  - id: cpt_payment_processing
    key: payment-processing
    name: "Payment Processing"
    status: active
    source: manual

  - id: cpt_fraud_detection
    key: fraud-detection
    name: "Fraud Detection"
    status: active
    source: manual

  - id: cpt_pci_compliance
    key: pci-compliance
    name: "PCI DSS Compliance"
    status: active
    source: manual

relationships:
  - from: concept:@cpt_fraud_detection
    to: concept:@cpt_payment_processing
    kind: related
    weight: 0.9

  - from: concept:@cpt_payment_processing
    to: concept:@cpt_pci_compliance
    kind: depends_on
    weight: 1.0

  - from: concept:@cpt_fraud_detection
    to: concept:@cpt_pci_compliance
    kind: related
    weight: 0.7
```

## Concept References in Journeys

Journeys can tag nodes with concepts:

```yaml
journey:
  nodes:
    - id: oauth-setup
      type: stage
      name: "Configure OAuth"
      concepts:
        - concept:@cpt_authentication@v1
        - concept:@cpt_oauth
        - concept:@cpt_security
```

## Concept References in Content

Content can be tagged with concepts:

```yaml
# Frontmatter
---
semcontext:
  segments:
    - id: oauth-guide
      type: tutorial
      concepts: [authentication, oauth, jwt]
---
```

Maps to:
```yaml
concepts:
  - concept:@cpt_authentication
  - concept:@cpt_oauth
  - concept:@cpt_jwt
```

## Coordinates

Concepts can be referenced:

```yaml
concept:@cpt_authentication                     # Basic reference
concept:@cpt_authentication@v1                  # Versioned concept
```

## Validation

- Concept ID MUST start with `cpt_`
- Key MUST be kebab-case, unique within tenant
- If `source: discovered`, confidence MUST be present (0.0-1.0)
- If `source: manual`, confidence MUST NOT be present
- Relationship `from` and `to` MUST reference existing concepts
- Relationship `kind` MUST be from taxonomy
- Weight MUST be 0.0-1.0

## Use Cases

### 1. Semantic Search

```python
# Find all content about authentication
auth_concept = find_concept('authentication')
related_content = find_content_by_concept(auth_concept)
```

### 2. Knowledge Graph

```typescript
// Build graph visualization
const concepts = parseConcepts('concepts.yaml');
const graph = buildKnowledgeGraph(concepts.relationships);
renderD3(graph);
```

### 3. Content Recommendations

```yaml
# User reading about OAuth
# System recommends related concepts
- JWT (implements)
- Security (related)
- API Design (related)
```

### 4. Discovery Validation

```python
# Review discovered concepts
discovered = [c for c in concepts if c.source == 'discovered']
for concept in discovered:
    if concept.confidence < 0.7:
        print(f"Review: {concept.name} (confidence: {concept.confidence})")
```

## See Also

- [Relationship Types](./relationship-types.md) - Relationship taxonomy
- [Concept Graph](./concept-graph.md) - Graph format
- [Coordinate System](../coordinates/COORDINATE_SPEC.md) - Reference format
- [Provenance Model](../models/provenance.md) - Source tracking
