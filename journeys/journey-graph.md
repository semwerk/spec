# Journey Graph Format

**Version:** 1.0.0
**Status:** Draft
**Format:** YAML

## Overview

User journeys are represented as directed acyclic graphs (DAGs) with nodes and connections. Journeys can span multiple projects and be version-specific.

## Format

```yaml
version: "2"
kind: journey

journey:
  id: <string>                    # Unique journey ID (jrn_*)
  key: <string>                   # Kebab-case key (unique within tenant)
  name: <string>                  # Display name
  description: <string>           # Journey description

  # Scope
  projects: [<project-ref>]       # Projects involved (multi-project support)
  primary_project: <project-ref>  # Primary owning project
  personas: [<string>]            # Target user personas

  # Versioning
  version_specific: <boolean>     # Is this journey version-scoped?
  versions: [<version-ref>]       # Applicable versions (if version_specific)

  # Lifecycle
  status: <enum>                  # draft | active | deprecated

  # Graph structure
  nodes: [<node>]                 # Journey nodes (stages, milestones, decisions)

  # Boundaries
  entry_points: [<entry>]         # How users enter this journey
  exit_points: [<exit>]           # How users complete this journey

  # Measurement
  success_metrics: [<metric>]     # KPIs for this journey

metadata:
  owner: <string>
  last_reviewed: <date>
  tags: [<string>]
```

## Node Structure

```yaml
nodes:
  - id: <string>                  # Unique node ID within journey
    type: <enum>                  # stage | milestone | decision | jump_off
    key: <string>                 # Kebab-case key
    name: <string>                # Display name
    description: <string>         # Node description

    # Visual layout
    position:
      x: <int>                    # X coordinate for graph rendering
      y: <int>                    # Y coordinate

    # Context
    project: <project-ref>        # Associated project
    features: [<string>]          # Feature flags/toggles

    # References (using coordinate system)
    code_refs: [<code-coord>]     # Related code
    content_refs: [<content-coord>]  # Related docs
    asset_refs: [<asset-coord>]   # Related assets
    concepts: [<concept-ref>]     # Semantic concepts

    # Graph edges
    connections: [<connection>]   # Outgoing edges

    # Metadata
    tags: [<string>]
    metrics: <object>             # Node-specific metrics
```

## Node Types

See [Node Types](./node-types.md) for full taxonomy.

### stage

Standard journey step (UI screen, workflow action, task)

```yaml
- id: node-setup
  type: stage
  key: setup-environment
  name: "Set Up Development Environment"
  connections:
    - target_node_id: node-first-call
```

### milestone

Checkpoint or achievement marker

```yaml
- id: node-email-verified
  type: milestone
  key: email-verified
  name: "Email Verified"
  milestone_type: activation
```

### decision

Branching point with conditional routing

```yaml
- id: node-check-experience
  type: decision
  key: experience-check
  name: "Check Developer Experience"
  connections:
    - target_node_id: node-beginner
      label: "Beginner"
      condition: "experience_level == beginner"
    - target_node_id: node-advanced
      label: "Advanced"
      condition: "experience_level == advanced"
```

### jump_off

Link to another journey (cross-journey navigation)

```yaml
- id: node-advanced-features
  type: jump_off
  key: jump-to-advanced
  name: "Explore Advanced Features"
  connections:
    - target_journey: journey:@jrn_advanced_features
      label: "Continue to Advanced"
```

## Connections

```yaml
connections:
  - target_node_id: <string>      # Target node within same journey
    label: <string>               # Edge label
    condition: <string>           # Optional: conditional expression

  - target_journey: <journey-ref>  # For jump_off nodes
    label: <string>
```

## Entry Points

```yaml
entry_points:
  - type: <enum>                  # documentation | marketing | referral | direct | support
    description: <string>
    url: <string>                 # Optional: entry URL
    content_ref: <content-coord>  # Optional: entry content
```

## Exit Points

```yaml
exit_points:
  - node: <string>                # Node ID where journey exits
    type: <enum>                  # success | failure | abandonment
    description: <string>
```

## Success Metrics

```yaml
success_metrics:
  - metric: <string>              # Metric name (e.g., time_to_first_call)
    target: <string>              # Target value (e.g., "< 30 minutes")
    description: <string>
```

## Examples

### Simple Linear Journey

```yaml
version: "2"
kind: journey

journey:
  id: jrn_quickstart
  key: quickstart
  name: "Quick Start Journey"
  description: "Get started in 5 minutes"

  projects:
    - project:@prj_payment_api

  personas:
    - developer

  status: active

  nodes:
    - id: signup
      type: stage
      key: signup
      name: "Sign Up"
      position: {x: 100, y: 50}
      connections:
        - target_node_id: verify-email

    - id: verify-email
      type: milestone
      key: email-verified
      name: "Email Verified"
      position: {x: 100, y: 150}
      connections:
        - target_node_id: first-call

    - id: first-call
      type: stage
      key: first-api-call
      name: "Make First API Call"
      position: {x: 100, y: 250}
      code_refs:
        - code:@cfn_create_charge@v2.0.0/CreateCharge
      content_refs:
        - content:@doc-quickstart@v2.0.0/first-call

  entry_points:
    - type: documentation
      url: /docs/quickstart

  exit_points:
    - node: first-call
      type: success

  success_metrics:
    - metric: time_to_first_call
      target: "< 30 minutes"
    - metric: completion_rate
      target: "> 70%"
```

