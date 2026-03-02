# Monitoring and Observability

## Introduction

Monitoring tells you **when** something is wrong. Observability tells you
**why**. A well-instrumented system allows operators to understand its internal
state from its external outputs -- without deploying new code to answer new
questions.

```
  ┌────────────────────────────────────────────────────────────┐
  │             THE THREE PILLARS OF OBSERVABILITY              │
  │                                                             │
  │   ┌──────────────┐  ┌──────────────┐  ┌──────────────┐    │
  │   │              │  │              │  │              │    │
  │   │    LOGS      │  │   METRICS    │  │   TRACES     │    │
  │   │              │  │              │  │              │    │
  │   │ Discrete     │  │ Numeric      │  │ Request      │    │
  │   │ events with  │  │ measurements │  │ journey      │    │
  │   │ context      │  │ over time    │  │ across       │    │
  │   │              │  │              │  │ services     │    │
  │   │ "What        │  │ "How much /  │  │ "What path   │    │
  │   │  happened?"  │  │  how fast?"  │  │  did it      │    │
  │   │              │  │              │  │  take?"      │    │
  │   └──────┬───────┘  └──────┬───────┘  └──────┬───────┘    │
  │          │                 │                  │            │
  │          └────────────┬────┴──────────────────┘            │
  │                       ▼                                    │
  │              ┌─────────────────┐                           │
  │              │  CORRELATION    │                           │
  │              │  Link logs,     │                           │
  │              │  metrics, and   │                           │
  │              │  traces by      │                           │
  │              │  request ID     │                           │
  │              └─────────────────┘                           │
  └────────────────────────────────────────────────────────────┘
```

---

## Structured Logging

Structured logs use a consistent, machine-parseable format (key-value pairs)
rather than free-form text. This enables efficient searching, filtering, and
aggregation.

### Log Levels

```
  SEVERITY HIERARCHY (most to least severe):

  ┌───────────┐
  │  FATAL    │  System is unusable, immediate attention required
  ├───────────┤
  │  ERROR    │  Operation failed, but system continues running
  ├───────────┤
  │  WARNING  │  Unexpected condition, potential problem ahead
  ├───────────┤
  │  INFO     │  Normal operational events worth recording
  ├───────────┤
  │  DEBUG    │  Detailed diagnostic information
  ├───────────┤
  │  TRACE    │  Very fine-grained diagnostic, high volume
  └───────────┘
```

### Structured Logging Pseudocode

```
FUNCTION create_logger(service_name, default_level)
  SET logger = NEW Logger
  SET logger.service = service_name
  SET logger.level = default_level
  SET logger.correlation_id = EMPTY

  RETURN logger
END FUNCTION

FUNCTION log(logger, level, message, context)
  IF SEVERITY(level) < SEVERITY(logger.level) THEN
    RETURN  // Skip logs below configured level
  END IF

  SET entry = {
    timestamp:      CURRENT_TIME_ISO8601(),
    level:          level,
    service:        logger.service,
    message:        message,
    correlation_id: logger.correlation_id,
    host:           HOSTNAME(),
    pid:            PROCESS_ID()
  }

  // Merge additional context fields
  FOR EACH key, value IN context DO
    SET entry[key] = value
  END FOR

  // Redact sensitive fields
  FOR EACH field IN ["password", "token", "secret", "ssn"] DO
    IF field IN entry THEN
      SET entry[field] = "***REDACTED***"
    END IF
  END FOR

  WRITE entry TO log_output AS serialized_format
END FUNCTION

// Usage example:
SET logger = create_logger("payment-service", "INFO")
SET logger.correlation_id = request.trace_id

log(logger, "INFO", "Payment initiated", {
  user_id: "u-1234",
  amount: 49.99,
  currency: "USD",
  method: "card"
})

log(logger, "ERROR", "Payment gateway timeout", {
  user_id: "u-1234",
  gateway_response_ms: 30000,
  retry_count: 3
})
```

### Log Aggregation Pattern

```
  ┌──────────┐  ┌──────────┐  ┌──────────┐
  │Service A │  │Service B │  │Service C │
  │  logs    │  │  logs    │  │  logs    │
  └────┬─────┘  └────┬─────┘  └────┬─────┘
       │              │              │
       └──────────────┼──────────────┘
                      ▼
              ┌───────────────┐
              │  Log Shipper  │  Collect, buffer, forward
              └───────┬───────┘
                      ▼
              ┌───────────────┐
              │  Log Store    │  Index, store, search
              └───────┬───────┘
                      ▼
              ┌───────────────┐
              │  Dashboard    │  Query, visualize, alert
              └───────────────┘
```

