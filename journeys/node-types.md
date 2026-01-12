# Journey Node Types

**Version:** 1.0.0
**Status:** Draft

## Overview

Four node types represent different elements in user journey flows.

## Node Types

### stage

**Description:** Standard journey step or action

**Purpose:**
- User performs an action
- System state changes
- UI screen or workflow step
- Task completion

**Characteristics:**
- Has predecessors and successors
- Linear or branching
- May have associated code and documentation
- Measurable (time, completion rate)

**Examples:**
- "Sign Up"
- "Configure API Keys"
- "Deploy to Production"
- "Submit Payment"

**YAML:**
```yaml
- id: node-setup
  type: stage
  key: setup-environment
  name: "Set Up Development Environment"
  description: "Install dependencies and configure local environment"

  code_refs:
    - code:@cfn_setup@v2.0.0/InitializeEnv

  content_refs:
    - content:@doc-quickstart@v2.0.0/setup.environment

  connections:
    - target_node_id: node-first-call
      label: "Continue"
```

---

### milestone

**Description:** Checkpoint or achievement marker

**Purpose:**
- Mark progress
- Celebrate achievement
- Unlock next phase
- Measure funnel conversion

**Characteristics:**
- Usually has single inbound connection
- May have milestone type classification
- Triggers events/notifications
- Key metric measurement point

**Examples:**
- "Email Verified"
- "First API Call Success"
- "Production Deployed"
- "100 Transactions Processed"

**Milestone Types:**
- `onboarding` - User onboarded
- `activation` - Product activated
- `conversion` - Converted to paid
- `retention` - Retained user
- `expansion` - Upsell/expansion

**YAML:**
```yaml
- id: node-email-verified
  type: milestone
  key: email-verified
  name: "Email Verified"
  description: "User has verified their email address"

  milestone_type: onboarding

  metrics:
    conversion_target: 0.95
    median_time_minutes: 2

  connections:
    - target_node_id: node-setup-env
```

---

### decision

**Description:** Branching point with conditional routing

**Purpose:**
- Route based on user attributes
- Handle different user paths
- A/B testing branches
- Feature flag routing

**Characteristics:**
- Multiple outbound connections
- Each connection has condition
- No direct user interaction
- System evaluates condition

**Examples:**
- "Check Experience Level" (beginner vs. advanced)
- "User Role Check" (admin vs. developer vs. user)
- "Feature Flag" (new UI vs. old UI)
- "Payment Method" (card vs. bank transfer)

**YAML:**
```yaml
- id: node-route-by-role
  type: decision
  key: role-routing
  name: "Route by User Role"
  description: "Different paths for different roles"

  connections:
    - target_node_id: node-admin-dashboard
      label: "Admin"
      condition: "user_role == admin"

    - target_node_id: node-developer-portal
      label: "Developer"
      condition: "user_role == developer"

    - target_node_id: node-user-app
      label: "End User"
      condition: "user_role == user"
```

---

### jump_off

**Description:** Link to another journey (cross-journey navigation)

**Purpose:**
- Connect related journeys
- Enable journey composition
- Reduce duplication
- Modular journey design

**Characteristics:**
- Has target_journey in connection
- May return to original journey
- Represents journey boundary
- Enables journey reuse

**Examples:**
- "Advanced Features Journey"
- "Troubleshooting Journey"
- "Upgrade Journey"
- "Onboarding Sub-journey"

**YAML:**
```yaml
- id: node-advanced
  type: jump_off
  key: jump-to-advanced
  name: "Explore Advanced Features"
  description: "Deep dive into advanced capabilities"

  connections:
    - target_journey: journey:@jrn_advanced_features@v2.0.0
      label: "Continue to Advanced"

    - target_journey: journey:@jrn_troubleshooting
      label: "Need Help?"
```

## Node Type Selection Guide

| Scenario | Node Type | Rationale |
|----------|-----------|-----------|
| User fills out form | `stage` | Active user action |
| Email confirmation received | `milestone` | Achievement checkpoint |
| Different paths for roles | `decision` | Conditional routing |
| Link to another journey | `jump_off` | Cross-journey navigation |
| User clicks button | `stage` | User action |
| System processes payment | `stage` | System action |
| Payment succeeded | `milestone` | Success marker |
| Free vs. paid user | `decision` | Conditional branch |

## Validation Rules

**stage:**
- MUST have at least one connection (can't be dead-end)
- CAN have code_refs, content_refs, asset_refs
- CAN have multiple outbound connections

**milestone:**
- SHOULD have milestone_type
- SHOULD have metrics
- TYPICALLY has single inbound, single outbound

**decision:**
- MUST have 2+ outbound connections
- Each connection SHOULD have condition
- SHOULD NOT have user-facing content (system routing only)

**jump_off:**
- MUST have target_journey in at least one connection
- CAN have multiple target journeys
- SHOULD explain jump-off purpose in description

## Graph Constraints

### Acyclic Requirement

Journeys MUST be directed acyclic graphs (DAGs):

```yaml
# ✓ Valid (linear)
A → B → C

# ✓ Valid (branching)
A → B → D
    ↓
    C → D

# ✗ Invalid (cycle)
A → B → C → A
```

### Entry/Exit Nodes

- At least one node MUST be reachable from entry_points
- At least one node MUST appear in exit_points
- No orphaned nodes (unreachable from any entry)

## See Also

- [Journey Graph](./journey-graph.md) - Journey format
- [Entry/Exit Points](./entry-exit-points.md) - Journey boundaries
- [Success Metrics](./success-metrics.md) - Measurement
