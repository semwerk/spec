# Project Types

**Version:** 1.0.0
**Status:** Draft

## Overview

Standard taxonomy for classifying projects. Determines expected structure, lifecycle, and relationships.

## Project Types

### product

**Description:** Customer-facing product or application

**Characteristics:**
- Has end users
- Has product requirements (PRD/MRD)
- Has user journeys
- May have multiple versions in production
- Marketing and sales assets

**Examples:**
- Mobile app
- Web application
- SaaS platform
- Desktop software

**Typical Assets:**
- User documentation
- Marketing site
- Product requirements
- User guides
- Release notes

### service

**Description:** Backend microservice or API

**Characteristics:**
- Part of larger system
- Has API contracts
- Has service-level objectives (SLOs)
- Consumed by other services or products
- Technical documentation focused

**Examples:**
- Payment processing service
- Authentication service
- Notification service
- Data pipeline

**Typical Assets:**
- API reference
- Architecture docs
- Runbooks
- Integration guides
- SLO/SLA docs

### library

**Description:** Reusable library, SDK, or package

**Characteristics:**
- Consumed by other projects
- Has stable API surface
- Versioned releases
- Published to package registry
- Developer-focused documentation

**Examples:**
- npm package
- Go module
- Python package
- Mobile SDK

**Typical Assets:**
- API reference
- Integration guide
- Code examples
- Migration guides
- Changelog

### platform

**Description:** Multi-service platform or ecosystem

**Characteristics:**
- Composed of multiple projects
- Parent to services/products
- Has platform-wide architecture
- Cross-cutting concerns
- Multiple stakeholder types

**Examples:**
- Microservices platform
- Developer platform
- Data platform
- Infrastructure platform

**Typical Assets:**
- Architecture documentation
- Platform overview
- Integration patterns
- Developer portal
- Governance docs

### internal

**Description:** Internal tooling or utilities

**Characteristics:**
- Not customer-facing
- Used by engineering/ops teams
- May have looser versioning
- Focused on internal efficiency
- Minimal external documentation

**Examples:**
- Build tools
- CI/CD pipelines
- Internal dashboards
- Dev utilities
- Deployment scripts

**Typical Assets:**
- Internal runbooks
- Setup guides
- Troubleshooting docs
- Team wiki

## Usage

### In Project Definition

```yaml
project:
  type: service                   # From this taxonomy
  key: payment-api
  name: "Payment API"
```

### Type-Specific Validation

Different types have different requirements:

**Product:**
- SHOULD have user journeys
- SHOULD have marketing assets
- MUST have end-user documentation

**Service:**
- SHOULD have API reference
- SHOULD have runbooks
- MUST have integration guides

**Library:**
- MUST have API reference
- SHOULD have code examples
- MUST have changelog

**Platform:**
- MUST have architecture docs
- SHOULD have multiple child projects
- MUST have integration patterns

**Internal:**
- MAY have minimal documentation
- Focus on operational guides

## Type Hierarchies

```
Platform
├── Product (customer-facing)
├── Service (backend)
└── Library (reusable)

Internal (separate category)
```

## Custom Types

Organizations can extend with custom types:

```yaml
project:
  type: "acme:data-pipeline"      # Custom type with org prefix
```

Standard types (above) have no prefix.

## See Also

- [Project Definition](./project-definition.md) - Project format
- [Version Specification](../versions/version-specification.md) - Versioning
