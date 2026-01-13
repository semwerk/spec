# SpecBook Entities

**Version:** 1.0.0
**Status:** Draft
**Format:** YAML

## Overview

SpecBook entities represent structured knowledge elements that can be referenced by journeys, content, and code. These are the building blocks of product and engineering knowledge.

## Entity Types

### Core Entities

| Entity | ID Prefix | Description |
|--------|-----------|-------------|
| **Task** | `stsk_` | User task or job to be done |
| **Problem** | `sprb_` | User problem or pain point |
| **Solution** | `ssol_` | Solution to a problem |
| **Outcome** | `sout_` | Desired outcome or result |
| **Validation** | `sval_` | Validation criteria or test |
| **NextStep** | `snxt_` | Recommended next action |
| **Impact** | `simp_` | Impact or benefit |
| **Feature** | `sftr_` | Product feature |

## Entity Format

```yaml
version: "1"
kind: specbook-entity

entity:
  id: <string>                    # Unique ID (entity-specific prefix)
  type: <enum>                    # task | problem | solution | outcome | etc.
  slug: <string>                  # URL-safe identifier
  title: <string>                 # Display title
  description: <string>           # Full description

  # Relationships
  related_entities: [<entity-ref>]  # Related entities
  concepts: [<concept-ref>]       # Semantic concepts

  # References
  code_refs: [<code-coord>]       # Related code
  content_refs: [<content-coord>] # Related documentation
  asset_refs: [<asset-coord>]     # Related assets

metadata:
  status: <enum>                  # draft | active | deprecated
  priority: <enum>                # critical | high | medium | low
  tags: [<string>]
```

## Task

User tasks or jobs to be done.

```yaml
version: "1"
kind: specbook-entity

entity:
  id: stsk_onboard_dev
  type: task
  slug: onboard-new-developer
  title: "Onboard New Developer"
  description: |
    Enable a new backend developer to make their first
    API call within 30 minutes

  related_entities:
    - problem:@sprb_slow_onboarding
    - outcome:@sout_fast_time_to_value
    - solution:@ssol_quickstart_guide

  concepts:
    - concept:@cpt_onboarding
    - concept:@cpt_developer_experience

  content_refs:
    - content:@doc-quickstart@v2.0.0/overview

metadata:
  status: active
  priority: high
  tags:
    - onboarding
    - developer
```

## Problem

User problems or pain points.

```yaml
version: "1"
kind: specbook-entity

entity:
  id: sprb_auth_complexity
  type: problem
  slug: authentication-setup-complexity
  title: "Authentication Setup Too Complex"
  description: |
    Developers struggle with OAuth configuration,
    taking 2+ hours on average

  related_entities:
    - solution:@ssol_simplified_auth
    - impact:@simp_reduced_support_tickets
    - task:@stsk_setup_auth

  concepts:
    - concept:@cpt_authentication
    - concept:@cpt_developer_experience

metadata:
  status: active
  priority: critical
  tags:
    - authentication
    - dx
    - pain-point
```

## Solution

Solutions to problems.

```yaml
version: "1"
kind: specbook-entity

entity:
  id: ssol_quickstart_guide
  type: solution
  slug: interactive-quickstart-guide
  title: "Interactive Quick Start Guide"
  description: |
    Step-by-step guided tutorial with code playground
    and immediate feedback

  related_entities:
    - problem:@sprb_slow_onboarding
    - outcome:@sout_fast_onboarding
    - task:@stsk_onboard_dev

  concepts:
    - concept:@cpt_onboarding
    - concept:@cpt_tutorial

  content_refs:
    - content:@doc-quickstart@v2.0.0/overview
    - content:@doc-quickstart@v2.0.0/tutorial

  code_refs:
    - code:@cfn_tutorial_handler@v2.0.0/GeneratePlayground

metadata:
  status: active
  priority: high
  tags:
    - solution
    - tutorial
```

## Outcome

Desired outcomes or results.

```yaml
version: "1"
kind: specbook-entity

entity:
  id: sout_fast_time_to_value
  type: outcome
  slug: fast-time-to-first-call
  title: "< 30 Min Time to First API Call"
  description: "Developer makes first successful API call in under 30 minutes"

  related_entities:
    - task:@stsk_onboard_dev
    - solution:@ssol_quickstart_guide
    - validation:@sval_first_call_metric

  concepts:
    - concept:@cpt_developer_experience
    - concept:@cpt_onboarding

metadata:
  status: active
  priority: high
  tags:
    - outcome
    - metric
```

## Validation

Validation criteria or acceptance tests.