### Multi-Project Journey with Branching

```yaml
version: "2"
kind: journey

journey:
  id: jrn_payment_integration
  key: payment-integration
  name: "Payment Integration Journey"

  projects:
    - project:@prj_payment_api
    - project:@prj_payment_sdk
    - project:@prj_dashboard

  primary_project: project:@prj_payment_api

  version_specific: true
  versions:
    - version:@v2.0.0
    - version:@v2.1.0

  status: active

  nodes:
    - id: choose-integration
      type: decision
      key: integration-method
      name: "Choose Integration Method"
      position: {x: 200, y: 50}
      connections:
        - target_node_id: api-integration
          label: "Direct API"
          condition: "integration_type == api"
        - target_node_id: sdk-integration
          label: "Use SDK"
          condition: "integration_type == sdk"

    - id: api-integration
      type: stage
      key: api-setup
      name: "API Integration"
      position: {x: 100, y: 200}
      project: project:@prj_payment_api
      code_refs:
        - code:@cfn_api_client@v2.0.0/CreatePayment
      content_refs:
        - content:@doc-api@v2.0.0/integration.direct-api
      connections:
        - target_node_id: test-payment

    - id: sdk-integration
      type: stage
      key: sdk-setup
      name: "SDK Integration"
      position: {x: 300, y: 200}
      project: project:@prj_payment_sdk
      code_refs:
        - code:@cfn_sdk_client@v2.0.0/Initialize
      content_refs:
        - content:@doc-sdk@v2.0.0/integration.sdk-setup
      connections:
        - target_node_id: test-payment

    - id: test-payment
      type: milestone
      key: first-payment
      name: "First Test Payment"
      position: {x: 200, y: 350}
      connections:
        - target_node_id: go-live

    - id: go-live
      type: stage
      key: production-deploy
      name: "Deploy to Production"
      position: {x: 200, y: 500}
```

### Version-Specific Journey

```yaml
version: "2"
kind: journey

journey:
  id: jrn_auth_v2
  key: authentication-v2
  name: "Authentication (v2.0 Only)"
  description: "New OAuth flow introduced in v2.0"

  projects:
    - project:@prj_auth_service

  version_specific: true
  versions:
    - version:@v2.0.0
    - version:@v2.1.0
  # NOT applicable to v1.x

  status: active

  nodes:
    - id: oauth-setup
      type: stage
      key: oauth-configuration
      name: "Configure OAuth"
      code_refs:
        - code:@cfn_oauth_handler@v2.0.0/AuthorizeUser
      content_refs:
        - content:@doc-auth@v2.0.0/oauth.setup
      connections:
        - target_node_id: obtain-token

    - id: obtain-token
      type: stage
      key: token-exchange
      name: "Exchange Code for Token"
      code_refs:
        - code:@cfn_token_exchange@v2.0.0/ExchangeCode
      content_refs:
        - content:@doc-auth@v2.0.0/oauth.token-exchange
```

## Coordinates

Journeys can be referenced:

```yaml
journey:@jrn_onboarding                         # Basic reference
journey:@jrn_onboarding@v2.0.0                  # At specific version
journey:@jrn_onboarding/node-setup              # Specific node
journey:@jrn_onboarding@v2.0.0/node-setup       # Version + node
```

## Validation

- Journey ID MUST start with `jrn_`
- Key MUST be kebab-case, unique within tenant
- Nodes MUST have unique IDs within journey
- Node IDs in connections MUST exist in nodes array
- If `version_specific: true`, versions array MUST not be empty
- If `type: jump_off`, node MUST have target_journey in connections
- Graph MUST be acyclic (no circular references)

## Use Cases

### 1. Product Management

```typescript
// Define journey in product tool
// Export as YAML
// Import into semcontext for tracking
const journey = parseJourney('onboarding.yaml');
visualize(journey);
```

### 2. AI Agent Navigation

```typescript
// Agent understands user context
const journey = parseJourney('.semcontext/journey.yaml');
const currentNode = journey.nodes.find(n => n.key === 'setup');
const nextSteps = getConnectedNodes(currentNode);

agent.suggest(`Next: ${nextSteps[0].name}`);
```

### 3. Analytics Integration

```python
# Import journeys into analytics platform
journey = parse_journey('onboarding.yaml')
for node in journey.nodes:
    track_funnel_step(node.key, node.name)
```

### 4. Version-Filtered Docs

```yaml
# Show only journeys for current version
journeys_v2 = [j for j in journeys if 'v2.0.0' in j.versions]
```

## See Also

- [Node Types](./node-types.md) - Node type taxonomy
- [Entry/Exit Points](./entry-exit-points.md) - Journey boundaries
- [Success Metrics](./success-metrics.md) - Measurement schema
- [Multi-Project Journeys](./multi-project-journeys.md) - Cross-project patterns
- [Version-Specific Journeys](./version-specific-journeys.md) - Version scoping
- [Coordinate System](../coordinates/COORDINATE_SPEC.md) - Reference format
