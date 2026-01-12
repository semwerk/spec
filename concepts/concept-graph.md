# Concept Graph Format

**Version:** 1.0.0
**Status:** Draft
**Format:** YAML

## Overview

Concept graphs represent knowledge networks with typed relationships. Can be visualized as directed graphs.

## Format

```yaml
version: "1"
kind: concept-graph

# Graph metadata
graph:
  id: <string>                    # Optional: graph identifier
  name: <string>                  # Optional: graph name
  description: <string>           # Optional: graph description

# Concepts (nodes)
concepts:
  - id: <string>                  # Concept ID (cpt_*)
    key: <string>                 # Kebab-case key
    name: <string>                # Display name
    description: <string>         # Description
    aliases: [<string>]           # Alternative names
    status: <enum>                # active | deprecated | proposed
    source: <enum>                # manual | discovered | imported
    confidence: <float>           # 0.0-1.0 (for discovered)
    tags: [<string>]              # Classification

# Relationships (edges)
relationships:
  - from: <concept-ref>           # Source concept
    to: <concept-ref>             # Target concept
    kind: <enum>                  # parent | related | implements | documents | depends_on
    weight: <float>               # 0.0-1.0 relationship strength
    description: <string>         # Optional: relationship description
    bidirectional: <boolean>      # Optional: is symmetric (for related)

metadata:
  created_by: <string>
  purpose: <string>               # Why this graph exists
```

## Examples

### Authentication Ontology

```yaml
version: "1"
kind: concept-graph

graph:
  id: graph_authentication
  name: "Authentication Concept Graph"
  description: "Authentication and authorization concepts"

concepts:
  # Core domain
  - id: cpt_authentication
    key: authentication
    name: "Authentication"
    description: "User identity verification"
    aliases: [auth, login, signin]
    status: active
    source: manual

  - id: cpt_authorization
    key: authorization
    name: "Authorization"
    description: "Access control and permissions"
    aliases: [authz, permissions]
    status: active
    source: manual

  # Technologies
  - id: cpt_oauth
    key: oauth
    name: "OAuth 2.0"
    description: "Delegated authorization protocol"
    aliases: [oauth2, oauth-2.0]
    status: active
    source: manual

  - id: cpt_jwt
    key: jwt
    name: "JSON Web Tokens"
    description: "Stateless authentication tokens"
    aliases: [jwt-token, bearer-token]
    status: active
    source: manual

  - id: cpt_saml
    key: saml
    name: "SAML"
    description: "Security Assertion Markup Language"
    aliases: [saml2]
    status: active
    source: manual

relationships:
  # Hierarchies
  - from: concept:@cpt_oauth
    to: concept:@cpt_authentication
    kind: parent
    weight: 1.0
    description: "OAuth is a type of authentication"

  - from: concept:@cpt_jwt
    to: concept:@cpt_authentication
    kind: parent
    weight: 1.0

  - from: concept:@cpt_saml
    to: concept:@cpt_authentication
    kind: parent
    weight: 1.0

  # Related concepts
  - from: concept:@cpt_authentication
    to: concept:@cpt_authorization
    kind: related
    weight: 0.9
    bidirectional: true
    description: "Authentication and authorization are closely related"

  # Implementation
  - from: concept:@cpt_jwt
    to: concept:@cpt_oauth
    kind: implements
    weight: 0.8
    description: "JWT commonly used with OAuth"

metadata:
  created_by: platform-team
  purpose: "Authentication knowledge base"
```

### Payment Processing Graph

```yaml
version: "1"
kind: concept-graph

graph:
  name: "Payment Processing"

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

  - id: cpt_tokenization
    key: tokenization
    name: "Card Tokenization"
    status: active
    source: manual

  - id: cpt_3ds
    key: 3d-secure
    name: "3D Secure"
    aliases: [3ds, three-d-secure]
    status: active
    source: manual

relationships:
  # Dependencies
  - from: concept:@cpt_payment_processing
    to: concept:@cpt_pci_compliance
    kind: depends_on
    weight: 1.0

  - from: concept:@cpt_payment_processing
    to: concept:@cpt_tokenization
    kind: depends_on
    weight: 0.9

  # Related
  - from: concept:@cpt_fraud_detection
    to: concept:@cpt_payment_processing
    kind: related
    weight: 0.9

  - from: concept:@cpt_3ds
    to: concept:@cpt_fraud_detection
    kind: related
    weight: 0.7

  # Implementations
  - from: concept:@cpt_tokenization
    to: concept:@cpt_pci_compliance
    kind: implements
    weight: 0.8
```

## Graph Traversal

### Finding Related Concepts

```typescript
// Find all concepts related to authentication
const auth = graph.concepts.find(c => c.key === 'authentication');
const related = graph.relationships
  .filter(r => r.from === auth.id || r.to === auth.id)
  .map(r => r.from === auth.id ? r.to : r.from);
```

### Building Hierarchy

```python
# Build parent-child tree
def build_hierarchy(concept_key):
    concept = find_concept(concept_key)
    children = [
        r.from for r in relationships
        if r.to == concept.id and r.kind == 'parent'
    ]
    return {
        'concept': concept,
        'children': [build_hierarchy(c.key) for c in children]
    }
```

### Dependency Chain

```typescript
// Find all dependencies
function getDependencies(concept) {
  const deps = relationships
    .filter(r => r.from === concept.id && r.kind === 'depends_on')
    .map(r => findConcept(r.to));

  const transitive = deps.flatMap(getDependencies);
  return [...deps, ...transitive];
}
```

## Graph Visualization

### Mermaid Diagram

```yaml
graph TD
  auth[Authentication] --> oauth[OAuth]
  auth --> jwt[JWT]
  auth --> saml[SAML]
  jwt -.implements.-> oauth

  payments[Payment Processing] -.depends.-> auth
  payments -.depends.-> pci[PCI Compliance]
  fraud[Fraud Detection] -.related.-> payments
```

### D3 Force-Directed Graph

```json
{
  "nodes": [
    {"id": "cpt_auth", "name": "Authentication", "group": "core"},
    {"id": "cpt_oauth", "name": "OAuth", "group": "tech"},
    {"id": "cpt_jwt", "name": "JWT", "group": "tech"}
  ],
  "links": [
    {"source": "cpt_oauth", "target": "cpt_auth", "type": "parent", "weight": 1.0},
    {"source": "cpt_jwt", "target": "cpt_auth", "type": "parent", "weight": 1.0}
  ]
}
```

## Subgraphs

Extract subgraphs by filtering:

```yaml
# Security subgraph
subgraph:
  concepts:
    - filter: {tags: [security]}
  relationships:
    - filter: {kind: [depends_on, related]}
```

## Validation

- All relationship `from` and `to` MUST reference concepts in graph
- No self-references (from == to)
- Relationship kind MUST be from taxonomy
- Weight MUST be 0.0-1.0
- Bidirectional relationships MUST be symmetric (both directions exist)

## See Also

- [Concept Definition](./concept-definition.md) - Concept format
- [Relationship Types](./relationship-types.md) - Relationship taxonomy
- [Coordinate System](../coordinates/COORDINATE_SPEC.md) - Reference format
