# Success Metrics Schema

**Version:** 1.0.0
**Status:** Draft

## Overview

Success metrics define how to measure journey effectiveness and user progress.

## Format

```yaml
success_metrics:
  - metric: <string>              # Metric name
    target: <string>              # Target value
    description: <string>         # What this measures
    type: <enum>                  # time | rate | count | score
    unit: <string>                # Optional: measurement unit
```

## Metric Types

### time

Measures duration or time-based metrics.

```yaml
- metric: time_to_first_call
  type: time
  target: "< 30 minutes"
  description: "Time from signup to first API call"
  unit: minutes
```

### rate

Measures percentages or ratios.

```yaml
- metric: completion_rate
  type: rate
  target: "> 70%"
  description: "Percentage of users who complete journey"
  unit: percentage
```

### count

Measures absolute numbers.

```yaml
- metric: api_calls_made
  type: count
  target: "> 10"
  description: "Number of API calls made during onboarding"
  unit: calls
```

### score

Measures quality or satisfaction scores.

```yaml
- metric: nps_score
  type: score
  target: "> 50"
  description: "Net Promoter Score after onboarding"
  unit: nps
```

## Common Metrics

### Onboarding Metrics

```yaml
success_metrics:
  - metric: time_to_first_value
    type: time
    target: "< 1 hour"
    description: "Time until user derives first value"

  - metric: setup_completion_rate
    type: rate
    target: "> 80%"
    description: "Users who complete initial setup"

  - metric: activation_rate
    type: rate
    target: "> 60%"
    description: "Users who reach activation milestone"
```

### Conversion Metrics

```yaml
success_metrics:
  - metric: trial_to_paid_conversion
    type: rate
    target: "> 25%"
    description: "Trial users who convert to paid"

  - metric: average_time_to_conversion
    type: time
    target: "< 14 days"
    description: "Days from trial start to paid conversion"
```

### Engagement Metrics

```yaml
success_metrics:
  - metric: weekly_active_users
    type: count
    target: "> 1000"
    description: "Users active in past 7 days"

  - metric: feature_adoption_rate
    type: rate
    target: "> 40%"
    description: "Users who use advanced features"
```

### Satisfaction Metrics

```yaml
success_metrics:
  - metric: csat_score
    type: score
    target: "> 4.0"
    description: "Customer Satisfaction Score (1-5)"
    unit: rating

  - metric: documentation_helpfulness
    type: score
    target: "> 4.5"
    description: "Doc helpfulness rating (1-5)"
    unit: rating
```

## Node-Level Metrics

Individual nodes can have specific metrics:

```yaml
nodes:
  - id: email-verification
    type: milestone
    metrics:
      conversion_target: 0.95       # 95% should verify
      median_time_minutes: 2        # Median time to verify
      max_duration_minutes: 30      # Timeout threshold
```

## Journey-Level Metrics

Overall journey success criteria:

```yaml
journey:
  success_metrics:
    - metric: overall_completion_rate
      type: rate
      target: "> 65%"

    - metric: average_journey_duration
      type: time
      target: "< 45 minutes"

    - metric: user_satisfaction
      type: score
      target: "> 4.2"
```

## Metric Aggregation

```yaml
success_metrics:
  - metric: funnel_dropoff
    type: rate
    target: "< 20%"
    description: "Users who drop off at any stage"
    aggregation: sum
    segments:
      - stage: signup
        dropoff_rate: "5%"
      - stage: verification
        dropoff_rate: "8%"
      - stage: first-call
        dropoff_rate: "7%"
```

## Use Cases

### 1. Journey Analytics

```typescript
// Measure journey performance
const metrics = journey.success_metrics;
const actual = measureJourney(journey);

metrics.forEach(metric => {
  const achieved = evaluateTarget(metric.target, actual[metric.metric]);
  console.log(`${metric.metric}: ${achieved ? '✓' : '✗'}`);
});
```

### 2. A/B Testing

```yaml
# Journey variant A
success_metrics:
  - metric: completion_rate
    target: "> 70%"

# Journey variant B
success_metrics:
  - metric: completion_rate
    target: "> 75%"        # Testing if variant B improves completion
```

### 3. SLO Monitoring

```yaml
success_metrics:
  - metric: p95_onboarding_time
    type: time
    target: "< 60 minutes"
    description: "95th percentile onboarding time"
    alert_threshold: "> 90 minutes"
```

## See Also

- [Journey Graph](./journey-graph.md) - Using metrics in journeys
- [Node Types](./node-types.md) - Node-level metrics
