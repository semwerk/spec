# Mermaid Graph Visualization Examples

**Version:** 1.0.0
**Status:** Draft

## Overview

Examples of rendering Semwerk journeys and concept graphs as Mermaid diagrams.

## Simple Linear Journey

### YAML

```yaml
journey:
  key: quickstart
  nodes:
    - id: signup
      type: stage
      key: signup
      name: "Sign Up"
      connections:
        - target_node_id: verify

    - id: verify
      type: milestone
      key: email-verified
      name: "Email Verified"
      connections:
        - target_node_id: first-call

    - id: first-call
      type: stage
      key: first-api-call
      name: "First API Call"
```

### Mermaid Diagram

```mermaid
graph TD
    signup[Sign Up<br/>STAGE]
    verify[Email Verified<br/>MILESTONE]
    first-call[First API Call<br/>STAGE]

    signup --> verify
    verify --> first-call

    classDef stage fill:#4A90E2,color:#fff
    classDef milestone fill:#50C878,color:#fff

    class signup,first-call stage
    class verify milestone
```

## Branching Journey with Decision

### YAML

```yaml
journey:
  key: onboarding
  nodes:
    - id: start
      type: stage
      name: "Get Started"
      connections:
        - target_node_id: check-experience

    - id: check-experience
      type: decision
      name: "Check Experience Level"
      connections:
        - target_node_id: beginner-path
          label: "Beginner"
          condition: "experience == beginner"
        - target_node_id: advanced-path
          label: "Advanced"
          condition: "experience == advanced"

    - id: beginner-path
      type: stage
      name: "Tutorial"

    - id: advanced-path
      type: stage
      name: "API Docs"
```

### Mermaid Diagram

```mermaid
graph TD
    start[Get Started]
    check{{Check Experience<br/>Level}}
    beginner[Tutorial]
    advanced[API Docs]

    start --> check
    check -->|Beginner| beginner
    check -->|Advanced| advanced

    classDef stage fill:#4A90E2,color:#fff
    classDef decision fill:#FF6B6B,color:#fff

    class start,beginner,advanced stage
    class check decision
```

## Multi-Path Journey with Milestones

### YAML

```yaml
journey:
  key: payment-integration
  nodes:
    - id: choose-method
      type: decision
      name: "Choose Integration"
      connections:
        - target_node_id: api-path
          label: "Direct API"
        - target_node_id: sdk-path
          label: "SDK"

    - id: api-path
      type: stage
      name: "API Setup"
      connections:
        - target_node_id: test-payment

    - id: sdk-path
      type: stage
      name: "SDK Setup"
      connections:
        - target_node_id: test-payment

    - id: test-payment
      type: milestone
      name: "First Test Payment"
      connections:
        - target_node_id: production

    - id: production
      type: stage
      name: "Go Live"
```

### Mermaid Diagram

```mermaid
graph TD
    choose{{Choose<br/>Integration}}
    api[API Setup]
    sdk[SDK Setup]
    test([First Test<br/>Payment])
    prod[Go Live]

    choose -->|Direct API| api
    choose -->|SDK| sdk
    api --> test
    sdk --> test
    test --> prod

    classDef stage fill:#4A90E2,color:#fff
    classDef milestone fill:#50C878,color:#fff
    classDef decision fill:#FF6B6B,color:#fff

    class api,sdk,prod stage
    class test milestone
    class choose decision
```

## Journey with Jump-Off

### YAML

```yaml
journey:
  key: basic-onboarding
  nodes:
    - id: complete-basics
      type: stage
      name: "Complete Basics"
      connections:
        - target_node_id: choose-next

    - id: choose-next
      type: jump_off
      name: "Explore Advanced"
      connections:
        - target_journey: journey:@jrn_advanced_features
          label: "Advanced Features"
        - target_journey: journey:@jrn_integrations
          label: "Integrations"
```

### Mermaid Diagram

```mermaid
graph LR
    basics[Complete Basics]
    jump[[Explore Advanced<br/>JUMP OFF]]

    basics --> jump
    jump -.->|Advanced Features| advanced[Journey: Advanced Features]
    jump -.->|Integrations| integrations[Journey: Integrations]

    classDef stage fill:#4A90E2,color:#fff
    classDef jumpoff fill:#9B59B6,color:#fff
    classDef external fill:#E0E0E0,color:#333

    class basics stage
    class jump jumpoff
    class advanced,integrations external
```

