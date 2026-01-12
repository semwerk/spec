# Entry and Exit Points

**Version:** 1.0.0
**Status:** Draft

## Overview

Entry and exit points define journey boundaries - how users enter and complete journeys.

## Entry Points

Entry points describe how users arrive at a journey.

### Format

```yaml
entry_points:
  - type: <enum>                  # Entry type
    description: <string>         # How user enters
    url: <string>                 # Optional: entry URL/path
    content_ref: <content-coord>  # Optional: entry content
    asset_ref: <asset-coord>      # Optional: entry asset
    metadata: <object>            # Optional: additional data
```

### Entry Types

**documentation**
- User lands on documentation page
- Clicks "Get Started" CTA
- Follows integration guide

**marketing**
- Landing page conversion
- Marketing campaign
- Product website

**referral**
- Invited by existing user
- Team member shared link
- Partner integration

**direct**
- Direct sign-up
- App download
- CLI installation

**support**
- Support ticket resolution
- Community forum
- Help desk

**search**
- Google search
- Documentation search
- Stack Overflow

**trial**
- Free trial signup
- Demo request
- Evaluation period

### Examples

**Documentation entry:**
```yaml
entry_points:
  - type: documentation
    description: "Landing on Getting Started guide"
    url: /docs/getting-started
    content_ref: content:@doc-quickstart@v2.0.0/overview
```

**Marketing entry:**
```yaml
entry_points:
  - type: marketing
    description: "Developer landing page CTA"
    url: /developers
    content_ref: content:@doc-marketing/developer-cta
    metadata:
      campaign: "Q1-2024-dev-outreach"
```

**Referral entry:**
```yaml
entry_points:
  - type: referral
    description: "Invited by team member"
    metadata:
      referral_source: "team-invite"
```

**Multiple entry points:**
```yaml
entry_points:
  - type: documentation
    url: /docs/quickstart

  - type: marketing
    url: /developers

  - type: trial
    description: "Free trial signup"
```

## Exit Points

Exit points describe journey completion or abandonment.

### Format

```yaml
exit_points:
  - node: <string>                # Node ID where exit occurs
    type: <enum>                  # Exit type
    description: <string>         # What constitutes exit
    target_journey: <journey-ref> # Optional: next journey
    metadata: <object>            # Optional: additional data
```

### Exit Types

**success**
- User completed journey successfully
- Achieved journey goal
- All steps finished

**failure**
- User could not complete
- Blocked by errors
- Technical issues

**abandonment**
- User gave up
- Lost interest
- Switched to competitor

**conversion**
- Free to paid conversion
- Trial to subscription
- Upgrade to higher tier

**offboarding**
- User left platform
- Account closed
- Churned

**continuation**
- Continues to another journey
- Graduates to advanced journey
- Enters related workflow

### Examples

**Success exit:**
```yaml
exit_points:
  - node: first-api-call
    type: success
    description: "Successfully made first API call"
```

**Failure exit:**
```yaml
exit_points:
  - node: payment-setup
    type: failure
    description: "Payment method validation failed"
    metadata:
      common_errors:
        - "invalid_card"
        - "insufficient_funds"
```

**Conversion exit:**
```yaml
exit_points:
  - node: upgrade-prompt
    type: conversion
    description: "Upgraded from free to paid plan"
    target_journey: journey:@jrn_paid_onboarding
```

**Multiple exit points:**
```yaml
exit_points:
  - node: production-deploy
    type: success
    description: "Deployed to production"

  - node: troubleshooting
    type: failure
    description: "Encountered deployment error"

  - node: dashboard-setup
    type: continuation
    description: "Continue to dashboard setup"
    target_journey: journey:@jrn_dashboard_config
```

## Entry-Exit Pairs

Journeys can chain:

```yaml
# Journey A exits to Journey B
journey:
  id: jrn_onboarding
  exit_points:
    - node: final-step
      type: continuation
      target_journey: journey:@jrn_advanced_features

# Journey B entry from Journey A
journey:
  id: jrn_advanced_features
  entry_points:
    - type: direct
      description: "Graduated from onboarding"
```

## Validation

- Entry points SHOULD have at least one
- Exit points SHOULD have at least one
- Exit point node MUST exist in nodes array
- If `type: continuation`, target_journey SHOULD be specified
- Entry and exit types MUST be from taxonomies

## Use Cases

### 1. Funnel Analysis

```typescript
// Track entry source conversion
const docEntries = getEntriesByType('documentation');
const successExits = getExitsByType('success');
const conversionRate = successExits.length / docEntries.length;
```

### 2. Drop-off Analysis

```python
# Find where users abandon
exit_points = [e for e in exits if e.type == 'abandonment']
for exit in exit_points:
    print(f"Drop-off at: {exit.node} - {exit.description}")
```

### 3. Journey Chaining

```yaml
# Map journey flow
onboarding → advanced → mastery → advocate
   (success)   (continuation)  (continuation)  (completion)
```

## See Also

- [Journey Graph](./journey-graph.md) - Journey format
- [Node Types](./node-types.md) - Node taxonomy
- [Success Metrics](./success-metrics.md) - Measurement
