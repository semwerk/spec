# Segment Taxonomy

**Version:** 1.0.0
**Status:** Draft

## Overview

Standard vocabulary for classifying documentation segments. Based on the Diataxis framework with extensions for enterprise documentation needs.

## Segment Types

### Learning-Oriented

**tutorial**
- Step-by-step lessons for beginners
- Guides learner through complete project or task
- Learning by doing
- Example: "Getting Started with Payments API"

**example**
- Standalone code examples
- Demonstrates specific use case
- Copy-paste friendly
- Example: "Example: Handling Webhooks"

### Task-Oriented

**howto**
- Practical steps for specific goal
- Assumes existing knowledge
- Problem-solving focused
- Example: "How to Configure OAuth"

**troubleshooting**
- Diagnose and fix specific problems
- Symptom → solution format
- Error message references
- Example: "Troubleshooting Connection Errors"

### Information-Oriented

**reference**
- Technical descriptions (APIs, parameters, functions)
- Comprehensive and accurate
- Structured for lookup
- Example: "API Reference: Users Endpoint"

**api**
- API endpoint documentation
- Request/response schemas
- Authentication requirements
- Example: "POST /v1/users"

**faq**
- Frequently asked questions
- Question → answer format
- Common concerns addressed
- Example: "FAQ: Billing and Usage"

### Understanding-Oriented

**overview**
- High-level introduction
- Big picture context
- System architecture
- Example: "System Overview"

**conceptual**
- Explains concepts and ideas
- Background and theory
- Why things work the way they do
- Example: "Understanding JWT Tokens"

### Planning & Decision

**rfc**
- Request for Comments
- Design proposals
- Architectural decisions
- Example: "RFC-001: Migration Strategy"

**requirements**
- Requirements specifications
- Product requirements (PRD)
- Functional specifications
- Example: "Requirements: Payment Processing"

**policy**
- Organizational policies
- Guidelines and standards
- Governance documentation
- Example: "Security Policy"

### Collaboration

**meeting**
- Meeting notes
- Decision records
- Action items
- Example: "Sprint Planning - Jan 2026"

**review**
- Code reviews
- Design reviews
- Post-mortems
- Example: "Architecture Review: Auth Service"

**roadmap**
- Future plans
- Feature roadmaps
- Strategic direction
- Example: "Q1 2026 Roadmap"

## Usage in Frontmatter

```yaml
---
semcontext:
  segments:
    - id: getting-started
      type: tutorial        # Learning-oriented

    - id: api-reference
      type: reference       # Information-oriented

    - id: troubleshooting
      type: troubleshooting # Task-oriented
---
```

## Usage in Segment Markers

```markdown
<!--semcontext:segment start key="overview" type="overview"-->
## System Overview
...
<!--semcontext:segment end-->

<!--semcontext:segment start key="quickstart" type="tutorial"-->
## Quick Start
...
<!--semcontext:segment end-->
```

## Classification Guidelines

| Type | Answers | Voice | Structure |
|------|---------|-------|-----------|
| tutorial | "Can you teach me to..." | Imperative | Sequential steps |
| howto | "How do I..." | Imperative | Goal-oriented |
| reference | "What is..." | Descriptive | Structured lookup |
| conceptual | "Why..." | Explanatory | Narrative |
| troubleshooting | "How do I fix..." | Diagnostic | Problem → solution |

## Validation

- Segment type MUST be from this taxonomy
- Type determines expected content structure
- Tools can validate appropriate content for each type

## Extensions

Projects can add custom types by prefixing with organization:

```yaml
type: "acme:internal-memo"
```

Standard types (listed above) have no prefix.

## See Also

- [Frontmatter](../formats/frontmatter.md) - How to use types in frontmatter
- [Segment Markers](../formats/segment-markers.md) - How to use types in markers
