# Classification Config Schema

**Version:** 1.0.0
**Status:** Draft
**Format:** YAML

## Overview

Defines customizable rules for automatic classification of documentation segments and audience targeting. Enables:

- Keyword-based type detection
- Audience inference from content
- Project-specific concept taxonomies
- LLM-based classification (optional)

## Location

`.werkcontext/classifier.yaml` or use system defaults

## Schema

```yaml
segment_types:
  <type_name>:
    keywords: [<string>]        # Keywords to match in headings/content
    prefixes: [<string>]        # Heading prefixes that indicate this type

audience_roles:
  <role_name>:
    keywords: [<string>]        # Keywords indicating this audience
    linked_types: [<string>]    # Segment types commonly for this audience

content_indicators:
  business: [<string>]          # Keywords indicating business content
  technical: [<string>]         # Keywords indicating technical content

concepts:
  domains: [<string>]           # Domain concepts
  technologies: [<string>]      # Technology concepts
  architecture: [<string>]      # Architecture patterns
  documentation: [<string>]     # Documentation concepts

inference:
  llm:
    enabled: <boolean>
    server_addr: <string>
    confidence_threshold: <float>
  embeddings:
    enabled: <boolean>
    similarity_threshold: <float>
```

## Examples

### Minimal Config

```yaml
segment_types:
  tutorial:
    keywords: [tutorial, getting started, guide]
    prefixes: []

  reference:
    keywords: [reference, api, configuration]
    prefixes: []

audience_roles:
  developer:
    keywords: [api, code, sdk]
    linked_types: [reference]

  user:
    keywords: [guide, tutorial]
    linked_types: [tutorial]
```

### Full Config

```yaml
segment_types:
  requirements:
    keywords:
      - requirements
      - specification
      - prd
      - functional spec
    prefixes: []

  rfc:
    keywords:
      - rfc
      - design doc
      - proposal
    prefixes:
      - "rfc "
      - "rfc-"

  api:
    keywords:
      - api
      - endpoint
      - rest
      - graphql
    prefixes: []

audience_roles:
  developer:
    keywords:
      - api
      - sdk
      - code
      - integration
    linked_types:
      - api
      - reference
      - example

  operator:
    keywords:
      - deployment
      - installation
      - monitoring
    linked_types:
      - reference
      - howto

content_indicators:
  business:
    - stakeholder
    - budget
    - revenue
    - roi
    - strategy

  technical:
    - function
    - database
    - api
    - endpoint
    - parameter

concepts:
  domains:
    - authentication
    - authorization
    - billing
    - notifications

  technologies:
    - rest
    - graphql
    - postgres
    - redis
    - docker

inference:
  llm:
    enabled: true
    server_addr: "localhost:50051"
    model: "default"
    confidence_threshold: 0.7
    fallback_to_rules: true

  embeddings:
    enabled: false
    similarity_threshold: 0.8
```

### Custom Project Config

```yaml
# .werkcontext/classifier.yaml
# Extends system defaults with project-specific rules

segment_types:
  # Add custom type
  architecture-decision:
    keywords:
      - architecture decision
      - adr
      - design decision
    prefixes:
      - "adr-"

audience_roles:
  # Add custom role
  field-engineer:
    keywords:
      - field
      - customer deployment
      - on-site
    linked_types:
      - howto
      - troubleshooting

concepts:
  # Project-specific concepts
  domains:
    - payment-processing
    - fraud-detection
    - kyc-verification
    - transaction-monitoring

  custom:
    - pci-compliance
    - anti-money-laundering
    - customer-onboarding
```

## Classification Algorithm

### 1. Heading-Based Detection

```python
# Check heading against prefixes
for type_name, config in segment_types.items():
    for prefix in config.prefixes:
        if heading.lower().startswith(prefix):
            return type_name, confidence=1.0  # Prefix match is definitive

# Check heading against keywords
for type_name, config in segment_types.items():
    for keyword in config.keywords:
        if keyword in heading.lower():
            return type_name, confidence=0.8  # Keyword match
```

### 2. Content-Based Detection

```python
# Analyze content indicators
business_score = count_matches(content, content_indicators.business)
technical_score = count_matches(content, content_indicators.technical)

if business_score > technical_score:
    # Prefer business-oriented types and roles
    filter_types = ['requirements', 'roadmap', 'policy']
    filter_roles = ['product', 'executive']
```

### 3. LLM-Based Inference (Optional)

```python
if inference.llm.enabled:
    result = llm_classify(content, server_addr, model)
    if result.confidence >= confidence_threshold:
        return result.type, result.confidence
    elif fallback_to_rules:
        # Fall back to keyword-based detection
```

## Use Cases

### 1. Automatic Classification

```typescript
// Load config
const config = loadClassifierConfig('.werkcontext/classifier.yaml');

// Classify heading
const heading = "## How to Deploy";
const type = classifyHeading(heading, config);
// Returns: "howto" (matched prefix "how to ")
```

### 2. Multi-Tenant Customization

```yaml
# Healthcare SaaS
concepts:
  domains:
    - patient-records
    - hipaa-compliance
    - ehr-integration

# E-commerce SaaS
concepts:
  domains:
    - inventory-management
    - order-processing
    - payment-gateway
```

### 3. Audience Detection

```python
# Detect audience from content
content = "This API endpoint accepts a JSON payload..."
audience = infer_audience(content, config)
# Returns: "developer" (matched keywords: api, json)
```

### 4. Concept Extraction

```javascript
// Extract concepts from segment
const concepts = extractConcepts(segment.content, config.concepts);
// Returns: ['authentication', 'jwt', 'oauth']
```

## Validation

- Segment types MUST reference valid types from [Segment Taxonomy](../taxonomies/segments.md)
- Audience roles MUST reference valid roles from [Audience Roles](../taxonomies/audience-roles.md)
- Keywords MUST be lowercase strings
- Confidence threshold MUST be 0.0-1.0

## Precedence

1. **Frontmatter** (asserted) - highest priority
2. **Segment markers** (asserted) - high priority
3. **LLM inference** (inferred) - medium priority
4. **Keyword matching** (inferred) - low priority
5. **Default** - fallback

## See Also

- [Segment Taxonomy](../taxonomies/segments.md) - Valid segment types
- [Audience Roles](../taxonomies/audience-roles.md) - Valid audience roles
- [Provenance Model](./provenance.md) - Tracking inferred vs. asserted