---

## Metric Types

Metrics are numeric measurements collected over time. There are four
fundamental metric types.

### Counter

A monotonically increasing value. It only goes up (or resets to zero on
restart). Used for totals.

```
  Value
    │
  50├─────────────────────────────────────╱──
    │                                  ╱
  40├──────────────────────────────╱────
    │                           ╱
  30├───────────────────────╱──────
    │                    ╱
  20├────────────────╱────────
    │             ╱
  10├─────────╱──────────
    │      ╱
   0├──╱───┬───┬───┬───┬───┬───┬───┬───▶ Time
         t1  t2  t3  t4  t5  t6  t7

  Examples: total_requests, errors_count, bytes_sent
```

### Gauge

A value that can go up and down. Represents a current measurement.

```
  Value
    │
  80├──────╲        ╱╲
    │       ╲      ╱  ╲         ╱╲
  60├────────╲────╱────╲───────╱──╲──
    │         ╲  ╱      ╲     ╱    ╲
  40├──────────╲╱────────╲───╱──────╲──
    │                     ╲ ╱
  20├──────────────────────╲───────────
    │
   0├───┬───┬───┬───┬───┬───┬───┬───▶ Time

  Examples: cpu_usage_percent, memory_used_gb, active_connections
```

### Histogram

Distributes observations into configurable buckets. Provides count, sum, and
per-bucket counts for calculating percentiles.

```
  Count
    │
  40├──       ┌───┐
    │         │   │
  30├──  ┌───┐│   │
    │    │   ││   │
  20├──  │   ││   │┌───┐
    │    │   ││   ││   │
  10├──┌─┤   ││   ││   │┌───┐
    │  │ │   ││   ││   ││   │
   0├──┴─┴───┴┴───┴┴───┴┴───┴──▶ Latency (ms)
     0-50 50-100 100-250 250-500 500+

  Buckets capture the distribution of values.
  Examples: request_duration, response_size
```

### Summary

Similar to histogram but calculates quantiles (percentiles) on the client
side. Pre-computes p50, p90, p99, etc.

```
  Quantile │ Value
  ─────────┼──────
  p50      │ 85 ms      50% of requests complete in <= 85ms
  p90      │ 210 ms     90% of requests complete in <= 210ms
  p95      │ 380 ms     95% of requests complete in <= 380ms
  p99      │ 820 ms     99% of requests complete in <= 820ms
  p999     │ 1450 ms    99.9% of requests complete in <= 1450ms
```

### Metrics Collection Pseudocode

```
FUNCTION create_counter(name, description, labels)
  SET counter = NEW Counter
  SET counter.name = name
  SET counter.description = description
  SET counter.labels = labels
  SET counter.value = 0
  REGISTER counter WITH metrics_registry
  RETURN counter
END FUNCTION

FUNCTION create_histogram(name, description, buckets)
  SET histogram = NEW Histogram
  SET histogram.name = name
  SET histogram.description = description
  SET histogram.buckets = buckets  // e.g., [10, 50, 100, 250, 500, 1000]
  SET histogram.observations = EMPTY LIST
  REGISTER histogram WITH metrics_registry
  RETURN histogram
END FUNCTION

// Usage:
SET request_counter = create_counter(
  "http_requests_total",
  "Total HTTP requests",
  ["method", "path", "status"]
)

SET latency_histogram = create_histogram(
  "http_request_duration_ms",
  "Request latency in milliseconds",
  [10, 50, 100, 250, 500, 1000, 5000]
)

FUNCTION handle_request(request)
  SET start_time = CURRENT_TIME_MS()

  SET response = process(request)

  SET duration = CURRENT_TIME_MS() - start_time
  INCREMENT request_counter WITH labels {
    method: request.method,
    path: request.path,
    status: response.status
  }
  OBSERVE latency_histogram WITH value duration

  RETURN response
END FUNCTION
```

---

## Distributed Tracing

In a system composed of multiple services, a single user request may traverse
many components. Distributed tracing follows a request's journey by propagating
a unique trace identifier across service boundaries.

