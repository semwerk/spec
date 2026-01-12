# Persona Definition

**Version:** 1.0.0
**Status:** Draft
**Format:** YAML

## Overview

Personas represent target user archetypes. Journeys target specific personas, and content can be filtered by persona.

## Format

```yaml
version: "1"
kind: persona

persona:
  id: <string>                    # Unique ID (per_*)
  key: <string>                   # Kebab-case key
  name: <string>                  # Display name
  description: <string>           # Persona description

  # Demographics
  role: <string>                  # Primary role (developer, operator, etc.)
  experience_level: <enum>        # beginner | intermediate | advanced | expert

  # Goals
  goals: [<string>]               # Primary goals
  pain_points: [<string>]         # Current challenges

  # Behavior
  preferred_channels: [<string>]  # documentation | support | community | sales
  tech_stack: [<string>]          # Technologies they use

  # Relationships
  projects: [<project-ref>]       # Associated projects

metadata:
  created_by: <string>
  tags: [<string>]
```

## Examples

### Developer Persona

```yaml
version: "1"
kind: persona

persona:
  id: per_backend_dev
  key: backend-developer
  name: "Backend Developer"
  description: "Experienced backend engineer building APIs"

  role: developer
  experience_level: intermediate

  goals:
    - "Integrate payment API quickly"
    - "Ensure PCI compliance"
    - "Handle high transaction volume"

  pain_points:
    - "Complex authentication setup"
    - "Unclear error messages"
    - "Poor webhook documentation"

  preferred_channels:
    - documentation
    - community
    - support

  tech_stack:
    - Go
    - Python
    - PostgreSQL
    - Redis
    - Docker

  projects:
    - project:@prj_payment_api

metadata:
  created_by: product-team
  tags:
    - engineering
    - backend
```

### End User Persona

```yaml
version: "1"
kind: persona

persona:
  id: per_end_user
  key: end-user
  name: "End User"
  description: "Non-technical user of the application"

  role: user
  experience_level: beginner

  goals:
    - "Complete task quickly"
    - "Understand what went wrong"
    - "Get help when stuck"

  pain_points:
    - "Technical jargon confusing"
    - "Don't know where to start"
    - "Error messages unclear"

  preferred_channels:
    - support
    - documentation

  tech_stack: []

metadata:
  tags:
    - end-user
    - customer
```

## Persona in Journeys

Journeys target specific personas:

```yaml
journey:
  key: developer-onboarding
  personas:
    - persona:@per_backend_dev
    - persona:@per_frontend_dev
```

## Persona in Content

Content can be tagged for personas:

```yaml
content_refs:
  - content:@doc-quickstart@v2.0.0/getting-started
    target_personas:
      - persona:@per_backend_dev
```

## Coordinates

```yaml
persona:@per_backend_dev                        # Persona reference
```

## See Also

- [Journey Graph](../journeys/journey-graph.md) - Using personas in journeys
- [Audience Roles](../taxonomies/audience-roles.md) - Role taxonomy