## Concept Hierarchy

### YAML

```yaml
concepts:
  - id: cpt_authentication
    key: authentication
    name: "Authentication"

  - id: cpt_oauth
    key: oauth
    name: "OAuth 2.0"

  - id: cpt_jwt
    key: jwt
    name: "JWT"

relationships:
  - from: concept:@cpt_oauth
    to: concept:@cpt_authentication
    kind: parent

  - from: concept:@cpt_jwt
    to: concept:@cpt_authentication
    kind: parent

  - from: concept:@cpt_jwt
    to: concept:@cpt_oauth
    kind: implements
```

### Mermaid Diagram

```mermaid
graph TD
    auth[Authentication]
    oauth[OAuth 2.0]
    jwt[JWT]

    oauth -->|parent| auth
    jwt -->|parent| auth
    jwt -.->|implements| oauth

    classDef concept fill:#3498DB,color:#fff

    class auth,oauth,jwt concept
```

## Concept Network

### YAML

```yaml
concepts:
  - id: cpt_payments
    key: payment-processing
    name: "Payment Processing"

  - id: cpt_fraud
    key: fraud-detection
    name: "Fraud Detection"

  - id: cpt_pci
    key: pci-compliance
    name: "PCI Compliance"

relationships:
  - from: concept:@cpt_payments
    to: concept:@cpt_pci
    kind: depends_on

  - from: concept:@cpt_fraud
    to: concept:@cpt_payments
    kind: related

  - from: concept:@cpt_fraud
    to: concept:@cpt_pci
    kind: related
```

### Mermaid Diagram

```mermaid
graph TD
    payments[Payment Processing]
    fraud[Fraud Detection]
    pci[PCI Compliance]

    payments -->|depends on| pci
    fraud -.->|related| payments
    fraud -.->|related| pci

    classDef concept fill:#3498DB,color:#fff

    class payments,fraud,pci concept
```

## Node Type Legend

```mermaid
graph LR
    subgraph Legend
        stage[Stage<br/>Regular Step]
        milestone([Milestone<br/>Checkpoint])
        decision{{Decision<br/>Branch Point}}
        jumpoff[[Jump Off<br/>To Another Journey]]
    end

    classDef stage fill:#4A90E2,color:#fff
    classDef milestone fill:#50C878,color:#fff
    classDef decision fill:#FF6B6B,color:#fff
    classDef jumpoff fill:#9B59B6,color:#fff

    class stage stage
    class milestone milestone
    class decision decision
    class jumpoff jumpoff
```

## TypeScript Generator

```typescript
import { Journey } from '@semwerk/semspec';

function generateMermaid(journey: Journey): string {
  let mermaid = 'graph TD\n';

  // Add nodes
  for (const node of journey.nodes) {
    const shape = getNodeShape(node.type);
    mermaid += `    ${node.id}${shape.open}${node.name}${shape.close}\n`;
  }

  mermaid += '\n';

  // Add connections
  for (const node of journey.nodes) {
    for (const conn of node.connections) {
      const label = conn.label ? `|${conn.label}|` : '';
      if (conn.target_node_id) {
        mermaid += `    ${node.id} -->${label} ${conn.target_node_id}\n`;
      }
    }
  }

  // Add styling
  mermaid += '\n';
  mermaid += '    classDef stage fill:#4A90E2,color:#fff\n';
  mermaid += '    classDef milestone fill:#50C878,color:#fff\n';
  mermaid += '    classDef decision fill:#FF6B6B,color:#fff\n';

  return mermaid;
}

function getNodeShape(type: string) {
  switch (type) {
    case 'stage': return { open: '[', close: ']' };
    case 'milestone': return { open: '([', close: '])' };
    case 'decision': return { open: '{{', close: '}}' };
    case 'jump_off': return { open: '[[', close: ']]' };
    default: return { open: '[', close: ']' };
  }
}
```

## See Also

- [Journey Graph](../journey-graph.md) - Journey format specification
- [Node Types](../node-types.md) - Node type taxonomy
- [Concept Graph](../../concepts/concept-graph.md) - Concept graph format