```yaml
version: "1"
kind: specbook-entity

entity:
  id: sval_first_call_success
  type: validation
  slug: validate-first-api-call
  title: "First API Call Returns 200"
  description: "Validation: API call completes successfully with 200 response"

  related_entities:
    - outcome:@sout_fast_time_to_value
    - task:@stsk_make_first_call

  code_refs:
    - code:@cfn_api_test@v2.0.0/TestFirstCall

metadata:
  status: active
  tags:
    - validation
    - test
```

## Impact

Impact or benefit statements.

```yaml
version: "1"
kind: specbook-entity

entity:
  id: simp_reduced_support
  type: impact
  slug: reduced-support-tickets
  title: "50% Reduction in Support Tickets"
  description: "Better docs and onboarding reduce auth-related support by 50%"

  related_entities:
    - solution:@ssol_simplified_auth
    - problem:@sprb_auth_complexity

  concepts:
    - concept:@cpt_developer_experience
    - concept:@cpt_support

metadata:
  status: active
  priority: high
  tags:
    - impact
    - metrics
```

## NextStep

Recommended next actions.

```yaml
version: "1"
kind: specbook-entity

entity:
  id: snxt_explore_webhooks
  type: next_step
  slug: explore-webhooks
  title: "Set Up Webhooks"
  description: "Configure webhooks to receive payment notifications"

  related_entities:
    - task:@stsk_configure_webhooks
    - outcome:@sout_realtime_notifications

  content_refs:
    - content:@doc-webhooks@v2.0.0/overview

metadata:
  status: active
  tags:
    - next-step
    - integration
```

## Feature

Product features.

```yaml
version: "1"
kind: specbook-entity

entity:
  id: sftr_oauth_integration
  type: feature
  slug: oauth-integration
  title: "OAuth 2.0 Integration"
  description: "Full OAuth 2.0 support with PKCE flow"

  related_entities:
    - task:@stsk_implement_oauth
    - outcome:@sout_secure_auth
    - validation:@sval_oauth_compliance

  concepts:
    - concept:@cpt_oauth
    - concept:@cpt_authentication
    - concept:@cpt_security

  code_refs:
    - code:@cfn_oauth_handler@v2.0.0/AuthorizeUser
    - code:@cfn_oauth_handler@v2.0.0/ExchangeToken

  content_refs:
    - content:@doc-auth@v2.0.0/oauth

metadata:
  status: active
  priority: high
  tags:
    - feature
    - authentication
```

## Entity References in Journeys

Journey nodes can reference SpecBook entities:

```yaml
journey:
  nodes:
    - id: setup-auth
      type: stage
      name: "Set Up Authentication"

      # SpecBook entity references
      tasks:
        - task:@stsk_configure_oauth
        - task:@stsk_test_auth

      problems:
        - problem:@sprb_auth_complexity

      solutions:
        - solution:@ssol_simplified_auth

      outcomes:
        - outcome:@sout_secure_auth

      validations:
        - validation:@sval_oauth_compliance

      next_steps:
        - next_step:@snxt_explore_webhooks
```

## Entity Graph

SpecBook entities form a knowledge graph:

```
Task: "Onboard Developer"
  ├─ solves → Problem: "Slow Onboarding"
  ├─ achieves → Outcome: "Fast Time to Value"
  ├─ validated_by → Validation: "First Call Success"
  └─ leads_to → NextStep: "Explore Advanced Features"

Solution: "Quick Start Guide"
  ├─ solves → Problem: "Slow Onboarding"
  ├─ enables → Task: "Onboard Developer"
  ├─ achieves → Outcome: "Fast Onboarding"
  └─ creates → Impact: "Reduced Support Tickets"
```

## Coordinates

```yaml
task:@stsk_abc123                               # Task reference
problem:@sprb_xyz789                            # Problem reference
solution:@ssol_def456                           # Solution reference
outcome:@sout_abc789                            # Outcome reference
validation:@sval_xyz123                         # Validation reference
next_step:@snxt_abc456                          # Next step reference
impact:@simp_xyz789                             # Impact reference
feature:@sftr_abc123                            # Feature reference
```

## Minimal Fields (Must-Have)

For ecosystem adoption, entities only need:

**Required:**
- `id` - Unique identifier
- `type` - Entity type
- `slug` - URL-safe key
- `title` - Display name
- `description` - Full description

**Recommended:**
- `related_entities` - Entity graph
- `concepts` - Semantic tagging
- `content_refs` - Documentation links
- `metadata.status` - Lifecycle state

**Optional (Semcontext-specific):**
- Priority scoring
- Detailed metadata
- Internal tracking fields

## See Also

- [Journey Graph](../journeys/journey-graph.md) - Referencing entities in journeys
- [Concept Definition](./concept.md) - Semantic concepts
- [Coordinate System](../coordinates/COORDINATE_SPEC.md) - Reference format