```
  ┌─────────────────────────────────────────────────────────────────┐
  │  TRACE: abc-123                                                  │
  │                                                                  │
  │  ┌─── Span A: API Gateway (12ms) ──────────────────────────┐   │
  │  │                                                          │   │
  │  │  ┌─── Span B: Auth Service (3ms) ───┐                   │   │
  │  │  └──────────────────────────────────┘                   │   │
  │  │                                                          │   │
  │  │  ┌─── Span C: Order Service (45ms) ─────────────────┐   │   │
  │  │  │                                                   │   │   │
  │  │  │  ┌─── Span D: Database Query (8ms) ──┐           │   │   │
  │  │  │  └───────────────────────────────────┘           │   │   │
  │  │  │                                                   │   │   │
  │  │  │  ┌─── Span E: Payment Service (30ms) ────────┐   │   │   │
  │  │  │  │                                            │   │   │   │
  │  │  │  │  ┌─── Span F: Gateway Call (22ms) ───┐    │   │   │   │
  │  │  │  │  └───────────────────────────────────┘    │   │   │   │
  │  │  │  └────────────────────────────────────────────┘   │   │   │
  │  │  └───────────────────────────────────────────────────┘   │   │
  │  └──────────────────────────────────────────────────────────┘   │
  │                                                                  │
  │  Timeline:                                                       │
  │  0ms     10ms     20ms     30ms     40ms     50ms     57ms      │
  │  |───A────────────────────────────────────────────────────|     │
  │    |─B─|                                                        │
  │         |───────────────C──────────────────────────────|        │
  │          |──D──|                                                 │
  │                  |──────────────E───────────────|               │
  │                     |──────────F──────────|                     │
  └─────────────────────────────────────────────────────────────────┘
```

### Tracing Pseudocode

```
FUNCTION start_trace(request)
  IF request.headers CONTAINS "trace-id" THEN
    SET trace_id = request.headers["trace-id"]
    SET parent_span_id = request.headers["span-id"]
  ELSE
    SET trace_id = GENERATE_UNIQUE_ID()
    SET parent_span_id = NONE
  END IF

  SET span = NEW Span
  SET span.trace_id = trace_id
  SET span.span_id = GENERATE_UNIQUE_ID()
  SET span.parent_id = parent_span_id
  SET span.service = CURRENT_SERVICE_NAME()
  SET span.operation = request.path
  SET span.start_time = CURRENT_TIME_NS()
  SET span.tags = {}

  RETURN span
END FUNCTION

FUNCTION finish_span(span, status)
  SET span.end_time = CURRENT_TIME_NS()
  SET span.duration = span.end_time - span.start_time
  SET span.status = status

  EXPORT span TO tracing_collector
END FUNCTION

FUNCTION propagate_context(span, outgoing_request)
  SET outgoing_request.headers["trace-id"] = span.trace_id
  SET outgoing_request.headers["span-id"] = span.span_id
  RETURN outgoing_request
END FUNCTION

// Usage:
FUNCTION handle_order(request)
  SET span = start_trace(request)
  SET span.tags["order.type"] = request.body.type

  // Call downstream service with propagated context
  SET payment_request = CREATE_REQUEST("/pay", request.body.payment)
  SET payment_request = propagate_context(span, payment_request)
  SET payment_response = SEND(payment_request)

  IF payment_response.status == "success" THEN
    finish_span(span, "OK")
  ELSE
    SET span.tags["error"] = TRUE
    SET span.tags["error.message"] = payment_response.error
    finish_span(span, "ERROR")
  END IF
END FUNCTION
```

---

## Alerting Strategies

Not all anomalies deserve a page to an on-call engineer. A thoughtful alerting
strategy reduces noise and ensures that alerts are actionable.

### Alert Types

```
  ┌──────────────────────────────────────────────────────────┐
  │                  ALERTING STRATEGIES                      │
  │                                                           │
  │  THRESHOLD          ANOMALY            COMPOSITE          │
  │  ─────────          ───────            ─────────          │
  │                                                           │
  │  value > X    value deviates from    (cond A AND          │
  │               predicted baseline      cond B) for         │
  │                                       duration T          │
  │  ┌────────┐   ┌────────────────┐   ┌────────────────┐   │
  │  │  ___   │   │  ~predicted~   │   │  if A && B     │   │
  │  │ /   \  │   │ /  actual  \   │   │  for 5 min     │   │
  │  │/     \ │   │/            \  │   │  then alert    │   │
  │  │-------|│   │───anomaly────│ │   │                │   │
  │  │threshold│   │  window     │ │   │                │   │
  │  └────────┘   └────────────────┘   └────────────────┘   │
  │                                                           │
  │  BEST FOR:     BEST FOR:           BEST FOR:             │
  │  Known limits  Seasonal patterns   Reducing false         │
  │  Hard caps     Organic growth      positives              │
  │  SLO burn rate Unknown baselines   Complex conditions     │
  └──────────────────────────────────────────────────────────┘
```

### Alerting Pseudocode

```
FUNCTION evaluate_threshold_alert(metric, threshold, duration, severity)
  SET values = QUERY metric OVER LAST duration
  SET breaching_count = 0

  FOR EACH value IN values DO
    IF value > threshold THEN
      SET breaching_count = breaching_count + 1
    END IF
  END FOR

  SET breach_ratio = breaching_count / values.count

  IF breach_ratio >= 0.9 THEN  // 90% of datapoints breaching
    FIRE alert WITH severity AND message FORMAT(
      "{metric} exceeded {threshold} for {breach_ratio * 100}% of last {duration}"
    )
  END IF
END FUNCTION

FUNCTION evaluate_anomaly_alert(metric, window, sensitivity)
  SET historical = QUERY metric OVER LAST 7 DAYS same_time_of_day
  SET current = QUERY metric OVER LAST window

  SET baseline_mean = MEAN(historical)
  SET baseline_stddev = STANDARD_DEVIATION(historical)
  SET current_mean = MEAN(current)

  SET deviation = ABSOLUTE(current_mean - baseline_mean) / baseline_stddev

  IF deviation > sensitivity THEN
    FIRE alert WITH message FORMAT(
      "{metric} deviates {deviation} sigma from baseline "
      + "(expected: {baseline_mean}, actual: {current_mean})"
    )
  END IF
END FUNCTION

FUNCTION evaluate_composite_alert(conditions, duration)
  SET all_true_since = NONE

  EVERY 30 SECONDS DO
    SET all_met = TRUE
    FOR EACH condition IN conditions DO
      IF NOT evaluate_condition(condition) THEN
        SET all_met = FALSE
        BREAK
      END IF
    END FOR

    IF all_met AND all_true_since == NONE THEN
      SET all_true_since = CURRENT_TIME()
    ELSE IF NOT all_met THEN
      SET all_true_since = NONE
    END IF

    IF all_true_since != NONE THEN
      IF CURRENT_TIME() - all_true_since >= duration THEN
        FIRE alert WITH message "Composite condition met for {duration}"
        SET all_true_since = NONE  // Reset after firing
      END IF
    END IF
  END EVERY
END FUNCTION
```

---

## SLI, SLO, and SLA

```
  ┌─────────────────────────────────────────────────────────┐
  │                                                          │
  │  SLI (Service Level Indicator)                           │
  │  ──────────────────────────────                          │
  │  A quantitative measure of service behavior.             │
  │  Example: "Proportion of requests served in < 200ms"     │
  │                                                          │
  │  SLO (Service Level Objective)                           │
  │  ──────────────────────────────                          │
  │  A target value for an SLI over a time window.           │
  │  Example: "99.9% of requests in < 200ms over 30 days"   │
  │                                                          │
  │  SLA (Service Level Agreement)                           │
  │  ──────────────────────────────                          │
  │  A contract with consequences if the SLO is not met.     │
  │  Example: "If availability < 99.9%, credit 10% of bill"  │
  │                                                          │
  │  Relationship:                                           │
  │  SLI ──measures──▶ SLO ──promises──▶ SLA               │
  │                                                          │
  └─────────────────────────────────────────────────────────┘
```

### Error Budget Calculation

The error budget is the amount of unreliability your SLO permits. If your
SLO is 99.9% availability over 30 days, you have a 0.1% error budget.

```
FUNCTION calculate_error_budget(slo_target, window_days)
  SET total_minutes = window_days * 24 * 60
  SET allowed_failure_rate = 1.0 - slo_target
  SET budget_minutes = total_minutes * allowed_failure_rate

  RETURN {
    total_minutes: total_minutes,
    budget_minutes: budget_minutes,
    budget_percentage: allowed_failure_rate * 100
  }
END FUNCTION

FUNCTION check_error_budget_remaining(slo_target, window_days, failures_so_far)
  SET budget = calculate_error_budget(slo_target, window_days)
  SET remaining = budget.budget_minutes - failures_so_far
  SET burn_rate = failures_so_far / budget.budget_minutes

  IF remaining <= 0 THEN
    LOG "ERROR BUDGET EXHAUSTED -- freeze deployments"
    ALERT operations WITH severity "critical"
  ELSE IF burn_rate > 0.5 THEN
    LOG "Error budget burn rate elevated: {burn_rate * 100}%"
    ALERT operations WITH severity "warning"
  END IF

  RETURN {
    total_budget: budget.budget_minutes,
    consumed: failures_so_far,
    remaining: remaining,
    burn_rate: burn_rate
  }
END FUNCTION

// Example:
// SLO = 99.9% over 30 days
// Total minutes = 43200
// Budget = 43.2 minutes of downtime allowed
// If 25 minutes consumed: 18.2 remaining, 57.9% burn rate -> warning
```

### Multi-Window Burn Rate Alerting

A single threshold can be too noisy or too slow. Multi-window alerting
compares burn rates over different time windows to catch both fast burns
(outages) and slow burns (degradation).

```
  Fast burn:  consuming > 14.4x normal rate over 1 hour
              AND > 14.4x over last 5 minutes → PAGE

  Slow burn:  consuming > 3x normal rate over 3 days
              AND > 3x over last 6 hours → TICKET

  ┌───────────────────────────────────────────────┐
  │  Window   │  Burn Rate  │  Budget at Risk  │  │
  │───────────┼─────────────┼──────────────────│  │
  │  1 hour   │  > 14.4x    │  2% in 1 hour   │  │
  │  6 hours  │  > 6x       │  5% in 6 hours  │  │
  │  3 days   │  > 3x       │  10% in 3 days  │  │
  └───────────────────────────────────────────────┘
```

---

## Dashboarding Principles

A dashboard should answer specific questions, not just display data. Follow
these principles:

```
  ┌──────────────────────────────────────────────────────┐
  │             DASHBOARD DESIGN PRINCIPLES               │
  │                                                       │
  │  1. START WITH QUESTIONS                              │
  │     "Is the service healthy?"                         │
  │     "Are users affected?"                             │
  │     "Where is the bottleneck?"                        │
  │                                                       │
  │  2. LAYER INFORMATION                                 │
  │     ┌────────────────────────┐                        │
  │     │  Overview: 4 golden    │  ◀── Glance (5 sec)   │
  │     │  signals (traffic,     │                        │
  │     │  errors, latency,      │                        │
  │     │  saturation)           │                        │
  │     ├────────────────────────┤                        │
  │     │  Detail: Breakdowns    │  ◀── Investigate       │
  │     │  by endpoint, status,  │      (2 min)           │
  │     │  dependency            │                        │
  │     ├────────────────────────┤                        │
  │     │  Debug: Individual     │  ◀── Root cause        │
  │     │  traces, logs, raw     │      (10+ min)         │
  │     │  queries               │                        │
  │     └────────────────────────┘                        │
  │                                                       │
  │  3. USE THE FOUR GOLDEN SIGNALS                       │
  │     - Traffic:    requests per second                  │
  │     - Errors:     error rate / error ratio             │
  │     - Latency:    p50, p90, p99 response time         │
  │     - Saturation: CPU, memory, queue depth             │
  │                                                       │
  │  4. AVOID ANTI-PATTERNS                               │
  │     - Too many panels (limit to 8-12 per view)        │
  │     - Vanity metrics that do not drive action          │
  │     - Missing time-range context                       │
  │     - No annotation of deployments or incidents        │
  └──────────────────────────────────────────────────────┘
```

---

## Practice Problems

1. **Structured Logging Design:** Design a structured logging function that
   automatically includes request context (trace ID, user ID, session ID),
   supports multiple output destinations, and redacts any field matching a
   configurable list of sensitive patterns.

2. **Metric Selection:** For each scenario, state which metric type (counter,
   gauge, histogram, or summary) is most appropriate and why:
   a. Number of items currently in a processing queue
   b. Total bytes transferred since server start
   c. Distribution of database query execution times
   d. Current temperature of a hardware sensor

3. **Trace Analysis:** Given the trace diagram in this document, identify the
   critical path (longest chain of sequential spans). If Span F's latency
   increased from 22ms to 200ms, what would be the new total trace duration?
   What optimization would reduce overall trace time the most?

4. **Alert Design:** Your service has an SLO of 99.95% availability over 30
   days. Design a multi-window burn-rate alerting strategy with at least three
   windows. For each window, specify the burn-rate threshold, the time window,
   and the response action (page, ticket, or log).

5. **Error Budget Policy:** Write pseudocode for an error budget policy
   function that: (a) freezes non-critical deployments when the budget is
   below 25%, (b) requires additional review for all changes when below 50%,
   and (c) alerts leadership when the budget is exhausted.

6. **Dashboard Critique:** You inherit a dashboard with 30 panels, half of
   which show raw counter values (not rates), no percentile breakdowns for
   latency, and no deployment annotations. List the specific changes you
   would make and explain the rationale for each.
